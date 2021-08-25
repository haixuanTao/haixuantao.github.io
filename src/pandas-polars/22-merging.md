## Merging

### Merging in Polars

Merging in Polars is dead easy, although the number of strategy for filling `none` values are limited for now.

```rust,noplaypen
    df = df
        .join(&df_wikipedia, "Tag1", "Language", JoinType::Left)?
        .fill_none(FillNoneStrategy::Min)?;
```

### Merging in Native Rust

Merging in native Rust can be done with nested structure and pairing with a Hashmap:

```rust,noplaypen
let mut hash_wikipedia: &HashMap<&String, &utils::WikiDataFrame> = &records_wikipedia
    .iter()
    .map(|record| (record.Language.as_ref().unwrap(), record))
    .collect();

records.iter_mut().for_each(|record| {
    record.Wikipedia = match hash_wikipedia.get(&record.Tag1.as_ref().unwrap()) {
        Some(wikipedia) => Some(wikipedia.clone().clone()),
        None => None,
    }
});
```

### Performance

| |Time\(s\) |Speedup Pandas |
| --- | --- | --- |
|Native Rust \(Single thread\) |.680 s |6.3x |
|**Native Rust \(Multithread\)** |**.215 s** |**20x** |
|Polars |.543 s |8x |
|Pandas |4.347 s | |

For merging, having a nested structure with `None` values can be very verbose. So, I’ll recommend using **Polars** for merging.

_I’m not sure If polars merging is done multi-threaded or not. It seems to be multithreaded by default._
