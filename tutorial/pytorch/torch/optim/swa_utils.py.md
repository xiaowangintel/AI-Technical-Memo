# swa_utils.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/optim/swa_utils.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements optimizer algorithms, shared optimizer utilities, and scheduling-facing helpers.
- **Purpose (CN)**: 实现优化器算法、共享优化器工具以及面向调度器的辅助逻辑。
## Line-by-Line Analysis / 逐行分析

### Lines 1-17 / 第 1-17 行
````python
# mypy: allow-untyped-defs
r"""Implementation for Stochastic Weight Averaging implementation."""

from __future__ import annotations

import itertools
import math
import warnings
from copy import deepcopy
from typing import Any, cast, Literal, TYPE_CHECKING
from typing_extensions import override

import torch
from torch import Tensor
from torch.nn import Module
from torch.optim.lr_scheduler import _format_param, LRScheduler
from torch.utils._foreach_utils import _get_foreach_kernels_supported_devices
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch.nn, torch.optim.lr_scheduler, ...; standard-library helpers such as __future__, itertools, math, ...; other helper packages such as typing_extensions. The future import postpones annotation evaluation, keeping type hints lightweight at import time. The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch.nn、torch.optim.lr_scheduler、...；标准库辅助模块，如 __future__、itertools、math、...；其他辅助包，如 typing_extensions。 `__future__` 导入会推迟注解求值，从而在导入阶段降低类型提示的开销。 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 20-36 / 第 20-36 行
````python
if TYPE_CHECKING:
    from collections.abc import Callable, Iterable

    from .optimizer import Optimizer


__all__ = [
    "AveragedModel",
    "update_bn",
    "SWALR",
    "get_ema_multi_avg_fn",
    "get_swa_multi_avg_fn",
    "get_ema_avg_fn",
    "get_swa_avg_fn",
]

from torch.utils._foreach_utils import _group_tensors_by_device_and_dtype
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as .optimizer, torch.utils._foreach_utils; standard-library helpers such as collections.abc. `__all__` defines the public symbols that this module chooses to export. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 .optimizer、torch.utils._foreach_utils；标准库辅助模块，如 collections.abc。 `__all__` 定义了本模块选择导出的公共符号。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 39-55 / 第 39-55 行
````python
PARAM_LIST = tuple[Tensor, ...] | list[Tensor]


def get_ema_multi_avg_fn(decay=0.999):
    """Get the function applying exponential moving average (EMA) across multiple params.

    The EMA is computed as:

    .. math::
        W_0^{\\text{EMA}} = W_0^{\\text{model}}

    .. math::
        W_{t+1}^{\\text{EMA}} = \\text{decay} \\times W_t^{\\text{EMA}} + (1 - \\text{decay}) \\times W_{t+1}^{\\text{model}}

    where :math:`W_t^{\\text{EMA}}` is the EMA parameter at step :math:`t`,
    :math:`W_t^{\\text{model}}` is the model parameter at step :math:`t`,
    and :math:`\\text{decay}` is the decay rate (default: 0.999).
````
- **EN**: This chunk defines `get_ema_multi_avg_fn`, which retrieves runtime state and exposes it through a Python-friendly accessor.
- **CN**: 这一段定义了 `get_ema_multi_avg_fn`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。

### Lines 57-76 / 第 57-76 行
````python
    Args:
        decay (float): Decay rate for EMA. Must be in the range [0, 1]. Default: 0.999

    Returns:
        Callable: A function that updates EMA parameters given current model parameters
    """

    if decay < 0.0 or decay > 1.0:
        raise ValueError(
            f"Invalid decay value {decay} provided. Please provide a value in [0,1] range."
        )

    @torch.no_grad()
    def ema_update(
        ema_param_list: PARAM_LIST, current_param_list: PARAM_LIST, _
    ) -> None:
        # foreach lerp only handles float and complex
        if torch.is_floating_point(ema_param_list[0]) or torch.is_complex(
            ema_param_list[0]
        ):
````
- **EN**: This chunk defines `ema_update`, which advances mutable state using the current inputs, gradients, or counters. Decorators such as `torch.no_grad` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `ema_update`，其作用是利用当前输入、梯度或计数器推进可变状态。 像 `torch.no_grad` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 77-96 / 第 77-96 行
````python
            torch._foreach_lerp_(ema_param_list, current_param_list, 1 - decay)
        else:
            for p_ema, p_model in zip(ema_param_list, current_param_list, strict=True):
                p_ema.copy_(p_ema * decay + p_model * (1 - decay))

    return ema_update


