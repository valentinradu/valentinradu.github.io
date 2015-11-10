---
layout: post
title: "Error handling strategies in Swift - Part 2"
date: '2015-11-11T01:13:00+03:00'
tags:
- swift
- ios
- throw
- NSError
- catch
- error handling
- swift lang
---

Initially I wanted to wrap this up in a single post but as I was writing it I decided to split it two parts. This is the second part, if you're not familiar yet with error handling basics in Swift 2.0, feel free to start with [the first part](http://cocoaexposed.com/2015/error-handling-strategies-in-swift-part-1).

Also, if you're not familiar with the composition operator from other functional languages (e.g. `>>` in `F#`), you can checkout this post on how to add it as a Swift infix operator.

Last time we talked about how to `throw`, `try` and `catch` errors. This time we will focus on how to plan and implement a generic system that will help us handle errors application wide.

There are several things to consider when designing such a system:

1.  **Consistency.** All the errors in our app should pass through the same processor. We want to avoid the scenario in which for each `catch` we write specific code. `defer` is much better suited for writing context specific code  like cleaning up. In the following example we use an error agent to sort and forward the information to interested parties.


```swift
do
{
    let r = self.moc.executeFetchRequest(request)
    //...
}
catch let e
{
    ErrorAgent().act(e)
}

struct ErrorAgent
{
    static var handledErrors:[Int] = []
    func act(error:ErrorType)
    {
        switch (error)
        {
        case let e as NSError where self.dynamicType.handledErrors.contains(e.code):
            NSNotificationCenter.defaultCenter().postNotificationName(
            ApplicationLevelErrorNotification,
            object: nil,
            userInfo: [ApplicationLevelErrorKey:e])
        default:
            assert(false, "Unhandled error: \(error)")
        }
    }

    func act(callback: ()->())(error:ErrorType)
    {
        self.act(error)
        callback()
    }
}
```

The above agent has only one error category, `handledErrors`, but we could add as many as we wish, sorting them accordingly in the `switch` below.

2. **Handle errors at the topmost level.** Let errors bubble up. Avoid handling them at various different levels as much as possible. Functions or closures that throw should be called from other throwing functions up to the topmost level.

```swift
//avoid this

func start()
{
    request()
}

func request()
{
    do
    {
        postHeader()
        try postBody()
    }
    catch let e
    {
        print(e)
    }
}

func postHeader()
{
    do
    {
        try write()
    }
    catch let e
    {
        print(e)
    }
}

func postBody() throws
{
    try write()
}

func write() throws
{
    print("write")
}

start()


//prefer this instead

func start() throws
{
    try request()
}

func request() throws
{
    try postHeader()
    try postBody()
}

func postHeader() throws
{
    try write()
}

func postBody() throws
{
    try write()
}

func write() throws
{
    print("write")
}

do
{
    try start()
}
catch let e
{
    print(e)
}

```

3. **Prefer a functional approach, use composition, whenever possible.** It won't take long before you will fail to bubble up your errors because some framework function that doesn't throw sits between you and the topmost level. In order to solve this, compose your functions into a non-throwing function that does the handling along the way. If this doesn't make sense right away, don't worry, you will find it all explained better below.

```swift
//We get a specific url from our factory
let urlRequest = NSURLRequest.URLRequestForPosts()

//CWSemaphore is a very simple struct built upon dispatch_semaphore_t
let semaphore = CWSemaphore(value: 0)

//PerformSaveAndWait simply performs a save on the context's thread
//and waits until it's finished
let save = NSBlockOperation(block:postListController.model.storeContext.performSaveAndWait)

//This one is nice. We compose all the functions
//needed to consume this request. The first function
//partially applies the post's entity name so that
//the only parameters that remain are
//the response, data and error, which fits perfectly
//the dataTaskWithRequest completionHandler.
//I love how we can chain the semaphore's signal
//function and transform what should have been an async
//operation into a sync operation allowing responseAgent.insertUpdate
//and errorAgent.act to perform before exiting the request
//NSOperation

let consume = postListController.responseAgent.insertUpdate(Post.name) |>>
              postListController.errorAgent.act(save.cancel) |>>
              semaphore.signal

let request = NSBlockOperation(block:postListController.session.dataTaskWithRequest( urlRequest, completionHandler:consume).resume |>> semaphore.wait)


save.addDependency(request)
postListController.queue.addOperation(request)
postListController.queue.addOperation(save)
```

4. **Always handle errors.** There are multiple errors types, there are errors that we'd like to show to the users, errors that would be useful in a crash logs, and maybe errors that we'd like to ignore in production but assert in development. No matter the kind, always handle your errors and avoid using `try!` and `try?` except when absolutely necessarily.

Using the above we were able to design a system that delegates all the error handling responsibilities to an agent but keeps the flow control within the same context. Even more, with a little bit of help from the `>>` infix custom operator, we chained together several functions into a more powerful one, which was called asynchronously without losing the ability to catch any possible error that might occurred.

This is why I love Swift.
