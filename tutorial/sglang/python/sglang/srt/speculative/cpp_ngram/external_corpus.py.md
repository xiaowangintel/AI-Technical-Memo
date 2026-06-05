# external_corpus.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/speculative/cpp_ngram/external_corpus.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file supports the n-gram speculative decoding path, especially corpus access or related auxiliary configuration for the C++ n-gram module. It helps the runtime reuse external text statistics during drafting. / 该文件服务于 n-gram 推测解码路径，尤其是 C++ n-gram 模块所需的语料访问或相关辅助配置。它帮助运行时在起草阶段复用外部文本统计信息。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11: Module header, imports, and shared constants
```python
import json
from collections.abc import Iterator
from pathlib import Path

# Must match SuffixAutomaton::kSeparatorToken in suffix_automaton.h.
SEPARATOR_TOKEN = -(2**31)

# Default chunk size for streaming tokenized documents into the SAM.
DEFAULT_CHUNK_SIZE = 4096
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 12-62: iter external corpus chunks function
```python
def iter_external_corpus_chunks(
    path: str, tokenizer, max_tokens: int, chunk_size: int = DEFAULT_CHUNK_SIZE
) -> Iterator[list[int]]:
    """Chunk documents and yield fixed-size token chunks from a JSONL corpus file."""
    corpus_path = Path(path)
    if not corpus_path.is_file():
        raise ValueError(f"External ngram corpus path does not exist: {path}")
    if tokenizer is None:
        raise ValueError("A tokenizer is required to load an external ngram corpus.")
    if max_tokens <= 0:
        raise ValueError("External ngram corpus max tokens must be positive.")

    total_tokens = 0
    has_previous_doc = False
    with corpus_path.open("r", encoding="utf-8") as f:
        for line_no, line in enumerate(f, start=1):
            if not line.strip():
                continue

            try:
                record = json.loads(line)
            except json.JSONDecodeError as e:
                raise ValueError(
                    f"Invalid JSON in external ngram corpus at line {line_no}: {e.msg}"
                ) from e

            if not isinstance(record, str):
                raise ValueError(
                    "Invalid external ngram corpus record at line "
                    f"{line_no}: expected a JSON string."
                )

            token_ids = list(tokenizer.encode(record, add_special_tokens=False))
            if not token_ids:
                continue

            separator_cost = 1 if has_previous_doc else 0
            next_total_tokens = total_tokens + separator_cost + len(token_ids)
            if next_total_tokens > max_tokens:
                raise ValueError(
                    "External ngram corpus exceeds the configured token limit "
                    f"({max_tokens}) at line {line_no} after loading "
                    f"{total_tokens} tokens."
                )
            total_tokens = next_total_tokens

            if has_previous_doc:
                token_ids = [SEPARATOR_TOKEN] + token_ids
            for i in range(0, len(token_ids), chunk_size):
                yield token_ids[i : i + chunk_size]
            has_previous_doc = True
```
**EN:** This block uses `iter_external_corpus_chunks` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `iter_external_corpus_chunks` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- Speculative decoding / 推测解码
- N-gram drafting / N-gram 起草
- Corpus management / 语料管理

## Dependencies / 依赖关系
### Internal / 内部
- None / 无
### External / 外部
- `collections` (stdlib)
- `json` (stdlib)
- `pathlib` (stdlib)
