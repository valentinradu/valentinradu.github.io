---
layout: post
title: "What does @NSManaged do?"
date: '2015-11-07T01:13:00+03:00'
tags:
- swift
- ios
- dynamic dispatch
- NSManaged
- runtime
---


There's not much about `@NSManaged` in the Apple docs and [the little it is](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Attributes.html) irrevocably ties it to `NSManagedObject`. I imagine there must have been a misunderstanding since the evidence suggests `@NSManaged` presents exactly the same behavior as `@dynamic` in Objective-C (which is different from `Swift`'s `dynamic`) and it's certainly not tied to `NSManagedObject` or Core Data.

Let us start by creating a `Hero` class. The first thing I've noticed was that, while `dynamic` can be successfully applied to properties of any class, `@NSManaged` makes no sense outside the `NSObject` protocol.

The compiler may not complain, but there's no way for you to provide an implementation and storage at runtime for a class that doesn't implement the `NSObject` protocol.

Inevitably your app will crash.

```swift
class Hero:NSObject
{
    @NSManaged var agility:Int
    dynamic var strength:Int
    var endurance:Int
    var inventory:[String]

    override init()
    {
        self.endurance = 0
        self.strength = 0
        self.inventory = ["sword"]
        super.init()
        self.agility = 0
    }
}
```

The compiler enforces the initialization of `endurance`, `strength` and `inventory` **before** `super.init()` but `agility` after it. It sort of makes sense since the `super` class could provide a valid implementation and storage for `agility` in its `initialization` method (not called until you send the first message to the class, which could be `init`), as `NSManagedObject` would do for its subclasses.

We override the `initialize` method and see what are those properties made of. It would have also been a good place to provide a runtime implementation for any `@NSManaged` properties.


```swift
override class func initialize()
{
    var propertiesCount:UInt32 = 0
    var ivarsCount:UInt32 = 0
    let properties = class_copyPropertyList(self.self, &propertiesCount)
    let ivars = class_copyIvarList(superclass(), &ivarsCount)

    for i in 0..<propertiesCount
    {
        let property = properties[Int(i)]
        let attributes = String.fromCString(property_getAttributes(property))?.characters.split(",").map{String($0)}
        let name = String.fromCString(property_getName(property))

        print("property \(name), \(attributes)")
    }

    for i in 0..<ivarsCount
    {
        let ivar = ivars[Int(i)]
        let name = String.fromCString(ivar_getName(ivar))

        print("ivar \(name)")
    }

    ivars.destroy()
    properties.destroy()
}
```

This outputs:

```
property Optional("agility"), Optional(["Tq", "N", "D"])
property Optional("strength"), Optional(["Tq", "N", "Vstrength"])
property Optional("endurance"), Optional(["Tq", "N", "Vendurance"])
property Optional("inventory"), Optional(["T@\"NSArray\"", "N", "C", "Vinventory"])
ivar Optional("isa")
ivar Optional("strength")
ivar Optional("endurance")
ivar Optional("inventory")
```
As you can see, `agility` is a `long long` (`Tq`), `nonatomic` (`N`) and `dynamic` property. Exactly the same signature a similar `@dynamic` property would have in Objective-C.

Another interesting fact is that the Swift `dynamic` is not really needed here, all the properties are dynamically dispatched. However, never rely on that, the compiler could optimize the properties, there are only two ways to guarantee dynamic dispatching of a property: `dynamic`(with storage and implementation) and `@NSManaged`(without storage and implementation).
