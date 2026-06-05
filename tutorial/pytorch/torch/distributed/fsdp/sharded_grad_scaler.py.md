# sharded_grad_scaler.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/fsdp/sharded_grad_scaler.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on Fully Sharded Data Parallel runtime and utilities. Its main entry points include _GeneralMultiDeviceReplicator, ShardedGradScaler, _refresh_per_optimizer_state, _is_supported_device.
- **用途 (CN)**: 该模块聚焦于Fully Sharded Data Parallel（FSDP）运行时与工具，其主要入口包括 _GeneralMultiDeviceReplicator, ShardedGradScaler, _refresh_per_optimizer_state, _is_supported_device。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# mypy: allow-untyped-defs
import logging
from collections import abc, defaultdict
from collections.abc import Iterable
from typing import Any, overload

import torch
import torch.distributed as dist
from torch.amp.grad_scaler import _MultiDeviceReplicator, GradScaler, OptState
from torch.distributed.distributed_c10d import ProcessGroup


logger = logging.getLogger(__name__)


def _refresh_per_optimizer_state() -> dict[str, Any]:
    return {"stage": OptState.READY, "found_inf_per_device": {}}


def _is_supported_device(tensor: torch.Tensor) -> bool:
````

- **L1** EN: Keeps the inline comment or directive: mypy: allow-untyped-defs | CN: 保留这一行注释或指令：mypy: allow-untyped-defs
- **L2** EN: Imports module dependencies: `logging`. | CN: 导入模块依赖：`logging`。
- **L3** EN: Imports selected names from `collections`. | CN: 从 `collections` 导入指定名称。
- **L4** EN: Imports selected names from `collections.abc`. | CN: 从 `collections.abc` 导入指定名称。
- **L5** EN: Imports selected names from `typing`. | CN: 从 `typing` 导入指定名称。
- **L6** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L7** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L8** EN: Imports module dependencies: `torch.distributed as dist`. | CN: 导入模块依赖：`torch.distributed as dist`。
- **L9** EN: Imports selected names from `torch.amp.grad_scaler`. | CN: 从 `torch.amp.grad_scaler` 导入指定名称。
- **L10** EN: Imports selected names from `torch.distributed.distributed_c10d`. | CN: 从 `torch.distributed.distributed_c10d` 导入指定名称。
- **L11** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L12** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L13** EN: Assigns or updates `logger`. | CN: 对 `logger` 进行赋值或更新。
- **L14** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L15** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L16** EN: Defines function `_refresh_per_optimizer_state`. | CN: 定义函数 `_refresh_per_optimizer_state`。
- **L17** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L18** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L19** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L20** EN: Defines function `_is_supported_device`. | CN: 定义函数 `_is_supported_device`。

### Lines 21-40 / 第 21-40 行

````python
    return tensor.is_cuda or tensor.device.type in (
        "xla",
        "cpu",
        "hpu",
        "mtia",
        "xpu",
        torch._C._get_privateuse1_backend_name(),
    )


class _GeneralMultiDeviceReplicator(_MultiDeviceReplicator):
    """
    Lazily serves tensor to request device. This class extends
    _MultiDeviceReplicator to allow support for "cpu" as a device.
    """

    def __init__(self, master_tensor: torch.Tensor) -> None:
        if not _is_supported_device(master_tensor):
            raise AssertionError(
                f"Expected supported device, got {master_tensor.device}"
````

- **L21** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L22** EN: Continues the implementation inside function `_is_supported_device`. | CN: 继续说明函数 `_is_supported_device` 内部的实现。
- **L23** EN: Continues the implementation inside function `_is_supported_device`. | CN: 继续说明函数 `_is_supported_device` 内部的实现。
- **L24** EN: Continues the implementation inside function `_is_supported_device`. | CN: 继续说明函数 `_is_supported_device` 内部的实现。
- **L25** EN: Continues the implementation inside function `_is_supported_device`. | CN: 继续说明函数 `_is_supported_device` 内部的实现。
- **L26** EN: Continues the implementation inside function `_is_supported_device`. | CN: 继续说明函数 `_is_supported_device` 内部的实现。
- **L27** EN: Calls `torch._C._get_privateuse1_backend_name` as part of the current workflow. | CN: 在当前流程中调用 `torch._C._get_privateuse1_backend_name`。
- **L28** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L29** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L30** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L31** EN: Defines class `_GeneralMultiDeviceReplicator`. | CN: 定义类 `_GeneralMultiDeviceReplicator`。
- **L32** EN: Starts the docstring for the class _GeneralMultiDeviceReplicator. | CN: 开始定义 class _GeneralMultiDeviceReplicator 的文档字符串。
- **L33** EN: Continues the docstring text for the class _GeneralMultiDeviceReplicator. | CN: 继续补充 class _GeneralMultiDeviceReplicator 的文档字符串内容。
- **L34** EN: Continues the docstring text for the class _GeneralMultiDeviceReplicator. | CN: 继续补充 class _GeneralMultiDeviceReplicator 的文档字符串内容。
- **L35** EN: Closes the docstring for the class _GeneralMultiDeviceReplicator. | CN: 结束 class _GeneralMultiDeviceReplicator 的文档字符串。
- **L36** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L37** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L38** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L39** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L40** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。

### Lines 41-60 / 第 41-60 行

````python
            )
        self.master = master_tensor
        self._per_device_tensors: dict[torch.device, torch.Tensor] = {}


