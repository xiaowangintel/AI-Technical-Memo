# bench_other.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/multi_chain_reasoning/bench_other.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on multi chain reasoning other. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 multi chain reasoning other 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Imports and setup / 导入与初始化
```python
import argparse
import ast
import asyncio
import json
import re
import time
from concurrent.futures import ThreadPoolExecutor

import numpy as np
from tqdm import tqdm

from sglang.test.test_utils import add_common_other_args_and_parse, get_call_generate
from sglang.utils import dump_state_text, read_jsonl

INVALID = -9999999
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, coordinates asynchronous or parallel execution, and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、协调异步或并行执行、加载、筛选或序列化基准测试数据集。

### Lines 18-26: Function `get_answer_value` / 函数 `get_answer_value`
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

### Lines 29-36: Top-level execution logic / 顶层执行逻辑
```python
prompt_lib = [
    "Let us think step by step.",
    "Approach this methodically. Let's dissect the problem into smaller, more manageable parts.",
    "It's important to proceed step by step, ensuring accuracy at each stage.",
    "Take a deep breath and break this down.",
    "A little bit of arithmetic and a logical approach will help us quickly arrive at the solution to this problem.",
    "I am extremely good at math.",
]
```
**EN:** This block performs top-level coordination such as preparing inputs, selecting modes, or invoking previously defined helpers. In practice it computes evaluation scores and aggregate statistics.
**CN:** 该代码块负责顶层协调，例如准备输入、选择模式或调用前面定义的辅助函数。 从实现上看，它会计算评测分数与聚合统计结果。

### Lines 39-62: Function `multi_chain_gsm8k` / 函数 `multi_chain_gsm8k`
```python
def multi_chain_gsm8k(question, num_chains, call_generate):
    s = "Question: " + question + "\n"
    # s += call_generate(s + "Answer: " + prompt_lib[0], max_tokens=256,
    #     stop="Question", temperature=0)
    # return s

    comps = []
    for i in range(num_chains):
        comps.append(
            call_generate(
                s + "Answer: " + prompt_lib[i % num_chains],
                max_tokens=256,
                temperature=0.3,
                stop="Question",
            )
        )

    s += "Answer: To answer this question, here are some possible solutions. "
    s += "After considering all of them, I will do a majority vote.\n\n"
    for i in range(num_chains):
        s += f"Solution {i+1}: " + comps[i].strip() + "\n\n"
    s += "\nBy considering the above solutions and doing a majority vote, I think the final answer (a single integer number) is "
    s += call_generate(s, max_tokens=16, temperature=0, stop=None)
    return s
```
**EN:** `multi_chain_gsm8k` is a function that processes tokenized prompts or decoded outputs and computes evaluation scores and aggregate statistics. It returns `s` to the caller. Notable calls include `range`, `call_generate`, `comps.append`.
**CN:** `multi_chain_gsm8k` 是一个函数，用于处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。它会向调用方返回 `s`。其中较关键的调用包括 `range`, `call_generate`, `comps.append`。

### Lines 65-88: Async function `multi_chain_gsm8k_async` / 异步函数 `multi_chain_gsm8k_async`
```python
async def multi_chain_gsm8k_async(question, num_chains, call_generate):
    s = "Question: " + question + "\n"
    # s += call_generate(s + "Answer: " + prompt_lib[0], max_tokens=256,
    #     stop="Question", temperature=0)
    # return s

    comps = []
    for i in range(num_chains):
        comps.append(
            await call_generate(
                s + "Answer: " + prompt_lib[i % num_chains],
                max_tokens=256,
                temperature=0.3,
                stop="Question",
            )
        )

    s += "Answer: To answer this question, here are some possible solutions. "
    s += "After considering all of them, I will do a majority vote.\n\n"
    for i in range(num_chains):
        s += f"Solution {i+1}: " + comps[i].strip() + "\n\n"
    s += "\nBy considering the above solutions and doing a majority vote, I think the final answer (a single integer number) is "
    s += await call_generate(s, max_tokens=16, temperature=0, stop=None)
    return s
```
**EN:** `multi_chain_gsm8k_async` is a async function that coordinates asynchronous or parallel execution, processes tokenized prompts or decoded outputs, and computes evaluation scores and aggregate statistics. It returns `s` to the caller. Notable calls include `range`, `comps.append`, `call_generate`.
**CN:** `multi_chain_gsm8k_async` 是一个异步函数，用于协调异步或并行执行、处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。它会向调用方返回 `s`。其中较关键的调用包括 `range`, `comps.append`, `call_generate`。

### Lines 91-176: Function `main` / 函数 `main`
```python
def main(args):
    lines = list(read_jsonl(args.data_path))

    # Construct prompts
    k = args.num_shot

    questions = []
    labels = []
    for i in range(len(lines[: args.num_questions])):
        questions.append(lines[i]["question"])
        labels.append(get_answer_value(lines[i]["answer"]))
    assert all(l != INVALID for l in labels)

    states = [None] * len(labels)

    # Select backend
    call_generate = get_call_generate(args)

    # Run requests
    if args.backend != "lmql":
        # Use thread pool
        def get_one_answer(i):
            answer = multi_chain_gsm8k(questions[i], args.num_chains, call_generate)
            states[i] = answer

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
        async def get_one_answer_asyncio(i):
            answer = await multi_chain_gsm8k_async(
                questions[i], args.num_chains, call_generate
            )
            states[i] = answer

        tic = time.perf_counter()
        loop = asyncio.get_event_loop()
        batches = [
            list(range(i, min(i + args.parallel, len(questions))))
            for i in range(0, len(questions), args.parallel)
        ]
        for bt in tqdm(batches):
            tasks = [get_one_answer_asyncio(k) for k in bt]
            loop.run_until_complete(asyncio.gather(*tasks))

    latency = time.perf_counter() - tic

    preds = []
    for i in range(len(states)):
        preds.append(get_answer_value(states[i]))

    # Compute accuracy
    acc = np.mean(np.array(preds) == np.array(labels))
    invalid = np.mean(np.array(preds) == INVALID)
    print(f"Latency: {latency:.3f}")
    print(f"Invalid: {invalid:.3f}")
    print(f"Accuracy: {acc:.3f}")

    # Write results
    dump_state_text(f"tmp_output_{args.backend}.txt", states)

    with open(args.result_file, "a") as fout:
        value = {
            "task": "multi_chain_gsm8k",
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
**EN:** `main` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. Notable calls include `list`, `range`, `all`.
**CN:** `main` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。其中较关键的调用包括 `list`, `range`, `all`。

### Lines 179-186: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--num-shot", type=int, default=0)
    parser.add_argument("--num-chains", type=int, default=5)
    parser.add_argument("--data-path", type=str, default="test.jsonl")
    parser.add_argument("--num-questions", type=int, default=50)
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
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `ast`, `asyncio`, `json`, `re`, `time`, `concurrent.futures`
- **Third-party / 第三方依赖**: `numpy`, `tqdm`
- **Internal / 项目内部依赖**: `sglang.test.test_utils`, `sglang.utils`
