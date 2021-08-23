# Hardware


### 1. Inference time

| |Inferencing time per phrase |Speedup |
| --- | --- | --- |
|\(Rust or Python\) ONNX CPU |~125ms | |
|\(Rust or Python\) ONNX GPU |~10ms |**x12**🔥 |

**DL inference using Onnxruntime will not be faster in Rust because both are wrapping the same C\+\+ underlying engine. What is going to make a difference is the GPU.**