class ShardedGradScaler(GradScaler):
    """
    ShardedGradScaler helps perform gradient scaling in a shard aware manner. It extends
    functionality from GradScaler:
    * Supports Pytorch DDP and FSDP implementations
    * Support CPU offloaded tensors (as used in fully sharded data parallel[FSDP])
    * Supports the custom Mixed Precision loss dtype (fp16, bf16) that FSDP returns
    * Sync inf/nan for scaled gradient tensors on any torch.device (where tensors are placed) across
    nodes

    Example::

        # Creates a ShardedGradScaler once at the beginning of training.
        scaler = ShardedGradScaler()

````

- **L41** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L42** EN: Assigns or updates `self.master`. | CN: 对 `self.master` 进行赋值或更新。
- **L43** EN: Assigns or updates `self._per_device_tensors`. | CN: 对 `self._per_device_tensors` 进行赋值或更新。
- **L44** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L45** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L46** EN: Defines class `ShardedGradScaler`. | CN: 定义类 `ShardedGradScaler`。
- **L47** EN: Starts the docstring for the class ShardedGradScaler. | CN: 开始定义 class ShardedGradScaler 的文档字符串。
- **L48** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L49** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L50** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L51** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L52** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L53** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L54** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L55** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L56** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L57** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L58** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L59** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L60** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。

### Lines 61-80 / 第 61-80 行

````python
        for epoch in epochs:
            for input, target in data:
                optimizer.zero_grad()
                output = model(input)
                loss = loss_fn(output, target)

                # Scales loss.  Calls backward() on scaled loss to create scaled gradients.
                scaler.scale(loss).backward()

                # scaler.step() first unscales gradients of the optimizer's params.
                # If gradients don't contain infs/NaNs, optimizer.step() is then called,
                # otherwise, optimizer.step() is skipped.
                scaler.step(optimizer)

                # Updates the scale for next iteration.
                scaler.update()

    See :class:`GradScaler` for explanation of scaling/unscaling and more use cases.

    Args:
````

- **L61** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L62** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L63** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L64** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L65** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L66** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L67** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L68** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L69** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L70** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L71** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L72** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L73** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L74** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L75** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L76** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L77** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L78** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L79** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L80** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。

### Lines 81-100 / 第 81-100 行

````python
        init_scale (float, optional, default=2.**16):  Initial scale factor.
        growth_factor (float, optional, default=2.0):  Factor by which the scale is multiplied during
            :meth:`update` if no inf/NaN gradients occur for ``growth_interval`` consecutive iterations.
        backoff_factor (float, optional, default=0.5):  Factor by which the scale is multiplied during
            :meth:`update` if inf/NaN gradients occur in an iteration.
        growth_interval (int, optional, default=2000):  Number of consecutive iterations without inf/NaN gradients
            that must occur for the scale to be multiplied by ``growth_factor``.
        enabled (bool, optional):  If ``False``, disables gradient scaling. :meth:`step` simply
            invokes the underlying ``optimizer.step()``, and other methods become no-ops.
            Default: ``True``
        process_group (ProcessGroup, optional, default=torch.distributed.group.WORLD):
            process group for sharding
    """

    def __init__(
        self,
        device: str = "cuda",
        init_scale: float = 2.0**16,
        backoff_factor: float = 0.5,
        growth_factor: float = 2.0,
````

- **L81** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L82** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L83** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L84** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L85** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L86** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L87** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L88** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L89** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L90** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L91** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L92** EN: Continues the docstring text for the class ShardedGradScaler. | CN: 继续补充 class ShardedGradScaler 的文档字符串内容。
- **L93** EN: Closes the docstring for the class ShardedGradScaler. | CN: 结束 class ShardedGradScaler 的文档字符串。
- **L94** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L95** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L96** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L97** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L98** EN: Assigns or updates `init_scale`. | CN: 对 `init_scale` 进行赋值或更新。
- **L99** EN: Assigns or updates `backoff_factor`. | CN: 对 `backoff_factor` 进行赋值或更新。
- **L100** EN: Assigns or updates `growth_factor`. | CN: 对 `growth_factor` 进行赋值或更新。

### Lines 101-120 / 第 101-120 行

````python
        growth_interval: int = 2000,
        enabled: bool = True,
        process_group: ProcessGroup | None = dist.group.WORLD,
    ) -> None:
        super().__init__(
            device,
            init_scale=init_scale,
            backoff_factor=backoff_factor,
            growth_factor=growth_factor,
            growth_interval=growth_interval,
            enabled=enabled,
        )
        if self._enabled:
            self.process_group = process_group
            self._per_optimizer_states = defaultdict(_refresh_per_optimizer_state)

    @overload
    def scale(self, outputs: torch.Tensor) -> torch.Tensor: ...

    @overload
````

- **L101** EN: Assigns or updates `growth_interval`. | CN: 对 `growth_interval` 进行赋值或更新。
- **L102** EN: Assigns or updates `enabled`. | CN: 对 `enabled` 进行赋值或更新。
- **L103** EN: Assigns or updates `process_group`. | CN: 对 `process_group` 进行赋值或更新。
- **L104** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L105** EN: Calls `super` as part of the current workflow. | CN: 在当前流程中调用 `super`。
- **L106** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L107** EN: Assigns or updates `init_scale`. | CN: 对 `init_scale` 进行赋值或更新。
- **L108** EN: Assigns or updates `backoff_factor`. | CN: 对 `backoff_factor` 进行赋值或更新。
- **L109** EN: Assigns or updates `growth_factor`. | CN: 对 `growth_factor` 进行赋值或更新。
- **L110** EN: Assigns or updates `growth_interval`. | CN: 对 `growth_interval` 进行赋值或更新。
- **L111** EN: Assigns or updates `enabled`. | CN: 对 `enabled` 进行赋值或更新。
- **L112** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L113** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L114** EN: Assigns or updates `self.process_group`. | CN: 对 `self.process_group` 进行赋值或更新。
- **L115** EN: Assigns or updates `self._per_optimizer_states`. | CN: 对 `self._per_optimizer_states` 进行赋值或更新。
- **L116** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L117** EN: Applies decorator `overload` to the following definition. | CN: 将装饰器 `overload` 应用于后续定义。
- **L118** EN: Defines function `scale`. | CN: 定义函数 `scale`。
- **L119** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L120** EN: Applies decorator `overload` to the following definition. | CN: 将装饰器 `overload` 应用于后续定义。

