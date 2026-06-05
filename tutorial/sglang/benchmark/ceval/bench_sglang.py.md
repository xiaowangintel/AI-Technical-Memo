# bench_sglang.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/ceval/bench_sglang.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on ceval sglang. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 ceval sglang 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and setup / 导入与初始化
```python
import argparse
import json
import os
import random
import re
import time

import numpy as np
from datasets import load_dataset

from sglang.lang.api import set_default_backend
from sglang.test.test_utils import (
    add_common_sglang_args_and_parse,
    select_sglang_backend,
)

choices = ["A", "B", "C", "D"]
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、加载、筛选或序列化基准测试数据集。

### Lines 20-29: Function `get_one_example` / 函数 `get_one_example`
```python
def get_one_example(line, include_answer):
    res = line["question"]
    res += f"\nA. {line['A']}"
    res += f"\nB. {line['B']}"
    res += f"\nC. {line['C']}"
    res += f"\nD. {line['D']}"

    if include_answer:
        res += f"\nAnswer: {line['answer']} \n\n"
    return res
```
**EN:** `get_one_example` is a function that computes evaluation scores and aggregate statistics. It returns `res` to the caller.
**CN:** `get_one_example` 是一个函数，用于计算评测分数与聚合统计结果。它会向调用方返回 `res`。

### Lines 32-36: Function `get_few_shot_examples` / 函数 `get_few_shot_examples`
```python
def get_few_shot_examples(lines):
    res = ""
    for line in lines:
        res += get_one_example(line, True) + "\n\n"
    return res
```
**EN:** `get_few_shot_examples` is a function that implements the core logic for this scope. It returns `res` to the caller. Notable calls include `get_one_example`.
**CN:** `get_few_shot_examples` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `res`。其中较关键的调用包括 `get_one_example`。

### Lines 39-46: Function `get_answer_value` / 函数 `get_answer_value`
```python
def get_answer_value(response):
    pattern = r"(Answer:|answer:|答案是|答案是:|正确答案是:|答案:|Assistant:)\s*([A-D])(?![\w])"
    match = re.search(pattern, response)

    if match:
        return match.group(2)

    return random.choice(choices)
```
**EN:** `get_answer_value` is a function that computes evaluation scores and aggregate statistics. It returns `random.choice(choices)` to the caller. Notable calls include `re.search`, `random.choice`, `match.group`.
**CN:** `get_answer_value` 是一个函数，用于计算评测分数与聚合统计结果。它会向调用方返回 `random.choice(choices)`。其中较关键的调用包括 `re.search`, `random.choice`, `match.group`。

### Lines 49-130: Function `main` / 函数 `main`
```python
def main(args):
    # Read data && Construct prompts
    arguments = []
    labels = []
    examples = "examples:\n"
    data_path = args.data_path
    for subject in os.listdir(data_path):
        subject_path = os.path.join(data_path, subject)
        if os.path.isdir(subject_path) and subject != ".git":
            dataset = load_dataset(data_path, name=subject)
            dev_lines_temp = dataset["dev"]
            val_lines_temp = dataset["val"]
            few_shot_examples = get_few_shot_examples(dev_lines_temp)
            examples += f"{few_shot_examples}"
            for val_line in val_lines_temp:
                arguments.append(
                    {
                        "examples": few_shot_examples,
                        "question": get_one_example(val_line, False),
                    }
                )
                labels.append(val_line["answer"])

    #####################################
    ######### SGL Program Begin #########
    #####################################

    import sglang as sgl

    @sgl.function
    def few_shot_ceval(s, examples, question):
        s += examples + question + sgl.gen("Answer")

    #####################################
    ########## SGL Program End ##########
    #####################################

    num_questions = args.num_questions if args.num_questions else len(arguments)

    # Select backend
    set_default_backend(select_sglang_backend(args))

    # Run requests
    tic = time.perf_counter()
    states = few_shot_ceval.run_batch(
        arguments[:num_questions],
        temperature=0,
        num_threads=args.parallel,
        progress_bar=True,
    )
    latency = time.perf_counter() - tic

    preds = [get_answer_value(states[i]["Answer"]) for i in range(num_questions)]

    # Compute accuracy
    acc = np.mean(np.array(preds) == np.array(labels[:num_questions]))

    # Compute speed
    num_output_tokens = sum(
        s.get_meta_info("Answer")["completion_tokens"] for s in states
    )
    output_throughput = num_output_tokens / latency

    # Print results
    print(f"Accuracy: {acc:.3f}")
    print(f"Latency: {latency:.3f} s")
    print(f"Output throughput: {output_throughput:.3f} token/s")

    # Write results
    with open(args.result_file, "a") as fout:
        value = {
            "task": "ceval",
            "backend": args.backend,
            "num_gpus": 1,
            "latency": round(latency, 3),
            "accuracy": round(acc, 3),
            "num_requests": args.num_questions,
            "other": {
                "parallel": args.parallel,
            },
        }
        fout.write(json.dumps(value) + "\n")
```
**EN:** `main` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and loads, filters, or serializes benchmark datasets. Notable calls include `os.listdir`, `set_default_backend`, `time.perf_counter`.
**CN:** `main` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集。其中较关键的调用包括 `os.listdir`, `set_default_backend`, `time.perf_counter`。

### Lines 133-138: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--data-path", type=str, default="ceval/ceval-exam")
    parser.add_argument("--num-questions", type=int, default=None)
    args = add_common_sglang_args_and_parse(parser)
    main(args)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `json`, `os`, `random`, `re`, `time`
- **Third-party / 第三方依赖**: `numpy`, `datasets`
- **Internal / 项目内部依赖**: `sglang.lang.api`, `sglang.test.test_utils`, `sglang`
