---
layout: post
title: sizeof(char) is always 1 but should be considered good practice
date: '2013-01-17T18:37:00+02:00'
tags:
- c
- code style
- sizeof
- char size
- best practices
---
I stumbled upon [this](https://drj11.wordpress.com/2007/04/08/sizeofchar-is-1/) post today, and while the author did some very good remarks:

```c
char i;
size_t iSize = sizeof i;
```

it’s easier to maintain and read than:

```c
char i;
size_t iSize = sizeof(char);
```

I’m ~~outraged~~ sadden (joke aside, the guy it's a versed programmer, you should check out his blog, many interesting things) by his belief that since sizeof(char) (or sizeof i in our first example) is always 1 as per C standard we should prefer this:

```c
aChar = malloc(kCharsCount);
```

over this:

```c
aChar = malloc(kCharsCount*sizeof(*aChar));
//aChar = malloc(kCharsCount*sizeof(char)); as he puts it
```

Because, “It adds clutter and it suggests that you don’t know enough C”. That’s utterly rubbish.

First of all, it’s not about portability, yes, a char is always 1 byte as per C standard, no matter if the byte has 8 bits or not(although it would be a serious challenge to find a modern system that has a different bit size for a char, I personally know none). However, there are other things to consider, namely, readability and maintainability.

Obviously, `aChar = malloc(kCharsCount*sizeof(*aChar));` is easier to maintain since if you change the type of aChar it will still work as expected. Also, in a function

```c
someFunction(aChar, sizeof(*aChar), kMultiplier);
```

is easier to read and more maintainable than

```c
someFunction(aChar, 1, kMultiplier);//what''s with the 1?
```

Finally, even sizeof(char) is better than 1. In

```c
#define buff_size (8 * sizeof(char))
```

is clear that buff_size is the size needed to hold up to 8 chars, whereas

```c
#define buff_size (8)
```

tells you absolutely nothing.
