# quant_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/utils/quant_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Collects utility helpers for quantization backends, schemes, and utilities. / 汇总量化后端、方案与工具的工具函数。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-3)
```python
"""This file is used for /tests and /benchmarks"""
```
**EN:** This docstring gives the module author's high-level intent: This file is used for /tests and /benchmarks It is useful for reading the rest of the file because later classes and helpers refine this stated purpose.
**CN:** 这个文档字符串给出了模块作者的高层意图：This file is used for /tests and /benchmarks 在阅读后续类和辅助函数时，可以把它视为整个文件的总纲。

### Imports and module setup (lines 5-15)
```python
from collections.abc import Callable, Mapping
from dataclasses import dataclass
from types import MappingProxyType
from typing import TYPE_CHECKING, ClassVar, NamedTuple

import numpy
import torch
from torch import fx

from vllm.platforms import current_platform
from vllm.scalar_type import ScalarType, scalar_types
```
**EN:** This opening block pulls in external dependencies such as `collections`, `dataclasses`, `types`, `typing`, `numpy`, `torch` and internal modules such as `vllm.platforms`, `vllm.scalar_type`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `collections`, `dataclasses`, `types`, `typing`, `numpy`, `torch`）以及内部模块（如 `vllm.platforms`, `vllm.scalar_type`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 20-68)
```python
FP8_DTYPE = current_platform.fp8_dtype()
FP4_DTYPE = torch.uint8
MXFP_SCALE_DTYPE = torch.uint8
INT4_DTYPE = scalar_types.uint4b8
INT8_DTYPE = scalar_types.uint8b128


def get_fp8_min_max() -> tuple[float, float]:
    """Get the min and max values for FP8 quantization."""
    # Using the default value (240.0) from pytorch will cause accuracy
    # issue on dynamic quantization models on ROCm. Here, use 224.0 for fnuz
    # on ROCm platforms that use the torch.float8_e4m3fnuz dtype.
    if current_platform.is_fp8_fnuz():
        return -224.0, 224.0
    finfo = torch.finfo(current_platform.fp8_dtype())
    return finfo.min, finfo.max


# Use proxy as NamedTuple direct subclasses cannot have static members
class _GroupShape(NamedTuple):
    row: int
    col: int


class GroupShape(_GroupShape):
    """
    This class describes the quantization group shape.
    It includes static members for common shapes (per-tensor, per-token).
    """

    # Aliases for common quantization group shapes
    PER_TENSOR: ClassVar["GroupShape"]
    PER_TOKEN: ClassVar["GroupShape"]
    PER_CHANNEL: ClassVar["GroupShape"]

    def is_per_tensor(self) -> bool:
        return self.row == -1 and self.col == -1

    def is_per_token(self) -> bool:
        return self.row == 1 and self.col == -1

    def is_per_channel(self) -> bool:
        return self.row == -1 and self.col == 1

    def is_per_group(self) -> bool:
        return self.row == 1 and self.col >= 1


GroupShape.PER_TENSOR = GroupShape(-1, -1)
```
**EN:** This block defines module-level metadata or constants such as `FP8_DTYPE`, `FP4_DTYPE`, `MXFP_SCALE_DTYPE`, `INT4_DTYPE`, `INT8_DTYPE`, `PER_TENSOR`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `FP8_DTYPE`, `FP4_DTYPE`, `MXFP_SCALE_DTYPE`, `INT4_DTYPE`, `INT8_DTYPE`, `PER_TENSOR`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Function `get_fp8_min_max` (lines 27-35)
```python
def get_fp8_min_max() -> tuple[float, float]:
    """Get the min and max values for FP8 quantization."""
    # Using the default value (240.0) from pytorch will cause accuracy
    # issue on dynamic quantization models on ROCm. Here, use 224.0 for fnuz
    # on ROCm platforms that use the torch.float8_e4m3fnuz dtype.
    if current_platform.is_fp8_fnuz():
        return -224.0, 224.0
    finfo = torch.finfo(current_platform.fp8_dtype())
    return finfo.min, finfo.max
```
**EN:** Defines function `get_fp8_min_max` with signature `get_fp8_min_max() -> tuple[float, float]`. It mainly works with object context only; returns a derived property or capability check. The body uses branching, tensor/kernel operations. Key calls include `current_platform.is_fp8_fnuz`, `torch.finfo`, `current_platform.fp8_dtype`.
**CN:** 定义函数 `get_fp8_min_max`，其签名为 `get_fp8_min_max() -> tuple[float, float]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含分支判断、张量或内核操作。关键调用包括 `current_platform.is_fp8_fnuz`, `torch.finfo`, `current_platform.fp8_dtype`。

### Function `scaled_quantize` (lines 286-338)
```python
def scaled_quantize(
    x: torch.Tensor,
    group_shape: GroupShape,
    quant_dtype: torch.dtype,
    compute_dtype: torch.dtype | None = None,
) -> tuple[torch.Tensor, torch.Tensor]:
    """
    Args:
        x: Input tensor to quantize
        group_shape: Shape of quantization groups
        quant_dtype: Target quantized dtype (e.g., torch.float8_e4m3fn)
        compute_dtype: Optional dtype for intermediate computations.
            If None, uses input dtype. Use torch.float32 for higher precision.
    """
    group_shape = _normalize_quant_group_shape(x, group_shape)
    assert quant_dtype.is_floating_point, (
        "currently `scaled_quantize` only supports floating point dtypes "
        "but could be extended to support other dtypes"
    )

    finfo = torch.finfo(quant_dtype)

    # Convert to compute dtype if specified
    x_compute = x if compute_dtype is None else x.to(compute_dtype)

    # Reshape (M, N) into (BLK_M, BLOCK_SIZE_M, BLK_N, BLOCK_SIZE_N)
    assert x.ndim == 2
    assert x.shape[0] % group_shape[0] == 0 and x.shape[1] % group_shape[1] == 0
    blk_m, blk_n = x.shape[0] // group_shape[0], x.shape[1] // group_shape[1]
    x_blkd = x_compute.reshape(blk_m, group_shape[0], blk_n, group_shape[1])

    # Permute to (BLK_M, BLK_N, BLOCK_SIZE_M, BLOCK_SIZE_N)
    x_blkd_permd = x_blkd.permute(0, 2, 1, 3)
    # Flatten to (BLK_M, BLK_N, BLOCK_SIZE_M * BLOCK_SIZE_N)
    x_blkd_permd = x_blkd_permd.flatten(start_dim=2)

    # Compute scales
    min_val, max_val = x_blkd_permd.aminmax(dim=-1)
    amax = torch.maximum(min_val.abs(), max_val.abs()).clamp(min=1e-12)
    _, fp8_max = get_fp8_min_max()
    scale = fp8_max / amax

    # Apply scale and convert from:
    # (BLK_M, BLK_N, BLOCK_SIZE_M * BLOCK_SIZE_N) to (M, N)
    x_scl_sat = (
        (x_blkd_permd * scale.unsqueeze(-1))
        .clamp(min=finfo.min, max=finfo.max)
        .reshape(blk_m, blk_n, group_shape[0], group_shape[1])
        .permute(0, 2, 1, 3)
        .reshape(x.shape)
    )

    return x_scl_sat.to(quant_dtype).contiguous(), scale.float().reciprocal()
