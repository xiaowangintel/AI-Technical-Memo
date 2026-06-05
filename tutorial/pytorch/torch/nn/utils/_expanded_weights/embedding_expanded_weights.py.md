# embedding_expanded_weights.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/utils/_expanded_weights/embedding_expanded_weights.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Collects helper routines that reshape, inspect, or adapt nn.Module behavior.
- **Purpose (CN)**: 汇集用于重塑、检查或适配 nn.Module 行为的辅助函数。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```python
from typing import Any

import torch
import torch.nn.functional as F

from .expanded_weights_impl import implements_per_sample_grads
from .expanded_weights_utils import (
    forward_helper,
    set_grad_sample_if_exists,
    standard_kwargs,
)
```
- **EN**: This module-level block helps organize reusable module behavior and state.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。

### Lines 13-20
```python

@implements_per_sample_grads(F.embedding)
class EmbeddingPerSampleGrad(torch.autograd.Function):
    @staticmethod
    # pyrefly: ignore [bad-override]
    def forward(
        ctx: Any, kwarg_names: list[str], _: Any, *expanded_args_and_kwargs: Any
    ) -> torch.Tensor:
```
- **EN**: These decorators register or transform the following definition so it can organize reusable module behavior and state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够组织可复用的模块行为与状态。

### Lines 21-28
```python
        expanded_args, expanded_kwargs = standard_kwargs(
            kwarg_names, expanded_args_and_kwargs
        )
        if len(expanded_args[0].shape) == 1:
            raise RuntimeError(
                f"Expanded Weights needs an input with a batch size, got a 1D tensor, {expanded_args[0]}"
            )
        output = forward_helper(F.embedding, expanded_args, expanded_kwargs)
```
- **EN**: This block continues `EmbeddingPerSampleGrad.forward` and works to track symbolic shape constraints and shape-dependent reasoning. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `EmbeddingPerSampleGrad.forward`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 29-38
```python
        ctx.input, ctx.weight = expanded_args
        ctx.padding_idx, ctx.scale_grad_by_freq = (
            expanded_kwargs["padding_idx"],
            expanded_kwargs["scale_grad_by_freq"],
        )
        ctx.sparse = expanded_kwargs["sparse"]
        return output

    @staticmethod
    # pyrefly: ignore [bad-override]
```
- **EN**: These decorators register or transform the following definition so it can connect execution with differentiation-aware logic.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将执行过程与可微分逻辑连接起来。

### Lines 39-48
```python
    def backward(
        ctx: Any, grad_output: torch.Tensor
    ) -> tuple[torch.Tensor | None, ...]:
        input, weight = ctx.input, ctx.weight
        padding_idx, scale_grad_by_freq, sparse = (
            ctx.padding_idx,
            ctx.scale_grad_by_freq,
            ctx.sparse,
        )
```
- **EN**: Defines the `EmbeddingPerSampleGrad.backward` method; this block introduces logic that connect execution with differentiation-aware logic.
- **CN**: 定义`EmbeddingPerSampleGrad.backward` 方法；该代码块引入了用于将执行过程与可微分逻辑连接起来的逻辑。

### Lines 49-56
```python
        def weight_per_sample_grad(weight: torch.Tensor) -> torch.Tensor:
            batch_size = input.shape[0]
            embedding_dim = weight.shape[1]
            index = (
                input.unsqueeze(-1)
                .expand(*input.shape, embedding_dim)
                .reshape(batch_size, -1, embedding_dim)
            )
```
- **EN**: Defines the `EmbeddingPerSampleGrad.backward.weight_per_sample_grad` method; this block introduces logic that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 定义`EmbeddingPerSampleGrad.backward.weight_per_sample_grad` 方法；该代码块引入了用于跟踪符号形状约束以及依赖形状的推理的逻辑。

### Lines 57-67
```python
            grad_sample = torch.zeros(  # type: ignore[attr-defined]
                batch_size, *weight.shape, device=weight.device, dtype=grad_output.dtype
            )
            return grad_sample.scatter_add_(
                1, index, grad_output.reshape(batch_size, -1, embedding_dim)
            )

        results: list[torch.Tensor | None] = []
        results.append(None)  # for kwarg names
        results.append(None)  # for op reference
```
- **EN**: This block continues `EmbeddingPerSampleGrad.backward` and works to track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `EmbeddingPerSampleGrad.backward`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果。

### Lines 68-75
```python
        if input.requires_grad:
            bw_fn = torch.ops.aten.embedding_backward
            results.append(
                bw_fn(
                    grad_output,
                    input,
                    weight.shape[0],
                    padding_idx,
```
- **EN**: This block continues `EmbeddingPerSampleGrad.backward` and works to track symbolic shape constraints and shape-dependent reasoning. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `EmbeddingPerSampleGrad.backward`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会根据运行时条件分支处理。

### Lines 76-85
```python
                    scale_grad_by_freq,
                    sparse,
                )
            )
        else:
            results.append(None)

        # weight doesn't compute batched gradients; no other arguments are differentiable (2 not saved from forward)
        results = results + [None] * 6
```
- **EN**: This block continues `EmbeddingPerSampleGrad.backward` and works to organize reusable module behavior and state.
- **CN**: 该代码块继续实现 `EmbeddingPerSampleGrad.backward`，用于组织可复用的模块行为与状态。

### Lines 86-88
```python
        # set grad_sample field for weight with per sample gradients
        set_grad_sample_if_exists(weight, weight_per_sample_grad)
        return tuple(results)
```
- **EN**: This block continues `EmbeddingPerSampleGrad.backward` and works to connect execution with differentiation-aware logic. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `EmbeddingPerSampleGrad.backward`，用于将执行过程与可微分逻辑连接起来。 同时它还会计算并返回中间值或结果。

## Key Concepts / 关键概念

- **Neural-network abstractions / 神经网络抽象**
  - EN: The file contributes to reusable neural-network layers, functionals, or support utilities.
  - CN: 该文件为可复用的神经网络层、函数式接口或辅助工具提供实现。
- **Symbolic shapes / 符号形状**
  - EN: The file reasons about tensor extents symbolically instead of requiring concrete integers everywhere.
  - CN: 该文件以符号方式推理张量形状，而不是处处要求具体整数。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.nn.functional`, `.expanded_weights_impl`, `.expanded_weights_utils`
- **Standard library / 标准库**: `typing`
- **Primary symbols / 核心符号**: `EmbeddingPerSampleGrad`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。
