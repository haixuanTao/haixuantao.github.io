## Batch inference: Running BERT on 10k phrases.	

At work, we often develop Deep Learning model to be used on large batches of data.

To see if Rust can improve this usecase, I trained a BERT-like model and infered 10k phrases using Python and Rust.

### Performance

|10k phrases |Python |Rust |
| --- | --- | --- |
|Booting |4s |1s |
|Encoding |0.7s |0.3s |
|DL Inference |75s |75s |
|Total |80s |76s |
|Memory usage |1 GiB |0.7 GiB |

As DL inference is taking the majority of the time, Rust will only marginely improve performance.

This is an example of a bad use case for Rust as time is consumed in the C API which does not get affected by Rust.

_You can check out the code for this specific job at:_  [_https://github.com/haixuanTao/bert-onnx-rs-pipeline_](https://github.com/haixuanTao/bert-onnx-rs-pipeline) 


[<img alt="github" src="https://img.shields.io/badge/bert--onnx--rs--pipeline-fff?labelColor=000&logo=github" height="20">](https://github.com/haixuantao/bert-onnx-rs-pipeline)
[![GitHub stars](https://img.shields.io/github/stars/haixuanTao/bert-onnx-rs-pipeline?style=social&label=Star&maxAge=2592000)](https://github.com/haixuanTao/bert-onnx-rs-pipeline/)
