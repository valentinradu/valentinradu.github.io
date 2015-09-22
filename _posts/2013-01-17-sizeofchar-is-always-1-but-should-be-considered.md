---
layout: post
title: sizeof(char) is always 1 but should be considered good practice
date: '2013-01-17T18:37:00+02:00'
tags:
- c
- code style
- sizeof
- char
tumblr_url: http://somerandomtechblog.tumblr.com/post/40774481339/sizeofchar-is-always-1-but-should-be-considered
---
I stumbled upon this post today, and while the author did some very good remarks:

char i;
size_t iSize = sizeof i;


it’s easier to maintain and read than:

char i;
size_t iSize = sizeof(char);


I’m outraged sadden by his belief that since sizeof(char) (or sizeof i in our first example) is always 1 as per C standard we should always write code like this:

aChar = malloc(kCharsCount);


instead of:

aChar = malloc(kCharsCount*sizeof(*aChar));
//aChar = malloc(kCharsCount*sizeof(char)); as he puts it


Because, “It adds clutter and it suggests that you don’t know enough C”. That’s utterly rubbish.



First of all, it’s not about portability, yes, a char is always 1 byte as per C standard, no matter if the byte has 8 bits or not. However, there are other things to consider, namely, readability and maintainability.

Obviously, aChar = malloc(kCharsCount*sizeof(*aChar)); is easier to maintain since if you change the type of aChar it will still work as expected. Also, in a function

someFunction(aChar, sizeof(*aChar), kMultiplier);


is easier to read and more maintainable than

someFunction(aChar, 1, kMultiplier);//what''s with the 1?


Finally, even sizeof(char) is better than 1. In

#define buff_size (8 * sizeof(char))


is clear that buff_size is the size needed to hold up to 8 chars, whereas

#define buff_size (8)


tells you absolutely nothing.
