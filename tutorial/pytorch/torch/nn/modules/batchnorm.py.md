# batchnorm.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/modules/batchnorm.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Defines reusable nn.Module classes, their state, and their forward-path helpers.
- **Purpose (CN)**: 定义可复用的 nn.Module 类、其状态以及前向路径辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-37
```python
# mypy: allow-untyped-defs
from typing import Any

import torch
from torch import Tensor
from torch.nn import functional as F, init
from torch.nn.parameter import Parameter, UninitializedBuffer, UninitializedParameter

from ._functions import SyncBatchNorm as sync_batch_norm
from .lazy import LazyModuleMixin
from .module import Module


__all__ = [
    "BatchNorm1d",
    "LazyBatchNorm1d",
    "BatchNorm2d",
    "LazyBatchNorm2d",
    "BatchNorm3d",
    "LazyBatchNorm3d",
    "SyncBatchNorm",
]


class _NormBase(Module):
    """Common base of _InstanceNorm and _BatchNorm."""

    _version = 2
    __constants__ = ["track_running_stats", "momentum", "eps", "num_features", "affine"]
    num_features: int
    eps: float
    momentum: float | None
    affine: bool
    track_running_stats: bool
    # WARNING: weight and bias purposely not defined here.
    # See https://github.com/pytorch/pytorch/issues/39670
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 38-65
```python
    def __init__(
        self,
        num_features: int,
        eps: float = 1e-5,
        momentum: float | None = 0.1,
        affine: bool = True,
        track_running_stats: bool = True,
        device=None,
        dtype=None,
        *,
        bias: bool = True,
    ) -> None:
        factory_kwargs = {"device": device, "dtype": dtype}
        super().__init__()
        self.num_features = num_features
        self.eps = eps
        self.momentum = momentum
        self.affine = affine
        self.track_running_stats = track_running_stats
        if self.affine:
            self.weight = Parameter(torch.empty(num_features, **factory_kwargs))
            if bias:
                self.bias = Parameter(torch.empty(num_features, **factory_kwargs))
            else:
                self.register_parameter("bias", None)
        else:
            self.register_parameter("weight", None)
            self.register_parameter("bias", None)
```
- **EN**: Defines the `_NormBase.__init__` method; this block introduces logic that organize reusable module behavior and state.
- **CN**: 定义`_NormBase.__init__` 方法；该代码块引入了用于组织可复用的模块行为与状态的逻辑。

### Lines 66-105
```python
        if self.track_running_stats:
            self.register_buffer(
                "running_mean", torch.zeros(num_features, **factory_kwargs)
            )
            self.register_buffer(
                "running_var", torch.ones(num_features, **factory_kwargs)
            )
            self.running_mean: Tensor | None
            self.running_var: Tensor | None
            self.register_buffer(
                "num_batches_tracked",
                torch.tensor(
                    0,
                    dtype=torch.long,
                    # pyrefly: ignore [bad-argument-type]
                    **{k: v for k, v in factory_kwargs.items() if k != "dtype"},
                ),
            )
            self.num_batches_tracked: Tensor | None
        else:
            self.register_buffer("running_mean", None)
            self.register_buffer("running_var", None)
            self.register_buffer("num_batches_tracked", None)
        self.reset_parameters()

    def reset_running_stats(self) -> None:
        if self.track_running_stats:
            # running_mean/running_var/num_batches... are registered at runtime depending
            # if self.track_running_stats is on
            self.running_mean.zero_()  # type: ignore[union-attr]
            self.running_var.fill_(1)  # type: ignore[union-attr]
            self.num_batches_tracked.zero_()  # type: ignore[union-attr,operator]

    def reset_parameters(self) -> None:
        self.reset_running_stats()
        if self.affine:
            init.ones_(self.weight)
            if self.bias is not None:
                init.zeros_(self.bias)
```
- **EN**: Declares `_NormBase(Module)`; this class packages state and methods that organize reusable module behavior and state.
- **CN**: 声明 `_NormBase(Module)`；该类封装了状态与方法，用于组织可复用的模块行为与状态。

### Lines 106-140
```python
    def _check_input_dim(self, input):
        raise NotImplementedError

    def extra_repr(self):
        return (
            "{num_features}, eps={eps}, momentum={momentum}, affine={affine}, "
            "bias={use_bias}, track_running_stats={track_running_stats}".format(
                **self.__dict__, use_bias=self.bias is not None
            )
        )

    def _load_from_state_dict(
        self,
        state_dict,
        prefix,
        local_metadata,
        strict,
        missing_keys,
        unexpected_keys,
        error_msgs,
    ) -> None:
        version = local_metadata.get("version", None)

        if (version is None or version < 2) and self.track_running_stats:
            # at version 2: added num_batches_tracked buffer
            #               this should have a default value of 0
            num_batches_tracked_key = prefix + "num_batches_tracked"
            if num_batches_tracked_key not in state_dict:
                state_dict[num_batches_tracked_key] = (
                    self.num_batches_tracked
                    if self.num_batches_tracked is not None
                    and self.num_batches_tracked.device != torch.device("meta")
                    else torch.tensor(0, dtype=torch.long)
                )
