# linear.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/nn/sparse/quantized/dynamic/linear.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements quantization-aware neural-network module wrappers, fused building blocks, and reference/quantized module variants. This specific file centers on `linear.py`. The file is especially concerned with quantization metadata, calibration, or fake-quant behavior.
- **Purpose (CN) / 用途（中文）**: 实现面向量化的神经网络模块包装、融合构件以及参考/量化模块变体。 该文件具体围绕 `linear.py` 展开。 该文件尤其关注量化元数据、校准流程或 fake-quant 行为。
## Line-by-Line Analysis / 逐行分析

### Lines 1-23 / 第 1-23 行
```python
# mypy: allow-untyped-defs

import torch
import torch.ao.nn.intrinsic as nni
from torch.ao.nn.quantized.modules.utils import (
    _hide_packed_params_repr,
    _quantize_weight,
)
from torch.ao.nn.sparse.quantized import linear
from torch.ao.nn.sparse.quantized.utils import LinearBlockSparsePattern


__all__ = ["Linear"]


class Linear(torch.nn.Module):
    r"""
    A dynamically quantized sparse linear module with float tensor as inputs and outputs.
    """

    _version = 1
    _op_type = "sparse_dynamic"
    _FLOAT_MODULE = torch.nn.Linear
```
- **EN**: The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export. It introduces or extends class-level abstractions such as `Linear`, which organize state and behavior for this subsystem. Named constants such as `_FLOAT_MODULE` centralize shared configuration or sentinel values. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。 它引入或扩展了 `Linear` 等类级抽象，用于组织该子系统的状态与行为。 `_FLOAT_MODULE` 等具名常量把共享配置或哨兵值集中定义在一起。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 25-47 / 第 25-47 行
```python
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

        if dtype != torch.qint8:
            raise NotImplementedError(
                "Only QINT8 is supported for Sparse Quantized Linear Dynamic"
            )

        self.in_features = in_features
        self.out_features = out_features

        if bias:
            bias = torch.zeros(self.out_features, dtype=torch.float)
        else:
            bias = None
```
- **EN**: It introduces or extends class-level abstractions such as `Linear`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Linear` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 49-69 / 第 49-69 行
```python
        qweight = torch._empty_affine_quantized(
            [out_features, in_features], scale=1, zero_point=0, dtype=torch.qint8
        )
        self._packed_params = linear.LinearPackedParams(
            row_block_size=row_block_size, col_block_size=col_block_size, dtype=dtype
        )
        self._packed_params.set_weight_bias(
            qweight, bias, row_block_size, col_block_size
        )

    def _get_name(self):
        return "SparseQuantizedDynamicLinear"

    def extra_repr(self):
        return f"in_features={self.in_features}, out_features={self.out_features}, qscheme={self.weight().qscheme()}"

    def __repr__(self):
        return _hide_packed_params_repr(self, linear.LinearPackedParams)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return torch.ops.sparse.qlinear_dynamic(x, self._packed_params._packed_params)
```
- **EN**: It introduces or extends class-level abstractions such as `Linear`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Linear` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 71-90 / 第 71-90 行
```python
    def _save_to_state_dict(self, destination, prefix, keep_vars):
        super()._save_to_state_dict(destination, prefix, keep_vars)
        destination[prefix + "op_type"] = self._op_type

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
        op_type = int(state_dict[prefix + "op_type"])
        if op_type != "sparse":
            raise AssertionError(
                f"Cannot load from op_type [{op_type}], expecting [{self._op_type}]"
            )
        state_dict.pop(prefix + "op_type")
```
- **EN**: It introduces or extends class-level abstractions such as `Linear`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Linear` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 92-115 / 第 92-115 行
```python
        version = local_metadata.get("version", None)
        if version is not None and version > self._version:
            raise AssertionError(f"version {version} > self._version {self._version}")

        # Is this code valid? In old quantization it seemed to be used to load
        # older model
        weight = state_dict.pop(prefix + "weight")
        bias = state_dict.pop(prefix + "bias")
        state_dict.update(
            {
                prefix + "_packed_params.weight": weight,
                prefix + "_packed_params.bias": bias,
            }
        )

        super()._load_from_state_dict(
            state_dict,
            prefix,
            local_metadata,
            False,
            missing_keys,
            unexpected_keys,
            error_msgs,
        )
