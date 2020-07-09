---
title: "Scraping Wikipedia With Rust"
date: 2020-06-20T23:15:02-05:00
description: "Building a toy Rust app to scrape data from wikipedia"
draft: false
toc: false
categories: ["programming"]
tags: ["rust", "scraper", "wikipedia"]
---

## Writing a toy scraper

As the quarantine continues and many things are still closed, I'm still spending a lot of time looking for fun side projects I can complete in a few days or less. As I drifted off to sleep one night wondering about some correlations between countries and various other data points. I realized that wikipedia probably had all the data I needed to find out. What if I could just scrape all the data off of wikipedia and throw it into sql tables so I could chart data that way? This sounded like the perfect job for another Rust CLI tool.

## Making a Request

The first thing we need is a library to actually make requests to a webpage. There are more than a few of them, but the one I settled on is [reqwest](https://docs.rs/reqwest/0.10.6/reqwest/), a highly recommended library with good support. You can find plenty of examples on how to use it and it also supports async await.

Using reqwest is as easy as this:

```rust
let body = reqwest::get("https://www.arm64.ca")
    .await?
    .text()
    .await?;

println!("body = {:?}", body);
```

The use case I had for this wasn't too much more complicated than this. I just needed to check that the response code had a successful response and I could continue with scraping the data I needed.

```rust
match get(&url).await {
    Ok(resp) => {
        if resp.status().is_success() {
            match resp.text().await {
                Ok(body) => extract_data(&body),
                Err(_) => {
                    eprintln("Oh noes! What happened to my body!");
                    std::process::exit(1);
                },
            }
        } else {
            // Return some err
        }
    }
    Err(_) => // Return some err,
}status
```

## Scraping data

Once we've made a request to a website we need to actually extract that data. To do so we'll need another library so we don't have to reinvent the wheel. Another library [scraper](https://docs.rs/scraper/0.9.1/scraper/) does the job for us. It allows us to parse html and select particular elements; Exactly what we'll need if we want to extract data from a table on wiki page. One of the useful parts of Scraper is that the `Selector` lets us chain selectors together so we can find elements inside of other elements. This is extremely useful if we want to narrow down our results to exactly what we need.

```rust
use scraper::{Html, Selector};

let html = r#"
    <div>
        <h1>Foo</h1>
        <h1>Bar</h1>
        <h1>Baz</h1>
    </div>
"#;

let fragment = Html::parse_fragment(html);
let div_selector = Selector::parse("div").unwrap();
let h1_selector = Selector::parse("h1").unwrap();

let h1 = fragment.select(&div_selector).next().unwrap();
for element in h1.select(&h1_selector) {
    assert_eq!("h1", element.value().name());
}
```

One disadvantage of using any library is that documentation isn't always perfect. At the time of this writing there's another use case I needed later which was not described in scrapers documentation or README. Selectors can be made to select multiple tags. This turns out to be really useful if you end up finding some heterogeneous data in the html you're parsing as I did. Just like the selector above we put the tag we're looking for into quotes, only this time, we separate all of the tags we're looking for in one string with commas.

```rust
use scraper::{Html, Selector};

let html = r#"
        <div>
            <h1>foo</h1>
            <h2>bar</h2>
            <h3>baz</h3>
        </div>"#;
let fragment = Html::parse_fragment(html);
let selector = Selector::parse("h1,h2,h3").unwrap();

for element in fragment.select(&selector) {
    let name = element.value().name();
    let is_htag = name == "h1" || name == "h2" || name == "h3";
    assert_eq!(true, is_htag);
}
```

## Working with the DB

Now that we can request the data and search it for what we're looking for, the last thing is to store it somewhere. I mentioned before that I'd like to put it in the SQL table so I chose SQLite3 because it's easily available and lightweight. There were a few different libraries to choose from, such as [rusqlite](https://github.com/rusqlite/rusqlite) but I went with [sqlite](https://docs.rs/sqlite/0.24.0/sqlite/) which was simple enough to use. We simply open a connection to the database and then execute a command.

```rust
let connection = sqlite::open("mySqlite3DbFileName.db").unwrap();

connection
    .execute("CREATE TABLE mytable (col1 TEXT, col2 INTEGER);")
    .unwrap();
```

Inserting is essentially the same though the library gives you a few better options if you are doing this more than once.

```rust
let connection = sqlite::open("mySqlite3DbFileName.db").unwrap();

connection
    .execute("INSERT INTO mytable (col1, col2) VALUES ('a', 10);")
    .unwrap();
```

## Putting everything together

### Cleaning the data

Now we have all the tools we need to start retrieving, parsing and storing data. The main work of course is going to be in parsing. We not only need to find the correct table elements we need, but there's also a lot of extra information that needs to be removed such as spans, fonts, and other invisible elements so that we insert data that looks like this:

`1|China|1403191760|18.0|21 Jun 2020|National population clock`

and not:

`<span class="flagicon"><img alt="" src="//upload.wikimedia.org/wikipedia/commons/thumb/f/fa/Flag_of_the_People%27s_Republic_of_China.svg/23px-Flag_of_the_People%27s_Republic_of_China.svg.png" decoding="async" class="thumbborder" srcset="//upload.wikimedia.org/wikipedia/commons/thumb/f/fa/Flag_of_the_People%27s_Republic_of_China.svg/35px-Flag_of_the_People%27s_Republic_of_China.svg.png 1.5x, //upload.wikimedia.org/wikipedia/commons/thumb/f/fa/Flag_of_the_People%27s_Republic_of_China.svg/45px-Flag_of_the_People%27s_Republic_of_China.svg.png 2x" data-file-width="900" data-file-height="600" width="23" height="15"></span>&nbsp;<a href="/wiki/Demographics_of_China" title="Demographics of China">China</a><sup id="cite_ref-4" class="reference"><a href="#cite_note-4">[b]</a></sup>`

Cleaning up this data is pretty tedious and requires a lot of tweaking depending on how each page is styled so I won't include any sort of insite here. You can see what I did in the project [here](https://github.com/adamrmelnyk/wtd/blob/master/src/main.rs#L225).

### Building the CLI tool elements

Because this is meant to be a CLI tool, we'll also need to add those elements such as Structop which will do a lot of the heavy lifting for us. For an example of how to set up a CLI tool, see the previous post [here](/post/building-a-cli-tool-in-rust/)

### Data types

Because we want to build a tool that requires as little work from the user as possible, we also need to determine the data type of each column if we want to put everything in a database table. Luckily, [parse](https://doc.rust-lang.org/std/primitive.str.html#method.parse) comes in handy. We use the turbofish syntax (a fancy name for `::<u32>` or `::<f64>` etc) to specify the type and if we return `Ok()` we can determine what kind of data each element might be.

The finished method looks something like this

```rust
/// Derives the type of the string
fn derive_type(sample_datum: &str) -> SqlTypes {
    let html_cleaned_data = remove_html_tags(sample_datum);
    let removed_citations = remove_wiki_citation_links(&html_cleaned_data);
    let cleaned = clean_integer_or_double_string(&removed_citations);
    if cleaned.parse::<i64>().is_ok() { return SqlTypes::INTEGER }
    if cleaned.parse::<f64>().is_ok() { return SqlTypes::REAL }
    if removed_citations.parse::<bool>().is_ok() { return SqlTypes::NUMERIC }
    SqlTypes::TEXT
}
```

This isn't an exhaustive list of types supported by Sqlite3 but it covers the basics that we might need.

## What's missing

### Dates

Perhaps one of the trickier points left to work on are dates. [Sqlite3 has a few options](https://www.sqlitetutorial.net/sqlite-date/) so we need to decide on what format of date we will use (perhaps a Real and use epoch time?). Because there are so many accepted date formats and many types of date formats a webpage might use. This is a more general problem that is beyond the scope of this small project but there is likely a rust library somewhere that will do some of this for you, much like Javascripts `Date.parse()` method.

### Testing, Testing, Testing

A lot of the work around parsing is going to be in cleaning the data, that means for each new source, we'll likely need to tweak how we clean the data and add more and more testing around this to make sure we don't break backwards compatibility. Over the course of building this tool I realized how inconsistent much of the data was, each page I ran it on required it's own tweaks to keep everything working.

## The finished (alpha) product

You can find the finished product [here](https://github.com/adamrmelnyk/wtd). The project doesn't work on everything and it likely will not work on all but the most simple of wikipedia pages (those with one table and no tables inside of tables) as many pages have different layouts and formatting that require more testing and tweaking to clean and parse the data. The basic ideas however, are all there to give anyone some ideas on how they might want to scrape data from somewhere else.
