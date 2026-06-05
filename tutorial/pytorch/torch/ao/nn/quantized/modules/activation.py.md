# activation.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/nn/quantized/modules/activation.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements quantization-aware neural-network module wrappers, fused building blocks, and reference/quantized module variants. This specific file centers on `activation.py`. The file is especially concerned with quantization metadata, calibration, or fake-quant behavior.
- **Purpose (CN) / 用途（中文）**: 实现面向量化的神经网络模块包装、融合构件以及参考/量化模块变体。 该文件具体围绕 `activation.py` 展开。 该文件尤其关注量化元数据、校准流程或 fake-quant 行为。
## Line-by-Line Analysis / 逐行分析

### Lines 1-23 / 第 1-23 行
```python
# mypy: allow-untyped-defs
from warnings import warn

import torch


__all__ = [
    "ReLU6",
    "Hardswish",
    "ELU",
    "LeakyReLU",
    "Sigmoid",
    "Softmax",
    "MultiheadAttention",
    "PReLU",
]


class ReLU6(torch.nn.ReLU):
    r"""Applies the element-wise function:

    :math:`\text{ReLU6}(x) = \min(\max(x_0, x), q(6))`, where :math:`x_0` is the
    zero_point, and :math:`q(6)` is the quantized representation of number 6.
```
- **EN**: The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export. It introduces or extends class-level abstractions such as `ReLU6`, which organize state and behavior for this subsystem. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。 它引入或扩展了 `ReLU6` 等类级抽象，用于组织该子系统的状态与行为。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 25-46 / 第 25-46 行
```python
    Args:
        inplace: can optionally do the operation in-place. Default: ``False``

    Shape:
        - Input: :math:`(N, *)` where `*` means, any number of additional
          dimensions
        - Output: :math:`(N, *)`, same shape as the input

    .. image:: ../scripts/activation_images/ReLU6.png

    Examples::

        >>> m = nn.quantized.ReLU6()
        >>> input = torch.randn(2)
        >>> # xdoctest: +SKIP
        >>> input = torch.quantize_per_tensor(input, 1.0, 0, dtype=torch.qint32)
        >>> output = m(input)
    """

    def __init__(self, inplace=False):
        super().__init__(inplace)
        self.inplace = inplace
```
- **EN**: It introduces or extends class-level abstractions such as `ReLU6`, which organize state and behavior for this subsystem. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `ReLU6` 等类级抽象，用于组织该子系统的状态与行为。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 48-65 / 第 48-65 行
```python
    def forward(self, input):
        return torch.ops.quantized.relu6(input, self.inplace)

    def _get_name(self):
        return "QuantizedReLU6"

    @staticmethod
    def from_float(mod, use_precomputed_fake_quant=False):
        return ReLU6(mod.inplace)


class Hardswish(torch.nn.Hardswish):
    r"""This is the quantized version of :class:`~torch.nn.Hardswish`.

    Args:
        scale: quantization scale of the output tensor
        zero_point: quantization zero point of the output tensor
    """
```
- **EN**: It introduces or extends class-level abstractions such as `ReLU6`, `Hardswish`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `ReLU6`, `Hardswish` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 67-86 / 第 67-86 行
```python
    def __init__(self, scale, zero_point, device=None, dtype=None):
        factory_kwargs = {"device": device, "dtype": dtype}
        super().__init__()
        self.register_buffer("scale", torch.tensor(scale, **factory_kwargs))
        self.register_buffer("zero_point", torch.tensor(zero_point, **factory_kwargs))

    def forward(self, input):
        return torch.ops.quantized.hardswish(input, self.scale, self.zero_point)

    def _get_name(self):
        return "QuantizedHardswish"

    @staticmethod
    def from_float(mod, use_precomputed_fake_quant=False):
        scale, zero_point = mod.activation_post_process.calculate_qparams()
        return Hardswish(float(scale), int(zero_point))

    @classmethod
    def from_reference(cls, mod, scale, zero_point):
        return cls(float(scale), int(zero_point))
```
- **EN**: It introduces or extends class-level abstractions such as `Hardswish`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Hardswish` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 89-109 / 第 89-109 行
```python
class ELU(torch.nn.ELU):
    r"""This is the quantized equivalent of :class:`~torch.nn.ELU`.

    Args:
        scale: quantization scale of the output tensor
        zero_point: quantization zero point of the output tensor
        alpha: the alpha constant
    """

    def __init__(self, scale, zero_point, alpha=1.0):
        super().__init__(alpha)
        self.scale = scale
        self.zero_point = zero_point

    def forward(self, input):
        return torch.ao.nn.quantized.functional.elu(
            input, self.scale, self.zero_point, self.alpha
        )

    def _get_name(self):
        return "QuantizedELU"