### Lines 121-140 / 第 121-140 行

````python
    def scale(self, outputs: list[torch.Tensor]) -> list[torch.Tensor]: ...

    @overload
    def scale(self, outputs: tuple[torch.Tensor, ...]) -> tuple[torch.Tensor, ...]: ...

    @overload
    def scale(self, outputs: Iterable[torch.Tensor]) -> Iterable[torch.Tensor]: ...

    def scale(
        self, outputs: torch.Tensor | Iterable[torch.Tensor]
    ) -> torch.Tensor | Iterable[torch.Tensor]:
        if not self._enabled:
            return outputs

        if isinstance(outputs, torch.Tensor):
            if not _is_supported_device(outputs):
                raise AssertionError(f"Expected supported device, got {outputs.device}")
            if self._scale is None:
                self._lazy_init_scale_growth_tracker(outputs.device)
            if self._scale is None:
````

- **L121** EN: Defines function `scale`. | CN: 定义函数 `scale`。
- **L122** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L123** EN: Applies decorator `overload` to the following definition. | CN: 将装饰器 `overload` 应用于后续定义。
- **L124** EN: Defines function `scale`. | CN: 定义函数 `scale`。
- **L125** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L126** EN: Applies decorator `overload` to the following definition. | CN: 将装饰器 `overload` 应用于后续定义。
- **L127** EN: Defines function `scale`. | CN: 定义函数 `scale`。
- **L128** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L129** EN: Defines function `scale`. | CN: 定义函数 `scale`。
- **L130** EN: Continues the implementation inside function `scale`. | CN: 继续说明函数 `scale` 内部的实现。
- **L131** EN: Continues the implementation inside function `scale`. | CN: 继续说明函数 `scale` 内部的实现。
- **L132** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L133** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L134** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L135** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L136** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L137** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L138** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L139** EN: Calls `self._lazy_init_scale_growth_tracker` as part of the current workflow. | CN: 在当前流程中调用 `self._lazy_init_scale_growth_tracker`。
- **L140** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 141-160 / 第 141-160 行

````python
                raise AssertionError("Expected _scale to be initialized, got None")
            scaled_output = outputs * self._scale.to(
                device=outputs.device, non_blocking=True
            )
            # Here we ensure the return dtype is the same as the outputs dtype.
            # For the FSDP + Mixed Precision use case, the loss output is in the Mixed Precision
            # format (fp16, bf16) and so the scaled loss should be of the same dtype.
            return scaled_output.type(outputs.dtype)

        stash: list[_GeneralMultiDeviceReplicator] = []

        def apply_scale(val: torch.Tensor | Iterable[torch.Tensor]):
            if isinstance(val, torch.Tensor):
                if not _is_supported_device(val):
                    raise AssertionError(f"Expected supported device, got {val.device}")
                if len(stash) == 0:
                    if self._scale is None:
                        self._lazy_init_scale_growth_tracker(val.device)
                    if self._scale is None:
                        raise AssertionError(
````

- **L141** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L142** EN: Assigns or updates `scaled_output`. | CN: 对 `scaled_output` 进行赋值或更新。
- **L143** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L144** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L145** EN: Keeps the inline comment or directive: Here we ensure the return dtype is the same as the outputs dtype. | CN: 保留这一行注释或指令：Here we ensure the return dtype is the same as the outputs dtype.
- **L146** EN: Keeps the inline comment or directive: For the FSDP + Mixed Precision use case, the loss output is in the Mixed Precisi | CN: 保留这一行注释或指令：For the FSDP + Mixed Precision use case, the loss output is in the Mixed Precisi
- **L147** EN: Keeps the inline comment or directive: format (fp16, bf16) and so the scaled loss should be of the same dtype. | CN: 保留这一行注释或指令：format (fp16, bf16) and so the scaled loss should be of the same dtype.
- **L148** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L149** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L150** EN: Assigns or updates `stash`. | CN: 对 `stash` 进行赋值或更新。
- **L151** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L152** EN: Defines function `apply_scale`. | CN: 定义函数 `apply_scale`。
- **L153** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L154** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L155** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L156** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L157** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L158** EN: Calls `self._lazy_init_scale_growth_tracker` as part of the current workflow. | CN: 在当前流程中调用 `self._lazy_init_scale_growth_tracker`。
- **L159** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L160** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。

### Lines 161-180 / 第 161-180 行

````python
                            "Expected _scale to be initialized, got None"
                        )
                    stash.append(_GeneralMultiDeviceReplicator(self._scale))
                scaled_val = val * stash[0].get(val.device)
                # Here we ensure the return dtype is the same as the outputs dtype.
                # For the FSDP + Mixed Precision use case, the loss output is in the Mixed Precision
                # format (fp16, bf16) and so the scaled loss should be of the same dtype.
                return scaled_val.type(val.dtype)
            if isinstance(val, abc.Iterable):
                iterator = map(apply_scale, val)
                if isinstance(val, (list, tuple)):
                    return type(val)(iterator)
                return iterator
            raise ValueError("outputs must be a Tensor or an iterable of Tensors")

        return apply_scale(outputs)

    def _unscale_grads_(
        self,
        optimizer: torch.optim.Optimizer,
````

- **L161** EN: Continues the implementation inside function `apply_scale`. | CN: 继续说明函数 `apply_scale` 内部的实现。
- **L162** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L163** EN: Calls `stash.append` as part of the current workflow. | CN: 在当前流程中调用 `stash.append`。
- **L164** EN: Assigns or updates `scaled_val`. | CN: 对 `scaled_val` 进行赋值或更新。
- **L165** EN: Keeps the inline comment or directive: Here we ensure the return dtype is the same as the outputs dtype. | CN: 保留这一行注释或指令：Here we ensure the return dtype is the same as the outputs dtype.
- **L166** EN: Keeps the inline comment or directive: For the FSDP + Mixed Precision use case, the loss output is in the Mixed Precisi | CN: 保留这一行注释或指令：For the FSDP + Mixed Precision use case, the loss output is in the Mixed Precisi
- **L167** EN: Keeps the inline comment or directive: format (fp16, bf16) and so the scaled loss should be of the same dtype. | CN: 保留这一行注释或指令：format (fp16, bf16) and so the scaled loss should be of the same dtype.
- **L168** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L169** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L170** EN: Assigns or updates `iterator`. | CN: 对 `iterator` 进行赋值或更新。
- **L171** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L172** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L173** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L174** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L175** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L176** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L177** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L178** EN: Defines function `_unscale_grads_`. | CN: 定义函数 `_unscale_grads_`。
- **L179** EN: Continues the implementation inside function `_unscale_grads_`. | CN: 继续说明函数 `_unscale_grads_` 内部的实现。
- **L180** EN: Continues the implementation inside function `_unscale_grads_`. | CN: 继续说明函数 `_unscale_grads_` 内部的实现。

### Lines 181-200 / 第 181-200 行

````python
        inv_scale: torch.Tensor,
        found_inf: torch.Tensor,
        allow_fp16: bool = True,
    ) -> dict[torch.device, torch.Tensor]:
        per_device_inv_scale = _GeneralMultiDeviceReplicator(inv_scale)
        per_device_found_inf = _GeneralMultiDeviceReplicator(found_inf)

        # To set up _amp_foreach_non_finite_check_and_unscale_, split grads by device and dtype.
        # There could be thousands of grads, so we'd like to iterate through them just once.
        # However, we don't know their devices or dtypes in advance.

        # https://stackoverflow.com/questions/5029934/defaultdict-of-defaultdict
        # Google says mypy struggles with defaultdicts type annotations.
        per_device_and_dtype_grads = defaultdict(lambda: defaultdict(list))  # type: ignore[var-annotated]
        with torch.no_grad():
            for group in optimizer.param_groups:
                for param in group["params"]:
                    if param.grad is None:
                        continue
                    if (not allow_fp16) and param.grad.dtype == torch.float16:
