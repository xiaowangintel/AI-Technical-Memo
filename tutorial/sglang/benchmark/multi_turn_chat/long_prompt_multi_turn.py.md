# long_prompt_multi_turn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/multi_turn_chat/long_prompt_multi_turn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on multi turn chat long prompt. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 multi turn chat long prompt 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Imports and setup / 导入与初始化
```python
import json
import random
import time
from argparse import ArgumentParser
from pathlib import Path

from tqdm import tqdm

import sglang as sgl
from sglang.srt.utils.hf_transformers_utils import get_tokenizer
from sglang.test.test_utils import (
    add_common_sglang_args_and_parse,
    select_sglang_backend,
)
from sglang.utils import dump_state_text
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, loads, filters, or serializes benchmark datasets, and processes tokenized prompts or decoded outputs.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。

### Lines 18-22: Function `gen_prompt` / 函数 `gen_prompt`
```python
def gen_prompt(tokenizer, token_num):
    all_available_tokens = list(tokenizer.get_vocab().values())
    selected_tokens = random.choices(all_available_tokens, k=token_num)
    ret = tokenizer.decode(selected_tokens)
    return ret
```
**EN:** `gen_prompt` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. It returns `ret` to the caller. Notable calls include `list`, `random.choices`, `tokenizer.decode`.
**CN:** `gen_prompt` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `ret`。其中较关键的调用包括 `list`, `random.choices`, `tokenizer.decode`。

### Lines 25-31: Function `get_cache_path` / 函数 `get_cache_path`
```python
def get_cache_path(args):
    # Create cache directory under ~/.cache/sglang
    cache_dir = Path.home() / ".cache" / "sglang"

    # Create a unique cache filename based on the arguments that affect generation
    cache_key = f"qa_{args.num_qa}_{args.turns}_{args.system_prompt_len}_{args.len_q}_{args.len_a}_{args.tokenizer.replace('/', '_')}.json"
    return cache_dir / cache_key
```
**EN:** `get_cache_path` is a function that loads, filters, or serializes benchmark datasets and processes tokenized prompts or decoded outputs. It returns `cache_dir / cache_key` to the caller. Notable calls include `Path.home`, `args.tokenizer.replace`.
**CN:** `get_cache_path` 是一个函数，用于加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。它会向调用方返回 `cache_dir / cache_key`。其中较关键的调用包括 `Path.home`, `args.tokenizer.replace`。

### Lines 34-68: Function `gen_arguments` / 函数 `gen_arguments`
```python
def gen_arguments(args, tokenizer):
    cache_path = get_cache_path(args)

    # Try to load from cache first
    if cache_path.exists():
        print(f"Loading cached arguments from {cache_path}")
        with open(cache_path, "r") as f:
            return json.load(f)

    print("Generating new arguments...")
    # First progress bar for system prompts
    multi_qas = []
    for _ in tqdm(range(args.num_qa), desc="Generating system prompts"):
        multi_qas.append(
            {"system_prompt": gen_prompt(tokenizer, args.system_prompt_len), "qas": []}
        )

    # Nested progress bars for QA pairs
    for i in tqdm(range(args.num_qa), desc="Generating QA pairs"):
        qas = multi_qas[i]["qas"]
        for j in range(args.turns):
            qas.append(
                {
                    "prompt": gen_prompt(tokenizer, args.len_q),
                    "new_tokens": args.len_a,
                }
            )

    # Save to cache
    cache_path.parent.mkdir(parents=True, exist_ok=True)
    with open(cache_path, "w") as f:
        json.dump(multi_qas, f)
    print(f"Cached arguments saved to {cache_path}")

    return multi_qas
```
**EN:** `gen_arguments` is a function that loads, filters, or serializes benchmark datasets, processes tokenized prompts or decoded outputs, and reads or writes local files and intermediate artifacts. It returns `multi_qas` to the caller. Notable calls include `get_cache_path`, `cache_path.exists`, `print`.
**CN:** `gen_arguments` 是一个函数，用于加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出、读写本地文件及中间产物。它会向调用方返回 `multi_qas`。其中较关键的调用包括 `get_cache_path`, `cache_path.exists`, `print`。

### Lines 72-77: Function `multi_turns` / 函数 `multi_turns`
```python
def multi_turns(s, system_prompt, qas):
    s += system_prompt

    for i, qa in enumerate(qas):
        s += qa["prompt"]
        s += sgl.gen(max_tokens=qa["new_tokens"], ignore_eos=True)
```
**EN:** `multi_turns` is a function that processes tokenized prompts or decoded outputs. Notable calls include `enumerate`, `sgl.gen`.
**CN:** `multi_turns` 是一个函数，用于处理分词后的提示词或解码后的输出。其中较关键的调用包括 `enumerate`, `sgl.gen`。

### Lines 80-112: Function `main` / 函数 `main`
```python
def main(args):
    tokenizer = get_tokenizer(args.tokenizer, trust_remote_code=args.trust_remote_code)

    multi_qas = gen_arguments(args, tokenizer)

    backend = select_sglang_backend(args)

    tic = time.perf_counter()
    states = multi_turns.run_batch(
        multi_qas,
        temperature=0,
        backend=backend,
        num_threads="auto",
        progress_bar=True,
    )
    latency = time.perf_counter() - tic

    print(f"Latency: {latency:.3f}")

    dump_state_text(f"tmp_output_{args.backend}.txt", states)

    with open(args.result_file, "a") as fout:
        value = {
            "task": "multi_turn_system_prompt_chat",
            "backend": args.backend,
            "latency": round(latency, 3),
            "num_requests": args.num_qa,
            "num_turns": args.turns,
            "other": {
                "parallel": args.parallel,
            },
        }
        fout.write(json.dumps(value) + "\n")
```
**EN:** `main` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and loads, filters, or serializes benchmark datasets. Notable calls include `get_tokenizer`, `gen_arguments`, `select_sglang_backend`.
**CN:** `main` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集。其中较关键的调用包括 `get_tokenizer`, `gen_arguments`, `select_sglang_backend`。

### Lines 115-129: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = ArgumentParser()
    parser.add_argument("--turns", type=int, default=8)
    parser.add_argument("--num-qa", type=int, default=128)
    parser.add_argument("--system-prompt-len", type=int, default=2048)
    parser.add_argument("--len-q", type=int, default=32)
    parser.add_argument("--len-a", type=int, default=128)
    parser.add_argument(
        "--tokenizer", type=str, default="meta-llama/Meta-Llama-3-8B-Instruct"
    )
    parser.add_argument("--trust-remote-code", action="store_true")
    args = add_common_sglang_args_and_parse(parser)

    print(args)
    main(args)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration and processes tokenized prompts or decoded outputs.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、处理分词后的提示词或解码后的输出。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `random`, `time`, `argparse`, `pathlib`
- **Third-party / 第三方依赖**: `tqdm`
- **Internal / 项目内部依赖**: `sglang`, `sglang.srt.utils.hf_transformers_utils`, `sglang.test.test_utils`, `sglang.utils`