```
- **EN**: It introduces or extends class-level abstractions such as `ELU`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `ELU` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 111-128 / 第 111-128 行
```python
    @staticmethod
    def from_float(mod, use_precomputed_fake_quant=False):
        scale, zero_point = mod.activation_post_process.calculate_qparams()
        return ELU(float(scale), int(zero_point), mod.alpha)

    @classmethod
    def from_reference(cls, mod, scale, zero_point):
        return cls(float(scale), int(zero_point), mod.alpha)


class LeakyReLU(torch.nn.LeakyReLU):
    r"""This is the quantized equivalent of :class:`~torch.nn.LeakyReLU`.

    Args:
        scale: quantization scale of the output tensor
        zero_point: quantization zero point of the output tensor
        negative_slope: Controls the angle of the negative slope. Default: 1e-2
    """
```
- **EN**: It introduces or extends class-level abstractions such as `ELU`, `LeakyReLU`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `ELU`, `LeakyReLU` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 130-150 / 第 130-150 行
```python
    def __init__(
        self,
        scale: float,
        zero_point: int,
        negative_slope: float = 1e-2,
        inplace: bool = False,
        device=None,
        dtype=None,
    ) -> None:
        factory_kwargs = {"device": device, "dtype": dtype}
        super().__init__(negative_slope, inplace)
        self.register_buffer("scale", torch.tensor(scale, **factory_kwargs))
        self.register_buffer("zero_point", torch.tensor(zero_point, **factory_kwargs))

    def forward(self, input):
        return torch.ops.quantized.leaky_relu(
            input, self.negative_slope, self.inplace, self.scale, self.zero_point
        )

    def _get_name(self):
        return "QuantizedLeakyReLU"
```
- **EN**: It introduces or extends class-level abstractions such as `LeakyReLU`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LeakyReLU` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 152-173 / 第 152-173 行
```python
    @classmethod
    def from_float(cls, mod, use_precomputed_fake_quant=False):
        scale, zero_point = mod.activation_post_process.calculate_qparams()
        return cls(float(scale), int(zero_point), mod.negative_slope, mod.inplace)

    @classmethod
    def from_reference(cls, mod, scale, zero_point):
        return cls(float(scale), int(zero_point), mod.negative_slope, mod.inplace)


class Sigmoid(torch.nn.Sigmoid):
    r"""This is the quantized equivalent of :class:`~torch.nn.Sigmoid`.

    Args:
        scale: quantization scale of the output tensor
        zero_point: quantization zero point of the output tensor
    """

    def __init__(self, output_scale: float, output_zero_point: int):
        super().__init__()
        self.output_scale = output_scale
        self.output_zero_point = output_zero_point
```
- **EN**: It introduces or extends class-level abstractions such as `LeakyReLU`, `Sigmoid`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LeakyReLU`, `Sigmoid` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 175-196 / 第 175-196 行
```python
    def forward(self, input):
        return torch.ops.quantized.sigmoid(
            input, self.output_scale, self.output_zero_point
        )

    @classmethod
    def from_float(cls, mod, use_precomputed_fake_quant=False):
        (
            output_scale,
            output_zero_point,
        ) = mod.activation_post_process.calculate_qparams()
        return cls(float(output_scale), int(output_zero_point))


class Softmax(torch.nn.Softmax):
    r"""This is the quantized version of :class:`~torch.nn.Softmax`.

    Args:
        dim: A dimension along which Softmax will be computed (so every slice along dim will sum to 1).
        scale: quantization scale of the output tensor
        zero_point: quantization zero point of the output tensor
    """
