# bench_sglang.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/gsm8k/bench_sglang.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on gsm8k sglang. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 gsm8k sglang 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Imports and setup / 导入与初始化
```python
import argparse
import ast
import json
import os
import re
import time

import numpy as np
from datasets import load_dataset

from sglang.lang.api import set_default_backend
from sglang.test.test_utils import (
    add_common_sglang_args_and_parse,
    dump_bench_raw_result,
    select_sglang_backend,
)
from sglang.utils import download_and_cache_file, dump_state_text, read_jsonl

INVALID = -9999999
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、加载、筛选或序列化基准测试数据集。

### Lines 22-26: Function `get_one_example` / 函数 `get_one_example`
```python
def get_one_example(lines, i, include_answer):
    ret = "Question: " + lines[i]["question"] + "\nAnswer:"
    if include_answer:
        ret += " " + lines[i]["answer"]
    return ret
```
**EN:** `get_one_example` is a function that computes evaluation scores and aggregate statistics. It returns `ret` to the caller.
**CN:** `get_one_example` 是一个函数，用于计算评测分数与聚合统计结果。它会向调用方返回 `ret`。

### Lines 29-33: Function `get_few_shot_examples` / 函数 `get_few_shot_examples`
```python
def get_few_shot_examples(lines, k):
    ret = ""
    for i in range(k):
        ret += get_one_example(lines, i, True) + "\n\n"
    return ret
```
**EN:** `get_few_shot_examples` is a function that implements the core logic for this scope. It returns `ret` to the caller. Notable calls include `range`, `get_one_example`.
**CN:** `get_few_shot_examples` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `ret`。其中较关键的调用包括 `range`, `get_one_example`。

### Lines 36-44: Function `get_answer_value` / 函数 `get_answer_value`
```python
def get_answer_value(answer_str):
    answer_str = answer_str.replace(",", "")
    numbers = re.findall(r"\d+", answer_str)
    if len(numbers) < 1:
        return INVALID
    try:
        return ast.literal_eval(numbers[-1])
    except SyntaxError:
        return INVALID
