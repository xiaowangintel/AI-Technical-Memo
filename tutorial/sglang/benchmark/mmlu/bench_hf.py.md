# bench_hf.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/mmlu/bench_hf.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on mmlu hf. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 mmlu hf 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and setup / 导入与初始化
```python
"""
Usage:
python3 bench_hf.py --model-path meta-llama/Llama-2-7b-hf --data-dir data --ntrain 5
"""

import argparse
import json
import os
import time

import numpy as np
import pandas as pd
import torch
from tqdm import tqdm
from transformers import AutoModelForCausalLM, AutoTokenizer

choices = ["A", "B", "C", "D"]
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, prepares tensors and invokes GPU kernels, and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、准备张量并调用 GPU 内核、加载、筛选或序列化基准测试数据集。

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

### Lines 51-140: Function `main` / 函数 `main`
```python
def main(args):
    print(f"Loading model: {args.model_path}")
    tokenizer = AutoTokenizer.from_pretrained(args.model_path, trust_remote_code=True)
    model = AutoModelForCausalLM.from_pretrained(
        args.model_path,
        torch_dtype=torch.bfloat16,
        trust_remote_code=True,
        device_map="auto",
    ).eval()

    subjects = sorted(
        [
            f.split("_test.csv")[0]
            for f in os.listdir(os.path.join(args.data_dir, "test"))
            if "_test.csv" in f
        ]
    )

    all_cors = []
    num_requests = 0
    total_latency = 0

    for subject in tqdm(subjects[: args.nsub]):
        dev_df = pd.read_csv(
            os.path.join(args.data_dir, "dev", subject + "_dev.csv"), header=None
        )[: args.ntrain]
        test_df = pd.read_csv(
            os.path.join(args.data_dir, "test", subject + "_test.csv"), header=None
        )

        k = args.ntrain
        few_shot_examples = gen_prompt(dev_df, subject, k)
        while len(tokenizer.encode(few_shot_examples)) > 1536:
            k -= 1
            if k < 0:
                break
            few_shot_examples = gen_prompt(dev_df, subject, k)

        preds = []
        labels = []
        tic = time.perf_counter()

        for i in range(test_df.shape[0]):
            prompt_end = format_example(test_df, i, include_answer=False)
            prompt = few_shot_examples + prompt_end

            input_ids = tokenizer.encode(prompt, return_tensors="pt").to(model.device)
            output_ids = model.generate(
                input_ids,
                max_new_tokens=1,
                do_sample=False,
                pad_token_id=tokenizer.eos_token_id,
            )

            output_str = tokenizer.decode(
                output_ids[0][input_ids.shape[-1] :], skip_special_tokens=True
            )
            preds.append(output_str.strip()[0] if len(output_str.strip()) > 0 else "")
            labels.append(test_df.iloc[i, test_df.shape[1] - 1])

        latency = time.perf_counter() - tic
        total_latency += latency

        cors = [pred == label for pred, label in zip(preds, labels)]
        all_cors.append(cors)
        num_requests += len(test_df)

        print(
            f"Subject: {subject}, Accuracy: {np.mean(cors):.3f}, Latency: {latency:.3f}s"
        )

    weighted_acc = np.mean(np.concatenate(all_cors))
    print(f"Total Latency: {total_latency:.3f}s")
    print(f"Average Accuracy: {weighted_acc:.3f}")

    if args.output:
        with open(args.output, "a") as fout:
            value = {
                "task": "mmlu",
                "backend": "hf",
                "model": args.model_path,
                "latency": round(total_latency, 3),
                "accuracy": round(weighted_acc, 3),
                "num_requests": num_requests,
                "other": {
                    "nsub": args.nsub,
                    "ntrain": args.ntrain,
                },
            }
            fout.write(json.dumps(value) + "\n")
```
**EN:** `main` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. Notable calls include `torch.no_grad`, `print`, `AutoTokenizer.from_pretrained`.
**CN:** `main` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。其中较关键的调用包括 `torch.no_grad`, `print`, `AutoTokenizer.from_pretrained`。

### Lines 143-151: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--model-path", type=str, required=True)
    parser.add_argument("--ntrain", type=int, default=5)
    parser.add_argument("--data-dir", type=str, default="data")
    parser.add_argument("--nsub", type=int, default=60)
    parser.add_argument("--output", type=str, help="Output file path")
    args = parser.parse_args()
    main(args)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `json`, `os`, `time`
- **Third-party / 第三方依赖**: `numpy`, `pandas`, `torch`, `tqdm`, `transformers`