```
- **EN**: Declares `_NormBase(Module)`; this class packages state and methods that organize reusable module behavior and state.
- **CN**: 声明 `_NormBase(Module)`；该类封装了状态与方法，用于组织可复用的模块行为与状态。

### Lines 141-178
```python
        super()._load_from_state_dict(
            state_dict,
            prefix,
            local_metadata,
            strict,
            missing_keys,
            unexpected_keys,
            error_msgs,
        )


class _BatchNorm(_NormBase):
    def __init__(
        self,
        num_features: int,
        eps: float = 1e-5,
        momentum: float | None = 0.1,
        affine: bool = True,
        track_running_stats: bool = True,
        device=None,
        dtype=None,
        *,
        bias: bool = True,
    ) -> None:
        factory_kwargs = {"device": device, "dtype": dtype}
        super().__init__(
            num_features,
            eps,
            momentum,
            affine,
            track_running_stats,
            **factory_kwargs,
            bias=bias,
        )

    def forward(self, input: Tensor) -> Tensor:
        self._check_input_dim(input)
```
- **EN**: This module-level block helps organize reusable module behavior and state.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。

### Lines 179-206
```python
        # exponential_average_factor is set to self.momentum
        # (when it is available) only so that it gets updated
        # in ONNX graph when this node is exported to ONNX.
        if self.momentum is None:
            exponential_average_factor = 0.0
        else:
            exponential_average_factor = self.momentum

        if self.training and self.track_running_stats:
            # TODO: if statement only here to tell the jit to skip emitting this when it is None
            if self.num_batches_tracked is not None:  # type: ignore[has-type]
                self.num_batches_tracked.add_(1)  # type: ignore[has-type]
                if self.momentum is None:  # use cumulative moving average
                    exponential_average_factor = 1.0 / float(self.num_batches_tracked)
                else:  # use exponential moving average
                    exponential_average_factor = self.momentum

        r"""
        Decide whether the mini-batch stats should be used for normalization rather than the buffers.
        Mini-batch stats are used in training mode, and in eval mode when buffers are None.
        """
        if self.training:
            bn_training = True
        else:
            bn_training = (self.running_mean is None) and (self.running_var is None)

        r"""
        Buffers are only updated if they are to be tracked and we are in training mode. Thus they only need to be
```
- **EN**: This block continues `_BatchNorm.forward` and works to build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `_BatchNorm.forward`，用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理。

### Lines 207-234
```python
        passed when the update should occur (i.e. in training mode when they are tracked), or when buffer stats are
        used for normalization (i.e. in eval mode when buffers are not None).
        """
        return F.batch_norm(
            input,
            # If buffers are not to be tracked, ensure that they won't be updated
            (
                self.running_mean
                if not self.training or self.track_running_stats
                else None
            ),
            self.running_var if not self.training or self.track_running_stats else None,
            self.weight,
            self.bias,
            bn_training,
            exponential_average_factor,
            self.eps,
        )


