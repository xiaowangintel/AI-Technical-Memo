# bench_sglang.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/boolq/bench_sglang.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on boolq sglang. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 boolq sglang 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Imports and setup / 导入与初始化
```python
import argparse
import json
import time

import numpy as np

from sglang.lang.api import set_default_backend
from sglang.test.test_utils import (
    add_common_sglang_args_and_parse,
    select_sglang_backend,
)
from sglang.utils import read_jsonl
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、加载、筛选或序列化基准测试数据集。

### Lines 15-19: Function `get_example` / 函数 `get_example`
```python
def get_example(lines, i, answer):
    prompt = "Question: " + lines[i]["question"] + lines[i]["passage"] + "\nAnswer:"
    if answer:
        prompt += str(lines[i]["answer"])
    return prompt
```
**EN:** `get_example` is a function that computes evaluation scores and aggregate statistics. It returns `prompt` to the caller. Notable calls include `str`.
**CN:** `get_example` 是一个函数，用于计算评测分数与聚合统计结果。它会向调用方返回 `prompt`。其中较关键的调用包括 `str`。

### Lines 22-26: Function `few_shot_examples` / 函数 `few_shot_examples`
```python
def few_shot_examples(lines, k):
    prompts = ""
    for i in range(k):
        prompts += get_example(lines, i, True) + "\n\n"
    return prompts
```
**EN:** `few_shot_examples` is a function that implements the core logic for this scope. It returns `prompts` to the caller. Notable calls include `range`, `get_example`.
**CN:** `few_shot_examples` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `prompts`。其中较关键的调用包括 `range`, `get_example`。

### Lines 29-108: Function `main` / 函数 `main`
```python
def main(args):
    # Select backend
    set_default_backend(select_sglang_backend(args))

    # Read data
    train_data_path = args.train_data_path
    test_data_path = args.test_data_path
    lines_train = list(read_jsonl(train_data_path))
    lines_test = list(read_jsonl(test_data_path))

    # Construct prompts
    num_questions = args.num_questions
    num_shots = args.num_shots
    few_shots = few_shot_examples(lines_train, num_shots)

    questions = []
    answer = []
    for i in range(len(lines_test[:num_questions])):
        questions.append(get_example(lines_test, i, False))
        answer.append(str(lines_test[i]["answer"]))
    arguments = [{"question": q} for q in questions]

    #####################################
    ######### SGL Program Begin #########
    #####################################

    import sglang as sgl

    @sgl.function
    def few_shot_boolq(s, question):
        s += few_shots + question
        s += sgl.gen("answer", max_tokens=5, stop=["\n"])

    #####################################
    ########## SGL Program End ##########
    #####################################

    # Run requests
    tic = time.perf_counter()
    states = few_shot_boolq.run_batch(
        arguments,
        temperature=0,
        num_threads=args.parallel,
        progress_bar=True,
    )
    latency = time.perf_counter() - tic

    preds = []
    for i in range(len(states)):
        preds.append(states[i]["answer"])

    # Compute accuracy
    acc = np.mean(np.array(preds) == np.array(answer))

    # Compute speed
    num_output_tokens = sum(
        s.get_meta_info("answer")["completion_tokens"] for s in states
    )
    output_throughput = num_output_tokens / latency

    # Print results
    print(f"Accuracy: {acc:.3f}")
    print(f"Latency: {latency:.3f} s")
    print(f"Output throughput: {output_throughput:.3f} token/s")

    # Results
    with open(args.result_file, "a") as fout:
        value = {
            "task": "boolq",
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
**EN:** `main` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and loads, filters, or serializes benchmark datasets. Notable calls include `set_default_backend`, `list`, `few_shot_examples`.
**CN:** `main` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集。其中较关键的调用包括 `set_default_backend`, `list`, `few_shot_examples`。

### Lines 111-124: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--num-shots", type=int, default=5)
    parser.add_argument(
        "--train-data-path", type=str, default="./boolq/data/train-00000-of-00001.json"
    )
    parser.add_argument(
        "--test-data-path",
        type=str,
        default="./boolq/data/validation-00000-of-00001.json",
    )
    parser.add_argument("--num-questions", type=int, default=200)
    args = add_common_sglang_args_and_parse(parser)
    main(args)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、加载、筛选或序列化基准测试数据集。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `json`, `time`
- **Third-party / 第三方依赖**: `numpy`
- **Internal / 项目内部依赖**: `sglang.lang.api`, `sglang.test.test_utils`, `sglang.utils`, `sglang`
