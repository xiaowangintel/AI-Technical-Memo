# linear.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/modules/linear.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Defines reusable nn.Module classes, their state, and their forward-path helpers.
- **Purpose (CN)**: 定义可复用的 nn.Module 类、其状态以及前向路径辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24
```python
# mypy: allow-untyped-defs
import math
from typing import Any

import torch
from torch import Tensor
from torch.nn import functional as F, init
from torch.nn.parameter import Parameter, UninitializedParameter

from .lazy import LazyModuleMixin
from .module import Module


__all__ = [
    "Bilinear",
    "Identity",
    "LazyLinear",
    "Linear",
]


class Identity(Module):
    r"""A placeholder identity operator that is argument-insensitive.
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 25-51
```python
    Args:
        args: any argument (unused)
        kwargs: any keyword argument (unused)

    Shape:
        - Input: :math:`(*)`, where :math:`*` means any number of dimensions.
        - Output: :math:`(*)`, same shape as the input.

    Examples::

        >>> m = nn.Identity(54, unused_argument1=0.1, unused_argument2=False)
        >>> input = torch.randn(128, 20)
        >>> output = m(input)
        >>> print(output.size())
        torch.Size([128, 20])

    """

    def __init__(self, *args: Any, **kwargs: Any) -> None:
        super().__init__()

    def forward(self, input: Tensor) -> Tensor:
        """
        Runs the forward pass.
        """
        return input
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果。

### Lines 52-71
```python

class Linear(Module):
    r"""Applies an affine linear transformation to the incoming data: :math:`y = xA^T + b`.

    This module supports :ref:`TensorFloat32<tf32_on_ampere>`.

    On certain ROCm devices, when using float16 inputs this module will use :ref:`different precision<fp16_on_mi200>` for backward.

    Args:
        in_features: size of each input sample
        out_features: size of each output sample
        bias: If set to ``False``, the layer will not learn an additive bias.
            Default: ``True``

    Shape:
        - Input: :math:`(*, H_\text{in})` where :math:`*` means any number of
          dimensions including none and :math:`H_\text{in} = \text{in\_features}`.
        - Output: :math:`(*, H_\text{out})` where all but the last dimension
          are the same shape as the input and :math:`H_\text{out} = \text{out\_features}`.
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。

### Lines 72-95
```python
    Attributes:
        weight: the learnable weights of the module of shape
            :math:`(\text{out\_features}, \text{in\_features})`. The values are
            initialized from :math:`\mathcal{U}(-\sqrt{k}, \sqrt{k})`, where
            :math:`k = \frac{1}{\text{in\_features}}`
        bias:   the learnable bias of the module of shape :math:`(\text{out\_features})`.
                If :attr:`bias` is ``True``, the values are initialized from
                :math:`\mathcal{U}(-\sqrt{k}, \sqrt{k})` where
                :math:`k = \frac{1}{\text{in\_features}}`

    Examples::

        >>> m = nn.Linear(20, 30)
        >>> input = torch.randn(128, 20)
        >>> output = m(input)
        >>> print(output.size())
        torch.Size([128, 30])
    """

    __constants__ = ["in_features", "out_features"]
    in_features: int
    out_features: int
    weight: Tensor
```
- **EN**: This block continues `Linear` and works to track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 该代码块继续实现 `Linear`，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 96-116
```python
    def __init__(
        self,
        in_features: int,
        out_features: int,
        bias: bool = True,
        device=None,
        dtype=None,
    ) -> None:
        factory_kwargs = {"device": device, "dtype": dtype}
        super().__init__()
        self.in_features = in_features
        self.out_features = out_features
        self.weight = Parameter(
            torch.empty((out_features, in_features), **factory_kwargs)
        )
        if bias:
            self.bias = Parameter(torch.empty(out_features, **factory_kwargs))
        else:
            self.register_parameter("bias", None)
        self.reset_parameters()
```
- **EN**: Declares `Linear(Module)`; this class packages state and methods that organize reusable module behavior and state.
- **CN**: 声明 `Linear(Module)`；该类封装了状态与方法，用于组织可复用的模块行为与状态。

### Lines 117-142
```python
    def reset_parameters(self) -> None:
        """
        Resets parameters based on their initialization used in ``__init__``.
        """
        # Setting a=sqrt(5) in kaiming_uniform is the same as initializing with
        # uniform(-1/sqrt(in_features), 1/sqrt(in_features)). For details, see
        # https://github.com/pytorch/pytorch/issues/57109
        init.kaiming_uniform_(self.weight, a=math.sqrt(5))
        if self.bias is not None:
            fan_in, _ = init._calculate_fan_in_and_fan_out(self.weight)
            bound = 1 / math.sqrt(fan_in) if fan_in > 0 else 0
            init.uniform_(self.bias, -bound, bound)

    def forward(self, input: Tensor) -> Tensor:
        """
        Runs the forward pass.
        """
        return F.linear(input, self.weight, self.bias)

    def extra_repr(self) -> str:
        """
        Return the extra representation of the module.
        """
        return f"in_features={self.in_features}, out_features={self.out_features}, bias={self.bias is not None}"
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 143-164
```python
# This class exists solely to avoid triggering an obscure error when scripting
# an improperly quantized attention layer. See this issue for details:
# https://github.com/pytorch/pytorch/issues/58969
# TODO: fail fast on quantization API usage error, then remove this class
# and replace uses of it with plain Linear
class NonDynamicallyQuantizableLinear(Linear):
    def __init__(
        self,
        in_features: int,
        out_features: int,
        bias: bool = True,
        device=None,
        dtype=None,
    ) -> None:
        super().__init__(
            in_features, out_features, bias=bias, device=device, dtype=dtype
        )


