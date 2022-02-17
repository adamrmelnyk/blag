---
title: "Creating Launch Plugins for Pop Os [Updated]"
date: 2022-02-16T22:08:48-06:00
draft: false
description: "[UPDATE] Customizing your desktop with Pop!_OS"
categories: ["Pop!_OS", "plugins", "System76"]
tags: ["programming", "plugins", "PopOS", "linux"]
---

## Background

This is an update to a [previous post](/post/creating-launch-plugins-for-pop-os/). In 2021, I wrote a short blog post on an easy way to create a plugin using the Pop_launcher in Pop!_OS. Unfortunately for me, Pop!_OS completely changed the way the their plugins work in 21.10! I found out when I received an email from someone saying that they had followed my instructions but didn't have any success with the plugin. Luckily for us the solution turned out to be really simple.

## Creating a launcher plugin

One of the cool features of Pop!_OS is the [Pop Shell](https://github.com/pop-os/shell), which brings the features of other tiling window managers like i3wm to GNOME. It also features launcher plugins which we can write ourselves in JS which is what we'll be doing in this post.

The easiest way to try this out is just to simply copy one of the existing plugin examples and just drop and replace what features we want to add instead.

## Making a Spotify plugin

I use Spotify pretty regularly, so it might be pretty handy to include a command on the launcher to pause/play Spotify or just select the next track.

## Where Plugins are located

Pop!_OS keeps it's plugins in two places:

- Apps are kept at `/usr/lib/pop-launcher/plugins/{plugin}/` and are written in Rust
- Scripts are kept in `/usr/lib/pop-launcher/scripts/{scripts}` and are written in Bash

For our purposes, writing a bash script will be sufficient, so we'll be working in the scripts directory.

## Pulling down an example

The first thing we can do is take an existing plugin and adapt it to our needs. We’ll need to copy one of the plugins from the scripts directory above. You can find these files on your machine running Pop!_OS at: `/usr/lib/pop-launcher/scripts/session`. *Do Not edit these plugins in place!!!*, otherwise the next time you try to restart your computer from the launcher you'll just end up switching the track on spotify. Copy them into some new directory of your choice, You can also take a look at them directly on [github](https://github.com/pop-os/launcher/blob/master/scripts/session/session-logout.sh). I've called mine `popify` and I've created a new file for the script we'll execute called: `popify-play.sh`.

## Toggling Pause and Play in Spotify

The next step is straight forward. We edit the file to look something like this:

```shell
#!/bin/sh
#
# name: Pause or Play Spotify
# icon: com.spotify.Client
# description: Pause or Play Spotify
# keywords: pause play spotify


dbus-send --print-reply --dest=org.mpris.MediaPlayer2.spotify /org/mpris/MediaPlayer2 org.mpris.MediaPlayer2.Player.PlayPause
```

The file is pretty simple:

- We've given it a name
- An existing icon (It's an svg file already on your computer if you already have spotify installed)
- A helpful description
- keywords which will be used to search for our plugin in the launcher
- lastly the command that will actually pause and play spotify

This is actually quite a bit easier than it was previously as it no longer requires a more complicated and rather boilerplate JavaScript setup, provided that you are comfortable doing it in Bash or don't want to do anything particularly complicated.

## Testing it out

Unlike the previous plugin which required you to restart the gnome shell. You should be able to save run the plugin as soon as you save the file.

![Pop_OS launcher](/images/launcher-2.png)

## Skipping to the next track

Now that you've made Spotify pause and play, you can easily make a second script to switch tracks: Simply make a new file in the same directory as you previous script. I've called mine: `popify-next.sh`

The file contents might look something like:

```shell
#!/bin/sh
#
# name: Play Next Song Spotify
# icon: com.spotify.Client
# description: Skip to the Next Song on Spotify
# keywords: next play spotify

dbus-send --print-reply --dest=org.mpris.MediaPlayer2.spotify /org/mpris/MediaPlayer2 org.mpris.MediaPlayer2.Player.Next
```

After you save the script above you should be able to skip to the next track.

![Pop_OS launcher](/images/launcher-next.png)

If you feel like a hacker at this point, it's because you are! So throw on those shades, put on your best black hoodie, and go forth and hack away on some cool plugins. Hack the ~~Planet~~ OS!

## Building more plugins

Hopefully this gives you a starting point for developing new plugins for Pop!_OS. There isn't much official documentation or tutorials on writing plugins other than what's in the github readme, but hopefully this inspires anyone who wanted to give Pop!_OS a try, another reason to checkout it out.

## Sources

* https://github.com/pop-os/launcher/blob/master/README.md
* Sample plugin: https://github.com/pop-os/launcher/blob/master/scripts/session/session-logout.sh
