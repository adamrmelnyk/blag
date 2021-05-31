---
title: "Why I use Lynx"
date: 2021-05-15T19:05:48-05:00
description: "Using lynx in 2021"
draft: false
toc: false
categories: ["technology"]
tags: ["web", "browsers", "lynx", "terminal"]
---

Lynx may be the oldest, currently maintained browser around. Though it looks nothing like a modern browser and will probably break trying to load any react based webpage, I still think it's one of the most underrated and underutilized browsers, which is why I still find myself using it on an almost daily basis.

What Lynx offers:

* VI (and Emacs) bindings
* its own dotfile (.lynxrc)
* distraction free reading by default
* explicitly asks your permission to keep each cookie
* no images, CSS, or Javascript
* makes you feel like a hacker

## Simple user interface

Like everything else in the terminal, the user interface for lynx is all keyboard based. `Q` quits, the arrow keys move your cursor to the next, clickable link, and the space bar will scroll you down to the next page. Everything else is listed at the bottom of the terminal.

```shell
Commands: Use arrow keys to move, '?' for help, 'q' to quit, '<-' to go back.
  Arrow keys: Up and Down to move.  Right to follow a link; Left to go back.
 H)elp O)ptions P)rint G)o M)ain screen Q)uit /=search [delete]=history list
 ```

## Minimal user tracking

If you're concerned about privacy and being tracked online, then look no further! Because lynx is a **text** browser and does not load any graphics, CSS, or javascript, there is very little that can be used to track the user other than accepting cookies in which case, Lynx will prompt you to accept or reject each cookie. You can also view which cookies have already been stored by hitting `Ctrl + k`.

```shell
www.theglobeandmail.com cookie: ak_user={"latitude":"30.2669" Allow? (Y/N/Always/neVer)
```

Viewing your favorite news website on lynx can be an eye opening experience, it's one of the few times when you might be suspicious of someone offering you a dozen different cookies. Of course if your intention is to reject all cookies, this can get very tedious so it's best to just black hole everything and do something like:

```shell
lynx -accept_all_cookies -cookie_file=/dev/null
```

<!-- This looks a little silly but, as far as I can tell this is the best way to get lynx to ignore all cookies. Though there is an accept-all-cookies flag, there is no reject-all-cookies flag. -->

Though changing your user agent likely provides very minimal amounts of privacy, Lynx will also let you easily change your user agent to anything you'd like:

```shell
lynx -useragent="NotChrome"
```

Or if you're feeling particularly evil:

```shell
lynx -useragent="Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)"'
```

## Focus and Readability

One of the key features that makes Lynx valuable as a browser, is how much easier it makes focusing on what I'm reading. Many websites feature a lot of really distracting content, images, pop up modals, banner ads, colours, and sponsored content, all of which are designed and A/B tested to maximize attention grabbing and get you to click away to somewhere else or keep you on their website longer. All of this pushing and pulling makes for a very unfocused reading experience. Lynx cuts down on the noise. Much like the reader view in Firefox, Lynx makes it a lot easier to focus on what you're reading rather than be distracted by anything else. All you see is the text on the website. No click-bait headlines in the margins, just text.

### A cleaner experience

Incredibly, some websites actually provide super light versions when viewed with lynx. DuckDuckGo automatically redirects to a "lite" version when using lynx (though not if you change the user agent!) giving you a very clean and usable webpage in lynx.

```shell
#←←←                                                                     DuckDuckGo
   #DuckDuckGo (Lite)

                                   DuckDuckGo
                ________________________________________ Search

```

## Configurability

Like many other command line utils, lynx is also configurable. It has it's own editable configuration file as well as giving you the option of defining your own `.lynxrc` file so you can take your configuration to other machines. There are [a lot of options](https://lynx.invisible-island.net/lynx_help/cattoc.html), many browsers provide similar functionality, though not without signing in. As well, because a lot of these configurations are available as command line flags, you can alias commands with flags giving you the ability to almost have various browser profiles depending on what you're doing.

```shell
alias lynxnocookies='lynx -accept_all_cookies -cookie_file=/dev/null'
alias lynxwhistory='lynx -accept_all_cookies -cookie_file=~/.lynx_cookies'
```

## What Lynx can't do

I'll admit there are some things that are less than perfect about Lynx. You aren't going to be able to watch Youtube, scroll Instagram, or browse Reddit easily. As modern web development evolves, more and more of the web is being developed with javascript. Many websites no longer work at all without javascript being enabled (Not this website! It looks great in lynx! Try it out!). 'Web apps' will almost never work with lynx as they will require javascript to do the most basic of functions such as submit forms and opening menus. Lynx will never excel at this kind of task, what it does well is text, and text only.

## Other terminal browsers

Lynx isn't the only terminal browser, w3m is also quite popular and provides similar functionality, though it uses the cursor like a mouse meaning you need to use the arrow keys to move the cursor around in order to select anything.

elinks provides a similar experience to lynx, however I would not recommend it's use. Because of it's lack of maintenance it has been removed from the debian repository.

The most interesting by far of course is [browsh](https://github.com/browsh-org/browsh). It works just like normal, modern browser, rendering javascript, CSS, and even video! It's truly amazing though very different from the use case that I normally use lynx for.

## Why you should Try Lynx

Lynx isn't for everyone, the first time you browse the web with lynx, when the oddly formatted text stares back at you, it might feel as if you took a step back in time to 80's. You won't be able to read your Twitter dms, or maybe even log into your email, but if you're looking for something without all the distractions of the modern web, lynx is the perfect choice.