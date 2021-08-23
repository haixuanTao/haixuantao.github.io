# Pandas vs Polars 

[<img alt="github" src="https://img.shields.io/badge/dataframe--python--rust-fff?labelColor=000&logo=github" height="20">](https://github.com/haixuantao/dataframe-python-rust)
[![GitHub stars](https://img.shields.io/github/stars/haixuanTao/dataframe-python-rust?style=social&label=Star&maxAge=2592000)](https://github.com/haixuanTao/dataframe-python-rust/)
## Introduction

Everyone loves the API of Pandas. It’s fast, easy, and well documented. There are some rough edges, but most times, it’s just a blast.

Now, when it comes to production, Pandas is slightly trickier. Pandas does not scale very well… there is no multithreading… It’s not thread-safe… It’s not memory efficient.

But all those problems are the raison d’être of Rust.

**But what if, there was a DataFrame API written in Rust that solves all those issues and at the same time keeps a nice API?**

## Case Study

**Now the question is, is it better than native Rust as I’ve explained** [**in my previous blog post**](https://able.bio/haixuanTao/data-manipulation-pandas-vs-rust--1d70e7fc)**?**

Let’s take a hands-on comparison for a Data Pipeline and get a feel for it.

In this case study, I’m going to use the [stack overflow kaggle dataset](https://www.kaggle.com/c/predict-closed-questions-on-stack-overflow/data?select=2012-07+Stack+Overflow.7z). I’m going to read the database, parse the dates, make a merge between the first tag and the[ Wikipedia comparison of programming language](https://en.wikipedia.org/wiki/Comparison_of_programming_languages#Failsafe_I/O_and_system_calls). Group by the status of the question asked. And retrieve the distribution of language’s features within each ‘status’ of questions.

We’ll compare Polars API & Native Rust generic heap structure to do this task.

* _I’ll go slightly quicker on the native Rust, as I already put more details_ [_here_](https://able.bio/haixuanTao/data-manipulation-pandas-vs-rust--1d70e7fc)_._
* _Multithreading is done on 12 threads Intel\(R\) Core\(TM\) i7-8750H / 20G RAM._
* _The database is 4.2G big for around 3.6 Million rows._