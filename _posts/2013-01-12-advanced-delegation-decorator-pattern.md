---
layout: post
title: Advanced delegation. A decorator design pattern example.
date: '2013-01-12T18:51:00+02:00'
tags:
- uitableview
- delegate
- datasource
- decorator pattern
- nsproxy
- forwardInvocation
- forwardingTargetForSelector
- cocoa
- objective-c
- ios
- macos
tumblr_url: http://somerandomtechblog.tumblr.com/post/40353790996/advanced-delegation-decorator-pattern
---
The motivation

You probably wrote the following lines in your view controllers plenty of times:

self.tableView.delegate = self;
self.tableView.dataSource = self;


Usually, that’s fine.

However, if you have another table view in your app which has more or less the same look and feel, but a different controller managing it, a problem rises.

It would be nice to reuse all the delegation code you wrote in your initial controller. And only change the data and do other small customisations. You could tackle this problem in several ways.

But before we start, know that you can find the Github project for this article here.


The inheritance solution

So, one way would be to have a generic view controller, make it the superclass of both of your controllers and implement the delegation methods in it.
Than would solve the initial problem, however if for example the two table views have a different header and footer and you override tableView:viewForHeaderInSection: and tableView:viewForFooterInSection: a third table view that would like to have the footer from the first and the header from the second would have no way to get it without duplicating code. 1

The aggregation solution

Another way to do it would be to create and use some delegate objects instead of passing self as a delegate.

dataSourceObject = [[VRDataSource alloc] init];
delegateObject = [[VRDelegate alloc] init];
self.tableView.dataSource = dataSourceObject;
self.tableView.delegate = delegateObject;


The objects are of a different class, so multiple table views can use different instances of it. That sounds nice, the delegates can even have their own inheritance structure, which means no code will ever be duplicated, which is even greater.

However, our objects don’t have direct access to the controller’s properties and methods, which means there will be one handling method on the delegate side and one on the controller side:

{% highlight objc %}
- (void)tableView:(UITableView *)tableView
{

{% endhighlight %}
}


Also, this means we will need a generic controller to respond to those messages, or a protocol which has to be implemented by all the controllers wishing to delegate responsibilities to our objects.

The decorator solution

A nice way to solve all the problems above and also have the freedom to implement parts of your delegates in different classes is by using the decorator design pattern. 2

So, we want the delegates to be different objects, but still be able to message transparently our controller. They sound like proxies to me 3. So, we first init our delegate objects keeping a reference to our decorated object, which in our case is the controller.

- (id)initWithDecoratedObject:(id<DEDecoratedObjectProtocol>) decObj
{
{% endhighlight %}

{% highlight javascript %}
    {
        self.decoratedObject = decObj;
{% endhighlight %}

{% endhighlight %}
}


Then, every message we receive in our decorators (delegates) and we can’t respond to, we forward towards our decorated object (controller). Each method override is explained in the code below.

//the first thing the forwarding mechanism does
//is calling this method if there is no selector
//found
//we override it and try to send the same message to our
//decorated method
//if that fails too, we have nothing better to do than let
//the system rise the exception by returning self
- (id)forwardingTargetForSelector:(SEL)aSelector
{
{% highlight javascript %}
    return _decoratedObject;
{% endhighlight %}

{% endhighlight %}
}

//objective-c is smart enough to know the
//difference between inheritence and this ''fake
//inheritence'' we''re trying to do
//methods like respondsToSelector: and isKindOfClass: look only
//at the inheritance hierarchy, never at the forwarding chain
//so we force them to consider it by overriding them
- (BOOL)respondsToSelector:(SEL)aSelector
{
{% highlight javascript %}
    return YES;
{% endhighlight %}

{% highlight javascript %}
        return YES;
{% endhighlight %}

{% highlight javascript %}
{% endhighlight %}
}

- (BOOL)isKindOfClass:(Class)aClass
{
{% highlight javascript %}
    return YES;
}
{% endhighlight %}

{% highlight javascript %}
        return YES;
{% endhighlight %}

{% endhighlight %}

{% endhighlight %}
}


This way, we will be able to implement some of the methods in our designated delegates and some in our view controller. Which saves us the trouble of forwarding the messages ourselves manually, like in the aggregate solution.

Another nice thing about this technique is that you can add or remove a delegate method from your delegate at runtime, which makes it very easy to customise. Also, a nice chain of delegates objects can be formed to give you extra control on how our table view looks. To better visualise all this, here is the set up code:

//init the controller
//this is just a basic UIViewController subclass implementing
//DEDecoratedViewControllerProtocol protocol
//a nice thing about this method is that you can keep
//your model-controller relation intact
//so, even if the controler will not be the table view delegate,
//it will still hold the model data for us
DERootViewController * viewController = [[DERootViewController alloc] init];
//our model data is simply a dictionary for the sake of simplicity
viewController.tableItems = @[@{@"title": @"Africa"},
{% highlight javascript %}
                        @{@"title": @"Asia"},
{% endhighlight %}


//decorate the text label with a color
dec = [[DERedDecorator alloc] initWithDecoratedObject:viewController];
//is as easy as this to change it to another color
//DEAbstractDecorator * dec =
//[[DEGrayDecorator alloc] initWithDecoratedObject:viewController];



//then decorate the table view with a footer
//notice that we pass the prev decoration as parameter
//this is really nice because we can have a chain of decorations on
//the same decorated object
dec = [[DEFooterDecorator alloc] initWithDecoratedObject:dec];


//maybe we want a header too
dec = [[DEHeaderDecorator alloc] initWithDecoratedObject:dec];



//create the tableview
UITableView * tableView =
[[UITableView alloc] initWithFrame:self.window.frame style:UITableViewStylePlain];
//delegate to decorator
tableView.delegate = dec;
tableView.dataSource = dec;


For a better overview on how all this works here is a Github project with all the code within a working example.



I know, that’s unlikely to happen, probably not the best example, but bear with me, the implications of the decorator pattern lay beyond this type of delegation. ↩



Is not a pure implementation, but it’s pretty close. ↩



When it comes to implementation, I choose not to use NSProxy because Apple’s Documentation states that forwardInvocation: is much more expensive than forwardingTargetForSelector: which is not implemented for NSProxy. ↩
