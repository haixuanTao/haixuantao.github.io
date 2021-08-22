# Pandas vs Rust
## Introduction

Pandas is the main Data analysis package of Python. For many reasons, Native Python has very poor performance on data analysis without vectorizing with NumPy and the likes. And historically, Pandas has been created by Wes McKinney to package those optimisations in a nice API to facilitate data analysis in Python.

This, however, is not necessary for Rust. Rust has great data performance natively. This is why Rust doesn’t really need a package like Pandas.

I believe the rustiest way to do Data Manipulation in Rust would be to build a **heap of data struct**.

_But, I could be wrong, let me know if that’s the case._

This is my experience and reasoning comparing **Pandas vs Rust**.

### Data

Performance benchmarks are done on this very random dataset: [https://www.kaggle.com/START-UMD/gtd](https://www.kaggle.com/START-UMD/gtd) that offers around 160,000 lines / 130 columns for a total size of 150Mb. The size of this dataset corresponds to the type of dataset I regularly encounter, that’s why I chose this one. It isn’t the biggest dataset in the world, and, more studies should probably be done on a larger dataset.

The merge will be done with another random dataset: [https://datacatalog.worldbank.org/dataset/world-development-indicators](https://datacatalog.worldbank.org/dataset/world-development-indicators), the `WDICountry.csv`

---

## 1. Reading and instantiating Data

### \[Pandas\]

Reading and instantiating Data in Pandas is pretty straightforward, and handles by default many data quality problems:

```
import pandas as pd

path = "/home/peter/Documents/TEST/RUST/terrorism/src/globalterrorismdb_0718dist.csv"
df = pd.read_csv(path)
```

### \[Rust\] Reading CSV

For Rust, Managing bad quality data is very very tedious. In this dataset, some fields are empty, some lines are badly formatted, and some are not UTF-8 encoded.

To open the CSV, I used the `csv` crate but it does not solve all the issues listed above. With well-formatted data, reading can be done like so:

```
let path = "/home/peter/Documents/TEST/RUST/terrorism/src/foo.csv"
let mut rdr = csv::Reader::from_path(path).unwrap();
```

But with bad quality formatting, mine looked like this:

```
use std::fs::File;    
use encoding_rs::WINDOWS_1252;
use encoding_rs_io::DecodeReaderBytesBuilder;

// ...

    let file = File::open(path)?;
    let transcoded = DecodeReaderBytesBuilder::new()
        .encoding(Some(WINDOWS_1252))
        .build(file);
    let mut rdr = csv::ReaderBuilder::new()
        .delimiter(b',')
        .from_reader(transcoded); 
```

_ref:_ [_https://stackoverflow.com/questions/53826986/how-to-read-a-non-utf8-encoded-csv-file_](https://stackoverflow.com/questions/53826986/how-to-read-a-non-utf8-encoded-csv-file)

### \[Rust\] Instantiating the data

To instantiate the data, I used Serde [https://serde.rs/](https://serde.rs/) for serializing and deserializing my data.

To use Serde, I needed to make a struct of my data. But, having a struct of my data is great has it makes my code follow a model-based coding paradigm with a well-defined type for each field. It also enables me to implement traits and methods on top of them.

However, the data I wanted to use has 130 columns… And, It seemed that there is no way to generate the definition of the struct automatically.

To avoid doing the definition manually, I had to build my own struct generator:

```
fn inspect(path: &str) {
    let mut record: Record = HashMap::new();

    let mut rdr = csv::Reader::from_path(path).unwrap();

    for result in rdr.deserialize() {
        match result {
            Ok(rec) => {
                record = rec;
                break;
            }
            Err(e) => (),
        };
    }
    // Print Struct
    println!("#[skip_serializing_none]");
    println!("#[derive(Debug, Deserialize, Serialize)]");
    println!("struct DataFrame {{");
    for (key, value) in &record {
        println!("    #[serialize_always]");

        match value.parse::<i64>() {
            Ok(n) => {
                println!("    {}: Option<i64>,", key);
                continue;
            }
            Err(e) => (),
        }
        match value.parse::<f64>() {
            Ok(n) => {
                println!("    {}: Option<f64>,", key);
                continue;
            }
            Err(e) => (),
        }
        println!("    {}: Option<String>,", key);
    }
    println!("}}");
}
```

This generated the struct as follows:

```
use serde::{Deserialize, Serialize};
use serde_with::skip_serializing_none;

#[skip_serializing_none]
#[derive(Debug, Clone, Deserialize, Serialize)]
struct DataFrame {
    #[serialize_always]
    individual: Option<f64>,
    #[serialize_always]
    natlty3_txt: Option<String>,
    #[serialize_always]
    ransom: Option<f64>,
    #[serialize_always]
    related: Option<String>,
    #[serialize_always]
    gsubname: Option<String>,
    #[serialize_always]
    claim2: Option<String>,
    #[serialize_always]

    // ...
```

_skip\_serializing\_none: Avoid having error on empty fields in the CSV._

_serialize\_always: Makes the number of field when writing csv fixed._

Now, that I had my struct, I used serde serialization to populate a vector of struct:

```
    let mut records: Vec<DataFrame> = Vec::new();

    for result in rdr.deserialize() {
        match result {
            Ok(rec) => {
                records.push(rec);
            }
            Err(e) => println!("{}", e),
        };
    }
```

This generated my vector of struct, hooray 🎉

On a general note with Rust, you shouldn’t expect things to work as smoothly as it would with Python.

### **Conclusion**

On reading / instantiating data, **Pandas** wins hands down for CSV.

---

## 2. Filtering

### \[Pandas\]

There are many ways to do filtering in pandas, the most common way for me is as follows:

```
df = df[df.country_txt == "United States"]
df.to_csv("python_output.csv")
```

### \[Rust\]

To do filtering in Rust, we can refer to the docs for vector in Rust [https://doc.rust-lang.org/std/vec/struct.Vec.html](https://doc.rust-lang.org/std/vec/struct.Vec.html)

There is a large umbrella of methods for Vector filtering, with many nightly features that are going to be great for data manipulation when they ship. For this use case, I used the `retain` method has it was fitted my need perfectly:

```
    records.retain(|x| &x.country_txt.unwrap() == "United States");
    let mut wtr =
        csv::Writer::from_path("output_rust_filter.csv")?;

    for record in &records {
        wtr.serialize(record)?;
    }
```

**One big difference between Pandas and Rust is that Rust filtering uses Closures \(_eq. lambda function in python_\) whereas Pandas filtering uses Pandas API based on columns. This means Rust can make more complex filters compared to Pandas. It also adds in readability in my opinion.**

### Performance

| |Time\(s\) |Mem Usage\(Gb\) |
| --- | --- | --- |
|Pandas |3.0s |2.5Gb |
|Rust |1.6s 🔥 -50% |1.7Gb 🔥 -32% |

Even though we’re using Pandas API for filtering, we get significantly better performance using Rust.

### **Conclusion**

On Filtering, **Rust** seems to be more capable and faster. 🚅

---

## 3. Group By

### \[Pandas\]

Group by are a big part of the data reduction pipeline in python, it goes usually as follows:

```
df = df.groupby(by="country_txt", as_index=False).agg(
    {"nkill": "sum", "individual": "mean", "eventid": "count"}
)
df.to_csv("python_output_groupby.csv")
```

### \[Rust\]

For group by and data reduction, thanks to [David Sanders](https://able.bio/insideoutclub), group by can be done as follows:

```
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
|Rust |2.0s🔥 -35% |1.7Gb🔥 -32% |

‌

### **Conclusion**

Although the performance is better for Rust, I would advise using **Pandas** for map-reduce heavy application, as it seems more appropriate.

---

## 4. Mutation

### \[Pandas\]

There are many ways to do mutation in Pandas, I usually do the following for performance and functional style:

```
df["computed"] = df["nkill"].map(lambda x: (x - 10) / 2 + x ** 2 / 3)
df.to_csv("python_output_map.csv")
```

### \[Rust\]

For mutation, the functional `iter` of Rust really makes this part a walk in the park:

```
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
|Rust |1.58s🔥 -87% |1.7Gb🔥 -64% |

This is where the difference really appeared to me. Pandas do not scale for line-by-line custom build lambda functions. Pandas would have been even worst if I had done an operation involving several columns.

### **Conclusion**

**Rust** is way better for line-by-line mutation natively.

---

## 5. Merge

### \[Python\]

Merging in python is pretty efficient generally speaking, it goes like this in general:

```
df_country = pd.read_csv(
    "/home/peter/Documents/TEST/RUST/terrorism/src/WDICountry.csv"
)

df_merge = pd.merge(
    df, df_country, left_on="country_txt", right_on="Short_Name"
)
df_merge.to_csv("python_output_merge.csv")
```

### \[Rust\]

For Rust, however, this is a tricky part as, with Struct, merging isn’t really a thing. For me, the rustiest way of doing a merge is by adding a nested field containing the other struct we want to join data with.

I  first created a new struct and a new heap for the new data:

```
#[skip_serializing_none]
#[derive(Clone, Debug, Deserialize, Serialize)]
struct DataFrameCountry {
    #[serialize_always]
    SNA_price_valuation: Option<String>,
    #[serialize_always]
    IMF_data_dissemination_standard: Option<String>,
    #[serialize_always]
    Latest_industrial_data: Option<String>,
    #[serialize_always]
    System_of_National_Accounts: Option<String>,
    //...

// ...

    let mut records_country: Vec<DataFrameCountry> = Vec::new();
    let file = File::open(path_country)?;
    let transcoded = DecodeReaderBytesBuilder::new()
        .encoding(Some(WINDOWS_1252))
        .build(file);
    let mut rdr = csv::ReaderBuilder::new()
        .delimiter(b',')
        .from_reader(transcoded); 

    for result in rdr.deserialize() {
        match result {
            Ok(rec) => {
                records_country.push(rec);
            }
            Err(e) => println!("{}", e),
        };
    }
```

I then cloned this new struct with the previous struct on a specific field that is unique.

```

impl DataFrame {
    fn add_country_ext(&mut self, country: Option<DataFrameCountry>) {
        self.country_merge = Some(country)
    }
}

//...

    for country in records_country {
        records
            .iter_mut()
            .filter(|record| record.country_txt == country.Short_Name)
            .for_each(|x| {
                x.add_country_ext(Some(country.clone()));
            });
    }
    let mut wtr =
        csv::Writer::from_path("output_rust_join.csv")
            .unwrap();
    for record in &records {
        wtr.serialize(record)?;
    }
```

I cloned the data for convenience and also for better comparability, but a reference can be passed if you can manage it.

And there we go! 🚀

Except, a nested struct is not yet serializable in CSV for Rust -> [https://github.com/BurntSushi/rust-csv/pull/197](https://github.com/BurntSushi/rust-csv/pull/197)

So I had to adapt it to:

```
impl DataFrame {
    fn add_country_ext(&mut self, country: Option<DataFrameCountry>) {
        self.country_ext = Some(format!("{:?}", country))
    }
}
```

But, then, we got a sort of merge! 🚀

### Performance

| |Time\(s\) |Mem\(Gb\) |
| --- | --- | --- |
|Pandas |22.47s |11.8Gb |
|Rust |5.48s🔥 -75% |2.6Gb🔥 -78% |

‌

### **Conclusion**

**Rust** is capable of doing nested structs that are going to be as capable if not more capable than **Pandas** merges. However, it isn’t really a one to one comparison and in this case, it is going to depend on your use case.

‌

---

## Ending conclusion

After this experience, this is my take away.

* Use Pandas when you can: small CSV\(<1M lines\), simple operation, data cleaning …
* Use Rust when you have: complex operations, memory heavy or time-consuming pipelines, custom build functions, scalable software…

That been said, Rust offers impressive flexibility compared to Pandas. Adding the fact that Rust is way more capable of multi-threading than Pandas, I believe that Rust can solve problems Pandas simply cannot.

Additionally, the possibility to run Rust on any platform\(Web, Android, or Embedded\) also create new opportunities for data manipulation in places inconceivable for Pandas and can provide solutions for yet to be resolved challenges.

### Performance

The performance table gives us an insight as to what to expect from Rust. I believe, the speedup can go from **x2** at the minimum and up to **x50** for large data pipelines. The memory use will have an even greater decrease as memory usage accumulates over time with python.

### Disclaimer

In many ways, Pandas can be optimized, but optimisation comes at a cost, whether it is hardware \(e.g. Cluster #Dask, GPU #Cudf\) or dependency on the reliability and maintenance of those optimisation packages.

What I like about using Native Rust is that it doesn’t rely on additional hardware nor does it depends on additional packages. This solution doesn’t require an additional layer of abstraction, which makes it way more intuitive in many regards.

‌

‌

**I wrote a continuation article covering polars a rust dataframe crate in another post:** [**https://able.bio/haixuanTao/data-manipulation-polars-vs-rust--3def44c8**](https://able.bio/haixuanTao/data-manipulation-polars-vs-rust--3def44c8)

**with another repo:** [**https://github.com/haixuanTao/dataframe-python-rust**](https://github.com/haixuanTao/dataframe-python-rust)

---

## Annexe

### Git repository

[https://github.com/haixuanTao/Data-Manipulation-Rust-Pandas](https://github.com/haixuanTao/Data-Manipulation-Rust-Pandas)

‌

### Data-Oriented Design

One other way to handle data in Rust is using a data-oriented design, that translates into building a struct of heap: [https://jamesmcm.github.io/blog/2020/07/25/intro-dod/](https://jamesmcm.github.io/blog/2020/07/25/intro-dod/)

I first tried to build it this way, but, I did not manage to make it work, as I had to implement a lot of function to make things like filtering work on all columns.

‌

### BTreeMap

One way to optimize for speed for Rust would be to replace the Vector with a Binary tree as the Heap. The filtering will then be done on this Tree, making the filtering faster. This would work if one field can be used as a primary filter.

‌

‌

‌