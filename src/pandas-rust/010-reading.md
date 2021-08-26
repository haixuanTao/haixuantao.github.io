## Reading

### Pandas

Reading and instantiating Data in Pandas is pretty straightforward, and handles by default many data quality problems:

```python
import pandas as pd

path = "/home/peter/Documents/TEST/RUST/terrorism/src/globalterrorismdb_0718dist.csv"
df = pd.read_csv(path)
```

### Rust Reading CSV

For Rust, Managing bad quality data is very very tedious. In this dataset, some fields are empty, some lines are badly formatted, and some are not UTF-8 encoded.

To open the CSV, I used the `csv` crate but it does not solve all the issues listed above. With well-formatted data, reading can be done like so:

```rust,noplaypen
let path = "/home/peter/Documents/TEST/RUST/terrorism/src/foo.csv";
let mut rdr = csv::Reader::from_path(path).unwrap();
```

But with bad quality formatting, I had to add additional parameters like:

```rust,noplaypen
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

### Rust Instantiating the data

To instantiate the data, I used Serde [https://serde.rs/](https://serde.rs/) for serializing and deserializing my data.

To use Serde, I needed to make a struct of my data. Having a struct of my data is great as it makes my code follow a model-based coding paradigm with a well-defined type for each field. It also enables me to implement traits and methods on top of them.

However, the data I wanted to use has 130 columns… And, It seemed that there is no way to generate the definition of the struct automatically.

To avoid doing the definition manually, I had to build my own struct generator:

```rust,noplaypen

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

```rust,noplaypen
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

```rust,noplaypen
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

On reading / instantiating data, **Pandas** wins hands down for CSV.
