# Pandas vs Polars 

[![GitHub stars](https://img.shields.io/github/stars/haixuanTao/dataframe-python-rust?style=social&label=Star&maxAge=2592000)](https://github.com/haixuanTao/dataframe-python-rust/)
[![GitHub forks](https://img.shields.io/github/forks/haixuanTao/dataframe-python-rust?style=social&label=Fork&maxAge=2592000)](https://github.com/haixuanTao/dataframe-python-rust/)
[![GitHub stars](https://img.shields.io/github/last-commit/haixuantao/dataframe-python-rust)](https://github.com/haixuanTao/dataframe-python-rust/)
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
### Polars Lazy

Polars also offers a query optimized version called Lazy with a slightly different API. In my use case, I did not find it hard to go from one to the other, but I did not find any significant increase in performance either. The result is in the overall performance.

---
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