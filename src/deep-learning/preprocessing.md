# preprocessing

### 2. Preprocessing time

| |Tokenizing time per phrase |Speedup |
| --- | --- | --- |
|Python BertTokenizer |1000μs | |
|Python BertTokenizerFast |200-600μs |**x2.5**🔥 |
|Rust [Tokenizer](https://docs.rs/tokenizers/0.10.1/tokenizers/) |50-150μs |**x4**🔥 |

**Gains can be made on DL Preprocessing! You can tokenize 4 times faster in Rust compared to Python, with the same Hugging Face Tokenizer library.**
