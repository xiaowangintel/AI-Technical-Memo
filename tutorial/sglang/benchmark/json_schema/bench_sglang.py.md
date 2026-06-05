# bench_sglang.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/json_schema/bench_sglang.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on json schema sglang. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 json schema sglang 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Imports and setup / 导入与初始化
```python
import argparse
import json
import time
from typing import List, Tuple

import jsonschema
from datasets import load_dataset

import sglang as sgl
from sglang.global_config import global_config
from sglang.srt.utils.hf_transformers_utils import get_tokenizer
from sglang.test.test_utils import (
    add_common_sglang_args_and_parse,
    select_sglang_backend,
)
from sglang.utils import dump_state_text
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, loads, filters, or serializes benchmark datasets, and processes tokenized prompts or decoded outputs.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。

### Lines 20-26: Function `schema_gen` / 函数 `schema_gen`
```python
def schema_gen(s, message: Tuple[str, str], json_schema: str):
    system, user = message
    s += sgl.system(system)
    s += sgl.user(user)
    s += sgl.assistant(
        sgl.gen("json_output", temperature=0, max_tokens=256, json_schema=json_schema)
    )
```
**EN:** `schema_gen` is a function that loads, filters, or serializes benchmark datasets and processes tokenized prompts or decoded outputs. Notable calls include `sgl.system`, `sgl.user`, `sgl.assistant`.
**CN:** `schema_gen` 是一个函数，用于加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `sgl.system`, `sgl.user`, `sgl.assistant`。

### Lines 29-40: Function `contains_formats` / 函数 `contains_formats`
```python
def contains_formats(schema, formats: List[str]):
    if isinstance(schema, dict):
        if schema.get("format", None) in formats:
            return True
        for value in schema.values():
            if contains_formats(value, formats):
                return True
    elif isinstance(schema, list):
        for item in schema:
            if contains_formats(item, formats):
                return True
    return False
```
**EN:** `contains_formats` is a function that implements the core logic for this scope. It returns `False` to the caller. Notable calls include `isinstance`, `schema.values`, `schema.get`.
**CN:** `contains_formats` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `False`。其中较关键的调用包括 `isinstance`, `schema.values`, `schema.get`。

### Lines 43-73: Function `convert_dataset` / 函数 `convert_dataset`
```python
def convert_dataset(path: str):
    raw_dataset = load_dataset(path)
    dataset = []
    for data in raw_dataset["train"]:
        messages = data["prompt"]
        schema = data["schema"]
        obj = json.loads(schema)

        # skip some corrupted examples
        if obj.get("type", None) is None:
            continue

        # skip schema with format "email"
        # which is not supported by outlines for now
        if contains_formats(obj, ["email"]):
            continue

        system = messages[0]
        user = messages[1]
        assert system["role"] == "system", "invalid role"
        assert user["role"] == "user", "invalid role"
        assert len(messages) == 2, "invalid message length"
        message = json.dumps(system["content"]), json.dumps(user["content"])
        dataset.append(
            {
                "message": message,
                "json_schema": schema,
            }
        )

    return dataset
```
**EN:** `convert_dataset` is a function that loads, filters, or serializes benchmark datasets and reads or writes local files and intermediate artifacts. It returns `dataset` to the caller. Notable calls include `load_dataset`, `json.loads`, `contains_formats`.
**CN:** `convert_dataset` 是一个函数，用于加载、筛选或序列化基准测试数据集、读写本地文件及中间产物。它会向调用方返回 `dataset`。其中较关键的调用包括 `load_dataset`, `json.loads`, `contains_formats`。

### Lines 76-108: Function `bench_schema` / 函数 `bench_schema`
```python
def bench_schema(args):
    arguments = convert_dataset(args.data_path)

    if args.num_jsons < 0 or args.num_jsons > len(arguments):
        args.num_jsons = len(arguments)
    arguments = arguments[: args.num_jsons]

    # Select backend
    backend = select_sglang_backend(args)
    sgl.set_default_backend(backend)

    # Run requests
    tic = time.perf_counter()
    states = schema_gen.run_batch(
        arguments,
        temperature=0,
        num_threads=args.parallel,
        progress_bar=True,
    )
    latency = time.perf_counter() - tic

    # Check if the outputs are valid
    indexes = []
    for i, state in enumerate(states):
        try:
            schema = json.loads(arguments[i]["json_schema"])
            obj = json.loads(state["json_output"])
            assert jsonschema.validate(obj, schema) is None
        except Exception as e:
            print(e)
            indexes.append(i)

    return states, latency
```
**EN:** `bench_schema` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and loads, filters, or serializes benchmark datasets. It returns `(states, latency)` to the caller. Notable calls include `convert_dataset`, `select_sglang_backend`, `sgl.set_default_backend`.
**CN:** `bench_schema` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集。它会向调用方返回 `(states, latency)`。其中较关键的调用包括 `convert_dataset`, `select_sglang_backend`, `sgl.set_default_backend`。

### Lines 111-138: Function `main` / 函数 `main`
```python
def main(args):
    states, latency = bench_schema(args)

    # Compute accuracy
    tokenizer = get_tokenizer(
        global_config.default_backend.get_server_info()["tokenizer_path"]
    )
    output_jsons = [state["json_output"] for state in states]
    num_output_tokens = sum(len(tokenizer.encode(x)) for x in output_jsons)
    print(f"Latency: {latency:.3f}")
    print(f"Output throughput: {num_output_tokens / latency:.3f} token/s")
    print(f"#output tokens: {num_output_tokens}")

    # Write results
    dump_state_text(f"tmp_output_{args.backend}.txt", states)
    with open(f"{args.backend}.jsonl", "w") as fout:
        for state in states:
            fout.write(state["json_output"] + "\n")

    with open(args.result_file, "a") as fout:
        value = {
            "task": "json_schema",
            "backend": args.backend,
            "latency": round(latency, 3),
            "num_jsons": args.num_jsons,
            "parallel": args.parallel,
        }
        fout.write(json.dumps(value) + "\n")
```
**EN:** `main` is a function that measures runtime latency, throughput, or other benchmark metrics, loads, filters, or serializes benchmark datasets, and processes tokenized prompts or decoded outputs. Notable calls include `bench_schema`, `get_tokenizer`, `sum`.
**CN:** `main` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `bench_schema`, `get_tokenizer`, `sum`。

### Lines 141-146: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--data-path", type=str, default="NousResearch/json-mode-eval")
    parser.add_argument("--num-jsons", type=int, default=-1)
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
- **Standard library / 标准库**: `argparse`, `json`, `time`, `typing`
- **Third-party / 第三方依赖**: `jsonschema`, `datasets`
- **Internal / 项目内部依赖**: `sglang`, `sglang.global_config`, `sglang.srt.utils.hf_transformers_utils`, `sglang.test.test_utils`, `sglang.utils`
