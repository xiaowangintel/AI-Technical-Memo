# common.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/utils/common.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements runtime layer support for the SGLang SRT runtime. It exposes symbols such as `get_layer_id`, `pad_or_narrow_weight`, `is_strict_contiguous`, and `strict_contiguous` and connects them to backend-specific paths such as `CUDA` and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了运行时层支持。它提供了 `get_layer_id`、`pad_or_narrow_weight`、`is_strict_contiguous` 以及 `strict_contiguous` 等符号，并把这些符号连接到 `CUDA` 和 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-2: File header and module overview
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** This opening block contains comments, licensing text, or other context that frames the rest of the file before executable code begins.
**CN:** 这一开头部分包含注释、许可证文本或其他上下文信息，用来为后续可执行代码建立背景。

### Lines 3-11: Imports, constants, and runtime setup
```python
import logging
import re

import torch
from torch.nn.parameter import Parameter

logger = logging.getLogger(__name__)
```
**EN:** This section prepares the module namespace. It imports `logging`, `re`, `torch`, and `torch.nn.parameter.Parameter`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `logger` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `logging`、`re`、`torch` 以及 `torch.nn.parameter.Parameter`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `logger` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 12-19: `get_layer_id` getter for layer id
```python
def get_layer_id(weight_name):
    # example weight name: model.layers.10.self_attn.qkv_proj.weight
    match = re.search(r"layers\.(\d+)\.", weight_name)
    if match:
        return int(match.group(1))
    return None
```
**EN:** This block defines `get_layer_id` and contains the main logic for this step. It mainly invokes `re.search`, `int`, and `match.group`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `match` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_layer_id`，并承载这一阶段的核心逻辑。 它主要调用 `re.search`、`int` 以及 `match.group`，说明该流程会编排底层辅助函数或计算内核。 像 `match` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 20-42: Function `pad_or_narrow_weight` and its core logic
```python
def pad_or_narrow_weight(
    loaded_weight: torch.Tensor, input_dim: int, start_idx: int, shard_size: int
) -> torch.Tensor:
    # Padding with zeros for special case such as qwen2_5_VL's mlp which is not 8-aligned
    valid_size = max(loaded_weight.shape[input_dim] - start_idx, 0)

    if valid_size > 0:
        loaded_slice = loaded_weight.narrow(input_dim, start_idx, valid_size)
        pad_shape = list(loaded_weight.shape)
        pad_shape[input_dim] = shard_size - valid_size
        pad = torch.zeros(
            pad_shape, dtype=loaded_weight.dtype, device=loaded_weight.device
        )
        return torch.cat([loaded_slice, pad], dim=input_dim)

    # All padding
    pad_shape = list(loaded_weight.shape)
    pad_shape[input_dim] = shard_size
    return torch.zeros(
        pad_shape, dtype=loaded_weight.dtype, device=loaded_weight.device
    )
```
**EN:** This block defines `pad_or_narrow_weight` and contains the main logic for this step. It mainly invokes `max`, `list`, `torch.zeros`, `loaded_weight.narrow`, and `torch.cat`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `valid_size`, `pad_shape`, `loaded_slice`, and `pad` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `pad_or_narrow_weight`，并承载这一阶段的核心逻辑。 它主要调用 `max`、`list`、`torch.zeros`、`loaded_weight.narrow` 以及 `torch.cat`，说明该流程会编排底层辅助函数或计算内核。 像 `valid_size`、`pad_shape`、`loaded_slice` 以及 `pad` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 43-51: `is_strict_contiguous` predicate for is strict contiguous
```python
def is_strict_contiguous(x: torch.Tensor) -> bool:
    expected_stride = 1
    for size, stride in zip(reversed(x.shape), reversed(x.stride())):
        if stride != expected_stride:
            return False
        expected_stride *= size
    return True
```
**EN:** This block defines `is_strict_contiguous` and contains the main logic for this step. It mainly invokes `zip`, `reversed`, and `x.stride`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `expected_stride` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `is_strict_contiguous`，并承载这一阶段的核心逻辑。 它主要调用 `zip`、`reversed` 以及 `x.stride`，说明该流程会编排底层辅助函数或计算内核。 像 `expected_stride` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 52-57: Function `strict_contiguous` and its core logic
```python
def strict_contiguous(x: torch.Tensor) -> torch.Tensor:
    if is_strict_contiguous(x):
        return x
    return x.clone(memory_format=torch.contiguous_format)
```
**EN:** This block defines `strict_contiguous` and contains the main logic for this step. It mainly invokes `is_strict_contiguous` and `x.clone`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `strict_contiguous`，并承载这一阶段的核心逻辑。 它主要调用 `is_strict_contiguous` 和 `x.clone`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 58-73: Function `copy_or_rebind_param` and its core logic
```python
def copy_or_rebind_param(
    module: torch.nn.Module, name: str, new_value: torch.Tensor
) -> None:
    """Keep parameter identities stable for CUDA graph reuse and hot reload."""
    new_value = new_value.detach()
    param = getattr(module, name, None)
    if isinstance(param, Parameter):
        if param.data.shape == new_value.shape and param.data.dtype == new_value.dtype:
            param.data.copy_(new_value)
        else:
            param.data = new_value
        param.requires_grad_(False)
    else:
        setattr(module, name, Parameter(new_value, requires_grad=False))
