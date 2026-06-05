# comm.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/parallel/comm.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Provides wrappers and helpers for running neural-network modules across devices or processes.
- **Purpose (CN)**: 提供跨设备或进程运行神经网络模块的包装器与辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```python
# mypy: allow-untyped-defs
import warnings

import torch
from torch._utils import (
    _flatten_dense_tensors,
    _get_device_index,
    _handle_complex,
    _reorder_tensors_as,
    _take_tensors,
    _unflatten_dense_tensors,
)
from torch.cuda import nccl


def broadcast(tensor, devices=None, *, out=None):
    r"""Broadcasts a tensor to specified GPU devices.
```
- **EN**: This module-level block helps normalize dtype/device related arguments and behavior.
- **CN**: 这个模块级代码块用于规范化 dtype/device 相关参数与行为。

### Lines 19-30
```python
    Args:
        tensor (Tensor): tensor to broadcast. Can be on CPU or GPU.
        devices (Iterable[torch.device, str or int], optional): an iterable of
          GPU devices, among which to broadcast.
        out (Sequence[Tensor], optional, keyword-only): the GPU tensors to
          store output results.

    .. note::
        Exactly one of :attr:`devices` and :attr:`out` must be specified.

    Returns:
        - If :attr:`devices` is specified,
```
- **EN**: This block continues `broadcast` and works to normalize dtype/device related arguments and behavior.
- **CN**: 该代码块继续实现 `broadcast`，用于规范化 dtype/device 相关参数与行为。

### Lines 31-48
```python
            a tuple containing copies of :attr:`tensor`, placed on
            :attr:`devices`.
        - If :attr:`out` is specified,
            a tuple containing :attr:`out` tensors, each containing a copy of
            :attr:`tensor`.
    """
    tensor = _handle_complex(tensor)
    if not ((devices is None) ^ (out is None)):
        raise RuntimeError(
            f"Exactly one of 'devices' and 'out' must be specified, but got devices={devices} and out={out}"
        )
    if devices is not None:
        devices = [_get_device_index(d) for d in devices]
        return torch._C._broadcast(tensor, devices)
    else:
        # pyrefly: ignore [bad-argument-type]
        return torch._C._broadcast_out(tensor, out)
```
- **EN**: This module-level block helps normalize dtype/device related arguments and behavior. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于规范化 dtype/device 相关参数与行为。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 49-61
```python

def broadcast_coalesced(tensors, devices, buffer_size=10485760):
    """Broadcast a sequence of tensors to the specified GPUs.

    Small tensors are first coalesced into a buffer to reduce the number of synchronizations.

    Args:
        tensors (sequence): tensors to broadcast. Must be on the same device,
          either CPU or GPU.
        devices (Iterable[torch.device, str or int]): an iterable of GPU
          devices, among which to broadcast.
        buffer_size (int): maximum size of the buffer used for coalescing
```
- **EN**: This module-level block helps organize reusable module behavior and state.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。

### Lines 62-75
```python
    Returns:
        A tuple containing copies of :attr:`tensor`, placed on :attr:`devices`.
    """
    devices = [_get_device_index(d) for d in devices]
    tensors = [_handle_complex(t) for t in tensors]
    return torch._C._broadcast_coalesced(tensors, devices, buffer_size)


def reduce_add(inputs, destination=None):
    """Sum tensors from multiple GPUs.

    All inputs should have matching shapes, dtype, and layout. The output tensor
    will be of the same shape, dtype, and layout.
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果。

### Lines 76-87
```python
    Args:
        inputs (Iterable[Tensor]): an iterable of tensors to add.
        destination (int, optional): a device on which the output will be
            placed (default: current device).

    Returns:
        A tensor containing an elementwise sum of all inputs, placed on the
        :attr:`destination` device.
    """
    destination = _get_device_index(destination, optional=True)
    input_size = inputs[0].size()
    root_index = None  # index of input tensor that already is on the correct device
```
- **EN**: This block continues `reduce_add` and works to normalize dtype/device related arguments and behavior.
- **CN**: 该代码块继续实现 `reduce_add`，用于规范化 dtype/device 相关参数与行为。

### Lines 88-105
```python
    for i, inp in enumerate(inputs):
        if inp.device.type == "cpu":
            raise AssertionError(
                f"reduce_add expects all inputs to be on GPUs, but input {i} is on CPU"
            )
        if inp.get_device() == destination:
            root_index = i
        if inp.size() != input_size:
            got = "x".join(str(x) for x in inp.size())
            expected = "x".join(str(x) for x in input_size)
            raise ValueError(
                f"input {i} has invalid size: got {got}, but expected {expected}"
            )
    if root_index is None:
        raise RuntimeError(
            "reduce_add expects destination to be on the same GPU with one of the tensors"
        )
```
- **EN**: This block continues `reduce_add` and works to normalize dtype/device related arguments and behavior. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `reduce_add`，用于规范化 dtype/device 相关参数与行为。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 106-123
```python
    if len(inputs) == 1:
        return inputs[0]

    if nccl.is_available(inputs):
        result = torch.empty_like(inputs[root_index])
        nccl.reduce(inputs, output=result, root=root_index)
    else:
        destination_device = torch.device(inputs[root_index].device.type, destination)
        nonroot = [t for i, t in enumerate(inputs) if i != root_index]
        # make a new tensor w/o clone
        result = inputs[root_index] + nonroot[0].to(
            device=destination_device, non_blocking=True
        )
        for other in nonroot[1:]:
            result.add_(other.to(device=destination_device, non_blocking=True))
    return result
```
- **EN**: This module-level block helps normalize dtype/device related arguments and behavior. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于规范化 dtype/device 相关参数与行为。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 124-136
```python
def reduce_add_coalesced(inputs, destination=None, buffer_size=10485760):
    """Sum tensors from multiple GPUs.

    Small tensors are first coalesced into a buffer to reduce the number
    of synchronizations.

    Args:
        inputs (Iterable[Iterable[Tensor]]): iterable of iterables that
            contain tensors from a single device.
        destination (int, optional): a device on which the output will be
            placed (default: current device).
        buffer_size (int): maximum size of the buffer used for coalescing