def get_swa_multi_avg_fn():
    """Get the function applying stochastic weight average (SWA) across multiple params."""

    @torch.no_grad()
    def swa_update(
        averaged_param_list: PARAM_LIST,
        current_param_list: PARAM_LIST,
        num_averaged: Tensor | int,
    ) -> None:
        # foreach lerp only handles float and complex
        if torch.is_floating_point(averaged_param_list[0]) or torch.is_complex(
            averaged_param_list[0]
````
- **EN**: This chunk defines `swa_update`, which advances mutable state using the current inputs, gradients, or counters. Decorators such as `torch.no_grad` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `swa_update`，其作用是利用当前输入、梯度或计数器推进可变状态。 像 `torch.no_grad` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 97-114 / 第 97-114 行
````python
        ):
            torch._foreach_lerp_(
                averaged_param_list,
                current_param_list,
                cast(float, 1 / (num_averaged + 1)),
            )
        else:
            diffs = torch._foreach_sub(current_param_list, averaged_param_list)
            if isinstance(num_averaged, Tensor):
                torch._foreach_addcdiv_(
                    averaged_param_list,
                    diffs,
                    [num_averaged + 1] * len(averaged_param_list),
                )
            else:
                torch._foreach_add_(
                    averaged_param_list, diffs, alpha=1.0 / (num_averaged + 1)
                )
````
- **EN**: This chunk continues `swa_update` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `swa_update`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 116-132 / 第 116-132 行
````python
    return swa_update


def get_ema_avg_fn(decay=0.999):
    """Get the function applying exponential moving average (EMA) across multiple params.

    The EMA is computed as:

    .. math::
        W_0^{\\text{EMA}} = W_0^{\\text{model}}

    .. math::
        W_{t+1}^{\\text{EMA}} = \\text{decay} \\times W_t^{\\text{EMA}} + (1 - \\text{decay}) \\times W_{t+1}^{\\text{model}}

    where :math:`W_t^{\\text{EMA}}` is the EMA parameter at step :math:`t`,
    :math:`W_t^{\\text{model}}` is the model parameter at step :math:`t`,
    and :math:`\\text{decay}` is the decay rate (default: 0.999).
````
- **EN**: This chunk defines `get_ema_avg_fn`, which retrieves runtime state and exposes it through a Python-friendly accessor. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `get_ema_avg_fn`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 134-150 / 第 134-150 行
````python
    Args:
        decay (float): Decay rate for EMA. Must be in the range [0, 1]. Default: 0.999

    Returns:
        Callable: A function that updates EMA parameters given current model parameters
    """

    if decay < 0.0 or decay > 1.0:
        raise ValueError(
            f"Invalid decay value {decay} provided. Please provide a value in [0,1] range."
        )

    @torch.no_grad()
    def ema_update(ema_param: Tensor, current_param: Tensor, num_averaged):
        return decay * ema_param + (1 - decay) * current_param

    return ema_update
````
- **EN**: This chunk defines `ema_update`, which advances mutable state using the current inputs, gradients, or counters. Decorators such as `torch.no_grad` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `ema_update`，其作用是利用当前输入、梯度或计数器推进可变状态。 像 `torch.no_grad` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 153-171 / 第 153-171 行
````python
def get_swa_avg_fn():
    """Get the function applying stochastic weight average (SWA) across a single param."""

    @torch.no_grad()
    def swa_update(
        averaged_param: Tensor, current_param: Tensor, num_averaged: Tensor | int
    ):
        return averaged_param + (current_param - averaged_param) / (num_averaged + 1)

    return swa_update


