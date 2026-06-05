# convert_sharegpt_to_openai.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/multi_turn/convert_sharegpt_to_openai.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, centered around `has_non_english_chars`, `content_is_valid`, `print_stats`, `convert_sharegpt_to_openai`. / 实现与基准测试编排相关的逻辑，核心符号包括 `has_non_english_chars`, `content_is_valid`, `print_stats`, `convert_sharegpt_to_openai`。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 3-10)
```python
"""
Download dataset from:
https://huggingface.co/datasets/philschmid/sharegpt-raw/blob/main/sharegpt_20230401_clean_lang_split.json

Convert to OpenAI API:
export INPUT_FILE=sharegpt_20230401_clean_lang_split.json
python convert_sharegpt_to_openai.py $INPUT_FILE sharegpt_conv_128.json --max-items=128
"""
```
**EN:** The module docstring introduces Download dataset from: https://huggingface.co/datasets/philschmid/sharegpt-raw/blob/main/sharegpt_20230401_clean_lang_split.json Convert to OpenAI API: export INPUT_FILE=sharegpt_20230401_clean_lang_split.json python convert_sharegpt_to_openai.py $INPUT_FILE sharegpt_conv_128.json --max-items=128. It gives readers the context needed before the implementation details start.
**CN:** 模块文档字符串先说明了 Download dataset from: https://huggingface.co/datasets/philschmid/sharegpt-raw/blob/main/sharegpt_20230401_clean_lang_split.json Convert to OpenAI API: export INPUT_FILE=sharegpt_20230401_clean_lang_split.json python convert_sharegpt_to_openai.py $INPUT_FILE sharegpt_conv_128.json --max-items=128 的背景与用途，在进入实现细节前先建立阅读上下文。

### Imports (lines 12-20)
```python
import argparse
import json
import random
from statistics import mean
from typing import Any

import pandas as pd  # type: ignore
import tqdm  # type: ignore
from transformers import AutoTokenizer  # type: ignore
```
**EN:** This block gathers standard-library helpers such as `argparse`, `json`, `random`, `statistics`, `typing`; third-party packages such as `pandas`, `tqdm`, `transformers`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`, `json`, `random`, `statistics`, `typing`；第三方依赖，如 `pandas`, `tqdm`, `transformers`。这些依赖构成了后续基准测试逻辑的基础。

### Function `has_non_english_chars` (lines 23-24)
```python
def has_non_english_chars(text: str) -> bool:
    return not text.isascii()
```
**EN:** `has_non_english_chars` implements a helper used by `convert_sharegpt_to_openai.py`. It mainly works with `text` and relies on `text.isascii` plus value production to move data through this part of the benchmark pipeline.
**CN:** `has_non_english_chars` 负责实现 `convert_sharegpt_to_openai.py` 使用的辅助逻辑。 它主要处理 `text`，并结合 `text.isascii` 以及 结果返回 来完成这一段基准测试流程。

### Function `content_is_valid` (lines 27-36)
```python
def content_is_valid(
    content: str, min_content_len: int | None, max_content_len: int | None
) -> bool:
    if min_content_len and len(content) < min_content_len:
        return False

    if max_content_len and len(content) > max_content_len:
        return False

    return has_non_english_chars(content)
```
**EN:** `content_is_valid` implements a helper used by `convert_sharegpt_to_openai.py`. It mainly works with `content`, `min_content_len`, `max_content_len` and relies on `len`, `has_non_english_chars` plus branching to move data through this part of the benchmark pipeline.
**CN:** `content_is_valid` 负责实现 `convert_sharegpt_to_openai.py` 使用的辅助逻辑。 它主要处理 `content`, `min_content_len`, `max_content_len`，并结合 `len`, `has_non_english_chars` 以及 条件分支 来完成这一段基准测试流程。

### Function `print_stats` (lines 39-105)
```python
def print_stats(
    conversations: "list[dict[Any, Any]]", tokenizer: AutoTokenizer | None = None
) -> None:
    # Collect statistics
    stats = []

    print("\nCollecting statistics...")
    for item in tqdm.tqdm(conversations):
        # item has "id" and "messages"
        messages = item["messages"]

        user_turns = 0
        assistant_turns = 0
        user_words = 0
        assistant_words = 0
        conv_chars = 0

        user_tokens: list[int] = []
        assistant_tokens: list[int] = []

        for m in messages:
            content = m["content"]
            conv_chars += len(content)
            content_num_words = content.count(" ") + 1

            num_tokens = 0
            if tokenizer:
                num_tokens = len(tokenizer(m["content"]).input_ids)

            if m["role"] == "user":
                user_turns += 1
                user_words += content_num_words
                if tokenizer:
                    user_tokens.append(num_tokens)
    # ... omitted for brevity ...
            item_stats["assistant_tokens"] = int(mean(assistant_tokens))

        stats.append(item_stats)

    print("\nStatistics:")
    percentiles = [0.25, 0.5, 0.75, 0.9, 0.99, 0.999, 0.9999]
    df = pd.DataFrame(stats)
    print(df.describe(percentiles=percentiles).transpose())
