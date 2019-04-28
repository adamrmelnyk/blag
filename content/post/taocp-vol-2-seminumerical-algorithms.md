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

It's been about eight months (As of April) since I finished reading the first volume. Though it's likely not a surprise to many, I'll admit that it was one of the more challenging reads and though I've read through my share of books on computer algorithms, they really can't compare to the TAOCP. Though the language itself is fairly the simple, the content was some of the heaviest I've encountered. [My summary of the experience reading through the first volume](/post/the-art-of-computer-programming/) was more or less my thoughts on the book, but didn't offer a lot in terms of the content that was covered or any interesting material that was gleamed from its pages; perhaps I'll remedy that when I have time for a second read through. Admittedly, there was quite a bit of content and I don't think I wanted to post a section by section summary of the volume, especially when you consider that much of the first volume was devoted to stacks, trees, linked lists, and other data structures, something most software engineers and programmers are familiar with. Luckily the second volume is far less familiar territory and may be more interesting to write about. <!-- re-write this last part -->

# Seminumerical Algorithms

* What does seminumerical mean (the borderline of numeric and symbolic)
  * Numeric vs symbolic calculations

* Making a complicated procedure will not always result in something random
  * use the example of the K algorithm.
  * you need statistical analysis to determine if something is sufficiently random.

## Not So Random Number Generators

Early in the volume Knuth shows how difficult it is to create a sufficiently random pseudorandom number generator. Simply adding a number of complicated steps will may not yield truly random numbers. He includes the following algorithm <b><i>K</i></b> to illustrate this point.

### Algorithm K

{{<highlight racket "linenos=table,linenostart=1">}}
#lang racket

(define (algorithm-k x)
  (do-algorithm-k x (+ 1 (floor (/ x (expt 10 9))))))

(define (do-algorithm-k x iterations)
  (define step (string-append
                "k"
                (number->string
                 (+ (modulo (floor (/ x (expt 10 8))) 10) 3))))
  (if (eq? iterations 0)
      x
      (eval '(step x iterations))))

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
  (if (< 100000000)
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
  ;; TODO: Fill in
  
  )

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

The steps here aren't particularly important. It's more important that you see a 

## Linear Congruential Generator

The most common pseudorandom number generator (according to Knuth) is a [Linear congruential generator](https://en.wikipedia.org/wiki/Linear_congruential_generator), which has the following equation:

<p style="text-align: center;"><b><i>X</i></b><sub><i>n</i> + 1</sub> = (<i>a</i><b>X</b><sub><i>n</i></sub> + <i>c</i>) mod <i>m</i></p>

The equation isn't difficult to explain; numbers for m,a, and c are chosen and set in the compiler (hopefully not randomly) and a seed is chosen for the first value of Xn. This being a recurrence relation, the result of each run then becomes the next seed or starting value for a subsequent run. You can see this in the code below:

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

Of course this isn't how Racket generates its random numbers (Racket uses and implementation of the MRG32k3a algorithm), however this method is used by other languages such as the function defined in [glibc](https://sourceware.org/git/?p=glibc.git;a=blob;f=stdlib/random_r.c;hb=glibc-2.26#l362)

### Choosing our <i>m</i> and <i>a</i> values


<!-- MISC Notes
  * Racket uses a different algorithm: MRG32k3a
    * https://github.com/racket/racket/blob/5bb837661c12a9752c6a99f952c0e1b267645b33/racket/src/cs/rumble/random.ss
    * https://software.intel.com/en-us/mkl-vsnotes-mrg32k3a
    * https://stackoverflow.com/questions/11075833/lecuyers-mrg32k3a-random-number-generator-in-cuda-c
    * https://srfi.schemers.org/srfi-27/ -->

* Choosing and m value:
  * Should be large since the period cannot be larger than m. ie. if you choose m = 2, you'll end up with something like 0,1,0,1,0,1...
  * division is a comparatively slow operation so you should choose something that makes division easy. in this case a good choice is the word size
    of your computer. For most people then you should be using 2 ^ 64.
    * Or use the largest prime number smaller than than 2 ^ 64.
* Choosing the multiplier / a value:
  * when m is the product of distinct primes, a only a = 1 will produce the full period, but when m is divisble by a high power of some prime there is considerable latitude in the choice of a.
  * the proof for this is a few pages long, and I'm no mathematician so it won't be included or explained here.
    * JK I might
* Potency
  * Add explaination here and why it's important