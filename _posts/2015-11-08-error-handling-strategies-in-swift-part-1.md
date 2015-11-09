---
layout: post
title: "Error handling strategies in Swift"
date: '2015-11-08T01:13:00+03:00'
tags:
- swift
- ios
- throw
- NSError
- catch
- error handling
- swift lang
---


When the official error handling model was updated in Swift 2.0 the designers behind it decided to go on a more traditional object oriented path than most of us expected. Before 2.0 the established *unofficial* way of handling errors followed the Haskell [monadic](https://en.wikipedia.org/wiki/Monad_(functional_programming) approach, which got a lot of love from the community.

While some embraced the new model, others were utterly disappointed. For the scope of this article, we shall not weight the advantages and disadvantages, but instead we will focus on simple but important strategies in error handling using the new model. As a side note, if you're interested in comparing the the two, [Brad Larson](http://www.sunsetlakesoftware.com/2015/06/12/swift-2-error-handling-practice) and [Nick Lockwood](https://gist.github.com/nicklockwood/21495c2015fd2dda56cf) wrote comprehensive articles on the matter. In any case, my feeling is that `try-catch` fits Swift better than the monadic model.

That being said. Let's get started with a little bit of background. Conceptually, an error is an unexpected result to an operation. Such a result can happen at different levels, so we have hardware, kernel, system library, application level errors and so on. Most of these **can be handled** from the current running process (yes, even hardware and kernel errors) however many leave the process is such a dire state that it can't be recovered, leaving us with the only valid option of terminating it.

Historically, Objective-C had several ways of handling application level errors, by either keeping them in the application space (usually for failures: `NSError` pointer to pointer, error code return) or forwarding them to lower level mechanisms (`NSException`, `@throw`). Using the latter for general flow-control was strongly discouraged by Apple because of performance issues. What Swift brings new is that it simply takes `throw` from the second category and puts it in the first. It no longer implicitly forwards the handling to lower levels, but forces the developer to handle the error as soon as the error throwing function had return. As you will soon see, this means `throw-try` is very similar with error returns, but with its handling enforced by the compiler.

In Swift, every single function that can throw an error is marked accordingly. This is a great thing, partly because it documents the function without additional effort and partly because a function that throws forces the developer to handle the possible return error one way or the other.

Also, an important thing to mention here is that adding `throws` or `rethrows` (which we will be discussed later on) to a function changes the function signature. Meaning that this:

```swift
func sendRequest(request:String) throws -> String
{
    return "Response"
}


func queueOperation(operation:String->String)
{
    //add the operation to the queue
}

queueOperation(sendRequest)
```

Will fail with `error: invalid conversion from throwing function of type '(String) throws -> String' to non-throwing function type 'String -> String'` and in order to fix it we will have to tell the compiler that `queueOperation` actually accepts throwing operations too.

However, it makes sense the other way around: any non-throwing function can replace a throwing one.

```swift
func queueOperation(operation:String throws->String)
```

Now let's spice things up a little and add the function that executes all the queued operations.

```swift
//don't be intimidated by the trailing `()`
//it simply allows partially apply the first parameter
//and be able to later call the function with the
//parameter already applied
func sendRequest(request:String)() throws -> String
{
    return request + "Response"
}

func sendRequestSafe(request:String)() -> String
{
    return request + "Safe Response"
}

var array = Array<() throws->String>()
func queueOperation(operation:() throws->String)
{
    array.append(operation)
}

func flushOperations() throws
{
    for op in array
    {
        print(try op())
    }
}

queueOperation(sendRequest("First"))
queueOperation(sendRequestSafe("Second"))

try? flushOperations()
```

All works nice and dandy. But what if one function throws an error.

```swift
func sendRequest(request:String)() throws -> String
{
    throw NSError(domain: "com.cocoaexposed", code: 1, userInfo: nil)
    return request + "Response"
}
```

`flushOperations()` will throw as well and the operations will get executed only up to the throwing one. In our case, the throwing one is the first one, so no operations will execute.

Let's modify `flushOperations()` so that it accepts a callback function with an optional parameter, the result (which will be nil if the operation fails).


```swift
func flushOperations(callback:(String? throws -> ())) rethrows
{
    for op in array
    {
        try callback(try? op())
    }
}

flushOperations
{
    result in
    print(result)
}
```

And here is `rethrows` in action. As you can see, we didn't had to mark `flushOperations` with `try` although is does rethrow. That's because when using `rethrows` a function will throw only if at least one of the parameter throws as well. Since only closures and functions can throw, `rethrow` can only be used on function that take at least one closure as a parameter.

So, if we were to write things different:

```swift
try? flushOperations
{
    result in
    throw NSError(domain: "com.cocoaexposed", code: 1, userInfo: nil)
}
```
We would have had to handle the error thrown by `flushOperations` as well.

So, we can conclude, there a 4 ways of handling a thrown error:

1. Do not handle but bubble up the error:

```swift
func fetchPersons() throws -> [AnyObject]
{
    try self.executeFetchRequest(NSFetchRequest(entityName: "Person"))
}
```

2. Handle and control the flow:

```swift
do
{
    let result = try self.executeFetchRequest(fetchRequest)
    //other statements that may or may not throw errors
}
catch let e as NSError
{
    //handle the error accordingly
}
```

3. Use result optionals:

```swift
let result = try? self.executeFetchRequest(fetchRequest)
//result is now an optional
```

4. Force unwrap. Forward the error to lower error handling levels (this will eventually send an `EXC_BAD_INSTRUCTION` mach message, by default terminating the process):

```swift
let result = try! self.executeFetchRequest(fetchRequest)
//if `executeFetchRequest` fails, you'll get a crash
```

So this is how we handle it. But how do we throw it? That's even easier:

```swift
throw NSError(domain: "com.cocoaexposed", code: 1, userInfo: nil)
```

And the beautiful part: we don't have to throw a NSError, any struct, class or enum that implements `ErrorType` can be used with `throw`. Swift lets us decide the error object  implementation details.

Finally, there's `defer`. Useful for a bunch of other things too, but probably created with this purpose in mind:

```swift
try? flushOperations
{
    result in
    let file = open(filename)
    defer {
       close(file)
    }

    //do things with the file
    throw NSError(domain: "com.cocoaexposed", code: 1, userInfo: nil)
}
```

`close(file)` will be called after the closure exits, no matter if you throw an error or not.

And so we learned how the basic error handling mechanism works in Swift. In our next part we will present a generic strategy to help us handle errors application wide.
