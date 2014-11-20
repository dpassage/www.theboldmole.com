---
layout: post
title: "Sounds and MVVM"
description: ""
category:
tags: mvvm,ios
---
I don't know why the following bit of insight took me so long...

I'm working on converting an existing app I've been working on to
[ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa) and MVVM. It
starts simply enough: find all the places where my `UIViewController`s refer
to model objects directly, and then figure out how to substitute a ViewModel
object.

Something I struggled with a little bit is what to do about sounds; this app
allows users to record sounds, play them back later, and submit them to our
central server for folding, spindling, and mutilation. So my ViewModel has
a `RACSignal` you can subscribe to which plays the sound as a side effect,
sends timing information as the sound plays, and completes when the sound
finishes playing.

This didn't feel right, though. And then I hit on something: the recording and
playing of sounds belongs to the View layer, not the ViewModel.

I got there by thinking about porting this thing to another platform. One of
the principles of MVVM is that the ViewModel acts as a layer between the UI
code and the model code. The goal is that the ViewModel should be flexible
enough that you could port the code to another platform and only change the
View layer, not the Model or ViewModel.

(Of course, for an app written in Objective-C, there's a limit on the number
of other platforms you can port to. But bear with me.)

So the code I have in the ViewModel which plays the sound doesn't belong there.
It should go in the View layer. All my ViewModel should do is provide a
reference to the sound itself, maybe with a ViewModel to wrap it, maybe not.

Another way I thought about it was presenting the sound information to a
hearing impaired user. You could "play" the sound by showing the level
information, or even by playing with phone vibration. All of that is about
presentation.

I think I was led astray by the API I made using a signal to report
progress playing the sound. There's no reason things in the View layer can't
use signals to communicate.

So! Back into the breach.
