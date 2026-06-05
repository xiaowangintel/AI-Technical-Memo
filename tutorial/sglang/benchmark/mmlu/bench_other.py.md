# bench_other.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/mmlu/bench_other.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on mmlu other. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 mmlu other 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and setup / 导入与初始化
```python
import argparse
import asyncio
import json
import os
import time
from concurrent.futures import ThreadPoolExecutor

import numpy as np
import pandas as pd
import tiktoken
from tqdm import tqdm

from sglang.test.test_utils import add_common_other_args_and_parse, get_call_generate

choices = ["A", "B", "C", "D"]

tokenizer = tiktoken.encoding_for_model("gpt-3.5-turbo")
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, coordinates asynchronous or parallel execution, and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、协调异步或并行执行、加载、筛选或序列化基准测试数据集。

### Lines 20-25: Function `format_subject` / 函数 `format_subject`
```python
def format_subject(subject):
    l = subject.split("_")
    s = ""
    for entry in l:
        s += " " + entry
    return s
```
**EN:** `format_subject` is a function that implements the core logic for this scope. It returns `s` to the caller. Notable calls include `subject.split`.
**CN:** `format_subject` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `s`。其中较关键的调用包括 `subject.split`。

### Lines 28-36: Function `format_example` / 函数 `format_example`
```python
def format_example(df, idx, include_answer=True):
    prompt = df.iloc[idx, 0]
    k = df.shape[1] - 2
    for j in range(k):
        prompt += "\n{}. {}".format(choices[j], df.iloc[idx, j + 1])
    prompt += "\nAnswer:"
    if include_answer:
        prompt += " {}\n\n".format(df.iloc[idx, k + 1])
    return prompt
```
**EN:** `format_example` is a function that computes evaluation scores and aggregate statistics. It returns `prompt` to the caller. Notable calls include `range`, `'\n{}. {}'.format`, `' {}\n\n'.format`.
**CN:** `format_example` 是一个函数，用于计算评测分数与聚合统计结果。它会向调用方返回 `prompt`。其中较关键的调用包括 `range`, `'\n{}. {}'.format`, `' {}\n\n'.format`。

### Lines 39-47: Function `gen_prompt` / 函数 `gen_prompt`
```python
def gen_prompt(train_df, subject, k=-1):
    prompt = "The following are multiple choice questions (with answers) about{}.\n\n".format(
        format_subject(subject)
    )
    if k == -1:
        k = train_df.shape[0]
    for i in range(k):
        prompt += format_example(train_df, i)
    return prompt
```
**EN:** `gen_prompt` is a function that computes evaluation scores and aggregate statistics. It returns `prompt` to the caller. Notable calls include `'The following are multiple choice questions (with answers) about{}.\n\n'.format`, `range`, `format_subject`.
**CN:** `gen_prompt` 是一个函数，用于计算评测分数与聚合统计结果。它会向调用方返回 `prompt`。其中较关键的调用包括 `'The following are multiple choice questions (with answers) about{}.\n\n'.format`, `range`, `format_subject`。

### Lines 50-112: Function `evaluate` / 函数 `evaluate`
```python
def evaluate(args, subject, dev_df, test_df, call_generate):
    prompts = []
    labels = []

    # Construct prompts
    k = args.ntrain
    train_prompt = gen_prompt(dev_df, subject, k)
    while len(tokenizer.encode(train_prompt)) > 1536:
        k -= 1
        train_prompt = gen_prompt(dev_df, subject, k)

    for i in range(test_df.shape[0]):
        prompt_end = format_example(test_df, i, include_answer=False)
        prompt = train_prompt + prompt_end
        prompts.append(prompt)

        label = test_df.iloc[i, test_df.shape[1] - 1]
        labels.append(label)

    preds = [None] * len(prompts)
    max_tokens = 1

    # Run requests
    if args.backend != "lmql":
        # Use thread pool
        def get_one_answer(i):
            pred = call_generate(prompts[i], temperature=0, max_tokens=max_tokens)
            preds[i] = pred.strip()[0]

        tic = time.perf_counter()
        if args.parallel == 1:
            for i in range(len(prompts)):
                get_one_answer(i)
        else:
            with ThreadPoolExecutor(args.parallel) as executor:
                executor.map(get_one_answer, list(range(len(prompts))))
    else:
        # Use asyncio
        async def batched_call(batch_size):
            for i in range(0, len(prompts), batch_size):
                tasks = []
                for p in prompts[i : i + batch_size]:
                    tasks.append(call_generate(p, temperature=0, max_tokens=max_tokens))
                rets = await asyncio.gather(*tasks)
                for j in range(len(rets)):
                    preds[i + j] = rets[j].strip()[0]

        tic = time.perf_counter()
        asyncio.run(batched_call(batch_size=args.parallel))
    latency = time.perf_counter() - tic

    # Compute accuracy
    cors = [pred == label for pred, label in zip(preds, labels)]
    acc = np.mean(cors)
    cors = np.array(cors)

    print(
        "Average accuracy {:.3f}, latency {:.2f}, #q: {} - {}".format(
            acc, latency, len(prompts), subject
        )
    )

    return cors, acc, latency
