# bench_other.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/multi_document_qa/bench_other.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on multi document qa other. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 multi document qa other 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Imports and setup / 导入与初始化
```python
import argparse
import json
import time
from concurrent.futures import ThreadPoolExecutor
from functools import partial

from tqdm import tqdm

from sglang.test.test_utils import add_common_other_args_and_parse, get_call_generate
from sglang.utils import dump_state_text, read_jsonl

USER_PREFIX = "[INST] "
USER_SUFFIX = " [/INST]"
ASSISTANT_PREFIX = ""
ASSISTANT_SUFFIX = " </s><s>"
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, coordinates asynchronous or parallel execution, and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、协调异步或并行执行、加载、筛选或序列化基准测试数据集。

### Lines 18-34: Function `multi_document_qa` / 函数 `multi_document_qa`
```python
def multi_document_qa(docs, question, generate):
    s = USER_PREFIX
    s += "Please answer a question according to given documents.\n"
    s += "Question:" + question + "Documents begin.\n"

    s += "".join(docs)

    s += "\nDocuments end."
    s += (
        "\n\nBased on the above documents, please answer this question:\n"
        + question
        + "\nAnswer in three words or fewer."
    )
    s += USER_SUFFIX
    s += ASSISTANT_PREFIX
    answer = generate(s, max_tokens=16, stop=None)
    return answer
```
**EN:** `multi_document_qa` is a function that processes tokenized prompts or decoded outputs and computes evaluation scores and aggregate statistics. It returns `answer` to the caller. Notable calls include `''.join`, `generate`.
**CN:** `multi_document_qa` 是一个函数，用于处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。它会向调用方返回 `answer`。其中较关键的调用包括 `''.join`, `generate`。

### Lines 37-106: Function `main` / 函数 `main`
```python
def main(args):
    lines = read_jsonl(args.data_path)
    l = lines[0]
    arguments = []
    labels = []

    num_docs = 10
    if args.backend == "guidance":
        num_docs = 7  # due to OOM

    for i in range(len(l["questions"][: args.num_questions])):
        arguments.append(
            {
                "docs": l["documents"][:num_docs],
                "question": l["questions"][i],
            }
        )
        labels.append(l["answers"][i])
    states = [None] * len(arguments)

    # Select backend
    call_generate = partial(get_call_generate(args), temperature=0)

    # Run requests
    def get_one_answer(i):
        states[i] = multi_document_qa(generate=call_generate, **arguments[i])

    tic = time.perf_counter()
    if args.parallel == 1:
        for i in tqdm(range(len(labels))):
            get_one_answer(i)
    else:
        with ThreadPoolExecutor(args.parallel) as executor:
            list(
                tqdm(
                    executor.map(get_one_answer, list(range(len(labels)))),
                    total=len(labels),
                )
            )

    latency = time.perf_counter() - tic

    # Compute accuracy
    print(states)
    correct = 0
    for s, label in zip(states, labels):
        answer = s.lower()
        if all(x in answer for x in label.lower().split(" ")):
            correct += 1
    accuracy = correct / len(labels)
    print(f"Accuracy: {accuracy:.3f}")
    print(f"Latency: {latency:.3f}")

    # Write results
    dump_state_text(f"tmp_output_{args.backend}.txt", states)

    with open(args.result_file, "a") as fout:
        value = {
            "task": "multi_document_qa",
            "backend": args.backend,
            "num_gpus": 1,
            "latency": round(latency, 3),
            "num_requests": args.num_questions,
            "accuracy": accuracy,
            "other": {
                "num_questions": args.num_questions,
                "parallel": args.parallel,
            },
        }
        fout.write(json.dumps(value) + "\n")
```
**EN:** `main` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. Notable calls include `read_jsonl`, `range`, `partial`.
**CN:** `main` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。其中较关键的调用包括 `read_jsonl`, `range`, `partial`。

### Lines 109-114: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--data-path", type=str, default="questions.jsonl")
    parser.add_argument("--num-questions", type=int, default=100)
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
- **Standard library / 标准库**: `argparse`, `json`, `time`, `concurrent.futures`, `functools`
- **Third-party / 第三方依赖**: `tqdm`
- **Internal / 项目内部依赖**: `sglang.test.test_utils`, `sglang.utils`
