# Pandas vs Rust (#1 Google Result)
[<img alt="github" src="https://img.shields.io/badge/Data--Manipulation--Rust--Pandas-fff?labelColor=000&logo=github" height="20">](https://github.com/haixuantao/Data-Manipulation-Rust-Pandas)
[![GitHub stars](https://img.shields.io/github/stars/haixuanTao/Data-Manipulation-Rust-Pandas?style=social&label=Star&maxAge=2592000)](https://github.com/haixuanTao/Data-Manipulation-Rust-Pandas/)
## Introduction


Pandas is the main Data analysis package of Python. For many reasons, Native Python has poor performance on data analysis without vectorization with NumPy and the likes. And historically, Pandas has been created by Wes McKinney to package those optimisations in a nice API to facilitate data analysis in Python.

This, however, is not necessary for Rust. Rust has great data performance natively. This is why Rust doesn’t really need a package like Pandas.

I believe the rustiest way to do Data Manipulation in Rust would be to build a **heap of data struct**.

This is my experience and reasoning comparing **Pandas vs Rust**.

### Data

Performance benchmarks are done on this very random dataset: [https://www.kaggle.com/START-UMD/gtd](https://www.kaggle.com/START-UMD/gtd) that offers around 160,000 lines / 130 columns for a total size of 150Mb. The size of this dataset corresponds to the type of dataset I regularly encounter, that’s why I chose this one. It isn’t the biggest dataset in the world, and, more studies should probably be done on a larger dataset.

The merge will be done with another random dataset: [https://datacatalog.worldbank.org/dataset/world-development-indicators](https://datacatalog.worldbank.org/dataset/world-development-indicators), the `WDICountry.csv`
