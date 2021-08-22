# Pandas vs Polars 

## Introduction

Everyone loves the API of Pandas. It’s fast, easy, and well documented. There are some rough edges, but most times, it’s just a blast.

Now, when it comes to production, Pandas is slightly trickier. Pandas does not scale very well… there is no multithreading… It’s not thread-safe… It’s not memory efficient.

But all those problems are the raison d’être of Rust.

**But what if, there was a DataFrame API written in Rust that solves all those issues and at the same time keeps a nice API?**

---

## Polars

Well, [**Polars**](https://github.com/ritchie46/polars) tries to do just that. It allows you to do read, write, filter, apply functions, group by and merge, all in a **thread-safe fashion**.

It uses [**Apache Arrow**](https://github.com/apache/arrow), a data framework purposely built for doing efficient data processing and data sharing across language.

## 3 reasons for choosing **Polars**

### Reason #1. Performance.

it’s killing it [performance-wise](https://h2oai.github.io/db-benchmark/).

### Reason #2. The API is straightforward.

Do you want to mutate the data? Use `apply`. Do you want to filter the data? use `filter`. Do you want to merge? Use `join` . There is not going to be rust syntax like `struct`, `derive`, `impl` …

### Reason #3. No troubles with the borrow checker.

It uses Arc, Mutex like referencing, which means that you can clone variables as much as you like. Variables are only references to in-memory data. No more fighting with the borrow checker. Mutability is limited to the API calls, which preserve the consistency/thread-safety of the data.

## 3 caveats of **Polars**

### ‌‌Caveat #1. Issues…

Building a DataFrame API is hard. It’s so hard, Pandas took 12 years to reach 1.0.0. And,  as Polars is rather young, you may face unexpected issues. In my cases, there were issues with [\\n characters](https://github.com/ritchie46/polars/issues/387),[ double quotes characters](https://github.com/ritchie46/polars/pull/399), and [long utf8](https://github.com/ritchie46/polars/pull/400).

On the other hand, those are great first issues to get started with contribution and getting better at Rust 🔨

### Caveat #2. Getting comfortable with two APIs: Polars and Arrow.

As many of the heavy liftings are done using the Apache Arrow backend, you’ll have to get used to reading both documentation. Both documentations are pretty straightforward, but it might feel tiring for someone who was looking for a drop-in replacement of Pandas.

### Caveat #3. Compiling time…

Sadly, compiling time takes around [6min](https://github.com/ritchie46/polars/issues/402) uncached. And, it uses a lot of resources.

---

## Case Study

**Now the question is, is it better than native Rust as I’ve explained** [**in my previous blog post**](https://able.bio/haixuanTao/data-manipulation-pandas-vs-rust--1d70e7fc)**?**

Let’s take a hands-on comparison for a Data Pipeline and get a feel for it.

In this case study, I’m going to use the [stack overflow kaggle dataset](https://www.kaggle.com/c/predict-closed-questions-on-stack-overflow/data?select=2012-07+Stack+Overflow.7z). I’m going to read the database, parse the dates, make a merge between the first tag and the[ Wikipedia comparison of programming language](https://en.wikipedia.org/wiki/Comparison_of_programming_languages#Failsafe_I/O_and_system_calls). Group by the status of the question asked. And retrieve the distribution of language’s features within each ‘status’ of questions.

We’ll compare Polars API & Native Rust generic heap structure to do this task.

* _I’ll go slightly quicker on the native Rust, as I already put more details_ [_here_](https://able.bio/haixuanTao/data-manipulation-pandas-vs-rust--1d70e7fc)_._
* _Multithreading is done on 12 threads Intel\(R\) Core\(TM\) i7-8750H / 20G RAM._
* _The database is 4.2G big for around 3.6 Million rows._

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

## Applying functions

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

```
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
|Native Rust \(Multithread\) |.148 s |52x |
|Polars\(Single thread\) |.88 s |8.8x |
|Pandas |7.8 s | |

### Performance for counting words

| |Time\(s\) |Speedup Pandas |
| --- | --- | --- |
|Native Rust \(Single thread\) |9 s |2.7x |
|Native Rust \(Multithread\) |1.3 s |19x |
|Polars\(Single thread\) |9 s |2.7x |
|Pandas |24.8 s | |

### Conclusion

**Polars** does not seem to offer increased performance over the standard library on a single thread, and I couldn’t find a way to do multi-threaded apply… In this scenario, I’ll prefer native rust.

## Merging

### Merging in Polars

Merging in Polars is dead easy, although the number of strategy for filling `none` values are limited for now.

```rust
    df = df
        .join(&df_wikipedia, "Tag1", "Language", JoinType::Left)?
        .fill_none(FillNoneStrategy::Min)?;
```

### Merging in Native Rust

Merging in native Rust can be done with nested structure and pairing with a Hashmap:

```rust
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
|Native Rust \(Multithread\) |.215 s |20x |
|Polars |.543 s |8x |
|Pandas |4.347 s | |

### Conclusion

For merging, having a nested structure with `None` values can be very verbose. Having a flat structure is a huge plus. So, I’ll recommend using **Polars** if merging is key.

_I’m not sure If polars merging is done multi-threaded or not. It seems to be multithreaded by default._

## Group By

‌

### Group By in Polars

Group by in polars are really easy

```
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

```
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
|Native Rust \(Multithread\) |.115 s |9.5x |
|Polars\(Single thread\) |.131 s |8.3x |
|Polars\(Multithread\) |.125 s |8.8x |
|Pandas |1.1 s | |

### Conclusion

Group By and Merging are the ideal case for Polars. You’ll get 8x more performance than Pandas on a single thread, and Polars handle multi-threading although, in my case, it didn’t matter much.

It can be done with native rust, but judging by the size of the code, it’s not an ideal use case.

---

### Polars Lazy

Polars also offers a query optimized version called Lazy with a slightly different API. In my use case, I did not find it hard to go from one to the other, but I did not find any significant increase in performance either. The result is in the overall performance.

---

## Overall

### Performance overall

| |Time\(s\) |Speedup Pandas |
| --- | --- | --- |
|Native Rust \(Single thread\) |24 s |3.3x |
|Native Rust \(Multithread\) |13.7 s |5.8x |
|Polars\(Single thread\) |30 s |2.6x |
|Polars\(Multithread\) |33 s |2.4x |
|Polars\(lazy, Multithreaded\) |32s |2.5x |
|Pandas |80 s | |

As reading is io bound, I wanted to make a benchmark of pure performance.

### Performance without Reading

| |Time\(s\) |Speedup Pandas |
| --- | --- | --- |
|Native Rust \(Single thread\) |12 s |3.3x |
|Native Rust \(Multithread\) |1.7 s |23x |
|Polars\(Single thread\) |10 s |4x |
|Polars\(Multithread\) |11 s |3.6x |
|Polars\(Lazy, Multithread\) |11 s |3.6x |
|Pandas |40 s | |

‌

### Overall takeaway

* Use Polars if you want a great API.
* Use Polars for merging and group by.
* Use Polars for [single instruction multiple data\(SIMD\) ](https://en.wikipedia.org/wiki/SIMD)operation.
* Use Native Rust if you’re already familiar with rust generic heap structure like vectors and hashmap.
* Use Native Rust for linear mutation of the data with `map` and `fold`. You’ll get O\(n\) scalability that can be parallelized almost instantly with `rayon`.
* Use pandas when performance, scalability, memory usage does not matter.

For me, both Polars and native Rust makes a lot of sense for data between 1Go and 1To, single-threaded or not.

I’ll invite you to make your own opinion. The code is available here: [https://github.com/haixuanTao/dataframe-python-rust](https://github.com/haixuanTao/dataframe-python-rust)

### _Future writing_

The output of our data pipeline show divergence in distribution between language feature and question status.

This means we may have a signal for doing Machine Learning.

Next Stop, ML in Rust

## ‌Annexe

### Dask

For the life of me, I tried to run **dask** for benchmarking but did not succeed in making it faster than native pandas. On the **dask** website, they say:

_If your dataset fits comfortably into RAM on your laptop, then you may be better off just using Pandas. There may be simpler ways to improve performance than through parallelism_

This means, there is this void in pandas optimization for data sized between 1Go to 1To. Dask seems to be a replacement of Spark but not pandas itself.

And even if it worked, performance increase would only be around 4-5x, from past experience

### Cudf

I tried to run **Cudf** on my 4G RAM GPU but run out of memory. I did not investigate further.

### Vagrind

I tried to run **valgrind** to do a profiling of the memory usage but it seems not to work with polars and native rust at this size.