```
**EN:** `evaluate` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. It returns `(cors, acc, latency)` to the caller. Notable calls include `gen_prompt`, `range`, `np.mean`.
**CN:** `evaluate` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。它会向调用方返回 `(cors, acc, latency)`。其中较关键的调用包括 `gen_prompt`, `range`, `np.mean`。

### Lines 115-164: Function `main` / 函数 `main`
```python
def main(args):
    subjects = sorted(
        [
            f.split("_test.csv")[0]
            for f in os.listdir(os.path.join(args.data_dir, "test"))
            if "_test.csv" in f
        ]
    )

    all_cors = []
    all_latencies = []
    num_requests = 0

    # Select backend
    call_generate = get_call_generate(args)

    for subject in tqdm(subjects[: args.nsub]):
        dev_df = pd.read_csv(
            os.path.join(args.data_dir, "dev", subject + "_dev.csv"), header=None
        )[: args.ntrain]
        test_df = pd.read_csv(
            os.path.join(args.data_dir, "test", subject + "_test.csv"), header=None
        )

        cors, acc, latency = evaluate(args, subject, dev_df, test_df, call_generate)
        all_cors.append(cors)
        all_latencies.append(latency)
        num_requests += len(test_df)

    total_latency = np.sum(all_latencies)
    print("Total latency: {:.3f}".format(total_latency))

    weighted_acc = np.mean(np.concatenate(all_cors))
    print("Average accuracy: {:.3f}".format(weighted_acc))

    # Write results
    with open(args.result_file, "a") as fout:
        value = {
            "task": "mmlu",
            "backend": args.backend,
            "num_gpus": 1,
            "latency": round(total_latency, 3),
            "accuracy": round(weighted_acc, 3),
            "num_requests": num_requests,
            "other": {
                "nsub": args.nsub,
                "parallel": args.parallel,
            },
        }
        fout.write(json.dumps(value) + "\n")
```
**EN:** `main` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and loads, filters, or serializes benchmark datasets. Notable calls include `sorted`, `get_call_generate`, `tqdm`.
**CN:** `main` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集。其中较关键的调用包括 `sorted`, `get_call_generate`, `tqdm`。

### Lines 167-173: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--ntrain", type=int, default=5)
    parser.add_argument("--data_dir", type=str, default="data")
    parser.add_argument("--nsub", type=int, default=60)
    args = add_common_other_args_and_parse(parser)
    main(args)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Concurrency / 并发**: Coordinates multiple requests or workers at the same time. / 同时协调多个请求或工作单元。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `asyncio`, `json`, `os`, `time`, `concurrent.futures`
- **Third-party / 第三方依赖**: `numpy`, `pandas`, `tiktoken`, `tqdm`
- **Internal / 项目内部依赖**: `sglang.test.test_utils`
