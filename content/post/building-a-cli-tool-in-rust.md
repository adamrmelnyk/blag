---
title: "Building a Cli Tool in Rust"
date: 2020-05-02T11:20:54-05:00
description: "A short tutorial on building a useful command line tool in Rust"
draft: false
toc: false
categories: ["technology"]
tags: ["Rust", "Cli", "sonos"]
---

Like almost everyone I know, I’ve been ~~sitting~~ pacing inside for almost 8 weeks, waiting out the sars-cov-2 pandemic. I’ve been “zooming” with friends, phoning my family members, and catching up on reading. Gone are the days of concerts and movie theaters. Now is the time for reading, Animal Crossing, and updating one’s websites. So here it is, a new theme for a new year and a new post to help stave off boredom. Now’s the perfect time to clean up my dirty markdown peppered with HTML that I’ve been meaning to remove, update those pesky dependencies that github keeps reminding me are out of date, and it’s the perfect opportunity to work on something new and exciting.

Earlier this week a reddit user posted their library [sonor](https://docs.rs/sonor/0.1.2/sonor/) for interacting with sonos devices. Having a penchant for anything I can use from the command line, this seemed like the perfect opportunity to build a command line application to use my sonos speakers. Though I have not used Rust very much other than reading through the book and tinkering with a few different frameworks, this seemed to fit neatly into everything I wanted to do:

* Use Rust for something novel
* Build something I might actually use
* Pass the time during a pandemic

## Building the Tool

### Setting up the project

Rust has some great tooling that is really easy to use and setup. Once you have Rust installed you can use cargo to start a new project like so:

```shell
cargo new sonos_tool
```

Running it is just one more step, just `cd` into the directory and then use `cargo run`

```shell
cargo run
   Compiling sonos_tool v0.1.0 (/home/arm/src/sonos_tool)
    Finished dev [unoptimized + debuginfo] target(s) in 6.61s
     Running `target/debug/sonos_tool`
Hello, world!
```

### Adding dependencies

Before going further we’ll need to add a few dependencies to our `Cargo.toml` file:

```text
[dependencies]
sonor = "0.1.2"
tokio = { version = "0.2", features = ["macros", "time"]}
futures = "0.3"
structopt = { version = "0.3", default-features = false }
```

### Running a single command from sonor

The first thing I want to do is the most basic thing I can find with sonor. The sonor repo has some examples so we’re going to start a little smaller and then expand upon it a little bit. Looking at the example the first thing we can do is find a speaker and return the name. So we can replace the main method with something like this:

```Rust
#[tokio::main]
async fn main() -> Result<(), sonor::Error> {
    let speaker = sonor::find("Bedroom", Duration::from_secs(2)).await?
    .expect("room exists");
    println!("Name: {}", speaker.name().await?);
    return Ok(())
}
```

In this example, we have a speaker, or speaker set in our home called `"Bedroom"` but you’ll need to use whatever name your speaker might be called.

Running this should produce something like this:

```shell
    Finished dev [unoptimized + debuginfo] target(s) in 5.88s
     Running `target/debug/sonos_tool`
Name: Bedroom
```

The library makes working with your sonos speakers easy as pie, we just need add more functions, and then make it usable as a command line tool.

### Adding functionality

Now that we can find the speaker we can use any of these commands at our disposal. All we need to do is make the commands available from the command line somehow and match them up with the appropriate speaker function.

Unless you want to dig into the docs to see what else we can do, the `speaker.rs` file has all the commands we can access and some good examples on how one might use what’s available. To start with we should add some basic functionality that we might need: `Play`, `Pause`, and because we’re going to need to know the names of what’s available, an `Info` command used for discovery.

To make things easier on ourselves, the first thing we can do is take a look at the [`structop` crate](https://docs.rs/structopt/0.3.14/structopt/) which will be helpful for listing the available commands.

```Rust
#[derive(StructOpt)]
enum Command {
    Info,
    Play,
    Pause,
}
```

We’ll also add this to our main method though we don’t need to do anything with the cmd variable for now

```Rust
let cmd = Command::from_args();
```

Now we can run `cargo build` and this time we’ll execute our binary like so

```shell
$ ./target/debug/sonos_tool --help
Name: Bedroom
sonos_tool 0.1.0

USAGE:
    sonos_tool <SUBCOMMAND>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

SUBCOMMANDS:
    help     Prints this message or the help of the given subcommand(s)
    info
    pause
    play
```

Without needing to do anything else at all, Structop gives us a help command (we can use `-h` as well), as well as a `--version` command, and some helpful info.

### Adding some better documentation to the command line tool

We aren’t making use of everything structop gives us however. Looking at the docs we can add the annotations which will build out the rest of this info for us.

```Rust
#[derive(StructOpt)]
enum Cli {
    Info,
    #[structopt(
        about = "Plays specified room",
        help = "USAGE: play MyRoomName"
    )]
    Play,
    #[structopt(
        about = "Pauses a specified room",
        help = "USAGE: volume MyRoomName"
    )]
    Pause,
}
```

Now that we’ve added the about and help data, using `-h` gives us something much more useful.

```shell
   Compiling sonos_tool v0.1.0 (/home/arm/src/sonos_tool)
    Finished dev [unoptimized + debuginfo] target(s) in 6.61s
     Running `target/debug/sonos_tool`
sonos_tool 0.1.0

USAGE:
    sonos_tool <SUBCOMMAND>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

SUBCOMMANDS:
    help          Prints this message or the help of the given subcommand(s)
    info          Displays info about all rooms
    pause         Pauses a specified room
    play          Plays specified room
```

There’s one last thing we need in our structop enum, as we saw with the first example, we’ll need a name to specify which speaker or group we’re talking about. This is just as easy as Rust enums are not normal enums and can have properties like this:

```Rust
#[derive(StructOpt)]
enum Command {
    Info,
    #[structopt(
        about = "Plays specified room",
        help = "USAGE: play MyRoomName"
    )]
    Play {
        name: String,
    },
    #[structopt(
        about = "Pauses a specified room",
        help = "USAGE: volume MyRoomName"
    )]
    Pause {
        name: String,
    },
}
```

### Making use of the commands

Now that we have the commands laid out we just need to parse our command line arguments and then implement the three functions. For our functions we can implement them similarly to the first method where we printed our speakers name however, this time we’ll need to provide a param to specify which speaker we will be sending the command to.

```Rust
fn play(name: String) -> Result<(), sonor::Error> {
    let speaker = sonor::find(&name, Duration::from_secs(2)).await?
        .expect("room exists");
    return speaker.play().await;
}

fn pause(name: String) -> Result<(), sonor::Error> {
    let speaker = sonor::find(&name, Duration::from_secs(2)).await?
        .expect("room exists");
    return speaker.pause().await;
}
```

Trying to build this however, will return an error:

```shell
error[E0728]: `await` is only allowed inside `async` functions and blocks
   --> src/main.rs:121:19
    |
120 | fn pause(name: String) -> Result<(), sonor::Error> {
    |    ----- this is not `async`
121 |     let speaker = sonor::find(&name, Duration::from_secs(2)).await?
    |                   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ only allowed inside `async` functions and blocks

error[E0728]: `await` is only allowed inside `async` functions and blocks
   --> src/main.rs:123:12
    |
120 | fn pause(name: String) -> Result<(), sonor::Error> {
    |    ----- this is not `async`
...
123 |     return speaker.pause().await;
    |            ^^^^^^^^^^^^^^^^^^^^^ only allowed inside `async` functions and blocks
```

Rust is laying out exactly what we need to make this work. The function needs to be `async` just like the main method. It’s an easy fix and all we need is to do is change our method to:

```Rust
async fn play(name: String) -> Result<(), sonor::Error> {
    let speaker = sonor::find(&name, Duration::from_secs(2)).await?
        .expect("room exists");
    return speaker.play().await;
}

async fn pause(name: String) -> Result<(), sonor::Error> {
    let speaker = sonor::find(&name, Duration::from_secs(2)).await?
        .expect("room exists");
    return speaker.pause().await;
}
```

Our info command will take a little more work. This function won’t be passed a name of a speaker like the other two functions, but rather, we’ll use it to find all our speakers on our wifi network and display some general info about the state of them.

```Rust
async fn info() -> Result<(), sonor::Error> {
    let mut devices = sonor::discover(Duration::from_secs(2)).await?;

    while let Some(device) = devices.try_next().await? {
        let name = device.name().await?;
        let speaker = sonor::find(&name, Duration::from_secs(2)).await?
            .expect("room exists");
        match speaker.track().await? {
            Some(track_info) => {
                println!("Room: {}", name);
                println!("Volume: {}", speaker.volume().await?);
                println!("Track: {}", track_info.track());
            }
            None => {
                println!("Room: {}", name);
                println!("Volume: {}", speaker.volume().await?);
            }
        }
        println!("----------");
    }

    Ok(())
}
```

The main difference here is we need to use `sonor::discover` to find our speakers before we can do anything, the rest is fairly self explanatory. For each device we find we want to print it’s name, the volume, and if it’s playing, display the track.

### Putting it all together

Now that we have our methods we just need to call set them up in our main method

```Rust
#[tokio::main]
async fn main() -> Result<(), sonor::Error> {
    let args = Command::from_args();
    return match args {
        Command::Info => info().await,
        Command::Play { name } => play(name).await,
        Command::Pause { name } => pause(name).await,
    }
}
```

The first thing you may notice is how we pass the name that we defined in our enum above, to each `match` arm. The second thing is that each of these async methods, like the methods from sonor are all async and as such will need `.await` to be executed. Without this, nothing will happen. Luckily Rust will prevent us from doing this, as you’ll be able to see if you remove one of them and try to build the project.

### Running the basic program

Now that our barebones tool is finished, we can try it out. Running the info command should give you something like this, depending on what your speakers are named:

```Shell
$ ./target/debug/sonos_tool info
Room: LivingRoom
Volume: 21
----------
Room: Bookshelf
Volume: 23
----------
Room: Kitchen
Volume: 22
----------
Room: Bedroom
Volume: 23
----------
```

Testing the pause and play functions will be a little different because you’ll need to actually turn on your speakers to test that it works but can be executed similarly `$ ./target/debug/sonos_tool play Bedroom`.

### Missing features

This was put together relatively quickly so there’s a lot left that we can add. Looking in the Speaker implementation there are a ton of other features to be added that we can play around with.

#### Better Error handling

All of our functions are using `.expect()` right now. What this means is that when we give them a room that can’t be found, the tool will panic and stop.

```Shell
$ ./target/debug/sonos_tool play asdf
thread 'main' panicked at 'room exists', src/main.rs:152:19
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

It would be much better if we removed this and handled the error gracefully.

#### Missing functionality

There are still a lot of functions that we need to set up, we should be able to set the volume, stop, change our equalization, bass, treble etc. All of these should be relatively trivial to implement but would make this tool much more useful.

#### Integrate with your music services

Right now we can manipulate the speakers themselves but we don’t have a way of getting them to play specific songs or podcasts. We’d need to do that from the app.

#### Tests

The tool might be small right now, but adding anything else might start making it harder to maintain, adding some tests will help us make sure that all of our features keep working as we add to it.

## Concluding Thoughts

Rust has a reputation for having a steep learning curve, but I hope that this post made it seem at least a little more approachable. Though my experience with the language as of writing this is fairly limited, I found this relatively simple to put together, and I hope that you will too. Async features were no more difficult to use than other normal functions. Structop really made making a Cli tool much simpler as well, giving us a simple way to set up a command line tool and expand it as needed.
