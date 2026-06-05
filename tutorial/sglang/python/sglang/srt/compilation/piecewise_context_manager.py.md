# piecewise_context_manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/compilation/piecewise_context_manager.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the compilation and graph lowering part of the SRT runtime and implements logic centered on `piecewise_context_manager`. It exposes primary entry points such as `is_in_piecewise_cuda_graph`, `is_in_pcg_torch_compile`, `get_pcg_capture_stream`. / 该模块属于 SRT 运行时的编译与图优化部分，主要实现围绕 `piecewise_context_manager` 的逻辑。 它对外提供的主要入口包括 `is_in_piecewise_cuda_graph`, `is_in_pcg_torch_compile`, `get_pcg_capture_stream`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Module imports, constants, and setup
```python
from __future__ import annotations

import logging
from contextlib import contextmanager
from dataclasses import dataclass
from typing import TYPE_CHECKING, Any, List, Optional

import torch

logger = logging.getLogger(__name__)


if TYPE_CHECKING:
    from sglang.srt.model_executor.forward_batch_info import ForwardBatch

_in_piecewise_cuda_graph = False
_in_pcg_torch_compile = False
_pcg_capture_stream = None


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; prepares compilation-related behavior.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；处理与编译相关的行为。

### Lines 21-22: Function is_in_piecewise_cuda_graph
```python
def is_in_piecewise_cuda_graph():
    return _in_piecewise_cuda_graph
```
**EN:** This callable implements `is_in_piecewise_cuda_graph` and mainly implements is in piecewise cuda graph. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `is_in_piecewise_cuda_graph`，主要用于实现 is in piecewise cuda graph 相关逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 25-26: Function is_in_pcg_torch_compile
```python
def is_in_pcg_torch_compile():
    return _in_pcg_torch_compile
```
**EN:** This callable implements `is_in_pcg_torch_compile` and mainly converts data into another representation. In this range it prepares compilation-related behavior.
**CN:** 这一可调用对象实现了 `is_in_pcg_torch_compile`，主要用于将数据转换为另一种表示。 在这一范围内，它会处理与编译相关的行为。

### Lines 29-30: Function get_pcg_capture_stream
```python
def get_pcg_capture_stream():
    return _pcg_capture_stream
```
**EN:** This callable implements `get_pcg_capture_stream` and mainly retrieves a value or derived view. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `get_pcg_capture_stream`，主要用于获取某个值或派生视图。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 33-38: Function enable_piecewise_cuda_graph_compile
```python
@contextmanager
def enable_piecewise_cuda_graph_compile():
    global _in_pcg_torch_compile
    _in_pcg_torch_compile = True
    yield
    _in_pcg_torch_compile = False
```
**EN:** This callable implements `enable_piecewise_cuda_graph_compile` and mainly implements enable piecewise cuda graph compile. In this range it prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `enable_piecewise_cuda_graph_compile`，主要用于实现 enable piecewise cuda graph compile 相关逻辑。 在这一范围内，它会处理与编译相关的行为；管理图捕获或回放逻辑。

### Lines 41-55: Function enable_piecewise_cuda_graph
```python
@contextmanager
def enable_piecewise_cuda_graph():
    global _in_piecewise_cuda_graph
    _in_piecewise_cuda_graph = True
    try:
        yield
    except Exception as e:
        logger.error(
            "Piecewise CUDA Graph failed with error: %s\n%s",
            e,
            PIECEWISE_CUDA_GRAPH_CAPTURE_FAILED_MSG,
        )
        raise
    finally:
        _in_piecewise_cuda_graph = False
```
**EN:** This callable implements `enable_piecewise_cuda_graph` and mainly implements enable piecewise cuda graph. In this range it emits logs for diagnostics; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `enable_piecewise_cuda_graph`，主要用于实现 enable piecewise cuda graph 相关逻辑。 在这一范围内，它会输出日志以便诊断；管理图捕获或回放逻辑。

### Lines 58-63: Function set_pcg_capture_stream
```python
@contextmanager
def set_pcg_capture_stream(stream: torch.cuda.Stream):
    global _pcg_capture_stream
    _pcg_capture_stream = stream
    yield
    _pcg_capture_stream = None
```
**EN:** This callable implements `set_pcg_capture_stream`. It takes `stream` and mainly applies configuration to mutable state. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `set_pcg_capture_stream`。它接收 `stream`，主要用于将配置写入可变状态。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 66-67: Class ForwardContext
```python
@dataclass
class ForwardContext:
```
**EN:** This range introduces `ForwardContext` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `ForwardContext`，并定义其后续方法依赖的结构或元数据。

### Lines 68-73: Method ForwardContext.__init__
```python
    def __init__(self):
        self.forward_batch = None
        self.attention_layers = None
        self.quant_config = None
        self.moe_layers = None
        self.moe_fusions = None