class _LazyNormBase(LazyModuleMixin, _NormBase):
    weight: UninitializedParameter  # type: ignore[assignment]
    bias: UninitializedParameter  # type: ignore[assignment]

    def __init__(
        self,
        eps=1e-5,
        momentum=0.1,
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 235-274
```python
        affine=True,
        track_running_stats=True,
        device=None,
        dtype=None,
        *,
        bias=True,
    ) -> None:
        factory_kwargs = {"device": device, "dtype": dtype}
        # pyrefly: ignore [bad-argument-type]
        super().__init__(
            # affine, bias and track_running_stats are hardcoded to False to
            # avoid creating tensors that will soon be overwritten.
            0,
            eps,
            momentum,
            False,
            False,
            **factory_kwargs,
            bias=False,
        )
        self.affine = affine
        self.track_running_stats = track_running_stats
        if self.affine:
            # pyrefly: ignore [unexpected-keyword]
            self.weight = UninitializedParameter(**factory_kwargs)
            if bias:
                # pyrefly: ignore  # bad-argument-type
                self.bias = UninitializedParameter(**factory_kwargs)
        if self.track_running_stats:
            # pyrefly: ignore [unexpected-keyword]
            self.running_mean = UninitializedBuffer(**factory_kwargs)
            # pyrefly: ignore [unexpected-keyword]
            self.running_var = UninitializedBuffer(**factory_kwargs)
            self.num_batches_tracked = torch.tensor(
                0,
                dtype=torch.long,
                # pyrefly: ignore [bad-argument-type]
                **{k: v for k, v in factory_kwargs.items() if k != "dtype"},
            )
```
- **EN**: This block continues `_LazyNormBase` and works to organize reusable module behavior and state. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `_LazyNormBase`，用于组织可复用的模块行为与状态。 同时它还会根据运行时条件分支处理。

### Lines 275-316
```python
    def reset_parameters(self) -> None:
        # pyrefly: ignore [bad-argument-type]
        if not self.has_uninitialized_params() and self.num_features != 0:
            super().reset_parameters()

    def initialize_parameters(self, input) -> None:  # type: ignore[override]
        # pyrefly: ignore [bad-argument-type]
        if self.has_uninitialized_params():
            self.num_features = input.shape[1]
            if self.affine:
                if not isinstance(self.weight, UninitializedParameter):
                    raise AssertionError(
                        "self.weight must be an UninitializedParameter"
                    )
                self.weight.materialize((self.num_features,))
                if self.bias is not None:
                    if not isinstance(self.bias, UninitializedParameter):
                        raise AssertionError(
                            "self.bias must be an UninitializedParameter"
                        )
                    self.bias.materialize((self.num_features,))
            if self.track_running_stats:
                self.running_mean.materialize(  # type:ignore[union-attr]
                    (self.num_features,)
                )
                self.running_var.materialize(  # type:ignore[union-attr]
                    (self.num_features,)
                )
            self.reset_parameters()


class BatchNorm1d(_BatchNorm):
    r"""Applies Batch Normalization over a 2D or 3D input.

    Method described in the paper
    `Batch Normalization: Accelerating Deep Network Training by Reducing
    Internal Covariate Shift <https://arxiv.org/abs/1502.03167>`__ .

    .. math::

        y = \frac{x - \mathrm{E}[x]}{\sqrt{\mathrm{Var}[x] + \epsilon}} * \gamma + \beta
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 317-345
```python
    The mean and standard-deviation are calculated per-dimension over
    the mini-batches and :math:`\gamma` and :math:`\beta` are learnable parameter vectors
    of size `C` (where `C` is the number of features or channels of the input). By default, the
    elements of :math:`\gamma` are set to 1 and the elements of :math:`\beta` are set to 0.
    At train time in the forward pass, the variance is calculated via the biased estimator,
    equivalent to ``torch.var(input, correction=0)``. However, the value stored in the
    moving average of the variance is calculated via the unbiased  estimator, equivalent to
    ``torch.var(input, correction=1)``.

    Also by default, during training this layer keeps running estimates of its
    computed mean and variance, which are then used for normalization during
    evaluation. The running estimates are kept with a default :attr:`momentum`
    of 0.1.

    If :attr:`track_running_stats` is set to ``False``, this layer then does not
    keep running estimates, and batch statistics are instead used during
    evaluation time as well.

    .. note::
        This :attr:`momentum` argument is different from one used in optimizer
        classes and the conventional notion of momentum. Mathematically, the
        update rule for running statistics here is
        :math:`\hat{x}_\text{new} = (1 - \text{momentum}) \times \hat{x} + \text{momentum} \times x_t`,
        where :math:`\hat{x}` is the estimated statistic and :math:`x_t` is the
        new observed value.

    Because the Batch Normalization is done over the `C` dimension, computing statistics
    on `(N, L)` slices, it's common terminology to call this Temporal Batch Normalization.
```
- **EN**: This block continues `BatchNorm1d` and works to organize reusable module behavior and state.
- **CN**: 该代码块继续实现 `BatchNorm1d`，用于组织可复用的模块行为与状态。

### Lines 346-386
```python
    Args:
        num_features: number of features or channels :math:`C` of the input
        eps: a value added to the denominator for numerical stability.
            Default: 1e-5
        momentum: the value used for the running_mean and running_var
            computation. Can be set to ``None`` for cumulative moving average
            (i.e. simple average). Default: 0.1
        affine: a boolean value that when set to ``True``, this module has
            learnable affine parameters. Default: ``True``
        track_running_stats: a boolean value that when set to ``True``, this
            module tracks the running mean and variance, and when set to ``False``,
            this module does not track such statistics, and initializes statistics
            buffers :attr:`running_mean` and :attr:`running_var` as ``None``.
            When these buffers are ``None``, this module always uses batch statistics.
            in both training and eval modes. Default: ``True``
        bias: If set to ``False``, the layer will not learn an additive bias (only relevant if
            :attr:`affine` is ``True``). Default: ``True``

    Shape:
        - Input: :math:`(N, C)` or :math:`(N, C, L)`, where :math:`N` is the batch size,
          :math:`C` is the number of features or channels, and :math:`L` is the sequence length
        - Output: :math:`(N, C)` or :math:`(N, C, L)` (same shape as input)

    Examples::

        >>> # With Learnable Parameters
        >>> m = nn.BatchNorm1d(100)
        >>> # Without Learnable Parameters
        >>> m = nn.BatchNorm1d(100, affine=False)
        >>> input = torch.randn(20, 100)
        >>> output = m(input)
    """

    def _check_input_dim(self, input) -> None:
        if input.dim() != 2 and input.dim() != 3:
            raise ValueError(f"expected 2D or 3D input (got {input.dim()}D input)")


class LazyBatchNorm1d(_LazyNormBase, _BatchNorm):
    r"""A :class:`torch.nn.BatchNorm1d` module with lazy initialization.
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 387-427
```python
    Lazy initialization based on the ``num_features`` argument of the :class:`BatchNorm1d` that is inferred
    from the ``input.size(1)``.
    The attributes that will be lazily initialized are `weight`, `bias`,
    `running_mean` and `running_var`.

    Check the :class:`torch.nn.modules.lazy.LazyModuleMixin` for further documentation
    on lazy modules and their limitations.

    Args:
        eps: a value added to the denominator for numerical stability.
            Default: 1e-5
        momentum: the value used for the running_mean and running_var
            computation. Can be set to ``None`` for cumulative moving average
            (i.e. simple average). Default: 0.1
        affine: a boolean value that when set to ``True``, this module has
            learnable affine parameters. Default: ``True``
        track_running_stats: a boolean value that when set to ``True``, this
            module tracks the running mean and variance, and when set to ``False``,
            this module does not track such statistics, and initializes statistics
            buffers :attr:`running_mean` and :attr:`running_var` as ``None``.
            When these buffers are ``None``, this module always uses batch statistics.
            in both training and eval modes. Default: ``True``
        bias: If set to ``False``, the layer will not learn an additive bias (only relevant if
            :attr:`affine` is ``True``). Default: ``True``
    """

    cls_to_become = BatchNorm1d  # type: ignore[assignment]

    def _check_input_dim(self, input) -> None:
        if input.dim() != 2 and input.dim() != 3:
            raise ValueError(f"expected 2D or 3D input (got {input.dim()}D input)")


class BatchNorm2d(_BatchNorm):
    r"""Applies Batch Normalization over a 4D input.

    4D is a mini-batch of 2D inputs
    with additional channel dimension. Method described in the paper
    `Batch Normalization: Accelerating Deep Network Training by Reducing
    Internal Covariate Shift <https://arxiv.org/abs/1502.03167>`__ .
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 428-460
```python
    .. math::

        y = \frac{x - \mathrm{E}[x]}{ \sqrt{\mathrm{Var}[x] + \epsilon}} * \gamma + \beta

    The mean and standard-deviation are calculated per-dimension over
    the mini-batches and :math:`\gamma` and :math:`\beta` are learnable parameter vectors
    of size `C` (where `C` is the input size). By default, the elements of :math:`\gamma` are set
    to 1 and the elements of :math:`\beta` are set to 0. At train time in the forward pass, the
    standard-deviation is calculated via the biased estimator, equivalent to
    ``torch.var(input, correction=0)``. However, the value stored in the moving average of the
    standard-deviation is calculated via the unbiased  estimator, equivalent to
    ``torch.var(input, correction=1)``.

    Also by default, during training this layer keeps running estimates of its
    computed mean and variance, which are then used for normalization during
    evaluation. The running estimates are kept with a default :attr:`momentum`
    of 0.1.

    If :attr:`track_running_stats` is set to ``False``, this layer then does not
    keep running estimates, and batch statistics are instead used during
    evaluation time as well.

    .. note::
        This :attr:`momentum` argument is different from one used in optimizer
        classes and the conventional notion of momentum. Mathematically, the
        update rule for running statistics here is
        :math:`\hat{x}_\text{new} = (1 - \text{momentum}) \times \hat{x} + \text{momentum} \times x_t`,
        where :math:`\hat{x}` is the estimated statistic and :math:`x_t` is the
        new observed value.

    Because the Batch Normalization is done over the `C` dimension, computing statistics
    on `(N, H, W)` slices, it's common terminology to call this Spatial Batch Normalization.
```
- **EN**: This block continues `BatchNorm2d` and works to organize reusable module behavior and state.
- **CN**: 该代码块继续实现 `BatchNorm2d`，用于组织可复用的模块行为与状态。

### Lines 461-501
```python
    Args:
        num_features: :math:`C` from an expected input of size
            :math:`(N, C, H, W)`
        eps: a value added to the denominator for numerical stability.
            Default: 1e-5
        momentum: the value used for the running_mean and running_var
            computation. Can be set to ``None`` for cumulative moving average
            (i.e. simple average). Default: 0.1
        affine: a boolean value that when set to ``True``, this module has
            learnable affine parameters. Default: ``True``
        track_running_stats: a boolean value that when set to ``True``, this
            module tracks the running mean and variance, and when set to ``False``,
            this module does not track such statistics, and initializes statistics
            buffers :attr:`running_mean` and :attr:`running_var` as ``None``.
            When these buffers are ``None``, this module always uses batch statistics.
            in both training and eval modes. Default: ``True``
        bias: If set to ``False``, the layer will not learn an additive bias (only relevant if
            :attr:`affine` is ``True``). Default: ``True``

    Shape:
        - Input: :math:`(N, C, H, W)`
        - Output: :math:`(N, C, H, W)` (same shape as input)

    Examples::

        >>> # With Learnable Parameters
        >>> m = nn.BatchNorm2d(100)
        >>> # Without Learnable Parameters
        >>> m = nn.BatchNorm2d(100, affine=False)
        >>> input = torch.randn(20, 100, 35, 45)
        >>> output = m(input)
    """

    def _check_input_dim(self, input) -> None:
        if input.dim() != 4:
            raise ValueError(f"expected 4D input (got {input.dim()}D input)")


class LazyBatchNorm2d(_LazyNormBase, _BatchNorm):
    r"""A :class:`torch.nn.BatchNorm2d` module with lazy initialization.
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 502-543
```python
    Lazy initialization is done for the ``num_features`` argument of the :class:`BatchNorm2d` that is inferred
    from the ``input.size(1)``.
    The attributes that will be lazily initialized are `weight`, `bias`,
    `running_mean` and `running_var`.

    Check the :class:`torch.nn.modules.lazy.LazyModuleMixin` for further documentation
    on lazy modules and their limitations.

    Args:
        eps: a value added to the denominator for numerical stability.
            Default: 1e-5
        momentum: the value used for the running_mean and running_var
            computation. Can be set to ``None`` for cumulative moving average
            (i.e. simple average). Default: 0.1
        affine: a boolean value that when set to ``True``, this module has
            learnable affine parameters. Default: ``True``
        track_running_stats: a boolean value that when set to ``True``, this
            module tracks the running mean and variance, and when set to ``False``,
            this module does not track such statistics, and initializes statistics
            buffers :attr:`running_mean` and :attr:`running_var` as ``None``.
            When these buffers are ``None``, this module always uses batch statistics.
            in both training and eval modes. Default: ``True``
        bias: If set to ``False``, the layer will not learn an additive bias (only relevant if
            :attr:`affine` is ``True``). Default: ``True``
    """

    cls_to_become = BatchNorm2d  # type: ignore[assignment]

    def _check_input_dim(self, input) -> None:
        if input.dim() != 4:
            raise ValueError(f"expected 4D input (got {input.dim()}D input)")


class BatchNorm3d(_BatchNorm):
    r"""Applies Batch Normalization over a 5D input.

    5D is a mini-batch of 3D inputs with additional channel dimension as described in the paper
    `Batch Normalization: Accelerating Deep Network Training by Reducing
    Internal Covariate Shift <https://arxiv.org/abs/1502.03167>`__ .

    .. math::
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 544-575
```python
        y = \frac{x - \mathrm{E}[x]}{ \sqrt{\mathrm{Var}[x] + \epsilon}} * \gamma + \beta

    The mean and standard-deviation are calculated per-dimension over
    the mini-batches and :math:`\gamma` and :math:`\beta` are learnable parameter vectors
    of size `C` (where `C` is the input size). By default, the elements of :math:`\gamma` are set
    to 1 and the elements of :math:`\beta` are set to 0. At train time in the forward pass, the
    standard-deviation is calculated via the biased estimator, equivalent to
    ``torch.var(input, correction=0)``. However, the value stored in the moving average of the
    standard-deviation is calculated via the unbiased  estimator, equivalent to
    ``torch.var(input, correction=1)``.

    Also by default, during training this layer keeps running estimates of its
    computed mean and variance, which are then used for normalization during
    evaluation. The running estimates are kept with a default :attr:`momentum`
    of 0.1.

    If :attr:`track_running_stats` is set to ``False``, this layer then does not
    keep running estimates, and batch statistics are instead used during
    evaluation time as well.

    .. note::
        This :attr:`momentum` argument is different from one used in optimizer
        classes and the conventional notion of momentum. Mathematically, the
        update rule for running statistics here is
        :math:`\hat{x}_\text{new} = (1 - \text{momentum}) \times \hat{x} + \text{momentum} \times x_t`,
        where :math:`\hat{x}` is the estimated statistic and :math:`x_t` is the
        new observed value.

    Because the Batch Normalization is done over the `C` dimension, computing statistics
    on `(N, D, H, W)` slices, it's common terminology to call this Volumetric Batch Normalization
    or Spatio-temporal Batch Normalization.
```
- **EN**: This block continues `BatchNorm3d` and works to organize reusable module behavior and state.
- **CN**: 该代码块继续实现 `BatchNorm3d`，用于组织可复用的模块行为与状态。

### Lines 576-616
```python
    Args:
        num_features: :math:`C` from an expected input of size
            :math:`(N, C, D, H, W)`
        eps: a value added to the denominator for numerical stability.
            Default: 1e-5
        momentum: the value used for the running_mean and running_var
            computation. Can be set to ``None`` for cumulative moving average
            (i.e. simple average). Default: 0.1
        affine: a boolean value that when set to ``True``, this module has
            learnable affine parameters. Default: ``True``
        track_running_stats: a boolean value that when set to ``True``, this
            module tracks the running mean and variance, and when set to ``False``,
            this module does not track such statistics, and initializes statistics
            buffers :attr:`running_mean` and :attr:`running_var` as ``None``.
            When these buffers are ``None``, this module always uses batch statistics.
            in both training and eval modes. Default: ``True``
        bias: If set to ``False``, the layer will not learn an additive bias (only relevant if
            :attr:`affine` is ``True``). Default: ``True``

    Shape:
        - Input: :math:`(N, C, D, H, W)`
        - Output: :math:`(N, C, D, H, W)` (same shape as input)

    Examples::

        >>> # With Learnable Parameters
        >>> m = nn.BatchNorm3d(100)
        >>> # Without Learnable Parameters
        >>> m = nn.BatchNorm3d(100, affine=False)
        >>> input = torch.randn(20, 100, 35, 45, 10)
        >>> output = m(input)
    """

    def _check_input_dim(self, input) -> None:
        if input.dim() != 5:
            raise ValueError(f"expected 5D input (got {input.dim()}D input)")


class LazyBatchNorm3d(_LazyNormBase, _BatchNorm):
    r"""A :class:`torch.nn.BatchNorm3d` module with lazy initialization.
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 617-658
```python
    Lazy initialization is done for the ``num_features`` argument of the :class:`BatchNorm3d` that is inferred
    from the ``input.size(1)``.
    The attributes that will be lazily initialized are `weight`, `bias`,
    `running_mean` and `running_var`.

    Check the :class:`torch.nn.modules.lazy.LazyModuleMixin` for further documentation
    on lazy modules and their limitations.

    Args:
        eps: a value added to the denominator for numerical stability.
            Default: 1e-5
        momentum: the value used for the running_mean and running_var
            computation. Can be set to ``None`` for cumulative moving average
            (i.e. simple average). Default: 0.1
        affine: a boolean value that when set to ``True``, this module has
            learnable affine parameters. Default: ``True``
        track_running_stats: a boolean value that when set to ``True``, this
            module tracks the running mean and variance, and when set to ``False``,
            this module does not track such statistics, and initializes statistics
            buffers :attr:`running_mean` and :attr:`running_var` as ``None``.
            When these buffers are ``None``, this module always uses batch statistics.
            in both training and eval modes. Default: ``True``
        bias: If set to ``False``, the layer will not learn an additive bias (only relevant if
            :attr:`affine` is ``True``). Default: ``True``
    """

    cls_to_become = BatchNorm3d  # type: ignore[assignment]

    def _check_input_dim(self, input) -> None:
        if input.dim() != 5:
            raise ValueError(f"expected 5D input (got {input.dim()}D input)")


class SyncBatchNorm(_BatchNorm):
    r"""Applies Batch Normalization over a N-Dimensional input.

    The N-D input is a mini-batch of [N-2]D inputs with additional channel dimension) as described in the paper
    `Batch Normalization: Accelerating Deep Network Training by Reducing
    Internal Covariate Shift <https://arxiv.org/abs/1502.03167>`__ .

    .. math::
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 659-695
```python
        y = \frac{x - \mathrm{E}[x]}{ \sqrt{\mathrm{Var}[x] + \epsilon}} * \gamma + \beta

    The mean and standard-deviation are calculated per-dimension over all
    mini-batches of the same process groups. :math:`\gamma` and :math:`\beta`
    are learnable parameter vectors of size `C` (where `C` is the input size).
    By default, the elements of :math:`\gamma` are sampled from
    :math:`\mathcal{U}(0, 1)` and the elements of :math:`\beta` are set to 0.
    The standard-deviation is calculated via the biased estimator, equivalent to
    `torch.var(input, correction=0)`.

    Also by default, during training this layer keeps running estimates of its
    computed mean and variance, which are then used for normalization during
    evaluation. The running estimates are kept with a default :attr:`momentum`
    of 0.1.

    If :attr:`track_running_stats` is set to ``False``, this layer then does not
    keep running estimates, and batch statistics are instead used during
    evaluation time as well.

    .. note::
        This :attr:`momentum` argument is different from one used in optimizer
        classes and the conventional notion of momentum. Mathematically, the
        update rule for running statistics here is
        :math:`\hat{x}_\text{new} = (1 - \text{momentum}) \times \hat{x} + \text{momentum} \times x_t`,
        where :math:`\hat{x}` is the estimated statistic and :math:`x_t` is the
        new observed value.

    Because the Batch Normalization is done for each channel in the ``C`` dimension, computing
    statistics on ``(N, +)`` slices, it's common terminology to call this Volumetric Batch
    Normalization or Spatio-temporal Batch Normalization.

    Currently :class:`SyncBatchNorm` only supports
    :class:`~torch.nn.DistributedDataParallel` (DDP) with single GPU per process. Use
    :meth:`torch.nn.SyncBatchNorm.convert_sync_batchnorm()` to convert
    :attr:`BatchNorm*D` layer to :class:`SyncBatchNorm` before wrapping
    Network with DDP.
```
- **EN**: This block continues `SyncBatchNorm` and works to organize reusable module behavior and state.
- **CN**: 该代码块继续实现 `SyncBatchNorm`，用于组织可复用的模块行为与状态。

