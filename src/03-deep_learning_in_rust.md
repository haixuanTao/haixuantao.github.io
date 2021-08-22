# Deep Learning in Rust

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

### 1. Deep Learning batch inference: Running BERT on a CSV

Let say you want the inference of a BERT model on one column of a 10 thousand lines CSV.

On my setup, I got those timings:

|10k phrases |Python |Rust |
| --- | --- | --- |
|Booting time |4s |1s |
|Encoding time |0.7s |0.3s |
|DL Inference time |75s |75s |
|Total time |80s |76s |
|Memory usage |1 GiB |0.7 GiB |

**As DL inference is taking the majority of the time, Rust will not increase performance and I would not bother with Rust and stay with Python for large batches of inference.**   👍🐍

_Git:_  [_https://github.com/haixuanTao/bert-onnx-rs-pipeline_](https://github.com/haixuanTao/bert-onnx-rs-pipeline)

### 2. ONNX Server: Serving BERT as an API

Let say you want to serve a BERT-like model through a server API endpoint.

On my setup, I got those metrics:

| |Python FastAPI |Rust Actix Web |Speedup |
| --- | --- | --- | --- |
|Encoding time |400μs |100μs | |
|ONNX Inference time |~10ms |~10ms | |
|API overhead time |~2ms |~1ms | |
|Mean Latency |12.8ms |10.4ms |-20%⏰ |
|Requests/secs |77.5 #/s |95 #/s |\+22%🍾 |

**The gain in performance comes from moving from considered “Fast” Python library to Rust: FastAPI -> Actix Web, BertokenizerFast -> Rust Tokenizer.**

**Thus, as Rust libraries tend to be faster than Python ones, the more functionalities you will have, the more speedup you’re going to see with Rust when serving Deep Learning.**

**That’s why, for performance-centric Deep Learning applications such as Real-Time Deep Learning, Embedded Deep Learning, Large-Scale AI servers …. I can definitely see Rust be a good fit!** ❤️‍🦀

_Git:_ [_https://github.com/haixuanTao/bert-onnx-rs-server_](https://github.com/haixuanTao/bert-onnx-rs-server)

### In conclusion, should you use Rust for Deep Learning?

* Like the whole Rust ecosystem, you should use it if it’s the best tool for the job. If you really need performance 🏎️ and resilience🛡️, and you are ok to have a stack in Rust, go ahead🚀! But be aware that making Rust fast is not easy!
* If you need quick prototyping with a data scientist friendly language, you should better use Python!

‌

**There will be a following blog post around the actual implementation of the DL pipeline and server so make sure to follow along** 😀 :

* On able: [https://able.bio/haixuanTao](https://able.bio/haixuanTao)
* On github: [https://github.com/haixuanTao](https://github.com/haixuanTao)
* On linkedin: [https://www.linkedin.com/in/haixuan-xavier-tao-7460b1102/](https://www.linkedin.com/in/haixuan-xavier-tao-7460b1102/)

    



‌

‌

‌

‌

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