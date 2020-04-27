+++
title = "The Art of Computer Programming"
date = 2018-08-25T17:56:57-05:00
description = "Making my way through TAOCP"
draft = false
toc = false
categories = ["computer science"]
tags = ["taocp", "knuth", "vol1"]
images = [] # overrides the site-wide open graph image
+++

An attempt at making my way through one of the most infamous computer science books

I was recently gifted the first volume of Donald Knuth's: "The Art of Computer Programming". Being the computer geek that I am, I have never been so excited yet, intimidated about sitting down and working through what some people would call a Reference book.

<!--more-->

## Reading the book

The biggest misconception that I had about this book was that I thought it would be more like a reference book, something you can quickly open, flip to the table of contents, and then over to page 254 for linked lists (or linked allocation in this case) and learn everything there is to know about linked lists. As stated by peers, Stack Overflow posts, Reddit, and Hacker News users, it's not something you would want to actually go about reading through. It's long, highly technical, and full of math equations. I suspect that the average reader would quickly get lost the first time Knuth shows off an algorithm in a custom assembly language written specifically for this book (yes, really). To put this more plainly: *Reading through TAOCP is an NP hard problem*. Naturally, I couldn't wait to read through it. So how should we go about reading the book? Luckily for us Knuth provides the reader with instructions, and of course, being the computer scientist that he is, a helpful algorithm flow chart to guide the reader through the process of reading the books which looks something like this:

```
           +
           |
           |
     +-----v-----+
     |1. Start in|
     +-----+-----+
           |
           |
+----------v-----------+
|                      |
| 2. Read pp xv - xvii |
|                      |
+----------+-----------+
           |
           |
     +-----v------+
     |            |
     | 3. N <- 1  |
     |            |
     +-----+------+
           |
           |
  +--------v---------+
  |4. Begin Chapter N|
  +--------+---------+
           |
           |
   +-------v-------+ Yes       +---------------------------------------+
   |5. Interesting?+---------->| 6. Read until Exhaustion or Whatever  |
   +---------------+           | the CS equivalent of Nirvana might be |
           |No                 +---------------------------------------+
           |                                        ^
           |                                        |
           +----------------------------------------+
```

The flow chart and written procedure following this contains other suggestions on how to read through the book based on if you've read through it before or just based on what your interests are.

### Exercises

A large portion of this book is dedicated to exercises as well as the included solutions in the back. *Ugh, homework*. One of the more helpful things that Knuth does is break down the relative difficulty of each problem using this notation:

```
00 Immediate                        > Recommended
10 Simple (one minute)                M Mathematically oriented
20 Medium (15 minutes)                HM Requiring "higher math"
30 Moderately Hard
40 Term Project
50 Research problem
```

In his reasoning for breaking down exercises this way, Knuth includes an anecdote about how Richard Bellman's book *Dynamic Programming* mixed trivial problems with research problems with no indication about which problems were solvable or not. *I bet his students loved that*. Though I wanted to get a lot out of this book the first time round I don't expect to go through all of these. Probably the easier ones to get the idea of it, but also doing a little bit of searching about the more Research type problems is certainly interesting.

## Induction, Combinations, Harmonic numbers, and Other Basic Concepts

This is where I imagine most people would stop. After a lengthy dissection of what algorithms are as well as the history behind the word Algorithm, Section 1.2 includes a lot of the foundational math behind computer sciences, induction, permutations and combinations, harmonic numbers, and elementary number theory; All the things you tried to wrap your head around in Algorithms I through III that you thought you could forget about have come back with a vengeance. I'll admit that when I started reading the introduction to this chapter and flipped through the next 10 pages or so I considered picking up some supplementary materials on the math involved. After some searching I came across some of his other works and I almost picked up his book on [concrete mathematics](https://www.amazon.com/dp/0201558025/). It was at this point that my adventures in computer science had started to look a lot like [this](https://www.youtube.com/watch?v=AbSehcT19u0). I resisted the temptation a bit and pressed on which I'm glad I did because two paragraphs down Knuth suggests you lightly skim this the first time you read through the book otherwise you might never get around to reading the actual computer science topics. *Hmmm... yeah that checks out*

## MIX

Not only did Knuth have to [create a typesetting language after the second edition of the second volume was created](https://en.wikipedia.org/wiki/TeX#History) but he also set about creating an instruction set and assembly language for a made up machine called MIX, reasoning that programming languages tend to influence the way that an engineer might code so having a more pure language to work with might solve that issue. *[Hmm... That sounds familiar](https://www.youtube.com/watch?v=AbSehcT19u0)*. You can find the entire set of instructions and diagram of the machine architecture [here](https://en.wikipedia.org/wiki/MIX) although if you can find a copy of the book I recommend looking at the diagram given in the book itself because it's beautiful. What I found particularly interesting is that this architecture is actually supported by [gcc](https://gcc.gnu.org/onlinedocs/gcc/MMIX-Options.html) meaning we can compile code for the theoretical computer. Knuth describes this computer, the MIX 1009 that runs MIX assembly code as:

*MIX is very much like nearly every computer of the 1960's and 70's, except that it is, perhaps, nicer.*

There you have it. Throw out that new Macbook pro you bought for $3000 and *rm -rf* that Vue JS project you've been working on because it's time we threw something new into the MIX! Knuth does admit that at this point MIX is a bit antequated so naturally he notes that a new version, dubbed [MMIX](http://mmix.cs.hm.edu/index.html), based on RISC, has been created and will likely be included in subsequent editions of the book... just as soon as they get around to rewriting all of the algorithms in MMMIX assembly.

## Concluding Thoughts

Though the book provides a very comprehensive and deep dive into every topic, the book is still missing some information. Completing this book proved to be an impossible task for Knuth and others; before Knuth can document everything, previous parts of it have already started to go out of date and the various *"Under construction"* sections throughout the book highlight just how quickly, and how much information there really is about this field of study. This doesn't come as particularly surprising to anyone who's spent time trying to include the newest npm module into their web project, only to find out it's unsupported three months later. I spent the better part of 4 years studying computer science in school yet barely scratched the surface of what the field has to offer and years later I'm still finding the need to learn more about it. At times it can feel overwhelming just trying to keep up with learning the basics of the newest language, the hottest Javascript framework or the the newest trends. Software engineering, web development, and computer sciences are moving at blazingly fast speeds with nary an end in sight. Even at it's most basic levels described here in the first volume, it's still being worked on let alone the work on the remaining volumes following volume 4, which have yet to be released.

So why read this then? Though the book is quite old I still think that there is a lot of value in reading through it. Much of it is still relevant today as data structures and basic algorithms (the underpinning of everything we still use daily) have not changed much, and more than that; Knuth is actually a fantastic writer. Though the material may be heavy, Knuth has a sort of dry humor that I've come to appreciate that has made reading this volume enjoyable. Though I don't think that [reading TAOCP alone will get you a job](https://www.businessinsider.com/bill-gates-loves-donald-knuth-the-art-of-computer-programming-2016-4), I recommend reading TAOCP for anyone who wishes to pique their intellectual curiosities about the field of computer science.