### Lines 696-728
```python
    Args:
        num_features: :math:`C` from an expected input of size
            :math:`(N, C, +)`
        eps: a value added to the denominator for numerical stability.
            Default: ``1e-5``
        momentum: the value used for the running_mean and running_var
            computation. Can be set to ``None`` for cumulative moving average
            (i.e. simple average). Default: 0.1
        affine: a boolean value that when set to ``True``, this module has
            learnable affine parameters. Default: ``True``
        track_running_stats: a boolean value that when set to ``True``, this
            module tracks the running mean and variance, and when set to ``False``,
            this module does not track such statistics, and initializes statistics
            buffers :attr:`running_mean` and :attr:`running_var` as ``None``.
            When these buffers are ``None``, this module always uses batch statistics.
            in both training and eval modes. Default: ``True``
        process_group: synchronization of stats happen within each process group
            individually. Default behavior is synchronization across the whole
            world
        bias: If set to ``False``, the layer will not learn an additive bias (only relevant if
            :attr:`affine` is ``True``). Default: ``True``

    Shape:
        - Input: :math:`(N, C, +)`
        - Output: :math:`(N, C, +)` (same shape as input)

    .. note::
        Synchronization of batchnorm statistics occurs only while training, i.e.
        synchronization is disabled when ``model.eval()`` is set or if
        ``self.training`` is otherwise ``False``.

    Examples::
```
- **EN**: This block continues `SyncBatchNorm` and works to track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 该代码块继续实现 `SyncBatchNorm`，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 729-756
```python
        >>> # xdoctest: +SKIP
        >>> # With Learnable Parameters
        >>> m = nn.SyncBatchNorm(100)
        >>> # creating process group (optional)
        >>> # ranks is a list of int identifying rank ids.
        >>> ranks = list(range(8))
        >>> r1, r2 = ranks[:4], ranks[4:]
        >>> # Note: every rank calls into new_group for every
        >>> # process group created, even if that rank is not
        >>> # part of the group.
        >>> process_groups = [torch.distributed.new_group(pids) for pids in [r1, r2]]
        >>> process_group = process_groups[0 if dist.get_rank() <= 3 else 1]
        >>> # Without Learnable Parameters
        >>> m = nn.BatchNorm3d(100, affine=False, process_group=process_group)
        >>> input = torch.randn(20, 100, 35, 45, 10)
        >>> output = m(input)

        >>> # network is nn.BatchNorm layer
        >>> sync_bn_network = nn.SyncBatchNorm.convert_sync_batchnorm(network, process_group)
        >>> # only single gpu per process is currently supported
        >>> ddp_sync_bn_network = torch.nn.parallel.DistributedDataParallel(
        >>>                         sync_bn_network,
        >>>                         device_ids=[args.local_rank],
        >>>                         output_device=args.local_rank)
    """

    def __init__(
        self,
```
- **EN**: Declares `SyncBatchNorm(_BatchNorm)`; this class packages state and methods that organize reusable module behavior and state.
- **CN**: 声明 `SyncBatchNorm(_BatchNorm)`；该类封装了状态与方法，用于组织可复用的模块行为与状态。