```
- **EN**: Defines the `reduce_add_coalesced` function; this block introduces logic that organize reusable module behavior and state.
- **CN**: 定义`reduce_add_coalesced` 函数；该代码块引入了用于组织可复用的模块行为与状态的逻辑。

### Lines 137-148
```python
    Returns:
        A tuple of tensors containing an elementwise sum of each group of
        inputs, placed on the ``destination`` device.
    """
    # TODO: When `len(inputs) == 1` and all inputs are on `destination`, just
    #       return `inputs`.
    dense_tensors: list[list] = [[] for _ in inputs]  # shape (num_gpus, num_tensors)
    output = []
    ref_order = []
    # process sparse ones first since they may have different sizes on different gpus
    for tensor_at_gpus in zip(*inputs, strict=True):
        if all(t.is_sparse for t in tensor_at_gpus):
```
- **EN**: This block continues `reduce_add_coalesced` and works to track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `reduce_add_coalesced`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 149-160
```python
            result = reduce_add(tensor_at_gpus, destination)  # this will be sparse too
            output.append(result)
            ref_order.append(tensor_at_gpus[0])
        else:
            for coll, t in zip(dense_tensors, tensor_at_gpus, strict=True):
                coll.append(t.to_dense() if t.is_sparse else t)
            ref_order.append(dense_tensors[0][-1])
    itrs = [_take_tensors(tensors, buffer_size) for tensors in dense_tensors]
    # now the dense ones, which have consistent sizes
    for chunks in zip(*itrs, strict=True):
        flat_tensors = [
            _flatten_dense_tensors(chunk) for chunk in chunks
```
- **EN**: This block continues `reduce_add_coalesced` and works to organize reusable module behavior and state. It also iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `reduce_add_coalesced`，用于组织可复用的模块行为与状态。 同时它还会遍历输入、节点或簿记结构。

### Lines 161-173
```python
        ]  # (num_gpus,)
        flat_result = reduce_add(flat_tensors, destination)
        for t in _unflatten_dense_tensors(flat_result, chunks[0]):
            # The unflattened tensors do not share storage, and we don't expose
            # base flat tensor anyways, so give them different version counters.
            # See NOTE [ Version Counter in comm.*_coalesced ]
            output.append(t.data)
    return tuple(_reorder_tensors_as(output, ref_order))


def scatter(tensor, devices=None, chunk_sizes=None, dim=0, streams=None, *, out=None):
    """Scatters tensor across multiple GPUs.
```
- **EN**: This module-level block helps normalize dtype/device related arguments and behavior. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于规范化 dtype/device 相关参数与行为。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构。

### Lines 174-191
```python
    Args:
        tensor (Tensor): tensor to scatter. Can be on CPU or GPU.
        devices (Iterable[torch.device, str or int], optional): an iterable of
          GPU devices, among which to scatter.
        chunk_sizes (Iterable[int], optional): sizes of chunks to be placed on
          each device. It should match :attr:`devices` in length and sums to
          ``tensor.size(dim)``. If not specified, :attr:`tensor` will be divided
          into equal chunks.
        dim (int, optional): A dimension along which to chunk :attr:`tensor`.
          Default: ``0``.
        streams (Iterable[torch.cuda.Stream], optional): an iterable of Streams, among
          which to execute the scatter. If not specified, the default stream will
          be utilized.
        out (Sequence[Tensor], optional, keyword-only): the GPU tensors to
          store output results. Sizes of these tensors must match that of
          :attr:`tensor`, except for :attr:`dim`, where the total size must
          sum to ``tensor.size(dim)``.
