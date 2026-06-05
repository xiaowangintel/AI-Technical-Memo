# model_parallel.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/model_parallel.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements runtime layer support for the SGLang SRT runtime. It exposes symbols such as `_shard_tensor`, `ColwiseParallelSharded`, `RowwiseParallelMaybeWait`, and `tensor_parallel` and connects them to backend-specific paths such as `CUDA` and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了运行时层支持。它提供了 `_shard_tensor`、`ColwiseParallelSharded`、`RowwiseParallelMaybeWait` 以及 `tensor_parallel` 等符号，并把这些符号连接到 `CUDA` 和 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Imports, conditional backend setup, and runtime guards
```python
"""
Common utilities for torch model parallelism.
"""

from typing import Optional, Sequence

import torch
import torch.nn as nn
from torch.distributed.device_mesh import DeviceMesh

try:
    import torch.distributed.tensor as dt
except ImportError:
    # torch 2.4 or older
    import torch.distributed._tensor as dt

from torch.distributed.tensor.parallel import (
    ColwiseParallel,
    RowwiseParallel,
    parallelize_module,
)
```
**EN:** This section prepares the module namespace. It imports `typing.Optional`, `typing.Sequence`, `torch`, `torch.nn`, `torch.distributed.device_mesh.DeviceMesh`, and `torch.distributed.tensor`, so later blocks can reuse runtime, tensor, or backend helpers. Control structures like `Try` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `typing.Optional`、`typing.Sequence`、`torch`、`torch.nn`、`torch.distributed.device_mesh.DeviceMesh` 以及 `torch.distributed.tensor`，让后续代码可以复用运行时、张量或后端辅助逻辑。 `Try` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 24-67: Internal helper `_shard_tensor`
```python
def _shard_tensor(
    full_tensor: torch.Tensor,
    device_mesh: DeviceMesh,
    placements: Sequence[dt.Shard],
) -> "dt.DTensor":
    """
    Locally shards a full tensor based on indicated sharding arrangement, and
    returns a DTensor containing the local shard.

    .. warning:: This is a private API that is subject to change. It skips the
        communication otherwise required by `distribute_tensor`. It is only
        applicable to cases where all ranks have the same `full_tensor`. For
        example, in distributed inference all ranks load from the same
        checkpoint. This API will not check for data equality between ranks, it
        is thus user's responsibility to ensure the `full_tensor` is the same
        across ranks.

    Args:
        full_tensor (torch.Tensor): the full tensor to be sharded.
        device_mesh (:class:`DeviceMesh`): DeviceMesh to place the
            DTensor.  Must have same dimension as the number of placements.
        placements (Sequence[:class:`Shard`]): the placements that
            describes how to place the local tensor on DeviceMesh.

    Returns:
        A :class:`DTensor` object with the shard as its local tensor.

    Examples:
        >>> # xdoctest: +SKIP("need world_size and rank")
        >>> device_mesh = dist.init_device_mesh("cuda", (world_size,))
        >>> full_tensor = torch.arange(world_size, device=f"cuda:{rank}")
        >>> dtensor = _shard_tensor(full_tensor, device_mesh, [Shard(1)])
    """
    shape, offset = dt._utils.compute_local_shape_and_global_offset(
        full_tensor.shape, device_mesh, placements
    )
    slices = [
        slice(cur_offset, cur_offset + cur_shape)
        for cur_shape, cur_offset in zip(shape, offset)
    ]
    local_tensor = full_tensor[slices]
    return dt.DTensor.from_local(local_tensor, device_mesh, placements)
```
**EN:** This block defines `_shard_tensor` and contains the main logic for this step. It mainly invokes `dt._utils.compute_local_shape_and_global_offset`, `dt.DTensor.from_local`, `slice`, and `zip`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `shape`, `offset`, `slices`, and `local_tensor` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_shard_tensor`，并承载这一阶段的核心逻辑。 它主要调用 `dt._utils.compute_local_shape_and_global_offset`、`dt.DTensor.from_local`、`slice` 以及 `zip`，说明该流程会编排底层辅助函数或计算内核。 像 `shape`、`offset`、`slices` 以及 `local_tensor` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 68-75: Class `ColwiseParallelSharded` declaration and shared state
```python
class ColwiseParallelSharded(ColwiseParallel):
    """
    A version of ColwiseParallel where the local weight has been already
    sharded.  This is used for the fused wqkv case, where during loading, we
    already sharded wq, wk, wv before fusing them.
    """

    # Override the _partition_linear_fn in ColwiseParallel
