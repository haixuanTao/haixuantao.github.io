# Deep Learning in Rust
[<img alt="github" src="https://img.shields.io/badge/onnxruntime_rs-fff?labelColor=000&logo=github" height="20">](https://github.com/haixuantao/onnxruntime-rs)
[<img alt="build status" src="https://img.shields.io/github/workflow/status/haixuantao/onnxruntime-rs/Rust/master?" height="20">](https://github.com/haixuantao/onnxruntime-rs/actions?query=branch%3Amaster)
[![GitHub stars](https://img.shields.io/github/stars/haixuanTao/onnxruntime-rs?style=social&label=Star&maxAge=2592000)](https://github.com/haixuanTao/onnxruntime-rs/)

[<img alt="github" src="https://img.shields.io/badge/bert--onnx--rs--pipeline-fff?labelColor=000&logo=github" height="20">](https://github.com/haixuantao/bert-onnx-rs-pipeline)
[![GitHub stars](https://img.shields.io/github/stars/haixuanTao/bert-onnx-rs-pipeline?style=social&label=Star&maxAge=2592000)](https://github.com/haixuanTao/bert-onnx-rs-pipeline/)

[<img alt="github" src="https://img.shields.io/badge/bert--onnx--rs--server-fff?labelColor=000&logo=github" height="20">](https://github.com/haixuantao/bert-onnx-rs-server)
[![GitHub stars](https://img.shields.io/github/stars/haixuanTao/bert-onnx-rs-server?style=social&label=Star&maxAge=2592000)](https://github.com/haixuanTao/bert-onnx-rs-server/)

I have searched for months for a way to do Deep Learning\(DL\) Inference with Rust on GPU and I finally did it!!✨👏✨ This blog post will try to answer if Rust is a good fit for the job!

_I have put an annexe at the end with the definition of Deep Learning words._

## My setup

I am using a Hugging Face **tokenizer** and a custom **BERT** Model from Pytorch that I have converted to **ONNX** to be run with [**onnxruntime-rs**](https://github.com/nbigaouette/onnxruntime-rs)**.**

I have tweaked [onnxruntime-rs](https://github.com/nbigaouette/onnxruntime-rs) to do Deep Learning on GPU with CUDA 11 and onnxruntime 1.8 You can check it out on my git: [https://github.com/haixuanTao/onnxruntime-rs](https://github.com/haixuanTao/onnxruntime-rs)

Hardware-side, I have a 6 cores/12 threads CPU and a GTX 1050 GPU.

## Case studies Results

Looking at those results alone is not enough. To dig a little further, I have built a DL data pipeline for batch inference and a DL server, to see what Rust for DL could be like on a daily basis.
