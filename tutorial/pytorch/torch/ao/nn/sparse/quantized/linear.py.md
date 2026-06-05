# linear.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/nn/sparse/quantized/linear.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements quantization-aware neural-network module wrappers, fused building blocks, and reference/quantized module variants. This specific file centers on `linear.py`. The file is especially concerned with quantization metadata, calibration, or fake-quant behavior.
- **Purpose (CN) / 用途（中文）**: 实现面向量化的神经网络模块包装、融合构件以及参考/量化模块变体。 该文件具体围绕 `linear.py` 展开。 该文件尤其关注量化元数据、校准流程或 fake-quant 行为。
## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
```python
# mypy: allow-untyped-defs

import torch
from torch.ao.nn.quantized.modules.utils import (
    _hide_packed_params_repr,
    _quantize_weight,
)


__all__ = ["LinearPackedParams", "Linear"]


# TODO (zaf): Inherit from `quantized.LinearPackedParams` (T83294430)
class LinearPackedParams(torch.nn.Module):
    _version = 1

    def __init__(self, row_block_size=1, col_block_size=4, dtype=torch.qint8):
        super().__init__()
```
- **EN**: The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export. It introduces or extends class-level abstractions such as `LinearPackedParams`, which organize state and behavior for this subsystem. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。 它引入或扩展了 `LinearPackedParams` 等类级抽象，用于组织该子系统的状态与行为。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 20-43 / 第 20-43 行
```python
        if dtype != torch.qint8:
            raise NotImplementedError("Linear prepacking only supports QINT8")
        self.dtype = dtype
        wq = torch._empty_affine_quantized(
            [1, 1], scale=1.0, zero_point=0, dtype=torch.qint8
        )
        self.set_weight_bias(wq, None, row_block_size, col_block_size)

    def _get_name(self):
        return "SparseQuantizedLinearPackedParams"

    @torch.jit.export
    def set_weight_bias(
        self,
        weight: torch.Tensor,
        bias: torch.Tensor | None,
        row_block_size: int | None,
        col_block_size: int | None,
    ) -> None:
        if row_block_size is None or col_block_size is None:
            raise AssertionError(
                "row_block_size and col_block_size must not be None, got "
                f"row_block_size={row_block_size}, col_block_size={col_block_size}"
            )
```
- **EN**: It introduces or extends class-level abstractions such as `LinearPackedParams`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LinearPackedParams` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 44-61 / 第 44-61 行
```python
        self._packed_params = torch.ops.sparse.qlinear_prepack(
            weight, bias, row_block_size, col_block_size
        )

    @torch.jit.export
    def _weight_bias(self):
        (weight, bias, block_sizes) = torch.ops.sparse.qlinear_unpack(
            self._packed_params
        )
        return (weight, bias, block_sizes[0], block_sizes[1])

    def forward(self, x):
        return x

    def _save_to_state_dict(self, destination, prefix, keep_vars):
        super()._save_to_state_dict(destination, prefix, keep_vars)
        destination[prefix + "dtype"] = self.dtype
        destination[prefix + "_packed_params"] = self._weight_bias()
```
- **EN**: It introduces or extends class-level abstractions such as `LinearPackedParams`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LinearPackedParams` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 63-81 / 第 63-81 行
```python
    def _load_from_state_dict(
        self,
        state_dict,
        prefix,
        local_metadata,
        strict,
        missing_keys,
        unexpected_keys,
        error_msgs,
    ):
        version = local_metadata.get("version", None)
        if version is not None and version > self._version:
            raise AssertionError(f"version {version} > self._version {self._version}")

        self.dtype = state_dict.pop(prefix + "dtype")
        weight, bias, row_block_size, col_block_size = state_dict.pop(
            prefix + "_packed_params"
        )
        self.set_weight_bias(weight, bias, row_block_size, col_block_size)
```
- **EN**: It introduces or extends class-level abstractions such as `LinearPackedParams`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LinearPackedParams` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 83-102 / 第 83-102 行
```python
        super()._load_from_state_dict(
            state_dict,
            prefix,
            local_metadata,
            False,
            missing_keys,
            unexpected_keys,
            error_msgs,
        )

    @torch.jit.export
    def __getstate__(self):
        return self._packed_params, self.training, self.dtype

    @torch.jit.export
    def __setstate__(self, state):
        (self._packed_params, self.training, self.dtype) = state

    def __repr__(self):
        return self._weight_bias().__repr__()
