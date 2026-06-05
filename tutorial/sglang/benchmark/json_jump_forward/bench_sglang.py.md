# bench_sglang.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/json_jump_forward/bench_sglang.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on json jump forward sglang. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 json jump forward sglang 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40: Imports and setup / 导入与初始化
```python
import argparse
import json
import time

import sglang as sgl
from sglang.test.test_utils import (
    add_common_sglang_args_and_parse,
    select_sglang_backend,
)
from sglang.utils import dump_state_text, read_jsonl

# there are some FSM bugs with json regex converted from pydantic model
# here use a string regex instead
# regex_string = build_regex_from_object(HarryPoterRole)
character_regex = (
    r"""\{\n"""
    + r"""    "name": "[\w\d\s]{1,16}",\n"""
    + r"""    "house": "(Gryffindor|Slytherin|Ravenclaw|Hufflepuff)",\n"""
    + r"""    "blood status": "(Pure-blood|Half-blood|Muggle-born)",\n"""
    + r"""    "occupation": "(student|teacher|auror|ministry of magic|death eater|order of the phoenix)",\n"""
    + r"""    "wand": \{\n"""
    + r"""        "wood": "[\w\d\s]{1,16}",\n"""
    + r"""        "core": "[\w\d\s]{1,16}",\n"""
    + r"""        "length": [0-9]{1,2}\.[0-9]{0,2}\n"""
    + r"""    \},\n"""
    + r"""    "alive": "(Alive|Deceased)",\n"""
    + r"""    "patronus": "[\w\d\s]{1,16}",\n"""
    + r"""    "bogart": "[\w\d\s]{1,16}"\n"""
    + r"""\}"""
)

city_regex = (
    r"""\{\n"""
    + r"""  "name": "[\w\d\s]{1,16}",\n"""
    + r"""  "country": "[\w\d\s]{1,16}",\n"""
    + r"""  "latitude": [-+]?[0-9]*\.?[0-9]{0,2},\n"""
    + r"""  "population": [-+]?[0-9]{1,9},\n"""
    + r"""  "top 3 landmarks": \["[\w\d\s]{1,16}", "[\w\d\s]{1,16}", "[\w\d\s]{1,16}"\]\n"""
    + r"""\}"""
)
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、加载、筛选或序列化基准测试数据集。

### Lines 44-46: Function `character_gen` / 函数 `character_gen`
```python
def character_gen(s, name):
    s += name + " is a character in Harry Potter. Please fill in the following information about this character.\n"
    s += sgl.gen("json_output", max_tokens=256, regex=character_regex)
```
**EN:** `character_gen` is a function that loads, filters, or serializes benchmark datasets and processes tokenized prompts or decoded outputs. Notable calls include `sgl.gen`.
**CN:** `character_gen` 是一个函数，用于加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `sgl.gen`。

### Lines 51-55: Function `city_gen` / 函数 `city_gen`
```python
def city_gen(s, document):
    s += "Please extract the information of a city from the following wikipedia page.\n"
    s += "Page begin.\n" + document + "Page end.\n"
    s += "Here is the name, country, and symbol of the city in JSON format.\n"
    s += sgl.gen("json_output",max_tokens=256, regex=city_regex)
```
**EN:** `city_gen` is a function that loads, filters, or serializes benchmark datasets and processes tokenized prompts or decoded outputs. Notable calls include `sgl.gen`.
**CN:** `city_gen` 是一个函数，用于加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `sgl.gen`。

### Lines 59-79: Function `bench_city_doc` / 函数 `bench_city_doc`
```python
def bench_city_doc(args):
    arguments = []
    for line in read_jsonl(args.data_path):
        arguments.append({"document": line["document"]})
    arguments = arguments[: args.num_jsons]

    # Select backend
    backend = select_sglang_backend(args)
    sgl.set_default_backend(backend)

    # Run requests
    tic = time.perf_counter()
    states = city_gen.run_batch(
        arguments,
        temperature=0,
        num_threads=args.parallel,
        progress_bar=True,
    )
    latency = time.perf_counter() - tic

    return states, latency
```
**EN:** `bench_city_doc` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and loads, filters, or serializes benchmark datasets. It returns `(states, latency)` to the caller. Notable calls include `read_jsonl`, `select_sglang_backend`, `sgl.set_default_backend`.
**CN:** `bench_city_doc` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集。它会向调用方返回 `(states, latency)`。其中较关键的调用包括 `read_jsonl`, `select_sglang_backend`, `sgl.set_default_backend`。

### Lines 82-103: Function `bench_character` / 函数 `bench_character`
```python
def bench_character(args):
    arguments = []
    with open(args.data_path, "r") as f:
        for line in f:
            arguments.append({"name": line.strip()})
    arguments = arguments[: args.num_jsons]

    # Select backend
    backend = select_sglang_backend(args)
    sgl.set_default_backend(backend)

    # Run requests
    tic = time.perf_counter()
    states = character_gen.run_batch(
        arguments,
        temperature=0,
        num_threads=args.parallel,
        progress_bar=True,
    )
    latency = time.perf_counter() - tic

    return states, latency
```
**EN:** `bench_character` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and loads, filters, or serializes benchmark datasets. It returns `(states, latency)` to the caller. Notable calls include `select_sglang_backend`, `sgl.set_default_backend`, `time.perf_counter`.
**CN:** `bench_character` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集。它会向调用方返回 `(states, latency)`。其中较关键的调用包括 `select_sglang_backend`, `sgl.set_default_backend`, `time.perf_counter`。

### Lines 106-132: Function `main` / 函数 `main`
```python
def main(args):
    if args.mode == "character":
        args.data_path = "dataset.txt"
        states, latency = bench_character(args)
    elif args.mode == "city":
        args.data_path = "questions.jsonl"
        states, latency = bench_city_doc(args)

    # Compute accuracy
    print(f"Latency: {latency:.3f}")

    # Write results
    dump_state_text(f"tmp_output_{args.backend}_{args.mode}.txt", states)
    with open(f"{args.backend}_{args.mode}.json", "w") as fout:
        for state in states:
            fout.write(state["json_output"] + "\n")

    with open(args.result_file, "a") as fout:
        value = {
            "task": "json_jump_forward",
            "backend": args.backend,
            "latency": round(latency, 3),
            "num_jsons": args.num_jsons,
            "mode": args.mode,
            "parallel": args.parallel,
        }
        fout.write(json.dumps(value) + "\n")
```
**EN:** `main` is a function that measures runtime latency, throughput, or other benchmark metrics, loads, filters, or serializes benchmark datasets, and computes evaluation scores and aggregate statistics. Notable calls include `print`, `dump_state_text`, `bench_character`.
**CN:** `main` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集、计算评测分数与聚合统计结果。其中较关键的调用包括 `print`, `dump_state_text`, `bench_character`。

### Lines 135-143: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--data-path", type=str)
    parser.add_argument("--num-jsons", type=int, default=50)
    parser.add_argument(
        "--mode", type=str, default="character", choices=["character", "city"]
    )
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
- **Internal / 项目内部依赖**: `sglang`, `sglang.test.test_utils`, `sglang.utils`
