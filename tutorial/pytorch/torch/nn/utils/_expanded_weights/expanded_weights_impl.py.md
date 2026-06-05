# expanded_weights_impl.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/utils/_expanded_weights/expanded_weights_impl.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Collects helper routines that reshape, inspect, or adapt nn.Module behavior.
- **Purpose (CN)**: 汇集用于重塑、检查或适配 nn.Module 行为的辅助函数。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```python
# mypy: allow-untyped-defs
import functools
from collections.abc import Callable
from contextlib import contextmanager

import torch
from torch._decomp import decomposition_table
from torch.utils._pytree import tree_map_only


HANDLED_FUNCTIONS: dict[Callable, torch.autograd.Function] = {}
```
- **EN**: This module-level block helps connect execution with differentiation-aware logic.
- **CN**: 这个模块级代码块用于将执行过程与可微分逻辑连接起来。

### Lines 13-24
```python
aten = torch._ops.ops.aten
# __torch_function__ runs before the pydispatcher so we need to manually use the same
# decompositions indexed by their torch equivalent
expanded_weights_rnn_decomps = {
    # func: (input_decomp, data_decomp)
    torch.rnn_relu: (
        decomposition_table[aten.rnn_relu.input],
        decomposition_table[aten.rnn_relu.data],
    ),
    torch.rnn_tanh: (
        decomposition_table[aten.rnn_tanh.input],
        decomposition_table[aten.rnn_tanh.data],
```
- **EN**: This module-level block helps route operators through dispatch and decomposition helpers.
- **CN**: 这个模块级代码块用于通过分发与分解辅助逻辑路由算子。

### Lines 25-42
```python
    ),
    torch.lstm: (
        decomposition_table[aten.lstm.input],
        decomposition_table[aten.lstm.data],
    ),
    torch.gru: (
        decomposition_table[aten.gru.input],
        decomposition_table[aten.gru.data],
    ),
}


# all of the RNN decomps run linear with the batch dimension second, even if batch_first was set
@contextmanager
def batch_second(args, kwargs):
    def set_batch_second(ew) -> None:
        ew.set_batch_first(False)
```
- **EN**: These decorators register or transform the following definition so it can route operators through dispatch and decomposition helpers.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够通过分发与分解辅助逻辑路由算子。

### Lines 43-60
```python
    def reset_batch_first(ew) -> None:
        ew.set_batch_first(True)

    tree_map_only(ExpandedWeight, set_batch_second, args)
    tree_map_only(ExpandedWeight, set_batch_second, kwargs)
    try:
        yield
    finally:
        tree_map_only(ExpandedWeight, reset_batch_first, args)
        tree_map_only(ExpandedWeight, reset_batch_first, kwargs)


# to support packed sequences, we need to allow for smaller batches. Expanded weights represents the largest batch
@contextmanager
def allow_smaller_batches(args, kwargs):
    def allow(ew) -> None:
        ew.set_allow_smaller_batches(True)
```
- **EN**: These decorators register or transform the following definition so it can prepare neural-network operators or module behavior.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够准备神经网络算子或模块行为。

### Lines 61-73
```python
    def reset(ew) -> None:
        ew.set_allow_smaller_batches(False)

    tree_map_only(ExpandedWeight, allow, args)
    tree_map_only(ExpandedWeight, allow, kwargs)
    try:
        yield
    finally:
        tree_map_only(ExpandedWeight, reset, args)
        tree_map_only(ExpandedWeight, reset, kwargs)


@contextmanager
```
- **EN**: These decorators register or transform the following definition so it can prepare neural-network operators or module behavior.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够准备神经网络算子或模块行为。

### Lines 74-91
```python
def setup_rnn(use_input_variant, args, kwargs):
    with (
        batch_second(args, kwargs)
        if use_input_variant
        else allow_smaller_batches(args, kwargs)
    ):
        yield


def implements_per_sample_grads(torch_function):
    @functools.wraps(torch_function)
    def decorator(autograd_func):
        HANDLED_FUNCTIONS[torch_function] = autograd_func
        return autograd_func

    return decorator
```
- **EN**: These decorators register or transform the following definition so it can connect execution with differentiation-aware logic.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将执行过程与可微分逻辑连接起来。

### Lines 92-104
```python
# ExpandedWeight represents a weight (parameter) Tensor that has an expanded
# batch dimension. Operations on the ExpandedWeight Tensor act exactly like
# those without an expanded batch dimension but a call to .backward() populates
# the original (unexpanded) tensor with per-sample-gradients for in the grad_sample field
#
# ExpandedWeight has a fallback that always fails since we cannot know what the batch
# dimension of the input tensor is and therefore cannot know if this is a valid call
#
# This is a __torch_function__ object but it could have also been a Tensor Extension
# with a dispatch key.
#
# Needs to be a tensor subclass to allow reparameterization
class ExpandedWeight(torch.Tensor):
```
- **EN**: This module-level block helps organize reusable module behavior and state.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。