```
**EN:** This block defines `copy_or_rebind_param` and contains the main logic for this step. It mainly invokes `new_value.detach`, `getattr`, `isinstance`, `param.requires_grad_`, and `setattr`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `new_value` and `param` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `copy_or_rebind_param`，并承载这一阶段的核心逻辑。 它主要调用 `new_value.detach`、`getattr`、`isinstance`、`param.requires_grad_` 以及 `setattr`，说明该流程会编排底层辅助函数或计算内核。 像 `new_value` 和 `param` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 74-108: Function `alias_or_bind_derived_param` and its core logic
```python
def alias_or_bind_derived_param(
    module: torch.nn.Module,
    source_name: str,
    derived_name: str,
    derived_value: torch.Tensor,
) -> None:
    """Bind a post-processed (derived) tensor to a derived attribute name.

    When `derived_value` is broadcastable to the source Parameter's shape (and
    dtype matches), write it broadcast-filled into the source's storage in
    place and register `derived_name` as an alias of the source Parameter. The
    two attribute names then share one underlying buffer, so:
      - apply() can read via `derived_name`
      - update_weights_from_disk can keep refilling `source_name` (the loader
        re-runs process_weights_after_loading which re-derives in place)
      - peak GPU memory is the source size, not source + derived.

    When the shapes are not broadcast-compatible, fall back to allocating a
    separate Parameter under `derived_name` via copy_or_rebind_param.
    """
    derived_value = derived_value.detach()
    source = getattr(module, source_name, None)
    if isinstance(source, Parameter) and source.data.dtype == derived_value.dtype:
        try:
            broadcast = torch.broadcast_to(derived_value, source.data.shape)
        except RuntimeError:
            broadcast = None
        if broadcast is not None:
            source.data.copy_(broadcast)
            source.requires_grad_(False)
            setattr(module, derived_name, source)
            return
    copy_or_rebind_param(module, derived_name, derived_value)
```
**EN:** This block defines `alias_or_bind_derived_param` and contains the main logic for this step. It mainly invokes `derived_value.detach`, `getattr`, `copy_or_rebind_param`, `isinstance`, and `torch.broadcast_to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `derived_value`, `source`, and `broadcast` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `alias_or_bind_derived_param`，并承载这一阶段的核心逻辑。 它主要调用 `derived_value.detach`、`getattr`、`copy_or_rebind_param`、`isinstance` 以及 `torch.broadcast_to`，说明该流程会编排底层辅助函数或计算内核。 像 `derived_value`、`source` 以及 `broadcast` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 109-115: Class `PPMissingLayer` declaration and shared state
```python
class PPMissingLayer(torch.nn.Identity):
    # Adapted from
    # https://github.com/vllm-project/vllm/blob/18ed3132d2bfe1df9a74729457b69243955221e8/vllm/model_executor/models/utils.py#L468C1-L486C1
    """
    A placeholder layer for missing layers in a pipeline parallel model.
    """
```
**EN:** This block introduces class `PPMissingLayer` and the state shared by its methods. It inherits from `torch.nn.Identity`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: A placeholder layer for missing layers in a pipeline parallel model.
**CN:** 该代码块引入类 `PPMissingLayer`，并定义其方法共享的状态。 它继承自 `torch.nn.Identity`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 116-119: `PPMissingLayer` initialization and state setup
```python
    def __init__(self, *args, **kwargs):
        super().__init__()
        self.return_tuple = kwargs.get("return_tuple", False)
```
**EN:** This block defines `PPMissingLayer.__init__` and contains the main logic for this step. It mainly invokes `super.__init__` and `kwargs.get`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.return_tuple` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `PPMissingLayer.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__` 和 `kwargs.get`，说明该流程会编排底层辅助函数或计算内核。 像 `self.return_tuple` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 120-127: `PPMissingLayer.forward` main forward path
```python
    def forward(self, *args, **kwargs):
        """
        Return the first arg from args or the first value from kwargs.

        Wraps the input in a tuple if `self.return_tuple` is True.
        """
        input = args[0] if args else next(iter(kwargs.values()))
        return (input,) if self.return_tuple else input
```
**EN:** This block defines `PPMissingLayer.forward` and contains the main logic for this step. It mainly invokes `next`, `iter`, and `kwargs.values`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `input` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `PPMissingLayer.forward`，并承载这一阶段的核心逻辑。 它主要调用 `next`、`iter` 以及 `kwargs.values`，说明该流程会编排底层辅助函数或计算内核。 像 `input` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `get_layer_id`, `pad_or_narrow_weight`, `is_strict_contiguous`, `strict_contiguous`, and `copy_or_rebind_param`. / **主要符号**：核心入口包括 `get_layer_id`、`pad_or_narrow_weight`、`is_strict_contiguous`、`strict_contiguous` 以及 `copy_or_rebind_param`。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。

## Dependencies / 依赖关系
- **Standard library**: `logging` and `re` / **标准库**：`logging` 和 `re`
- **Third-party**: `torch` and `torch.nn.parameter.Parameter` / **第三方依赖**：`torch` 和 `torch.nn.parameter.Parameter`
