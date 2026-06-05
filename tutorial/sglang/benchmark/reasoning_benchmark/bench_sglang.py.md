# bench_sglang.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/reasoning_benchmark/bench_sglang.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on reasoning sglang. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 reasoning sglang 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Imports and setup / 导入与初始化
```python
import argparse
import json
import time

import answer_extraction
import eval_utils
import numpy as np
from datasets import load_dataset

import sglang as sgl
from sglang.test.test_utils import (
    add_common_sglang_args_and_parse,
    select_sglang_backend,
)
from sglang.utils import dump_state_text
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, loads, filters, or serializes benchmark datasets, and computes evaluation scores and aggregate statistics.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、加载、筛选或序列化基准测试数据集、计算评测分数与聚合统计结果。

### Lines 19-28: Function `reasoning_gen` / 函数 `reasoning_gen`
```python
def reasoning_gen(s, question: str):
    s += sgl.user(
        question
        + "\nPlease reason step by step, and put your final answer within \boxed{}."
    )
    s += sgl.assistant(
        sgl.gen(
            "answer",
        )
    )
```
**EN:** `reasoning_gen` is a function that computes evaluation scores and aggregate statistics. Notable calls include `sgl.user`, `sgl.assistant`, `sgl.gen`.
**CN:** `reasoning_gen` 是一个函数，用于计算评测分数与聚合统计结果。其中较关键的调用包括 `sgl.user`, `sgl.assistant`, `sgl.gen`。

### Lines 31-41: Function `convert_dataset` / 函数 `convert_dataset`
```python
def convert_dataset(path: str, question_key: str, answer_key: str, num_tries: int):
    raw_dataset = load_dataset(path)
    questions = []
    answers = []
    for data in raw_dataset["train"]:
        question = data[question_key]
        answer = data[answer_key]
        for _ in range(num_tries):
            questions.append({"question": question})
            answers.append({"answer": answer})
    return questions, answers
```
**EN:** `convert_dataset` is a function that loads, filters, or serializes benchmark datasets and computes evaluation scores and aggregate statistics. It returns `(questions, answers)` to the caller. Notable calls include `load_dataset`, `range`, `questions.append`.
**CN:** `convert_dataset` 是一个函数，用于加载、筛选或序列化基准测试数据集、计算评测分数与聚合统计结果。它会向调用方返回 `(questions, answers)`。其中较关键的调用包括 `load_dataset`, `range`, `questions.append`。

### Lines 44-124: Function `main` / 函数 `main`
```python
def main(args):
    # Select backend
    sgl.set_default_backend(select_sglang_backend(args))

    # Get dataset
    questions, answers = convert_dataset(
        args.data_path, args.question_key, args.answer_key, args.num_tries
    )

    # Run requests
    tic = time.perf_counter()
    states = reasoning_gen.run_batch(
        questions,
        num_threads=args.parallel,
        progress_bar=True,
        temperature=0.6,
        max_new_tokens=32768,
        top_p=0.95,
    )
    latency = time.perf_counter() - tic

    # Extract results and record outcomes in a list.
    outcomes = []
    for i, state in enumerate(states):
        try:
            pred_answer = answer_extraction.extract_math_answer(
                questions[i]["question"], state["answer"], "limo"
            )
            gt_answer = str(answers[i]["answer"])
            pred_answer = (
                pred_answer[-1] if isinstance(pred_answer, list) else pred_answer
            )
            is_correct = 1 if eval_utils.math_equal(pred_answer, gt_answer) else 0
        except Exception as e:
            print(f"Error extracting answer: {e}")
            is_correct = 0

        outcomes.append(is_correct)

    # Calculate overall accuracy using numpy
    overall_accuracy = np.mean(outcomes)
    print(f"Overall Accuracy: {overall_accuracy}")

    # Calculate mean standard error over questions if num_tries >= 2
    if args.num_tries > 1:
        outcomes_np = np.array(outcomes).reshape(-1, args.num_tries)
        # Using sample standard deviation with ddof=1
        std_per_question = np.std(outcomes_np, axis=1, ddof=1)
        # Compute the standard error for each question: std / sqrt(num_tries)
        se_per_question = std_per_question / np.sqrt(args.num_tries)
        mean_se = se_per_question.mean()
        print(f"Mean Standard Error of Accuracy across questions: {mean_se}")
    else:
        mean_se = None
        print("Not enough samples per question to compute standard error.")

    # Calculate output throughput
    num_output_tokens = sum(
        s.get_meta_info("answer")["completion_tokens"] for s in states
    )
    output_throughput = num_output_tokens / latency
    print(f"Output throughput: {output_throughput} token/s")

    # Dump results
    dump_state_text(f"tmp_output_{args.backend}.txt", states)

    # Write results
    with open(args.result_file, "a") as fout:
        value = {
            "task": "limo",
            "backend": args.backend,
            "latency": round(latency, 3),
            "overall_accuracy": round(overall_accuracy, 3),
            "mean_se_accuracy": round(mean_se, 3) if mean_se is not None else None,
            "num_requests": len(questions),
            "other": {
                "num_questions": len(questions),
                "parallel": args.parallel,
            },
        }
        fout.write(json.dumps(value) + "\n")
```
**EN:** `main` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and loads, filters, or serializes benchmark datasets. Notable calls include `sgl.set_default_backend`, `convert_dataset`, `time.perf_counter`.
**CN:** `main` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集。其中较关键的调用包括 `sgl.set_default_backend`, `convert_dataset`, `time.perf_counter`。

### Lines 127-135: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--data-path", type=str, default="GAIR/LIMO")
    parser.add_argument("--question-key", type=str, default="question")
    parser.add_argument("--answer-key", type=str, default="answer")
    parser.add_argument("--num-tries", type=int, default=1)
    add_common_sglang_args_and_parse(parser)
    args = parser.parse_args()
    main(args)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration and computes evaluation scores and aggregate statistics.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、计算评测分数与聚合统计结果。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `json`, `time`
- **Third-party / 第三方依赖**: `answer_extraction`, `eval_utils`, `numpy`, `datasets`
- **Internal / 项目内部依赖**: `sglang`, `sglang.test.test_utils`, `sglang.utils`
