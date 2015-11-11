---
layout: post
title: "Adding the composition operator in Swift"
date: '2015-11-12T01:13:00+03:00'
tags:
- swift
- ios
- swift lang
- function composition
- functional programming
- functional swift
---

There's no doubt that certain operations are very well suited for a functional approach. Conceptually, when you're sending a server request, the entire flow can be described like this:

```swift
actOnResponse(makeRequest(request))
```

Where both `actOnResponse` and `makeRequest` are a (possibly long) chain of other operations:

```swift
    insertIntoDB(convertToDataObjects(parseData(parseResponse(makeRequest(request)))))
```

The composition operator (`|>>`) helps us greatly improve readability:

```swift
let getUsers =  makeRequest |>>
                parseResponse |>>
                parseData |>>
                convertToDataObjects |>>
                insertIntoDB
getUsers(request)
```

Make the request then parse whatever it returns, then convert it to data objects, and so on. Of course, in real life things are almost never that simple. You might have to partially apply functions or use semaphores to wait on async functions. Below you can find a more appropriate example taken from a real app:

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

Since Swift doesn't support the composition operator out of the box, we will have to add it ourselves:

```swift
infix operator |>> { associativity left }
func |>> <W, T, U>(left: W -> T, right: T -> U) -> (W -> U)
{
    let union: W -> U = {
        (w:W) -> U in
        right(left(w))
    }
    return union
}

func |>> <W, T>(left:W throws -> T, right: ErrorType -> ()) -> W -> T?
{
    let union: W -> T? =
    {
        w in
        do
        {
            let r = try left(w)
            return r
        }
        catch let e
        {
            right(e)
            return nil
        }
    }
    return union
}

func |>> <W>(left:W throws -> (), right: ErrorType -> ()) -> W -> ()
{
    let union: W -> () =
    {
        w in
        do
        {
            try left(w)
        }
        catch let e
        {
            right(e)
        }
    }
    return union
}
```
