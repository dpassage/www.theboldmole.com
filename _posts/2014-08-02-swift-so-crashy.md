---
layout: post
title: "Swift so crashy!"
description: ""
category: 
tags: [swift]
---
One of the best resources available for those trying to learn iOS programming
is Paul Hagerty's course on iTunes. And I say that as someone usually
reluctant to say anything good about Stanford.

When I was learning about RubyMotion, I decided to work through the course and
its exercising doing the programming in Ruby. I only got into the first few
weeks, but it gave me at least a passing feel for the environment. 

So it was natural to take on that course as a way of learning Swift. The first
couple of lectures are a walkthrough of building the skeleton of a memory
game app in Xcode and Objective-C. Right away, I saw how Swift's language
features changed how you want to build things. For instance, the sample project
uses subclassing to provide a generic Deck of cards or a specific
PlayingCardDeck for standard playing cards; this is a place where I plan on
using generics instead.

But that's for another post. This post is about the rough edges in a beta
language and how that can drive you nuts.

Let's take the code that shuffles the deck. In the sample project, Paul's got:

```
- (Card *)drawRandomCard
{
    Card *randomCard = nil;
    if ([self.cards count]) {
        unsigned index = arc4random() % [self.cards count];
        randomCard = self.cards[index];
        [self.cards removeObjectAtIndex:index];
}
    return randomCard;
}
```

Pretty straightforward. `arc4random()` is defined to return `u_int32_t`, i.e.,
a 32-bit unsigned integer. `NSArray`'s `count` method returns an unsigned
integer as well. It's all good.

Let's translate the above code into Swift:

```
func drawRandomCard() -> Card? {
    var randomCard: Card? = nil

    if cards.count > 0 {
        let index = Int(arc4random()) % cards.count
        randomCard = cards[index]
        cards.removeAtIndex(index)
    }
    return randomCard
}
```

Pretty straightforward. Swift's `Array.count` returns an `Int`, and it expects
an `Int` for indexing, which is a change from `NSArray`. Also, Swift doesn't
do any implied casting--you have to explicitly cast all numbers involved in
operations like `%`. So I cast the return value of `arc4random()` from `UInt32`
to `Int` and all's good.

Until I run it. And the code crashes intermittenly when drawing a card. And it
crashes with `EXC_BAD_INSTRUCTION` and no other errors messages.

As of this writing, `lldb` isn't really useful for Swift yet. The debugger said
the crash occurred either at the declaration of `randomCard` or at the `if`
statement. So I spent a while going down the rathole of making sure my array
was still there. And spent many frustrating hours down that rathole.

After some `NSLog()` debugging, I finally figured it out. The crash was actually
occuring inside the `if` statement, in this line:

```
        let index = Int(arc4random()) % cards.count
```

`arc4random()` returns an unsigned 32-bit integer. When running in 32 bit mode,
as I was since I was using the iPhone 4S simulator, Swift's `Int` is a singed
32-bit quantity. So about half the time, the random number's too big for the
cast.

In C, that's basically OK; it'll just copy the bits and it'll silently become
a negative number. But in Swift, that's a runtime error. And a runtime error
means a crash.

I rewrote the code thusly:

```
        let index = Int(arc4random_uniform(UInt32(cards.count)))
```

...casts all around, use `arc4random_uniform()` instead of `arc4_random` to
push the `%` into the library.

I'm sure my code is safer now, but it's also a little uglier.

Plus, it would have been really, really nice if there had been a message of
some sort about why it was crashing.


