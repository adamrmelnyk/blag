---
title: "Building A Basic File Scanning Tool With Rust"
date: 2024-07-27T19:36:23-05:00
description: "Building a signature scanning antivirus function with rust"
draft: false
toc: false
categories: ["technology"]
tags: ["programming", "rust", "tutorial"]
---

### Bare bones scanning

Early AV scanning relied mostly on signature scanning. Mostly looking for string patterns in a file or something as simple as just hashing a file and comparing it against a set of known hashes. For our first pass, we're going to do exactly that. We're going to load up a list of hashes we know to be bad and then we're going to check if a given file matches something in our list of hashes. Easy.

First we'll make our test sample, which we'll do by just creating a file called sample.txt. which contains only the text: _"Hello There"_

Next we'll create the code for the scanner:

```rust
use std::env;
use std::collections::HashSet;

fn main() {
    println!("############### Running scan! ###############");
    let args: Vec<String> = env::args().collect();
    let file_path = &args[1];

    let bytes = std::fs::read(file_path).unwrap();
    let file_hash = sha256::digest(&bytes);

    let mut preset_hashes = HashSet::new();

    // The Sha256 of our "Hello There" file
    preset_hashes.insert("00688350913f2f292943a274b57019d58889eda272370af261c84e78e204743c".to_string());
    
    if preset_hashes.contains(&file_hash) {
        println!("We found a match!")
    }
}
```

The code is pretty simple so far. We have one single hash. which we're adding to a set of preset hashes. And then we check to see if the file contains the same hash. This isn't exactly what we should be doing by any means, it's more just to illustrate one very simple way to detect malware. We should now be able to compile the rust code with cargo and test it against our sample file.

#### Testing our scanner

```shell
$ ./target/release/terriblescanner sample.txt
############### Running scan! ###############
We found a match!
```

It works! Well, it works for this one file anyway. If we had a much larger list of hashes we'd want to set this code up a little better, however we're not going to do that.

### Using Yara rules

You may have already guessed but, there's a problem with the approach we used above. By simply adding a character after "Hello there" the hash changes, and the file will evade our scan. It's too basic for even our very basic tutorial. To make this more effective, we need to use a tool like [Yara](https://en.wikipedia.org/wiki/YARA). Yara is a tool for signature scanning that will let us do a lot more than simple hashing. Yara will let us write more complicated rules to identify potentially unwanted files.

#### Installing yara

We'll need to install yara first before we can use the libraries. For our uses we'll need to use the latest version as well since it's required by the library we're using. At the time of writing, I used:

* yara 4.5.1
* yara-rust 0.28.0 (https://github.com/Hugal31/yara-rust)

#### Install

Run the following commands in a terminal:

1. sudo apt update
1. sudo apt install build-essential automake libtool make pkg-config
1. wget https://github.com/VirusTotal/yara/archive/refs/tags/v4.5.1.tar.gz
1. tar -xzf v4.5.1.tar.gz
1. cd yara-4.5.1/
1. ./bootstrap.sh
1. ./configure
1. make
1. sudo make install
1. sudo ldconfig
1. yara \-\-version

If the last step gives you the matching version number: 4.5.1 then you've successfully installed yara.

***Note that these installation instructions may not work for you depending on your setup***

Before we do any compiling, we're also going to have to set an environment variable which will be needed when we try to compile our rust binary:

```shell
export YARA_LIBRARY_PATH=/usr/local/lib/libyara.so
```

The above line *should* work however, you may want to double check that is the correct location of libyara.so which is needed for compilation.

***It absolutely will not work without this step!*** so make sure you do this.

#### Writing yara rules

First we'll need to write a simple rule to test against. Our first rule will be looking for the String "Hello There", anywhere in the file. We'll call this rule GeneralKenobi.

```text
rule GeneralKenobi
{
    strings:
        $hello_there = "Hello There"
        
    condition:
        $hello_there
}
```

Simply explained this rule is triggered by a single condition, when the string $hello_there is found in the file.

#### Writing our scanner using yara

Take a look at [the example code](https://github.com/Hugal31/yara-rust) and we can use almost exactly the same code for our scanner. The code should now look something like this:

```rust
use std::env;
use yara::Compiler;

const DEFAULT_YARA_FILE: &str = "rules.yara";
const DEFAULT_TIMEOUT: i32 = 10;

fn scan_file(file_path: &str) {
    println!("We are scanning file {file_path}");
    let compiler = Compiler::new().unwrap();
    let compiler = compiler
        .add_rules_file(DEFAULT_YARA_FILE)
        .expect("Adds rules file");

    let rules = compiler
        .compile_rules()
        .expect("Compiled rules");

    let results = rules
        .scan_file(file_path, DEFAULT_TIMEOUT)
        .expect("Scanned file");
    
    for r in results.iter() {
        println!("{0}", r.identifier);
    }
}

fn main() {
    println!("############### Running scan! ###############");
    let args: Vec<String> = env::args().collect();
    let file_path = &args[1];
    scan_file(file_path)
}
```

We probably don't want to be writing our own rules. One of the advantages of using yara is that it is widely adopted so we can use rules that detect files others have seen in the wild. We can easily modify our code so that we can import more than one yara rules file. For the purposes of this tutorial, I have used [this set of yara rules](https://raw.githubusercontent.com/HydraDragonAntivirus/HydraDragonAntivirus/main/yara/comment_lines.yar) which is quite a large rule set.

because we could be adding many more rule sets, we're going to add an array of rule files and iterate over our list of rules files. The final code should look like this:


```rust
use std::env;
use yara::Compiler;

const DEFAULT_YARA_FILE: &str = "rules.yara";
const OTHER_YARA_FILE: &str = "otherrules.yara";
const DEFAULT_TIMEOUT: i32 = 10;

const RULES_FILES: &[&str] = &[
    DEFAULT_YARA_FILE,
    OTHER_YARA_FILE
];

fn scan_file(file_path: &str) {
    println!("We are scanning file {file_path}");
    let mut compiler = Compiler::new().unwrap();

    for &rule_file in RULES_FILES {
        compiler = compiler
            .add_rules_file(&rule_file)
            .expect("Adds rules file");
    }

    let rules = compiler
        .compile_rules()
        .expect("Compiled rules");

    let results = rules
        .scan_file(file_path, DEFAULT_TIMEOUT)
        .expect("Scanned file");
    
    for r in results.iter() {
        println!("Rule Triggered: {0}", r.identifier);
    }
}

fn main() {
    println!("############### Running scan! ###############");
    let args: Vec<String> = env::args().collect();
    let file_path = &args[1];
    scan_file(file_path)
}
```

Both sets of  signatures should be available now so we can test it once again and see if the custom rule we added is still triggered by the sample file.

#### Testing the scan

We want to test that our new rules work for the file we changed. So any changes to the file so long as it still contains the "Hello There" text should work.

```text
Hello There$$$$
```

```shell
$ ./target/release/basicAv sample.txt
############### Running scan! ###############
We are scanning file sample.txt
Rule Triggered GeneralKenobi
```

And there we have it! We built a very simple av scanning function that can check if a single file triggers a predetermined set of yara rules.

### Further reading

* Info on writing Yara rules https://github.com/VirusTotal/yara/blob/master/docs/writingrules.rst