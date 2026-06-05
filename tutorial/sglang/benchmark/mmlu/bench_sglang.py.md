# bench_sglang.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/mmlu/bench_sglang.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on mmlu sglang. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 mmlu sglang 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Imports and setup / 导入与初始化
```python
import argparse
import json
import os
import subprocess
import tarfile
import time

import numpy as np
import pandas as pd
import tiktoken

from sglang.test.test_utils import (
    add_common_sglang_args_and_parse,
    dump_bench_raw_result,
    select_sglang_backend,
)

SCRIPT_DIR = os.path.dirname(os.path.abspath(__file__))

choices = ["A", "B", "C", "D"]

tokenizer = tiktoken.encoding_for_model("gpt-3.5-turbo")
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, loads, filters, or serializes benchmark datasets, and processes tokenized prompts or decoded outputs.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。

### Lines 25-30: Function `format_subject` / 函数 `format_subject`
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

### Lines 33-41: Function `format_example` / 函数 `format_example`
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

### Lines 44-52: Function `gen_prompt` / 函数 `gen_prompt`
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

### Lines 55-74: Function `download_data` / 函数 `download_data`
```python
def download_data(data_dir):
    """Download and extract MMLU data if it doesn't exist."""
    if os.path.isdir(os.path.join(data_dir, "test")):
        return
    print(f"Data not found at {data_dir}. Downloading...")
    os.makedirs(data_dir, exist_ok=True)
    tar_path = os.path.join(data_dir, "data.tar")
    subprocess.check_call(
        ["wget", "-O", tar_path, "https://people.eecs.berkeley.edu/~hendrycks/data.tar"]
    )
    with tarfile.open(tar_path) as tar:
        tar.extractall(path=data_dir, filter="data")
    # The tarball extracts into a "data/" subdirectory; move contents up if needed
    nested = os.path.join(data_dir, "data")
    if os.path.isdir(nested):
        for item in os.listdir(nested):
            os.rename(os.path.join(nested, item), os.path.join(data_dir, item))
        os.rmdir(nested)
    os.remove(tar_path)
    print("Download complete.")
```
**EN:** `download_data` is a function that orchestrates external commands and log collection and reads or writes local files and intermediate artifacts. The docstring frames it as: Download and extract MMLU data if it doesn't exist. Notable calls include `os.path.isdir`, `print`, `os.makedirs`.
**CN:** `download_data` 是一个函数，用于编排外部命令并收集日志、读写本地文件及中间产物。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `os.path.isdir`, `print`, `os.makedirs`。

### Lines 77-197: Function `main` / 函数 `main`
```python
def main(args):
    subjects = sorted(
        [
            f.split("_test.csv")[0]
            for f in os.listdir(os.path.join(args.data_dir, "test"))
            if "_test.csv" in f
        ]
    )

    # Build prompts
    arguments = []
    labels = []
    num_questions = []

    for subject in subjects[: args.nsub]:
        dev_df = pd.read_csv(
            os.path.join(args.data_dir, "dev", subject + "_dev.csv"), header=None
        )[: args.ntrain]
        test_df = pd.read_csv(
            os.path.join(args.data_dir, "test", subject + "_test.csv"), header=None
        )
        num_questions.append(test_df.shape[0])

        k = args.ntrain
        few_shot_examples = gen_prompt(dev_df, subject, k)
        while len(tokenizer.encode(few_shot_examples)) > 1536:
            k -= 1
            few_shot_examples = gen_prompt(dev_df, subject, k)

        for i in range(test_df.shape[0]):
            prompt_end = format_example(test_df, i, include_answer=False)

            arguments.append(
                {
                    "examples": few_shot_examples,
                    "question": prompt_end,
                }
            )

            label = test_df.iloc[i, test_df.shape[1] - 1]
            labels.append(label)

    #####################################
    ######### SGL Program Begin #########
    #####################################

    import sglang as sgl

    if args.backend.startswith("gpt-"):

        @sgl.function
        def few_shot_mmlu(s, examples, question):
            s += sgl.user(examples + question)
            s += sgl.assistant(sgl.gen("answer"))

    else:

        @sgl.function
        def few_shot_mmlu(s, examples, question):
            s += examples + question + sgl.gen("answer")

    #####################################
    ########## SGL Program End ##########
    #####################################

    # Select backend
    backend = select_sglang_backend(args)

    # Run
    tic = time.perf_counter()
    states = few_shot_mmlu.run_batch(
        arguments,
        temperature=0,
        max_new_tokens=1,
        backend=backend,
        num_threads=args.parallel,
        progress_bar=True,
    )
    preds = [
        s["answer"].strip()[0] if len(s["answer"].strip()) > 0 else "" for s in states
    ]
    latency = time.perf_counter() - tic

    # Compute accuracy
    cors = [pred == label for pred, label in zip(preds, labels)]

    pt = 0
    for subject, num_qs in zip(subjects[: args.nsub], num_questions):
        print(
            f"subject: {subject}, #q:{num_qs}, acc: {np.mean(cors[pt: pt + num_qs]):.3f}"
        )
        pt += num_qs
    assert pt == len(cors)
    weighted_acc = np.mean(cors)

    dump_bench_raw_result(
        path=args.raw_result_file,
        states=states,
        preds=preds,
        labels=labels,
    )

    # Print results
    print("Total latency: {:.3f}".format(latency))
    print("Average accuracy: {:.3f}".format(weighted_acc))

    # Write results
    with open(args.result_file, "a") as fout:
        value = {
            "task": "mmlu",
            "backend": args.backend,
            "num_gpus": 1,
            "latency": round(latency, 3),
            "accuracy": round(weighted_acc, 3),
            "num_requests": len(arguments),
            "other": {
                "nsub": args.nsub,
                "parallel": args.parallel,
            },
        }
        fout.write(json.dumps(value) + "\n")
```
**EN:** `main` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and loads, filters, or serializes benchmark datasets. Notable calls include `sorted`, `args.backend.startswith`, `select_sglang_backend`.
**CN:** `main` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集。其中较关键的调用包括 `sorted`, `args.backend.startswith`, `select_sglang_backend`。

### Lines 200-210: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--ntrain", "-k", type=int, default=5)
    parser.add_argument(
        "--data_dir", "-d", type=str, default=os.path.join(SCRIPT_DIR, "data")
    )
    parser.add_argument("--save_dir", "-s", type=str, default="results")
    parser.add_argument("--nsub", type=int, default=60)
    args = add_common_sglang_args_and_parse(parser)
    download_data(args.data_dir)
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
- **Orchestration / 编排**: Launches tools, captures logs, or stitches multi-step experiments together. / 启动工具、抓取日志或拼接多步实验流程。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `json`, `os`, `subprocess`, `tarfile`, `time`
- **Third-party / 第三方依赖**: `numpy`, `pandas`, `tiktoken`
- **Internal / 项目内部依赖**: `sglang.test.test_utils`, `sglang`
