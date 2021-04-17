---
title: "Creating Launch Plugins for Pop!_OS"
date: 2021-04-17T15:16:33-05:00
draft: false
description: "Customizing your desktop with Pop!_OS"
categories: ["Pop!_OS", "plugins", "System76"]
tags: ["programming", "plugins", "PopOS", "linux"]
---

## Background

I've been running linux on all of my machines for almost 15 years. For the first five I probably used a dozen or more flavours of Linux including OpenSuse, Fedora, Arch, Ubuntu, and many others. Eventually wiping my computer every month became more and more impractical and I abanonded my distrohopping ways, settling on Ubuntu. Ubuntu was stable, regularly updated, had an environment that I really liked (despite what some people may say about the unity interface) and I never looked back. That is until recently when I started using Pop!_OS.

It's been a long time since I last swtiched operating systems but, it made me realized what I liked about Linux so much to begin with: You control the software, not the other way around. Want to install a new desktop environment? There are multiple to choose from. Want to add custom keymappings and shortcuts? You can do that. Want add additional functionality to an existing app on your machine? Download the source code and modify it as you like. Ubuntu was the clear, safe course but, now that I'm running a new OS, I'm feeling more in the spirit of making my operating system do what I want it to do. New OS, new me! So let's get hacking.

## Creating a launcher plugin

One of the cool features of Pop!_OS is the [Pop Shell](https://github.com/pop-os/shell), which brings the features of other tiling window managers like i3wm to GNOME. It also features launcher plugins which we can write ourselves in JS which is what we'll be doing in this post.

The easiest way to try this out is just to simply copy one of the existing plugin examples and just drop and replace what features we want to add instead.

## Making a spotify plugin

I use Spotify pretty regularly, so it might be pretty handy to include a command on the launcher to pause/play spotify or just select the next track.

### Pulling down an example

The first thing we can do is take an existing plugin and adapt it to our needs. We'll need to copy the two files [here](https://github.com/pop-os/shell/tree/master/src/plugins/pulse). You can also find these files on your machine running Pop!_OS at: `/usr/lib/pop-shell/launcher/pulse`. You can either edit these files in place (Which I don't recommend since you're changing how the existing launcher plugins work!) or copy them into some new directory of your choice.

### Adapting the plugin to use with Spotify instead

Adapting the plugin for our purposes is actually relatively little work. The launcher plugin we're looking at just has a list of ID's that map to some command that can be executed in a shell, so the first thing we need to do is change those commands.

The App class contains a constructor with default selections. We want to change that to something like this:

```javascript
    constructor() {
        this.last_query = ""
        this.shell_only = false

        this.default_selections = [
            {
                id: 0,
                name: "Toggle Pause Play Spotify",
                description: "Plays or pauses the Spotify player",
            },

            {
                id: 1,
                name: "Next Spotify",
                description: "Plays the next track on the Spotify player"
            },
        ]
    }
```

If you walk through the code you'll see that it contains a function called `submit()` that reads the id codes from above and executes a command. We can change the variable being set by the switches above to something like:

```javascript
    submit(id) {
        let cmd = null

        switch (id) {
            case 0:
                cmd = "dbus-send --print-reply --dest=org.mpris.MediaPlayer2.spotify /org/mpris/MediaPlayer2 org.mpris.MediaPlayer2.Player.PlayPause"
                break
            case 1:
                cmd = "dbus-send --print-reply --dest=org.mpris.MediaPlayer2.spotify /org/mpris/MediaPlayer2 org.mpris.MediaPlayer2.Player.Next"
                break
        }
```

**NOTE:** I've also removed the line `let sinks = pactl_sinks()` above as well as the associated function `pactl_sinks()` since it's only related to pulse audio and not necessary for our spotify plugin.

If you're curious about what the commands above do, you can try running spotify and then plugging them in your terminal yourself. The first toggles pausing and playing spotify and the second switches to the next song.

The code for utilizing spotify is actually a bit simpler as well. We can just execute the string `cmd` instead of going through a loop.

```javascript
        if (cmd) {
            try {
                GLib.spawn_command_line_async(`${cmd}`)
            } catch (e) {
                log(`session command '${cmd}' failed: ${e}`)
            }
        }

        this.send({ event: "close" })
    }
```

There's one thing to change, we need to change the `meta.json` file to something that makes a little more sense:

```json
{
    "name": "Spotify Control",
    "description": "Control Spotify Play and Track selection",
    "exec": "main.js",
    "icon": "multimedia-volume-control"
}
```

Now that we've made the necessary changes we need to move it to where the rest of the plugins are, I've called my plugin popify (I know, so clever) but use whatever folder you've put your files into. You'll need to use sudo because the location requires privledged access.

```sh
sudo cp -r popify/ /usr/lib/pop-shell/launcher/.
```

After copying the files over to the correct directory last thing to do is restart the GNOME shell. There are many ways to do this such as just killing GNOME `killall -1 gnome-shell`, but if you're unsure, you can simply reboot your machine and the new plugin will be ready. If you've done everything right you should be able to see the new launcher when we type pause:

![Pop_OS launcher](/images/launcher.png)

this should toggle spotify to pause or play. You should also be able to type next and the player should switch songs.

If you feel like a hacker at this point, it's because you are! So throw on those shades, put your best black hoodie on, and go forth and hack away on some cool plugins. Hack the ~~Planet~~ OS!

## Building more plugins

Hopefully this gives you a starting point for developing new plugins for Pop!_OS. There isn't much official documentation or tutorials on writing plugins other than what's in the github readme, but hopefully this inspires anyone who wanted to give Pop!_OS a try, another reason to checkout it out.