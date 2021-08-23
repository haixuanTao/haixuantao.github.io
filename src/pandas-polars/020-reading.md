## Reading

### Reading in Polars

Reading is pretty straightforward with many configurations possible.

```rust
use polars::prelude::*;

//...

    let mut df = CsvReader::from_path(path)?
        .with_n_threads(Some(1)) // comment for multithreading
        .with_encoding(CsvEncoding::LossyUtf8)
        .has_header(true)
        .finish()?;
```

### Reading in Native Rust

Reading in Rust using csv and serde requires that you already have a `struct` , in my case my struct is `utils::NativeDataFrame`

```rust
    let file = File::open(path)?;

    let mut rdr = csv::ReaderBuilder::new().delimiter(b',').from_reader(file);
    let mut records: Vec<utils::NativeDataFrame> = rdr
        .deserialize()
        .into_iter()
        .filter_map(|result| match result {
            Ok(rec) => rec,
            Err(e) => None,
        })
        .collect();
```

### Performance

| |Time\(s\) |Speedup Pandas |
| --- | --- | --- |
|Native Rust \(Single thread\) |12 s |2.4x |
|Polars\(Single thread\) |19 s |1.5x |
|Polars\(Multithread\) |22 s |1.3x |
|Pandas |29.6 s | |

### Conclusion

For reading, **Polars** has a nice happy and I bet it’s also doing some indexing which explains the difference in timing between the native implementation. There seems to be a bug for multithreaded Polars that makes it slower than single-threaded. _\(Probably a good first issue…_ 🤪 _\)_

Note that reading is bound by the reading speed of my SSD around ~300Mb/s.