class AveragedModel(Module):
    r"""Implements averaged model for Stochastic Weight Averaging (SWA) and Exponential Moving Average (EMA).

    Stochastic Weight Averaging was proposed in `Averaging Weights Leads to
    Wider Optima and Better Generalization`_ by Pavel Izmailov, Dmitrii
    Podoprikhin, Timur Garipov, Dmitry Vetrov and Andrew Gordon Wilson
    (UAI 2018).
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. It introduces or extends `AveragedModel`, which hold the main object-oriented state for this portion of the file. This chunk defines `swa_update`, which advances mutable state using the current inputs, gradients, or counters. Decorators such as `torch.no_grad` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 它引入或扩展了 `AveragedModel`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `swa_update`，其作用是利用当前输入、梯度或计数器推进可变状态。 像 `torch.no_grad` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 173-192 / 第 173-192 行
````python
    Exponential Moving Average is a variation of `Polyak averaging`_,
    but using exponential weights instead of equal weights across iterations.

    AveragedModel class creates a copy of the provided module :attr:`model`
    on the device :attr:`device` and allows to compute running averages of the
    parameters of the :attr:`model`.

    Args:
        model (torch.nn.Module): model to use with SWA/EMA
        device (torch.device, optional): if provided, the averaged model will be
            stored on the :attr:`device`
        avg_fn (function, optional): the averaging function used to update
            parameters; the function must take in the current value of the
            :class:`AveragedModel` parameter, the current value of :attr:`model`
            parameter, and the number of models already averaged; if None,
            an equally weighted average is used (default: None)
        multi_avg_fn (function, optional): the averaging function used to update
            parameters inplace; the function must take in the current values of the
            :class:`AveragedModel` parameters as a list, the current values of :attr:`model`
            parameters as a list, and the number of models already averaged; if None,
````
- **EN**: This chunk continues `AveragedModel` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `AveragedModel`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 193-212 / 第 193-212 行
````python
            an equally weighted average is used (default: None)
        use_buffers (bool): if ``True``, it will compute running averages for
            both the parameters and the buffers of the model. (default: ``False``)

    Example:
        >>> # xdoctest: +SKIP("undefined variables")
        >>> loader, optimizer, model, loss_fn = ...
        >>> swa_model = torch.optim.swa_utils.AveragedModel(model)
        >>> scheduler = torch.optim.lr_scheduler.CosineAnnealingLR(optimizer,
        >>>                                     T_max=300)
        >>> swa_start = 160
        >>> swa_scheduler = SWALR(optimizer, swa_lr=0.05)
        >>> for i in range(300):
        >>>      for input, target in loader:
        >>>          optimizer.zero_grad()
        >>>          loss_fn(model(input), target).backward()
        >>>          optimizer.step()
        >>>      if i > swa_start:
        >>>          swa_model.update_parameters(model)
        >>>          swa_scheduler.step()
````
- **EN**: This chunk continues `AveragedModel` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `AveragedModel`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 213-227 / 第 213-227 行
````python
        >>>      else:
        >>>          scheduler.step()
        >>>
        >>> # Update bn statistics for the swa_model at the end
        >>> torch.optim.swa_utils.update_bn(loader, swa_model)

    You can also use custom averaging functions with the `avg_fn` or `multi_avg_fn` parameters.
    If no averaging function is provided, the default is to compute
    equally-weighted average of the weights (SWA).

    Example:
        >>> # xdoctest: +SKIP("undefined variables")
        >>> # Compute exponential moving averages of the weights and buffers
        >>> ema_model = torch.optim.swa_utils.AveragedModel(model,
        >>>             torch.optim.swa_utils.get_ema_multi_avg_fn(0.9), use_buffers=True)
````
- **EN**: This chunk continues `AveragedModel` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `AveragedModel`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 229-241 / 第 229-241 行
````python
    .. note::
        When using SWA/EMA with models containing Batch Normalization you may
        need to update the activation statistics for Batch Normalization.
        This can be done either by using the :meth:`torch.optim.swa_utils.update_bn`
        or by setting :attr:`use_buffers` to `True`. The first approach updates the
        statistics in a post-training step by passing data through the model. The
        second does it during the parameter update phase by averaging all buffers.
        Empirical evidence has shown that updating the statistics in normalization
        layers increases accuracy, but you may wish to empirically test which
        approach yields the best results in your problem.

    .. note::
        :attr:`avg_fn` and `multi_avg_fn` are not saved in the :meth:`state_dict` of the model.
````
- **EN**: This chunk continues `AveragedModel` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `AveragedModel`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 243-262 / 第 243-262 行
````python
    .. note::
        When :meth:`update_parameters` is called for the first time (i.e.
        :attr:`n_averaged` is `0`) the parameters of `model` are copied
        to the parameters of :class:`AveragedModel`. For every subsequent
        call of :meth:`update_parameters` the function `avg_fn` is used
        to update the parameters.

    .. _Averaging Weights Leads to Wider Optima and Better Generalization:
        https://arxiv.org/abs/1803.05407
    .. _There Are Many Consistent Explanations of Unlabeled Data: Why You Should
        Average:
        https://arxiv.org/abs/1806.05594
    .. _SWALP: Stochastic Weight Averaging in Low-Precision Training:
        https://arxiv.org/abs/1904.11943
    .. _Stochastic Weight Averaging in Parallel: Large-Batch Training That
        Generalizes Well:
        https://arxiv.org/abs/2001.02312
    .. _Polyak averaging:
        https://paperswithcode.com/method/polyak-averaging
    """
