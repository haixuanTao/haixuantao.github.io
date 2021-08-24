## ONNX Server: Serving BERT as an API

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
[<img alt="github" src="https://img.shields.io/badge/bert--onnx--rs--pipeline-fff?labelColor=000&logo=github" height="20">](https://github.com/haixuantao/bert-onnx-rs-pipeline)
[![GitHub stars](https://img.shields.io/github/stars/haixuanTao/bert-onnx-rs-pipeline?style=social&label=Star&maxAge=2592000)](https://github.com/haixuanTao/bert-onnx-rs-pipeline/)