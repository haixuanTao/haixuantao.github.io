## Polars

Well, [**Polars**](https://github.com/ritchie46/polars) tries to do just that. It allows you to do read, write, filter, apply functions, group by and merge, all in a **thread-safe fashion**.

It uses [**Apache Arrow**](https://github.com/apache/arrow), a data framework purposely built for doing efficient data processing and data sharing across language.

### Polars Lazy

Polars also offers a query optimized version called Lazy with a slightly different API. In my use case, I did not find it hard to go from one to the other, but I did not find any significant increase in performance either. The result is in the overall performance.

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