````
- **EN**: This chunk continues `AveragedModel` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `AveragedModel`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 264-283 / 第 264-283 行
````python
    n_averaged: Tensor

    def __init__(
        self,
        model: Module,
        device: int | torch.device | None = None,
        avg_fn: Callable[[Tensor, Tensor, Tensor | int], Tensor] | None = None,
        multi_avg_fn: Callable[[PARAM_LIST, PARAM_LIST, Tensor | int], None]
        | None = None,
        use_buffers=False,
    ) -> None:
        super().__init__()
        if avg_fn is not None and multi_avg_fn is not None:
            raise AssertionError(
                "Only one of avg_fn and multi_avg_fn should be provided"
            )
        self.module = deepcopy(model)
        if device is not None:
            self.module = self.module.to(device)
        self.register_buffer(
````
- **EN**: This chunk defines `__init__`, which initializes object state and binds construction-time configuration. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 284-303 / 第 284-303 行
````python
            "n_averaged", torch.tensor(0, dtype=torch.long, device=device)
        )
        self.avg_fn = avg_fn
        self.multi_avg_fn = multi_avg_fn
        self.use_buffers = use_buffers

    def forward(self, *args, **kwargs):
        """Forward pass."""
        return self.module(*args, **kwargs)

    def update_parameters(self, model: Module) -> None:
        """Update model parameters."""
        self_param = (
            # pyrefly: ignore [bad-argument-type]
            itertools.chain(self.module.parameters(), self.module.buffers())
            if self.use_buffers
            else self.parameters()
        )
        model_param = (
            # pyrefly: ignore [bad-argument-type]
````
- **EN**: This chunk defines `update_parameters`, which advances mutable state using the current inputs, gradients, or counters. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `update_parameters`，其作用是利用当前输入、梯度或计数器推进可变状态。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 304-316 / 第 304-316 行
````python
            itertools.chain(model.parameters(), model.buffers())
            if self.use_buffers
            else model.parameters()
        )
        self_param_detached: list[Tensor | None] = []
        model_param_detached: list[Tensor | None] = []
        copy_param = bool(self.n_averaged == 0)
        for p_averaged, p_model in zip(self_param, model_param, strict=False):
            p_model_ = p_model.detach().to(p_averaged.device)
            self_param_detached.append(p_averaged.detach())
            model_param_detached.append(p_model_)
            if copy_param:
                p_averaged.detach().copy_(p_model_)
````
- **EN**: This chunk continues `update_parameters` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `update_parameters`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 318-337 / 第 318-337 行
````python
        if self.n_averaged > 0:
            if self.multi_avg_fn is not None or self.avg_fn is None:
                grouped_tensors = _group_tensors_by_device_and_dtype(
                    [self_param_detached, model_param_detached]
                )
                for (device, _), (
                    [self_params, model_params],
                    _,
                ) in grouped_tensors.items():
                    if self.multi_avg_fn:
                        self.multi_avg_fn(
                            self_params,  # type: ignore[arg-type]
                            model_params,  # type: ignore[arg-type]
                            self.n_averaged.to(device),
                        )
                    elif (
                        device is not None
                        and device.type in _get_foreach_kernels_supported_devices()
                    ):
                        multi_avg_fn = get_swa_multi_avg_fn()
