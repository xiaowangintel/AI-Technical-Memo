# bench_sglang_eagle.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/mtbench/bench_sglang_eagle.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on mtbench sglang eagle. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 mtbench sglang eagle 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Imports and setup / 导入与初始化
```python
"""
Adapted from https://github.com/chromecast56/sglang/blob/6f145d2eadb93a116134f703358ce76f15381045/benchmark/mtbench/bench_sglang.py

Benchmark SGLang EAGLE/EAGLE3 Speculative Decoding

Usage:
python3 benchmark/mtbench/bench_sglang_eagle.py --num-questions 80 --parallel 1
"""

import argparse
import json
import os
import time
import uuid

import sglang as sgl
from sglang.test.test_utils import (
    add_common_sglang_args_and_parse,
    select_sglang_backend,
)
from sglang.utils import download_and_cache_file
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集。

### Lines 24-30: Function `load_questions` / 函数 `load_questions`
```python
def load_questions(filename):
    questions = []
    with open(filename, "r") as fin:
        for line in fin:
            obj = json.loads(line)
            questions.append(obj)
    return questions
```
**EN:** `load_questions` is a function that loads, filters, or serializes benchmark datasets and reads or writes local files and intermediate artifacts. It returns `questions` to the caller. Notable calls include `open`, `json.loads`, `questions.append`.
**CN:** `load_questions` 是一个函数，用于加载、筛选或序列化基准测试数据集、读写本地文件及中间产物。它会向调用方返回 `questions`。其中较关键的调用包括 `open`, `json.loads`, `questions.append`。

### Lines 33-46: Function `write_answers` / 函数 `write_answers`
```python
def write_answers(filename, model_id, questions, answers):
    with open(os.path.expanduser(filename), "w") as fout:
        for i in range(len(answers)):
            ans_json = {
                "question_id": questions[i]["question_id"],
                "answer_id": uuid.uuid4().hex,
                "model_id": model_id,
                "choices": {
                    "index": 0,
                    "turns": [answers[i][0], answers[i][1]],
                },
                "tstamp": time.time(),
            }
            fout.write(json.dumps(ans_json) + "\n")
```
**EN:** `write_answers` is a function that measures runtime latency, throughput, or other benchmark metrics, loads, filters, or serializes benchmark datasets, and computes evaluation scores and aggregate statistics. Notable calls include `open`, `range`, `os.path.expanduser`.
**CN:** `write_answers` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集、计算评测分数与聚合统计结果。其中较关键的调用包括 `open`, `range`, `os.path.expanduser`。

### Lines 50-57: Function `answer_mt_bench` / 函数 `answer_mt_bench`
```python
def answer_mt_bench(s, question_1, question_2):
    s += sgl.system(
        "You are a helpful, respectful and honest assistant. Always answer as helpfully as possible, while being safe.  Your answers should not include any harmful, unethical, racist, sexist, toxic, dangerous, or illegal content. Please ensure that your responses are socially unbiased and positive in nature.\n\nIf a question does not make any sense, or is not factually coherent, explain why instead of answering something not correct. If you don't know the answer to a question, please don't share false information."
    )
    s += sgl.user(question_1)
    s += sgl.assistant(sgl.gen("answer_1"))
    s += sgl.user(question_2)
    s += sgl.assistant(sgl.gen("answer_2"))
```
**EN:** `answer_mt_bench` is a function that computes evaluation scores and aggregate statistics. Notable calls include `sgl.system`, `sgl.user`, `sgl.assistant`.
**CN:** `answer_mt_bench` 是一个函数，用于计算评测分数与聚合统计结果。其中较关键的调用包括 `sgl.system`, `sgl.user`, `sgl.assistant`。

### Lines 60-135: Function `main` / 函数 `main`
```python
def main(args):
    # Download question file if not exist
    question_file = args.question_file
    url = "https://raw.githubusercontent.com/lm-sys/FastChat/main/fastchat/llm_judge/data/mt_bench/question.jsonl"
    if not os.path.isfile(question_file):
        question_file = download_and_cache_file(url)

    # Construct prompts
    questions = load_questions(question_file)[: args.num_questions]
    arguments = [
        {"question_1": q["turns"][0], "question_2": q["turns"][1]} for q in questions
    ]

    # Select backend
    backend = select_sglang_backend(args)
    sgl.set_default_backend(backend)

    # Run requests
    tic = time.perf_counter()
    rets = answer_mt_bench.run_batch(
        arguments,
        temperature=0,
        max_new_tokens=2048,
        num_threads=args.parallel,
        progress_bar=True,
    )
    answers = [[s["answer_1"], s["answer_2"]] for s in rets]

    latency = time.perf_counter() - tic
    num_output_tokens = sum(
        s.get_meta_info("answer_1")["completion_tokens"]
        + s.get_meta_info("answer_2")["completion_tokens"]
        for s in rets
    )

    # NOTE: acceptance length is just completion_tokens / spec_verify_ct
    # {'id': '3bb9c5ead109488d8ed5ee9cbecaec29', 'finish_reason': {'type': 'length', 'length': 256}, 'prompt_tokens': 37, 'spec_verify_ct': 101, 'completion_tokens': 256, 'cached_tokens': 0}

    output_throughput = num_output_tokens / latency

    has_verify = "spec_verify_ct" in rets[0].get_meta_info("answer_1")
    if has_verify:
        num_verify_tokens = sum(
            s.get_meta_info("answer_1")["spec_verify_ct"]
            + s.get_meta_info("answer_2")["spec_verify_ct"]
            for s in rets
        )

        accept_length = num_output_tokens / num_verify_tokens
    else:
        accept_length = 1.0

    print(
        f"#questions: {len(questions)}, Throughput: {output_throughput:.2f} token/s, Acceptance length: {accept_length:.2f}"
    )

    # Write results
    model_id = backend.model_info["model_path"]
    answer_file = args.answer_file or f"tmp_output_{args.backend}.txt"
    write_answers(answer_file, model_id, questions, answers)

    with open(args.result_file, "a") as fout:
        value = {
            "task": "mtbench",
            "backend": args.backend,
            "num_gpus": 1,
            "latency": round(latency, 3),
            "throughput": round(output_throughput, 3),
            "accept_length": round(accept_length, 3),
            "num_requests": args.num_questions,
            "other": {
                "num_questions": args.num_questions,
                "parallel": args.parallel,
            },
        }
        fout.write(json.dumps(value) + "\n")
```
**EN:** `main` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and loads, filters, or serializes benchmark datasets. Notable calls include `select_sglang_backend`, `sgl.set_default_backend`, `time.perf_counter`.
**CN:** `main` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集。其中较关键的调用包括 `select_sglang_backend`, `sgl.set_default_backend`, `time.perf_counter`。

### Lines 138-144: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--question-file", type=str, default="question.jsonl")
    parser.add_argument("--answer-file", type=str, default=None)
    parser.add_argument("--num-questions", type=int, default=80)
    args = add_common_sglang_args_and_parse(parser)
    main(args)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration, loads, filters, or serializes benchmark datasets, and computes evaluation scores and aggregate statistics.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、加载、筛选或序列化基准测试数据集、计算评测分数与聚合统计结果。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。
- **Orchestration / 编排**: Launches tools, captures logs, or stitches multi-step experiments together. / 启动工具、抓取日志或拼接多步实验流程。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `json`, `os`, `time`, `uuid`
- **Internal / 项目内部依赖**: `sglang`, `sglang.test.test_utils`, `sglang.utils`