```
- **EN**: It introduces or extends class-level abstractions such as `LinearPackedParams`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `LinearPackedParams` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 105-123 / 第 105-123 行
```python
# TODO (zaf): Inherit from `quantized.Linear` (T83294430)
class Linear(torch.nn.Module):
    r"""
    A quantized sparse linear module with quantized tensor as inputs and outputs.
    """

    _version = 1
    _FLOAT_MODULE = torch.nn.Linear

    def __init__(
        self,
        in_features,
        out_features,
        row_block_size,
        col_block_size,
        bias=True,
        dtype=torch.qint8,
    ):
        super().__init__()
```
- **EN**: It introduces or extends class-level abstractions such as `Linear`, which organize state and behavior for this subsystem. Named constants such as `_FLOAT_MODULE` centralize shared configuration or sentinel values. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Linear` 等类级抽象，用于组织该子系统的状态与行为。 `_FLOAT_MODULE` 等具名常量把共享配置或哨兵值集中定义在一起。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 125-148 / 第 125-148 行
```python
        if dtype != torch.qint8:
            raise NotImplementedError(
                "Only QINT8 is supported for Sparse Quantized Linear"
            )

        self.in_features = in_features
        self.out_features = out_features

        if bias:
            bias = torch.zeros(self.out_features, dtype=torch.float)
        else:
            bias = None

        qweight = torch._empty_affine_quantized(
            [out_features, in_features], scale=1, zero_point=0, dtype=torch.qint8
        )
        self._packed_params = LinearPackedParams(
            row_block_size=row_block_size, col_block_size=col_block_size, dtype=dtype
        )
        self._packed_params.set_weight_bias(
            qweight, bias, row_block_size, col_block_size
        )
        self.scale = 1.0
        self.zero_point = 0
```
- **EN**: It introduces or extends class-level abstractions such as `Linear`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Linear` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 150-171 / 第 150-171 行
```python
    @classmethod
    def _get_name(cls):
        return "SparseQuantizedLinear"

    def extra_repr(self):
        return (
            f"in_features={self.in_features}, out_features={self.out_features}, scale={self.scale}, "
            f"zero_point={self.zero_point}, qscheme={self.weight().qscheme()}"
        )

    def __repr__(self):
        return _hide_packed_params_repr(self, LinearPackedParams)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return torch.ops.sparse.qlinear(
            x, self._packed_params._packed_params, self.scale, self.zero_point
        )

    def _save_to_state_dict(self, destination, prefix, keep_vars):
        super()._save_to_state_dict(destination, prefix, keep_vars)
        destination[prefix + "scale"] = torch.tensor(self.scale)
        destination[prefix + "zero_point"] = torch.tensor(self.zero_point)
```
- **EN**: It introduces or extends class-level abstractions such as `Linear`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Linear` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 173-193 / 第 173-193 行
```python
    def _load_from_state_dict(
        self,
        state_dict,
        prefix,
        local_metadata,
        strict,
        missing_keys,
        unexpected_keys,
        error_msgs,
    ):
        self.scale = float(state_dict[prefix + "scale"])
        state_dict.pop(prefix + "scale")

        self.zero_point = int(state_dict[prefix + "zero_point"])
        state_dict.pop(prefix + "zero_point")

        state_dict.pop(prefix + "op_type")

        version = local_metadata.get("version", None)
        if version is not None and version > self._version:
            raise AssertionError(f"version {version} > self._version {self._version}")
```
- **EN**: It introduces or extends class-level abstractions such as `Linear`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Linear` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 195-212 / 第 195-212 行
```python
        super()._load_from_state_dict(
            state_dict,
            prefix,
            local_metadata,
            False,
            missing_keys,
            unexpected_keys,
            error_msgs,
        )

    def _weight_bias(self):
        return self._packed_params._weight_bias()

    def weight(self):
        return self._weight_bias()[0]

    def bias(self):
        return self._weight_bias()[1]