````
- **EN**: This chunk continues `update_parameters` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `update_parameters`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 338-357 / 第 338-357 行
````python
                        multi_avg_fn(
                            self_params, model_params, self.n_averaged.to(device)
                        )
                    else:
                        avg_fn = get_swa_avg_fn()
                        n_averaged = self.n_averaged.to(device)
                        for p_averaged, p_model in zip(  # type: ignore[assignment]
                            self_params, model_params, strict=True
                        ):
                            # pyrefly: ignore [missing-attribute]
                            p_averaged.copy_(avg_fn(p_averaged, p_model, n_averaged))
            else:
                for p_averaged, p_model in zip(  # type: ignore[assignment]
                    self_param_detached, model_param_detached, strict=True
                ):
                    # pyrefly: ignore [missing-attribute]
                    n_averaged = self.n_averaged.to(p_averaged.device)
                    # pyrefly: ignore [missing-attribute]
                    p_averaged.detach().copy_(
                        # pyrefly: ignore [missing-attribute, bad-argument-type]
````
- **EN**: This chunk continues `update_parameters` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `update_parameters`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 358-377 / 第 358-377 行
````python
                        self.avg_fn(p_averaged.detach(), p_model, n_averaged)
                    )

        if not self.use_buffers:
            # If not apply running averages to the buffers,
            # keep the buffers in sync with the source model.
            for b_swa, b_model in zip(
                self.module.buffers(), model.buffers(), strict=True
            ):
                b_swa.detach().copy_(b_model.detach().to(b_swa.device))
        self.n_averaged += 1


@torch.no_grad()
def update_bn(
    loader: Iterable[Any],
    model: Module,
    device: int | torch.device | None = None,
) -> None:
    r"""Update BatchNorm running_mean, running_var buffers in the model.
````
- **EN**: This chunk defines `update_bn`, which advances mutable state using the current inputs, gradients, or counters. Decorators such as `torch.no_grad` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `update_bn`，其作用是利用当前输入、梯度或计数器推进可变状态。 像 `torch.no_grad` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 379-395 / 第 379-395 行
````python
    It performs one pass over data in `loader` to estimate the activation
    statistics for BatchNorm layers in the model.

    Args:
        loader (torch.utils.data.DataLoader): dataset loader to compute the
            activation statistics on. Each data batch should be either a
            tensor, or a list/tuple whose first element is a tensor
            containing data.
        model (torch.nn.Module): model for which we seek to update BatchNorm
            statistics.
        device (torch.device, optional): If set, data will be transferred to
            :attr:`device` before being passed into :attr:`model`.

    Example:
        >>> # xdoctest: +SKIP("Undefined variables")
        >>> loader, model = ...
        >>> torch.optim.swa_utils.update_bn(loader, model)
````
- **EN**: This chunk continues `update_bn` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `update_bn`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 397-415 / 第 397-415 行
````python
    .. note::
        The `update_bn` utility assumes that each data batch in :attr:`loader`
        is either a tensor or a list or tuple of tensors; in the latter case it
        is assumed that :meth:`model.forward()` should be called on the first
        element of the list or tuple corresponding to the data batch.
    """
    momenta = {}
    for module in model.modules():
        if isinstance(module, torch.nn.modules.batchnorm._BatchNorm):
            module.reset_running_stats()
            momenta[module] = module.momentum

    if not momenta:
        return

    was_training = model.training
    model.train()
    for module in momenta:
        module.momentum = None
````
- **EN**: This chunk continues `update_bn` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `update_bn`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 417-434 / 第 417-434 行
````python
    for input in loader:
        if isinstance(input, (list, tuple)):
            input = input[0]
        if device is not None:
            input = input.to(device)

        model(input)

    for bn_module in momenta:
        bn_module.momentum = momenta[bn_module]
    model.train(was_training)


