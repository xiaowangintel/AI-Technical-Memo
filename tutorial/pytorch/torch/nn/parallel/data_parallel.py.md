# data_parallel.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/parallel/data_parallel.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Provides wrappers and helpers for running neural-network modules across devices or processes.
- **Purpose (CN)**: 提供跨设备或进程运行神经网络模块的包装器与辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```python
# mypy: allow-untyped-defs
import operator
import warnings
from collections.abc import Sequence
from itertools import chain
from typing import Any, Generic, TypeVar

import torch
from torch._utils import (
    _get_all_device_indices,
    _get_available_device_type,
    _get_device_index,
```
- **EN**: This module-level block helps normalize dtype/device related arguments and behavior.
- **CN**: 这个模块级代码块用于规范化 dtype/device 相关参数与行为。

### Lines 13-24
```python
    _get_devices_properties,
)
from torch.nn.modules import Module
from torch.nn.parallel.parallel_apply import parallel_apply
from torch.nn.parallel.replicate import replicate
from torch.nn.parallel.scatter_gather import gather, scatter_kwargs


__all__ = ["DataParallel", "data_parallel"]


def _check_balance(device_ids: Sequence[int | torch.device]) -> None:
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 25-36
```python
    imbalance_warn = """
    There is an imbalance between your GPUs. You may want to exclude GPU {} which
    has less than 75% of the memory or cores of GPU {}. You can do so by setting
    the device_ids argument to DataParallel, or by setting the CUDA_VISIBLE_DEVICES
    environment variable."""
    device_ids = [_get_device_index(x, True) for x in device_ids]
    dev_props = _get_devices_properties(device_ids)

    def warn_imbalance(get_prop) -> bool:
        values = [get_prop(props) for props in dev_props]
        min_pos, min_val = min(enumerate(values), key=operator.itemgetter(1))
        max_pos, max_val = max(enumerate(values), key=operator.itemgetter(1))
```
- **EN**: Defines the `_check_balance` function; this block introduces logic that normalize dtype/device related arguments and behavior.
- **CN**: 定义`_check_balance` 函数；该代码块引入了用于规范化 dtype/device 相关参数与行为的逻辑。

### Lines 37-53
```python
        if min_val / max_val < 0.75:
            warnings.warn(
                imbalance_warn.format(device_ids[min_pos], device_ids[max_pos]),
                stacklevel=2,
            )
            return True
        return False

    if warn_imbalance(lambda props: props.total_memory):
        return
    if warn_imbalance(lambda props: props.multi_processor_count):
        return


