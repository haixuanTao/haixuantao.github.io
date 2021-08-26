
### Synchronous Python code

```python
import requests
import bs4 as bs
import csv
URL = "http://books.toscrape.com/catalogue/page-%d.html"

with open('./test_python.csv', 'w') as csvfile:
    spamwriter = csv.writer(csvfile, delimiter=',')
    for i in range(1, 50):
        response = requests.get(URL % i)
        if response.status_code == 200:
            content = response.content
            soup = bs.BeautifulSoup(content, 'lxml')
            articles = soup.find_all('article')

            for article in articles:
                information = []
                information.append(article.find(
                    'p', class_='price_color').text)
                information.append(article.find('h3').find('a').get('title'))
                spamwriter.writerow(information)
```

‌

### Synchronous Rust code:

```rust,noplaygen
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

        // println!("{:#?} ", url);
        wtr.write_record(&[&url, &price, &name]).unwrap();
    }

    Ok(())
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    for i in 1..50 {
        test(&i).await.unwrap();
    }
    Ok(())
}
```