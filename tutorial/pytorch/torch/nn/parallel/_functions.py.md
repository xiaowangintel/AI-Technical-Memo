# _functions.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/parallel/_functions.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Provides wrappers and helpers for running neural-network modules across devices or processes.
- **Purpose (CN)**: 提供跨设备或进程运行神经网络模块的包装器与辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```python
import warnings
from itertools import chain

import torch
from torch._utils import _get_device_index
from torch.autograd import Function
from torch.nn.parallel import comm


class Broadcast(Function):
    @staticmethod
    def forward(ctx, target_gpus, *inputs):
```
- **EN**: These decorators register or transform the following definition so it can organize reusable module behavior and state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够组织可复用的模块行为与状态。

### Lines 13-30
```python
        if not all(i.device.type != "cpu" for i in inputs):
            raise AssertionError("Broadcast function not implemented for CPU tensors")
        target_gpus = [_get_device_index(x, True) for x in target_gpus]
        ctx.target_gpus = target_gpus
        if len(inputs) == 0:
            return ()
        ctx.num_inputs = len(inputs)
        ctx.input_device = inputs[0].get_device()

        ctx.complex_mask = [inp.is_complex() for inp in inputs]

        outputs = comm.broadcast_coalesced(inputs, ctx.target_gpus)

        for device_outputs in outputs:
            for i, is_complex in enumerate(ctx.complex_mask):
                if is_complex:
                    device_outputs[i] = torch.view_as_complex(device_outputs[i])
```
- **EN**: This block continues `Broadcast.forward` and works to normalize dtype/device related arguments and behavior. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `Broadcast.forward`，用于规范化 dtype/device 相关参数与行为。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

### Lines 31-46
```python
        non_differentiables = []
        for idx, input_requires_grad in enumerate(ctx.needs_input_grad[1:]):
            if not input_requires_grad:
                non_differentiables.extend(output[idx] for output in outputs)
        ctx.mark_non_differentiable(*non_differentiables)
        return tuple(chain.from_iterable(outputs))

    @staticmethod
    def backward(ctx, *grad_outputs):
        grads = ReduceAddCoalesced.apply(
            ctx.input_device, ctx.num_inputs, *grad_outputs
        )

        return (None,) + grads
```
- **EN**: These decorators register or transform the following definition so it can connect execution with differentiation-aware logic.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将执行过程与可微分逻辑连接起来。

### Lines 47-60
```python
class ReduceAddCoalesced(Function):
    @staticmethod
    def forward(ctx, destination, num_inputs, *grads):
        ctx.target_gpus = [
            grads[i].get_device() for i in range(0, len(grads), num_inputs)
        ]

        complex_mask = [grads[i].is_complex() for i in range(num_inputs)]
        ctx.complex_mask = complex_mask

        grads_converted = tuple(
            torch.view_as_real(g) if g.is_complex() else g for g in grads
        )
```
- **EN**: These decorators register or transform the following definition so it can organize reusable module behavior and state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够组织可复用的模块行为与状态。

### Lines 61-73
```python
        grads_ = [
            grads_converted[i : i + num_inputs]
            for i in range(0, len(grads_converted), num_inputs)
        ]
        results = comm.reduce_add_coalesced(grads_, destination)

        results = tuple(
            torch.view_as_complex(r) if is_complex else r
            for r, is_complex in zip(results, complex_mask)
        )

        return results
```
- **EN**: This block continues `ReduceAddCoalesced` and works to connect execution with differentiation-aware logic. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `ReduceAddCoalesced`，用于将执行过程与可微分逻辑连接起来。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构。

### Lines 74-85
```python
    @staticmethod
    def backward(ctx, *grad_outputs):
        return (
            None,
            None,
        ) + Broadcast.apply(ctx.target_gpus, *grad_outputs)


class Gather(Function):
    @staticmethod
    def forward(ctx, target_device, dim, *inputs):
        if not all(i.device.type != "cpu" for i in inputs):
```
- **EN**: These decorators register or transform the following definition so it can organize reusable module behavior and state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够组织可复用的模块行为与状态。