```
**EN:** `print_stats` formats results for display or export. It mainly works with `conversations`, `tokenizer` and relies on `print`, `tqdm.tqdm`, `len`, `content.count`, `tokenizer`, `user_tokens.append` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `print_stats` 负责格式化结果以便展示或导出。 它主要处理 `conversations`, `tokenizer`，并结合 `print`, `tqdm.tqdm`, `len`, `content.count`, `tokenizer`, `user_tokens.append` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `convert_sharegpt_to_openai` (lines 108-287)
```python
def convert_sharegpt_to_openai(
    seed: int,
    input_file: str,
    output_file: str,
    max_items: int | None,
    min_content_len: int | None = None,
    max_content_len: int | None = None,
    min_turns: int | None = None,
    max_turns: int | None = None,
    model: str | None = None,
) -> None:
    if min_turns and max_turns:
        assert min_turns <= max_turns

    if min_content_len and max_content_len:
        # Verify that min is not larger than max if both were given
        assert min_content_len <= max_content_len

    print(
        f"Input parameters:\n{seed=}, {max_items=}, {min_content_len=},"
        f" {max_content_len=}, {min_turns=}, {max_turns=}\n"
    )

    random.seed(seed)

    tokenizer = None
    if model is not None:
        print(f"Loading tokenizer from: {model}")
        tokenizer = AutoTokenizer.from_pretrained(model)

    # Read the ShareGPT JSON file
    print(f"Reading file: {input_file}")
    with open(input_file, encoding="utf-8") as f:
        # Should be a list of dicts
    # ... omitted for brevity ...
        print_stats(final_openai_dataset, tokenizer)

    # Write the converted data to a new JSON file
    final_size = len(final_openai_dataset)
    print(f"\nTotal conversations converted (after filtering): {final_size:,}")
    print(f"\nWriting file: {output_file}")
    with open(output_file, "w", encoding="utf-8") as f:
        json.dump(final_openai_dataset, f, ensure_ascii=False, indent=2)
```
**EN:** `convert_sharegpt_to_openai` implements a helper used by `convert_sharegpt_to_openai.py`. It mainly works with `seed`, `input_file`, `output_file`, `max_items`, `min_content_len`, ... and relies on `print`, `random.seed`, `AutoTokenizer.from_pretrained`, `open`, `json.load`, `isinstance` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `convert_sharegpt_to_openai` 负责实现 `convert_sharegpt_to_openai.py` 使用的辅助逻辑。 它主要处理 `seed`, `input_file`, `output_file`, `max_items`, `min_content_len`, ...，并结合 `print`, `random.seed`, `AutoTokenizer.from_pretrained`, `open`, `json.load`, `isinstance` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Function `main` (lines 290-350)
```python
def main() -> None:
    parser = argparse.ArgumentParser(
        description="Convert ShareGPT dataset to OpenAI API format"
    )
    parser.add_argument("input_file", help="Path to the input ShareGPT JSON file")
    parser.add_argument(
        "output_file", help="Path to the output OpenAI format JSON file"
    )
    parser.add_argument(
        "--seed", type=int, default=0, help="Seed for random number generators"
    )
    parser.add_argument(
        "--max-items",
        type=int,
        default=None,
        help="Maximum number of items in the output file",
    )
    parser.add_argument(
        "--min-turns",
        type=int,
        default=None,
        help="Minimum number of turns per conversation",
    )
    parser.add_argument(
        "--max-turns",
        type=int,
        default=None,
        help="Maximum number of turns per conversation",
    )
    parser.add_argument(
        "--min-content-len",
        type=int,
        default=None,
        help="Min number of characters in the messages' content",
    # ... omitted for brevity ...
        args.output_file,
        args.max_items,
        args.min_content_len,
        args.max_content_len,
        args.min_turns,
        args.max_turns,
        args.model,
    )
```
**EN:** `main` serves as the main entry point for this script. It mainly works with no explicit parameters and relies on `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `convert_sharegpt_to_openai` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 无显式参数，并结合 `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `convert_sharegpt_to_openai` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Entry point (lines 353-354)
```python
if __name__ == "__main__":
    main()
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `main` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `main` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Serving path benchmarking: the script interacts with HTTP services, servers, or request proxies.
- **CN:** 服务路径基准：脚本会与 HTTP 服务、服务器或请求代理交互。
- **EN:** Result presentation: the module aggregates measurements and renders tables or plots.
- **CN:** 结果呈现：该模块会汇总测量结果并生成表格或图形。
- **EN:** Dataset handling: benchmark inputs are loaded, normalized, or synthesized before execution.
- **CN:** 数据集处理：基准输入会在执行前被加载、规范化或合成。

## Dependencies / 依赖关系
- **EN:** Standard library: `argparse`, `json`, `random`, `statistics`, `typing`.
- **CN:** 标准库依赖：`argparse`, `json`, `random`, `statistics`, `typing`。
- **EN:** Third-party packages: `pandas`, `tqdm`, `transformers`.
- **CN:** 第三方依赖：`pandas`, `tqdm`, `transformers`。
- **EN:** Internal modules: none.
- **CN:** 内部模块：无。
