# parallel_apply.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/parallel/parallel_apply.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Provides wrappers and helpers for running neural-network modules across devices or processes.
- **Purpose (CN)**: 提供跨设备或进程运行神经网络模块的包装器与辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```python
import threading
from collections.abc import Sequence
from typing import Any, cast

import torch
from torch._utils import ExceptionWrapper
from torch.cuda._utils import _get_device_index
from torch.nn.modules import Module


__all__ = ["get_a_var", "parallel_apply"]
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 14-30
```python
def get_a_var(
    obj: torch.Tensor | list[Any] | tuple[Any, ...] | dict[Any, Any],
) -> torch.Tensor | None:
    if isinstance(obj, torch.Tensor):
        return obj

    if isinstance(obj, (list, tuple)):
        for result in map(get_a_var, obj):
            if isinstance(result, torch.Tensor):
                return result
    if isinstance(obj, dict):
        for result in map(get_a_var, obj.items()):
            if isinstance(result, torch.Tensor):
                return result
    return None
```
- **EN**: This module-level block helps prepare neural-network operators or module behavior. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于准备神经网络算子或模块行为。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 31-43
```python
def parallel_apply(
    modules: Sequence[Module],
    inputs: Sequence[Any],
    kwargs_tup: Sequence[dict[str, Any]] | None = None,
    devices: Sequence[int | torch.device | None] | None = None,
) -> list[Any]:
    r"""Apply each `module` in :attr:`modules` in parallel on each of :attr:`devices`.

    Args:
        modules (Module): modules to be parallelized
        inputs (tensor): inputs to the modules
        devices (list of int or torch.device): CUDA devices
```
- **EN**: Defines the `parallel_apply` function; this block introduces logic that organize reusable module behavior and state.
- **CN**: 定义`parallel_apply` 函数；该代码块引入了用于组织可复用的模块行为与状态的逻辑。

### Lines 44-55
```python
    :attr:`modules`, :attr:`inputs`, :attr:`kwargs_tup` (if given), and
    :attr:`devices` (if given) should all have same length. Moreover, each
    element of :attr:`inputs` can either be a single object as the only argument
    to a module, or a collection of positional arguments.
    """
    if len(modules) != len(inputs):
        raise AssertionError(
            f"The number of modules {len(modules)} is not equal to "
            f"the number of inputs {len(inputs)}"
        )
    if kwargs_tup is not None:
        if len(modules) != len(kwargs_tup):
```
- **EN**: This block continues `parallel_apply` and works to organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `parallel_apply`，用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 56-67
```python
            raise AssertionError(
                f"The number of modules {len(modules)} is not equal to "
                f"the number of kwargs_tup {len(kwargs_tup)}"
            )
    else:
        kwargs_tup = (cast(dict[str, Any], {}),) * len(modules)
    if devices is not None:
        if len(modules) != len(devices):
            raise AssertionError(
                f"The number of modules {len(modules)} is not equal to "
                f"the number of devices {len(devices)}"
            )
```
- **EN**: This block continues `parallel_apply` and works to organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `parallel_apply`，用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 68-81
```python
    else:
        devices = [None] * len(modules)
    devices = [_get_device_index(x, True) for x in devices]
    streams = [torch.accelerator.current_stream(x) for x in devices]
    if not torch.accelerator.is_available():
        raise AssertionError("No available accelerator found.")
    device_type = torch.accelerator.current_accelerator().type  # type: ignore[union-attr]
    lock = threading.Lock()
    results = {}
    grad_enabled, autocast_enabled = (
        torch.is_grad_enabled(),
        torch.is_autocast_enabled(),
    )
```
- **EN**: This block continues `parallel_apply` and works to organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `parallel_apply`，用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 82-93
```python
    def _worker(
        i: int,
        module: Module,
        input: Any,
        kwargs: dict[str, Any],
        device: int | torch.device | None = None,
        stream: torch.Stream | None = None,
    ) -> None:
        torch.set_grad_enabled(grad_enabled)
        if device is None:
            t = get_a_var(input)
            if t is None:
```
- **EN**: Defines the `parallel_apply._worker` function; this block introduces logic that organize reusable module behavior and state.
- **CN**: 定义`parallel_apply._worker` 函数；该代码块引入了用于组织可复用的模块行为与状态的逻辑。

### Lines 94-105
```python
                with lock:
                    results[i] = ExceptionWrapper(
                        where=f"in replica {i}, no device was provided and no tensor input was found; "
                        "device cannot be resolved"
                    )
                return
            device = t.get_device()
        if isinstance(device, torch.device):
            device = device.index
        if stream is None:
            stream = torch.accelerator.current_stream(device)
        try:
```
- **EN**: This block continues `parallel_apply._worker` and works to normalize dtype/device related arguments and behavior. It also branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `parallel_apply._worker`，用于规范化 dtype/device 相关参数与行为。 同时它还会根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 106-122
```python
            with (
                torch.accelerator.device_index(device),
                stream,
                torch.amp.autocast(device_type, enabled=autocast_enabled),
            ):
                # this also avoids accidental slicing of `input` if it is a Tensor
                if not isinstance(input, (list, tuple)):
                    input = (input,)
                output = module(*input, **kwargs)
            with lock:
                results[i] = output
        except Exception:
            with lock:
                results[i] = ExceptionWrapper(
                    where=f"in replica {i} on device {device}"
                )
```
- **EN**: This block continues `parallel_apply` and works to organize reusable module behavior and state. It also branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `parallel_apply`，用于组织可复用的模块行为与状态。 同时它还会根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 123-139
```python
    if len(modules) > 1:
        threads = [
            threading.Thread(
                target=_worker, args=(i, module, input, kwargs, device, stream)
            )
            for i, (module, input, kwargs, device, stream) in enumerate(
                zip(modules, inputs, kwargs_tup, devices, streams, strict=True)
            )
        ]

        for thread in threads:
            thread.start()
        for thread in threads:
            thread.join()
    else:
        _worker(0, modules[0], inputs[0], kwargs_tup[0], devices[0], streams[0])
```
- **EN**: This block continues `parallel_apply` and works to organize reusable module behavior and state. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `parallel_apply`，用于组织可复用的模块行为与状态。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 140-146
```python
    outputs = []
    for i in range(len(inputs)):
        output = results[i]
        if isinstance(output, ExceptionWrapper):
            output.reraise()
        outputs.append(output)
    return outputs
```
- **EN**: This block continues `parallel_apply` and works to validate invariants and surface meaningful failures. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `parallel_apply`，用于校验不变量并给出有意义的失败信息。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

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

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch._utils`, `torch.cuda._utils`, `torch.nn.modules`
- **Standard library / 标准库**: `threading`, `collections.abc`, `typing`
- **Primary symbols / 核心符号**: `__all__`, `get_a_var`, `parallel_apply`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。
