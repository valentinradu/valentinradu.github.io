---
layout: post
title: How to set breakpoints for specific functions/methods using LLDB command line
date: '2013-01-14T01:36:00+02:00'
tags:
- lldbbreakpointsdebugconsole
tumblr_url: http://somerandomtechblog.tumblr.com/post/40480356516/set-breakpoints-for-specific-methods-or-functions
---
The motivation

Let’s say that during debugging you want to pause execution every time a view will appear in an iOS app (viewWillAppear) and you have so many controllers that adding the breakpoints manually would be cumbersome.



The solution

You can do just that using lldb commands. Pause your execution at the start of your program by adding a logical breakpoint.



Then in the lldb command line write the following command breakpoint set -n NSLog. Which basically says, set a breakpoint in the C function named NSLog. There are other flavours and aliases of it on LLDB tutorial page under the Setting breakpoints section.



Now resume your thread execution. The code should break every time NSLog gets called. However, you will initially get the assembly version of it, so you’ll have to manually move one frame up, like in the second image below.





Similar, you could pause execution every time viewWillAppear gets messaged using:

breakpoint set -S viewWillAppear:


Also, to save you the trouble of writing this every time you run your app, you could add it as a breakpoint command and enable/disable the parent breakpoint at will.



Note how “Automatically continue after evaluating” is selected so that the parent breakpoint will only execute the command and not pause the program.