```
- **EN**: This block continues `scatter` and works to normalize dtype/device related arguments and behavior.
- **CN**: 该代码块继续实现 `scatter`，用于规范化 dtype/device 相关参数与行为。

### Lines 192-203
```python
    .. note::
        Exactly one of :attr:`devices` and :attr:`out` must be specified. When
        :attr:`out` is specified, :attr:`chunk_sizes` must not be specified and
        will be inferred from sizes of :attr:`out`.

    Returns:
        - If :attr:`devices` is specified,
            a tuple containing chunks of :attr:`tensor`, placed on
            :attr:`devices`.
        - If :attr:`out` is specified,
            a tuple containing :attr:`out` tensors, each containing a chunk of
            :attr:`tensor`.
```
- **EN**: This block continues `scatter` and works to normalize dtype/device related arguments and behavior.
- **CN**: 该代码块继续实现 `scatter`，用于规范化 dtype/device 相关参数与行为。

### Lines 204-221
```python
    """
    tensor = _handle_complex(tensor)
    if out is None:
        # pyrefly: ignore [not-iterable]
        devices = [_get_device_index(d) for d in devices]
        return tuple(torch._C._scatter(tensor, devices, chunk_sizes, dim, streams))
    else:
        if devices is not None:
            raise RuntimeError(
                f"'devices' must not be specified when 'out' is specified, but got devices={devices}"
            )
        if chunk_sizes is not None:
            raise RuntimeError(
                f"'chunk_sizes' must not be specified when 'out' is specified, but got chunk_sizes={chunk_sizes}"
            )
        return tuple(torch._C._scatter_out(tensor, out, dim, streams))
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 222-239
```python
def gather(tensors, dim=0, destination=None, *, out=None):
    r"""Gathers tensors from multiple GPU devices.

    Args:
        tensors (Iterable[Tensor]): an iterable of tensors to gather.
          Tensor sizes in all dimensions other than :attr:`dim` have to match.
        dim (int, optional): a dimension along which the tensors will be
          concatenated. Default: ``0``.
        destination (torch.device, str, or int, optional): the output device.
          Can be CPU or CUDA. Default: the current CUDA device.
        out (Tensor, optional, keyword-only): the tensor to store gather result.
          Its sizes must match those of :attr:`tensors`, except for :attr:`dim`,
          where the size must equal ``sum(tensor.size(dim) for tensor in tensors)``.
          Can be on CPU or CUDA.

    .. note::
        :attr:`destination` must not be specified when :attr:`out` is specified.
```
- **EN**: Defines the `gather` function; this block introduces logic that normalize dtype/device related arguments and behavior.
- **CN**: 定义`gather` 函数；该代码块引入了用于规范化 dtype/device 相关参数与行为的逻辑。

### Lines 240-251
```python
    Returns:
        - If :attr:`destination` is specified,
            a tensor located on :attr:`destination` device, that is a result of
            concatenating :attr:`tensors` along :attr:`dim`.
        - If :attr:`out` is specified,
            the :attr:`out` tensor, now containing results of concatenating
            :attr:`tensors` along :attr:`dim`.
    """
    tensors = [_handle_complex(t) for t in tensors]
    if out is None:
        if destination == -1:
            warnings.warn(
```
- **EN**: This block continues `gather` and works to normalize dtype/device related arguments and behavior. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `gather`，用于规范化 dtype/device 相关参数与行为。 同时它还会根据运行时条件分支处理。

### Lines 252-263
```python
                "Using -1 to represent CPU tensor is deprecated. Please use a "
                'device object or string instead, e.g., "cpu".',
                FutureWarning,
                stacklevel=2,
            )
        destination = _get_device_index(destination, allow_cpu=True, optional=True)
        return torch._C._gather(tensors, dim, destination)
    else:
        if destination is not None:
            raise RuntimeError(
                f"'destination' must not be specified when 'out' is specified, but got destination={destination}"
            )
```
- **EN**: This block continues `gather` and works to normalize dtype/device related arguments and behavior. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `gather`，用于规范化 dtype/device 相关参数与行为。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 264-264
```python
        return torch._C._gather_out(tensors, out, dim)
```
- **EN**: This block continues `gather` and works to prepare neural-network operators or module behavior. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `gather`，用于准备神经网络算子或模块行为。 同时它还会计算并返回中间值或结果。

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

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch._utils`, `torch.cuda`
- **Standard library / 标准库**: `warnings`
- **Primary symbols / 核心符号**: `broadcast`, `broadcast_coalesced`, `reduce_add`, `reduce_add_coalesced`, `scatter`, `gather`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。