### Lines 105-116
```python
    def __init__(self, orig_weight, batch_size, loss_reduction) -> None:
        self.batch_size = batch_size
        self.batch_first = True
        self.allow_smaller_batches = False
        self.orig_weight = orig_weight
        self.loss_reduction = loss_reduction

    handled_functions = HANDLED_FUNCTIONS

    def __new__(cls, orig_weight, batch_size, loss_reduction):
        if not isinstance(orig_weight, torch.Tensor):
            raise RuntimeError(
```
- **EN**: Declares `ExpandedWeight(torch.Tensor)`; this class packages state and methods that initialize learnable tensors and related state.
- **CN**: 声明 `ExpandedWeight(torch.Tensor)`；该类封装了状态与方法，用于初始化可学习张量及相关状态。

### Lines 117-128
```python
                f"Can only make Expanded Weights of Tensors, got {type(orig_weight).__name__}"
            )
        if not orig_weight.requires_grad:
            raise RuntimeError(
                "Can only build ExpandedWeights objects of tensors that require_grad"
            )
        ret = torch.Tensor._make_subclass(cls, orig_weight, True)
        return ret

    @classmethod
    def __torch_function__(cls, func, _, args=(), kwargs=None):
        if kwargs is None:
```
- **EN**: These decorators register or transform the following definition so it can connect execution with differentiation-aware logic.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将执行过程与可微分逻辑连接起来。

### Lines 129-140
```python
            kwargs = {}
        if func in expanded_weights_rnn_decomps:
            # in aten, choosing the input or data variants is done by parsing logic. This mimics some of that
            decomp_opts = expanded_weights_rnn_decomps[func]
            use_input_variant = isinstance(
                # pyrefly: ignore [bad-index]
                args[2],
                list,
            )  # data variant uses a list here
            decomp = decomp_opts[0] if use_input_variant else decomp_opts[1]

            if decomp is not None:
```
- **EN**: This block continues `ExpandedWeight.__torch_function__` and works to prepare neural-network operators or module behavior. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `ExpandedWeight.__torch_function__`，用于准备神经网络算子或模块行为。 同时它还会根据运行时条件分支处理。

### Lines 141-155
```python
                with setup_rnn(use_input_variant, args, kwargs):
                    return decomp(*args, **kwargs)
        if func is torch._cudnn_rnn_flatten_weight:
            # since we aren't using the fused cuda kernels for RNNs, don't do this
            return
        if func in cls.handled_functions:
            return cls.handled_functions[func].apply(
                tuple(kwargs.keys()), func, *(args + tuple(kwargs.values()))
            )
        # We cannot use a fallback here because we do not know the batch dimension for any regular tensor inputs,
        # i.e. torch.add(torch.Tensor, ExpandedWeight)
        raise RuntimeError(
            f"Expanded Weights encountered but cannot handle function {func.__name__}"
        )
```
- **EN**: This block continues `ExpandedWeight` and works to validate invariants and surface meaningful failures. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `ExpandedWeight`，用于校验不变量并给出有意义的失败信息。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 156-171
```python
    @property
    def dtype(self):  # type: ignore[override]
        return self.orig_weight.dtype

    @property
    def data(self):  # type: ignore[override]
        return self.orig_weight.data

    @property
    def shape(self):  # type: ignore[override]
        return self.orig_weight.shape

    @property
    def device(self):  # type: ignore[override]
        return self.orig_weight.device
```
- **EN**: These decorators register or transform the following definition so it can track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够跟踪符号形状约束以及依赖形状的推理。

### Lines 172-184
```python
    @property
    def is_cuda(self):  # type: ignore[override]
        return self.orig_weight.is_cuda

    def data_ptr(self):
        return self.orig_weight.data_ptr()

    def get_device(self):
        return self.orig_weight.get_device()

    def set_allow_smaller_batches(self, is_allow_smaller_batches) -> None:
        self.allow_smaller_batches = is_allow_smaller_batches
```
- **EN**: These decorators register or transform the following definition so it can normalize dtype/device related arguments and behavior.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够规范化 dtype/device 相关参数与行为。

### Lines 185-186
```python
    def set_batch_first(self, is_batch_first=True) -> None:
        self.batch_first = is_batch_first
```
- **EN**: Defines the `ExpandedWeight.set_batch_first` method; this block introduces logic that prepare neural-network operators or module behavior.
- **CN**: 定义`ExpandedWeight.set_batch_first` 方法；该代码块引入了用于准备神经网络算子或模块行为的逻辑。

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
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch._decomp`, `torch.utils._pytree`
- **Standard library / 标准库**: `functools`, `collections.abc`, `contextlib`
- **Primary symbols / 核心符号**: `HANDLED_FUNCTIONS`, `batch_second`, `allow_smaller_batches`, `setup_rnn`, `implements_per_sample_grads`, `ExpandedWeight`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。
