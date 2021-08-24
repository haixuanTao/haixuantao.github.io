# Hardware

Initially, onnxruntime-rs did not support GPU / CUDA despite having a C API.

But by tweaking [Onnxruntime-rs](https://github.com/nbigaouette/onnxruntime-rs), I could use the GPU C API and run DL Inference on GPU. 

I opened a PR: [https://github.com/nbigaouette/onnxruntime-rs/pull/87](https://github.com/nbigaouette/onnxruntime-rs/pull/87) providing the CUDA support for Linux and Windows.

And with similar work, a majority of the acceleration hardware could be added actually.

➡️ So, yes, Rust can run DL on GPU!

## GPU Support

GPU Support was enabled by:
- adding 2 header files in bindgen's `wrapper.h` file as follows: 
```c
#include "onnxruntime_c_api.h"
#if !defined(__APPLE__)
  #include "cpu_provider_factory.h"
  #include "cuda_provider_factory.h"
#endif
```

- adding a safe API to the newly added bindings:

```rust
    /// Set the session to use cpu
    #[cfg(feature = "cuda")]
    pub fn use_cpu(self, use_arena: i32) -> Result<SessionBuilder<'a>> {
        unsafe {
            sys::OrtSessionOptionsAppendExecutionProvider_CPU(self.session_options_ptr, use_arena);
        }
        Ok(self)
    }

    /// Set the session to use cuda
    #[cfg(feature = "cuda")]
    pub fn use_cuda(self, device_id: i32) -> Result<SessionBuilder<'a>> {
        unsafe {
            sys::OrtSessionOptionsAppendExecutionProvider_CUDA(self.session_options_ptr, device_id);
        }
        Ok(self)
    }
```
- Generating the binding for Linux:

```bash
>>> cargo build --package onnxruntime-sys --features "generate-bindings cuda" --target x86_64-unknown-linux-gnu
```

- And, generating the bindings for Windows in a Windows VM:
```bash
>>> cargo build --features "generate-bindings cuda" --target x86_64-pc-windows-msvc
```

## Performance

| |Time per phrase |Speedup |
| --- | --- | --- |
|Rust ONNX CPU |~125ms | |
|Rust ONNX GPU |~10ms |**x12**🔥 |

*Note: I have a six cores CPU and a GTX 1050 GPU.*

As expected, the GPU drastically reduced the time of inference.

However, I did not found significant speedup between Onnxruntime Rust and Onnxruntime Python.