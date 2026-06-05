# linear_relu.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/nn/intrinsic/quantized/modules/linear_relu.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements quantization-aware neural-network module wrappers, fused building blocks, and reference/quantized module variants. This specific file centers on `linear_relu.py`. The file is especially concerned with quantization metadata, calibration, or fake-quant behavior. Runtime configuration, environment overrides, or feature gating are important in this module.
- **Purpose (CN) / 用途（中文）**: 实现面向量化的神经网络模块包装、融合构件以及参考/量化模块变体。 该文件具体围绕 `linear_relu.py` 展开。 该文件尤其关注量化元数据、校准流程或 fake-quant 行为。 运行时配置、环境变量覆盖或特性开关是此模块的重要主题。
## Line-by-Line Analysis / 逐行分析

### Lines 1-17 / 第 1-17 行
```python
# mypy: allow-untyped-defs
import torch
import torch.ao.nn.intrinsic as nni
import torch.ao.nn.quantized as nnq
from torch.ao.nn.quantized.modules.utils import _quantize_weight


__all__ = [
    "LinearReLU",
    "LinearLeakyReLU",
    "LinearTanh",
]


class LinearReLU(nnq.Linear):
    r"""
    A LinearReLU module fused from Linear and ReLU modules
```
- **EN**: The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export. It introduces or extends class-level abstractions such as `LinearReLU`, which organize state and behavior for this subsystem. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。 它引入或扩展了 `LinearReLU` 等类级抽象，用于组织该子系统的状态与行为。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 19-34 / 第 19-34 行
```python
    We adopt the same interface as :class:`torch.ao.nn.quantized.Linear`.

    Attributes:
        Same as torch.ao.nn.quantized.Linear

    Examples::

        >>> # xdoctest: +SKIP
        >>> m = nn.intrinsic.LinearReLU(20, 30)
        >>> input = torch.randn(128, 20)
        >>> output = m(input)
        >>> print(output.size())
        torch.Size([128, 30])
    """

    _FLOAT_MODULE = nni.LinearReLU  # type: ignore[assignment]
```
- **EN**: It introduces or extends class-level abstractions such as `LinearReLU`, which organize state and behavior for this subsystem. Named constants such as `_FLOAT_MODULE` centralize shared configuration or sentinel values. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LinearReLU` 等类级抽象，用于组织该子系统的状态与行为。 `_FLOAT_MODULE` 等具名常量把共享配置或哨兵值集中定义在一起。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 36-49 / 第 36-49 行
```python
    def __init__(self, in_features, out_features, bias=True, dtype=torch.qint8):
        super().__init__(in_features, out_features, bias, dtype)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return torch.ops.quantized.linear_relu(
            x, self._packed_params._packed_params, self.scale, self.zero_point
        )

    def _get_name(self):
        return "QuantizedLinearReLU"

    @classmethod
    def from_float(cls, mod, use_precomputed_fake_quant=False):
        return super().from_float(mod, use_precomputed_fake_quant)