```
**EN:** Defines function `scaled_quantize` with signature `scaled_quantize(x: torch.Tensor, group_shape: GroupShape, quant_dtype: torch.dtype, compute_dtype: torch.dtype | None=None) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `x`, `group_shape`, `quant_dtype`, `compute_dtype`; handles quantization-related transformation logic. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `_normalize_quant_group_shape`, `torch.finfo`, `x_compute.reshape`, `x_blkd.permute`, `x_blkd_permd.flatten`, `x_blkd_permd.aminmax`.
**CN:** 定义函数 `scaled_quantize`，其签名为 `scaled_quantize(x: torch.Tensor, group_shape: GroupShape, quant_dtype: torch.dtype, compute_dtype: torch.dtype | None=None) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `x`, `group_shape`, `quant_dtype`, `compute_dtype` 展开；处理量化相关的变换逻辑。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `_normalize_quant_group_shape`, `torch.finfo`, `x_compute.reshape`, `x_blkd.permute`, `x_blkd_permd.flatten`, `x_blkd_permd.aminmax`。

### Function `get_attribute_fallback` (lines 356-360)
```python
def get_attribute_fallback(obj, attributes: list[str]):
    for attr in attributes:
        if hasattr(obj, attr):
            return getattr(obj, attr)
    raise AttributeError(f"'{obj}' has no recognized attributes: {attributes}.")
