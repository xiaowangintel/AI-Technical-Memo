# communication_op.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/communication_op.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides thin wrappers around tensor-parallel collective operations. / 提供对张量并行 collective 操作的轻量封装。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化
```python
from typing import Any

import torch
import torch.distributed

from .parallel_state import get_tp_group
```
**EN:** This block imports `typing`, `torch`, `torch.distributed`, `.parallel_state` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `typing`, `torch`, `torch.distributed`, `.parallel_state`，为后续实现准备运行时、类型与辅助 API。

### Function `tensor_model_parallel_all_reduce` / 函数 `tensor_model_parallel_all_reduce`
```python
def tensor_model_parallel_all_reduce(input_: torch.Tensor) -> torch.Tensor:
    """All-reduce the input tensor across model parallel group."""
    return get_tp_group().all_reduce(input_)
```
**EN:** `tensor_model_parallel_all_reduce` is a thin wrapper around `get_tp_group().all_reduce`, exposing that operation through a module-level helper. The docstring frames it as: All-reduce the input tensor across model parallel group. It primarily works with arguments like `input_`. Key calls include `get_tp_group().all_reduce`, `get_tp_group`.
**CN:** `tensor_model_parallel_all_reduce` 是对 `get_tp_group().all_reduce` 的轻量封装，通过模块级辅助函数暴露该操作。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `input_` 这样的参数。 关键调用包括 `get_tp_group().all_reduce`, `get_tp_group`。

### Function `tensor_model_parallel_all_gather` / 函数 `tensor_model_parallel_all_gather`
```python
def tensor_model_parallel_all_gather(
    input_: torch.Tensor, dim: int = -1
) -> torch.Tensor:
    """All-gather the input tensor across model parallel group."""
    return get_tp_group().all_gather(input_, dim)
```
**EN:** `tensor_model_parallel_all_gather` is a thin wrapper around `get_tp_group().all_gather`, exposing that operation through a module-level helper. The docstring frames it as: All-gather the input tensor across model parallel group. It primarily works with arguments like `input_`, `dim`. Key calls include `get_tp_group().all_gather`, `get_tp_group`.
**CN:** `tensor_model_parallel_all_gather` 是对 `get_tp_group().all_gather` 的轻量封装，通过模块级辅助函数暴露该操作。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `input_`, `dim` 这样的参数。 关键调用包括 `get_tp_group().all_gather`, `get_tp_group`。

### Function `tensor_model_parallel_reduce_scatter` / 函数 `tensor_model_parallel_reduce_scatter`
```python
def tensor_model_parallel_reduce_scatter(
    input_: torch.Tensor, dim: int = -1
) -> torch.Tensor:
    """Reduce-Scatter the input tensor across model parallel group."""
    return get_tp_group().reduce_scatter(input_, dim)
```
**EN:** `tensor_model_parallel_reduce_scatter` is a thin wrapper around `get_tp_group().reduce_scatter`, exposing that operation through a module-level helper. The docstring frames it as: Reduce-Scatter the input tensor across model parallel group. It primarily works with arguments like `input_`, `dim`. Key calls include `get_tp_group().reduce_scatter`, `get_tp_group`.
**CN:** `tensor_model_parallel_reduce_scatter` 是对 `get_tp_group().reduce_scatter` 的轻量封装，通过模块级辅助函数暴露该操作。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `input_`, `dim` 这样的参数。 关键调用包括 `get_tp_group().reduce_scatter`, `get_tp_group`。

### Function `tensor_model_parallel_gather` / 函数 `tensor_model_parallel_gather`
```python
def tensor_model_parallel_gather(
    input_: torch.Tensor, dst: int = 0, dim: int = -1
) -> torch.Tensor | None:
    """Gather the input tensor across model parallel group."""
    return get_tp_group().gather(input_, dst, dim)
```
**EN:** `tensor_model_parallel_gather` is a thin wrapper around `get_tp_group().gather`, exposing that operation through a module-level helper. The docstring frames it as: Gather the input tensor across model parallel group. It primarily works with arguments like `input_`, `dst`, `dim`. Key calls include `get_tp_group().gather`, `get_tp_group`.
**CN:** `tensor_model_parallel_gather` 是对 `get_tp_group().gather` 的轻量封装，通过模块级辅助函数暴露该操作。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `input_`, `dst`, `dim` 这样的参数。 关键调用包括 `get_tp_group().gather`, `get_tp_group`。

### Function `broadcast_tensor_dict` / 函数 `broadcast_tensor_dict`
```python
def broadcast_tensor_dict(
    tensor_dict: dict[Any, torch.Tensor | Any] | None = None, src: int = 0
):
    if not torch.distributed.is_initialized():
        return tensor_dict
    return get_tp_group().broadcast_tensor_dict(tensor_dict, src)
```
**EN:** `broadcast_tensor_dict` broadcasts data across ranks for this module. It primarily works with arguments like `tensor_dict`, `src`. Key calls include `get_tp_group().broadcast_tensor_dict`, `torch.distributed.is_initialized`, `get_tp_group`.
**CN:** `broadcast_tensor_dict` 负责在不同 rank 之间广播数据。 它主要处理诸如 `tensor_dict`, `src` 这样的参数。 关键调用包括 `get_tp_group().broadcast_tensor_dict`, `torch.distributed.is_initialized`, `get_tp_group`。

## Key Concepts / 关键概念
- Tensor-parallel collective wrappers / 张量并行 collective 封装
- `tensor_model_parallel_all_reduce`: module-level helper or API entry / `tensor_model_parallel_all_reduce`：模块级辅助函数或 API 入口
- `tensor_model_parallel_all_gather`: module-level helper or API entry / `tensor_model_parallel_all_gather`：模块级辅助函数或 API 入口
- `tensor_model_parallel_reduce_scatter`: module-level helper or API entry / `tensor_model_parallel_reduce_scatter`：模块级辅助函数或 API 入口
- `tensor_model_parallel_gather`: module-level helper or API entry / `tensor_model_parallel_gather`：模块级辅助函数或 API 入口
- `broadcast_tensor_dict`: module-level helper or API entry / `broadcast_tensor_dict`：模块级辅助函数或 API 入口

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Third-party / 第三方**: `torch`, `torch.distributed`
- **Internal modules / 内部模块**: `.parallel_state`