```
- **EN**: It introduces or extends class-level abstractions such as `Sigmoid`, `Softmax`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Sigmoid`, `Softmax` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 198-216 / 第 198-216 行
```python
    def __init__(self, dim=None, scale=1.0, zero_point=0):
        super().__init__()
        self.dim = dim
        self.scale = scale
        self.zero_point = zero_point

    def forward(self, input):
        dim = self.dim
        if dim is None:
            stacklevel = 3
            # Note: adding the mypy ignore on _get_softmax_dim seems less bad
            # than making `_get_softmax_dim` an official API.
            dim = torch.nn.functional._get_softmax_dim(  # type: ignore[attr-defined]
                "softmax", input.dim(), stacklevel
            )
        return torch.ops.quantized.softmax(input, dim, self.scale, self.zero_point)

    def _get_name(self):
        return "QuantizedSoftmax"
```
- **EN**: It introduces or extends class-level abstractions such as `Softmax`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Softmax` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 218-232 / 第 218-232 行
```python
    @staticmethod
    def from_float(mod, use_precomputed_fake_quant=False):
        scale, zero_point = mod.activation_post_process.calculate_qparams()
        return Softmax(mod.dim, float(scale), int(zero_point))

    @classmethod
    def from_reference(cls, mod, scale, zero_point):
        return cls(mod.dim, float(scale), int(zero_point))


class MultiheadAttention(torch.ao.nn.quantizable.MultiheadAttention):
    _FLOAT_MODULE = torch.ao.nn.quantizable.MultiheadAttention

    def _get_name(self):
        return "QuantizedMultiheadAttention"
```
- **EN**: It introduces or extends class-level abstractions such as `Softmax`, `MultiheadAttention`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Named constants such as `_FLOAT_MODULE` centralize shared configuration or sentinel values. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Softmax`, `MultiheadAttention` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 `_FLOAT_MODULE` 等具名常量把共享配置或哨兵值集中定义在一起。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 234-257 / 第 234-257 行
```python
    @classmethod
    def from_float(cls, other):
        # The whole flow is float -> observed -> quantized
        # This class does observed -> quantized only
        raise NotImplementedError(
            "It looks like you are trying to convert a "
            "non-observed MHA module. Please, see "
            "the examples on quantizable MHAs."
        )

    @classmethod
    def from_observed(cls, other):
        converted = torch.ao.quantization.convert(
            other,
            mapping=None,
            inplace=False,
            remove_qconfig=True,
            convert_custom_config_dict=None,
        )
        converted.__class__ = cls
        # Remove the parameters for the bias_k and bias_v to quantize them
        # TODO: This is a potential source of accuracy drop.
        #       quantized cat takes the scale and zp of the first
        #       element, which might lose the precision in the bias_k
```
- **EN**: It introduces or extends class-level abstractions such as `MultiheadAttention`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `MultiheadAttention` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 258-277 / 第 258-277 行
```python
        #       and the bias_v (which are cat'ed with k/v being first).
        if converted.bias_k is not None:
            bias_k = converted._parameters.pop("bias_k")
            sc, zp = torch._choose_qparams_per_tensor(bias_k, reduce_range=False)
            bias_k = torch.quantize_per_tensor(bias_k, sc, zp, torch.quint8)
            setattr(converted, "bias_k", bias_k)  # noqa: B010

        if converted.bias_v is not None:
            bias_v = converted._parameters.pop("bias_v")
            sc, zp = torch._choose_qparams_per_tensor(
                bias_k,  # type: ignore[possibly-undefined]
                reduce_range=False,
            )
            bias_v = torch.quantize_per_tensor(bias_v, sc, zp, torch.quint8)
            setattr(converted, "bias_v", bias_v)  # noqa: B010

        del converted.in_proj_weight
        del converted.in_proj_bias

        return converted
```
- **EN**: It introduces or extends class-level abstractions such as `MultiheadAttention`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `MultiheadAttention` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 280-301 / 第 280-301 行
```python
class PReLU(torch.nn.Module):
    r"""This is the quantized equivalent of :class:`~torch.nn.PReLU`.

    Args:
        scale: quantization scale of the output tensor
        zero_point: quantization zero point of the output tensor
        num_parameters: number of parameters: 1, or the number of channels at input. Default: 1
    """

    def __init__(
        self, output_scale: float, output_zero_point: int, num_parameters: int = 1
    ) -> None:
        super().__init__()
        self.num_parameters = num_parameters
        self.scale = output_scale
        self.zero_point = output_zero_point
        w = torch.randn(num_parameters, dtype=torch.float)
        qw = torch.quantize_per_tensor(w, scale=1.0, zero_point=0, dtype=torch.quint8)
        self.set_weight(qw)

    def set_weight(self, w: torch.Tensor) -> None:
        self.weight = w
```
- **EN**: It introduces or extends class-level abstractions such as `PReLU`, which organize state and behavior for this subsystem. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `PReLU` 等类级抽象，用于组织该子系统的状态与行为。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 303-326 / 第 303-326 行
```python
    def forward(self, input: torch.Tensor) -> torch.Tensor:
        return torch.ops.quantized.prelu(
            input, self.weight, self.scale, self.zero_point
        )

    def _get_name(self):
        return "QuantizedPReLU"

    @classmethod
    def from_float(cls, mod, use_precomputed_fake_quant=False):
        scale, zero_point = mod.activation_post_process.calculate_qparams()
        qprelu = cls(float(scale), int(zero_point), mod.num_parameters)
        float_wt = mod.weight.float()
        observer = mod.qconfig.weight()
        observer(float_wt)
        if observer.dtype != torch.quint8:
            warn(
                f"PReLU's weight observer should have dtype quint8 but got {observer.dtype}",
                stacklevel=2,
            )
        wt_scale, wt_zp = observer.calculate_qparams()
        qweight = torch.quantize_per_tensor(
            float_wt, float(wt_scale), int(wt_zp), torch.quint8
        )
```
- **EN**: It introduces or extends class-level abstractions such as `PReLU`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `PReLU` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 327-346 / 第 327-346 行
```python
        qprelu.set_weight(qweight)
        return qprelu

    @classmethod
    def from_reference(cls, mod, scale, zero_point):
        qprelu = cls(float(scale), int(zero_point), mod.num_parameters)
        float_wt = mod.weight.float()
        observer = mod.qconfig.weight()
        observer(float_wt)
        if observer.dtype != torch.quint8:
            warn(
                f"PReLU's weight observer should have dtype quint8 but got {observer.dtype}",
                stacklevel=2,
            )
        wt_scale, wt_zp = observer.calculate_qparams()
        qweight = torch.quantize_per_tensor(
            float_wt, float(wt_scale), int(wt_zp), torch.quint8
        )
        qprelu.set_weight(qweight)
        return qprelu
```
- **EN**: It introduces or extends class-level abstractions such as `PReLU`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `PReLU` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements quantization-aware neural-network module wrappers, fused building blocks, and reference/quantized module variants.
  - CN: 实现面向量化的神经网络模块包装、融合构件以及参考/量化模块变体。