```
**EN:** Defines function `get_attribute_fallback` with signature `get_attribute_fallback(obj, attributes: list[str])`. It mainly works with `obj`, `attributes`; returns a derived property or capability check. The body uses branching, iteration, validation/error handling. Key calls include `AttributeError`, `hasattr`, `getattr`.
**CN:** 定义函数 `get_attribute_fallback`，其签名为 `get_attribute_fallback(obj, attributes: list[str])`。它主要围绕 `obj`, `attributes` 展开；返回派生属性或能力判断结果。函数体包含分支判断、循环处理、校验或报错逻辑。关键调用包括 `AttributeError`, `hasattr`, `getattr`。

### Function `get_and_maybe_dequant_weights` (lines 363-418)
```python
def get_and_maybe_dequant_weights(
    layer: "LinearBase", out_dtype: torch.dtype = torch.float32
):
    """Return layer's unquantized weights in [out, in] layout"""
    from vllm.model_executor.layers.linear import UnquantizedLinearMethod
    from vllm.model_executor.layers.quantization.fp8 import Fp8LinearMethod

    # LoRA linear wrappers store quantization metadata on `base_layer`.
    # Unwrap here so callers can pass either a raw linear layer or its LoRA
    # wrapper without special-casing.
    while hasattr(layer, "base_layer") and hasattr(layer.base_layer, "quant_method"):
        layer = layer.base_layer

    weight = get_attribute_fallback(layer, ["weight", "qweight", "weight_packed"])

    # Unquantized layer: just return base weights
    if layer.quant_method is None or isinstance(
        layer.quant_method, UnquantizedLinearMethod
    ):
        return weight.to(out_dtype)

    # Simple Fp8 case: rescale with tensor or block weight scales
    if (
        isinstance(layer.quant_method, Fp8LinearMethod)
        and not layer.quant_method.use_marlin
        # DeepGEMM transforms the scales using `transform_sf_into_required_layout` into
        # a layout that is not compatible with `scaled_dequantize`.
        and not layer.quant_method.use_deep_gemm
    ):
        weight_scales = get_attribute_fallback(
            layer, ["weight_scale", "weight_scale_inv"]
        )
        dequant_weights = scaled_dequantize(
            weight,
            weight_scales,
            group_shape=layer.weight_block_size,
            out_dtype=out_dtype,
        )
        # per-tensor scaling stores weights in [in, out] layout
        if not layer.quant_method.block_quant:
            dequant_weights = dequant_weights.T
        return dequant_weights

    # NOTE: Most generic base case
    # - Call the layer with identity matrix which returns unquantized weights.
    # - Must be used with extra care when dealing with static activation quantization:
    #   quantizing 1.0 may lead to over/underflows
    # - Should only be used offline, since it's O(N^3)
    assert hasattr(layer, "input_size_per_partition")
    eye = torch.eye(
        layer.input_size_per_partition,
        dtype=out_dtype,
        device=weight.device,
    )
    dequant_weights = layer.quant_method.apply(layer, eye, bias=None).to(out_dtype)
    return dequant_weights.T
