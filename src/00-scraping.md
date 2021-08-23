# Scraping Python vs Rust

## Error handling

Web scraping is about as error-prone as you can imagine. Pages might not exist, HTML elements might not always be there… And so, a language that can support errors and edge cases well at runtime and not crash is a huge plus.

### \[Python\]

In python, the most common way to handle errors in code would be with an if-statement like so:

```python
response = requests.get(URL)
if response.status_code == 200:
    content = response.content

    soup = bs.BeautifulSoup(content, 'lxml')
    articles = soup.find_all('article')
    if articles:
        ...
```

In itself, handling edge cases with  `if`  is not so bad and pretty natural. But, it has some drawbacks:

* It makes the code harder to read as it is difficult to dissociate business logic and error handling.
* if-statements are most of the time added in retroaction of a bug, which makes coding slower and not fun.
* if-statements error handling logic might vary between packages, which makes coding very tedious.

### \[Rust\]

In Rust, functions return either a success or an error, and, you have to deal with each case before compiling. This makes the code way more robust against errors at runtimes. In practice, the code might look like this:

```rust
let name = match node.find(Name("h3")).next() {
    Some(h3) => h3.text(),
    None => "".to_string(),
};
```

Rust also has other ways to handle Success / Errors Result, such as “?” à la Typescript:

```rust
let response = reqwest::get(&url).await?.text().await?;
```

The drawbacks are that developing in Rust is slower as you have to deal with every edge case during development.

Sometimes, it might even be tricky to understand which edge cases you’re trying to deal with. To be noted, that you HAVE to deal with every edge case in Rust otherwise, Rust will not compile. This is how serious Rust is about error handling.

## Async scraping

During scraping, most of the time is lost downloading file rather than computing.

However, with synchronous runtimes, pages are scraped one by one and so downloaded one by one. Each download can take time and idle the whole process. Therefore, if we can manage to not wait for the completion of each download, we will gain efficiency.

### \[Python\]

Unfortunately, Python does not perform well in asynchronous computing and it is also not thread-safe. But it is possible using the “asyncio” library, and it might look like that:

```python
import asyncio
import requests
import bs4 as bs
import csv

URL = "http://books.toscrape.com/catalogue/page-%d.html"


async def get_book(url, spamwriter):
    response = requests.get(url)
    if response.status_code == 200:
        content = response.content
        soup = bs.BeautifulSoup(content, 'lxml')
        articles = soup.find_all('article')

        for article in articles:
            information = [url]
            information.append(article.find(
                'p', class_='price_color').text)
            information.append(article.find('h3').find('a').get('title'))
            spamwriter.writerow(information)


async def main():
    with open('./test_async_python.csv', 'w') as csvfile:
        spamwriter = csv.writer(csvfile, delimiter=',')
        tasks = []
        for i in range(1, 50):
            tasks.append(asyncio.create_task(
                get_book(URL % i, spamwriter)))

        for task in tasks:
            await task

asyncio.run(main())
```

Python does provide the `async/await` terminology which makes it easier to read and write.

### \[Rust\]

Rust, on the contrary to Python, has been built with asynchronous computation in mind. It is thread-safe and extremely efficient. The fact that the language, in its nature. is super fast makes it great for coroutines. The code might look like that:

```rust
use csv::Writer;
use select::document::Document;
use select::predicate::{Attr, Class, Name};
use std::fs::OpenOptions;

async fn test(i: &i32) -> Result<(), Box<dyn std::error::Error + Send + Sync>> {
    let url = format!("http://books.toscrape.com/catalogue/page-{}.html", i);
    let response = reqwest::get(&url).await?.text().await?;
    let file = OpenOptions::new()
        .write(true)
        .create(true)
        .append(true)
        .open("test2.csv")
        .unwrap();
    let mut wtr = Writer::from_writer(file);

    let document = Document::from(response.as_str());

    for node in document.find(Name("article")) {
        let name = match node.find(Name("h3")).next() {
            Some(h3) => h3.find(Name("a")).next().unwrap().text(),
            None => "".to_string(),
        };
        let price = node
            .find(Attr("class", "price_color"))
            .next()
            .unwrap()
            .text();

        println!("{:#?} ", url);
        wtr.write_record(&[&url, &price, &name]).unwrap();
    }

    Ok(())
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {

    let mut handles: std::vec::Vec<_> = Vec::new();
    for i in 1..50 {
        let job = tokio::spawn(async move { test(&i).await });
        handles.push(job);
    }

    let mut results = Vec::new();
    for job in handles {
        results.push(job.await);
    }

    Ok(())
}
```

‌

## Performance

Performance test of scraping the 50 pages of [http://books.toscrape.com/catalogue/page-1.html](http://books.toscrape.com/catalogue/page-2.html)

|Name |CPU Usage |Time\(s\) |
| --- | --- | --- |
|Synchronous Python |5% |44.363s |
|Synchronous Rust |7% |55s |
|Async Python |63% |2.5s |
|Async Rust |107% |2.25s |

‌

Performance are pretty similar due to the fact that web scraping is pretty much io bound
