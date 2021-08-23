## Merging

### \[Python\]

Merging in python is pretty efficient generally speaking, it goes like this in general:

```python
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

```rust
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

```rust

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

```rust
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
|**Rust** |**5.48s🔥 -75%** |**2.6Gb🔥 -78%** |

**Rust** is capable of doing nested structs that are going to be as capable if not more capable than **Pandas** merges. However, it isn’t really a one to one comparison and in this case, it is going to depend on your use case.