```
**EN:** Defines function `get_and_maybe_dequant_weights` with signature `get_and_maybe_dequant_weights(layer: 'LinearBase', out_dtype: torch.dtype=torch.float32)`. It mainly works with `layer`, `out_dtype`; handles quantization-related transformation logic. The body uses branching, iteration, validation/error handling, tensor/kernel operations. Key calls include `get_attribute_fallback`, `hasattr`, `torch.eye`, `layer.quant_method.apply.to`, `isinstance`, `weight.to`.
**CN:** 定义函数 `get_and_maybe_dequant_weights`，其签名为 `get_and_maybe_dequant_weights(layer: 'LinearBase', out_dtype: torch.dtype=torch.float32)`。它主要围绕 `layer`, `out_dtype` 展开；处理量化相关的变换逻辑。函数体包含分支判断、循环处理、校验或报错逻辑、张量或内核操作。关键调用包括 `get_attribute_fallback`, `hasattr`, `torch.eye`, `layer.quant_method.apply.to`, `isinstance`, `weight.to`。

### Function `is_layer_skipped` (lines 464-518)
```python
def is_layer_skipped(
    prefix: str,
    ignored_layers: list[str],
    fused_mapping: Mapping[str, list[str]] = MappingProxyType({}),
    *,
    skip_with_substr: bool = False,
) -> bool:
    def prefix_full_match(prefix: str, ignored_layers: list[str]) -> bool:
        return prefix in ignored_layers

    # For case like: ignored_layers = ["self_attn"]
    def substr_match(prefix: str, ignored_layers: list[str]) -> bool:
        return any(layer in prefix for layer in ignored_layers)

    match_func = substr_match if skip_with_substr else prefix_full_match

    # prefix: model.layers.0.self_attn.q_proj
    # proj_name: q_proj
    proj_name = prefix.split(".")[-1]

    # Fused layers like gate_up_proj or qkv_proj will not be fused
    # in the safetensors checkpoint. So, we convert the name
    # from the fused version to unfused + check to make sure that
    # each shard of the fused layer has the same scheme.
    if proj_name in fused_mapping:
        shard_prefixes = [
            prefix.replace(proj_name, shard_proj_name)
            for shard_proj_name in fused_mapping[proj_name]
        ]

        is_skipped = None
        for shard_prefix in shard_prefixes:
            is_shard_skipped = match_func(shard_prefix, ignored_layers)

            if is_skipped is None:
                is_skipped = is_shard_skipped
            elif is_shard_skipped != is_skipped:
                raise ValueError(
                    f"Detected some but not all shards of {prefix} "
                    "are quantized. All shards of fused layers "
                    "to have the same precision."
                )
    elif "experts" in prefix and not skip_with_substr:
        expert_ignore_layers = filter(
            lambda layer_name: "experts" in layer_name, ignored_layers
        )
        return any(
            prefix in layer_name if not skip_with_substr else layer_name in prefix
            for layer_name in expert_ignore_layers
        )
    else:
        is_skipped = match_func(prefix, ignored_layers)

    assert is_skipped is not None
    return is_skipped
```
**EN:** Defines function `is_layer_skipped` with signature `is_layer_skipped(prefix: str, ignored_layers: list[str], fused_mapping: Mapping[str, list[str]]=MappingProxyType({}), *, skip_with_substr: bool=False) -> bool`. It mainly works with `prefix`, `ignored_layers`, `fused_mapping`, `skip_with_substr`; returns a derived property or capability check. The body uses branching, iteration, comprehensions, validation/error handling. Key calls include `MappingProxyType`, `any`, `prefix.split`, `prefix.replace`, `match_func`, `filter`.
**CN:** 定义函数 `is_layer_skipped`，其签名为 `is_layer_skipped(prefix: str, ignored_layers: list[str], fused_mapping: Mapping[str, list[str]]=MappingProxyType({}), *, skip_with_substr: bool=False) -> bool`。它主要围绕 `prefix`, `ignored_layers`, `fused_mapping`, `skip_with_substr` 展开；返回派生属性或能力判断结果。函数体包含分支判断、循环处理、推导式、校验或报错逻辑。关键调用包括 `MappingProxyType`, `any`, `prefix.split`, `prefix.replace`, `match_func`, `filter`。

### Function `get_pack_factor` (lines 521-523)
```python
def get_pack_factor(num_bits):
    assert 32 % num_bits == 0, f"Unsupported num_bits = {num_bits}"
    return 32 // num_bits
