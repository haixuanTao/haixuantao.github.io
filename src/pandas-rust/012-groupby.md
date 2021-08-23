## Groupby

### \[Pandas\]

Group by are a big part of the data reduction pipeline in python, it goes usually as follows:

```python
df = df.groupby(by="country_txt", as_index=False).agg(
    {"nkill": "sum", "individual": "mean", "eventid": "count"}
)
df.to_csv("python_output_groupby.csv")
```

### \[Rust\]

For group by and data reduction, thanks to [David Sanders](https://able.bio/insideoutclub), group by can be done as follows:

```rust
use itertools::Itertools;


// ...

#[derive(Debug, Deserialize, Serialize)]
struct GroupBy {
    country: String,
    total_nkill: f64,
    average_individual: f64,
    count: f64,
}

// ... 

    let groups = records
        .into_iter()
        .sorted_unstable_by(|a, b| Ord::cmp(&a.country_txt, &b.country_txt))
        .group_by(|record| record.country_txt.clone())
        .into_iter()
        .map(|(country, group)| {
            let (total_nkill, count, average_individual) = group.into_iter().fold(
                (0., 0., 0.),
                |(total_nkill, count, average_individual), record| {
                    (
                        total_nkill + record.nkill.unwrap_or(0.),
                        count + 1.,
                        average_individual + record.individual.unwrap_or(0.),
                    )
                },
            );
            GroupBy {
                country: country.unwrap(),
                total_nkill,
                average_individual: average_individual / count,
                count,
            }
        })
        .collect::<Vec<_>>();
    let mut wtr =
        csv::Writer::from_path("output_rust_groupby.csv")
            .unwrap();

    for group in &groups {
        wtr.serialize(group)?;
    }
```

‌

Although this solution is not as elegant as Pandas groupby, it gives a lot of flexibility on the computation of the reduced fields. Again, thanks to Closures.

I think more reduction method other than `sum` and `fold` would greatly improve the development experience of map-reduce style operation in rust. We will then probably have equivalent experience between Rust and Pandas.

### Performance

| |Time\(s\) |Mem\(Gb\) |
| --- | --- | --- |
|Pandas |2.78s |2.5Gb |
|**Rust** |**2.0s🔥 -35%** |**1.7Gb🔥 -32%** |

Although the performance is better for Rust, I would advise using **Pandas** for map-reduce heavy application, as it seems more appropriate.
