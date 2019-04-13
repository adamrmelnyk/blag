---
title: "Seminumerical Algorithms"
date: 2019-04-03T16:27:33-05:00
description: "Working through Volume 2 of TAOCP"
draft: false
toc: false
categories: ["computer science"]
tags: ["taocp", "knuth", "vol2"]
images: []
---

An overview of the second volume of The art of Computer Programming.

<!--more-->

# Prelude to the Second Volume

It's been about eight months (As of April) since I finished reading the first volume. Though it's likely not a surprise to many, I'll admit that it was one of the more challenging reads and though I've read through my share of books on computer algorithms, they really can't compare to the TAOCP. Though the language itself is fairly the simple, the content was some of the heaviest I've encountered. [My summary of the experience reading through the first volume](/post/the-art-of-computer-programming/) was more or less my thoughts on the book, but didn't offer a lot in terms of the content that was covered or any interesting material that was gleamed from its pages; perhaps I'll remedy that when I have time for a second read through. Admittedly, there was quite a bit of content and I don't think I wanted to post a section by section summary of the volume, especially when you consider that much of the first volume was devoted to stacks, trees, linked lists, and other data structures, something most software engineers and programmers are familiar with. Luckily the second volume is far less familiar territory and may be more interesting to talk about. <!-- re-write this last part -->.

# Seminumerical Algorithms

* What does seminumerical mean (the borderline of numeric and symbolic)
  * Numeric vs symbolic calculations

* Making a complicated procedure will not always result in something random
  * use the example of the K algorithm.
  * you need statistical analysis to determine if something is sufficiently random.

## Linear Congruential Generator

The most common pseudorandom number generator (according to Knuth) is a [Linear congruential generator](https://en.wikipedia.org/wiki/Linear_congruential_generator), which has the following equation:

<b><i>X</i></b><sub><i>n</i> + 1</sub> = (<i>a</i><b>X</b><sub><i>n</i></sub> + <i>c</i>) mod <i>m</i>

The equation isn't difficult to explain; numbers for m,a, and c are chosen and set in the compiler and a seed is chosen for the first value of Xn. This being a recurrence relation, the result of each run then becomes the next seed or starting value for a subsequent run. Choosing the same seed number will always produce the same result as shown by this C code:

{{< highlight c>}}
#include <stdio.h>
#include <stdlib.h>

int main()
{
    srand(0);
    int value = rand();
    printf("value is %d\n", value);
}
{{< / highlight >}}

<!-- TODO: Explain that other languages use different algos -->

  * the most common pseudorandom number generator
  * A recurrence relation: Each preceding iteration is defined as a function of the preceding terms.
  * has a seed / start value. I think this is how C does it? (I'm less sure after looking at the racket implementation)
  * Racket uses a different algorithm: MRG32k3a
    * https://github.com/racket/racket/blob/5bb837661c12a9752c6a99f952c0e1b267645b33/racket/src/cs/rumble/random.ss
    * https://software.intel.com/en-us/mkl-vsnotes-mrg32k3a
    * https://stackoverflow.com/questions/11075833/lecuyers-mrg32k3a-random-number-generator-in-cuda-c
    * https://srfi.schemers.org/srfi-27/