```
**EN:** Defines function `get_pack_factor` with signature `get_pack_factor(num_bits)`. It mainly works with `num_bits`; returns a derived property or capability check. The body uses validation/error handling. Key calls include no notable helper calls.
**CN:** 定义函数 `get_pack_factor`，其签名为 `get_pack_factor(num_bits)`。它主要围绕 `num_bits` 展开；返回派生属性或能力判断结果。函数体包含校验或报错逻辑。关键调用包括 无显著辅助调用。

### Function `quantize_weights` (lines 556-648)
```python
def quantize_weights(
    w: torch.Tensor,
    quant_type: ScalarType,
    group_size: int | None,
    zero_points: bool = False,
    ref_zero_points_after_scales: bool = False,
):
    assert quant_type.is_integer(), (
        "Floating point quantization may work but has not been tested"
    )
    assert not zero_points or group_size is not None, (
        "to have group zero points, group_size must be provided "
        "(-1 group_size is channelwise)"
    )

    orig_device = w.device
    orig_type = w.dtype
    size_k, size_n = w.shape

    assert w.is_floating_point(), "w must be float"

    if group_size == -1:
        group_size = size_k

    # Reshape to [groupsize, -1]
    if group_size is not None and group_size < size_k:
        w = w.reshape((-1, group_size, size_n))
        w = w.permute(1, 0, 2)
        w = w.reshape((group_size, -1))

    # Compute scale for each group
    max_val = torch.max(w, 0, keepdim=True).values
    min_val = torch.min(w, 0, keepdim=True).values

    max_q_val = quant_type.max()
    min_q_val = quant_type.min()

    w_s = torch.Tensor([1.0]).to(w.device)  # unscaled case
# ... truncated for analysis ...
        w_q = reshape_w(w_q)
        w_ref = reshape_w(w_ref)
        w_s = w_s.reshape((-1, size_n)).contiguous()

    if maybe_w_zp is not None:
        maybe_w_zp = maybe_w_zp.reshape((-1, size_n)).contiguous()
        maybe_w_zp = maybe_w_zp.to(device=orig_device)

    return (
        w_ref.to(device=orig_device),
        w_q.to(device=orig_device),
        w_s if group_size is not None else None,
        maybe_w_zp,
    )
```
**EN:** Defines function `quantize_weights` with signature `quantize_weights(w: torch.Tensor, quant_type: ScalarType, group_size: int | None, zero_points: bool=False, ref_zero_points_after_scales: bool=False)`. It mainly works with `w`, `quant_type`, `group_size`, `zero_points`, `ref_zero_points_after_scales`; handles quantization-related transformation logic. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `quant_type.is_integer`, `w.is_floating_point`, `quant_type.max`, `quant_type.min`, `torch.Tensor.to`, `torch.clamp`.
**CN:** 定义函数 `quantize_weights`，其签名为 `quantize_weights(w: torch.Tensor, quant_type: ScalarType, group_size: int | None, zero_points: bool=False, ref_zero_points_after_scales: bool=False)`。它主要围绕 `w`, `quant_type`, `group_size`, `zero_points`, `ref_zero_points_after_scales` 展开；处理量化相关的变换逻辑。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `quant_type.is_integer`, `w.is_floating_point`, `quant_type.max`, `quant_type.min`, `torch.Tensor.to`, `torch.clamp`。

### Class `_GroupShape` overview (lines 39-41)
```python
class _GroupShape(NamedTuple):
    row: int
    col: int
