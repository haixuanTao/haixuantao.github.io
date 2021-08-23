# Scraping Python vs Rust

## Error handling

Web scraping is about as error-prone as you can imagine. Pages might not exist, HTML elements might not always be there… And so, a language that can support errors and edge cases well at runtime and not crash is a huge plus.

## Performance

Performance test of scraping the 50 pages of [http://books.toscrape.com/catalogue/page-1.html](http://books.toscrape.com/catalogue/page-2.html)

|Name |CPU Usage |Time\(s\) |
| --- | --- | --- |
|Synchronous Python |5% |44.363s |
|Synchronous Rust |7% |55s |
|Async Python |63% |2.5s |
|Async Rust |107% |2.25s |

‌

Performance are pretty similar due to the fact that web scraping is pretty much io bound.
