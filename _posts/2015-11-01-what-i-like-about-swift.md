---
layout: post
title: "Thoughts on Swift: 5 things I ❤"
date: '2015-11-01T01:13:00+03:00'
tags:
- swift
- ios
- learning
---

I'm still in the process of learning Swift and I could say that so far it was a wonderful journey. Still, nothing's perfect, so here are my totally biased opinions on the matter.


1. **Type Inference.** Oh man, how I love to simply write `let x = self.something(param1, param2)`. It's so elegant and slick I could watch it all day. But wait, there's more:

```swift
car.start {
  startTime in
  //startTime is of right type and ready to be used
}
```

2. **Verbosity.** I don't know if that's the right word, but I love how Swift let's you decide how verbose you'd like to be, without trying to force your hand.

```swift
view.subviews.forEach({
  (subview:UIView) -> () in
  subview.removeFromSuperview()
})
//or
view.subviews.forEach {
  subview in
  subview.removeFromSuperview()
}
//or
view.subviews.forEach{$0.removeFromSuperview()}
```

3. **Closures** *Finally*, closures are first-class citizens on iOS. I eagerly waited this moment. But how about Objective-C blocks? I'm just going to say the little things in life make the difference (starting with syntax...). On top of it, closures and named functions are now interchangeable. You can't ask more than that!

```swift
func processData(data:NSString, response:NSString)
{
    print("Function: \(data), \(response)")
}

func finishedLoading(callback:(data:NSString, response:NSString) -> ())
{
    callback(data: "Some Random Data", response: "Some Random Response")
}

finishedLoading
{
    data, response in
    print("Closure: \(data), \(response)")
}

finishedLoading(processData)
```

4. **Partials or curried functions** This one's really awesome. Together with closures it changed so many things for me. Starting with the fact that it rendered `NSOperations` and `NSValueTransformer` useless. And I was so much into the two, I felt sad to let them go.

```swift
func processData(request:NSString)(data:NSString, response:NSString)
{
    print("Function: \(data), \(response)")
}

func finishedLoading(callback:(data:NSString, response:NSString) -> ())
{
    callback(data: "Some Random Data", response: "Some Random Response")
}

let processData = self.processData("The request")
finishedLoading(processData)
```

5. **Function and operator overloading.** Again, a feature I missed dearly in Objective-C. I know some of you guys have a C++ background where operator overloading is seen as the root of all evil, but really, can you argue with the beautifulness of [this](http://www.alloc-init.com/2015/10/if-let-assignment/). One of the best example I've ever seen, but then again you can have function composition and many other awesome stuff with it. *Note to self: don't abuse it*.

```swift
//where
if let value = someOptionalValue as? String {
  self.value = value
}

//rather beautifully becomes
self.value ?= value

//with the help of
infix operator ?= { associativity right precedence 90 }

func ?=<T>(inout left: T, right: T?) {
    if let value = right {
        left = value
    }
}

func ?=<T>(inout left: T?, right: T?) {
    if let value = right {
        left = value
    }
}
```
