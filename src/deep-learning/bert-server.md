## ONNX Server: Serving BERT as an API

Another use case is serving a BERT-like model as a server with a REST endpoint.

To see if Rust could be more performant than Python, I served the onnx model through [actix-web](https://actix.rs/), and to benchmark it, I made a clone in Python with [FastAPI](https://fastapi.tiangolo.com/).



### Performance 

For a request of one phrase:

| |Python FastAPI |Rust Actix Web |Speedup |
| --- | --- | --- | --- |
|Encoding |400μs |100μs | |
|ONNX Inference |~10ms |~10ms | |
|API overhead |~2ms |~1ms | |
|Mean Latency |12.8ms |10.4ms |-20%⏰ |
|Requests/secs |77.5 #/s |95 #/s |\+22%🔥 |

The gain in performance comes from moving from considered “Fast” Python library to Rust: 
- FastAPI ⏩ Actix Web
- BertokenizerFast ️⏩ Rust Tokenizer

Thus, as Rust libraries tend to be faster than Python ones, Rust will be faster when the application is a composition of libraries.

That’s why, I can see Rust be a good fit for excessively performance centric applications such as Real-Time Deep Learning, Embedded Deep Learning, Large-Scale AI servers! ❤️‍🦀

_Check the code:_ [_https://github.com/haixuanTao/bert-onnx-rs-server_](https://github.com/haixuanTao/bert-onnx-rs-server)  
[<img alt="github" src="https://img.shields.io/badge/bert--onnx--rs--server-fff?labelColor=000&logo=github" height="20">](https://github.com/haixuantao/bert-onnx-rs-server)
[![GitHub stars](https://img.shields.io/github/stars/haixuanTao/bert-onnx-rs-server?style=social&label=Star&maxAge=2592000)](https://github.com/haixuanTao/bert-onnx-rs-server/)