T = TypeVar("T", bound=Module)
```
- **EN**: This range initializes module-level constants or registries that later code reuses to organize reusable module behavior and state.
- **CN**: 这一段初始化模块级常量或注册表，供后续代码重复使用以组织可复用的模块行为与状态。

### Lines 54-70
```python
class DataParallel(Module, Generic[T]):
    r"""Implements data parallelism at the module level.

    This container parallelizes the application of the given :attr:`module` by
    splitting the input across the specified devices by chunking in the batch
    dimension (other objects will be copied once per device). In the forward
    pass, the module is replicated on each device, and each replica handles a
    portion of the input. During the backwards pass, gradients from each replica
    are summed into the original module.

    The batch size should be larger than the number of GPUs used.

    .. warning::
        It is recommended to use :class:`~torch.nn.parallel.DistributedDataParallel`,
        instead of this class, to do multi-GPU training, even if there is only a single
        node. See: :ref:`cuda-nn-ddp-instead` and :ref:`ddp`.
```
- **EN**: Declares `DataParallel(Module, Generic[T])`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `DataParallel(Module, Generic[T])`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 71-82
```python
    Arbitrary positional and keyword inputs are allowed to be passed into
    DataParallel but some types are specially handled. tensors will be
    **scattered** on dim specified (default 0). tuple, list and dict types will
    be shallow copied. The other types will be shared among different threads
    and can be corrupted if written to in the model's forward pass.

    The parallelized :attr:`module` must have its parameters and buffers on
    ``device_ids[0]`` before running this :class:`~torch.nn.DataParallel`
    module.

    .. warning::
        In each forward, :attr:`module` is **replicated** on each device, so any
```
- **EN**: This block continues `DataParallel` and works to organize reusable module behavior and state.
- **CN**: 该代码块继续实现 `DataParallel`，用于组织可复用的模块行为与状态。

### Lines 83-94
```python
        updates to the running module in ``forward`` will be lost. For example,
        if :attr:`module` has a counter attribute that is incremented in each
        ``forward``, it will always stay at the initial value because the update
        is done on the replicas which are destroyed after ``forward``. However,
        :class:`~torch.nn.DataParallel` guarantees that the replica on
        ``device[0]`` will have its parameters and buffers sharing storage with
        the base parallelized :attr:`module`. So **in-place** updates to the
        parameters or buffers on ``device[0]`` will be recorded. E.g.,
        :class:`~torch.nn.BatchNorm2d` and :func:`~torch.nn.utils.spectral_norm`
        rely on this behavior to update the buffers.

    .. warning::
```
- **EN**: This block continues `DataParallel` and works to organize reusable module behavior and state. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `DataParallel`，用于组织可复用的模块行为与状态。 同时它还会根据运行时条件分支处理。

### Lines 95-110
```python
        Forward and backward hooks defined on :attr:`module` and its submodules
        will be invoked ``len(device_ids)`` times, each with inputs located on
        a particular device. Particularly, the hooks are only guaranteed to be
        executed in correct order with respect to operations on corresponding
        devices. For example, it is not guaranteed that hooks set via
        :meth:`~torch.nn.Module.register_forward_pre_hook` be executed before
        `all` ``len(device_ids)`` :meth:`~torch.nn.Module.forward` calls, but
        that each such hook be executed before the corresponding
        :meth:`~torch.nn.Module.forward` call of that device.

    .. warning::
        When :attr:`module` returns a scalar (i.e., 0-dimensional tensor) in
        :func:`forward`, this wrapper will return a vector of length equal to
        number of devices used in data parallelism, containing the result from
        each device.
```
- **EN**: This block continues `DataParallel` and works to organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `DataParallel`，用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

### Lines 111-128
```python
    .. note::
        There is a subtlety in using the
        ``pack sequence -> recurrent network -> unpack sequence`` pattern in a
        :class:`~torch.nn.Module` wrapped in :class:`~torch.nn.DataParallel`.
        See :ref:`pack-rnn-unpack-with-data-parallelism` section in FAQ for
        details.


    Args:
        module (Module): module to be parallelized
        device_ids (list of int or torch.device): CUDA devices (default: all devices)
        output_device (int or torch.device): device location of output (default: device_ids[0])

    Attributes:
        module (Module): the module to be parallelized

    Example::
```
- **EN**: This block continues `DataParallel` and works to organize reusable module behavior and state.
- **CN**: 该代码块继续实现 `DataParallel`，用于组织可复用的模块行为与状态。

### Lines 129-140
```python
        >>> # xdoctest: +SKIP
        >>> net = torch.nn.DataParallel(model, device_ids=[0, 1, 2])
        >>> output = net(input_var)  # input_var can be on any device, including CPU
    """

    # TODO: update notes/cuda.rst when this class handles 8+ GPUs well

    def __init__(
        self,
        module: T,
        device_ids: Sequence[int | torch.device] | None = None,
        output_device: int | torch.device | None = None,
```
- **EN**: Declares `DataParallel(Module, Generic[T])`; this class packages state and methods that organize reusable module behavior and state.
- **CN**: 声明 `DataParallel(Module, Generic[T])`；该类封装了状态与方法，用于组织可复用的模块行为与状态。

### Lines 141-156
```python
        dim: int = 0,
    ) -> None:
        super().__init__()
        torch._C._log_api_usage_once("torch.nn.parallel.DataParallel")
        device_type = _get_available_device_type()
        if device_type is None or device_type == "mps":
            self.module = module
            self.device_ids = []
            return

        if device_ids is None:
            device_ids = _get_all_device_indices()

        if device_ids is None:
            raise RuntimeError("no available devices were found")
