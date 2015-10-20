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


Short answer, they are not, quite the opposite. So, what's with the gloomy aura? Why every master developer you ever met told you otherwise? Honestly, I don't know either, everybody seems to be bashing them lately. Poor singletons, I will defend them!

No really, it's actually your fault. The way you're abusing them made people who worked on your code think twice before properly using them.

So, what's to be done? How can we restore our faith in singletons? I like to think about it this way: if I ever feel like I need one, I ask myself, do I intend to use it just because I'm lazy and badly need to access an instance from **everywhere** without refactoring my code, or is this object really the only one of his kind in my entire application. Usually the first applies.

In other words, use singletons whenever you feel like, as long as you don't use them for the sole purpose of accessing an instance from allover your app.

*"But, but, that was my definition of a singleton until now!"*

I know, I know ... shush now, shush ...

Bottom line:

This is bad:
```
//Allover your app
[[Model shared] doThis]
[[Model shared] doThat]
```

This is good:
```
//In your dependency injector
controller.model = [Model shared]
//In your controller
[self.model toThis]
[self.model toThat]
```
