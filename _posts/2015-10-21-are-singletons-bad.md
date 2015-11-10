---
layout: post
title: Are singletons bad?
date: '2015-10-21T01:13:00+03:00'
tags:
- singleton
- design patterns
- ios
- objective-c
- swift
---

In their purest form, if they **enforce** the instantiation of a class to one object **and** are used accordingly throughout your app, **yes**, they are highly ineffective, inimical and contra-productive on long term.

However, I have yet to see a pure singleton in the entire Cocoa framework and we should probably learn from that. Think about it, absolutely no class enforces one single instance, some default to an instance, true, but still allow you instantiate other instances as you see fit.

The first example I can think of is `NSNotiicationCenter`. Although there's no point in having another notification center than the default one, or as Apple puts it, *you typically don’t create your own*, you still can, if you really want to.

```swift
let center = NSNotificationCenter()
center.addObserver(self, selector: "selector", name: "notification", object: nil)
center.postNotificationName("notification", object: nil)
```
The code above will successfully call the selector.

Then, are `NSNotificationCenter`, `NSURLSession` and others truly singletons? Probably not, but let's not linger on semantics. The truly important lesson to be learned here is:

**Always design your app as if every single class you create can and will have multiple instances**

But to do so, providing a valid `init` method is often not enough. You will also have to treat your sole instance as if it's one of the many. So, instead of this:

```swift
//Allover your app
Model.shared.doThis()
Model.shared.doThat()
```

Prefer this:

```swift
//In your dependency injector
//or in your init
controller.model = Model.shared()
//In your controller
self.model.toThis()
self.model.toThat()
```

Of course, there are exceptions, you don't need a `controller.notificationCenter` (I must confess I never used it like that either), although, it wouldn't hurt, since it could be implemented in a category and it's overall shorter.
