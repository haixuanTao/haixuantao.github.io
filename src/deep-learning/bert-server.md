## ONNX Server: Serving BERT as an API
[![GitHub stars](https://img.shields.io/github/stars/haixuanTao/bert-onnx-rs-server?style=social&label=Star&maxAge=2592000)](https://github.com/haixuanTao/bert-onnx-rs-server/)
[![GitHub forks](https://img.shields.io/github/forks/haixuanTao/bert-onnx-rs-server?style=social&label=Fork&maxAge=2592000)](https://github.com/haixuanTao/bert-onnx-rs-server/)
[![GitHub stars](https://img.shields.io/github/last-commit/haixuantao/bert-onnx-rs-server)](https://github.com/haixuanTao/bert-onnx-rs-server/)

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