````

- **L181** EN: Continues the implementation inside function `_unscale_grads_`. | CN: 继续说明函数 `_unscale_grads_` 内部的实现。
- **L182** EN: Continues the implementation inside function `_unscale_grads_`. | CN: 继续说明函数 `_unscale_grads_` 内部的实现。
- **L183** EN: Assigns or updates `allow_fp16`. | CN: 对 `allow_fp16` 进行赋值或更新。
- **L184** EN: Continues the implementation inside function `_unscale_grads_`. | CN: 继续说明函数 `_unscale_grads_` 内部的实现。
- **L185** EN: Assigns or updates `per_device_inv_scale`. | CN: 对 `per_device_inv_scale` 进行赋值或更新。
- **L186** EN: Assigns or updates `per_device_found_inf`. | CN: 对 `per_device_found_inf` 进行赋值或更新。
- **L187** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L188** EN: Keeps the inline comment or directive: To set up _amp_foreach_non_finite_check_and_unscale_, split grads by device and  | CN: 保留这一行注释或指令：To set up _amp_foreach_non_finite_check_and_unscale_, split grads by device and 
- **L189** EN: Keeps the inline comment or directive: There could be thousands of grads, so we'd like to iterate through them just onc | CN: 保留这一行注释或指令：There could be thousands of grads, so we'd like to iterate through them just onc
- **L190** EN: Keeps the inline comment or directive: However, we don't know their devices or dtypes in advance. | CN: 保留这一行注释或指令：However, we don't know their devices or dtypes in advance.
- **L191** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L192** EN: Keeps the inline comment or directive: https://stackoverflow.com/questions/5029934/defaultdict-of-defaultdict | CN: 保留这一行注释或指令：https://stackoverflow.com/questions/5029934/defaultdict-of-defaultdict
- **L193** EN: Keeps the inline comment or directive: Google says mypy struggles with defaultdicts type annotations. | CN: 保留这一行注释或指令：Google says mypy struggles with defaultdicts type annotations.
- **L194** EN: Assigns or updates `per_device_and_dtype_grads`. | CN: 对 `per_device_and_dtype_grads` 进行赋值或更新。
- **L195** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。
- **L196** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L197** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L198** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L199** EN: Skips to the next loop iteration. | CN: 跳到下一次循环迭代。
- **L200** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 201-220 / 第 201-220 行

````python
                        raise ValueError("Attempting to unscale FP16 gradients.")
                    if param.grad.is_sparse:
                        # is_coalesced() == False means the sparse grad has values with duplicate indices.
                        # coalesce() deduplicates indices and adds all values that have the same index.
                        # For scaled fp16 values, there's a good chance coalescing will cause overflow,
                        # so we should check the coalesced _values().
                        if param.grad.dtype is torch.float16:
                            # coalesce is not supported in torch.float16
                            param_grad_fp32 = param.grad.type(torch.float32).coalesce()
                            param.grad = param_grad_fp32.type(torch.float16)
                        to_unscale = param.grad._values()
                    else:
                        to_unscale = param.grad

                    per_device_and_dtype_grads[to_unscale.device][
                        to_unscale.dtype
                    ].append(to_unscale)

            for device, per_dtype_grads in per_device_and_dtype_grads.items():
                for grads in per_dtype_grads.values():