```
**EN:** This callable implements `ForwardContext.__init__` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `ForwardContext.__init__`，主要用于初始化实例状态与默认值。

### Lines 75-76: Method ForwardContext.set_forward_batch
```python
    def set_forward_batch(self, forward_batch: ForwardBatch):
        self.forward_batch = forward_batch
```
**EN:** This callable implements `ForwardContext.set_forward_batch`. It takes `forward_batch` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `ForwardContext.set_forward_batch`。它接收 `forward_batch`，主要用于将配置写入可变状态。

### Lines 78-79: Method ForwardContext.set_attention_layers
```python
    def set_attention_layers(self, layers: List[Any]):
        self.attention_layers = layers
```
**EN:** This callable implements `ForwardContext.set_attention_layers`. It takes `layers` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `ForwardContext.set_attention_layers`。它接收 `layers`，主要用于将配置写入可变状态。

### Lines 81-82: Method ForwardContext.set_quant_config
```python
    def set_quant_config(self, quant_config: Any):
        self.quant_config = quant_config
```
**EN:** This callable implements `ForwardContext.set_quant_config`. It takes `quant_config` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `ForwardContext.set_quant_config`。它接收 `quant_config`，主要用于将配置写入可变状态。

### Lines 84-85: Method ForwardContext.set_moe_layers
```python
    def set_moe_layers(self, layers: List[Any]):
        self.moe_layers = layers
```
**EN:** This callable implements `ForwardContext.set_moe_layers`. It takes `layers` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `ForwardContext.set_moe_layers`。它接收 `layers`，主要用于将配置写入可变状态。

### Lines 87-88: Method ForwardContext.set_moe_fusions
```python
    def set_moe_fusions(self, fusions: List[Any]):
        self.moe_fusions = fusions
```
**EN:** This callable implements `ForwardContext.set_moe_fusions`. It takes `fusions` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `ForwardContext.set_moe_fusions`。它接收 `fusions`，主要用于将配置写入可变状态。

### Lines 89-93: Module-level constants and helpers
```python


_forward_context: Optional[ForwardContext] = None


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 94-97: Function get_forward_context
```python
def get_forward_context() -> Optional[ForwardContext]:
    if _forward_context is None:
        return None
    return _forward_context
```
**EN:** This callable implements `get_forward_context` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `get_forward_context`，主要用于获取某个值或派生视图。

### Lines 100-118: Function set_forward_context
```python
@contextmanager
def set_forward_context(
    forward_batch: ForwardBatch,
    attention_layers: List[Any],
    quant_config: Any,
    moe_layers: List[Any],
    moe_fusions: List[Any],
):
    global _forward_context
    _forward_context = ForwardContext()
    _forward_context.set_forward_batch(forward_batch)
    _forward_context.set_attention_layers(attention_layers)
    _forward_context.set_quant_config(quant_config)
    _forward_context.set_moe_layers(moe_layers)
    _forward_context.set_moe_fusions(moe_fusions)
    try:
        yield
    finally:
        _forward_context = None
```
**EN:** This callable implements `set_forward_context`. It takes `forward_batch`, `attention_layers`, `quant_config`, `moe_layers` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `set_forward_context`。它接收 `forward_batch`, `attention_layers`, `quant_config`, `moe_layers`，主要用于将配置写入可变状态。

### Lines 119-125: Module-level constants and helpers
```python


PIECEWISE_CUDA_GRAPH_CAPTURE_FAILED_MSG = (
    "Piecewise CUDA Graph is enabled by default as an experimental feature.\n"
    "To work around this error, add --disable-piecewise-cuda-graph to your launch command.\n"
    "Please report this issue at https://github.com/sgl-project/sglang/issues/new/choose"
)
```
**EN:** This range organizes module-level state and shared setup. In this range it manages graph capture or replay logic.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

## Key Concepts / 关键概念
- `is_in_piecewise_cuda_graph`: implements is in piecewise cuda graph / 实现 is in piecewise cuda graph 相关逻辑
- `is_in_pcg_torch_compile`: converts data into another representation / 将数据转换为另一种表示
- `get_pcg_capture_stream`: retrieves a value or derived view / 获取某个值或派生视图
- `enable_piecewise_cuda_graph_compile`: implements enable piecewise cuda graph compile / 实现 enable piecewise cuda graph compile 相关逻辑
- `enable_piecewise_cuda_graph`: implements enable piecewise cuda graph / 实现 enable piecewise cuda graph 相关逻辑
- `set_pcg_capture_stream`: applies configuration to mutable state / 将配置写入可变状态
- `ForwardContext`: core class or state container / 核心类或状态容器
- `get_forward_context`: retrieves a value or derived view / 获取某个值或派生视图
- `set_forward_context`: applies configuration to mutable state / 将配置写入可变状态
- `PIECEWISE_CUDA_GRAPH_CAPTURE_FAILED_MSG`: module constant or capability flag / 模块常量或能力标记

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `contextlib`, `dataclasses`, `typing`
- **Third-party / 第三方**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.model_executor.forward_batch_info`
