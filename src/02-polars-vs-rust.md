# Pandas vs Polars 

[<img alt="github" src="https://img.shields.io/badge/dataframe--python--rust-fff?labelColor=000&logo=github" height="20">](https://github.com/haixuantao/dataframe-python-rust)
[![GitHub stars](https://img.shields.io/github/stars/haixuanTao/dataframe-python-rust?style=social&label=Star&maxAge=2592000)](https://github.com/haixuanTao/dataframe-python-rust/)
## Introduction

Everyone loves the API of Pandas. It’s fast, easy, and well documented. There are some rough edges, but most times, it’s just a blast.

Now, when it comes to production, Pandas is slightly trickier. Pandas does not scale very well… there is no multithreading… It’s not thread-safe… It’s not memory efficient.

But all those problems are the raison d’être of Rust.

**What if, there was a DataFrame API written in Rust that solves all those issues and at the same time keeps a nice API?**
