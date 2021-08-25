## Batch inference: Running BERT on 10k phrases.	

At work, we often have situations where we develop a Deep Learning model, and then use it on large batches of data.

To see if this could be done with Rust, I have customed trained a BERT model and tried to do the inference 10 thousand phrases.

### Profiling

|10k phrases |Python |Rust |
| --- | --- | --- |
|Booting time |4s |1s |
|Encoding time |0.7s |0.3s |
|DL Inference time |75s |75s |
|Total time |80s |76s |
|Memory usage |1 GiB |0.7 GiB |

As DL inference is taking the majority of the time, Rust will not increase performance and I would not bother with Rust and stay with Python for large batches of inference.  👍🐍

_Git:_  [_https://github.com/haixuanTao/bert-onnx-rs-pipeline_](https://github.com/haixuanTao/bert-onnx-rs-pipeline)

[<img alt="github" src="https://img.shields.io/badge/bert--onnx--rs--server-fff?labelColor=000&logo=github" height="20">](https://github.com/haixuantao/bert-onnx-rs-server)
[![GitHub stars](https://img.shields.io/github/stars/haixuanTao/bert-onnx-rs-server?style=social&label=Star&maxAge=2592000)](https://github.com/haixuanTao/bert-onnx-rs-server/)