````

- **L201** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L202** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L203** EN: Keeps the inline comment or directive: is_coalesced() == False means the sparse grad has values with duplicate indices. | CN: 保留这一行注释或指令：is_coalesced() == False means the sparse grad has values with duplicate indices.
- **L204** EN: Keeps the inline comment or directive: coalesce() deduplicates indices and adds all values that have the same index. | CN: 保留这一行注释或指令：coalesce() deduplicates indices and adds all values that have the same index.
- **L205** EN: Keeps the inline comment or directive: For scaled fp16 values, there's a good chance coalescing will cause overflow, | CN: 保留这一行注释或指令：For scaled fp16 values, there's a good chance coalescing will cause overflow,
- **L206** EN: Keeps the inline comment or directive: so we should check the coalesced _values(). | CN: 保留这一行注释或指令：so we should check the coalesced _values().
- **L207** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L208** EN: Keeps the inline comment or directive: coalesce is not supported in torch.float16 | CN: 保留这一行注释或指令：coalesce is not supported in torch.float16
- **L209** EN: Assigns or updates `param_grad_fp32`. | CN: 对 `param_grad_fp32` 进行赋值或更新。
- **L210** EN: Assigns or updates `param.grad`. | CN: 对 `param.grad` 进行赋值或更新。
- **L211** EN: Assigns or updates `to_unscale`. | CN: 对 `to_unscale` 进行赋值或更新。
- **L212** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L213** EN: Assigns or updates `to_unscale`. | CN: 对 `to_unscale` 进行赋值或更新。
- **L214** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L215** EN: Continues the implementation inside function `_unscale_grads_`. | CN: 继续说明函数 `_unscale_grads_` 内部的实现。
- **L216** EN: Continues the implementation inside function `_unscale_grads_`. | CN: 继续说明函数 `_unscale_grads_` 内部的实现。
- **L217** EN: Continues the implementation inside function `_unscale_grads_`. | CN: 继续说明函数 `_unscale_grads_` 内部的实现。
- **L218** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L219** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L220** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。

### Lines 221-240 / 第 221-240 行

````python
                    torch._amp_foreach_non_finite_check_and_unscale_(
                        grads,
                        per_device_found_inf.get(device),
                        per_device_inv_scale.get(device),
                    )
        # There exist contexts (e.g. w/ `use_orig_params=True`) wherein some
        # ranks may have no (non-zero sized) parameter shards, necessitating the
        # initialization of `per_device_found_inf._per_device_tensors` here
        if not per_device_found_inf._per_device_tensors:
            if self._scale is None:
                raise AssertionError("Expected _scale to be initialized, got None")
            per_device_found_inf.get(self._scale.device)
        return per_device_found_inf._per_device_tensors

    def unscale_(self, optimizer: torch.optim.Optimizer) -> None:
        if not self._enabled:
            return

        self._check_scale_growth_tracker("unscale_")

````

- **L221** EN: Calls `torch._amp_foreach_non_finite_check_and_unscale_` as part of the current workflow. | CN: 在当前流程中调用 `torch._amp_foreach_non_finite_check_and_unscale_`。
- **L222** EN: Continues the implementation inside function `_unscale_grads_`. | CN: 继续说明函数 `_unscale_grads_` 内部的实现。
- **L223** EN: Calls `per_device_found_inf.get` as part of the current workflow. | CN: 在当前流程中调用 `per_device_found_inf.get`。
- **L224** EN: Calls `per_device_inv_scale.get` as part of the current workflow. | CN: 在当前流程中调用 `per_device_inv_scale.get`。
- **L225** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L226** EN: Keeps the inline comment or directive: There exist contexts (e.g. w/ `use_orig_params=True`) wherein some | CN: 保留这一行注释或指令：There exist contexts (e.g. w/ `use_orig_params=True`) wherein some
- **L227** EN: Keeps the inline comment or directive: ranks may have no (non-zero sized) parameter shards, necessitating the | CN: 保留这一行注释或指令：ranks may have no (non-zero sized) parameter shards, necessitating the
- **L228** EN: Keeps the inline comment or directive: initialization of `per_device_found_inf._per_device_tensors` here | CN: 保留这一行注释或指令：initialization of `per_device_found_inf._per_device_tensors` here
- **L229** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L230** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L231** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L232** EN: Calls `per_device_found_inf.get` as part of the current workflow. | CN: 在当前流程中调用 `per_device_found_inf.get`。
- **L233** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L234** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L235** EN: Defines function `unscale_`. | CN: 定义函数 `unscale_`。
- **L236** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L237** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L238** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L239** EN: Calls `self._check_scale_growth_tracker` as part of the current workflow. | CN: 在当前流程中调用 `self._check_scale_growth_tracker`。
- **L240** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 241-260 / 第 241-260 行

````python
        optimizer_state = self._per_optimizer_states[id(optimizer)]

        if optimizer_state["stage"] is OptState.UNSCALED:
            raise RuntimeError(
                "unscale_() has already been called on this optimizer since the last update()."
            )
        elif optimizer_state["stage"] is OptState.STEPPED:
            raise RuntimeError("unscale_() is being called after step().")

        # FP32 division can be imprecise for certain compile options, so we carry out the reciprocal in FP64.
        if self._scale is None:
            raise AssertionError("Expected _scale to be initialized, got None")
        inv_scale = self._scale.double().reciprocal().float()
        found_inf = torch.full(
            (1,), 0.0, dtype=torch.float32, device=self._scale.device
        )

        optimizer_state["found_inf_per_device"] = self._unscale_grads_(
            optimizer, inv_scale, found_inf, True
        )
