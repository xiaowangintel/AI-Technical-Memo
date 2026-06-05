# bench_other.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/hellaswag/bench_other.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on hellaswag other. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 hellaswag other 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Imports and setup / 导入与初始化
```python
import argparse
import asyncio
import json
import time
from concurrent.futures import ThreadPoolExecutor

import numpy as np
from tqdm import tqdm

from sglang.test.test_utils import add_common_other_args_and_parse, get_call_select
from sglang.utils import download_and_cache_file, read_jsonl
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, coordinates asynchronous or parallel execution, and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、协调异步或并行执行、加载、筛选或序列化基准测试数据集。

### Lines 14-18: Function `get_one_example` / 函数 `get_one_example`
```python
def get_one_example(lines, i, include_answer):
    ret = lines[i]["activity_label"] + ": " + lines[i]["ctx"] + " "
    if include_answer:
        ret += lines[i]["endings"][lines[i]["label"]]
    return ret
```
**EN:** `get_one_example` is a function that computes evaluation scores and aggregate statistics. It returns `ret` to the caller.
**CN:** `get_one_example` 是一个函数，用于计算评测分数与聚合统计结果。它会向调用方返回 `ret`。

### Lines 21-25: Function `get_few_shot_examples` / 函数 `get_few_shot_examples`
```python
def get_few_shot_examples(lines, k):
    ret = ""
    for i in range(k):
        ret += get_one_example(lines, i, True) + "\n\n"
    return ret
```
**EN:** `get_few_shot_examples` is a function that implements the core logic for this scope. It returns `ret` to the caller. Notable calls include `range`, `get_one_example`.
**CN:** `get_few_shot_examples` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `ret`。其中较关键的调用包括 `range`, `get_one_example`。

### Lines 28-109: Function `main` / 函数 `main`
```python
def main(args):
    # Select backend
    call_select = get_call_select(args)

    # Read data
    url = "https://raw.githubusercontent.com/rowanz/hellaswag/master/data/hellaswag_val.jsonl"
    filename = download_and_cache_file(url)
    lines = list(read_jsonl(filename))

    # Construct prompts
    num_questions = args.num_questions
    num_shots = args.num_shots
    few_shot_examples = get_few_shot_examples(lines, num_shots)

    questions = []
    choices = []
    labels = []
    for i in range(len(lines[:num_questions])):
        questions.append(get_one_example(lines, i, False))
        choices.append(lines[i]["endings"])
        labels.append(lines[i]["label"])

    preds = [None] * len(labels)

    # Run requests
    if args.backend != "lmql":
        # Use thread pool
        def get_one_answer(i):
            preds[i] = call_select(
                context=few_shot_examples + questions[i], choices=choices[i]
            )

        tic = time.perf_counter()
        if args.parallel == 1:
            for i in tqdm(range(len(questions))):
                get_one_answer(i)
        else:
            with ThreadPoolExecutor(args.parallel) as executor:
                list(
                    tqdm(
                        executor.map(get_one_answer, list(range(len(questions)))),
                        total=len(questions),
                    )
                )
    else:
        # Use asyncio
        async def batched_call(batch_size):
            for i in range(0, len(questions), batch_size):
                tasks = []
                for q, c in zip(
                    questions[i : i + batch_size], choices[i : i + batch_size]
                ):
                    tasks.append(call_select(context=few_shot_examples + q, choices=c))
                rets = await asyncio.gather(*tasks)
                for j in range(len(rets)):
                    preds[i + j] = rets[j]

        tic = time.perf_counter()
        asyncio.run(batched_call(batch_size=args.parallel))

    latency = time.perf_counter() - tic

    # Compute accuracy
    acc = np.mean(np.array(preds) == np.array(labels))
    print(f"Latency: {latency:.3f}")
    print(f"Accuracy: {acc:.3f}")

    # Write results
    with open(args.result_file, "a") as fout:
        value = {
            "task": "hellaswag",
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
**EN:** `main` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. Notable calls include `get_call_select`, `download_and_cache_file`, `list`.
**CN:** `main` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。其中较关键的调用包括 `get_call_select`, `download_and_cache_file`, `list`。

### Lines 112-118: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--num-shots", type=int, default=20)
    parser.add_argument("--data-path", type=str, default="hellaswag_val.jsonl")
    parser.add_argument("--num-questions", type=int, default=200)
    args = add_common_other_args_and_parse(parser)
    main(args)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、加载、筛选或序列化基准测试数据集。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Concurrency / 并发**: Coordinates multiple requests or workers at the same time. / 同时协调多个请求或工作单元。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `asyncio`, `json`, `time`, `concurrent.futures`
- **Third-party / 第三方依赖**: `numpy`, `tqdm`
- **Internal / 项目内部依赖**: `sglang.test.test_utils`, `sglang.utils`