class SWALR(LRScheduler):
    r"""Anneals the learning rate in each parameter group to a fixed value.

    This learning rate scheduler is meant to be used with Stochastic Weight
    Averaging (SWA) method (see `torch.optim.swa_utils.AveragedModel`).
````
- **EN**: It introduces or extends `SWALR`, which hold the main object-oriented state for this portion of the file. This chunk continues `SWALR` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 它引入或扩展了 `SWALR`，这些类承载了本段涉及的主要面向对象状态。 这一段延续了 `SWALR`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 436-449 / 第 436-449 行
````python
    Args:
        optimizer (torch.optim.Optimizer): wrapped optimizer
        swa_lrs (float or list): the learning rate value for all param groups
            together or separately for each group.
        annealing_epochs (int): number of epochs in the annealing phase
            (default: 10)
        annealing_strategy (str): "cos" or "linear"; specifies the annealing
            strategy: "cos" for cosine annealing, "linear" for linear annealing
            (default: "cos")
        last_epoch (int): the index of the last epoch (default: -1)

    The :class:`SWALR` scheduler can be used together with other
    schedulers to switch to a constant learning rate late in the training
    as in the example below.
````
- **EN**: This chunk continues `SWALR` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `SWALR`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 451-468 / 第 451-468 行
````python
    Example:
        >>> # xdoctest: +SKIP("Undefined variables")
        >>> loader, optimizer, model = ...
        >>> lr_lambda = lambda epoch: 0.9
        >>> scheduler = torch.optim.lr_scheduler.MultiplicativeLR(optimizer,
        >>>        lr_lambda=lr_lambda)
        >>> swa_scheduler = torch.optim.swa_utils.SWALR(optimizer,
        >>>        anneal_strategy="linear", anneal_epochs=20, swa_lr=0.05)
        >>> swa_start = 160
        >>> for i in range(300):
        >>>      for input, target in loader:
        >>>          optimizer.zero_grad()
        >>>          loss_fn(model(input), target).backward()
        >>>          optimizer.step()
        >>>      if i > swa_start:
        >>>          swa_scheduler.step()
        >>>      else:
        >>>          scheduler.step()
````
- **EN**: This chunk continues `SWALR` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `SWALR`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 470-489 / 第 470-489 行
````python
    .. _Averaging Weights Leads to Wider Optima and Better Generalization:
        https://arxiv.org/abs/1803.05407
    """

    def __init__(
        self,
        optimizer: Optimizer,
        swa_lr: float,
        anneal_epochs=10,
        anneal_strategy: Literal["cos", "linear"] = "cos",
        last_epoch=-1,
    ) -> None:
        swa_lrs = _format_param("swa_lr", optimizer, swa_lr)
        for swa_lr, group in zip(swa_lrs, optimizer.param_groups, strict=True):
            group["swa_lr"] = swa_lr
        if anneal_strategy not in ["cos", "linear"]:
            raise ValueError(
                "anneal_strategy must by one of 'cos' or 'linear', "
                f"instead got {anneal_strategy}"
            )
````
- **EN**: Typing-related scaffolding keeps the Python API explicit about expected tensors, literals, or callable shapes. This chunk defines `__init__`, which initializes object state and binds construction-time configuration. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 与类型相关的脚手架让 Python API 更明确地描述期望的张量、字面量或可调用对象形状。 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 490-504 / 第 490-504 行
````python
        self._set_anneal_func(anneal_strategy)
        if not isinstance(anneal_epochs, int) or anneal_epochs < 0:
            raise ValueError(
                f"anneal_epochs must be equal or greater than 0, got {anneal_epochs}"
            )
        self.anneal_epochs = anneal_epochs
        super().__init__(optimizer, last_epoch)

    @staticmethod
    def _linear_anneal(t):
        return t

    @staticmethod
    def _cosine_anneal(t):
        return (1 - math.cos(math.pi * t)) / 2
