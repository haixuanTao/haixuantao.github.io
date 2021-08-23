# Deep Learning in Rust
[![GitHub stars](https://img.shields.io/github/stars/haixuanTao/onnxruntime-rs?style=social&label=Star&maxAge=2592000)](https://github.com/haixuanTao/onnxruntime-rs/)
[![GitHub forks](https://img.shields.io/github/forks/haixuanTao/onnxruntime-rs?style=social&label=Fork&maxAge=2592000)](https://github.com/haixuanTao/onnxruntime-rs/)
[![GitHub stars](https://img.shields.io/github/last-commit/haixuantao/onnxruntime-rs)](https://github.com/haixuanTao/onnxruntime-rs/)

I have searched for months for a way to do Deep Learning\(DL\) Inference with Rust on GPU and I finally did it!!✨👏✨ This blog post will try to answer if Rust is a good fit for the job!

_I have put an annexe at the end with the definition of Deep Learning words._

## My setup

I am using a Hugging Face **tokenizer** and a custom **BERT** Model from Pytorch that I have converted to **ONNX** to be run with [**onnxruntime-rs**](https://github.com/nbigaouette/onnxruntime-rs)**.**

I have tweaked [onnxruntime-rs](https://github.com/nbigaouette/onnxruntime-rs) to do Deep Learning on GPU with CUDA 11 and onnxruntime 1.8 You can check it out on my git: [https://github.com/haixuanTao/onnxruntime-rs](https://github.com/haixuanTao/onnxruntime-rs)

Hardware-side, I have a 6 cores/12 threads CPU and a GTX 1050 GPU.

## Unit Results

### 1. Inference time

| |Inferencing time per phrase |Speedup |
| --- | --- | --- |
|\(Rust or Python\) ONNX CPU |~125ms | |
|\(Rust or Python\) ONNX GPU |~10ms |**x12**🔥 |

**DL inference using Onnxruntime will not be faster in Rust because both are wrapping the same C\+\+ underlying engine. What is going to make a difference is the GPU.**

### 2. Preprocessing time

| |Tokenizing time per phrase |Speedup |
| --- | --- | --- |
|Python BertTokenizer |1000μs | |
|Python BertTokenizerFast |200-600μs |**x2.5**🔥 |
|Rust [Tokenizer](https://docs.rs/tokenizers/0.10.1/tokenizers/) |50-150μs |**x4**🔥 |

**Gains can be made on DL Preprocessing! You can tokenize 4 times faster in Rust compared to Python, with the same Hugging Face Tokenizer library.**

## Case studies Results

Looking at those results alone is not enough. To dig a little further, I have built a DL data pipeline for batch inference and a DL server, to see what Rust for DL could be like on a daily basis.
### Git reference

_Git of my tweaked onnxruntime-rs library with ONNX 1.8 and GPU features with CUDA 11:_ [https://github.com/haixuanTao/onnxruntime-rs](https://github.com/haixuanTao/onnxruntime-rs)

_Git of bert - onnxruntime-rs - Pipeline:_  [_https://github.com/haixuanTao/bert-onnx-rs-pipeline_](https://github.com/haixuanTao/bert-onnx-rs-pipeline)

_Git of bert - onnxruntime-rs - actix - server:_ [_https://github.com/haixuanTao/bert-onnx-rs-server_](https://github.com/haixuanTao/bert-onnx-rs-server)

‌

### Annexe

[ONNX](https://onnx.ai/) is an open format built to represent machine learning models. You can convert Pytorch, Tensorflow and Sklearn models into an ONNX format and then run them with [onnxruntime](https://github.com/nbigaouette/onnxruntime-rs).

[ONNXRuntime](https://www.onnxruntime.ai/) is the inference and optimized training engine that can read and run ONNX model. It is written in C\+\+. There are official wrappers for Python, JS, JAVA, C and C\+\+.

[Onnxruntime-rs](https://github.com/nbigaouette/onnxruntime-rs) is the Onnxruntime wrapper for Rust, it is a community-developed solution and does not wrap all the features of the C\+\+ engine. I have tweaked the initial repo in order to have version 1.8 of Onnxruntime with the possibility of running it in GPU. My version is here: [https://github.com/haixuanTao/onnxruntime-rs](https://github.com/haixuanTao/onnxruntime-rs) with branch onnx1.8.

[Tokenizer](https://huggingface.co/transformers/main_classes/tokenizer.html) enables you to transform words into index given a list of word.

[BERT](https://huggingface.co/transformers/model_doc/bert.html) is a Deep Learning model used for natural language processing.

[GPU](https://en.wikipedia.org/wiki/Graphics_processing_unit) is a graphical processing unit for parallel computing. You will need CUDA and therefore an NVIDIA GPU to run Onnxruntime-rs.