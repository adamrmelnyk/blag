---
title: "Random Numbers"
date: 2019-05-08T15:30:34-05:00
description: "Working through Volume 2 of TAOCP"
draft: false
toc: false
categories: ["computer science"]
tags: ["taocp", "knuth", "vol2"]
images: []
---

Random numbers from the second volume of TAOCP and experimentation with PRNGs.

It's been about nine months since I finished reading the first volume. Though it's likely not a surprise to many, it's one of the more daunting reads, but is surprisingly entertaining and well written. [My summary of the experience reading through the first volume](/post/the-art-of-computer-programming/) was more or less my thoughts on the book, but didn't offer a lot in terms of the content that was covered or any interesting material that was gleamed from its pages; perhaps I'll remedy that when I have time for a second read through. Admittedly, there was quite a bit of content and I don't think I wanted to post a section by section summary of the volume, especially when you consider that much of the first volume was devoted to stacks, trees, linked lists, and other data structures, something most software engineers and programmers are familiar with. Luckily the second volume is far less familiar territory (at least to me) and may be more interesting to write about.

<!--more-->

### Not So Random Number Generators

Early in the volume Knuth shows how difficult it is to create a sufficiently random pseudorandom number generator. Simply adding a number of complicated steps will may not yield truly random numbers. Knuth includes the following algorithm, <b><i>Algorithm-K</i></b>, to illustrate this point. You can find an excerpt of the algorithm [here](http://www.informit.com/articles/article.aspx?p=2221790).

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

The first two examples produce what one might expect, but the third produces itself. Knuth points out that the algorithm happens to degenerate to 6065038420, with the lesson being:

<p style="text-align: center;"><b><i>"Random numbers should not be generated with a method chosen at random"</b></i></p>

Besides coincidentally degenerating to a seemingly random number, Algorithm K has a number of other downsides, it's difficult to implement and because it iterates a certain amount of times depending on the input, takes a relatively long time to produce a single random number. Somewhere inside that complexity there is a reason why the generator degenerates to 6065038420. In order to be usable we need to be able to easily prove that won't happen. The key take away from that is that we need this to be fast and simple.

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
* if m is a power of 2, pick a so that a % 8 = 5. This is to ensure that we produce all m possible values and gives our generator "high potency". There is a proof for this but it's far too long to include. In general it would seem that choosing the a value is more complicated than m but equally important as poor choices for a have historically resulted in poor generators. The rules are dependant on the values of m, and c but are too long to include and explain here. [The wiki entry](https://en.wikipedia.org/wiki/Linear_congruential_generator) contains a more detailed explanation of the choices when it comes to choosing a.

### Why not both?

The LCG is just one form of PRNG such as [xorshift](https://en.wikipedia.org/wiki/Xorshift), [WELL](https://en.wikipedia.org/wiki/Well_equidistributed_long-period_linear), or the [Mersenne Twister](https://en.wikipedia.org/wiki/Mersenne_Twister) so, if having one random number generator produces something fairly random, would two random number generators produce something even more random? As it turns out yes, but only if applied properly otherwise you could cause issues [like the one seen in Javascripts Math.random() library](https://medium.com/@betable/tifu-by-using-math-random-f1c308c4fd9d). If a person is unconvinced that a random number generator is not sufficiently random Putting two generators together should in Knuth words "convince all but the most hardened of skeptics". One method involves taking two generators and shuffling them together, it's only weakness being that it does not change the number themselves, only the sequence so this may still fail the [Birthday spacings](https://www.cs.indiana.edu/~kapadia/project2/node21.html) test. Knuth suggest instead that we use an even simpler method by combining results from the same generator, producing 500 or so results, choosing the first 55, then throwing the rest away and repeating the process.

## Statistical Tests Or How we can prove what's <i>really</i> random

Once we've created a random number generator, we need a way to decided if it's actually random or not. Knuth suggests we run a battery of tests on the generator in order to determine that it's satisfactorily random. He also humorously suggests that if the tests don't come out the way we want to try reading [another book](https://en.wikipedia.org/wiki/How_to_Lie_with_Statistics).

To thoroughly evaluate a random number generator he suggests running the following tests multiple times:

* Equidistribution test (Frequency Test)
* Serial Test
* Gap Test
* Poker Test
* Coupon Collectors Test
* Permutation Test
* Runs test
* Maximum of t test
* Collision test
* Birthday Spacings test
* Serial correlation test
* Tests on subsequences

Then analyzing them using the [chi-squared test](https://en.wikipedia.org/wiki/Chi-squared_test) and the [Kolmogorov–Smirnov test](https://en.wikipedia.org/wiki/Kolmogorov%E2%80%93Smirnov_test) to analyze the results of the tests.

### Spectral test

The most important of all the tests is the Spectral test. This test is make or break for random number generators. All good PRNG's will pass this test, and more importantly, bad PRNG's will fail. There are various examples of this online but one of the more interesting that I came across was of the RANDU generator and why it fails the spectral test can be seen [here](https://www.youtube.com/watch?v=7wVk_XsxyNk)

### Why So Many Tests?

True randomness is difficult to obtain, some might argue that it may be impossible. This is even more true because we are working with computers after all which <i>should</i> not behave randomly. For this reason we need to determine that they are significantly random enough. To determine this we need to use thorough statistical analysis to show us in a quantifiable way how random they are. This can be difficult and time consuming to prove as it turns out even using just a few of these tests presents problems as some random generators will pass a few of these tests, yet completely fail others. Luckily most of the math here has been done for us, and we are unlikely to be developing our own, but there are a few notable examples where this happened to fail such as [RANDU](https://en.wikipedia.org/wiki/RANDU) which Knuth called "truly horrible".

### Running some tests

Reading about these tests was interesting, but what would be more interesting is to run it on the two generators I had used above, as well as RANDU which is shown below. Comparing the results should illustrate the differences between the generators that ought to be good, and those that we know will preform poorly. There are a number of random number generator testers out there but the one I used was `dieharder` based on a suite of tests called diehard. For each run I generated 1,000,0000 random numbers (algorithm k took significantly longer ~20 min) and fed it to diehard. You can see the results here:

# Standard LCG

{{< highlight shell "linenos=table,linenostart=1">}}
#=============================================================================#
#            dieharder version 3.31.1 Copyright 2003 Robert G. Brown          #
#=============================================================================#
   rng_name    |           filename             |rands/second|
     file_input|                   example.input|  6.15e+06  |
#=============================================================================#
        test_name   |ntup| tsamples |psamples|  p-value |Assessment
#=============================================================================#
# The file file_input was rewound 1 times
   diehard_birthdays|   0|       100|     100|0.72874473|  PASSED  
# The file file_input was rewound 11 times
      diehard_operm5|   0|   1000000|     100|0.00008973|   WEAK   
# The file file_input was rewound 24 times
  diehard_rank_32x32|   0|     40000|     100|0.24249094|  PASSED  
# The file file_input was rewound 30 times
    diehard_rank_6x8|   0|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 32 times
   diehard_bitstream|   0|   2097152|     100|0.00000000|  FAILED  
# The file file_input was rewound 53 times
        diehard_opso|   0|   2097152|     100|0.00000000|  FAILED  
# The file file_input was rewound 67 times
        diehard_oqso|   0|   2097152|     100|0.00000000|  FAILED  
# The file file_input was rewound 74 times
         diehard_dna|   0|   2097152|     100|0.00000000|  FAILED  
# The file file_input was rewound 74 times
diehard_count_1s_str|   0|    256000|     100|0.00000000|  FAILED  
# The file file_input was rewound 87 times
diehard_count_1s_byt|   0|    256000|     100|0.00000000|  FAILED  
# The file file_input was rewound 87 times
 diehard_parking_lot|   0|     12000|     100|0.33164281|  PASSED  
# The file file_input was rewound 88 times
    diehard_2dsphere|   2|      8000|     100|0.08003531|  PASSED  
# The file file_input was rewound 88 times
    diehard_3dsphere|   3|      4000|     100|0.55288986|  PASSED  
# The file file_input was rewound 111 times
     diehard_squeeze|   0|    100000|     100|0.00371770|   WEAK   
# The file file_input was rewound 111 times
        diehard_sums|   0|       100|     100|0.06120371|  PASSED  
# The file file_input was rewound 112 times
        diehard_runs|   0|    100000|     100|0.35781871|  PASSED  
        diehard_runs|   0|    100000|     100|0.07426726|  PASSED  
# The file file_input was rewound 125 times
       diehard_craps|   0|    200000|     100|0.00004611|   WEAK   
       diehard_craps|   0|    200000|     100|0.44395534|  PASSED  
# The file file_input was rewound 325 times
 marsaglia_tsang_gcd|   0|  10000000|     100|0.00000000|  FAILED  
 marsaglia_tsang_gcd|   0|  10000000|     100|0.00000000|  FAILED  
# The file file_input was rewound 326 times
         sts_monobit|   1|    100000|     100|0.34446776|  PASSED  
# The file file_input was rewound 327 times
            sts_runs|   2|    100000|     100|0.34889544|  PASSED  
# The file file_input was rewound 330 times
         rgb_bitdist|   1|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 334 times
         rgb_bitdist|   2|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 340 times
         rgb_bitdist|   3|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 348 times
         rgb_bitdist|   4|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 358 times
         rgb_bitdist|   5|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 370 times
         rgb_bitdist|   6|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 384 times
         rgb_bitdist|   7|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 400 times
         rgb_bitdist|   8|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 418 times
         rgb_bitdist|   9|    100000|     100|0.00005693|   WEAK   
# The file file_input was rewound 438 times
         rgb_bitdist|  10|    100000|     100|0.53597412|  PASSED  
# The file file_input was rewound 460 times
         rgb_bitdist|  11|    100000|     100|0.89275780|  PASSED  
# The file file_input was rewound 484 times
         rgb_bitdist|  12|    100000|     100|0.01596294|  PASSED  
# The file file_input was rewound 486 times
rgb_minimum_distance|   2|     10000|    1000|0.00000000|  FAILED  
# The file file_input was rewound 489 times
rgb_minimum_distance|   3|     10000|    1000|0.00020551|   WEAK   
# The file file_input was rewound 493 times
rgb_minimum_distance|   4|     10000|    1000|0.00000000|  FAILED  
# The file file_input was rewound 498 times
rgb_minimum_distance|   5|     10000|    1000|0.00000000|  FAILED  
# The file file_input was rewound 500 times
    rgb_permutations|   2|    100000|     100|0.27181166|  PASSED  
# The file file_input was rewound 503 times
    rgb_permutations|   3|    100000|     100|0.67601712|  PASSED  
# The file file_input was rewound 507 times
    rgb_permutations|   4|    100000|     100|0.05999982|  PASSED  
# The file file_input was rewound 512 times
    rgb_permutations|   5|    100000|     100|0.16764489|  PASSED  
# The file file_input was rewound 522 times
      rgb_lagged_sum|   0|   1000000|     100|0.00003809|   WEAK   
# The file file_input was rewound 542 times
      rgb_lagged_sum|   1|   1000000|     100|0.00000000|  FAILED  
# The file file_input was rewound 572 times
      rgb_lagged_sum|   2|   1000000|     100|0.00000015|  FAILED  
# The file file_input was rewound 612 times
      rgb_lagged_sum|   3|   1000000|     100|0.00000000|  FAILED  
# The file file_input was rewound 662 times
      rgb_lagged_sum|   4|   1000000|     100|0.00000000|  FAILED
{{< / highlight>}}

# RANDU
{{< highlight racket "linenos=table,linenostart=1">}}
#lang racket

(define next 1090815429)

(define (randu)
  (set! next (modulo (* next 65539) (expt 2 31)))
  next)
{{< / highlight >}}
</br>
{{< highlight shell "linenos=table,linenostart=1">}}
#=============================================================================#
#            dieharder version 3.31.1 Copyright 2003 Robert G. Brown          #
#=============================================================================#
   rng_name    |           filename             |rands/second|
     file_input|                     randu.input|  7.94e+06  |
#=============================================================================#
        test_name   |ntup| tsamples |psamples|  p-value |Assessment
#=============================================================================#
# The file file_input was rewound 1 times
   diehard_birthdays|   0|       100|     100|0.00000053|  FAILED  
# The file file_input was rewound 11 times
      diehard_operm5|   0|   1000000|     100|0.00000000|  FAILED  
# The file file_input was rewound 24 times
  diehard_rank_32x32|   0|     40000|     100|0.00000000|  FAILED  
# The file file_input was rewound 30 times
    diehard_rank_6x8|   0|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 32 times
   diehard_bitstream|   0|   2097152|     100|0.00000000|  FAILED  
# The file file_input was rewound 53 times
        diehard_opso|   0|   2097152|     100|0.00000000|  FAILED  
# The file file_input was rewound 67 times
        diehard_oqso|   0|   2097152|     100|0.00000000|  FAILED  
# The file file_input was rewound 74 times
         diehard_dna|   0|   2097152|     100|0.00000000|  FAILED  
# The file file_input was rewound 74 times
diehard_count_1s_str|   0|    256000|     100|0.00000000|  FAILED  
# The file file_input was rewound 87 times
diehard_count_1s_byt|   0|    256000|     100|0.00000000|  FAILED  
# The file file_input was rewound 87 times
 diehard_parking_lot|   0|     12000|     100|0.00000000|  FAILED  
# The file file_input was rewound 88 times
    diehard_2dsphere|   2|      8000|     100|0.00000000|  FAILED  
# The file file_input was rewound 88 times
    diehard_3dsphere|   3|      4000|     100|0.00000000|  FAILED  
# The file file_input was rewound 101 times
     diehard_squeeze|   0|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 101 times
        diehard_sums|   0|       100|     100|0.00000000|  FAILED  
# The file file_input was rewound 102 times
        diehard_runs|   0|    100000|     100|0.00055174|   WEAK   
        diehard_runs|   0|    100000|     100|0.00915037|  PASSED  
# The file file_input was rewound 118 times
       diehard_craps|   0|    200000|     100|0.00000000|  FAILED  
       diehard_craps|   0|    200000|     100|0.00000000|  FAILED  
# The file file_input was rewound 318 times
 marsaglia_tsang_gcd|   0|  10000000|     100|0.00000000|  FAILED  
 marsaglia_tsang_gcd|   0|  10000000|     100|0.00000000|  FAILED  
# The file file_input was rewound 319 times
         sts_monobit|   1|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 320 times
            sts_runs|   2|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 323 times
         rgb_bitdist|   1|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 327 times
         rgb_bitdist|   2|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 333 times
         rgb_bitdist|   3|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 341 times
         rgb_bitdist|   4|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 351 times
         rgb_bitdist|   5|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 363 times
         rgb_bitdist|   6|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 377 times
         rgb_bitdist|   7|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 393 times
         rgb_bitdist|   8|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 411 times
         rgb_bitdist|   9|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 431 times
         rgb_bitdist|  10|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 453 times
         rgb_bitdist|  11|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 477 times
         rgb_bitdist|  12|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 479 times
rgb_minimum_distance|   2|     10000|    1000|0.00000000|  FAILED  
# The file file_input was rewound 482 times
rgb_minimum_distance|   3|     10000|    1000|0.00000000|  FAILED  
# The file file_input was rewound 486 times
rgb_minimum_distance|   4|     10000|    1000|0.00000000|  FAILED  
# The file file_input was rewound 491 times
rgb_minimum_distance|   5|     10000|    1000|0.00000000|  FAILED  
# The file file_input was rewound 493 times
    rgb_permutations|   2|    100000|     100|0.14813789|  PASSED  
# The file file_input was rewound 496 times
    rgb_permutations|   3|    100000|     100|0.49241270|  PASSED  
# The file file_input was rewound 500 times
    rgb_permutations|   4|    100000|     100|0.00108537|   WEAK   
# The file file_input was rewound 505 times
    rgb_permutations|   5|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 515 times
      rgb_lagged_sum|   0|   1000000|     100|0.00000000|  FAILED  
# The file file_input was rewound 535 times
      rgb_lagged_sum|   1|   1000000|     100|0.00000000|  FAILED  
# The file file_input was rewound 565 times
      rgb_lagged_sum|   2|   1000000|     100|0.00000000|  FAILED  
# The file file_input was rewound 605 times
      rgb_lagged_sum|   3|   1000000|     100|0.00000000|  FAILED  
# The file file_input was rewound 655 times
      rgb_lagged_sum|   4|   1000000|     100|0.00000000|  FAILED  
{{< / highlight >}}


# Algorithm-k

{{< highlight shell "linenos=table,linenostart=1">}}
#=============================================================================#
#            dieharder version 3.31.1 Copyright 2003 Robert G. Brown          #
#=============================================================================#
   rng_name    |           filename             |rands/second|
     file_input|                    backup.input|  7.99e+06  |
#=============================================================================#
        test_name   |ntup| tsamples |psamples|  p-value |Assessment
#=============================================================================#
# The file file_input was rewound 1 times
   diehard_birthdays|   0|       100|     100|0.00000000|  FAILED  
# The file file_input was rewound 11 times
      diehard_operm5|   0|   1000000|     100|0.00000000|  FAILED  
# The file file_input was rewound 24 times
  diehard_rank_32x32|   0|     40000|     100|0.00000000|  FAILED  
# The file file_input was rewound 30 times
    diehard_rank_6x8|   0|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 32 times
   diehard_bitstream|   0|   2097152|     100|0.00000000|  FAILED  
# The file file_input was rewound 53 times
        diehard_opso|   0|   2097152|     100|0.00000000|  FAILED  
# The file file_input was rewound 67 times
        diehard_oqso|   0|   2097152|     100|0.00000000|  FAILED  
# The file file_input was rewound 74 times
         diehard_dna|   0|   2097152|     100|0.00000000|  FAILED  
# The file file_input was rewound 74 times
diehard_count_1s_str|   0|    256000|     100|0.00000000|  FAILED  
# The file file_input was rewound 87 times
diehard_count_1s_byt|   0|    256000|     100|0.00000000|  FAILED  
# The file file_input was rewound 87 times
 diehard_parking_lot|   0|     12000|     100|0.00000000|  FAILED  
# The file file_input was rewound 88 times
    diehard_2dsphere|   2|      8000|     100|0.00000000|  FAILED  
# The file file_input was rewound 88 times
    diehard_3dsphere|   3|      4000|     100|0.00000000|  FAILED  
# The file file_input was rewound 108 times
     diehard_squeeze|   0|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 108 times
        diehard_sums|   0|       100|     100|0.00000018|  FAILED  
# The file file_input was rewound 109 times
        diehard_runs|   0|    100000|     100|0.00000000|  FAILED  
        diehard_runs|   0|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 123 times
       diehard_craps|   0|    200000|     100|0.00000000|  FAILED  
       diehard_craps|   0|    200000|     100|0.00000000|  FAILED  
# The file file_input was rewound 323 times
 marsaglia_tsang_gcd|   0|  10000000|     100|0.00000000|  FAILED  
 marsaglia_tsang_gcd|   0|  10000000|     100|0.00000000|  FAILED  
# The file file_input was rewound 324 times
         sts_monobit|   1|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 325 times
      diehard_operm5|   0|   1000000|     100|0.34393548|  PASSED  
            sts_runs|   2|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 328 times
         rgb_bitdist|   1|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 332 times
         rgb_bitdist|   2|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 338 times
         rgb_bitdist|   3|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 346 times
         rgb_bitdist|   4|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 356 times
         rgb_bitdist|   5|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 368 times
         rgb_bitdist|   6|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 382 times
         rgb_bitdist|   7|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 398 times
         rgb_bitdist|   8|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 416 times
         rgb_bitdist|   9|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 436 times
         rgb_bitdist|  10|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 458 times
         rgb_bitdist|  11|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 482 times
         rgb_bitdist|  12|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 484 times
rgb_minimum_distance|   2|     10000|    1000|0.00000000|  FAILED  
# The file file_input was rewound 487 times
rgb_minimum_distance|   3|     10000|    1000|0.00000000|  FAILED  
# The file file_input was rewound 491 times
rgb_minimum_distance|   4|     10000|    1000|0.00000000|  FAILED  
# The file file_input was rewound 496 times
rgb_minimum_distance|   5|     10000|    1000|0.00000000|  FAILED  
# The file file_input was rewound 498 times
    rgb_permutations|   2|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 501 times
    rgb_permutations|   3|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 505 times
    rgb_permutations|   4|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 510 times
    rgb_permutations|   5|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 520 times
      rgb_lagged_sum|   0|   1000000|     100|0.00000000|  FAILED  
# The file file_input was rewound 540 times
      rgb_lagged_sum|   1|   1000000|     100|0.00000000|  FAILED  
# The file file_input was rewound 570 times
      rgb_lagged_sum|   2|   1000000|     100|0.00000000|  FAILED  
# The file file_input was rewound 610 times
      rgb_lagged_sum|   3|   1000000|     100|0.00000000|  FAILED  
# The file file_input was rewound 660 times
      rgb_lagged_sum|   4|   1000000|     100|0.00000000|  FAILED
{{</ highlight >}}

There's already quite a bit to digest here so I didn't include all of the test results as the first few are enough to illustrate the differences between the three algorithms. One thing you will notice is that the standard LCG used actually failed a lot of tests. After doing a little testing and some digging it seems that dieharder will also fail these tests with its own generator with the same amount of numbers. In fact, you can really only expect good results with about 10 million or more numbers, but because diehard runs on a single cpu this would have taken a considerable amount of time to run. The comparison is really what we're after and from the results it's clear that algorithm-k is not suitable, nor is RANDU much better. I want to be able to see what kind of results I would see from running a much larger set so I ran one last run with 100,000,000 which shows that the lcg faired a little bit better.

{{< highlight shell >}}
#=============================================================================#
#            dieharder version 3.31.1 Copyright 2003 Robert G. Brown          #
#=============================================================================#
   rng_name    |           filename             |rands/second|
     file_input|                   example.input|  6.16e+06  |
#=============================================================================#
        test_name   |ntup| tsamples |psamples|  p-value |Assessment
#=============================================================================#
   diehard_birthdays|   0|       100|     100|0.50369944|  PASSED  
# The file file_input was rewound 1 times
      diehard_operm5|   0|   1000000|     100|0.34393548|  PASSED  
# The file file_input was rewound 2 times
  diehard_rank_32x32|   0|     40000|     100|0.76284541|  PASSED  
# The file file_input was rewound 3 times
    diehard_rank_6x8|   0|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 3 times
   diehard_bitstream|   0|   2097152|     100|0.00000000|  FAILED  
# The file file_input was rewound 5 times
        diehard_opso|   0|   2097152|     100|0.00000000|  FAILED  
# The file file_input was rewound 6 times
        diehard_oqso|   0|   2097152|     100|0.00000000|  FAILED  
# The file file_input was rewound 7 times
         diehard_dna|   0|   2097152|     100|0.00000000|  FAILED  
# The file file_input was rewound 7 times
diehard_count_1s_str|   0|    256000|     100|0.00000000|  FAILED  
# The file file_input was rewound 8 times
diehard_count_1s_byt|   0|    256000|     100|0.00000000|  FAILED  
# The file file_input was rewound 8 times
 diehard_parking_lot|   0|     12000|     100|0.82658516|  PASSED  
# The file file_input was rewound 8 times
    diehard_2dsphere|   2|      8000|     100|0.48942059|  PASSED  
# The file file_input was rewound 8 times
    diehard_3dsphere|   3|      4000|     100|0.95108230|  PASSED  
# The file file_input was rewound 11 times
     diehard_squeeze|   0|    100000|     100|0.38225241|  PASSED  
# The file file_input was rewound 11 times
        diehard_sums|   0|       100|     100|0.07389024|  PASSED  
# The file file_input was rewound 11 times
        diehard_runs|   0|    100000|     100|0.69622840|  PASSED  
        diehard_runs|   0|    100000|     100|0.74440767|  PASSED  
# The file file_input was rewound 12 times
       diehard_craps|   0|    200000|     100|0.34755313|  PASSED  
       diehard_craps|   0|    200000|     100|0.93778967|  PASSED  
# The file file_input was rewound 32 times
 marsaglia_tsang_gcd|   0|  10000000|     100|0.00000000|  FAILED  
 marsaglia_tsang_gcd|   0|  10000000|     100|0.00000000|  FAILED  
# The file file_input was rewound 32 times
         sts_monobit|   1|    100000|     100|0.13113454|  PASSED  
# The file file_input was rewound 32 times
            sts_runs|   2|    100000|     100|0.22502954|  PASSED  
# The file file_input was rewound 33 times
         rgb_bitdist|   1|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 33 times
         rgb_bitdist|   2|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 34 times
         rgb_bitdist|   3|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 34 times
         rgb_bitdist|   4|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 35 times
         rgb_bitdist|   5|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 37 times
         rgb_bitdist|   6|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 38 times
         rgb_bitdist|   7|    100000|     100|0.00000000|  FAILED  
^[[1;5D# The file file_input was rewound 40 times
         rgb_bitdist|   8|    100000|     100|0.00000000|  FAILED  
# The file file_input was rewound 41 times
         rgb_bitdist|   9|    100000|     100|0.00048263|   WEAK   
# The file file_input was rewound 43 times
         rgb_bitdist|  10|    100000|     100|0.00000207|   WEAK   
# The file file_input was rewound 46 times
         rgb_bitdist|  11|    100000|     100|0.11712560|  PASSED  
# The file file_input was rewound 48 times
         rgb_bitdist|  12|    100000|     100|0.00070481|   WEAK   
# The file file_input was rewound 48 times
rgb_minimum_distance|   2|     10000|    1000|0.00000000|  FAILED  
# The file file_input was rewound 48 times
rgb_minimum_distance|   3|     10000|    1000|0.00073621|   WEAK   
# The file file_input was rewound 49 times
rgb_minimum_distance|   4|     10000|    1000|0.00000000|  FAILED  
# The file file_input was rewound 49 times
rgb_minimum_distance|   5|     10000|    1000|0.00000148|   WEAK   
# The file file_input was rewound 50 times
    rgb_permutations|   2|    100000|     100|0.64531526|  PASSED  
# The file file_input was rewound 50 times
    rgb_permutations|   3|    100000|     100|0.12491885|  PASSED  
# The file file_input was rewound 50 times
    rgb_permutations|   4|    100000|     100|0.29544943|  PASSED  
# The file file_input was rewound 51 times
    rgb_permutations|   5|    100000|     100|0.57519225|  PASSED  
# The file file_input was rewound 52 times
      rgb_lagged_sum|   0|   1000000|     100|0.42557400|  PASSED  
# The file file_input was rewound 54 times
      rgb_lagged_sum|   1|   1000000|     100|0.94028320|  PASSED  
# The file file_input was rewound 57 times
      rgb_lagged_sum|   2|   1000000|     100|0.34329913|  PASSED  
# The file file_input was rewound 61 times
      rgb_lagged_sum|   3|   1000000|     100|0.02887377|  PASSED  
# The file file_input was rewound 66 times
      rgb_lagged_sum|   4|   1000000|     100|0.02328575|  PASSED  
# The file file_input was rewound 72 times
      rgb_lagged_sum|   5|   1000000|     100|0.10863760|  PASSED  
{{</ highlight >}}

Here we can see the larger set actually did quite a bit better, passing where it was either weak or had even failed before. It's not perfect of course, and certainly not suitable for cryptographic use but we can see that the generator works well enough. I would have liked to have run more tests and tried a few different random number generators, but testing with dieharder was taking quite a long time on my underpowered laptop CPU. Of course no generator is going to pass every test as Knuth states: <i>"A truly random sequence will exhibit local non-randomness"</i>. Proving that a generator is suitable would require many more multiple runs on larger data sets. Only then might say that the PRNG is satisfiably random.

## Not So Random Musings

In a set of infinite numbers we might expect to see 1,000,000 7's or 8's in a row at some point. Maybe the numbers, viewed in a single line would map to ascii values that could print the works of Shakespeare. It may not be desirable for applications, but it would be consistent with a truly random number generator. Though the math was certainly beyond me, the first half of volume 2 was incredibly interesting. If anything it's made me question how random numbers are generated a little more, forcing me to dig through the random number generators that I have trusted. A simple mistake could result in unexpected deterministic behaviour. Though this is unlikely to be a problem in my day to day life as a web developer, it will still make me think the next time I need to rely on random behaviour. Further fueling my paranoia, Knuth gives some great advice to summarise the learnings of the chapter:

<p style="text-align: center;"><b><i>"Look at the subroutine library of each computer installation in your organization, and replace the random number generators by good ones. Try to avoid being too shocked at what you find."</i></b></p>