### Lines 757-796
```python
        num_features: int,
        eps: float = 1e-5,
        momentum: float | None = 0.1,
        affine: bool = True,
        track_running_stats: bool = True,
        process_group: Any | None = None,
        device=None,
        dtype=None,
        *,
        bias: bool = True,
    ) -> None:
        factory_kwargs = {"device": device, "dtype": dtype}
        super().__init__(
            num_features,
            eps,
            momentum,
            affine,
            track_running_stats,
            **factory_kwargs,
            bias=bias,
        )
        self.process_group = process_group

    def _check_input_dim(self, input) -> None:
        if input.dim() < 2:
            raise ValueError(f"expected at least 2D input (got {input.dim()}D input)")

    def _check_non_zero_input_channels(self, input) -> None:
        if input.size(1) == 0:
            raise ValueError(
                "SyncBatchNorm number of input channels should be non-zero"
            )

    def forward(self, input: Tensor) -> Tensor:
        """
        Runs the forward pass.
        """
        self._check_input_dim(input)
        self._check_non_zero_input_channels(input)
```
- **EN**: Declares `SyncBatchNorm(_BatchNorm)`; this class packages state and methods that organize reusable module behavior and state.
- **CN**: 声明 `SyncBatchNorm(_BatchNorm)`；该类封装了状态与方法，用于组织可复用的模块行为与状态。