```
- **EN**: This block continues `DataParallel.__init__` and works to organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `DataParallel.__init__`，用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 157-171
```python
        if output_device is None:
            output_device = device_ids[0]

        self.dim = dim
        self.module = module
        self.device_ids = [_get_device_index(x, True) for x in device_ids]
        self.output_device = _get_device_index(output_device, True)
        self.src_device_obj = torch.device(device_type, self.device_ids[0])

        if device_type == "cuda":
            _check_balance(self.device_ids)

        if len(self.device_ids) == 1:
            self.module.to(self.src_device_obj)
```
- **EN**: This block continues `DataParallel` and works to organize reusable module behavior and state. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `DataParallel`，用于组织可复用的模块行为与状态。 同时它还会根据运行时条件分支处理。

### Lines 172-185
```python
    def forward(self, *inputs: Any, **kwargs: Any) -> Any:
        with torch.autograd.profiler.record_function("DataParallel.forward"):
            if not self.device_ids:
                return self.module(*inputs, **kwargs)

            # pyrefly: ignore [bad-argument-type]
            for t in chain(self.module.parameters(), self.module.buffers()):
                if t.device != self.src_device_obj:
                    raise RuntimeError(
                        "module must have its parameters and buffers "
                        f"on device {self.src_device_obj} (device_ids[0]) but found one of "
                        f"them on device: {t.device}"
                    )
```
- **EN**: Defines the `DataParallel.forward` method; this block introduces logic that organize reusable module behavior and state.
- **CN**: 定义`DataParallel.forward` 方法；该代码块引入了用于组织可复用的模块行为与状态的逻辑。

### Lines 186-201
```python
            inputs, module_kwargs = self.scatter(inputs, kwargs, self.device_ids)
            # for forward function without any inputs, empty list and dict will be created
            # so the module can be executed on one device which is the first one in device_ids
            if not inputs and not module_kwargs:
                inputs = ((),)
                module_kwargs = ({},)

            if len(self.device_ids) == 1:
                return self.module(*inputs[0], **module_kwargs[0])
            replicas = self.replicate(self.module, self.device_ids[: len(inputs)])
            outputs = self.parallel_apply(replicas, inputs, module_kwargs)
            return self.gather(outputs, self.output_device)

    def replicate(self, module: T, device_ids: Sequence[int | torch.device]) -> list[T]:
        return replicate(module, device_ids, not torch.is_grad_enabled())
```
- **EN**: Declares `DataParallel(Module, Generic[T])`; this class packages state and methods that organize reusable module behavior and state.
- **CN**: 声明 `DataParallel(Module, Generic[T])`；该类封装了状态与方法，用于组织可复用的模块行为与状态。

### Lines 202-219
```python
    def scatter(
        self,
        inputs: tuple[Any, ...],
        kwargs: dict[str, Any] | None,
        device_ids: Sequence[int | torch.device],
    ) -> Any:
        return scatter_kwargs(inputs, kwargs, device_ids, dim=self.dim)

    def parallel_apply(
        self, replicas: Sequence[T], inputs: Sequence[Any], kwargs: Any
    ) -> list[Any]:
        return parallel_apply(
            replicas, inputs, kwargs, self.device_ids[: len(replicas)]
        )

    def gather(self, outputs: Any, output_device: int | torch.device) -> Any:
        return gather(outputs, output_device, dim=self.dim)
```
- **EN**: This module-level block helps normalize dtype/device related arguments and behavior. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于规范化 dtype/device 相关参数与行为。 同时它还会计算并返回中间值或结果。

### Lines 220-232
```python

