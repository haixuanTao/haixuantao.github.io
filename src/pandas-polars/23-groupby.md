## Groupby

### Group By in Polars

Group by in polars are really easy

```rust
    // Groupby series as a clone of reference
    let groupby_series = vec![
        df.column("OpenStatus")?.clone(),
    ];

    let target_column = vec![
        "ReputationAtPostCreation",
        "OwnerUndeletedAnswerCountAtPostTime",
        "Imperative",
        "Object-oriented",
        "Functional",
        "Procedural",
        "Generic",
        "Reflective",
        "Event-driven",
    ];

    let groups = df
        .groupby_with_series(groupby_series, false)?
        .select(target_column)
        .mean()?;
```

### Group By in Native Rust

This part is quite tricky. To make a group by in a thread-safe manner, you’ll need to use a Hashmap with `fold`. Note that, [parallel fold](https://docs.rs/rayon/0.7.1/rayon/iter/trait.ParallelIterator.html#method.fold)s are slightly more complicated as folding requires passing data around threads.

```rust
    let groups_hash: HashMap<String, (utils::GroupBy, i16)> = records
        .iter() // .par_iter()
        .fold(
            HashMap::new(), // || HashMap::new()
            |mut hash_group: HashMap<String, (utils::GroupBy, i16)>, record| {
                let group: utils::GroupBy = if let Some(wiki) = &record.Wikipedia {
                    utils::GroupBy {
                        status: record.OpenStatus.as_ref().unwrap().to_string(),
                        ReputationAtPostCreation: record.ReputationAtPostCreation.unwrap(),
                        OwnerUndeletedAnswerCountAtPostTime: record
                            .OwnerUndeletedAnswerCountAtPostTime
                            .unwrap(),
                        Imperative: wiki.Imperative.unwrap(),
                        ObjectOriented: wiki.ObjectOriented.unwrap(),
                        Functional: wiki.Functional.unwrap(),
                        Procedural: wiki.Procedural.unwrap(),
                        Generic: wiki.Generic.unwrap(),
                        Reflective: wiki.Reflective.unwrap(),
                        EventDriven: wiki.EventDriven.unwrap(),
                    }
                } else {
                    utils::GroupBy {
                        status: record.OpenStatus.as_ref().unwrap().to_string(),
                        ReputationAtPostCreation: record.ReputationAtPostCreation.unwrap(),
                        OwnerUndeletedAnswerCountAtPostTime: record
                            .OwnerUndeletedAnswerCountAtPostTime
                            .unwrap(),
                        ..Default::default()
                    }
                };
                if let Some((previous, count)) = hash_group.get_mut(&group.status.to_string()) {
                    *previous = previous.clone() + group;
                    *count += 1;
                } else {
                    hash_group.insert(group.status.to_string(), (group, 1));
                };
                hash_group
            },
        ); // }
           // .reduce(
           //     || HashMap::new(),
           //     |prev, other| {
           //         let set1: HashSet<String> = prev.keys().cloned().collect();
           //         let set2: HashSet<String> = other.keys().cloned().collect();
           //         let unions: HashSet<String> = set1.union(&set2).cloned().collect();
           //         let mut map = HashMap::new();
           //         for key in unions.iter() {
           //             map.insert(
           //                 key.to_string(),
           //                 match (prev.get(key), other.get(key)) {
           //                     (Some((previous, count_prev)), Some((group, count_other))) => {
           //                         (previous.clone() + group.clone(), count_prev + count_other)
           //                     }
           //                     (Some(previous), None) => previous.clone(),
           //                     (None, Some(other)) => other.clone(),
           //                     (None, None) => (utils::GroupBy::new(), 0),
           //                 },
           //             );
           //         }
           //         map
           //     },
           // );

    let groups: Vec<utils::GroupBy> = groups_hash
        .iter()
        .map(|(_, (group, count))| utils::GroupBy {
            status: group.status.to_string(),
            ReputationAtPostCreation: group.ReputationAtPostCreation / count.clone() as f64,
            OwnerUndeletedAnswerCountAtPostTime: group.OwnerUndeletedAnswerCountAtPostTime
                / count.clone() as f64,
            Imperative: group.Imperative / count.clone() as f64,
            ObjectOriented: group.ObjectOriented / count.clone() as f64,
            Functional: group.Functional / count.clone() as f64,
            Procedural: group.Procedural / count.clone() as f64,
            Generic: group.Generic / count.clone() as f64,
            Reflective: group.Reflective / count.clone() as f64,
            EventDriven: group.EventDriven / count.clone() as f64,
        })
        .collect();
```

_Uncomment for multithreading_

### Performance

| |Time\(s\) |Speedup Pandas |
| --- | --- | --- |
|Native Rust \(Single thread\) |.536 s |2x |
|**Native Rust \(Multithread\)** |**.115 s** |**9.5x** |
|Polars\(Single thread\) |.131 s |8.3x |
|Polars\(Multithread\) |.125 s |8.8x |
|Pandas |1.1 s | |

Group By and Merging are the ideal case for **Polars**. You’ll get 8x more performance than Pandas on a single thread, and Polars handle multi-threading although, in my case, it didn’t matter much.

It can be done with native rust, but judging by the size of the code, it’s not an ideal use case.