### Lines 797-835
```python
        # exponential_average_factor is set to self.momentum
        # (when it is available) only so that it gets updated
        # in ONNX graph when this node is exported to ONNX.
        if self.momentum is None:
            exponential_average_factor = 0.0
        else:
            exponential_average_factor = self.momentum

        if self.training and self.track_running_stats:
            if self.num_batches_tracked is None:
                raise AssertionError("num_batches_tracked must not be None")
            self.num_batches_tracked.add_(1)
            if self.momentum is None:  # use cumulative moving average
                exponential_average_factor = 1.0 / self.num_batches_tracked.item()
            else:  # use exponential moving average
                exponential_average_factor = self.momentum

        r"""
        Decide whether the mini-batch stats should be used for normalization rather than the buffers.
        Mini-batch stats are used in training mode, and in eval mode when buffers are None.
        """
        if self.training:
            bn_training = True
        else:
            bn_training = (self.running_mean is None) and (self.running_var is None)

        r"""
        Buffers are only updated if they are to be tracked and we are in training mode. Thus they only need to be
        passed when the update should occur (i.e. in training mode when they are tracked), or when buffer stats are
        used for normalization (i.e. in eval mode when buffers are not None).
        """
        # If buffers are not to be tracked, ensure that they won't be updated
        running_mean = (
            self.running_mean if not self.training or self.track_running_stats else None
        )
        running_var = (
            self.running_var if not self.training or self.track_running_stats else None
        )
```
- **EN**: This block continues `SyncBatchNorm.forward` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `SyncBatchNorm.forward`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 836-863
```python
        # Don't sync batchnorm stats in inference mode (model.eval()).
        need_sync = (
            bn_training
            and self.training
            and torch.distributed.is_available()
            and torch.distributed.is_initialized()
        )
        if need_sync:
            # currently only GPU/PrivateUse1 input is supported
            if input.device.type not in [
                "cuda",
                "hpu",
                "xpu",
                torch._C._get_privateuse1_backend_name(),
            ]:
                raise ValueError(
                    "SyncBatchNorm expected input tensor to be on GPU or XPU or "
                    f"{torch._C._get_privateuse1_backend_name()}"
                )

            process_group = torch.distributed.group.WORLD
            if self.process_group:
                process_group = self.process_group
            world_size = torch.distributed.get_world_size(process_group)
            need_sync = world_size > 1

        # fallback to framework BN when synchronization is not necessary
        if not need_sync:
```
- **EN**: This block continues `SyncBatchNorm.forward` and works to hand work to a compiler/backend pipeline. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `SyncBatchNorm.forward`，用于将工作移交给编译器或后端流水线。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 864-905
```python
            return F.batch_norm(
                input,
                running_mean,
                running_var,
                self.weight,
                self.bias,
                bn_training,
                exponential_average_factor,
                self.eps,
            )
        else:
            if not bn_training:
                raise AssertionError("bn_training must be True")
            return sync_batch_norm.apply(
                input,
                self.weight,
                self.bias,
                running_mean,
                running_var,
                self.eps,
                exponential_average_factor,
                process_group,  # type: ignore[possibly-undefined]
                world_size,  # type: ignore[possibly-undefined]
            )

    @classmethod
    def convert_sync_batchnorm(cls, module, process_group=None):
        r"""Converts all :attr:`BatchNorm*D` layers in the model to :class:`torch.nn.SyncBatchNorm` layers.

        Args:
            module (nn.Module): module containing one or more :attr:`BatchNorm*D` layers
            process_group (optional): process group to scope synchronization,
                default is the whole world

        Returns:
            The original :attr:`module` with the converted :class:`torch.nn.SyncBatchNorm`
            layers. If the original :attr:`module` is a :attr:`BatchNorm*D` layer,
            a new :class:`torch.nn.SyncBatchNorm` layer object will be returned
            instead.

        Example::
```
- **EN**: These decorators register or transform the following definition so it can organize reusable module behavior and state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够组织可复用的模块行为与状态。