```
**EN:** This block introduces class `ColwiseParallelSharded` and the state shared by its methods. It inherits from `ColwiseParallel`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: A version of ColwiseParallel where the local weight has been already sharded.
**CN:** 该代码块引入类 `ColwiseParallelSharded`，并定义其方法共享的状态。 它继承自 `ColwiseParallel`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 76-85: Internal helper `ColwiseParallelSharded._partition_linear_fn`
```python
    def _partition_linear_fn(self, name, module, device_mesh):
        # colwise shard weight/bias to Shard(0), weight be Shard(0)
        # means Colwise as Linear is input * weight^T + bias, where
        # weight would become Shard(1)
        for name, param in module.named_parameters():
            dtensor = dt.DTensor.from_local(param, device_mesh, [dt.Shard(0)])
            dist_param = torch.nn.Parameter(dtensor, requires_grad=False)
            module.register_parameter(name, dist_param)
```
**EN:** This block defines `ColwiseParallelSharded._partition_linear_fn` and contains the main logic for this step. It mainly invokes `module.named_parameters`, `dt.DTensor.from_local`, `torch.nn.Parameter`, `module.register_parameter`, and `dt.Shard`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `dtensor` and `dist_param` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `ColwiseParallelSharded._partition_linear_fn`，并承载这一阶段的核心逻辑。 它主要调用 `module.named_parameters`、`dt.DTensor.from_local`、`torch.nn.Parameter`、`module.register_parameter` 以及 `dt.Shard`，说明该流程会编排底层辅助函数或计算内核。 像 `dtensor` 和 `dist_param` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 86-93: Class `RowwiseParallelMaybeWait` declaration and shared state
```python
class RowwiseParallelMaybeWait(RowwiseParallel):
    """
    A version of RowwiseParallel that waits for the output (establish dependency
    between comm stream and compute stream in CUDA sense) before going into the
    next op. This is needed to workaround the current interaction between
    AsyncCollectiveTensor and multi-platform ops, such as `RMSNorm`.
    """
```
**EN:** This block introduces class `RowwiseParallelMaybeWait` and the state shared by its methods. It inherits from `RowwiseParallel`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: A version of RowwiseParallel that waits for the output (establish dependency between comm stream and compute stream in CUDA sense) before going into the next op.
**CN:** 该代码块引入类 `RowwiseParallelMaybeWait`，并定义其方法共享的状态。 它继承自 `RowwiseParallel`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 94-110: Internal helper `RowwiseParallelMaybeWait._partition_linear_fn`
```python
    def _partition_linear_fn(self, name, module, device_mesh):
        # Rowwise shard weight to Shard(1), bias to Replicate(), weight be Shard(1)
        # means Rowwise as nn.Linear is input * weight^T + bias, where
        # weight would become Shard(0)
        module.register_parameter(
            "weight",
            nn.Parameter(_shard_tensor(module.weight, device_mesh, [dt.Shard(1)])),
        )
        if getattr(module, "bias", None) is not None:
            # The Linear module has bias
            module.register_parameter(
                "bias",
                nn.Parameter(
                    dt.distribute_tensor(module.bias, device_mesh, [dt.Replicate()])
                ),
            )
```
**EN:** This block defines `RowwiseParallelMaybeWait._partition_linear_fn` and contains the main logic for this step. It mainly invokes `module.register_parameter`, `nn.Parameter`, `getattr`, `_shard_tensor`, and `dt.distribute_tensor`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `RowwiseParallelMaybeWait._partition_linear_fn`，并承载这一阶段的核心逻辑。 它主要调用 `module.register_parameter`、`nn.Parameter`、`getattr`、`_shard_tensor` 以及 `dt.distribute_tensor`，说明该流程会编排底层辅助函数或计算内核。

### Lines 111-120: Internal helper `RowwiseParallelMaybeWait._prepare_output_fn`
```python
    @staticmethod
    def _prepare_output_fn(output_layouts, use_local_output, mod, outputs, device_mesh):
        outputs = super(
            RowwiseParallelMaybeWait, RowwiseParallelMaybeWait
        )._prepare_output_fn(
            output_layouts, use_local_output, mod, outputs, device_mesh
        )
        return torch.distributed._functional_collectives.wait_tensor(outputs)
