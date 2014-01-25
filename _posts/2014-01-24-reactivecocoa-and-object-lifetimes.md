---
layout: post
title: "ReactiveCocoa and object lifetimes"
description: ""
category:
tags: [ios, reactivecocoa]
---
I've been doing more and more with
[ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa). It's changing
a lot of how I'm thinking about code in iOS, and making some things much
simpler.

There's more to be said there, but I'll start today with what I learned about
object lifetimes.

I've got an app that plays a short sound when the user hits a button. Because
`AVAudioPlayer` is a pain in the behind, I wrapped it in a class called
`SoundPlayer` to try and hide some of the complexity. In particular, I don't
care if the sound ends prematurely. The money method in the class is this:

    - (void)playWithCompletionBlock:(void(^)())completionBlock
               andProgressBlock:(void(^)(NSTimeInterval current, NSTimeInterval duration))progressBlock;

Tell me when it's done and what the progress is, otherwise leave me alone.

Given that method signature, this was a natural to wrap in a `RACSignal`:

    - (RACSignal *)playSoundSignal
    {
        RACSignal *soundSignal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
            SoundPlayer *soundPlayer = [[SoundPlayer alloc] initWithURL:self.meow.soundFileURL];

            [soundPlayer playWithCompletionBlock:^{
                [subscriber sendCompleted];
            } andProgressBlock:^(NSTimeInterval current, NSTimeInterval duration) {
                [subscriber sendNext:@(current / duration)];
            }];

            return nil;
        }];

        return soundSignal;
    }

All well and good. The signal sends the progress information, and completes
when the sound's done playing. Since the `SoundPlayer` class doesn't have any
way to cancel, I figure I may as well just return `nil` for the disposable.

Compile, build, run, boom. Weird behavior - the completion block never gets
called, sometimes it crashes, etc. Fire up Instruments with the Zombies
template - and that `SoundPlayer` object is getting sent signals after it's
been deallocated. Huh-wha?

Poking around, it appears that the SoundPlayer object gets released at the
end of the block passed to `createSignal:`. At that point, the only thing with
a reference to it is the timer it uses internally to decide when to send
progress updates. When the sound finishes playing and I invalidate the
timer - boom, whole object goes away, right in the middle of that method!

So okay - how do I add a reference? Playing around a bit, I change the
subscriber block so it returns a disposable:

        return [RACDisposable disposableWithBlock:^{
            [soundPlayer description];
        }];

Problem goes away! But wait - the code that subscribes to this signal doesn't
hang on to the disposable. Why did this work?

The answer I found deep in the [memory management][RACmemory] documentation.
When the signal is created, the ReactiveCocoa library itself keeps a reference
to the `RACDisposable` created at subscription time. When the signal finishes
completing, it calls `dispose` on it for you.

To my mind, this changes the semantics of `RACDisposable` a little bit. It's
not just a place to clean up after your signal, or to let users of the signal
cancel an operation in process. It's also a way to ensure that resources
your signal uses aren't disposed of until everything's done.

[RACmemory]: https://github.com/ReactiveCocoa/ReactiveCocoa/blob/master/Documentation/MemoryManagement.md
