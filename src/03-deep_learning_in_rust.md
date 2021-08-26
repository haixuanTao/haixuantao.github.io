# Deep Learning in Rust
[<img alt="github" src="https://img.shields.io/badge/onnxruntime_rs-fff?labelColor=000&logo=github" height="20">](https://github.com/haixuantao/onnxruntime-rs)
[<img alt="build status" src="https://img.shields.io/github/workflow/status/haixuantao/onnxruntime-rs/Rust/master?" height="20">](https://github.com/haixuantao/onnxruntime-rs/actions?query=branch%3Amaster)
[![GitHub stars](https://img.shields.io/github/stars/haixuanTao/onnxruntime-rs?style=social&label=Star&maxAge=2592000)](https://github.com/haixuanTao/onnxruntime-rs/)

## Introduction

Building Deep Learning algorithms is paramount for doing Data Science in Rust. In this post, I show how:
- Rust can support GPU.
- Rust can provide superior performance than Python and by how much.
- Good and bad use case for Deep Learning in Rust.


## State of the art of Deep Learning in Rust

Deep Learning in the Rust ecosystem is spread between native libraries like [linfa](https://github.com/rust-ml/linfa) and C++ binding of common libraries like [Tensorflow](https://github.com/tensorflow/rust), [Pytorch](https://github.com/LaurentMazare/tch-rs) and [Onnxruntime](https://github.com/nbigaouette/onnxruntime-rs).

I have found [onnxruntime-rs](https://github.com/nbigaouette/onnxruntime-rs) to be a convenient crate for DL offering:
- the ability to load sklearn, tensorflow and pytorch model.
- superior performance than native Pytorch or Tensorflow.  
- a small bundle size ~30Mb compared to [tch-rs](https://github.com/pytorch/pytorch/issues/34058) 1.2 Gb bundle. 

➡️ this post is therefore going to be based on onnxruntime-rs.

<br/>

_This blog was originally published here: [https://able.bio/haixuanTao/deep-learning-in-rust-with-gpu--26c53a7f](https://able.bio/haixuanTao/deep-learning-in-rust-with-gpu--26c53a7f)_