### Lines 906-933
```python
            >>> # Network with nn.BatchNorm layer
            >>> # xdoctest: +REQUIRES(env:TORCH_DOCTEST_CUDA)
            >>> module = torch.nn.Sequential(
            >>>            torch.nn.Linear(20, 100),
            >>>            torch.nn.BatchNorm1d(100),
            >>>          ).cuda()
            >>> # creating process group (optional)
            >>> # ranks is a list of int identifying rank ids.
            >>> ranks = list(range(8))
            >>> r1, r2 = ranks[:4], ranks[4:]
            >>> # Note: every rank calls into new_group for every
            >>> # process group created, even if that rank is not
            >>> # part of the group.
            >>> # xdoctest: +SKIP("distributed")
            >>> process_groups = [torch.distributed.new_group(pids) for pids in [r1, r2]]
            >>> process_group = process_groups[0 if dist.get_rank() <= 3 else 1]
            >>> sync_bn_module = torch.nn.SyncBatchNorm.convert_sync_batchnorm(module, process_group)

        """
        module_output = module
        if isinstance(module, torch.nn.modules.batchnorm._BatchNorm):
            module_output = torch.nn.SyncBatchNorm(
                module.num_features,
                module.eps,
                module.momentum,
                module.affine,
                module.track_running_stats,
                process_group,
```
- **EN**: This block continues `SyncBatchNorm.convert_sync_batchnorm` and works to organize reusable module behavior and state. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `SyncBatchNorm.convert_sync_batchnorm`，用于组织可复用的模块行为与状态。 同时它还会根据运行时条件分支处理。

