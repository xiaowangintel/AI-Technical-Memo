# compilation_counter.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/compilation/compilation_counter.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the compilation and graph lowering part of the SRT runtime and implements logic centered on `compilation_counter`. It exposes primary entry points such as `CompilationCounter`. / 该模块属于 SRT 运行时的编译与图优化部分，主要实现围绕 `compilation_counter` 的逻辑。 它对外提供的主要入口包括 `CompilationCounter`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Module imports, constants, and setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/v0.10.0/vllm/compilation/compilation_counter.py

import copy
import dataclasses
from contextlib import contextmanager


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 10-33: Class CompilationCounter
```python
@dataclasses.dataclass
class CompilationCounter:
    num_models_seen: int = 0
    num_graphs_seen: int = 0
    # including the splitting ops
    num_piecewise_graphs_seen: int = 0
    # not including the splitting ops
    num_piecewise_capturable_graphs_seen: int = 0
    num_backend_compilations: int = 0
    # Number of gpu_model_runner attempts to trigger CUDAGraphs capture
    num_gpu_runner_capture_triggers: int = 0
    # Number of CUDAGraphs captured
    num_cudagraph_captured: int = 0
    # InductorAdapter.compile calls
    num_inductor_compiles: int = 0
    # EagerAdapter.compile calls
    num_eager_compiles: int = 0
    # The number of time vLLM's compiler cache entry was updated
    num_cache_entries_updated: int = 0
    # The number of standalone_compile compiled artifacts saved
    num_compiled_artifacts_saved: int = 0
    # Number of times a model was loaded with CompilationLevel.DYNAMO_AS_IS
    dynamo_as_is_count: int = 0

```
**EN:** This range introduces `CompilationCounter` and defines the structure or metadata that its methods rely on. In this range it prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一段引入 `CompilationCounter`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会处理与编译相关的行为；管理图捕获或回放逻辑。

### Lines 34-35: Method CompilationCounter.clone
```python
    def clone(self) -> "CompilationCounter":
        return copy.deepcopy(self)
```
**EN:** This callable implements `CompilationCounter.clone` and mainly implements clone.
**CN:** 这一可调用对象实现了 `CompilationCounter.clone`，主要用于实现 clone 相关逻辑。

### Lines 37-46: Method CompilationCounter.expect
```python
    @contextmanager
    def expect(self, **kwargs):
        old = self.clone()
        yield
        for k, v in kwargs.items():
            assert getattr(self, k) - getattr(old, k) == v, (
                f"{k} not as expected, before it is {getattr(old, k)}"
                f", after it is {getattr(self, k)}, "
                f"expected diff is {v}"
            )
```
**EN:** This callable implements `CompilationCounter.expect`. It takes `**kwargs` and mainly implements expect. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `CompilationCounter.expect`。它接收 `**kwargs`，主要用于实现 expect 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 47-49: Module-level constants and helpers
```python


compilation_counter = CompilationCounter()
```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

## Key Concepts / 关键概念
- `CompilationCounter`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `copy`, `dataclasses`, `contextlib`
