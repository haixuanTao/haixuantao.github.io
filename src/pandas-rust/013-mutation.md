## Mutation

### Pandas

There are many ways to do mutation in Pandas, I usually do the following for performance and functional style:

```python
df["computed"] = df["nkill"].map(lambda x: (x - 10) / 2 + x ** 2 / 3)
df.to_csv("python_output_map.csv")
```

### Rust

For mutation, the functional `iter` of Rust really makes this part a walk in the park:

```rust,noplaypen
    records.iter_mut().for_each(|x: &mut DataFrame| {
        let nkill = match &x.nkill {
            Some(nkill) => nkill,
            None => &0.,
        };

        x.computed = Some((nkill - 10.) / 2. + nkill * nkill / 3.);
    });

    let mut wtr = csv::Writer::from_path(
        "output_rust_map.csv",
    )?;
    for record in &records {
        wtr.serialize(record)?;
    }
```

### Performance

| |Time\(s\) |Mem\(Gb\) |
| --- | --- | --- |
|Pandas |12.82s |4.7Gb |
|**Rust** |**1.58s🔥 -87%** |**1.7Gb🔥 -64%** |

This is where the difference really appeared to me. Pandas do not scale for line-by-line custom build lambda functions. Pandas would have been even worst if I had done an operation involving several columns.

**Rust** is way better for line-by-line mutation natively.