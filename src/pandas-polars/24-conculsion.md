## Conclusion

### Performance overall

| |Time\(s\) |Speedup Pandas |
| --- | --- | --- |
|Native Rust \(Single thread\) |24 s |3.3x |
|**Native Rust \(Multithread\)** |**13.7 s** |**5.8x** |
|Polars \(Single thread\) |30 s |2.6x |
|Polars \(Multithread\) |17 s |4.7x |
|Polars \(lazy, Multithreaded\) |16.5 s |4.8x |
|Pandas |80 s | |

As reading is IO bound, I wanted to make a benchmark of pure performance.

### Performance without Reading

| |Time\(s\) |Speedup Pandas |
| --- | --- | --- |
|Native Rust \(Single thread\) |12 s |3.3x |
|**Native Rust \(Multithread\)** |**1.7 s** |**23x** |
|Polars \(Single thread\) |10 s |4x |
|Polars \(Multithread\) |11 s |3.6x |
|Polars \(Lazy, Multithread\) |11 s |3.6x |
|Pandas |40 s | |

‌

### Overall takeaway

* Use Polars if you want a great API.
* Use Polars for merging and group by.
* Use Polars for [single instruction multiple data\(SIMD\) ](https://en.wikipedia.org/wiki/SIMD)operation.
* Use Native Rust if you’re already familiar with rust generic heap structure like vectors and hashmap.
* Use Native Rust for linear mutation of the data with `map` and `fold`. You’ll get O\(n\) scalability that can be parallelized almost instantly with `rayon`.
* Use pandas when performance, scalability, memory usage does not matter.

For me, both Polars and native Rust makes a lot of sense for data between 1Go and 1To.

I’ll invite you to make your own opinion. The code is available here: [https://github.com/haixuanTao/dataframe-python-rust](https://github.com/haixuanTao/dataframe-python-rust)

[<img alt="github" src="https://img.shields.io/badge/dataframe--python--rust-fff?labelColor=000&logo=github" height="20">](https://github.com/haixuantao/dataframe-python-rust)
[![GitHub stars](https://img.shields.io/github/stars/haixuanTao/dataframe-python-rust?style=social&label=Star&maxAge=2592000)](https://github.com/haixuanTao/dataframe-python-rust/)
