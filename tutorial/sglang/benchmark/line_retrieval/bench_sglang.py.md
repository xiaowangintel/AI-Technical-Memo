# bench_sglang.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/line_retrieval/bench_sglang.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on line retrieval sglang. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 line retrieval sglang 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Imports and setup / 导入与初始化
```python
import argparse
import json
import re
import time

import numpy as np

import sglang as sgl
from sglang.test.test_utils import (
    add_common_sglang_args_and_parse,
    select_sglang_backend,
)
from sglang.utils import dump_state_text
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、加载、筛选或序列化基准测试数据集。

### Lines 17-27: Function `line_retrieval` / 函数 `line_retrieval`
```python
def line_retrieval(s, prefix, suffix, body_0, body_1, body_2, body_3):
    s += prefix + "\n"

    contexts = [body_0, body_1, body_2, body_3]
    position_ids_offset = [i * 1000 for i in range(len(contexts))]
    forks = s.fork(len(contexts), position_ids_offset)
    forks += lambda i: contexts[i] + "\n"
    forks.join(mode="concate_and_append")

    s += "\n" + suffix
    s += sgl.gen("answer", max_tokens=16)
```
**EN:** `line_retrieval` is a function that processes tokenized prompts or decoded outputs and computes evaluation scores and aggregate statistics. Notable calls include `s.fork`, `forks.join`, `sgl.gen`.
**CN:** `line_retrieval` 是一个函数，用于处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。其中较关键的调用包括 `s.fork`, `forks.join`, `sgl.gen`。

### Lines 30-128: Function `eval_model` / 函数 `eval_model`
```python
def eval_model(args, line_obj, num_hoops, src_indices, dst_percents):
    arguments = []
    labels = []
    sum_src_indices = []
    sum_dst_indices = []

    for i in range(len(src_indices)):
        for j in range(len(dst_percents)):
            src_index = src_indices[i]
            dst_percent = dst_percents[j]

            query_indices = line_obj["group_by_num_hoops"][str(num_hoops)]
            query_indices = [
                q
                for q in query_indices
                if all(l <= src_index for l in line_obj["links"][q]) and q < src_index
            ]
            dst_index = query_indices[
                min(int(len(query_indices) * dst_percent), len(query_indices) - 1)
            ]
            label = line_obj["values"][dst_index]

            body = line_obj["lines"][: src_index + 1]
            suffix = line_obj["suffix"].replace("???", line_obj["indices"][dst_index])
            body_part_len = len(body) // 4

            arguments.append(
                {
                    "prefix": line_obj["prefix"],
                    "body_0": "\n".join(body[:body_part_len]),
                    "body_1": "\n".join(body[body_part_len : 2 * body_part_len]),
                    "body_2": "\n".join(body[2 * body_part_len : 3 * body_part_len]),
                    "body_3": "\n".join(body[3 * body_part_len :]),
                    "suffix": suffix,
                }
            )
            labels.append(label)
            sum_src_indices.append(src_index)
            sum_dst_indices.append(dst_index)

    # Select backend
    backend = select_sglang_backend(args)

    tic = time.perf_counter()
    states = line_retrieval.run_batch(
        arguments,
        temperature=0,
        backend=backend,
        num_threads=args.parallel,
        progress_bar=True,
    )
    latency = time.perf_counter() - tic

    corrects = []
    for i in range(len(arguments)):
        output = states[i]["answer"]
        prompt_len = states[i].get_meta_info("answer").get("prompt_length", -1)
        label = labels[i]

        # Try all numbers
        findall = re.findall("\d+", output)
        if not findall:
            response_number = output
        else:
            for response_number in findall:
                if response_number == label:
                    break

        correct = response_number == label
        corrects.append(correct)

        # Log results
        summary = (
            f"Line index: {sum_src_indices[i]} -> {sum_dst_indices[i]}, "
            f"Prompt len: {prompt_len}, "
            f"Correct: {correct}, "
            f"Label: {label}, Predicted: {response_number}, "
        )
        print(summary)

    accuracy = np.mean(corrects)
    print(f"Accuracy: {accuracy:.3f}, latency: {latency:.2f} s")

    # Write results
    dump_state_text(f"tmp_output_{args.backend}.txt", states)

    with open(args.result_file, "a") as fout:
        value = {
            "task": "line_retrieval",
            "backend": args.backend,
            "num_gpus": 1,
            "latency": round(latency, 3),
            "num_requests": len(arguments),
            "other": {
                "num_questions": len(arguments),
                "parallel": args.parallel,
            },
        }
        fout.write(json.dumps(value) + "\n")
```
**EN:** `eval_model` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and loads, filters, or serializes benchmark datasets. Notable calls include `range`, `select_sglang_backend`, `time.perf_counter`.
**CN:** `eval_model` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集。其中较关键的调用包括 `range`, `select_sglang_backend`, `time.perf_counter`。

### Lines 131-139: Function `main` / 函数 `main`
```python
def main(args):
    line_obj = json.load(open(args.data_path, "r"))

    num_hoops = args.num_hoops
    for src_index in args.src_index:
        src_indices = [src_index]
        num_queries = args.num_queries_per_src
        dst_percents = [i * (1 / (num_queries)) for i in range(num_queries)]
        eval_model(args, line_obj, num_hoops, src_indices, dst_percents)
```
**EN:** `main` is a function that loads, filters, or serializes benchmark datasets and reads or writes local files and intermediate artifacts. Notable calls include `json.load`, `open`, `eval_model`.
**CN:** `main` 是一个函数，用于加载、筛选或序列化基准测试数据集、读写本地文件及中间产物。其中较关键的调用包括 `json.load`, `open`, `eval_model`。

### Lines 142-149: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--data-path", type=str, default="lines_1000_0.0.json")
    parser.add_argument("--src-index", type=int, nargs="+", default=[100])
    parser.add_argument("--num-queries-per-src", type=int, default=10)
    parser.add_argument("--num-hoops", type=int, default=1)
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
- **Standard library / 标准库**: `argparse`, `json`, `re`, `time`
- **Third-party / 第三方依赖**: `numpy`
- **Internal / 项目内部依赖**: `sglang`, `sglang.test.test_utils`, `sglang.utils`