```
**EN:** Defines class `_GroupShape` with base classes `NamedTuple` and decorators none. It acts as a quantization-oriented module building block and exposes 0 direct methods, with notable entries no direct methods.
**CN:** 定义类 `_GroupShape`，其基类为 `NamedTuple`，装饰器为 无。它在整体实现中充当面向量化的模块构件，并直接暴露 0 个方法，较重要的包括 无直接方法。

### Class `GroupShape` overview (lines 44-65)
```python
class GroupShape(_GroupShape):
    """
    This class describes the quantization group shape.
    It includes static members for common shapes (per-tensor, per-token).
    """

    # Aliases for common quantization group shapes
    PER_TENSOR: ClassVar["GroupShape"]
    PER_TOKEN: ClassVar["GroupShape"]
    PER_CHANNEL: ClassVar["GroupShape"]

    def is_per_tensor(self) -> bool:
        return self.row == -1 and self.col == -1

    def is_per_token(self) -> bool:
        return self.row == 1 and self.col == -1

    def is_per_channel(self) -> bool:
        return self.row == -1 and self.col == 1

    def is_per_group(self) -> bool:
        return self.row == 1 and self.col >= 1
```
**EN:** Defines class `GroupShape` with base classes `_GroupShape` and decorators none. It acts as a quantization-oriented module building block and exposes 4 direct methods, with notable entries `is_per_tensor`, `is_per_token`, `is_per_channel`, `is_per_group`. Its docstring says: This class describes the quantization group shape.
**CN:** 定义类 `GroupShape`，其基类为 `_GroupShape`，装饰器为 无。它在整体实现中充当面向量化的模块构件，并直接暴露 4 个方法，较重要的包括 `is_per_tensor`, `is_per_token`, `is_per_channel`, `is_per_group`。 文档字符串进一步说明了该类的定位。

### Method `GroupShape.is_per_tensor` (lines 55-56)
```python
    def is_per_tensor(self) -> bool:
        return self.row == -1 and self.col == -1
```
**EN:** Defines function `GroupShape.is_per_tensor` with signature `is_per_tensor(self) -> bool`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `GroupShape.is_per_tensor`，其签名为 `is_per_tensor(self) -> bool`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `GroupShape.is_per_token` (lines 58-59)
```python
    def is_per_token(self) -> bool:
        return self.row == 1 and self.col == -1
```
**EN:** Defines function `GroupShape.is_per_token` with signature `is_per_token(self) -> bool`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `GroupShape.is_per_token`，其签名为 `is_per_token(self) -> bool`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `GroupShape.is_per_channel` (lines 61-62)
```python
    def is_per_channel(self) -> bool:
        return self.row == -1 and self.col == 1
```
**EN:** Defines function `GroupShape.is_per_channel` with signature `is_per_channel(self) -> bool`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `GroupShape.is_per_channel`，其签名为 `is_per_channel(self) -> bool`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `GroupShape.is_per_group` (lines 64-65)
```python
    def is_per_group(self) -> bool:
        return self.row == 1 and self.col >= 1
```
**EN:** Defines function `GroupShape.is_per_group` with signature `is_per_group(self) -> bool`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `GroupShape.is_per_group`，其签名为 `is_per_group(self) -> bool`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Class `ScaleDesc` overview (lines 74-96)
```python
class ScaleDesc:
    """
    Class for describing a single quantization scaling factor.
    dtype: data type of the scale
    static: static scale if True, dynamic if False
    group_shape: group shape of the scale
    """

    dtype: torch.dtype
    static: bool
    group_shape: GroupShape

    def __str__(self):
        d = {
            GroupShape.PER_TENSOR: "per_tensor",
            GroupShape.PER_TOKEN: "per_token",
            GroupShape.PER_CHANNEL: "per_channel",
        }
        group_shape = d.get(self.group_shape, str(self.group_shape))
        return (
            f"{fx.graph.dtype_abbrs[self.dtype]},"
            f"{'static' if self.static else 'dynamic'},{group_shape}"
        )
