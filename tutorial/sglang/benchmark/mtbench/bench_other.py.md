# bench_other.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/mtbench/bench_other.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on mtbench other. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 mtbench other 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Imports and setup / 导入与初始化
```python
import argparse
import json
import os
import time
import uuid
from concurrent.futures import ThreadPoolExecutor

from fastchat.model import get_conversation_template
from tqdm import tqdm

from sglang.test.test_utils import add_common_other_args_and_parse, get_call_generate
from sglang.utils import download_and_cache_file
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, coordinates asynchronous or parallel execution, and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、协调异步或并行执行、加载、筛选或序列化基准测试数据集。

### Lines 15-21: Function `load_questions` / 函数 `load_questions`
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

### Lines 24-37: Function `write_answers` / 函数 `write_answers`
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

### Lines 40-109: Function `main` / 函数 `main`
```python
def main(args):
    # Download question file if not exist
    question_file = args.question_file
    url = "https://raw.githubusercontent.com/lm-sys/FastChat/main/fastchat/llm_judge/data/mt_bench/question.jsonl"
    if not os.path.isfile(question_file):
        question_file = download_and_cache_file(url)

    questions = load_questions(question_file)
    questions = (questions * 10)[: args.num_questions]
    max_tokens = 256
    model_id = "llama-2-chat"

    conv_main = get_conversation_template(model_id)

    # Select backend
    call_generate = get_call_generate(args)

    answers = [None] * len(questions)

    def get_answer(i):
        conv = conv_main.copy()
        cur_answers = []
        for j in range(2):
            q = questions[i]["turns"][j]
            conv.append_message(conv.roles[0], q)
            conv.append_message(conv.roles[1], None)

            prompt = conv.get_prompt()
            output = call_generate(prompt, temperature=0, max_tokens=max_tokens).strip()

            cur_answers.append(output)
            conv.update_last_message(output)

        answers[i] = cur_answers

    # Run requests
    tic = time.perf_counter()
    if args.parallel == 1:
        for i in tqdm(range(len(questions))):
            get_answer(i)
    else:
        with ThreadPoolExecutor(args.parallel) as executor:
            list(
                tqdm(
                    executor.map(get_answer, list(range(len(questions)))),
                    total=len(questions),
                )
            )

    latency = time.perf_counter() - tic

    print(f"#questions: {len(questions)}, Latency: {latency:.2f}")

    # Write results
    answer_file = args.answer_file or f"tmp_output_{args.backend}.txt"
    write_answers(answer_file, model_id, questions, answers)

    with open(args.result_file, "a") as fout:
        value = {
            "task": "mtbench",
            "backend": args.backend,
            "num_gpus": 1,
            "latency": round(latency, 3),
            "num_requests": args.num_questions,
            "other": {
                "num_questions": args.num_questions,
                "parallel": args.parallel,
            },
        }
        fout.write(json.dumps(value) + "\n")
```
**EN:** `main` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. Notable calls include `load_questions`, `get_conversation_template`, `get_call_generate`.
**CN:** `main` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。其中较关键的调用包括 `load_questions`, `get_conversation_template`, `get_call_generate`。

### Lines 112-118: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--question-file", type=str, default="question.jsonl")
    parser.add_argument("--answer-file", type=str, default=None)
    parser.add_argument("--num-questions", type=int, default=80)
    args = add_common_other_args_and_parse(parser)
    main(args)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration, loads, filters, or serializes benchmark datasets, and computes evaluation scores and aggregate statistics.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、加载、筛选或序列化基准测试数据集、计算评测分数与聚合统计结果。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Concurrency / 并发**: Coordinates multiple requests or workers at the same time. / 同时协调多个请求或工作单元。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `json`, `os`, `time`, `uuid`, `concurrent.futures`
- **Third-party / 第三方依赖**: `fastchat.model`, `tqdm`
- **Internal / 项目内部依赖**: `sglang.test.test_utils`, `sglang.utils`
