---
layout: post
title: 4 reasons for which I still wouldn’t use Swift on a large scale project in 2017.
date: '2017-02-27T03:31:00+02:00'
tags:
- ios
- objective-c
- swift
- macos
---
Yeah, this must be one of those unpopular opinions, especially since Apple makes such an effort promoting its new language. Also, all this hype around it. Don’t get me wrong, I’d love to use Swift, and I totally understand the desire to ignore all its shortcomings, but let’s face it, it’s simply not mature enough yet and although it sounds great on paper, you might regret using it in some cases.

1. The interoperability between Swift and C++ is next to zero. Sure, you can wrap the C++ code and then call it from Swift (sounds like fun, right?), but I don’t want that. I just want it to work as it does with Objective-C++.
C interoperability isn’t much better either. Although advertised otherwise, you might be in for a surprise (recently I had to integrate libssh and it was a pain, I assume you can get better and better at it, but the code will still be verbose and hard to read).

2. The tools surrounding Swift still feel like beta. Xcode is terribly slow for large projects. Sure, you could write Swift in a manner that would partially overcome this, however, that would uglify it and would require you to learn how to avoid all those static analyser / compiler / indexer issues that are slowing it down. Yeah, exactly, more work for you mate.
Hell, Xcode still can’t even refactor (as in, rename…) Swift properties or functions.

3. This one’s probably very controversial. I could write an entire article on it alone. So, I have a Python background and before that, I did mostly C. I remember when I started with Python I was completely terrified by its loose type system and dynamic behaviour. I felt insecure. But as I learned it, I saw the good in it and realised that most of the bugs I had, were not avoidable by using a more enforcing type system. However, I was writing code way faster.
How is this relevant? Well, even if Swift it’s way less verbose than Objective-C, I still spend quite a lot of time satisfying the type system’s requirements. You might argue that in order to write correct code, you’d had to tie those loose ends anyways, regardless of the language. You might be right and I’m all for safer code, but, even if you catch an unexpected behaviour because you used guard instead of simply crashing, you still can’t perform the operation itself. The best you can do is rollback, which is a gain, can’t argue, but you probably shouldn’t have got there in the first place and catch such a behaviour early in unit tests. 
Point is, safer code requires more work and satisfying the type system clutters your code. In the end, it might not worth the trouble and sacrificing safetly for speed might not be such a bad idea.

4. Swift is continuously changing. If Apple plans to change it as they do with their hardware, we’re in for some fun times. Have you tried to migrate your project from 2.2 to 3.0? It’s not that bad. Not that good either.

Bottom line. Swift is great, I might have preferred Objective-C 3.0 over it, but that’s a different discussion. However, given the current status, I’ll keep using Objective-C/C++. I can’t help wondering how much time will I be able to do so? Apple is aggressively pushing Swift and it’s probably just a matter of time until using Objective-C will become more trouble than it’s worth it. I can only hope that, by then, Swift will be ready.
