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

It's been about eight months (As of April) since I finished reading the first volume. Though it's likely not a surprise to many, I'll admit that it was one of the more challenging reads as well as being surprisingly entertaining and well written. [My summary of the experience reading through the first volume](/post/the-art-of-computer-programming/) was more or less my thoughts on the book, but didn't offer a lot in terms of the content that was covered or any interesting material that was gleamed from its pages; perhaps I'll remedy that when I have time for a second read through. Admittedly, there was quite a bit of content and I don't think I wanted to post a section by section summary of the volume, especially when you consider that much of the first volume was devoted to stacks, trees, linked lists, and other data structures, something most software engineers and programmers are familiar with. Luckily the second volume is far less familiar territory (at least to me) and may be more interesting to write about.

## Seminumerical Algorithms

* What does seminumerical mean (the borderline of numeric and symbolic)
  * Numeric vs symbolic calculations

### Not So Random Number Generators

Early in the volume Knuth shows how difficult it is to create a sufficiently random pseudorandom number generator. Simply adding a number of complicated steps will may not yield truly random numbers. Knuth includes the following algorithm <b><i>K</i></b> to illustrate this point. You can find an excerpt of the algorithm [here](http://www.informit.com/articles/article.aspx?p=2221790).

#### Algorithm K

For the purposes of demonstration I've implemented the algorithm below. The steps aren't particularly important, but as you can see it's a fairly involved algorithm that looks like it might produce something random.

{{<highlight racket "linenos=table,linenostart=1">}}
#lang racket

(define (algorithm-k x)
  (do-algorithm-k x (+ 1 (floor (/ x (expt 10 9))))))

(define ns (variable-reference->namespace (#%variable-reference)))

(define (string->procedure s)
  (define sym (string->symbol s))
  (eval sym ns))

(define (do-algorithm-k x iterations)
  (define step (string-append
                "k"
                (number->string
                 (+ (modulo (floor (/ x (expt 10 8))) 10) 3))))
  (if (eq? iterations 0)
      x
      ((string->procedure step) x iterations)))

;; Ensure >= 5 x 10^9
(define (k3 x iterations)
  (if (> x 5000000000)
      (k4 x iterations)
      (k4 (+ x 5000000000) iterations)))

;; Middle square
(define (k4 x iterations)
  (k5 (modulo (floor (/ (expt x 2) (expt 10 5))) (expt 10 10)) iterations))

;; Multiply
(define (k5 x iterations)
  (k6 (modulo (* x 1001001001) (expt 10 10)) iterations))

;; Psuedo-complement
(define (k6 x iterations)
  (if (< x 100000000)
      (k7 (+ x 9814055677) iterations)
      (k7 (- (expt 10 10) x) iterations)))

;; Interchange halves
;; ie 1234567890 -> 6789012345
(define (k7 x iterations)
  (k8 (+
       (* (expt 10 5) (modulo x (expt 10 5)))
       (floor (/ x (expt 10 5)))) iterations))

;; Multiply / same as k5
(define (k8 x iterations)
  (k9 (modulo (* x 1001001001) (expt 10 10)) iterations))

;; Decrease digits
(define (k9 x iterations)
  (k10 (sub-one x 0 0) iterations))

(define (sub-one x sum mult)
  (if (eq? x 0)
      sum
      (if (eq? (modulo x 10) 0)
          (sub-one (floor (/ x 10)) (+ (* (modulo x 10) (expt 10 mult)) sum) (+ 1 mult))
          (sub-one (floor (/ x 10)) (+ (* (- (modulo x 10) 1) (expt 10 mult)) sum) (+ 1 mult)))))

;; 99999 modify
(define (k10 x iterations)
  (if (< x (expt 10 5))
      (k11 (+ (expt x 2) 99999) iterations)
      (k11 (- x 99999) iterations)))

;; Normalize
(define (k11 x iterations)
  (k12 (go-until x) iterations))

(define (go-until x)
  (if (< x (expt 10 9)) (go-until (* x 10)) x))

;; Modified middle square
(define (k12 x iterations)
  (do-algorithm-k (modulo (floor (* x (/ (- x 1) (expt 10 5)))) (expt 10 10)) (- iterations 1)))
{{< / highlight >}}

The results however random seeming do present a problem:

{{< highlight shell >}}
> (algorithm-k 1)
3831577709
> (algorithm-k 12345)
5827337884
> (algorithm-k 6065038420)
6065038420
{{< / highlight >}}

The first two examples produce what one might expect, but the third produces itself. Knuth points out that the algorithm happens to converge on 6065038420 the lesson being:

<p style="text-align: center;"><b><i>"Random numbers should not be generated with a method chosen at random"</b></i></p>

Besides coincidentally converging on a seemingly random number, Algorithm K has a number of other downsides, it's difficult to implement and because it iterates a certain amount of times depending on the input, could take a relatively long time to produce a single random number. Somewhere inside that complexity there is a reason why the generator converges on 6065038420. In order to be usable we need to be able to easily prove that won't happen. The key take away from that is that we need this to be fast and simple.

### Linear Congruential Generator

The most common pseudorandom number generator (according to Knuth) is a [Linear congruential generator](https://en.wikipedia.org/wiki/Linear_congruential_generator), which has the following equation:

<p style="text-align: center;"><b><i>X</i></b><sub><i>n</i> + 1</sub> = (<i>a</i><b>X</b><sub><i>n</i></sub> + <i>c</i>) mod <i>m</i></p>

Unlike the previous algorithm, the equation isn't difficult to explain; numbers for m,a, and c are chosen and set in the compiler (hopefully not randomly) and a seed is chosen for the first value of Xn. This being a recurrence relation, the result of each run then becomes the next seed or starting value for a subsequent run. You can see this in the code below:

{{< highlight racket "linenos=table,linenostart=1">}}
#lang racket

(define next (current-seconds))

(define (my-random)
  (set! next (modulo
              (+ (* next 6364136223846793005) 1442695040888963407)
              (expt 2 64)))
  next)
{{< / highlight >}}

You can see the output this produces from the racket repl here:

{{< highlight shell>}}
> (my-random)
14843871667761315205
> (my-random)
534325907729616048
> (my-random)
5734414832404007999
{{< / highlight >}}

This isn't how Racket generates its random numbers (it uses an implementation of the [MRG32k3a algorithm](https://github.com/racket/racket/blob/5bb837661c12a9752c6a99f952c0e1b267645b33/racket/src/cs/rumble/random.ss)), however this method is used by other languages such as the function defined in [glibc](https://sourceware.org/git/?p=glibc.git;a=blob;f=stdlib/random_r.c;hb=glibc-2.26#l362)

#### Choosing our <i>m</i> and <i>a</i> values

It's important to note here that the numbers used above were not chosen by random, in particular it is important to pay extra attention to the numbers we choose for <i>m</i> and <i>a</i>.

* our m value controls the "period" of the random number generator which is how long it takes for it to start repeating. ie if we should choose something such as 2, we'll end up producing: 0,1,0,1,0... so choosing a larger number is ideal.
* m should also be a number that will make division quick since it is a comparatively slow operation. Knuth suggests that you use the word size of the computer ie. 2<sup>64</sup> or use the largest prime smaller than the word size.
* Choosing the a value is more complicated than m but equally important as poor choices for a have historically resulted in poor generators. The rules are dependant on the values of c, and m but are too long to include and explain here. [The wiki entry](https://en.wikipedia.org/wiki/Linear_congruential_generator) contains a more detailed explanation of the choices when it comes to choosing a.

### Combining random number generators

<!-- TODO: Finish this section -->
<!-- Knuth mentions that putting two of them together obliterates any feelings that they may not be random enough -->

* Algorithm M
  * And the one that came after it that only uses one algorithm
* The monty carlo method and just throwing results away (so long as we have a long period we can do this fairly well)
  * generating 500 numbers, using the first 55, then throwing out the rest and generating another 500

## Statistical Tests Or How we can prove what's <i>really</i> random

Once we've created a random number generator, we need a way to decided if it's actually random or not. Knuth suggests we run a battery of tests on the generator in order to determine that it's satisfactorily random. He also humorously suggests that if the tests don't come out the way we want to try reading [another book](https://en.wikipedia.org/wiki/How_to_Lie_with_Statistics).

To thoroughly evaluate a random number generator he suggests running the following tests multiple times:

* Equidistribution test (Frequency Test)
* Serial Test: <!-- TODO: this needs some explanation -->
* Gap Test
* Poker Test:
* Coupon Collectors Test: How many times do you need to run until you have one of each of the possible values.
* Permutation Test
* Runs test: can be runs up or runs down, finding the lengths of all runs such as 1298567 would have runs |129|8|567|
* maximum of t test: 
* collision test: Testing how often we get the same number in a set. ie if we have 2^20 possibilities and we run 2^14 times we would expect that on average, each number would appear 2^6 times.
* Birthday Spacings test:
* Serial correlation test
* tests on subsequences

Then analyzing them using the [chi-squared test](https://en.wikipedia.org/wiki/Chi-squared_test) and the [Kolmogorov–Smirnov test](https://en.wikipedia.org/wiki/Kolmogorov%E2%80%93Smirnov_test) to analyze the results of the tests.

<!-- TODO: Add a little more briefly explaining each test -->
<!-- TODO: recheck to see if I'm correct abut how these tests are being applied -->

### Why So Many Tests?

True randomness is difficult to obtain, some might argue that it may be impossible. This is even more true because we are working with computers after all which <i>should</i> not behave randomly. For this reason we need to determine that they are significantly random enough. To determine this we need to use thorough statistical analysis to show us in a quantifiable way how random they are. This can be difficult and time consuming to prove as it turns out even using just a few of these tests presents problems as some random generators will pass a few of these tests, yet completely fail others. Luckily most of the math here has been done for us, and we are unlikely to be developing our own, but there are a few notable examples where this happened to fail such as [RANDU](https://en.wikipedia.org/wiki/RANDU) which Knuth called "truly horrible".