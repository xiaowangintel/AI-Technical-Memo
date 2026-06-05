# bench_sglang.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/json_decode_regex/bench_sglang.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on json decode regex sglang. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 json decode regex sglang 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Imports and setup / 导入与初始化
```python
import argparse
import json
import time

import sglang as sgl
from sglang.lang.ir import REGEX_FLOAT, REGEX_INT, REGEX_STR
from sglang.test.test_utils import (
    add_common_sglang_args_and_parse,
    select_sglang_backend,
)
from sglang.utils import dump_state_text, read_jsonl

REGEX_LIST = r"\[(" + REGEX_STR + ", )*" + REGEX_STR + r"\]"
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、加载、筛选或序列化基准测试数据集。

### Lines 17-27: Function `json_warm_up` / 函数 `json_warm_up`
```python
def json_warm_up(s):
    s += "The information about Hogwarts is in the following JSON format.\n"
    with s.var_scope("json_output"):
        s += "{\n"
        s += '  "name": ' + sgl.gen("name", max_tokens=8, regex=REGEX_STR + ",") + "\n"
        s += '  "country": ' + sgl.gen("country", max_tokens=8, regex=REGEX_STR + ",") + "\n"
        s += '  "latitude": ' + sgl.gen("latitude", max_tokens=8, regex=REGEX_FLOAT + ",") + "\n"
        s += '  "population": ' + sgl.gen("population", max_tokens=8, regex=REGEX_INT + ",") + "\n"
        s += '  "top 3 landmarks": ' + sgl.gen( "landmarks", max_tokens=24, regex=REGEX_LIST) + "\n"
        s += "}\n"
    print(f'The warmp up json result is:\n{s["json_output"]}')
```
**EN:** `json_warm_up` is a function that loads, filters, or serializes benchmark datasets and processes tokenized prompts or decoded outputs. Notable calls include `print`, `s.var_scope`, `sgl.gen`.
**CN:** `json_warm_up` 是一个函数，用于加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `print`, `s.var_scope`, `sgl.gen`。

### Lines 32-43: Function `json_decode` / 函数 `json_decode`
```python
def json_decode(s, document):
    s += "Please extract the information of a city from the following wikipedia page.\n"
    s += "Page begin.\n" + document + "Page end.\n"
    s += "Here is the name, country, and symbol of the city in JSON format.\n"
    with s.var_scope("json_output"):
        s += "{\n"
        s += '  "name": ' + sgl.gen("name", max_tokens=8, regex=REGEX_STR + ",") + "\n"
        s += '  "country": ' + sgl.gen("country", max_tokens=8, regex=REGEX_STR + ",") + "\n"
        s += '  "latitude": ' + sgl.gen("latitude", max_tokens=8, regex=REGEX_FLOAT + ",") + "\n"
        s += '  "population": ' + sgl.gen("population", max_tokens=8, regex=REGEX_INT + ",") + "\n"
        s += '  "top 3 landmarks": ' + sgl.gen( "landmarks", max_tokens=24, regex=REGEX_LIST) + "\n"
        s += "}\n"
```
**EN:** `json_decode` is a function that prepares tensors and invokes GPU kernels, loads, filters, or serializes benchmark datasets, and processes tokenized prompts or decoded outputs. Notable calls include `s.var_scope`, `sgl.gen`.
**CN:** `json_decode` 是一个函数，用于准备张量并调用 GPU 内核、加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `s.var_scope`, `sgl.gen`。

### Lines 47-93: Function `main` / 函数 `main`
```python
def main(args):
    lines = read_jsonl(args.data_path)
    lines = list(lines)
    arguments = []
    for i in range(len(lines[: args.num_questions])):
        arguments.append(
            {
                "document": lines[i]["document"],
            }
        )

    # Select backend
    backend = select_sglang_backend(args)
    sgl.set_default_backend(backend)

    # Warm up
    json_warm_up.run().sync()

    # Run requests
    tic = time.perf_counter()
    states = json_decode.run_batch(
        arguments, temperature=0, num_threads=args.parallel, progress_bar=True
    )
    latency = time.perf_counter() - tic

    # Compute accuracy
    print(f"Latency: {latency:.3f}")

    # Write results
    dump_state_text(f"tmp_output_{args.backend}.txt", states)

    with open(f"tmp_{args.backend}_json_results.txt", "w") as fout:
        for state in states:
            fout.write(state["json_output"] + "\n")

    with open(args.result_file, "a") as fout:
        value = {
            "task": "json_decode_regex",
            "backend": args.backend,
            "num_gpus": 1,
            "latency": round(latency, 3),
            "num_requests": args.num_questions,
            "other": {
                "parallel": args.parallel,
            },
        }
        fout.write(json.dumps(value) + "\n")
```
**EN:** `main` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. Notable calls include `read_jsonl`, `list`, `range`.
**CN:** `main` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。其中较关键的调用包括 `read_jsonl`, `list`, `range`。

### Lines 96-101: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--data-path", type=str, default="questions.jsonl")
    parser.add_argument("--num-questions", type=int, default=20)
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
- **Internal / 项目内部依赖**: `sglang`, `sglang.lang.ir`, `sglang.test.test_utils`, `sglang.utils`
