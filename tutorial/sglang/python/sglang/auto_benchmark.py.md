# auto_benchmark.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/auto_benchmark.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file provides top-level SGLang runtime utilities, launch helpers, benchmarking entrypoints, or environment configuration used by the broader package. / 该文件提供 SGLang 顶层运行时工具、启动辅助逻辑、基准入口或环境配置，供更大的包范围使用。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module-level supporting statements
```python
import argparse
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 2-2: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 3-8: Module-level supporting statements
```python
from sglang.auto_benchmark_lib import (
    SUPPORTED_DATASETS,
    convert_dataset,
    run_auto_benchmark,
    validate_dataset,
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 9-10: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 11-43: add dataset args function
```python
def add_dataset_args(parser: argparse.ArgumentParser) -> None:
    parser.add_argument(
        "--kind",
        required=True,
        choices=sorted(SUPPORTED_DATASETS),
        help="Dataset kind: sharegpt, custom, random, or generated-shared-prefix.",
    )
    parser.add_argument(
        "--path",
        default="",
        help="Dataset file path. Leave empty for sharegpt auto-download.",
    )
    parser.add_argument("--tokenizer", required=True)
    parser.add_argument("--model", default=None)
    parser.add_argument("--num-prompts", type=int, default=1000)
    parser.add_argument("--output-len", type=int, default=None)
    parser.add_argument("--context-len", type=int, default=None)
    parser.add_argument("--prompt-suffix", type=str, default="")
    parser.add_argument("--apply-chat-template", action="store_true")
    parser.add_argument("--random-input-len", type=int, default=1024)
    parser.add_argument("--random-output-len", type=int, default=256)
    parser.add_argument("--random-range-ratio", type=float, default=0.0)
    parser.add_argument("--gsp-num-groups", type=int, default=64)
    parser.add_argument("--gsp-prompts-per-group", type=int, default=16)
    parser.add_argument("--gsp-system-prompt-len", type=int, default=2048)
    parser.add_argument("--gsp-question-len", type=int, default=128)
    parser.add_argument("--gsp-output-len", type=int, default=256)
    parser.add_argument("--gsp-range-ratio", type=float, default=1.0)
    parser.add_argument("--gsp-fast-prepare", action="store_true")
    parser.add_argument("--gsp-send-routing-key", action="store_true")
    parser.add_argument("--gsp-num-turns", type=int, default=1)
    parser.add_argument("--gsp-ordered", action="store_true")
    parser.add_argument("--seed", type=int, default=1)
```
**EN:** This block uses `add_dataset_args` to store configuration or metadata. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `add_dataset_args` 来存储配置或元数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 44-45: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 46-68: build parser function
```python
def build_parser() -> argparse.ArgumentParser:
    parser = argparse.ArgumentParser(description="SGLang auto benchmark utilities.")
    subparsers = parser.add_subparsers(dest="command", required=True)

    run_parser = subparsers.add_parser(
        "run", help="Run auto benchmark from YAML config."
    )
    run_parser.add_argument("--config", required=True)

    convert_parser = subparsers.add_parser(
        "convert",
        help="Prepare sharegpt/custom/random/generated-shared-prefix data into canonical autobench JSONL.",
    )
    add_dataset_args(convert_parser)
    convert_parser.add_argument("--output", required=True)

    validate_parser = subparsers.add_parser(
        "validate", help="Validate a canonical autobench JSONL dataset."
    )
    validate_parser.add_argument("--dataset-path", required=True)
    validate_parser.add_argument("--tokenizer", required=True)

    return parser
```
**EN:** This block uses `build_parser` to build helper structures or requests. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `build_parser` 来构建辅助结构或请求。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 69-70: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 71-78: main function
```python
def main() -> None:
    args = build_parser().parse_args()
    if args.command == "run":
        run_auto_benchmark(args.config)
    elif args.command == "convert":
        convert_dataset(args)
    elif args.command == "validate":
        validate_dataset(args)
```
**EN:** This block uses `main` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `main` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 79-80: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 81-82: Module-level supporting statements
```python
if __name__ == "__main__":
    main()
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

## Key Concepts / 关键概念
- Benchmarking workflows / 基准测试流程

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.auto_benchmark_lib`
### External / 外部
- `argparse` (stdlib)
