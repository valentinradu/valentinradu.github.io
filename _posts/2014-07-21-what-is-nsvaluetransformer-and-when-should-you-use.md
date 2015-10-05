---
layout: post
title: What is NSValueTransformer and when should you use it?
date: '2014-07-21T11:17:00+03:00'
tags:
- ios
- objective-c
- nsvaluetransformer
- example
---
`NSValueTransformer` does exactly what its name suggests, takes a value of any kind and transforms it into another one, think of it as a function encapsulated within an object.

Ok, but how does that help you? One of the most appropriate use case for `NSValueTransformer` is transforming data from an ill-defined form into a canonical form.

Think of the following scenario. You have to connect to a series of web services which return basically the same information, let’s say weather information, however, in different formats, some are maybe using JSON, others XML. Moreover, even for the ones that return the data in the same format, the structure, keys or values (e.g. temperature could be expressed in either °C or °F) differs.

A good design to solve this problem should, at least, allow you to add new services easily and not mix your business logic with the service data parsing and extracting.

Here is where `NSValueTransformer` comes really handy. You could have different `NSValueTransformer` subclasses for each service, transforming the received http `NSData` into a canonical `NSDictionary` or a custom class that could be used throughout your app. This gives you the possibility to add new services just by creating a new `NSValueTransformer` subclass that knows how to make this conversion happen. Below there’s an example on how one of this `NSValueTransformer` subclass implementation might look like.

```objc
+ (Class)transformedValueClass {
    return NSDictionary.class;
}

+ (BOOL)allowsReverseTransformation {
    return NO;
}

- (id)transformedValue:(NSData*)data {

    //the respones might not be what we''re expecting,
    //so we''re interested in the error, if any
    NSError* error;

    //we assume this service response is JSON
    NSMutableDictionary* response =
[NSJSONSerialization JSONObjectWithData:data
options:NSJSONReadingMutableContainers|NSJSONReadingMutableLeaves
error:&error];

    //we let others know if the transform ended with an error
    //and return right away if it did
    self.error = error;
    if (self.error) return nil;


    //our canonical response
    NSMutableDictionary* canonicalResponse = [NSMutableDictionary dictionary];

    //we copy the values we''re interested in into the canonical dictionary
    //using the appropiate keys and transforming values where needed
    canonicalResponse[@"temperature"] = [self fahrenheitToCelsius:response[@"temp"]];
    canonicalResponse[@"location"] = [response[@"area"] capitalizedString];

    //and so on
    //...


    //the returned dictionary has a consistent
    //structure for each and every service now
    return canonicalResponse;
}
```

There you go, this little gem just saved you from a lot of tangled ifs and cases.