````

- **L241** EN: Assigns or updates `optimizer_state`. | CN: 对 `optimizer_state` 进行赋值或更新。
- **L242** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L243** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L244** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L245** EN: Continues the implementation inside function `unscale_`. | CN: 继续说明函数 `unscale_` 内部的实现。
- **L246** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L247** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L248** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L249** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L250** EN: Keeps the inline comment or directive: FP32 division can be imprecise for certain compile options, so we carry out the  | CN: 保留这一行注释或指令：FP32 division can be imprecise for certain compile options, so we carry out the 
- **L251** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L252** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L253** EN: Assigns or updates `inv_scale`. | CN: 对 `inv_scale` 进行赋值或更新。
- **L254** EN: Assigns or updates `found_inf`. | CN: 对 `found_inf` 进行赋值或更新。
- **L255** EN: Continues the implementation inside function `unscale_`. | CN: 继续说明函数 `unscale_` 内部的实现。
- **L256** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L257** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L258** EN: Continues the implementation inside function `unscale_`. | CN: 继续说明函数 `unscale_` 内部的实现。
- **L259** EN: Continues the implementation inside function `unscale_`. | CN: 继续说明函数 `unscale_` 内部的实现。
- **L260** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

### Lines 261-280 / 第 261-280 行

````python
        optimizer_state["stage"] = OptState.UNSCALED

        # Synchronize the detected inf across the ranks
        optimizer_state = self._per_optimizer_states[id(optimizer)]
        works = []
        found_inf_on_cpus = []
        found_inf_on_devices = []

        for found_inf in optimizer_state["found_inf_per_device"].values():
            if self._device != "cpu" and found_inf.device.type == "cpu":
                found_inf_on_cpus.append(found_inf)
                found_inf_on_device = found_inf.to(self._device)
                found_inf_on_devices.append(found_inf_on_device)
                works.append(
                    dist.all_reduce(
                        found_inf_on_device, async_op=True, group=self.process_group
                    )
                )
            else:
                works.append(
````

- **L261** EN: Continues the implementation inside function `unscale_`. | CN: 继续说明函数 `unscale_` 内部的实现。
- **L262** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L263** EN: Keeps the inline comment or directive: Synchronize the detected inf across the ranks | CN: 保留这一行注释或指令：Synchronize the detected inf across the ranks
- **L264** EN: Assigns or updates `optimizer_state`. | CN: 对 `optimizer_state` 进行赋值或更新。
- **L265** EN: Assigns or updates `works`. | CN: 对 `works` 进行赋值或更新。
- **L266** EN: Assigns or updates `found_inf_on_cpus`. | CN: 对 `found_inf_on_cpus` 进行赋值或更新。
- **L267** EN: Assigns or updates `found_inf_on_devices`. | CN: 对 `found_inf_on_devices` 进行赋值或更新。
- **L268** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L269** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L270** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L271** EN: Calls `found_inf_on_cpus.append` as part of the current workflow. | CN: 在当前流程中调用 `found_inf_on_cpus.append`。
- **L272** EN: Assigns or updates `found_inf_on_device`. | CN: 对 `found_inf_on_device` 进行赋值或更新。
- **L273** EN: Calls `found_inf_on_devices.append` as part of the current workflow. | CN: 在当前流程中调用 `found_inf_on_devices.append`。
- **L274** EN: Calls `works.append` as part of the current workflow. | CN: 在当前流程中调用 `works.append`。
- **L275** EN: Calls `dist.all_reduce` as part of the current workflow. | CN: 在当前流程中调用 `dist.all_reduce`。
- **L276** EN: Assigns or updates `found_inf_on_device, async_op`. | CN: 对 `found_inf_on_device, async_op` 进行赋值或更新。
- **L277** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L278** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L279** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L280** EN: Calls `works.append` as part of the current workflow. | CN: 在当前流程中调用 `works.append`。

### Lines 281-300 / 第 281-300 行

````python
                    dist.all_reduce(found_inf, async_op=True, group=self.process_group)
                )
        for work in works:
            work.wait()
        if found_inf_on_cpus:
            torch._foreach_copy_(found_inf_on_cpus, found_inf_on_devices)

    def _amp_update_scale_cpu_(self, found_inf: torch.Tensor) -> None:
        """
        If found_inf is 1.0 (True), then scale is multiplied by backoff_factor and growth_tracker is set to zero.
        Otherwise, scale is multiplied by the growth factor when the growth interval is reached.
        """
        if self._scale is None or self._growth_tracker is None:
            raise AssertionError(
                "Expected _scale and _growth_tracker to be initialized, got None"
            )

        if found_inf.item() >= 1.0:
            self._scale *= self._backoff_factor
            self._growth_tracker.fill_(0)
````

