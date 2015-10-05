---
layout: post
title: Concurrent programming with Core Data
date: '2014-08-04T00:52:00+03:00'
tags:
- core data
- design patterns
- multithreading
- ios
- objective-c
---
Core Data is powerful and when harnessed correctly, it can transform your app into an ultra responsive, easy to maintain, scalable, background data consumer gem.

Unfortunately, it’s notoriously hard to master, especially when working with multiple threads, in which case, there are many possible issue that may arise, like UI blocking, deadlocks, data concurrency conflicts and many others. This article describes a technique that aims to solve these problems.

The pattern recommended by Apple for concurrent programming with Core Data is thread confinement: each thread must have its own entirely private managed object context.

Having just the main thread and its corresponding UI context is certainly not going to cut it: every time you make a time consuming persistent operation the UI will freeze. That’s certainly not what you want for your awesome app.

Using two threads, each with its context, may seem like the easy way out, but has one major flaw, complex fetching (or faulting) will always block the UI thread, no matter how we set the hierarchy between the two (siblings or parent-child).

Instead, consider this design:

Have a root context which runs on a low priority thread and saves its changes directly to the persistent store.
Keep the UI context as the root’s child, this way, when saved, it will push its changes to the root context, not the persistent store. This means that saving will be really fast (in memory). Also, complex fetches can be pre-fetched in the root or a slave context. All these leaves the UI context really light and fast.
Use multiple slave contexts as children of the root context. Every time one gets saved, it pushes the changes to the root context and then notifies and merge the changes into the UI context. This makes a save very cheap since all the changes are propagated in memory. The slave context usually does the heavy lifting, from huge and complex fetches to data importing, every time an expensive operations is done, this grunt will handle it.
Below you have a diagram that puts in perspective the relations between the UI, root and slave contexts.

![Core Data Design Diagram](/assets/images/coredata-technique.png)

[Here](https://github.com/valentinradu/CWPersistentMaster)’s a class that handles the initial setup.
Also, below there’s a background fetch example. Inserting or updating works in a similar fashion.

```objc
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0), ^{

    //spawn a slave
    //you will typically have a singleton from which
    //you can access the master (in this example SPM)
    NSManagedObjectContext* slave = [SPM.localMaster spawnSlave];
    NSManagedObjectContext* ui = SPM.localMaster.uiMOC;
    __weak typeof(slave) weakSlave = slave;
    __weak typeof(ui) weakUi = ui;

    __block NSArray* resultIDs;

    //perform the fetch on the slave''s thread
    [slave performBlockAndWait:^{
         NSFetchRequest *request=[[NSFetchRequest alloc] initWithEntityName:@"Entity"];
         request.predicate=[NSPredicate predicateWithFormat:@"Complex fetch predicate"];
         request.resultType = NSManagedObjectIDResultType;

         NSError *error;
         resultIDs = [weakSlave executeFetchRequest:request error:&error];
         //check the error
    }];

    [ui performBlock:^{
         //do something on the main thread with the results
         NSManagedObject* firstObject = [weakUi objectWithID:[resultIDs firstObject]];
    }];
});
```