class Bilinear(Module):
    r"""Applies a bilinear transformation to the incoming data: :math:`y = x_1^T A x_2 + b`.
```
- **EN**: This module-level block helps organize reusable module behavior and state.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。

### Lines 165-191
```python
    Args:
        in1_features: size of each first input sample, must be > 0
        in2_features: size of each second input sample, must be > 0
        out_features: size of each output sample, must be > 0
        bias: If set to ``False``, the layer will not learn an additive bias.
            Default: ``True``

    Shape:
        - Input1: :math:`(*, H_\text{in1})` where :math:`H_\text{in1}=\text{in1\_features}` and
          :math:`*` means any number of additional dimensions including none. All but the last dimension
          of the inputs should be the same.
        - Input2: :math:`(*, H_\text{in2})` where :math:`H_\text{in2}=\text{in2\_features}`.
        - Output: :math:`(*, H_\text{out})` where :math:`H_\text{out}=\text{out\_features}`
          and all but the last dimension are the same shape as the input.

    Attributes:
        weight: the learnable weights of the module of shape
            :math:`(\text{out\_features}, \text{in1\_features}, \text{in2\_features})`.
            The values are initialized from :math:`\mathcal{U}(-\sqrt{k}, \sqrt{k})`, where
            :math:`k = \frac{1}{\text{in1\_features}}`
        bias:   the learnable bias of the module of shape :math:`(\text{out\_features})`.
                If :attr:`bias` is ``True``, the values are initialized from
                :math:`\mathcal{U}(-\sqrt{k}, \sqrt{k})`, where
                :math:`k = \frac{1}{\text{in1\_features}}`

    Examples::
```
- **EN**: This block continues `Bilinear` and works to track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 该代码块继续实现 `Bilinear`，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 192-209
```python
        >>> m = nn.Bilinear(20, 30, 40)
        >>> input1 = torch.randn(128, 20)
        >>> input2 = torch.randn(128, 30)
        >>> output = m(input1, input2)
        >>> print(output.size())
        torch.Size([128, 40])
    """

    __constants__ = ["in1_features", "in2_features", "out_features"]
    in1_features: int
    in2_features: int
    out_features: int
    weight: Tensor

    def __init__(
        self,
        in1_features: int,
        in2_features: int,
```
- **EN**: Declares `Bilinear(Module)`; this class packages state and methods that initialize learnable tensors and related state.
- **CN**: 声明 `Bilinear(Module)`；该类封装了状态与方法，用于初始化可学习张量及相关状态。

### Lines 210-229
```python
        out_features: int,
        bias: bool = True,
        device=None,
        dtype=None,
    ) -> None:
        factory_kwargs = {"device": device, "dtype": dtype}
        super().__init__()
        self.in1_features = in1_features
        self.in2_features = in2_features
        self.out_features = out_features
        self.weight = Parameter(
            torch.empty((out_features, in1_features, in2_features), **factory_kwargs)
        )

        if bias:
            self.bias = Parameter(torch.empty(out_features, **factory_kwargs))
        else:
            self.register_parameter("bias", None)
        self.reset_parameters()
```
- **EN**: This block continues `Bilinear` and works to organize reusable module behavior and state. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `Bilinear`，用于组织可复用的模块行为与状态。 同时它还会根据运行时条件分支处理。

### Lines 230-248
```python
    def reset_parameters(self) -> None:
        """
        Resets parameters based on their initialization used in ``__init__``.
        """
        if self.in1_features <= 0:
            raise ValueError(
                f"in1_features must be > 0, but got (in1_features={self.in1_features})"
            )
        bound = 1 / math.sqrt(self.weight.size(1))
        init.uniform_(self.weight, -bound, bound)
        if self.bias is not None:
            init.uniform_(self.bias, -bound, bound)

    def forward(self, input1: Tensor, input2: Tensor) -> Tensor:
        """
        Runs the forward pass.
        """
        return F.bilinear(input1, input2, self.weight, self.bias)
```
- **EN**: Declares `Bilinear(Module)`; this class packages state and methods that organize reusable module behavior and state.
- **CN**: 声明 `Bilinear(Module)`；该类封装了状态与方法，用于组织可复用的模块行为与状态。

### Lines 249-274
```python
    def extra_repr(self) -> str:
        """
        Return the extra representation of the module.
        """
        return (
            f"in1_features={self.in1_features}, in2_features={self.in2_features}, "
            f"out_features={self.out_features}, bias={self.bias is not None}"
        )