- **L281** EN: Calls `dist.all_reduce` as part of the current workflow. | CN: 在当前流程中调用 `dist.all_reduce`。
- **L282** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L283** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L284** EN: Calls `work.wait` as part of the current workflow. | CN: 在当前流程中调用 `work.wait`。
- **L285** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L286** EN: Calls `torch._foreach_copy_` as part of the current workflow. | CN: 在当前流程中调用 `torch._foreach_copy_`。
- **L287** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L288** EN: Defines function `_amp_update_scale_cpu_`. | CN: 定义函数 `_amp_update_scale_cpu_`。
- **L289** EN: Starts the docstring for the function _amp_update_scale_cpu_. | CN: 开始定义 function _amp_update_scale_cpu_ 的文档字符串。
- **L290** EN: Continues the docstring text for the function _amp_update_scale_cpu_. | CN: 继续补充 function _amp_update_scale_cpu_ 的文档字符串内容。
- **L291** EN: Continues the docstring text for the function _amp_update_scale_cpu_. | CN: 继续补充 function _amp_update_scale_cpu_ 的文档字符串内容。
- **L292** EN: Closes the docstring for the function _amp_update_scale_cpu_. | CN: 结束 function _amp_update_scale_cpu_ 的文档字符串。
- **L293** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L294** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L295** EN: Continues the implementation inside function `_amp_update_scale_cpu_`. | CN: 继续说明函数 `_amp_update_scale_cpu_` 内部的实现。
- **L296** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L297** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L298** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L299** EN: Assigns or updates `self._scale *`. | CN: 对 `self._scale *` 进行赋值或更新。
- **L300** EN: Calls `self._growth_tracker.fill_` as part of the current workflow. | CN: 在当前流程中调用 `self._growth_tracker.fill_`。

### Lines 301-320 / 第 301-320 行

````python
        else:
            successful = self._growth_tracker + 1
            if successful == self._growth_interval:
                self._scale *= self._growth_factor
                self._growth_tracker.fill_(0)
            else:
                self._growth_tracker = successful

    def update(self, new_scale: float | torch.Tensor | None = None) -> None:
        """
        Updates the scale factor.
        If any optimizer steps were skipped the scale is multiplied by ``backoff_factor``
        to reduce it. If ``growth_interval`` unskipped iterations occurred consecutively,
        the scale is multiplied by ``growth_factor`` to increase it.
        Passing ``new_scale`` sets the new scale value manually. (``new_scale`` is not
        used directly, it's used to fill GradScaler's internal scale tensor. So if
        ``new_scale`` was a tensor, later in-place changes to that tensor will not further
        affect the scale GradScaler uses internally.)
        Args:
            new_scale (float or :class:`torch.Tensor`, optional, default=None):  New scale factor.
````

- **L301** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L302** EN: Assigns or updates `successful`. | CN: 对 `successful` 进行赋值或更新。
- **L303** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L304** EN: Assigns or updates `self._scale *`. | CN: 对 `self._scale *` 进行赋值或更新。
- **L305** EN: Calls `self._growth_tracker.fill_` as part of the current workflow. | CN: 在当前流程中调用 `self._growth_tracker.fill_`。
- **L306** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L307** EN: Assigns or updates `self._growth_tracker`. | CN: 对 `self._growth_tracker` 进行赋值或更新。
- **L308** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L309** EN: Defines function `update`. | CN: 定义函数 `update`。
- **L310** EN: Starts the docstring for the function update. | CN: 开始定义 function update 的文档字符串。
- **L311** EN: Continues the docstring text for the function update. | CN: 继续补充 function update 的文档字符串内容。
- **L312** EN: Continues the docstring text for the function update. | CN: 继续补充 function update 的文档字符串内容。
- **L313** EN: Continues the docstring text for the function update. | CN: 继续补充 function update 的文档字符串内容。
- **L314** EN: Continues the docstring text for the function update. | CN: 继续补充 function update 的文档字符串内容。
- **L315** EN: Continues the docstring text for the function update. | CN: 继续补充 function update 的文档字符串内容。
- **L316** EN: Continues the docstring text for the function update. | CN: 继续补充 function update 的文档字符串内容。
- **L317** EN: Continues the docstring text for the function update. | CN: 继续补充 function update 的文档字符串内容。
- **L318** EN: Continues the docstring text for the function update. | CN: 继续补充 function update 的文档字符串内容。
- **L319** EN: Continues the docstring text for the function update. | CN: 继续补充 function update 的文档字符串内容。
- **L320** EN: Continues the docstring text for the function update. | CN: 继续补充 function update 的文档字符串内容。

### Lines 321-340 / 第 321-340 行

````python
        .. warning::
            :meth:`update` should only be called at the end of the iteration, after ``scaler.step(optimizer)`` has
            been invoked for all optimizers used this iteration.
        """

        if not self._enabled:
            return

        _scale, _growth_tracker = self._check_scale_growth_tracker("update")  # type: ignore[var-annotated]

        if new_scale is not None:
            # Accept a new user-defined scale.
            if isinstance(new_scale, float):
                self._scale.fill_(new_scale)  # type: ignore[union-attr]
            else:
                reason = (
                    "new_scale should be a float or a 1-element torch.cuda.FloatTensor or "
                    "torch.FloatTensor with requires_grad=False."
                )
                if new_scale.device.type != self._device:
````

- **L321** EN: Continues the docstring text for the function update. | CN: 继续补充 function update 的文档字符串内容。
- **L322** EN: Continues the docstring text for the function update. | CN: 继续补充 function update 的文档字符串内容。
- **L323** EN: Continues the docstring text for the function update. | CN: 继续补充 function update 的文档字符串内容。
- **L324** EN: Closes the docstring for the function update. | CN: 结束 function update 的文档字符串。
- **L325** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L326** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L327** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L328** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L329** EN: Assigns or updates `_scale, _growth_tracker`. | CN: 对 `_scale, _growth_tracker` 进行赋值或更新。
- **L330** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L331** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L332** EN: Keeps the inline comment or directive: Accept a new user-defined scale. | CN: 保留这一行注释或指令：Accept a new user-defined scale.
- **L333** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L334** EN: Calls `self._scale.fill_` as part of the current workflow. | CN: 在当前流程中调用 `self._scale.fill_`。
- **L335** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L336** EN: Assigns or updates `reason`. | CN: 对 `reason` 进行赋值或更新。
- **L337** EN: Continues the implementation inside function `update`. | CN: 继续说明函数 `update` 内部的实现。
- **L338** EN: Continues the implementation inside function `update`. | CN: 继续说明函数 `update` 内部的实现。
- **L339** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L340** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 341-360 / 第 341-360 行

