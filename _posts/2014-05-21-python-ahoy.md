---
layout: post
title: "Python ahoy!"
description: ""
category: 
tags: []
---
I of course started writing with good intentions, then got sucked into the next
project...

It's an election-related website, and it's built in Python and Django. So I
get to learn a whole new language and set of idioms. 

The structure of a Django site does at least roughly translate. Models are
models; controllers are views; views are templates.

What doesn't really translate is the unit of abstraction. Rails sets things up
from the outset to encourage a certain code organization--not just in the MVC
layering, but in how files are organized on disk, and even that each class
gets its own file. And there's grown a whole set of conventions and knowledge
about how to do good object-oriented design in Ruby.

In contrast, Django appears to want the main unit of abstraction to be the
"app", which is a combination of models, views, and templates, all together.
Which is fair enough.

What I'm seeing in the real world, though, is that there aren't good signals
to the developer on when she should separate some code out into another app.
And the default layout has all your models and views in a single file each--
`models.py` and `views.py`.

The upshot is that it's very easy to end up with thousands of lines of code in
each one.

Let the code spelunking, and refactoring, begin.
