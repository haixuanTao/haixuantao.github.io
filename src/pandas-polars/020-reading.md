## Reading

### Reading in Polars

Reading in Polars is pretty straightforward:

```rust,noplaypen
use polars::prelude::*;

//...

    let mut df = CsvReader::from_path(path)?
        .with_n_threads(Some(1)) // comment for multithreading
        .with_encoding(CsvEncoding::LossyUtf8)
        .has_header(true)
        .finish()?;
```

### Reading in Native Rust

Reading in Rust using `csv` and `serde` requires that you already have a `struct`, in my case my struct is `utils::NativeDataFrame`

```rust,noplaypen
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
|**Polars\(Multithread\)** |**6.6 s** |**4.5x** |
|Pandas |29.6 s | |

For reading, **Polars** is faster than Pandas and Native Rust, being able to do it in multithreading.