```
- **EN**: It introduces or extends class-level abstractions such as `LinearReLU`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LinearReLU` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 51-68 / 第 51-68 行
```python
    @classmethod
    def from_reference(cls, ref_linear_relu, output_scale, output_zero_point):
        return super().from_reference(
            ref_linear_relu[0], output_scale, output_zero_point
        )


class LinearLeakyReLU(nnq.Linear):
    r"""
    For onednn backend only
    A LinearLeakyReLU module fused from Linear and LeakyReLU modules
    We adopt the same interface as :class:`torch.ao.nn.quantized.Linear`.
    Attributes:
        Same as torch.ao.nn.quantized.Linear
        + negative_slope
    Examples::
        >>> # xdoctest: +SKIP
        >>> m = nn.intrinsic.LinearLeakyReLU(20, 30, 0.01)
```
- **EN**: It introduces or extends class-level abstractions such as `LinearReLU`, `LinearLeakyReLU`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LinearReLU`, `LinearLeakyReLU` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 69-81 / 第 69-81 行
```python
        >>> input = torch.randn(128, 20)
        >>> output = m(input)
        >>> print(output.size())
        torch.Size([128, 30])
    """

    _FLOAT_MODULE = nni.LinearLeakyReLU  # type: ignore[assignment]

    def __init__(
        self, in_features, out_features, negative_slope, bias=True, dtype=torch.qint8
    ):
        super().__init__(in_features, out_features, bias, dtype)
        self.negative_slope = negative_slope
```
- **EN**: It introduces or extends class-level abstractions such as `LinearLeakyReLU`, which organize state and behavior for this subsystem. Named constants such as `_FLOAT_MODULE` centralize shared configuration or sentinel values. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LinearLeakyReLU` 等类级抽象，用于组织该子系统的状态与行为。 `_FLOAT_MODULE` 等具名常量把共享配置或哨兵值集中定义在一起。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 83-93 / 第 83-93 行
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return torch.ops.quantized.linear_leaky_relu(
            x,
            self._packed_params._packed_params,
            self.scale,
            self.zero_point,
            self.negative_slope,
        )

    def _get_name(self):
        return "QuantizedLinearLeakyReLU"
```
- **EN**: It introduces or extends class-level abstractions such as `LinearLeakyReLU`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LinearLeakyReLU` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 95-112 / 第 95-112 行
```python
    @classmethod
    def from_float(cls, mod, use_precomputed_fake_quant=False):
        if type(mod) is not nni.LinearLeakyReLU:
            raise AssertionError("Input float module should be LinearLeakyReLU")
        if not hasattr(mod, "qconfig"):
            raise AssertionError("Input float module must have qconfig defined")
        activation_post_process = mod.activation_post_process
        leaky_relu = mod[1]
        mod = mod[0]
        weight_post_process = mod.qconfig.weight()  # type: ignore[union-attr, operator]
        weight_post_process(mod.weight)
        dtype = weight_post_process.dtype
        act_scale, act_zp = activation_post_process.calculate_qparams()  # type: ignore[union-attr,operator]
        if dtype != torch.qint8:
            raise AssertionError(
                f"Weight observer must have dtype torch.qint8, got {dtype}"
            )
        qweight = _quantize_weight(mod.weight.float(), weight_post_process)
```
- **EN**: It introduces or extends class-level abstractions such as `LinearLeakyReLU`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LinearLeakyReLU` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 113-130 / 第 113-130 行
```python
        qlinear_leaky_relu = cls(
            mod.in_features, mod.out_features, leaky_relu.negative_slope, dtype=dtype
        )
        qlinear_leaky_relu.set_weight_bias(qweight, mod.bias)  # type: ignore[arg-type]
        qlinear_leaky_relu.scale = float(act_scale)
        qlinear_leaky_relu.zero_point = int(act_zp)
        return qlinear_leaky_relu

    @classmethod
    def from_reference(cls, ref_mod, output_scale, output_zero_point):
        linear = ref_mod[0]
        leaky_relu = ref_mod[1]
        qlinear_leaky_relu = cls(
            linear.in_features, linear.out_features, leaky_relu.negative_slope
        )
        qweight = linear.get_quantized_weight()
        qlinear_leaky_relu.set_weight_bias(qweight, linear.bias)
        qlinear_leaky_relu.scale = float(output_scale)
```
- **EN**: It introduces or extends class-level abstractions such as `LinearLeakyReLU`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LinearLeakyReLU` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 131-144 / 第 131-144 行
```python
        qlinear_leaky_relu.zero_point = int(output_zero_point)
        return qlinear_leaky_relu


class LinearTanh(nnq.Linear):
    r"""
    A LinearTanh module fused from Linear and Tanh modules

    We adopt the same interface as :class:`torch.ao.nn.quantized.Linear`.

    Attributes:
        Same as torch.ao.nn.quantized.Linear

    Examples::
