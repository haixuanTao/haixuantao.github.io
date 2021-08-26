## Conclusion

After this experience, this is my take away.

* Use Pandas when you can: small CSV\(<1M lines\), simple operation, data cleaning …
* Use Rust when you have: complex operations, memory heavy or time-consuming pipelines, custom functions, scalable software…

That been said, Rust offers impressive flexibility compared to Pandas. Adding the fact that Rust is way more capable of multi-threading than Pandas, I believe that Rust can solve problems Pandas simply cannot.

Additionally, the possibility to run Rust on any platform\(Web, Android, or Embedded\) also create new opportunities for data manipulation in places inconceivable for Pandas and can provide solutions for yet to be resolved challenges.

### Performance

The performance table gives us an insight as to what to expect from Rust. I believe, the speedup can go from **x2** at the minimum and up to **x50** for large data pipelines. The memory use will have an even greater decrease as memory usage accumulates over time with python.