### Lines 934-951
```python
                bias=module.bias is not None,
            )
            if module.affine:
                with torch.no_grad():
                    module_output.weight = module.weight
                    module_output.bias = module.bias
            module_output.running_mean = module.running_mean
            module_output.running_var = module.running_var
            module_output.num_batches_tracked = module.num_batches_tracked
            module_output.training = module.training
            if hasattr(module, "qconfig"):
                module_output.qconfig = module.qconfig
        for name, child in module.named_children():
            module_output.add_module(
                name, cls.convert_sync_batchnorm(child, process_group)
            )
        del module
        return module_output
```
- **EN**: This block continues `SyncBatchNorm.convert_sync_batchnorm` and works to organize reusable module behavior and state. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `SyncBatchNorm.convert_sync_batchnorm`，用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

## Key Concepts / 关键概念

- **Neural-network abstractions / 神经网络抽象**
  - EN: The file contributes to reusable neural-network layers, functionals, or support utilities.
  - CN: 该文件为可复用的神经网络层、函数式接口或辅助工具提供实现。
- **Symbolic shapes / 符号形状**
  - EN: The file reasons about tensor extents symbolically instead of requiring concrete integers everywhere.
  - CN: 该文件以符号方式推理张量形状，而不是处处要求具体整数。
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
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.nn`, `torch.nn.parameter`, `._functions`, `.lazy`, `.module`
- **Standard library / 标准库**: `typing`
- **Primary symbols / 核心符号**: `__all__`, `_NormBase`, `_BatchNorm`, `_LazyNormBase`, `BatchNorm1d`, `LazyBatchNorm1d`, `BatchNorm2d`, `LazyBatchNorm2d`, `BatchNorm3d`, `LazyBatchNorm3d`, `SyncBatchNorm`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。
