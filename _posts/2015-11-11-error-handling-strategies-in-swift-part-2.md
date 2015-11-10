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

This article has two parts, if you don't know yet how the error handling model works in Swift 2, you could check out the first part. Also, if you're not familiar with the composition operator in other functional languages (e.g. `>>` in F#), here's a way to add it as a Swift `infix` operator.

Last time we talked about how to throw, try and catch errors. This time we will focus on how to plan and implement a generic system that will help us handle errors application wide.

There are several things to consider when designing:

1.  **Consistency.** All the errors in our app should pass through the same processor. We want to avoid the scenario in which for each `catch` we write specific code. `defer` is a much suited place for writing context specific code, like cleaning up. In the following example we use an error agent to sort and forward the error information to interested parties.


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

The above agent has only one error category, `handledErrors` but we could add as many as we wish, sorting them accordingly in the `switch` below.

2. **Handle errors at the topmost level** Let errors bubble up. Avoid handling them at micro level as much as possible. Functions or closures that throw should be called from other throwing functions up to the topmost level.

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

3. **Prefer a functional approach, use composition, whenever possible** If you follow the previous advice, it won't take long before you will fail to bubble up your error because some framework function that doesn't throw sits between you and the topmost level. In order to solve this, compose your functions into a non-throwing more powerful function that does the error handling along the way.

```swift
let urlRequest = NSURLRequest.URLRequestForPosts()
let semaphore = CWSemaphore(value: 0)

let save = NSBlockOperation(block:postListController.model.storeContext.performSaveAndWait)

let consume = postListController.responseAgent.insertUpdate(Post.name) |>>
              postListController.errorAgent.act(save.cancel) |>>
              semaphore.signal

let request = NSBlockOperation(block:postListController.session.dataTaskWithRequest( urlRequest, completionHandler:consume).resume |>> semaphore.wait)


save.addDependency(request)
postListController.queue.addOperation(request)
postListController.queue.addOperation(save)
```

4. **Always handle errors accordingly** There are multiple errors types, there are errors that we'd like to show to the users, errors that would be useful in a crash logs, and maybe errors that we'd like to ignore in production but assert in development. No matter the kind, always handle your errors and avoid using `try!` and `try?` except when absolutely necessarily.
