## Filtering

### \[Pandas\]

There are many ways to do filtering in pandas, the most common way for me is as follows:

```python
df = df[df.country_txt == "United States"]
df.to_csv("python_output.csv")
```

### \[Rust\]

To do filtering in Rust, we can refer to the docs for vector in Rust [https://doc.rust-lang.org/std/vec/struct.Vec.html](https://doc.rust-lang.org/std/vec/struct.Vec.html)

There is a large umbrella of methods for Vector filtering, with many nightly features that are going to be great for data manipulation when they ship. For this use case, I used the `retain` method has it was fitted my need perfectly:

```rust
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