````
- **EN**: This chunk defines `_cosine_anneal`, which implements a focused step in optimizer state updates. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_cosine_anneal`，其作用是实现优化器状态更新中的一个关键步骤。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 506-520 / 第 506-520 行
````python
    @staticmethod
    def _get_initial_lr(lr, swa_lr, alpha):
        if alpha == 1:
            return swa_lr
        return (lr - alpha * swa_lr) / (1 - alpha)

    @override
    def get_lr(self):
        r"""Compute the next learning rate for each of the optimizer's
        :attr:`~torch.optim.Optimizer.param_groups`.

        Uses :attr:`anneal_func` to interpolate between each group's
        ``group["lr"]`` and ``group["swa_lr"]`` over :attr:`anneal_epochs`
        epochs. Once :attr:`anneal_epochs` is reached, keeps the learning rate
        fixed at ``group["swa_lr"]``.
````
- **EN**: This chunk defines `get_lr`, which retrieves runtime state and exposes it through a Python-friendly accessor. Decorators such as `staticmethod`, `override` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `get_lr`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 像 `staticmethod`、`override` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 522-541 / 第 522-541 行
````python
        Returns:
            list[float | Tensor]: A :class:`list` of learning rates for each of
            the optimizer's :attr:`~torch.optim.Optimizer.param_groups` with the
            same types as their current ``group["lr"]``\s.

        .. note::
            If you're trying to inspect the most recent learning rate, use
            :meth:`get_last_lr()` instead.

        .. note::
            The returned :class:`~torch.Tensor`\s are copies, and never alias
            the optimizer's ``group["lr"]``\s.
        """
        # `_get_lr_called_within_step` is only available `_enable_get_lr_call`,
        # so we ignore the type error here. See `LRScheduler.step()` for more details.
        if not self._get_lr_called_within_step:
            warnings.warn(
                "To get the last learning rate computed by the scheduler, "
                "please use `get_last_lr()`.",
                UserWarning,
````
- **EN**: This chunk continues `get_lr` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `get_lr`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 542-561 / 第 542-561 行
````python
                stacklevel=2,
            )
        # Set in `LRScheduler._initial_step()`
        step = self._step_count - 1
        if self.anneal_epochs == 0:
            step = max(1, step)
        # pyrefly: ignore [no-matching-overload]
        prev_t = max(0, min(1, (step - 1) / max(1, self.anneal_epochs)))
        prev_alpha = self.anneal_func(prev_t)
        prev_lrs = [
            self._get_initial_lr(group["lr"], group["swa_lr"], prev_alpha)
            for group in self.optimizer.param_groups
        ]
        # pyrefly: ignore [no-matching-overload]
        t = max(0, min(1, step / max(1, self.anneal_epochs)))
        alpha = self.anneal_func(t)
        return [
            group["swa_lr"] * alpha + lr * (1 - alpha)
            for group, lr in zip(self.optimizer.param_groups, prev_lrs, strict=True)
        ]
````
- **EN**: This chunk continues `get_lr` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `get_lr`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 563-581 / 第 563-581 行
````python
    def _set_anneal_func(self, anneal_strategy: Literal["cos", "linear"]) -> None:
        self._anneal_strategy = anneal_strategy
        if anneal_strategy == "cos":
            self.anneal_func = self._cosine_anneal
        else:
            self.anneal_func = self._linear_anneal

    @override
    def state_dict(self) -> dict[str, Any]:
        """Return the state of the scheduler as a :class:`dict`.

        It contains an entry for every variable in self.__dict__ which
        is not the optimizer or anneal_func.
        """
        return {
            key: value
            for key, value in self.__dict__.items()
            if key not in ("optimizer", "anneal_func")
        }
````
- **EN**: Typing-related scaffolding keeps the Python API explicit about expected tensors, literals, or callable shapes. This chunk defines `state_dict`, which implements a focused step in optimizer state updates. Decorators such as `override` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 与类型相关的脚手架让 Python API 更明确地描述期望的张量、字面量或可调用对象形状。 这一段定义了 `state_dict`，其作用是实现优化器状态更新中的一个关键步骤。 像 `override` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 583-592 / 第 583-592 行
````python
    @override
    def load_state_dict(self, state_dict: dict[str, Any]) -> None:
        """Load the scheduler's state.

        Args:
            state_dict (dict): scheduler state. Should be an object returned
                from a call to :meth:`state_dict`.
        """
        self.__dict__.update(state_dict)
        self._set_anneal_func(self._anneal_strategy)
````
- **EN**: This chunk defines `load_state_dict`, which serializes or reconstructs state across a Python-visible boundary. Decorators such as `override` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks.
- **CN**: 这一段定义了 `load_state_dict`，其作用是在 Python 可见边界上序列化或重建状态。 像 `override` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。

## Key Concepts / 关键概念

- **Optimization algorithms**
  - EN: Updates model parameters using optimizer state, gradients, and step-wise scheduling rules.
  - CN: 利用优化器状态、梯度和逐步调度规则更新模型参数。
- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **PARAM_LIST**
  - EN: `PARAM_LIST` is one of the main symbols declared or implemented in this file.
  - CN: `PARAM_LIST` 是本文件声明或实现的主要符号之一。
- **Gradient semantics**
  - EN: The implementation preserves gradient correctness and explains how backward/JVP state is tracked.
  - CN: 实现会维护梯度正确性，并说明如何跟踪 backward/JVP 状态。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch.nn`, `torch.optim.lr_scheduler`, `torch.utils._foreach_utils`, `.optimizer`
- **Standard library / 标准库**: `__future__`, `itertools`, `math`, `warnings`, `copy`, `typing`, `collections.abc`
- **Other helper packages / 其他辅助包**: `typing_extensions`
- **Primary symbols in this file / 本文件核心符号**: `__all__`, `PARAM_LIST`, `get_ema_multi_avg_fn`, `get_swa_multi_avg_fn`, `get_ema_avg_fn`, `get_swa_avg_fn`, `AveragedModel`, `update_bn`, `SWALR`