```
**EN:** Defines class `ScaleDesc` with base classes no explicit base class and decorators `dataclass(frozen=True)`. It acts as a quantization-oriented module building block and exposes 1 direct methods, with notable entries `__str__`. Its docstring says: Class for describing a single quantization scaling factor.
**CN:** 定义类 `ScaleDesc`，其基类为 无显式基类，装饰器为 `dataclass(frozen=True)`。它在整体实现中充当面向量化的模块构件，并直接暴露 1 个方法，较重要的包括 `__str__`。 文档字符串进一步说明了该类的定位。

### Method `ScaleDesc.__str__` (lines 86-96)
```python
    def __str__(self):
        d = {
            GroupShape.PER_TENSOR: "per_tensor",
            GroupShape.PER_TOKEN: "per_token",
            GroupShape.PER_CHANNEL: "per_channel",
        }
        group_shape = d.get(self.group_shape, str(self.group_shape))
        return (
            f"{fx.graph.dtype_abbrs[self.dtype]},"
            f"{'static' if self.static else 'dynamic'},{group_shape}"
        )
```
**EN:** Defines function `ScaleDesc.__str__` with signature `__str__(self)`. It mainly works with object context only; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `d.get`, `str`.
**CN:** 定义函数 `ScaleDesc.__str__`，其签名为 `__str__(self)`。它主要围绕 仅依赖对象上下文 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `d.get`, `str`。

### Class `QuantKey` overview (lines 100-120)
```python
class QuantKey:
    """
    Class for identifying the type of quantization.
    dtype: quantized data type
    scale: scale descriptor
    scale2: second-level scale descriptor
    symmetric: symmetric if True, asymmetric if False
    """

    dtype: torch.dtype
    scale: ScaleDesc
    scale2: ScaleDesc | None = None
    symmetric: bool = True

    def __str__(self):
        scale2_str = f"scale2({self.scale2})," if self.scale2 else ""
        return (
            f"QuantKey({fx.graph.dtype_abbrs[self.dtype]},"
            f"scale({self.scale}),{scale2_str}"
            f"{'a' if not self.symmetric else ''}symmetric)"
        )
```
**EN:** Defines class `QuantKey` with base classes no explicit base class and decorators `dataclass(frozen=True)`. It acts as a quantization-oriented module building block and exposes 1 direct methods, with notable entries `__str__`. Its docstring says: Class for identifying the type of quantization.
**CN:** 定义类 `QuantKey`，其基类为 无显式基类，装饰器为 `dataclass(frozen=True)`。它在整体实现中充当面向量化的模块构件，并直接暴露 1 个方法，较重要的包括 `__str__`。 文档字符串进一步说明了该类的定位。

### Method `QuantKey.__str__` (lines 114-120)
```python
    def __str__(self):
        scale2_str = f"scale2({self.scale2})," if self.scale2 else ""
        return (
            f"QuantKey({fx.graph.dtype_abbrs[self.dtype]},"
            f"scale({self.scale}),{scale2_str}"
            f"{'a' if not self.symmetric else ''}symmetric)"
        )
```
**EN:** Defines function `QuantKey.__str__` with signature `__str__(self)`. It mainly works with object context only; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include no notable helper calls.
**CN:** 定义函数 `QuantKey.__str__`，其签名为 `__str__(self)`。它主要围绕 仅依赖对象上下文 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 无显著辅助调用。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `_GroupShape`, `GroupShape`, `ScaleDesc`, `QuantKey`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `_GroupShape`, `GroupShape`, `ScaleDesc`, `QuantKey`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `get_fp8_min_max`, `create_fp8_quant_key`, `_normalize_quant_group_shape`, `group_broadcast`, `prep_scale_for_group_broadcast` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `get_fp8_min_max`, `create_fp8_quant_key`, `_normalize_quant_group_shape`, `group_broadcast`, `prep_scale_for_group_broadcast` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `collections`, `dataclasses`, `types`, `typing`, `numpy`, `torch`
- **Internal / 内部**: `vllm.platforms`, `vllm.scalar_type`