class LazyLinear(LazyModuleMixin, Linear):
    r"""A :class:`torch.nn.Linear` module where `in_features` is inferred.

    In this module, the `weight` and `bias` are of :class:`torch.nn.UninitializedParameter`
    class. They will be initialized after the first call to ``forward`` is done and the
    module will become a regular :class:`torch.nn.Linear` module. The ``in_features`` argument
    of the :class:`Linear` is inferred from the ``input.shape[-1]``.

    Check the :class:`torch.nn.modules.lazy.LazyModuleMixin` for further documentation
    on lazy modules and their limitations.

    Args:
        out_features: size of each output sample
        bias: If set to ``False``, the layer will not learn an additive bias.
            Default: ``True``
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果。

### Lines 275-292
```python
    Attributes:
        weight: the learnable weights of the module of shape
            :math:`(\text{out\_features}, \text{in\_features})`. The values are
            initialized from :math:`\mathcal{U}(-\sqrt{k}, \sqrt{k})`, where
            :math:`k = \frac{1}{\text{in\_features}}`
        bias:   the learnable bias of the module of shape :math:`(\text{out\_features})`.
                If :attr:`bias` is ``True``, the values are initialized from
                :math:`\mathcal{U}(-\sqrt{k}, \sqrt{k})` where
                :math:`k = \frac{1}{\text{in\_features}}`


    """

    cls_to_become = Linear  # type: ignore[assignment]
    # pyrefly: ignore [bad-override]
    weight: UninitializedParameter
    bias: UninitializedParameter  # type: ignore[assignment]
```
- **EN**: This block continues `LazyLinear` and works to track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 该代码块继续实现 `LazyLinear`，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 293-315
```python
    def __init__(
        self, out_features: int, bias: bool = True, device=None, dtype=None
    ) -> None:
        factory_kwargs = {"device": device, "dtype": dtype}
        # bias is hardcoded to False to avoid creating tensor
        # that will soon be overwritten.
        # pyrefly: ignore [bad-argument-type]
        super().__init__(0, 0, False)
        # pyrefly: ignore [unexpected-keyword]
        self.weight = UninitializedParameter(**factory_kwargs)
        self.out_features = out_features
        if bias:
            # pyrefly: ignore [unexpected-keyword]
            self.bias = UninitializedParameter(**factory_kwargs)

    def reset_parameters(self) -> None:
        """
        Resets parameters based on their initialization used in ``__init__``.
        """
        # pyrefly: ignore [bad-argument-type]
        if not self.has_uninitialized_params() and self.in_features != 0:
            super().reset_parameters()
```
- **EN**: Declares `LazyLinear(LazyModuleMixin, Linear)`; this class packages state and methods that organize reusable module behavior and state.
- **CN**: 声明 `LazyLinear(LazyModuleMixin, Linear)`；该类封装了状态与方法，用于组织可复用的模块行为与状态。

### Lines 316-337
```python
    def initialize_parameters(self, input) -> None:  # type: ignore[override]
        """
        Infers ``in_features`` based on ``input`` and initializes parameters.
        """
        # pyrefly: ignore [bad-argument-type]
        if self.has_uninitialized_params():
            with torch.no_grad():
                self.in_features = input.shape[-1]
                self.weight.materialize((self.out_features, self.in_features))
                if self.bias is not None:
                    self.bias.materialize((self.out_features,))
                self.reset_parameters()
        if self.in_features == 0:
            if input.shape[-1] != self.weight.shape[-1]:
                raise AssertionError(
                    f"The in_features inferred from input: {input.shape[-1]} "
                    f"is not equal to in_features from self.weight: "
                    f"{self.weight.shape[-1]}"
                )
            self.in_features = input.shape[-1]
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also validates assumptions before proceeding; branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会在继续前校验关键假设；根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 338-338
```python
# TODO: PartialLinear - maybe in sparse?
```
- **EN**: This comment block records intent, caveats, or maintainers’ notes for the code that follows.
- **CN**: 这一注释块记录了后续代码的设计意图、注意事项或维护者说明。

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
- **Attention mechanics / 注意力机制**
  - EN: The logic specializes in attention-style data movement, masking, or projection patterns.
  - CN: 该逻辑专门处理注意力风格的数据搬运、掩码或投影模式。
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
- **Program transforms / 程序变换**
  - EN: The implementation rewrites existing program structure into a more useful form.
  - CN: 该实现会把现有程序结构改写成更有用的形式。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.nn`, `torch.nn.parameter`, `.lazy`, `.module`
- **Standard library / 标准库**: `math`, `typing`
- **Primary symbols / 核心符号**: `__all__`, `Identity`, `Linear`, `NonDynamicallyQuantizableLinear`, `Bilinear`, `LazyLinear`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。
