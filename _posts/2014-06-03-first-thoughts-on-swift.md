---
layout: post
title: "First thoughts on Swift"
description: ""
category:
tags: [swift]
---
Everyone's writing about it--Apple sort of dropped a bomb on their developers
yesterday by announcing a new programming language, called
[Swift](https://developer.apple.com/library/prerelease/ios/documentation/swift/conceptual/swift_programming_language/index.html).

Here are some of my thoughts about it, oriented towards an audience that
hasn't done application development on Apple platforms. I was prompted to
write this by some comments a CS professor friend made on Facebook, but
there's some other stuff here, too.

First, some thoughts about how we got here. Ever since the Next acquisition,
Apple's been investing in a platform in which all of the user-level libraries
are written in either Objective-C or plain C, and even the plain C API's use
a lot of the Objective-C paradigms like memory management.

And we're not talking about libc here. There are tens of thousands of API
items--classes, methods, functions, constants, enums--which give developers
access to everything from a high-level object store (CoreData) to media
management and manipulation (AVFoundation) to UI objects (UIKit/AppKit) to
everything else (Foundation).

That means that any new platform language needs to adapt to that object model.
Otherwise, Apple would need to rewrite too much of the platform to make that
feasible--both for them and for the developer community.

This effectively gave Apple a set of constraints they had to work with:

* Memory model. In the beginning, Objective-C had a manually managed reference
counting system to ensure that unused objects are freed when no longer in
use--but not until then. This eventually evolved into Automatic Reference
Counting, in which most (but not all) memory management concerns are
abstracted away from the developer. (OS X had support for garbage collection
for a few years, but it was never supported for iOS apps and has now been
deprecated.) It's fair to talk about the pros and cons of ARC, but given the
investment Apple has made in it, it's fair to say it had to live on in Swift.

* Structure-based primitive types. Many parts of the UI libraries use small C
structs as primitive data types--mostly for sizes and dimensions of UI
elements. By using C structs, these are effectively treated as values in the
runtime, trading some extra copying on assignment for the relative simplicity
of allocating them on the stack for locals and arguments. Swift has to handle
these, too.

* Concurrency. On Apple platforms, concurrency has historically been handled
as part of the platform, not part of the base language or its runtime. In
Objective-C applications, higher-level
[NSOperationQueues](http://nshipster.com/nsoperation/) and lower-level
[dispatch queues](https://developer.apple.com/library/mac/documentation/performance/reference/gcd_libdispatch_ref/Reference/reference.html)
are used by application developers to manage background tasks.

* Crappy exceptions. Unlike better-behaved languages like
[Ruby](http://www.ruby-doc.org/core-2.1.1/Exception.html),
[Python](https://docs.python.org/2/tutorial/errors.html), and even
[C++](http://www.tutorialspoint.com/cplusplus/cpp_exceptions_handling.htm),
when an exception gets thrown in Objective-C, there are very few guarantees
about what happened to your stack, allocated objects, and so forth. The best
you can hope for is logging something informative as the ship goes down.

Those are just some of the constraints. Given the limited degrees of freedom,
why make the investment?

Objective-C is a mess. It started life as a blend of one of the highest-level
languages to exist at the time (SmallTalk) and one of the lowest (C). This
creates a fair amount of cognitive dissonance between the dynamic
object-oriented concepts and the need to manage pointers and array bounds
manually.

In fact, it's [really](http://nakedsecurity.sophos.com/2014/02/24/anatomy-of-a-goto-fail-apples-ssl-bug-explained-plus-an-unofficial-patch/)
[easy](http://www.imore.com/understanding-apples-ssl-tls-bug)
[to](https://www.imperialviolet.org/2014/02/22/applebug.html) write broken,
insecure, crashing code in Objective-C--mostly becaues it's just C with some
fancy wrapper stuff layered on top. Apple's made a huge investment in
[static analysis](http://clang-analyzer.llvm.org) tools to help prefent these
issues, but you're talking about solving the halting problem in a languge
whose specification is full of
[undefined behavior](http://blog.regehr.org/archives/213).

At first glance, Swift eliminates a whole bunch of the most common ways an
Objective-C programmer can screw up. Everything is higher-level--there are no
unbounded arrays, implicit casts, or C-style pointer arithmetic. If an object
reference might be `nil`, you have to deal with that every time you assign
it. And there looks to be a whole lot less undefined behavior in general.

I'm optimistic about it. I've already started trying to port some of my
in-process app code into Swift; the tools aren't quite there yet, but the
results are interesting.

That said, here are some of the challenges I see:

* Third-party libraries. The developer community has built hundreds of
[great](http://afnetworking.com)
[open-source](https://github.com/magicalpanda/MagicalRecord)
[libraries](https://github.com/ccgus/fmdb) in Objective-C, including a
completely non-Apple managed
[packaging and dependency system](http://cocoapods.org). Hopefully it will be
straightforward to expose those Objective-C libraries to Swift--at the moment,
the interface feels like it has some rough edges.

* Namespacing. A continuing challenge for developers is Objective-C's flat
namespace. While there is a
[naming convention](https://developer.apple.com/library/ios/documentation/cocoa/conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html)
for classes to help avoid this issue, it's not foolproof, especially when
Apple
[violates their own guidance](https://github.com/Mantle/Mantle/issues/341).
There was some mention of namespace support in the keynote, but it's not
well explained in the available documentation.

* Introspection and method lookup. Objective-C does dynamic method lookup and
gives developers access to the runtime itself, enabling things like replacing
methods at runtime. While much danger lies here, third-party libraries have
used this capability to
[great effect](https://github.com/ReactiveCocoa/ReactiveCocoa). Hopefully
Swift will allow those capabilities to live on.

The upshot? Objective-C's heritage has been a decidedly mixed bag--it enables
developers to to both stupid and amazing things. Hopefully Swift will evolve
to eliminate most of the stupid, but keep the amazing.
