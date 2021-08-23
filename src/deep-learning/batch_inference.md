## Deep Learning batch inference: Running BERT on a CSV


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