- **Quantization pipeline**
  - EN: Tracks calibration statistics, quantization parameters, or graph/module rewrites for lower-precision execution.
  - CN: 跟踪校准统计信息、量化参数，或面向低精度执行的图/模块改写。
- **Observer statistics**
  - EN: Collects runtime ranges or histograms so later code can derive quantization settings.
  - CN: 收集运行时范围或直方图，以便后续逻辑推导量化设置。
- **Fake quantization**
  - EN: Simulates quantized numerics during training while keeping tensors in differentiable forms.
  - CN: 在训练阶段模拟量化数值行为，同时保持张量处于可求导形式。
- **Graph rewriting**
  - EN: Rewrites module or FX graph structure so later execution follows a transformed pipeline.
  - CN: 改写模块或 FX 图结构，使后续执行遵循转换后的流水线。
- **ReLU6**
  - EN: `ReLU6` is one of the main classes that structures the file's behavior.
  - CN: `ReLU6` 是组织该文件行为的核心类之一。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`
- **Python standard library / Python 标准库**: `warnings:warn`
- **Explicit exports / 显式导出**: `ReLU6`, `Hardswish`, `ELU`, `LeakyReLU`, `Sigmoid`, `Softmax`, `MultiheadAttention`, `PReLU`
- **Primary symbols / 核心符号**: `ReLU6`, `Hardswish`, `ELU`, `LeakyReLU`, `Sigmoid`, `Softmax`, `MultiheadAttention`, `PReLU`
