---
title: "Debugging CHIP8 code"
date: 2020-08-23T15:45:10-05:00
description: "Debugging and overcoming some of the challenges with writing CHIP8 hex code"
draft: false
toc: false
categories: ["rust", "emulator"]
tags: ["chip8", "hexcode", "emulator", "rust"]
---

The first time I wrote a program in hexidecimal or assembly was using a made up, educational cpu called Pep8, this was before I had read anything about how to properly organize code, programming practices, and before I had read [clean code](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882). Over the last month and a half, I've been working on a [CHIP8](https://en.wikipedia.org/wiki/CHIP-8) emulator, so for the first time in a while, I'm revisiting old topics and old books on computer architecture. Now that the project is largely complete, what's left is to test it out, and the best way to do that, is to mess around with it and see what I can build for myself using the chip8 instruction set.

As I wrote many small programs to test each of the operations of the emulator, I ran into a number of scenarios where I'd end up re-writing instructions after parts of the code had moved around a lot. In my day to day work, I spend time thinking a lot about how to best organize my code. It's not enough to have it work, it's important to organize it as well so that it will be easier to maintain, reduce complexity, and be readable for engineers who may not be familiar with it. This got me thinking about how alien it was to debug CHIP8 programs and made me wonder whether or not there existed a "Clean Hex Code" or a "Clean assembly code" or perhaps something more akin to Dijkstra's [Structured Programming](https://en.wikipedia.org/wiki/Structured_programming) for CHIP8 code.

## Debugging is hard

Debugging is challenging when I write code in Java, Rust, Javascript etc. That's why it's so important that all modern languages have their own IDE's, editors, plugins, and other tools to aid the programmer in development. Even the C language, designed around the same time as CHIP8, has it's own tools that people use today. So when I encounter a bug in my CHIP8 program, I'm really left to my own devices. Aside from the fact that any time I encounter an error, I need to determine whether or not the error resides in the emulator I've written, or the assembly itself. Debugging the hex code is much more challenging for much less code.

To edit my CHIP8 programs I used ghex, a simple editor that I had used in the past to inspect binaries. There may be better tools out there but this was one that I am familiar with. Here's a simple program with the breakdown so we can follow what's going on.

| row | op | op | Description |
|-----|----|----|-------------|
| 0x200 | A2 | 06 | Set our address register to begin reading at location 206 |
| 0x202 | D0 | 05 | Draw at (0,0), 5 bytes starting from the location in memory specified by the address register |
| 0x204 | 00 | 00 | Halt |
| 0x206 | FF | C3 | The bytes that will be drawn |
| 0x208 | FF | C3 | |
| 0x210 | 00 |    | |

In this small program, what we're doing is printing a single letter to the top right hand corner of the screen. In this case, the letter "A". We load the address of where we want to start reading our letter from, into register 0, Then draw from five locations in memory beginning at the location stored in register 0. I should also note that although I am creating this letter myself after the Halt operation, CHIP8 loads sprites for letters and numbers in the first 512 address spaces so we could actually make this easier on ourselves by using the sprites given to us.

The result looks like this:

```sh
// ******** 0xFF 0b1111_1111
// **    ** 0xC3 0b1100_0011
// ******** 0xFF 0b1111_1111
// **    ** 0xC3 0b1100_0011
// **    ** 0xC3 0b1100_0011
```

The next thing I wanted to do was put a "+" sign next to the A. To do this I just repeat what I did before. Add the sprite for the plus after the A. Then I'll just draw once more.

| row | op | op | Description |
|-----|----|----|-------------|
| 0x200 | A2 | 06 | Set our address register to begin reading at location 206 |
| 0x202 | D0 | 05 | Draw the A |
| 0x204 | D0 | 06 | Draw the +|
| 0x206 | 00 | 00 | Halt |
| 0x208 | FF | C3 | The A Sprite |
| 0x20A | FF | C3 | |
| 0x20C | 18 | 18 | The + Sprite |
| 0x20E | FF | FF | |
| 0x210 | 18 | 18 | |

This should produce and A, and then a "+" that looks like this:

```sh
//    **    0x18 0b0001_1000
//    **    0x18 0b0001_1000
// ******** 0xFF 0b1111_1111
// ******** 0xFF 0b1111_1111
//    **    0x18 0b0001_1000
//    **    0x18 0b0001_1000
```

Of course *it doesn't*, because we've made a few mistakes which we'll fix next. Not only are we drawing twice in the same spot, but we're also reading our sprite from the wrong location! When we added the extra draw function, our sprite was loaded into a different place! To fix this we need to make a few changes. Part of why this doesn't work is because of how much set up the draw function requires. It reads the X coordinate from a specified register, the Y coordinate from another register, and then begins drawing a sprite reading from wherever the address register or _i_ register is pointing to. What this means is that though we got our first drawing without much work since everything defaulted to zero we'll need to:

* set a register to where we want to start printing our second character
* draw our first sprite
* set the address register a second time pointing at our new sprite
* draw the new sprite
* wait for a keystroke and pause so we can actually see what we printed
* go back and fix the last time we set the address because it's moved

Here's what the working solution looks like:

| row | op | op | Description |
|-----|----|----|-------------|
| 0x200 | 61 | 0A | Load 0x0A into register 0x1 |
| 0x202 | A2 | 0E | Set the address register to 0x214 |
| 0x204 | D0 | 05 | Draw from location (x,y) in register x = 0x1, y = 0x0, 5 addressess starting from the location of the address register |
| 0x206 | A2 | 13 | Set the address register to 0x219 |
| 0x208 | D1 | 06 | Draw from location (x,y) in register x = 0x1, y = 0x0, 6 addressess |
| 0x20A | F0 | 0A | Set key press to register 0 (Waits for keypress) |
| 0x20C | 00 | 00 | Halt |
| 0x20E | FF | C3 | Start of A sprite|
| 0x210 | FF | C3 | |
| 0x212 | C3 | 18 | End of A sprite, beginning of + sprite|
| 0x214 | 18 | FF | |
| 0x216 | FF | 18 | |
| 0x218 | 18 |    | End of + sprite |

Let's say we aren't quite finished though, we want to print another "A" on the display. We'll have to make more changes, and for a second time, go back and change the addresses of where we start reading our sprites.

The finished product now looks like this:

| row | op | op | Description |
|-----|----|----|-------------|
| 0x200 | 61 | 0A | Load 0x0A into register 0x1 |
| 0x202 | 62 | 14 | Load 0x14 into register 0x2 |
| 0x204 | A2 | 14 | Set the address register to 0x214 |
| 0x206 | D0 | 05 | Draw from location (x,y) in register x = 0x1, y = 0x0, 5 addressess starting from the location of the address register |
| 0x208 | A2 | 19 | Set the address register to 0x219 |
| 0x20A | D1 | 06 | Draw from location (x,y) in register x = 0x1, y = 0x0, 6 addressess |
| 0x20C | A2 | 14 | set address register to 0x214 |
| 0x20E | D2 | 05 | Draw from location (x,y) in register x = 0x2, y = 0x0, 6 addressess |
| 0x210 | F0 | 0A | Set key press to register 0 (Waits for keypress) |
| 0x212 | 00 | 00 | Halt |
| 0x214 | FF | C3 | Start of A sprite|
| 0x216 | FF | C3 | |
| 0x218 | C3 | 18 | End of A sprite, beginning of + sprite|
| 0x21A | 18 | FF | |
| 0x21C | FF | 18 | |
| 0x21E | 18 |    | End of + sprite |

### How to avoid rewriting everything

What this example progarm above illustrates is that we really need to plan things ahead otherwise we're going to end up needing to rewrite code. So what's that solution to all of this? Because this is a small program, we could probably get away with having more space in between where our sprites are stored and where we execute our program.

| row | op | op | Description |
|-----|----|----|-------------|
| 0x200 | 61 | 0A | Load 0x0A into register 0x1 |
| 0x202 | 62 | 14 | Load 0x14 into register 0x2 |
| 0x204 | A5 | 00 | Set the address register to 0x500 |
| 0x206 | D0 | 05 | Draw from location (x,y) in register x = 0x1, y = 0x0, 5 addressess starting from the location of the address register |
| 0x208 | A5 | 05 | Set the address register to 0x505 |
| 0x20A | D1 | 06 | Draw from location (x,y) in register x = 0x1, y = 0x0, 6 addressess |
| 0x20C | A5 | 00 | set address register to 0x500 |
| 0x20E | D2 | 05 | Draw from location (x,y) in register x = 0x2, y = 0x0, 6 addressess |
| 0x210 | F0 | 0A | Set key press to register 0 (Waits for keypress) |
| 0x212 | 00 | 00 | Halt |
| ..... | .. | .. | .... |
| 0x500 | FF | C3 | Start of A sprite|
| 0x502 | FF | C3 | |
| 0x504 | C3 | 18 | End of A sprite, beginning of + sprite|
| 0x506 | 18 | FF | |
| 0x508 | FF | 18 | |
| 0x50A | 18 |    | End of + sprite |

 One thing you might notice about hexeditors is that they default to overwrite instead of insert mode. This default isn't very helpful in most situations but, it's perfect for editing hexcode, it makes sense that we don't want other code to be moved around too much as we require that our sprites or future method calls stay in place.

 After moving our sprites down to location `0x500`, we now have a little more room to work with, so we can add some extra operations in between before we need to worry. The caveat is that in a larger one we'd need to be more efficient with our space as our memory constraints are a lot lower than more modern machines.

### Adding a debugger

After overcoming a few actual bugs within the emulator itself, I ran into some bugs with my code much like the ones above. Unfortunately for us, there are very few tools to help us debug. There's no CHIP8 IDE that will let us know that we didn't load anything into the register we're going to read from, or that we're going to read from the wrong place in memory. Because this is an emulator, we're no longer limited by the orginal technology of CHIP8, we can add our own debugging tools. To help overcome these challenges, I added a debugger which would halt the execution cycle, waiting for enter to be pressed, exiting debug mode, or exit the emulator itself which looks like this:

```Rust
/// Loop until a valid key is pressed
fn wait_on_debug_input(&mut self) {
    let mut key_pressed = false;
    while !key_pressed {
        self.window.update();
        if let Some(keys) = self.window.get_keys_pressed(KeyRepeat::No) {
            for t in keys {
                match t {
                    Key::Enter => { key_pressed = true },
                    Key::Escape => { std::process::exit(0) }, // Kills the execution and returns 0
                    Key::Delete => {
                        key_pressed = true;
                        self.debug = false; // removes us from the debug cycle
                    },
                    _ => {}
                }
            }
        }
    }
}
```

![debuggif](https://github.com/adamrmelnyk/chip_8_emulator/raw/master/examples/debug.gif)

At each press of the enter key, I either load a register, draw, set the address register, or any other operation. This isn't perfect but it will go a long way to understanding what's going on, one step at a time.

### What's next

There's a lot left to be desired here in terms of what tools we can offer the CHIP8 programmer, for one, we aren't displaying the op codes, nor are we showing any additional information about state such as memory and register values. Adding a second window might help to display some of this, though it is a little strange to display that separately. We should also add the concept of state so we can replay operations. We probably won't ever be able to recreate the necessary tools that make modern programming ergonomic, but it can definitely be improved.
