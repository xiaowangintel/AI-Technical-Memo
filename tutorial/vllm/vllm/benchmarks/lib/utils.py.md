# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/benchmarks/lib/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements utils support for the `lib` portion of vLLM. / 为 vLLM 的 `lib` 子目录实现与 工具 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-9)
```python
import argparse

import json

import math

import os

from contextlib import contextmanager

from typing import Any
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Function `extract_field` (lines 12-24)
```python
def extract_field(
    args: argparse.Namespace, extra_info: dict[str, Any], field_name: str
) -> str:
    if field_name in extra_info:
        return extra_info[field_name]

    v = args
    # For example, args.compilation_config.mode
    for nested_field in field_name.split("."):
        if not hasattr(v, nested_field):
            return ""
        v = getattr(v, nested_field)
    return v
```
**EN:** Function `extract_field` provides a reusable helper around the module's main workflow. Key calls such as `field_name.split`, `hasattr`, `getattr` show the concrete execution path.
**CN:** Function `extract_field` 为模块主流程提供可复用的辅助逻辑。 像 `field_name.split`, `hasattr`, `getattr` 这样的关键调用展示了该代码块的具体执行路径。

### Function `use_compile` (lines 27-35)
```python
def use_compile(args: argparse.Namespace, extra_info: dict[str, Any]) -> bool:
    """
    Check if the benchmark is run with torch.compile
    """
    return not (
        extract_field(args, extra_info, "compilation_config.mode") == "0"
        or "eager" in getattr(args, "output_json", "")
        or "eager" in getattr(args, "result_filename", "")
    )