### Lines 86-97
```python
            raise AssertionError("Gather function not implemented for CPU tensors")
        if target_device == "cpu":
            ctx.target_device = "cpu"
        else:
            target_device = _get_device_index(target_device, True)
            ctx.target_device = target_device
        ctx.dim = dim
        ctx.input_gpus = tuple(i.get_device() for i in inputs)
        if all(t.dim() == 0 for t in inputs) and dim == 0:
            inputs = tuple(t.view(1) for t in inputs)
            warnings.warn(
                "Was asked to gather along dimension 0, but all "
```
- **EN**: This block continues `Gather.forward` and works to normalize dtype/device related arguments and behavior. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `Gather.forward`，用于规范化 dtype/device 相关参数与行为。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 98-115
```python
                "input tensors were scalars; will instead unsqueeze "
                "and return a vector.",
                stacklevel=2,
            )
            ctx.unsqueezed_scalar = True
        else:
            ctx.unsqueezed_scalar = False
        ctx.input_sizes = tuple(i.size(ctx.dim) for i in inputs)

        is_complex = len(inputs) > 0 and inputs[0].is_complex()

        output = comm.gather(inputs, ctx.dim, ctx.target_device)

        if is_complex:
            output = torch.view_as_complex(output)

        return output
```
- **EN**: This block continues `Gather` and works to normalize dtype/device related arguments and behavior. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `Gather`，用于规范化 dtype/device 相关参数与行为。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 116-127
```python
    @staticmethod
    def backward(ctx, grad_output):
        scattered_grads = Scatter.apply(
            ctx.input_gpus, ctx.input_sizes, ctx.dim, grad_output
        )
        if ctx.unsqueezed_scalar:
            scattered_grads = tuple(g[0] for g in scattered_grads)
        return (None, None) + scattered_grads


class Scatter(Function):
    @staticmethod
```
- **EN**: These decorators register or transform the following definition so it can connect execution with differentiation-aware logic.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将执行过程与可微分逻辑连接起来。

### Lines 128-143
```python
    def forward(ctx, target_gpus, chunk_sizes, dim, input):
        target_gpus = [_get_device_index(x, True) for x in target_gpus]
        ctx.dim = dim
        ctx.input_device = input.get_device() if input.device.type != "cpu" else -1
        streams = None
        if torch.accelerator.is_available() and ctx.input_device == -1:
            # Perform CPU to GPU copies in a background stream
            streams = [_get_stream(torch.device(device)) for device in target_gpus]

        is_complex = input.is_complex()

        outputs = comm.scatter(input, target_gpus, chunk_sizes, ctx.dim, streams)

        if is_complex:
            outputs = tuple(torch.view_as_complex(o) for o in outputs)
```
- **EN**: Defines the `Scatter.forward` method; this block introduces logic that organize reusable module behavior and state.
- **CN**: 定义`Scatter.forward` 方法；该代码块引入了用于组织可复用的模块行为与状态的逻辑。

### Lines 144-161
```python
        # Synchronize with the copy stream
        if streams is not None:
            for i, output in enumerate(outputs):
                with torch.accelerator.device_index(target_gpus[i]):
                    main_stream = torch.accelerator.current_stream()
                    main_stream.wait_stream(streams[i])
                    output.record_stream(main_stream)
        return outputs

    @staticmethod
    def backward(ctx, *grad_output):
        return None, None, None, Gather.apply(ctx.input_device, ctx.dim, *grad_output)


# background streams used for copying
_streams: list[torch.Stream | None] | None = None
```
- **EN**: These decorators register or transform the following definition so it can connect execution with differentiation-aware logic.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将执行过程与可微分逻辑连接起来。

### Lines 162-173
```python
def _get_stream(device: torch.device):
    """Get a background stream for copying between CPU and target device."""
    global _streams
    if device.type == "cpu" or not torch.accelerator.is_available():
        return None
    if torch.accelerator.current_accelerator().type != device.type:
        raise AssertionError(
            f"Expected current accelerator type {torch.accelerator.current_accelerator().type} "
            f"to match device type {device.type}"
        )
    if _streams is None:
        _streams = [None] * torch.accelerator.device_count()
```
- **EN**: Defines the `_get_stream` function; this block introduces logic that normalize dtype/device related arguments and behavior.
- **CN**: 定义`_get_stream` 函数；该代码块引入了用于规范化 dtype/device 相关参数与行为的逻辑。

### Lines 174-176
```python
    if _streams[device.index] is None:
        _streams[device.index] = torch.Stream(device.index)
    return _streams[device.index]
```
- **EN**: This block continues `_get_stream` and works to normalize dtype/device related arguments and behavior. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_get_stream`，用于规范化 dtype/device 相关参数与行为。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

## Key Concepts / 关键概念

- **Neural-network abstractions / 神经网络抽象**
  - EN: The file contributes to reusable neural-network layers, functionals, or support utilities.
  - CN: 该文件为可复用的神经网络层、函数式接口或辅助工具提供实现。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch._utils`, `torch.autograd`, `torch.nn.parallel`
- **Standard library / 标准库**: `warnings`, `itertools`
- **Primary symbols / 核心符号**: `Broadcast`, `ReduceAddCoalesced`, `Gather`, `Scatter`, `_get_stream`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。