```
**EN:** This block defines `RowwiseParallelMaybeWait._prepare_output_fn` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `super._prepare_output_fn` and `torch.distributed._functional_collectives.wait_tensor`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `outputs` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RowwiseParallelMaybeWait._prepare_output_fn`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `super._prepare_output_fn` 和 `torch.distributed._functional_collectives.wait_tensor`，说明该流程会编排底层辅助函数或计算内核。 像 `outputs` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 121-155: Function `tensor_parallel` and its core logic
```python
def tensor_parallel(
    module: torch.nn.Module,
    device_mesh: Optional[DeviceMesh] = None,
):
    """
    Tensor parallelize the model across the given device mesh.
    Args:
        module (`torch.nn.Module`):
            The module to tensor parallelize.
        device_mesh (`torch.distributed.DeviceMesh`):
            The device mesh to use for tensor parallelism.
    """

    # Tensor parallelize a nn.Module based on the `_tp_plan` attribute of the module.
    # No op if `_tp_plan` attribute does not exist under the module.
    # This is a helper function to be used with `model.apply` to recursively
    # parallelize a model.
    def tplize(mod: torch.nn.Module) -> None:
        tp_plan = getattr(mod, "_tp_plan", None)
        if tp_plan is None:
            return
        for child_name, tp_style in tp_plan.items():
            submod = mod.get_submodule(child_name)
            if tp_style == "Colwise":
                parallelize_module(submod, device_mesh, ColwiseParallel())
            elif tp_style == "Rowwise":
                parallelize_module(submod, device_mesh, RowwiseParallelMaybeWait())
            elif tp_style == "Colwise_Sharded":
                parallelize_module(submod, device_mesh, ColwiseParallelSharded())
            else:
                raise ValueError(f"Unknown TP style {tp_style}")

    # `apply` is a native method of `nn.Module` that recursively applies a
    # function to every submodule.
    module.apply(tplize)
```
**EN:** This block defines `tensor_parallel` and contains the main logic for this step. It mainly invokes `module.apply`, `getattr`, `tp_plan.items`, `mod.get_submodule`, and `parallelize_module`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `tp_plan` and `submod` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `tensor_parallel`，并承载这一阶段的核心逻辑。 它主要调用 `module.apply`、`getattr`、`tp_plan.items`、`mod.get_submodule` 以及 `parallelize_module`，说明该流程会编排底层辅助函数或计算内核。 像 `tp_plan` 和 `submod` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `_shard_tensor`, `ColwiseParallelSharded`, `RowwiseParallelMaybeWait`, and `tensor_parallel`. / **主要符号**：核心入口包括 `_shard_tensor`、`ColwiseParallelSharded`、`RowwiseParallelMaybeWait` 以及 `tensor_parallel`。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Projection layers**: Focuses on matrix multiplication, weight layout, and projection-oriented wrappers. / **投影层**：关注矩阵乘法、权重布局与面向投影的包装层。

## Dependencies / 依赖关系
- **Standard library**: `typing.Optional` and `typing.Sequence` / **标准库**：`typing.Optional` 和 `typing.Sequence`
- **Third-party**: `torch`, `torch.nn`, `torch.distributed.device_mesh.DeviceMesh`, `torch.distributed.tensor.parallel.ColwiseParallel`, `torch.distributed.tensor.parallel.RowwiseParallel`, `torch.distributed.tensor.parallel.parallelize_module`, `torch.distributed.tensor`, and `torch.distributed._tensor` / **第三方依赖**：`torch`、`torch.nn`、`torch.distributed.device_mesh.DeviceMesh`、`torch.distributed.tensor.parallel.ColwiseParallel`、`torch.distributed.tensor.parallel.RowwiseParallel`、`torch.distributed.tensor.parallel.parallelize_module`、`torch.distributed.tensor` 以及 `torch.distributed._tensor`
