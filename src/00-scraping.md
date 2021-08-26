# Scraping Python vs Rust

## Introduction

Web scraping is about as error-prone as you can imagine. Pages might not exist, HTML elements might not always be there… And so, a language that can support errors and edge cases well at runtime and not crash is a huge plus.

## Performance

Performance test of scraping the 50 pages of [http://books.toscrape.com/catalogue/page-1.html](http://books.toscrape.com/catalogue/page-2.html)

|Name |CPU Usage |Time\(s\) |
| --- | --- | --- |
|Synchronous Python |5% |44.3s |
|Synchronous Rust |7% |55s |
|Async Python |63% |2.5s |
|Async Rust |107% |2.25s |

‌
Performances are pretty similar for such low level of requests. Time is consumed downloading. Maybe with significantly more requests, bigger difference would be seen.