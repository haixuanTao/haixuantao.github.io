# Asynchronous

During scraping, most of the time is lost downloading file rather than computing.

However, with synchronous runtimes, pages are scraped one by one and so downloaded one by one. Each download can take time and idle the whole process. Therefore, if we can manage to not wait for the completion of each download, we will gain efficiency.

### Python
It is possible using the “asyncio” library, and it might look like that:

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

### Rust

Rust, on the contrary to Python, has been built with asynchronous computation in mind. It is thread-safe and extremely efficient. The fact that the language, in its nature. is super fast makes it great for coroutines. The code might look like that:

```rust,noplaypen,editable
use csv::Writer;
use select::document::Document;
use select::predicate::{Attr, Name};
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