---
layout: post
title: "Slowly converting Objective C to Swift"
description: ""
category: 
tags: [ios, swift]
---
I've decided to take the plunge on the same app from the previous post, and
convert it to Swift.

The start of the process is pretty straightforward. Apple even gives you
some
[starting directions](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/BuildingCocoaApps/Migration.html).
Which almost work...

The first thing I ran into was a linker error after I re-wrote my first class
in Swift:

~~~~
Undefined symbols for architecture x86_64:
  "_OBJC_CLASS_$_Chunk", referenced from:
      objc-class-ref in ViewController.o
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
~~~~

I re-read the "Migration" chapter in the above document. Then, I read down into
the "Mix and Match" chapter, which included this tidbit:

> Import the Swift code from that target into any Objective-C .m file within that target using this syntax and substituting the appropriate name: \\
> `#import "ProductModuleName-Swift.h"`

Adding an include file fixes a *linker* error. Thanks, Apple!
