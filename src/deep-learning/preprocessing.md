# Preprocessing

Preprocessing for Deep Learning is inevitable and can be very expensive. In the case of NLP, preprocessing translates to tokenizing. 

To compare performance, I used HuggingFace tokenizer which is implemented in Rust, in Python and in Rust-Pyo3 Python.

The code is as follows for the python native tokenizer:
```python
from transformers import BertTokenizer

PRE_TRAINED_MODEL_NAME = "bert-base-cased"

tokenizer = BertTokenizer.from_pretrained(PRE_TRAINED_MODEL_NAME)

encoding = tokenizer(
        df["Title"].to_numpy().tolist(),
        add_special_tokens=True,
        max_length=60,
        return_token_type_ids=False,
        padding="max_length",
        truncation=True,
        return_attention_mask=True,
        return_tensors="np",
    )
```

The Rust-Python Bertokenizer:
```python
from transformers import BertTokenizerFast

PRE_TRAINED_MODEL_NAME = "bert-base-cased"

tokenizer = BertTokenizerFast.from_pretrained(PRE_TRAINED_MODEL_NAME)

encoding = tokenizer(
        df["Title"].to_numpy().tolist(),
        add_special_tokens=True,
        max_length=60,
        return_token_type_ids=False,
        padding="max_length",
        truncation=True,
        return_attention_mask=True,
        return_tensors="np",
    )
```
And, the native Rust HuggingFace Tokenizer:
```rust

use tokenizers::models::wordpiece::WordPieceBuilder;
use tokenizers::normalizers::bert::BertNormalizer;
use tokenizers::pre_tokenizers::bert::BertPreTokenizer;
use tokenizers::processors::bert::BertProcessing;
use tokenizers::tokenizer::AddedToken;
use tokenizers::tokenizer::{EncodeInput, Encoding, Tokenizer};
use tokenizers::utils::padding::{PaddingDirection::Right, PaddingParams, PaddingStrategy::Fixed};
use tokenizers::utils::truncation::TruncationParams;
use tokenizers::utils::truncation::TruncationStrategy::LongestFirst;

fn main() -> std::result::Result<(), OrtError> {
    let vocab_path = "./src/vocab.txt";
    let wp_builder = WordPieceBuilder::new()
        .files(vocab_path.into())
        .continuing_subword_prefix("##".into())
        .max_input_chars_per_word(100)
        .unk_token("[UNK]".into())
        .build()
        .unwrap();

    let mut tokenizer = Tokenizer::new(Box::new(wp_builder));
    tokenizer.with_pre_tokenizer(Box::new(BertPreTokenizer));
    tokenizer.with_truncation(Some(TruncationParams {
        max_length: 60,
        strategy: LongestFirst,
        stride: 0,
    }));
    tokenizer.with_post_processor(Box::new(BertProcessing::new(
        ("[SEP]".into(), 102),
        ("[CLS]".into(), 101),
    )));
    tokenizer.with_normalizer(Box::new(BertNormalizer::new(true, true, false, false)));
    tokenizer.add_special_tokens(&[
        AddedToken {
            content: "[PAD]".into(),
            single_word: false,
            lstrip: false,
            rstrip: false,
        },
        AddedToken {
            content: "[CLS]".into(),
            single_word: false,
            lstrip: false,
            rstrip: false,
        },
        AddedToken {
            content: "[SEP]".into(),
            single_word: false,
            lstrip: false,
            rstrip: false,
        },
        AddedToken {
            content: "[MASK]".into(),
            single_word: false,
            lstrip: false,
            rstrip: false,
        },
    ]);
    tokenizer.with_padding(Some(PaddingParams {
        strategy: Fixed(60),
        direction: Right,
        pad_id: 0,
        pad_type_id: 0,
        pad_token: "[PAD]".into(),
    }));
}
```
## Performance 

| |Time per phrase |Speedup |
| --- | --- | --- |
|Python BertTokenizer |1000μs | |
|Python BertTokenizerFast |200-600μs |x2.5 🔥 |
|**Rust [Tokenizer](https://docs.rs/tokenizers/0.10.1/tokenizers/)** |**50-150μs** |**x4** 🔥 |

You can tokenize 4 times faster in Rust than Python, with the same Hugging Face Tokenizer library.

Preprocessing can be very performant in Rust, making a case that Rust can outperform Python for Deep Learning.