```
- **EN**: It introduces or extends class-level abstractions such as `LinearLeakyReLU`, `LinearTanh`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LinearLeakyReLU`, `LinearTanh` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 146-162 / 第 146-162 行
```python
        >>> # xdoctest: +SKIP
        >>> m = nn.intrinsic.LinearTanh(20, 30)
        >>> input = torch.randn(128, 20)
        >>> output = m(input)
        >>> print(output.size())
        torch.Size([128, 30])
    """

    _FLOAT_MODULE = nni.LinearTanh  # type: ignore[assignment]

    def __init__(self, in_features, out_features, bias=True, dtype=torch.qint8):
        super().__init__(in_features, out_features, bias, dtype)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return torch.ops.quantized.linear_tanh(
            x, self._packed_params._packed_params, self.scale, self.zero_point
        )
```
- **EN**: It introduces or extends class-level abstractions such as `LinearTanh`, which organize state and behavior for this subsystem. Named constants such as `_FLOAT_MODULE` centralize shared configuration or sentinel values. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LinearTanh` 等类级抽象，用于组织该子系统的状态与行为。 `_FLOAT_MODULE` 等具名常量把共享配置或哨兵值集中定义在一起。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 164-181 / 第 164-181 行
```python
    def _get_name(self):
        return "QuantizedLinearTanh"

    @classmethod
    def from_float(cls, mod, use_precomputed_fake_quant=False):
        if type(mod) is not nni.LinearTanh:
            raise AssertionError("Input float module should be LinearTanh")
        if not hasattr(mod, "qconfig"):
            raise AssertionError("Input float module must have qconfig defined")
        activation_post_process = mod.activation_post_process
        mod = mod[0]
        weight_post_process = mod.qconfig.weight()  # type: ignore[union-attr,operator]
        weight_post_process(mod.weight)
        dtype = weight_post_process.dtype
        act_scale, act_zp = activation_post_process.calculate_qparams()  # type: ignore[union-attr,operator]
        if dtype != torch.qint8:
            raise AssertionError(
                f"Weight observer must have dtype torch.qint8, got {dtype}"
```
- **EN**: It introduces or extends class-level abstractions such as `LinearTanh`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LinearTanh` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 182-198 / 第 182-198 行
```python
            )
        qweight = _quantize_weight(mod.weight.float(), weight_post_process)
        qlinear_tanh = cls(mod.in_features, mod.out_features, dtype=dtype)
        qlinear_tanh.set_weight_bias(qweight, mod.bias)  # type: ignore[arg-type]
        qlinear_tanh.scale = float(act_scale)
        qlinear_tanh.zero_point = int(act_zp)
        return qlinear_tanh

    @classmethod
    def from_reference(cls, ref_mod, output_scale, output_zero_point):
        linear = ref_mod[0]
        qlinear_tanh = cls(linear.in_features, linear.out_features)
        qweight = linear.get_quantized_weight()
        qlinear_tanh.set_weight_bias(qweight, linear.bias)
        qlinear_tanh.scale = float(output_scale)
        qlinear_tanh.zero_point = int(output_zero_point)
        return qlinear_tanh
```
- **EN**: It introduces or extends class-level abstractions such as `LinearTanh`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LinearTanh` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

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
- **Configuration management**
  - EN: Centralizes feature flags, environment overrides, and runtime-config wiring.
  - CN: 集中管理特性开关、环境变量覆盖以及运行时配置接线。
- **LinearReLU**
  - EN: `LinearReLU` is one of the main classes that structures the file's behavior.
  - CN: `LinearReLU` 是组织该文件行为的核心类之一。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch.ao.nn.intrinsic`, `torch.ao.nn.quantized`, `torch.ao.nn.quantized.modules.utils:_quantize_weight`
- **Explicit exports / 显式导出**: `LinearReLU`, `LinearLeakyReLU`, `LinearTanh`
- **Primary symbols / 核心符号**: `LinearReLU`, `LinearLeakyReLU`, `LinearTanh`