def data_parallel(
    module: Module,
    inputs: Any,
    device_ids: Sequence[int | torch.device] | None = None,
    output_device: int | torch.device | None = None,
    dim: int = 0,
    module_kwargs: Any | None = None,
) -> torch.Tensor:
    r"""Evaluate module(input) in parallel across the GPUs given in device_ids.

    This is the functional version of the DataParallel module.
```
- **EN**: This module-level block helps organize reusable module behavior and state.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。

### Lines 233-250
```python
    Args:
        module (Module): the module to evaluate in parallel
        inputs (Tensor): inputs to the module
        device_ids (list of int or torch.device): GPU ids on which to replicate module
        output_device (list of int or torch.device): GPU location of the output  Use -1 to indicate the CPU.
            (default: device_ids[0])
    Returns:
        a Tensor containing the result of module(input) located on
        output_device
    """
    if not isinstance(inputs, tuple):
        inputs = (inputs,) if inputs is not None else ()

    device_type = _get_available_device_type()

    if device_type is None:
        raise RuntimeError("device type could not be determined")
```
- **EN**: This block continues `data_parallel` and works to organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `data_parallel`，用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 251-264
```python
    if device_ids is None:
        device_ids = _get_all_device_indices()

    if device_ids is None:
        raise RuntimeError("no available devices were found")

    if output_device is None:
        output_device = device_ids[0]

    device_ids = [_get_device_index(x, True) for x in device_ids]
    output_device = _get_device_index(output_device, True)
    # pyrefly: ignore [bad-argument-type, no-matching-overload]
    src_device_obj = torch.device(device_type, device_ids[0])
```
- **EN**: This block continues `data_parallel` and works to normalize dtype/device related arguments and behavior. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `data_parallel`，用于规范化 dtype/device 相关参数与行为。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 265-280
```python
    # pyrefly: ignore [bad-argument-type]
    for t in chain(module.parameters(), module.buffers()):
        if t.device != src_device_obj:
            raise RuntimeError(
                "module must have its parameters and buffers "
                f"on device {src_device_obj} (device_ids[0]) but found one of "
                f"them on device: {t.device}"
            )

    inputs, module_kwargs = scatter_kwargs(inputs, module_kwargs, device_ids, dim)
    # for module without any inputs, empty list and dict will be created
    # so the module can be executed on one device which is the first one in device_ids
    if not inputs and not module_kwargs:
        inputs = ((),)
        module_kwargs = ({},)
```
- **EN**: This block continues `data_parallel` and works to organize reusable module behavior and state. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `data_parallel`，用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 281-289
```python
    if module_kwargs is None:
        raise AssertionError("module_kwargs should not be None after scatter_kwargs")

    if len(device_ids) == 1:
        return module(*inputs[0], **module_kwargs[0])
    used_device_ids = device_ids[: len(inputs)]
    replicas = replicate(module, used_device_ids)
    outputs = parallel_apply(replicas, inputs, module_kwargs, used_device_ids)
    return gather(outputs, output_device, dim)
```
- **EN**: This block continues `data_parallel` and works to organize reusable module behavior and state. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `data_parallel`，用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

## Key Concepts / 关键概念

- **Neural-network abstractions / 神经网络抽象**
  - EN: The file contributes to reusable neural-network layers, functionals, or support utilities.
  - CN: 该文件为可复用的神经网络层、函数式接口或辅助工具提供实现。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
- **Program transforms / 程序变换**
  - EN: The implementation rewrites existing program structure into a more useful form.
  - CN: 该实现会把现有程序结构改写成更有用的形式。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch._utils`, `torch.nn.modules`, `torch.nn.parallel.parallel_apply`, `torch.nn.parallel.replicate`, `torch.nn.parallel.scatter_gather`
- **Standard library / 标准库**: `operator`, `warnings`, `collections.abc`, `itertools`, `typing`
- **Primary symbols / 核心符号**: `__all__`, `_check_balance`, `T`, `DataParallel`, `data_parallel`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。
