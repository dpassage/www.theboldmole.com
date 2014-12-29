---
layout: post
title: "Migrating to Swift: Unit Tests"
description: ""
category: 
tags: []
---
In the [last post](/2014/11/23/slowly-converting-objective-c-to-swift.html),
I migrated a utility class in my app from Objective-C to Swift. I didn't have
unit tests on this class (bad programmer!), but I tested it by hand and things
seemed to work OK.

Before checking in, I ran my unit tests. Boom, won't compile:

~~~
In file included from /Users/dpassage/Documents/CatApp/Meow/MeowTests/MeowViewModelTest.m:15:
/Users/dpassage/Documents/CatApp/Meow/Meow/MeowStore.h:12:9: fatal error: 'Meow-Swift.h' file not found
#import "Meow-Swift.h"
        ^
1 error generated.
~~~

OK, great. Turns out that the auto-generated header that describes the Swift
classes to Objective-C isn't made available outside the module, which means
it isn't available to the unit tests.

To be fair, Apple did [release note](https://developer.apple.com/library/ios/releasenotes/DeveloperTools/RN-Xcode/Chapters/xc6_release_notes.html#//apple_ref/doc/uid/TP40001051-CH4-DontLinkElementID_43)
this, but their advice is pretty short:

> Tests for Swift code should be written in Swift.

Fair enough, if I was testing a class written in Swift. I'm not, and don't
feel like rewriting test code into Swift if I don't have to.

How do I address this? Digging in, turns out there were two problems.

The first was caused by the import of `Meow-Swift.h` into header files of other
classes which use the class I converted, `MeowUploader`. I addressed those
by turning the include into a forward declaration:

~~~
@class MeowUploader;
~~~
{: .language-objective-c}

I then added the `Meow-Swift.h` header in the `.m` file to get access to
the method definitions and the like.

This is good hygiene in any case; forward declarations like this keep your
includes smaller.

The second problem was really in one place. I had used the class to define
a mock object, to test that a particular method of `MeowUploader` was
called when a change is made to another object. I'm using [OCMock](http://ocmock.org/)
for mocking.

In this case, I ended up just writing a mock object by hand. There was only
one method and the mock was easily injectible.

The next step in Swifitying my whole app is re-writing all my tests in Swift.
Then I can start in on it in earnest.