```
- **EN**: It introduces or extends class-level abstractions such as `Linear`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Linear` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 214-232 / 第 214-232 行
```python
    def set_weight_bias(
        self,
        w: torch.Tensor,
        b: torch.Tensor | None,
        row_block_size: int | None,
        col_block_size: int | None,
    ) -> None:
        if row_block_size is None or col_block_size is None:
            raise AssertionError(
                "row_block_size and col_block_size must not be None, "
                f"got row_block_size={row_block_size}, col_block_size={col_block_size}"
            )
        self._packed_params.set_weight_bias(w, b, row_block_size, col_block_size)

    @classmethod
    def from_float(cls, mod, use_precomputed_fake_quant=False):
        r"""Create a quantized sparse module from a float module.

        We only care about the convert at this stage, no need for observers just yet.
```
- **EN**: It introduces or extends class-level abstractions such as `Linear`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Linear` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 234-257 / 第 234-257 行
```python
        TODO(zaf): Need to add the sparse params to the qconfig
        """
        if type(mod) is not cls._FLOAT_MODULE:
            raise AssertionError(
                cls._get_name()
                + ".from_float only works for "
                + cls._FLOAT_MODULE.__name__
            )
        if not hasattr(mod, "sparse_params"):
            raise AssertionError(
                "Expecting the Linear to have `sparse_params`. Make sure you have provided arguments "
                'in the `sparsifier.squash_mask(params_to_save=("sparse_block_shape",))` method.'
            )
        sparse_block_shape = mod.sparse_params.get("sparse_block_shape", None)  # type: ignore[operator, union-attr]
        if not isinstance(sparse_block_shape, (tuple, list)):
            raise AssertionError(
                f"sparse_block_shape must be tuple or list, got {type(sparse_block_shape)}"
            )
        if len(sparse_block_shape) != 2:
            raise AssertionError(
                f"sparse_block_shape must have length 2, got {len(sparse_block_shape)}"
            )
        # TODO: Need to add options to qconfig to avoid the calibration.
        # TODO: Add calibration for the sparsity
```
- **EN**: It introduces or extends class-level abstractions such as `Linear`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Linear` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 258-281 / 第 258-281 行
```python
        if not hasattr(mod, "qconfig"):
            raise AssertionError("Input float module must have qconfig defined")
        activation_post_process = mod.activation_post_process
        weight_post_process = mod.qconfig.weight()  # type: ignore[operator, union-attr]

        # Assumption is that the weight is already sparsified by the
        # `sparsifier.convert`
        weight = mod.weight

        weight_post_process(weight)
        dtype = weight_post_process.dtype
        act_scale, act_zp = activation_post_process.calculate_qparams()  # type: ignore[operator, union-attr]
        if dtype != torch.qint8:
            raise AssertionError(
                f"Weight observer must have dtype torch.qint8, got {dtype}"
            )
        w_sc, w_zp = weight_post_process.calculate_qparams()
        if isinstance(w_zp, torch.Tensor):
            if torch.any(w_zp.bool()):
                raise AssertionError("All weight zero points must map to 0")
        else:
            if w_zp != 0:
                raise AssertionError(f"Weight zero point must map to 0, got {w_zp}")
        qweight = _quantize_weight(weight.float(), weight_post_process)
```
- **EN**: It introduces or extends class-level abstractions such as `Linear`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Linear` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 283-300 / 第 283-300 行
```python
        row_block_size = mod.sparse_params["sparse_block_shape"][0]  # type: ignore[index]
        col_block_size = mod.sparse_params["sparse_block_shape"][1]  # type: ignore[index]
        qlinear = cls(
            mod.in_features,
            mod.out_features,
            row_block_size,
            col_block_size,
            dtype=dtype,
        )
        qlinear.set_weight_bias(
            qweight,
            mod.bias,
            row_block_size,  # type: ignore[arg-type]
            col_block_size,  # type: ignore[arg-type]
        )
        qlinear.scale = float(act_scale)
        qlinear.zero_point = int(act_zp)
        return qlinear
```
- **EN**: It introduces or extends class-level abstractions such as `Linear`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Linear` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

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
- **Pruning and sparsity**
  - EN: Controls parameter masking, sparse layouts, or pruning schedules that reduce effective model size.
  - CN: 控制参数掩码、稀疏布局或剪枝计划，以降低模型的有效规模。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch.ao.nn.quantized.modules.utils:_hide_packed_params_repr`, `torch.ao.nn.quantized.modules.utils:_quantize_weight`
- **Explicit exports / 显式导出**: `LinearPackedParams`, `Linear`
- **Primary symbols / 核心符号**: `LinearPackedParams`, `Linear`
