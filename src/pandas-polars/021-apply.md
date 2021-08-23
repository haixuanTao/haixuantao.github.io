## Apply

### Applying Function in Polars

To Apply a function in Polars, you can use the default `apply` or `may_apply`. I prefer the latter. This will mutate the original data.

```rust
fn str_to_date(dates: &Series) -> std::result::Result<Series, PolarsError> {
    let fmt = Some("%m/%d/%Y %H:%M:%S");

    Ok(dates.utf8()?.as_date64(fmt)?.into_series())
}

fn count_words(dates: &Series) -> std::result::Result<Series, PolarsError> {
    Ok(dates
	.utf8()?
	.into_iter()
	.map(|opt_name: Option<&str>| 
		 opt_name.map(|name: &str| name.split(" ").count() as u64
	))
	.collect::<UInt64Chunked>()
	.into_series())
}

// ...

    // Apply Format Date
    df.may_apply("PostCreationDate", str_to_date)?;

    let t_formatting = Instant::now();

    // Apply Custom counting words in string
    df.may_apply("BodyMarkdown", count_words)?;
```

### Applying Function in Native Rust

Now, what I like about native rust mutation, is that the syntax is standard among iterator, and so once you get comfortable with the syntax, you can apply it everywhere 😀

```rust
use chrono::{DateTime, NaiveDate, NaiveDateTime, NaiveTime};
// use rayon::prelude::*;  for multithreads

    // Apply Format Date
    let fmt = "%m/%d/%Y %H:%M:%S";

    records
	.iter_mut()  // .par_iter_mut() for multithreads
	.for_each(|record: &mut utils::NativeDataFrame| {
	    record.PostCreationDatetime =
		match DateTime::parse_from_str(
		  record.PostCreationDate.as_ref().unwrap(), fmt) {
		    Ok(dates) => Some(dates),
		    Err(_) => None,
		}
	});

    // Apply Custom Formatting counting words in string
    records
	.iter_mut() // .par_iter_mut() for multithreads
	.for_each(|record: &mut utils::NativeDataFrame| {
	    record.CountWords =
		Some(
	  record.BodyMarkdown.as_ref().unwrap().split(' ').count() as f64
		)
	});
```

### Performance for formatting dates

| |Time\(s\) |Speedup Pandas |
| --- | --- | --- |
|Native Rust \(Single thread\) |.98 s |8x |
|**Native Rust \(Multithread\)** |**.148 s** |**52x** |
|Polars\(Single thread\) |.88 s |8.8x |
|Pandas |7.8 s | |

### Performance for counting words

| |Time\(s\) |Speedup Pandas |
| --- | --- | --- |
|Native Rust \(Single thread\) |9 s |2.7x |
|**Native Rust \(Multithread\)** |**1.3 s** |**19x** |
|Polars\(Single thread\) |9 s |2.7x |
|Pandas |24.8 s | |

**Polars** does not seem to offer increased performance over the standard library on a single thread, and I couldn’t find a way to do multi-threaded apply… In this scenario, I’ll prefer native rust.