```
**EN:** `get_answer_value` is a function that computes evaluation scores and aggregate statistics. It returns `INVALID` to the caller. Notable calls include `answer_str.replace`, `re.findall`, `len`.
**CN:** `get_answer_value` 是一个函数，用于计算评测分数与聚合统计结果。它会向调用方返回 `INVALID`。其中较关键的调用包括 `answer_str.replace`, `re.findall`, `len`。

### Lines 47-171: Function `main` / 函数 `main`
```python
def main(args):
    # Select backend
    set_default_backend(select_sglang_backend(args))

    # Load tokenizer if enable_thinking is set
    tokenizer = None
    if args.enable_thinking:
        from transformers import AutoTokenizer

        assert (
            args.tokenizer_path is not None
        ), "--tokenizer-path is required when --enable-thinking is set"
        tokenizer = AutoTokenizer.from_pretrained(
            args.tokenizer_path, trust_remote_code=True
        )

    # Read data
    if args.platinum:
        print("Loading GSM8K Platinum dataset from HuggingFace...")
        dataset = load_dataset("madrylab/gsm8k-platinum", "main", split="test")
        lines = [
            {"question": item["question"], "answer": item["answer"]} for item in dataset
        ]
    else:
        data_path = args.data_path
        url = "https://raw.githubusercontent.com/openai/grade-school-math/master/grade_school_math/data/test.jsonl"
        if not os.path.isfile(data_path):
            data_path = download_and_cache_file(url)
        lines = list(read_jsonl(data_path))

    # Construct prompts
    num_questions = args.num_questions
    num_shots = args.num_shots
    few_shot_examples = get_few_shot_examples(lines, num_shots)

    questions = []
    labels = []
    for i in range(len(lines[:num_questions])):
        raw_question = few_shot_examples + get_one_example(lines, i, False)
        if tokenizer is not None:
            messages = [{"role": "user", "content": raw_question}]
            raw_question = tokenizer.apply_chat_template(
                messages,
                tokenize=False,
                add_generation_prompt=True,
                enable_thinking=True,
            )
        questions.append(raw_question)
        labels.append(get_answer_value(lines[i]["answer"]))
    assert all(l != INVALID for l in labels)
    arguments = [{"question": q} for q in questions]

    #####################################
    ######### SGL Program Begin #########
    #####################################

    import sglang as sgl

    @sgl.function
    def few_shot_gsm8k(s, question):
        s += question
        s += sgl.gen(
            "answer",
            max_tokens=args.max_new_tokens,
            stop=["Question", "Assistant:", "<|separator|>"],
        )

    #####################################
    ########## SGL Program End ##########
    #####################################

    # Run requests
    tic = time.perf_counter()
    states = few_shot_gsm8k.run_batch(
        arguments,
        temperature=args.temperature,
        top_p=args.top_p,
        num_threads=args.parallel,
        progress_bar=True,
    )
    latency = time.perf_counter() - tic

    preds = []
    for i in range(len(states)):
        preds.append(get_answer_value(states[i]["answer"]))

    # Compute accuracy
    acc = np.mean(np.array(preds) == np.array(labels))
    invalid = np.mean(np.array(preds) == INVALID)

    # Compute speed
    num_output_tokens = sum(
        s.get_meta_info("answer")["completion_tokens"] for s in states
    )
    output_throughput = num_output_tokens / latency

    # Print results
    print(f"Accuracy: {acc:.3f}")
    print(f"Invalid: {invalid:.3f}")
    print(f"Latency: {latency:.3f} s")
    print(f"Output throughput: {output_throughput:.3f} token/s")

    # Dump results
    dump_state_text(f"tmp_output_{args.backend}.txt", states)
    dump_bench_raw_result(
        path=args.raw_result_file,
        states=states,
        preds=preds,
        labels=labels,
    )

    with open(args.result_file, "a") as fout:
        value = {
            "task": "gsm8k-platinum" if args.platinum else "gsm8k",
            "backend": args.backend,
            "num_gpus": 1,
            "latency": round(latency, 3),
            "accuracy": round(acc, 3),
            "num_requests": args.num_questions,
            "other": {
                "num_questions": args.num_questions,
                "parallel": args.parallel,
            },
        }
        fout.write(json.dumps(value) + "\n")
```
**EN:** `main` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and loads, filters, or serializes benchmark datasets. Notable calls include `set_default_backend`, `get_few_shot_examples`, `range`.
**CN:** `main` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集。其中较关键的调用包括 `set_default_backend`, `get_few_shot_examples`, `range`。

### Lines 174-199: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--num-shots", type=int, default=5)
    parser.add_argument("--data-path", type=str, default="test.jsonl")
    parser.add_argument("--num-questions", type=int, default=200)
    parser.add_argument("--max-new-tokens", type=int, default=512)
    parser.add_argument("--temperature", type=float, default=0.0)
    parser.add_argument("--top-p", type=float, default=1.0)
    parser.add_argument(
        "--enable-thinking",
        action="store_true",
        help="Enable thinking mode by wrapping prompts with chat template",
    )
    parser.add_argument(
        "--tokenizer-path",
        type=str,
        default=None,
        help="Path to tokenizer (required when --enable-thinking is set)",
    )
    parser.add_argument(
        "--platinum",
        action="store_true",
        help="Use GSM8K Platinum dataset (drop-in replacement with corrected labels)",
    )
    args = add_common_sglang_args_and_parse(parser)
    main(args)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration, loads, filters, or serializes benchmark datasets, and processes tokenized prompts or decoded outputs.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `ast`, `json`, `os`, `re`, `time`
- **Third-party / 第三方依赖**: `numpy`, `datasets`, `transformers`
- **Internal / 项目内部依赖**: `sglang.lang.api`, `sglang.test.test_utils`, `sglang.utils`, `sglang`