````python
                    raise AssertionError(reason)
                if new_scale.numel() != 1:
                    raise AssertionError(reason)
                if new_scale.requires_grad is not False:
                    raise AssertionError(reason)
                self._scale.copy_(new_scale)  # type: ignore[union-attr]
        else:
            # Consume shared inf/nan data collected from optimizers to update the scale.
            # If all found_inf tensors are on the same device as self._scale, this operation is asynchronous.
            found_infs = [
                found_inf.to(device=_scale.device, non_blocking=True)
                for state in self._per_optimizer_states.values()
                for found_inf in state["found_inf_per_device"].values()
            ]

            if len(found_infs) == 0:
                raise AssertionError("No inf checks were recorded prior to update.")

            found_inf_combined = found_infs[0]
            if len(found_infs) > 1:
````

- **L341** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L342** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L343** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L344** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L345** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L346** EN: Calls `self._scale.copy_` as part of the current workflow. | CN: 在当前流程中调用 `self._scale.copy_`。
- **L347** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L348** EN: Keeps the inline comment or directive: Consume shared inf/nan data collected from optimizers to update the scale. | CN: 保留这一行注释或指令：Consume shared inf/nan data collected from optimizers to update the scale.
- **L349** EN: Keeps the inline comment or directive: If all found_inf tensors are on the same device as self._scale, this operation i | CN: 保留这一行注释或指令：If all found_inf tensors are on the same device as self._scale, this operation i
- **L350** EN: Assigns or updates `found_infs`. | CN: 对 `found_infs` 进行赋值或更新。
- **L351** EN: Calls `found_inf.to` as part of the current workflow. | CN: 在当前流程中调用 `found_inf.to`。
- **L352** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L353** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L354** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L355** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L356** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L357** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L358** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L359** EN: Assigns or updates `found_inf_combined`. | CN: 对 `found_inf_combined` 进行赋值或更新。
- **L360** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 361-377 / 第 361-377 行

````python
                for i in range(1, len(found_infs)):
                    found_inf_combined += found_infs[i]

            if _scale.device.type == "cpu":
                self._amp_update_scale_cpu_(found_inf_combined)
            else:
                torch._amp_update_scale_(
                    self._scale,  # type: ignore[arg-type]
                    self._growth_tracker,  # type: ignore[arg-type]
                    found_inf_combined,
                    self._growth_factor,  # type: ignore[arg-type]
                    self._backoff_factor,  # type: ignore[arg-type]
                    self._growth_interval,  # type: ignore[arg-type]
                )

        # To prepare for next iteration, clear the data collected from optimizers this iteration.
        self._per_optimizer_states = defaultdict(_refresh_per_optimizer_state)
````

- **L361** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L362** EN: Continues the implementation inside function `update`. | CN: 继续说明函数 `update` 内部的实现。
- **L363** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L364** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L365** EN: Calls `self._amp_update_scale_cpu_` as part of the current workflow. | CN: 在当前流程中调用 `self._amp_update_scale_cpu_`。
- **L366** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L367** EN: Calls `torch._amp_update_scale_` as part of the current workflow. | CN: 在当前流程中调用 `torch._amp_update_scale_`。
- **L368** EN: Continues the implementation inside function `update`. | CN: 继续说明函数 `update` 内部的实现。
- **L369** EN: Continues the implementation inside function `update`. | CN: 继续说明函数 `update` 内部的实现。
- **L370** EN: Continues the implementation inside function `update`. | CN: 继续说明函数 `update` 内部的实现。
- **L371** EN: Continues the implementation inside function `update`. | CN: 继续说明函数 `update` 内部的实现。
- **L372** EN: Continues the implementation inside function `update`. | CN: 继续说明函数 `update` 内部的实现。
- **L373** EN: Continues the implementation inside function `update`. | CN: 继续说明函数 `update` 内部的实现。
- **L374** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L375** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L376** EN: Keeps the inline comment or directive: To prepare for next iteration, clear the data collected from optimizers this ite | CN: 保留这一行注释或指令：To prepare for next iteration, clear the data collected from optimizers this ite
- **L377** EN: Assigns or updates `self._per_optimizer_states`. | CN: 对 `self._per_optimizer_states` 进行赋值或更新。

## Key Concepts / 关键概念

- **EN**: Fully Sharded Data Parallel runtime and utilities  
  **CN**: Fully Sharded Data Parallel（FSDP）运行时与工具
- **EN**: process groups  
  **CN**: 进程组
- **EN**: sharding  
  **CN**: 分片
- **EN**: all-reduce  
  **CN**: all-reduce 聚合
- **EN**: optimizer coordination  
  **CN**: 优化器协同
- **EN**: Primary classes: _GeneralMultiDeviceReplicator, ShardedGradScaler  
  **CN**: 主要类：_GeneralMultiDeviceReplicator, ShardedGradScaler
- **EN**: Core callables: _refresh_per_optimizer_state, _is_supported_device  
  **CN**: 核心可调用对象：_refresh_per_optimizer_state, _is_supported_device

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed`, `torch.distributed.distributed_c10d`
- **PyTorch / PyTorch**: `torch`, `torch.amp.grad_scaler`
- **Python Stdlib / Python 标准库**: `collections`, `collections.abc`, `logging`, `typing`
- **Third-party / 第三方**: None detected / 未检测到