```
- **EN**: It introduces or extends class-level abstractions such as `Linear`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Linear` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 117-139 / 第 117-139 行
```python
    def _weight_bias(self):
        return self._packed_params._weight_bias()

    def weight(self):
        return self._weight_bias()[0]

    def bias(self):
        return self._weight_bias()[1]

    def set_weight_bias(
        self,
        w: torch.Tensor,
        b: torch.Tensor | None,
        row_block_size: int | None,
        col_block_size: int | None,
    ) -> None:
        if row_block_size is None or col_block_size is None:
            raise AssertionError(
                f"row_block_size and col_block_size must not be None, got {row_block_size=}, {col_block_size=}"
            )
        self.out_features = w.shape[0]
        self.in_features = w.shape[1]
        self._packed_params.set_weight_bias(w, b, row_block_size, col_block_size)
```
- **EN**: It introduces or extends class-level abstractions such as `Linear`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Linear` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 141-164 / 第 141-164 行
```python
    @classmethod
    def from_float(cls, mod, use_precomputed_fake_quant=False):
        r"""Create a quantized sparse dynamic module from a float module.

        We only care about the convert at this stage, no need for observers just yet.
        """
        if type(mod) is not cls._FLOAT_MODULE:
            raise AssertionError(
                " nnq."
                + cls.__name__
                + ".from_float only works for "
                + cls._FLOAT_MODULE.__name__
            )
        # TODO: Need to add options to qconfig to avoid the calibration.
        # TODO: Add calibration for the sparsity
        if not hasattr(mod, "qconfig"):
            raise AssertionError("Input float module must have qconfig defined")
        if type(mod) is nni.LinearReLU:
            mod = mod[0]
        # pyrefly: ignore [missing-attribute]
        if mod.qconfig is not None and mod.qconfig.weight is not None:
            # pyrefly: ignore [not-callable]
            weight_observer = mod.qconfig.weight()
        else:
```
- **EN**: It introduces or extends class-level abstractions such as `Linear`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Linear` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 165-188 / 第 165-188 行
```python
            # We have the circular import issues if we import the qconfig in the beginning of this file:
            # https://github.com/pytorch/pytorch/pull/24231. The current workaround is to postpone the
            # import until we need it.
            from torch.ao.quantization.qconfig import default_dynamic_qconfig

            weight_observer = default_dynamic_qconfig.weight()

        # It is important to multiply by the mask BEFORE calling the `weight_observer`
        # TODO (zaf): Mask might not be part of the qconfig (T83295194)
        weight = mod.weight
        if getattr(mod.qconfig, "mask", False):
            weight = mod.qconfig.mask * mod.weight

        weight_observer(weight)
        dtype = weight_observer.dtype
        if dtype != torch.qint8:
            raise AssertionError(
                f"Weight observer must have dtype torch.qint8, got {dtype}"
            )
        _w_sc, w_zp = weight_observer.calculate_qparams()
        if isinstance(w_zp, torch.Tensor):
            if torch.any(w_zp.bool()):
                raise AssertionError("All weight zero points must map to 0")
        else:
```
- **EN**: It introduces or extends class-level abstractions such as `Linear`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Linear` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 189-203 / 第 189-203 行
```python
            if w_zp != 0:
                raise AssertionError(f"Weight zero point must map to 0, got {w_zp}")
        qweight = _quantize_weight(weight.float(), weight_observer)

        row_block_size, col_block_size = LinearBlockSparsePattern.block_size()
        qlinear = cls(
            mod.in_features,
            mod.out_features,
            row_block_size,
            col_block_size,
            dtype=dtype,
        )
        # pyrefly: ignore [bad-argument-type]
        qlinear.set_weight_bias(qweight, mod.bias, row_block_size, col_block_size)
        return qlinear
```
- **EN**: It introduces or extends class-level abstractions such as `Linear`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Linear` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

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

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch.ao.nn.intrinsic`, `torch.ao.nn.quantized.modules.utils:_hide_packed_params_repr`, `torch.ao.nn.quantized.modules.utils:_quantize_weight`, `torch.ao.nn.sparse.quantized:linear`, `torch.ao.nn.sparse.quantized.utils:LinearBlockSparsePattern`
- **Explicit exports / 显式导出**: `Linear`
- **Primary symbols / 核心符号**: `Linear`