```
**EN:** Function `use_compile` provides a reusable helper around the module's main workflow. The docstring highlights: Check if the benchmark is run with torch.compile Key calls such as `extract_field`, `getattr` show the concrete execution path.
**CN:** Function `use_compile` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Check if the benchmark is run with torch.compile 像 `extract_field`, `getattr` 这样的关键调用展示了该代码块的具体执行路径。

### Function `convert_to_pytorch_benchmark_format` (lines 38-91)
```python
def convert_to_pytorch_benchmark_format(
    args: argparse.Namespace, metrics: dict[str, list], extra_info: dict[str, Any]
) -> list:
    """
    Save the benchmark results in the format used by PyTorch OSS benchmark with
    on metric per record
    https://github.com/pytorch/pytorch/wiki/How-to-integrate-with-PyTorch-OSS-benchmark-database
    """
    records = []
    if not os.environ.get("SAVE_TO_PYTORCH_BENCHMARK_FORMAT", False):
        return records

    for name, benchmark_values in metrics.items():
        if not isinstance(benchmark_values, list):
            raise TypeError(
                f"benchmark_values for metric '{name}' must be a list, "
                f"but got {type(benchmark_values).__name__}"
            )

        record = {
            "benchmark": {
                "name": "vLLM benchmark",
                "extra_info": {
                    "args": vars(args),
                    "compilation_config.mode": extract_field(
    # ... omitted for brevity ...

    return records
```
**EN:** Function `convert_to_pytorch_benchmark_format` coordinates benchmarking or serving-oriented control flow. The docstring highlights: Save the benchmark results in the format used by PyTorch OSS benchmark with on metric per record https://github.com/pytorch/pytorch/wiki/How-to-integrate-with-PyTorch-OSS-benchm... Key calls such as `os.environ.get`, `metrics.items`, `isinstance`, `TypeError`, `type` show the concrete execution path.
**CN:** Function `convert_to_pytorch_benchmark_format` 负责协调基准测试或服务侧控制流程。 文档字符串强调：Save the benchmark results in the format used by PyTorch OSS benchmark with on metric per record https://github.com/pytorch/pytorch/wiki/How-to-integrate-with-PyTorch-OSS-benchm... 像 `os.environ.get`, `metrics.items`, `isinstance`, `TypeError`, `type` 这样的关键调用展示了该代码块的具体执行路径。

### Class `InfEncoder` (lines 94-110)
```python
class InfEncoder(json.JSONEncoder):
    def clear_inf(self, o: Any):
        if isinstance(o, dict):
            return {
                str(k)
                if not isinstance(k, (str, int, float, bool, type(None)))
                else k: self.clear_inf(v)
                for k, v in o.items()
            }
        elif isinstance(o, list):
            return [self.clear_inf(v) for v in o]
        elif isinstance(o, float) and math.isinf(o):
            return "inf"
        return o

    def iterencode(self, o: Any, *args, **kwargs) -> Any:
        return super().iterencode(self.clear_inf(o), *args, **kwargs)
```
**EN:** Class `InfEncoder` is a structured building block in this module. It inherits from `json.JSONEncoder`. Key methods include `clear_inf`, `iterencode`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `InfEncoder` 是该模块中的结构化构件，继承自 `json.JSONEncoder`。 关键方法包括 `clear_inf`, `iterencode`，它们共同定义初始化、校验、变换或访问模式。

### Method `InfEncoder.clear_inf` (lines 95-107)
```python
    def clear_inf(self, o: Any):
        if isinstance(o, dict):
            return {
                str(k)
                if not isinstance(k, (str, int, float, bool, type(None)))
                else k: self.clear_inf(v)
                for k, v in o.items()
            }
        elif isinstance(o, list):
            return [self.clear_inf(v) for v in o]
        elif isinstance(o, float) and math.isinf(o):
            return "inf"
        return o
```
**EN:** Method `InfEncoder.clear_inf` provides a reusable helper around the module's main workflow. Key calls such as `isinstance`, `type`, `str`, `self.clear_inf`, `o.items` show the concrete execution path.
**CN:** Method `InfEncoder.clear_inf` 为模块主流程提供可复用的辅助逻辑。 像 `isinstance`, `type`, `str`, `self.clear_inf`, `o.items` 这样的关键调用展示了该代码块的具体执行路径。

### Method `InfEncoder.iterencode` (lines 109-110)
```python
    def iterencode(self, o: Any, *args, **kwargs) -> Any:
        return super().iterencode(self.clear_inf(o), *args, **kwargs)
```
**EN:** Method `InfEncoder.iterencode` implements an encoding/decoding or token-transformation step. Key calls such as `super().iterencode`, `super`, `self.clear_inf` show the concrete execution path.
**CN:** Method `InfEncoder.iterencode` 实现编码/解码或 Token 变换步骤。 像 `super().iterencode`, `super`, `self.clear_inf` 这样的关键调用展示了该代码块的具体执行路径。

### Function `write_to_json` (lines 113-120)
```python
def write_to_json(filename: str, records: list) -> None:
    with open(filename, "w") as f:
        json.dump(
            records,
            f,
            cls=InfEncoder,
            default=lambda o: f"<{type(o).__name__} is not JSON serializable>",
        )
```
**EN:** Function `write_to_json` serializes data and writes it to a target representation. Key calls such as `open`, `json.dump`, `type` show the concrete execution path.
**CN:** Function `write_to_json` 负责序列化数据并写入目标表示。 像 `open`, `json.dump`, `type` 这样的关键调用展示了该代码块的具体执行路径。

### Function `default_vllm_config` (lines 124-131)
```python
def default_vllm_config():
    """Set a default VllmConfig for cases that directly test CustomOps or pathways
    that use get_current_vllm_config() outside of a full engine context.
    """
    from vllm.config import VllmConfig, set_current_vllm_config

    with set_current_vllm_config(VllmConfig()):
        yield
```
**EN:** Function `default_vllm_config` parses configuration, arguments, or structured metadata. The docstring highlights: Set a default VllmConfig for cases that directly test CustomOps or pathways that use get_current_vllm_config() outside of a full engine context. Key calls such as `set_current_vllm_config`, `VllmConfig` show the concrete execution path.
**CN:** Function `default_vllm_config` 负责解析配置、参数或结构化元数据。 文档字符串强调：Set a default VllmConfig for cases that directly test CustomOps or pathways that use get_current_vllm_config() outside of a full engine context. 像 `set_current_vllm_config`, `VllmConfig` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。
- **CLI workflow / 命令行流程**
  - **EN:** The module exposes a command-line entrypoint so it can be run as a standalone tool.
  - **CN:** 该模块暴露了命令行入口，因此可以作为独立工具运行。
- **Benchmarking/data prep / 基准测试/数据准备**
  - **EN:** The file includes utilities for preparing benchmark inputs or measuring runtime behavior.
  - **CN:** 该文件包含用于准备基准测试输入或测量运行时行为的工具。
- **Serialization / 序列化**
  - **EN:** The module converts in-memory structures into stable textual or binary representations.
  - **CN:** 该模块会把内存中的结构转换为稳定的文本或二进制表示。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import argparse`, `import json`, `import math`, `import os`, `from contextlib import contextmanager`, `from typing import Any`
