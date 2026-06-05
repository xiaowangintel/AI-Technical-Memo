# bench_sglang.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/llava_bench/bench_sglang.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on llava sglang. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 llava sglang 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Imports and setup / 导入与初始化
```python
import argparse
import json
import os
import time

import tqdm

import sglang as sgl
from sglang.test.test_utils import (
    add_common_sglang_args_and_parse,
    select_sglang_backend,
)
from sglang.utils import dump_state_text, read_jsonl
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、加载、筛选或序列化基准测试数据集。

### Lines 17-19: Function `image_qa` / 函数 `image_qa`
```python
def image_qa(s, image_file, question):
    s += sgl.user(sgl.image(image_file) + question)
    s += sgl.assistant(sgl.gen("answer", max_tokens=args.max_tokens))
```
**EN:** `image_qa` is a function that processes tokenized prompts or decoded outputs and computes evaluation scores and aggregate statistics. Notable calls include `sgl.user`, `sgl.assistant`, `sgl.gen`.
**CN:** `image_qa` 是一个函数，用于处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。其中较关键的调用包括 `sgl.user`, `sgl.assistant`, `sgl.gen`。

### Lines 22-84: Function `main` / 函数 `main`
```python
def main(args):
    lines = list(read_jsonl(args.question_file))[: args.num_questions]
    arguments = [
        {
            "image_file": os.path.abspath(args.image_folder + "/" + l["image"]),
            "question": l["text"],
        }
        for l in lines
    ]
    # arguments = [
    #    {"image_file":
    #        Image.open(os.path.abspath(args.image_folder + "/" + l["image"])),
    #      "question": l["text"]} for l in lines
    # ]

    states = [None] * len(lines)

    # Select backend
    backend = select_sglang_backend(args)
    sgl.set_default_backend(backend)

    # Run requests
    tic = time.perf_counter()
    if args.parallel == 1:
        for i in tqdm.tqdm(range(len(lines))):
            image_file = arguments[i]["image_file"]
            question = arguments[i]["question"]
            ret = image_qa.run(image_file=image_file, question=question, temperature=0)
            states[i] = ret
    else:
        states = image_qa.run_batch(
            arguments, temperature=0, num_threads=args.parallel, progress_bar=True
        )
    latency = time.perf_counter() - tic

    print(f"Latency: {latency:.3f}")

    # Write results
    dump_state_text(f"tmp_output_{args.backend}.txt", states)

    print(f"Write output to {args.answer_file}")
    with open(args.answer_file, "w") as fout:
        for i in range(len(lines)):
            value = {
                "question_id": lines[i]["question_id"],
                "prompt": lines[i]["text"],
                "text": states[i]["answer"].strip(),
                "model_id": backend.model_info["model_path"],
                "answer_id": i,
                "metadata": {},
            }
            fout.write(json.dumps(value) + "\n")

    with open(args.result_file, "a") as fout:
        value = {
            "task": "llava_bench",
            "backend": args.backend,
            "num_gpus": 1,
            "latency": round(latency, 3),
            "num_requests": len(lines),
            "parallel": args.parallel,
        }
        fout.write(json.dumps(value) + "\n")
```
**EN:** `main` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and loads, filters, or serializes benchmark datasets. Notable calls include `select_sglang_backend`, `sgl.set_default_backend`, `time.perf_counter`.
**CN:** `main` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集。其中较关键的调用包括 `select_sglang_backend`, `sgl.set_default_backend`, `time.perf_counter`。

### Lines 87-96: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--question-file", type=str, default="questions.jsonl")
    parser.add_argument("--answer-file", type=str, default="answers.jsonl")
    parser.add_argument("--image-folder", type=str, default="./images")
    parser.add_argument("--temperature", type=float, default=0.0)
    parser.add_argument("--num-questions", type=int, default=None)
    parser.add_argument("--max-tokens", type=int, default=768)
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
- **Standard library / 标准库**: `argparse`, `json`, `os`, `time`
- **Third-party / 第三方依赖**: `tqdm`
- **Internal / 项目内部依赖**: `sglang`, `sglang.test.test_utils`, `sglang.utils`
