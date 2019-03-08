---
title: "Functional Programming in Racket"
date: 2019-03-03T11:14:34-06:00
draft: false
description: "A short dive into Racket"
toc: false
categories: ["technology", "languages"]
tags: ["Functional", "Racket"]
images: [] # overrides the site-wide open graph image
---

A dive into functional programming in Racket.

# Preface

I've been playing around with functional languages for a few years now; I'd pick up an ebook on scala or Clojure some Saturday afternoon and try going through a tutorial or two. Maybe attend a workshop on it over a weekend with a friend who found the event on meetup.com and get a somewhat shallow overview of the language. The next weekend I'd look at it again and see if I could get a small webpage up and running or maybe a small command line tool but for the most part, it has always ended there. I rarely end up going back the environment because I inevitably get distracted by another cool technology. What follows is an attempt to remedy that and do a deeper dive over a period of a month and a half.

# Why Racket? Why not X

Almost two years ago, I found about about the Racket language and worked through the [Beautiful Racket](https://beautifulracket.com/) tutorial. I found the language to actually be pretty fun to work with and was interested in writing a DSL which Racket was built for. Though I have used Clojure and scala before as well, I had written more code in Racket as well as some Chez-Scheme (which is quite similar) in school. Between that and [John Carmacks tweet from a few years ago](https://twitter.com/ID_AA_Carmack/status/577877590070919168), Racket seemed like a good choice.

# Getting started

Before I start building something I want to get a feel for the language so I started by making a simple FizzBuzz function. The code is simple enough to read, define the function, which loops through 0 to some given argument and prints when certain conditions are met. I'm not sure if this is idiomatic Racket but I'm not concerned about that just yet.

{{< highlight racket>}}
#lang racket
(define (fizz-buzz num)
  (for ([i num])
    (cond
      [(= i 0)]
      [(and (= (modulo i 3) 0) (= (modulo i 5) 0)) (println "FizzBuzz")]
      [(= (modulo i 3) 0) (println "Fizz")]
      [(= (modulo i 5) 0) (println "Buzz")]
      [else (println i)])))

(fizz-buzz 20)
{{< / highlight >}}

# Diving into the documentation

I wouldn't usually do it this way, but I decided I should read through the docs, more specifically the [Racket guide](https://docs.racket-lang.org/guide/index.html?q=loop) on the language before continuing on to something slightly larger. There was a lot of information there but here were some of things that I found interesting.

# Getting started: A few things I noticed

* The guide is actually pretty thorough, giving examples of by showing how functions in the standard language can be formed using other calls which I feel is really in the spirit of using a recursive language. <!-- reword -->

* Development in Dr Racket, although dated looking is nice. The interactive shell at the bottom makes it easy to experiment while you're building without having to rebuild or reload the file. Hitting `Ctrl + ?` in the shell will also give you suggestions for words. I thought that tab completion was working at some point as well but, when I updated to the latest version of DrRacket it seemed to stop working for whatever reason.

* `car`, and `cdr`, one of the things I remembered from using scheme in college, can still be used though they can be swapped for the more intuitive `first` and `rest` when working with lists.

* I kind of love the conciseness that comes from using ternaries in other languages, but because it's a functional language, pretty much everything has that feel to it. The statement: `(if (> 3 4) "greater" "less")` looks a lot like a ternary. The only caveat here is that you need to return something for where it's true or false. When you want just and if you need to use something like when: `(when (condition) body)`.

* There is no while loop: I spend a lot of time working on OO programming in my daily life, so I found on a few occasions after making a few searches, trying to figure out why something so simple doesn't exist, I finally realized my mistake. Getting out of the trap of OO thinking takes some getting used to but, it turns out to be really powerful. Being able to pass along a function instead of just data really does simplify logic and saves you a lot of boilerplate code that OO languages like Java are notorious for.

* `cond` is particularly useful; in a number of cases and I found it to be much more expressive than the switches than many other languages provide. Instead of passing in a single arg and checking the value like Java does you can test for any condition you want, similar to chaining several `else if` statements, though much more powerful, as it can be passed like any other function.

* Though most OO languages have some aspects of functional programming with lambdas, it's nowhere near as intuitive. Treating functions are arguments allows you to things you would never even think about in Object Oriented programming. Instead of checking if an argument was a string or an Integer, casting it and then applying the correct function we check if it's a string and pass the function along to the two arguments. This isn't something unique to Racket, but it is something that I appreciated a lot the longer I used it. The example taken from the docs here, shows off just how easy it is:

{{< highlight racket>}}
(define (double v)
    ((if (string? v) string-append +) v v))
{{< / highlight >}}

* Racket is almost syntax-less; whereas many languages have different structures, rules and often shortcuts for defining variables, checking conditionals, creating switches, or creating functions is all the same. Not having to look up syntax greatly simplified the process of learning the language.

# Something Slightly Larger

Naturally, I want to be able to work on something slightly larger next so I started with something I've already worked on before, and I set out creating a script for parsing and making small edits to gif files. To my surprise working at the byte level in racket is fairly simple.

## Byte-ing off more than I can chew

This may have not been the best choice for the first thing I should try to build, but I wanted to pick something somewhat off the beaten path to see what kind of problems I might run into. I thought I might run into issues with bracket completion, but that never really happened even though there wasn't any bracket completion turned on. The majority of the issues that I ran into where more or less around using functions properly such as returning an expression or passing in the correct types to a function as even though the language is untyped there are functions that return errors if passed the wrong type of data.

## Getting to work

Bytes can be read using `read-bytes` and passes an input port which in this case was made from opening a file, and reads bytes in octal. I won't include the entire script here, just enough to show off the simplicity of Racket.

{{< highlight racket>}}
(define in (open-input-file "gif.gif"))
{{< / highlight >}}

{{< highlight racket>}}
#lang racket

(define (size-of-color-table-from-packed-field bstr bytes-before-pf)
  (define packed-field (extract-packed-field bstr bytes-before-pf))
  (* 3 (expt 2 (+ (bitwise-and (integer-bytes->integer packed-field #f) least-three-sig-mask) 1))))

(define (extract-packed-field bstr bytes-before-pf)
  (define in (open-input-bytes bstr))
  (read-bytes bytes-before-pf in) ;; The number of bytes in front of the packed field in the bstr
  (read-bytes 1 in))

(provide size-of-color-table-from-packed-field
         extract-packed-field)
{{< / highlight >}}

### Testing

If I'm going to want to use this for anything useful, a good test suite is an important consideration. Luckily testing was pretty straightforward as well, just make sure you provide the functions you intend to expose and [rackUnit](https://docs.racket-lang.org/rackunit/index.html) provides easy methods for testing.

{{< highlight racket>}}
#lang racket

(require rackunit "readgif.rkt")

(define readgif-tests
  (test-suite "Tests for readgif"
              (test-case "Extracting Size of color table"
                         (check-equal?
                          (size-of-color-table-from-packed-field #"\360\24\253;\344\21O\262\20\227" 4)
                          96))
              (test-case "Extracting the packed-field from a bstr"
                         (check-equal?
                          (extract-packed-field #"\314\1\367\0\367\377\0" 4)
                          #"\367")))
{{< / highlight >}}

You of course don't need all of this, the `check-equal?` is enough, but being able to organize the test suites like this is nice if you want to run just one of them. Though I don't know exactly how complicated a larger test suite would be overall, I'm happy with the minimal test framework that racket provides.

## Web development

The majority of my experience is in web development so the next step was to create a small website or an api. I had heard that Hackernews was written in racket or, more accurately, written arc, a language written in Racket, so I wanted to see how easy it would be to write a webpage in racket. The docs again shine through here with instant web servlets:

{{< highlight racket >}}
#lang web-server/insta
 
(define (start req)
  (response/xexpr
   `(html (head (title "Hello world!"))
          (body (p "This website was made with Racket!")))))
{{< / highlight >}}

Thi is nice of course, but ideally I don't want to have to roll everything myself. Luckily there are a few different web frameworks on github, though a few of them are in early stages or don't seem to have many contributions:

* https://github.com/nihirash/holy
* https://github.com/dmac/spin

### Taking Spin for a whirl

I decided to take a look at spin mostly because it reminded me a lot of the minimalist framework [Sinatra](http://sinatrarb.com/) which I have used before.

{{< highlight racket >}}
#lang racket

(require (planet dmac/spin))
(require web-server/templates)

(get "/"
  (λ () "Hello!"))

(get "/hi/:name" (λ (req)
  (string-append "Hello, " (params req 'name) "!")))

(run)
{{< / highlight >}}

Boom! This was more what I was looking for: Throw down some routes and then then the use whatever logic needs to run. The framework also includes templating and the like if you need something a little more complicated but there doesn't really seem to be anything in the way of react, rails, spring, or anything in the realm of fully fleshed out web frameworks. That really shouldn't be that surprising considering how small the user base is, and what the typical use case for the language is, so I didn't go much further in the way of creating a more robust web application though there is a static site generator [Frog](https://docs.racket-lang.org/frog/index.html). Though I probably won't consider moving over to Frog from the current static site generator I use for this site: [Hugo](https://gohugo.io/). It looked like it had some thorough documentation and room for customization.

### Making web requests

A lot of applications depend on making calls to third party apis, so I wanted to see what parsing json was like. There are a number of useful libraries for making [rest requests](https://github.com/jackfirth/racket-request) with racket, but I wanted to use just the base language. There were a few different ways to make requests but I found this to be the most intuitive and relevant to the majority of my use cases.

{{< highlight racket >}}
#lang racket
(require net/http-client
         json)

(define-values (status header response)
  (http-sendrecv "api.github.com" "/users" #:ssl? 'auto))
(define data (read-json response))

(when (jsexpr? data)
    (hash-ref (first data) 'login))
{{< / highlight >}}

Here the `http-sendrecv` produces three values: status, header, and response. The function then returns an input port which we can read as json. Really it builds a `#hasheq` type or a list of hashes depending on the results, so the Racket libraries for getting information from hashes makes this pretty simple to read from. In this example the json is a list of several results, so we read the first hash from the list in order to access the `'login` element.

# Conclusion

This is easily the fastest I've become productive in any particular language. The biggest strength of racket is being largely syntax-less meaning the only real learning that I needed to do was find out what functions were available through the base racket language. The docs were thorough, and though there were a lot of functions, the language still felt small enough that I started to feel like I knew what I was doing relatively quickly. I'm not sure that I would build a website from Racket at the moment, but I think that it does make a great general purpose language, and the next time I write a script or tool, I might just try using Racket first.