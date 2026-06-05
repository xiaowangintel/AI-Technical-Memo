# mxfp4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/mxfp4.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines configuration objects and method-selection logic for mxfp4 quantization in SGLang SRT. / 该模块定义了用于 SGLang SRT 的 MXFP4 量化配置对象与方法选择逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: module imports and setup
```python
# SPDX-FileCopyrightText: Copyright (c) 2024 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# SPDX-License-Identifier: Apache-2.0
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# Adapted from https://github.com/vllm-project/vllm/blob/main/vllm/model_executor/layers/quantization/mxfp4.py

from __future__ import annotations

import os
from dataclasses import replace
from typing import TYPE_CHECKING, List, Optional

import torch
from torch.nn.parameter import Parameter
```
**EN:** This block imports __future__, aiter.ops.shuffle, aiter.ops.triton.quant, aiter.utility.fp4_utils, sglang.srt.distributed, sglang.srt.distributed.device_communicators.pynccl_allocator, sglang.srt.layers.dp_attention, sglang.srt.layers.linear and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, aiter.ops.shuffle, aiter.ops.triton.quant, aiter.utility.fp4_utils, sglang.srt.distributed, sglang.srt.distributed.device_communicators.pynccl_allocator, sglang.srt.layers.dp_attention, sglang.srt.layers.linear 等依赖，并为当前量化实现准备模块命名空间。

### Lines 30-30: expr block
```python
os.environ.setdefault("TLLM_LOG_LEVEL", "INFO")
```
**EN:** This block contributes supporting control flow or state used by the surrounding quantization logic.
**CN:** 该代码块为周围量化逻辑提供支撑性的控制流或状态。

### Lines 32-62: module imports and setup
```python
from sglang.srt.distributed import get_tp_group
from sglang.srt.distributed.device_communicators.pynccl_allocator import (
    use_symmetric_memory,
)
from sglang.srt.layers.dp_attention import is_allocation_symmetric
from sglang.srt.layers.moe import MoeRunner, MoeRunnerBackend, MoeRunnerConfig
from sglang.srt.layers.moe.moe_runner.marlin import MarlinMoeQuantInfo
from sglang.srt.layers.moe.moe_runner.triton import TritonMoeQuantInfo
from sglang.srt.layers.moe.utils import get_moe_a2a_backend, get_moe_runner_backend
from sglang.srt.layers.quantization.base_config import (
    FusedMoEMethodBase,
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.srt.layers.quantization.utils import is_layer_skipped
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import (
    is_flashinfer_available,
    is_gfx95_supported,
    is_hip,
    is_sm90_supported,
    is_sm100_supported,
    is_sm120_supported,
    is_triton_kernels_available,
    mxfp_supported,
    next_power_of_2,
    round_up,
    set_weight_attrs,
)
from sglang.srt.utils.common import get_bool_env_var
from sglang.srt.utils.custom_op import register_custom_op
```
**EN:** This block imports __future__, aiter.ops.shuffle, aiter.ops.triton.quant, aiter.utility.fp4_utils, sglang.srt.distributed, sglang.srt.distributed.device_communicators.pynccl_allocator, sglang.srt.layers.dp_attention, sglang.srt.layers.linear and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, aiter.ops.shuffle, aiter.ops.triton.quant, aiter.utility.fp4_utils, sglang.srt.distributed, sglang.srt.distributed.device_communicators.pynccl_allocator, sglang.srt.layers.dp_attention, sglang.srt.layers.linear 等依赖，并为当前量化实现准备模块命名空间。

### Lines 64-64: initialize has_triton_kernels
```python
has_triton_kernels = is_triton_kernels_available()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as has_triton_kernels.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 has_triton_kernels。

### Lines 67-93: conditional logic for is_flashinfer_available()
```python
if is_flashinfer_available():
    from flashinfer import (
        mxfp8_quantize,
        nvfp4_block_scale_interleave,
        trtllm_fp4_block_scale_moe,
    )
    from flashinfer.fused_moe import cutlass_fused_moe as flashinfer_cutlass_fused_moe
    from flashinfer.fused_moe.core import (
        ActivationType,
        get_w2_permute_indices_with_cache,
    )

    # SM90 mixed-input helpers landed in FlashInfer #3084 (post-0.6.10). Older
    # versions don't ship them; gate at import so unrelated code paths still load.
    try:
        from flashinfer.fused_moe import (
            interleave_moe_scales_for_sm90_mixed_gemm,
            interleave_moe_weights_for_sm90_mixed_gemm,
        )

        _FI_HAS_SM90_CUTLASS_MXFP4 = True
    except ImportError:
        interleave_moe_scales_for_sm90_mixed_gemm = None
        interleave_moe_weights_for_sm90_mixed_gemm = None
        _FI_HAS_SM90_CUTLASS_MXFP4 = False
else:
    _FI_HAS_SM90_CUTLASS_MXFP4 = False
```
**EN:** This block applies conditional logic controlled by `is_flashinfer_available()`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `is_flashinfer_available()` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 95-95: annotate _flashinfer_mxfp4_permute_indices_cache
```python
_flashinfer_mxfp4_permute_indices_cache: dict[torch.Size, torch.Tensor] = {}
```
**EN:** This block declares and initializes the annotated symbol `_flashinfer_mxfp4_permute_indices_cache`.
**CN:** 该代码块声明并初始化带类型注解的符号 `_flashinfer_mxfp4_permute_indices_cache`。

### Lines 96-98: annotate _flashinfer_mxfp4_permute_indices_device_cache
```python
_flashinfer_mxfp4_permute_indices_device_cache: dict[
    tuple[tuple[int, ...], int, int, str, int], torch.Tensor
] = {}
```
**EN:** This block declares and initializes the annotated symbol `_flashinfer_mxfp4_permute_indices_device_cache`.
**CN:** 该代码块声明并初始化带类型注解的符号 `_flashinfer_mxfp4_permute_indices_device_cache`。

### Lines 101-132: _get_flashinfer_mxfp4_device_permute_indices()
```python
def _get_flashinfer_mxfp4_device_permute_indices(
    x: torch.Tensor,
    epilogue_tile_m: int,
    num_elts_per_sf: Optional[int] = None,
) -> torch.Tensor:
    extra_args = {} if num_elts_per_sf is None else {"num_elts_per_sf": num_elts_per_sf}
    permute_indices = get_w2_permute_indices_with_cache(
        _flashinfer_mxfp4_permute_indices_cache,
        x,
        epilogue_tile_m,
        **extra_args,
    )

    device_index = -1 if x.device.index is None else x.device.index
    num_elts_per_sf_key = -1 if num_elts_per_sf is None else num_elts_per_sf
    cache_key = (
        tuple(x.shape),
        epilogue_tile_m,
        num_elts_per_sf_key,
        x.device.type,
        device_index,
    )
    cached_device_indices = _flashinfer_mxfp4_permute_indices_device_cache.get(
        cache_key
    )
    if cached_device_indices is None:
        cached_device_indices = permute_indices.to(x.device)
        _flashinfer_mxfp4_permute_indices_device_cache[cache_key] = (
            cached_device_indices
        )

    return cached_device_indices
```
**EN:** This block defines `_get_flashinfer_mxfp4_device_permute_indices()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `_get_flashinfer_mxfp4_device_permute_indices()`，用于实现量化栈中的可复用模块逻辑。

### Lines 135-139: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        StandardDispatchOutput,
    )
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 141-141: initialize _is_hip
```python
_is_hip = is_hip()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_hip.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_hip。

### Lines 142-142: initialize _use_aiter
```python
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _use_aiter.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _use_aiter。

### Lines 143-143: initialize _is_shuffle_moe_mxfp4
```python
_is_shuffle_moe_mxfp4 = is_gfx95_supported()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_shuffle_moe_mxfp4.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_shuffle_moe_mxfp4。

### Lines 144-144: initialize _sm120_mxfp4_min_warps_patched
```python
_sm120_mxfp4_min_warps_patched = False
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _sm120_mxfp4_min_warps_patched.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _sm120_mxfp4_min_warps_patched。

### Lines 146-157: conditional logic for _is_hip
```python
if _is_hip:
    # import aiter
    try:
        from aiter.ops.shuffle import (
            shuffle_scale_a16w4,
            shuffle_weight,
            shuffle_weight_a16w4,
        )
        from aiter.ops.triton.quant import dynamic_mxfp4_quant
        from aiter.utility.fp4_utils import e8m0_shuffle
    except ImportError as err:
        dynamic_mxfp4_quant = e8m0_shuffle = err
```
**EN:** This block applies conditional logic controlled by `_is_hip`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `_is_hip` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 160-200: _patch_sm120_mxfp4_min_warps()
```python
def _patch_sm120_mxfp4_min_warps():
    global _sm120_mxfp4_min_warps_patched
    if _sm120_mxfp4_min_warps_patched:
        return

    import inspect

    from triton_kernels.matmul_ogs_details.opt_flags_details import opt_flags_nvidia
    from triton_kernels.tensor import get_layout
    from triton_kernels.tensor_details.layout import StridedLayout

    compute_num_warps = opt_flags_nvidia.compute_num_warps
    params = inspect.signature(compute_num_warps).parameters

    if "is_persistent" in params and not getattr(
        compute_num_warps, "_sglang_sm120_mxfp4_patch", False
    ):

        def _compute_num_warps_sm120_mxfp4(
            block_m, block_n, is_persistent, precision_config
        ):
            selected_num_warps = compute_num_warps(
                block_m, block_n, is_persistent, precision_config
            )
            weight_scale = getattr(precision_config, "weight_scale", None)
            weight_scale_layout = get_layout(weight_scale)
            if (
                not is_persistent
                and weight_scale is not None
                and (
                    weight_scale_layout is StridedLayout
                    or isinstance(weight_scale_layout, StridedLayout)
                )
            ):
                return max(selected_num_warps, 4)
            return selected_num_warps

        _compute_num_warps_sm120_mxfp4._sglang_sm120_mxfp4_patch = True
        opt_flags_nvidia.compute_num_warps = _compute_num_warps_sm120_mxfp4

    _sm120_mxfp4_min_warps_patched = True
```
**EN:** This block defines `_patch_sm120_mxfp4_min_warps()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `_patch_sm120_mxfp4_min_warps()`，用于实现量化栈中的可复用模块逻辑。

### Lines 203-252: _swizzle_mxfp4()
```python
def _swizzle_mxfp4(quant_tensor, scale, num_warps):
    """weight swizzle for mxfp4 moe, used for OAI mxfp4 kernel"""
    import triton_kernels.matmul_ogs_details.opt_flags as opt_flags
    from triton_kernels.numerics import InFlexData
    from triton_kernels.tensor import FP4, convert_layout, wrap_torch_tensor
    from triton_kernels.tensor_details import layout

    if is_sm120_supported():
        # SM120 desktop Blackwell does not support the persistent/TMA MXFP4 path.
        # This MXFP4 path uses StridedLayout and the non-persistent kernel.
        _patch_sm120_mxfp4_min_warps()
        from triton_kernels.tensor_details.layout import StridedLayout

        value_layout = StridedLayout
        value_layout_opts = {}
        scale_layout = StridedLayout
        scale_layout_opts = {}
        constraints = {
            "is_persistent": False,
            "num_stages": 1,
        }
        opt_flags.update_opt_flags_constraints(constraints)
    else:
        value_layout, value_layout_opts = layout.make_default_matmul_mxfp4_w_layout(
            mx_axis=1
        )
        scale_layout, scale_layout_opts = (
            layout.make_default_matmul_mxfp4_w_scale_layout(
                mx_axis=1, num_warps=num_warps
            )
        )
        if is_sm100_supported():
            constraints = {
                "is_persistent": True,
                "epilogue_subtile": 1,
            }
            opt_flags.update_opt_flags_constraints(constraints)
        elif is_sm90_supported():
            constraints = {
                "split_k": 1,
            }
            opt_flags.update_opt_flags_constraints(constraints)
    # transpose the tensor so that the quantization axis is on dim1
    quant_tensor = quant_tensor.transpose(-2, -1)
    scale = scale.transpose(-2, -1)
    quant_tensor = convert_layout(
        wrap_torch_tensor(quant_tensor, dtype=FP4), value_layout, **value_layout_opts
    )
    scale = convert_layout(wrap_torch_tensor(scale), scale_layout, **scale_layout_opts)
    return quant_tensor, InFlexData(), scale
```
**EN:** This block defines `_swizzle_mxfp4()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `_swizzle_mxfp4()`，用于实现量化栈中的可复用模块逻辑。

### Lines 255-260: _dequant_mxfp4_fake()
```python
def _dequant_mxfp4_fake(
    x: torch.Tensor, scale: torch.Tensor, float_dtype: torch.dtype
) -> torch.Tensor:
    return torch.empty(
        (*x.shape[:-1], x.shape[-1] * 2), dtype=float_dtype, device=x.device
    )
```
**EN:** This block defines `_dequant_mxfp4_fake()`, which restores low-precision data for downstream compute.
**CN:** 该代码块定义了 `_dequant_mxfp4_fake()`，用于为后续计算恢复低精度数据。

### Lines 263-276: dequant_mxfp4()
```python
@register_custom_op(fake_impl=_dequant_mxfp4_fake)
def dequant_mxfp4(
    x: torch.Tensor, scale: torch.Tensor, float_dtype: torch.dtype
) -> torch.Tensor:
    try:
        from quark.torch.kernel import mx
    except ImportError as err:
        raise ImportError(
            "The package `amd-quark` is required to use "
            "MX-FP4 models. Please install it with `pip install "
            "amd-quark`."
        ) from err

    return mx.dq_mxfp4(x, scale, float_dtype)
```
**EN:** This block defines `dequant_mxfp4()`, which restores low-precision data for downstream compute.
**CN:** 该代码块定义了 `dequant_mxfp4()`，用于为后续计算恢复低精度数据。

### Lines 279-292: quant_dequant_mxfp4()
```python
@register_custom_op(out_shape="x")
def quant_dequant_mxfp4(
    x: torch.Tensor, scale_calculation_mode: str = "even"
) -> torch.Tensor:
    try:
        from quark.torch.kernel import mx
    except ImportError as err:
        raise ImportError(
            "The package `amd-quark` is required to use "
            "MX-FP4 models. Please install it with `pip install "
            "amd-quark`."
        ) from err

    return mx.qdq_mxfp4(x, scale_calculation_mode)
```
**EN:** This block defines `quant_dequant_mxfp4()`, which restores low-precision data for downstream compute.
**CN:** 该代码块定义了 `quant_dequant_mxfp4()`，用于为后续计算恢复低精度数据。

### Lines 295-296: class Mxfp4Config: definition
```python
class Mxfp4Config(QuantizationConfig):
```
**EN:** This block declares `Mxfp4Config`, a configuration class for the quantization stack. It organizes behaviors such as __init__, from_config, get_min_capability, get_name.
**CN:** 该代码块声明 `Mxfp4Config`，它是量化栈中的配置类，组织了 __init__, from_config, get_min_capability, get_name 等行为。

### Lines 297-304: Mxfp4Config.__init__()
```python
    def __init__(
        self,
        ignored_layers: Optional[list[str]] = None,
        is_checkpoint_mxfp4_serialized: bool = False,
    ):
        super().__init__()
        self.is_checkpoint_mxfp4_serialized = is_checkpoint_mxfp4_serialized
        self.ignored_layers = ignored_layers
```
**EN:** This block defines `Mxfp4Config.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `Mxfp4Config.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 306-324: Mxfp4Config.from_config()
```python
    @classmethod
    def from_config(cls, config):

        quant_method = cls.get_from_keys(config, ["quant_method"])
        is_checkpoint_mxfp4_serialized = "mxfp4" in quant_method

        if _is_hip:
            if mxfp_supported():
                return cls(
                    is_checkpoint_mxfp4_serialized=is_checkpoint_mxfp4_serialized
                )
            else:

                platform = torch.cuda.get_device_properties(0).gcnArchName
                raise ValueError(
                    f"Current platform {platform} not support mxfp4 computation"
                )

        return cls(is_checkpoint_mxfp4_serialized=is_checkpoint_mxfp4_serialized)
```
**EN:** This block defines `Mxfp4Config.from_config()`, which constructs an object from serialized configuration data.
**CN:** 该代码块定义了 `Mxfp4Config.from_config()`，用于从序列化配置数据构造对象。

### Lines 326-328: Mxfp4Config.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 80
```
**EN:** This block defines `Mxfp4Config.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `Mxfp4Config.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 330-332: Mxfp4Config.get_name()
```python
    @classmethod
    def get_name(cls) -> str:
        return "mxfp4"
```
**EN:** This block defines `Mxfp4Config.get_name()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `Mxfp4Config.get_name()`，用于为调用方获取或计算派生值。

### Lines 334-336: Mxfp4Config.get_supported_act_dtypes()
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> list[torch.dtype]:
        return [torch.bfloat16, torch.float16]
```
**EN:** This block defines `Mxfp4Config.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `Mxfp4Config.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 338-340: Mxfp4Config.get_config_filenames()
```python
    @classmethod
    def get_config_filenames(cls) -> list[str]:
        return []
```
**EN:** This block defines `Mxfp4Config.get_config_filenames()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `Mxfp4Config.get_config_filenames()`，用于为调用方获取或计算派生值。

### Lines 342-343: Mxfp4Config.is_static_cfg()
```python
    def is_static_cfg(self):
        return self.is_checkpoint_mxfp4_serialized
```
**EN:** This block defines `Mxfp4Config.is_static_cfg()`, which checks a condition used by higher-level control flow.
**CN:** 该代码块定义了 `Mxfp4Config.is_static_cfg()`，用于检查供上层控制流使用的条件。

### Lines 345-370: Mxfp4Config.get_quant_method()
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Optional["QuantizeMethodBase"]:

        from sglang.srt.layers.linear import LinearBase
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoE
        from sglang.srt.layers.quantization.unquant import UnquantizedLinearMethod

        if isinstance(layer, LinearBase):
            if self.ignored_layers and is_layer_skipped(
                prefix=prefix,
                ignored_layers=self.ignored_layers,
                fused_mapping=self.packed_modules_mapping,
            ):
                return UnquantizedLinearMethod()
            elif _is_hip:
                return UnquantizedLinearMethod()
        elif isinstance(layer, FusedMoE):
            if self.is_checkpoint_mxfp4_serialized:
                return Mxfp4MoEMethod(prefix=prefix)
            else:
                return Mxfp4DynamicQuantMoEMethod()
        else:
            if self.is_checkpoint_mxfp4_serialized:
                raise NotImplementedError("Mxfp4 attention layer is not implemented")
        return None
```
**EN:** This block defines `Mxfp4Config.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `Mxfp4Config.get_quant_method()`，用于为调用方获取或计算派生值。

### Lines 372-373: Mxfp4Config.get_scaled_act_names()
```python
    def get_scaled_act_names(self) -> List[str]:
        return []
```
**EN:** This block defines `Mxfp4Config.get_scaled_act_names()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `Mxfp4Config.get_scaled_act_names()`，用于为调用方获取或计算派生值。

### Lines 376-377: class Mxfp4MoEMethod: definition
```python
class Mxfp4MoEMethod(FusedMoEMethodBase):
```
**EN:** This block declares `Mxfp4MoEMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, _process_weights_for_sm90_cutlass.
**CN:** 该代码块声明 `Mxfp4MoEMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, process_weights_after_loading, _process_weights_for_sm90_cutlass 等行为。

### Lines 378-414: Mxfp4MoEMethod.__init__()
```python
    def __init__(
        self,
        prefix: str,
    ):
        super().__init__()

        self.prefix = prefix
        self.topk_indices_dtype = None
        self.use_triton_kernels = get_moe_runner_backend().is_triton_kernels()
        self.with_bias = False
        self.use_flashinfer = get_moe_runner_backend().is_flashinfer_mxfp4()
        self.use_marlin = get_moe_runner_backend().is_marlin()
        self.flashinfer_mxfp4_moe_precision = (
            get_global_server_args().flashinfer_mxfp4_moe_precision
        )
        # When `flashinfer_mxfp4` is enabled, dispatch to one of two FlashInfer
        # entry points depending on the GPU:
        #   - SM100 (Blackwell)  -> trtllm_fp4_block_scale_moe (existing)
        #   - SM90  (Hopper)     -> cutlass_fused_moe(use_w4_group_scaling=True)
        #                           (FlashInfer PR #3084, post-0.6.10)
        self._fi_kernel: Optional[str] = None
        if self.use_flashinfer:
            if is_sm100_supported():
                self._fi_kernel = "trtllm_sm100"
            elif is_sm90_supported():
                if not _FI_HAS_SM90_CUTLASS_MXFP4:
                    raise RuntimeError(
                        "moe_runner_backend=flashinfer_mxfp4 on SM90 requires the "
                        "interleave_moe_{weights,scales}_for_sm90_mixed_gemm helpers "
                        "from FlashInfer PR #3084 (>= 0.6.11). Upgrade flashinfer-python "
                        "or pick a different backend (e.g. marlin / triton_kernel)."
                    )
                self._fi_kernel = "cutlass_sm90"
            else:
                raise NotImplementedError(
                    "moe_runner_backend=flashinfer_mxfp4 requires SM90 or SM100."
                )
```
**EN:** This block defines `Mxfp4MoEMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `Mxfp4MoEMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 416-460: Mxfp4MoEMethod.create_weights() (part 1/4)
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        with_bias: bool = False,
        **extra_weight_attrs,
    ):
        self.num_experts = num_experts
        weight_dtype = torch.uint8
        scale_dtype = torch.uint8
        self.with_bias = with_bias
        mxfp4_block = 32
        triton_kernels_padding_alignment = 64

        # pad the intermediate size to be a multiple of 2 * mxfp4_block
        # for to hold non-uniform sharded tensor as well as swizzling
        intermediate_size_per_partition_after_pad = intermediate_size_per_partition
        if is_sm100_supported():
            if self.use_flashinfer:
                intermediate_size_per_partition_after_pad = round_up(
                    intermediate_size_per_partition, 256
                )
                hidden_size = round_up(hidden_size, 256)
            else:
                intermediate_size_per_partition_after_pad = round_up(
                    intermediate_size_per_partition, triton_kernels_padding_alignment
                )
        elif self._fi_kernel == "cutlass_sm90":
            # cutlass mixed-input GEMM contraction dim K must be % 128 == 0
            # (interleave factor for MXFP4 group_size=32 is 4). The kernel
            # also expects ``fc1_expert_weights`` in halved ``[up; gate]``
            # layout, which means the padding boundary must fall on the
            # gate / up split.
            #
            # The mxfp4 weight loader (FusedMoE.weight_loader fast path) does
            # a NAIVE copy of HF's ``[2*intermediate_size, hidden_packed]``
            # tensor into the buffer's ``[:dim1, :dim2]`` slice. Padding the
            # buffer here would push the gate/up boundary, so HF's "up"
            # rows would land in the buffer's "gate" half and vice versa.
            # Marlin sidesteps this by not padding; we do the same and
            # rebuild a properly-padded buffer in
            # ``_process_weights_for_sm90_cutlass`` after the load completes.
```
**EN:** This segment of `Mxfp4MoEMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `Mxfp4MoEMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 461-505: Mxfp4MoEMethod.create_weights() (part 2/4)
```python
            self._padded_intermediate = round_up(intermediate_size_per_partition, 128)
            self._padded_hidden = round_up(hidden_size, 128)
            # create_weights below uses the *unpadded* sizes so the loader's
            # naive-copy fast path is correct.
            intermediate_size_per_partition_after_pad = intermediate_size_per_partition
        elif _use_aiter:

            intermediate_size_per_partition_after_pad = round_up(
                intermediate_size_per_partition, 256
            )

            hidden_size = round_up(hidden_size, 256)
            self.hidden_pad = hidden_size - layer.hidden_size
            self.intermediate_pad = (
                intermediate_size_per_partition_after_pad
                - layer.intermediate_size_per_partition
            )
        elif has_triton_kernels:
            intermediate_size_per_partition_after_pad = round_up(
                intermediate_size_per_partition, triton_kernels_padding_alignment
            )

        self.intermediate_size_per_partition = intermediate_size_per_partition_after_pad

        self.hidden_size = hidden_size
        # Fused gate_up_proj (column parallel)
        w13_weight = torch.nn.Parameter(
            torch.zeros(
                layer.num_local_experts,
                2 * intermediate_size_per_partition_after_pad,
                hidden_size // 2,
                dtype=weight_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight", w13_weight)
        set_weight_attrs(w13_weight, extra_weight_attrs)

        w13_weight_scale = torch.nn.Parameter(
            torch.zeros(
                layer.num_local_experts,
                2 * intermediate_size_per_partition_after_pad,
                hidden_size // mxfp4_block,
                dtype=scale_dtype,
            ),
```
**EN:** This segment of `Mxfp4MoEMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `Mxfp4MoEMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 506-550: Mxfp4MoEMethod.create_weights() (part 3/4)
```python
            requires_grad=False,
        )
        layer.register_parameter("w13_weight_scale", w13_weight_scale)
        set_weight_attrs(w13_weight_scale, extra_weight_attrs)

        w13_weight_bias = torch.nn.Parameter(
            torch.zeros(
                layer.num_local_experts,
                2 * intermediate_size_per_partition_after_pad,
                dtype=torch.bfloat16,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight_bias", w13_weight_bias)
        set_weight_attrs(w13_weight_bias, extra_weight_attrs)

        # down_proj (row parallel)
        w2_weight = torch.nn.Parameter(
            torch.zeros(
                layer.num_local_experts,
                hidden_size,
                intermediate_size_per_partition_after_pad // 2,
                dtype=weight_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight", w2_weight)
        set_weight_attrs(w2_weight, extra_weight_attrs)

        w2_weight_scale = torch.nn.Parameter(
            torch.zeros(
                layer.num_local_experts,
                hidden_size,
                intermediate_size_per_partition_after_pad // mxfp4_block,
                dtype=scale_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight_scale", w2_weight_scale)
        set_weight_attrs(w2_weight_scale, extra_weight_attrs)

        w2_weight_bias = torch.nn.Parameter(
            torch.zeros(layer.num_local_experts, hidden_size, dtype=torch.bfloat16),
            requires_grad=False,
        )
```
**EN:** This segment of `Mxfp4MoEMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `Mxfp4MoEMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 551-552: Mxfp4MoEMethod.create_weights() (part 4/4)
```python
        layer.register_parameter("w2_weight_bias", w2_weight_bias)
        set_weight_attrs(w2_weight_bias, extra_weight_attrs)
```
**EN:** This segment of `Mxfp4MoEMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `Mxfp4MoEMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 554-598: Mxfp4MoEMethod.process_weights_after_loading() (part 1/8)
```python
    def process_weights_after_loading(self, layer):
        if self.use_marlin:
            from sglang.srt.layers.quantization.marlin_utils import (
                check_moe_marlin_supports_layer,
            )
            from sglang.srt.layers.quantization.marlin_utils_fp4 import (
                prepare_moe_mxfp4_layer_for_marlin,
            )

            if not is_sm90_supported():
                raise RuntimeError("MXFP4 Marlin requires Hopper/SM90 or above.")
            if not check_moe_marlin_supports_layer(layer, 32):
                raise RuntimeError(
                    "Current MXFP4 MoE layer is not supported by Marlin."
                )

            prepare_moe_mxfp4_layer_for_marlin(layer)
            layer._mxfp4_backend = "marlin"
            return

        if self._fi_kernel == "cutlass_sm90":
            self._process_weights_for_sm90_cutlass(layer)
            return
        if self.use_flashinfer:
            # TODO: these values are hardcoded for now, we need to get them from the model
            layer.gemm1_alpha = Parameter(
                torch.tensor([1.702] * self.num_experts, dtype=torch.float32).cuda(),
                requires_grad=False,
            )
            layer.gemm1_beta = Parameter(
                torch.tensor([1.0] * self.num_experts, dtype=torch.float32).cuda(),
                requires_grad=False,
            )
            layer.gemm1_clamp_limit = Parameter(
                torch.tensor([7.0] * self.num_experts, dtype=torch.float32).cuda(),
                requires_grad=False,
            )
            sf_block_size = 32  # mxfp4 block size

            assert (
                layer.w13_weight.dim() == 3
                and layer.w13_weight.shape[0] == self.num_experts
                and layer.w13_weight.shape[1]
                == self.intermediate_size_per_partition * 2
                and layer.w13_weight.shape[2] == self.hidden_size // 2
```
**EN:** This segment of `Mxfp4MoEMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `Mxfp4MoEMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 599-643: Mxfp4MoEMethod.process_weights_after_loading() (part 2/8)
```python
            )
            assert (
                layer.w13_weight_scale.dim() == 3
                and layer.w13_weight_scale.shape[0] == self.num_experts
                and layer.w13_weight_scale.shape[1]
                == self.intermediate_size_per_partition * 2
                and layer.w13_weight_scale.shape[2] == self.hidden_size // sf_block_size
            )
            assert (
                layer.w2_weight.dim() == 3
                and layer.w2_weight.shape[0] == self.num_experts
                and layer.w2_weight.shape[1] == self.hidden_size
                and layer.w2_weight.shape[2]
                == self.intermediate_size_per_partition // 2
            )
            assert (
                layer.w2_weight_scale.dim() == 3
                and layer.w2_weight_scale.shape[1] == self.hidden_size
                and layer.w2_weight_scale.shape[2]
                == self.intermediate_size_per_partition // sf_block_size
            )
            assert (
                layer.w13_weight_bias.dim() == 2
                and layer.w13_weight_bias.shape[0] == self.num_experts
                and layer.w13_weight_bias.shape[1]
                == self.intermediate_size_per_partition * 2
            )
            assert (
                layer.w2_weight_bias.dim() == 2
                and layer.w2_weight_bias.shape[0] == self.num_experts
                and layer.w2_weight_bias.shape[1] == self.hidden_size
            )

            w13_weight_scale = layer.w13_weight_scale.data
            w2_weight_scale = layer.w2_weight_scale.data
            w13_weight = layer.w13_weight.data
            w2_weight = layer.w2_weight.data
            w13_bias = layer.w13_weight_bias.data.to(torch.float32)
            w2_bias = layer.w2_weight_bias.data.to(torch.float32)

            # Swap w1 and w3 as the definition of
            # swiglu is different in the trtllm-gen
            def swap_every_two_rows(x, axis=-1):
                shape = x.shape
                if axis < 0:
```
**EN:** This segment of `Mxfp4MoEMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `Mxfp4MoEMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 644-688: Mxfp4MoEMethod.process_weights_after_loading() (part 3/8)
```python
                    axis = len(shape) + axis

                # Create a new shape with pairs swapped along specified axis
                new_shape = list(shape)
                new_shape[axis] = shape[axis] // 2
                new_shape.insert(axis + 1, 2)

                # Reshape to expose pairs, swap them, and reshape back
                x = x.reshape(*new_shape)
                x = x.flip(axis + 1)
                new_shape = list(shape)
                return x.reshape(*new_shape)

            w13_weight_scale = swap_every_two_rows(w13_weight_scale, -2)
            w13_weight = swap_every_two_rows(w13_weight, -2)
            w13_bias = swap_every_two_rows(w13_bias, -1)

            # Shuffle weights and scaling factors for transposed mma output
            gemm1_weights_mxfp4_shuffled = []
            gemm1_scales_mxfp4_shuffled = []
            gemm2_weights_mxfp4_shuffled = []
            gemm2_scales_mxfp4_shuffled = []
            gemm1_bias_shuffled = []
            gemm2_bias_shuffled = []
            epilogue_tile_m = 128  # FIXME: this depends on the kernel internals
            w13_weight_permute_indices = _get_flashinfer_mxfp4_device_permute_indices(
                w13_weight[0].view(torch.uint8),
                epilogue_tile_m,
            )
            w13_scale_permute_indices = _get_flashinfer_mxfp4_device_permute_indices(
                w13_weight_scale[0].view(torch.uint8),
                epilogue_tile_m,
                num_elts_per_sf=16,
            )
            w13_bias_permute_indices = _get_flashinfer_mxfp4_device_permute_indices(
                w13_bias[0].reshape(-1, 1),
                epilogue_tile_m,
            )

            w2_weight_permute_indices = _get_flashinfer_mxfp4_device_permute_indices(
                w2_weight[0].view(torch.uint8),
                epilogue_tile_m,
            )
            w2_scale_permute_indices = _get_flashinfer_mxfp4_device_permute_indices(
                w2_weight_scale[0].view(torch.uint8),
```
**EN:** This segment of `Mxfp4MoEMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `Mxfp4MoEMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 689-733: Mxfp4MoEMethod.process_weights_after_loading() (part 4/8)
```python
                epilogue_tile_m,
                num_elts_per_sf=16,
            )
            w2_bias_permute_indices = _get_flashinfer_mxfp4_device_permute_indices(
                w2_bias[0].reshape(-1, 1),
                epilogue_tile_m,
            )

            for i in range(self.num_experts):
                gemm1_weights_mxfp4_shuffled.append(
                    w13_weight[i]
                    .view(torch.uint8)[w13_weight_permute_indices]
                    .contiguous()
                )

                gemm1_scales_mxfp4_shuffled.append(
                    nvfp4_block_scale_interleave(
                        w13_weight_scale[i]
                        .view(torch.uint8)[w13_scale_permute_indices]
                        .contiguous()
                    )
                )

                gemm1_bias_shuffled.append(
                    w13_bias[i].reshape(-1, 1)[w13_bias_permute_indices].contiguous()
                )

                gemm2_weights_mxfp4_shuffled.append(
                    w2_weight[i]
                    .view(torch.uint8)[w2_weight_permute_indices]
                    .contiguous()
                )

                gemm2_scales_mxfp4_shuffled.append(
                    nvfp4_block_scale_interleave(
                        w2_weight_scale[i]
                        .view(torch.uint8)[w2_scale_permute_indices]
                        .contiguous()
                    )
                )

                gemm2_bias_shuffled.append(
                    w2_bias[i].reshape(-1, 1)[w2_bias_permute_indices].contiguous()
                )
```
**EN:** This segment of `Mxfp4MoEMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `Mxfp4MoEMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 734-778: Mxfp4MoEMethod.process_weights_after_loading() (part 5/8)
```python
            w13_weight = torch.stack(gemm1_weights_mxfp4_shuffled)
            w13_weight_scale = (
                torch.stack(gemm1_scales_mxfp4_shuffled)
                .reshape(
                    self.num_experts,
                    2 * self.intermediate_size_per_partition,
                    self.hidden_size // sf_block_size,
                )
                .view(torch.float8_e4m3fn)
            )

            w2_weight = torch.stack(gemm2_weights_mxfp4_shuffled)
            w2_weight_scale = (
                torch.stack(gemm2_scales_mxfp4_shuffled)
                .reshape(
                    self.num_experts,
                    self.hidden_size,
                    self.intermediate_size_per_partition // sf_block_size,
                )
                .view(torch.float8_e4m3fn)
            )

            layer.w13_weight = Parameter(w13_weight, requires_grad=False)
            layer.w13_weight_scale = Parameter(w13_weight_scale, requires_grad=False)
            layer.w2_weight = Parameter(w2_weight, requires_grad=False)
            layer.w2_weight_scale = Parameter(w2_weight_scale, requires_grad=False)
            layer.w13_weight_bias = Parameter(
                torch.stack(gemm1_bias_shuffled).reshape(self.num_experts, -1),
                requires_grad=False,
            )
            layer.w2_weight_bias = Parameter(
                torch.stack(gemm2_bias_shuffled).reshape(self.num_experts, -1),
                requires_grad=False,
            )
            return
        if _use_aiter:
            if layer.w13_weight_bias is not None:
                layer.w13_weight_bias.data = layer.w13_weight_bias.data.to(
                    torch.float32
                )
            if layer.w2_weight_bias is not None:
                layer.w2_weight_bias.data = layer.w2_weight_bias.data.to(torch.float32)

            e, n, k = layer.w13_weight.shape
            layer.w13_weight.view(torch.uint8).copy_(
```
**EN:** This segment of `Mxfp4MoEMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `Mxfp4MoEMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 779-823: Mxfp4MoEMethod.process_weights_after_loading() (part 6/8)
```python
                layer.w13_weight.data.view(torch.uint8)
                .view(e, n // 2, 2, k)
                .permute(0, 2, 1, 3)
                .contiguous()
                .view(e, n, k)
            )
            layer.w13_weight_scale.data = (
                layer.w13_weight_scale.data.view(e, n // 2, 2, -1)
                .permute(0, 2, 1, 3)
                .contiguous()
                .view(e, n, -1)
            )

            layer.w13_weight.data = shuffle_weight_a16w4(layer.w13_weight, 16, True)
            shuffled_w13_scale = shuffle_scale_a16w4(
                layer.w13_weight_scale.view(-1, layer.w13_weight_scale.shape[-1]),
                self.num_experts,
                True,
            )

            layer.w2_weight.data = shuffle_weight_a16w4(layer.w2_weight, 16, False)
            shuffled_w2_scale = shuffle_scale_a16w4(
                layer.w2_weight_scale.view(-1, layer.w2_weight_scale.shape[-1]),
                self.num_experts,
                False,
            )

            layer.w13_weight_bias.data = (
                layer.w13_weight_bias.data.view(-1, n // 2, 2)
                .permute(0, 2, 1)
                .contiguous()
                .view(-1, n)
            )

            layer.w13_weight_scale = torch.nn.Parameter(
                shuffled_w13_scale, requires_grad=False
            )
            layer.w2_weight_scale = torch.nn.Parameter(
                shuffled_w2_scale, requires_grad=False
            )

            return

        if self.use_triton_kernels:
```
**EN:** This segment of `Mxfp4MoEMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `Mxfp4MoEMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 824-868: Mxfp4MoEMethod.process_weights_after_loading() (part 7/8)
```python
            from triton_kernels.matmul_ogs import FlexCtx, PrecisionConfig

            w13_weight_bias = layer.w13_weight_bias.to(torch.float32)
            w2_weight_bias = layer.w2_weight_bias.to(torch.float32)

            layer.w13_weight_bias = Parameter(w13_weight_bias, requires_grad=False)
            layer.w2_weight_bias = Parameter(w2_weight_bias, requires_grad=False)

            num_warps = 8

            w13_weight, w13_flex, w13_scale = _swizzle_mxfp4(
                layer.w13_weight, layer.w13_weight_scale, num_warps
            )
            w2_weight, w2_flex, w2_scale = _swizzle_mxfp4(
                layer.w2_weight, layer.w2_weight_scale, num_warps
            )

            self.w13_precision_config = PrecisionConfig(
                weight_scale=w13_scale, flex_ctx=FlexCtx(rhs_data=w13_flex)
            )
            self.w2_precision_config = PrecisionConfig(
                weight_scale=w2_scale, flex_ctx=FlexCtx(rhs_data=w2_flex)
            )

            self.w13_weight_triton_tensor = w13_weight
            self.w2_weight_triton_tensor = w2_weight
            del layer.w13_weight
            del layer.w2_weight
        else:
            from triton_kernels.numerics_details.mxfp import upcast_from_mxfp

            w13_weight = upcast_from_mxfp(
                layer.w13_weight,
                layer.w13_weight_scale,
                target_dtype=torch.bfloat16,
                axis=-1,
            )
            w2_weight = upcast_from_mxfp(
                layer.w2_weight,
                layer.w2_weight_scale,
                target_dtype=torch.bfloat16,
                axis=-1,
            )
            del layer.w13_weight
            del layer.w2_weight
```
**EN:** This segment of `Mxfp4MoEMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `Mxfp4MoEMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 869-873: Mxfp4MoEMethod.process_weights_after_loading() (part 8/8)
```python
            del layer.w13_weight_scale
            del layer.w2_weight_scale
            layer.w13_weight = Parameter(w13_weight.data, requires_grad=False)
            layer.w2_weight = Parameter(w2_weight.data, requires_grad=False)
        torch.cuda.empty_cache()
```
**EN:** This segment of `Mxfp4MoEMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `Mxfp4MoEMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 875-919: Mxfp4MoEMethod._process_weights_for_sm90_cutlass() (part 1/3)
```python
    def _process_weights_for_sm90_cutlass(self, layer):
        """De-interleave + pad + halving-swap + byte-interleave MXFP4 weights
        for FlashInfer's SM90 ``cutlass_fused_moe(use_w4_group_scaling=True)``
        path (PR #3084).

        The cutlass kernel needs (a) K (contraction dim) % 128 == 0, and (b)
        ``fc1_expert_weights`` in halved ``[up; gate]`` order -- the
        ``compute_with_experts`` reference in FlashInfer's
        ``test_trtllm_cutlass_fused_moe.py`` splits
        ``w3, w1 = chunk(W, 2, dim=0)`` and uses w3 as up, w1 as gate.

        GPT-OSS's HF layout is *interleaved* ``[g_0, u_0, g_1, u_1, ..., g_{N-1}, u_{N-1}]``
        (each pair occupies two adjacent rows). The mxfp4 weight loader does
        a naive copy, so our unpadded buffer is interleaved post-load. We
        de-interleave (even rows -> gate, odd rows -> up), pad each half from
        N_un to N_pad, concatenate as halved ``[up; gate]``, and then run
        FlashInfer's byte / scale interleave helpers.
        """
        sf_block_size = 32  # MXFP4 group size

        # Sizes from the unpadded loaded buffers.
        N_un = layer.w13_weight.shape[1] // 2  # intermediate (unpadded)
        K_un = (
            layer.w13_weight.shape[2] * 2
        )  # hidden (unpadded, *2 because packed 4-bit)
        N_pad = self._padded_intermediate
        K_pad = self._padded_hidden
        # Use the local expert count (matches the existing buffer allocation in
        # create_weights) so the SM90 cutlass path remains correct under
        # Expert Parallelism. `self.num_experts` is the *global* count.
        E = layer.num_local_experts
        device = layer.w13_weight.device
        bias_dtype = layer.w13_weight_bias.dtype

        # ---- De-interleave + pad w13 weight/scale/bias to halved [up; gate]
        # Even rows of HF = gate, odd rows = up. After splitting we pad each
        # half along its row dim (N) from N_un to N_pad with zeros, and along
        # its last dim (K) from K_un (or K_un / sf_block_size) to K_pad.

        def _stack_up_gate_w13(unpadded_w13, last_pad, last_un):
            # unpadded_w13: [E, 2*N_un, last_un]
            # Returns: [E, 2*N_pad, last_pad] in [up_padded; gate_padded] order.
            gate_rows = unpadded_w13[:, 0::2, :]  # [E, N_un, last_un]
            up_rows = unpadded_w13[:, 1::2, :]  # [E, N_un, last_un]
            out = torch.zeros(
```
**EN:** This segment of `Mxfp4MoEMethod._process_weights_for_sm90_cutlass()` implements reusable module logic for the quantization stack. It covers one portion of the full implementation.
**CN:** `Mxfp4MoEMethod._process_weights_for_sm90_cutlass()` 的这一段代码用于实现量化栈中的可复用模块逻辑，展示了完整实现中的一部分。

### Lines 920-964: Mxfp4MoEMethod._process_weights_for_sm90_cutlass() (part 2/3)
```python
                E, 2 * N_pad, last_pad, dtype=unpadded_w13.dtype, device=device
            )
            # First half: up (with row + col padding zeros).
            out[:, :N_un, :last_un] = up_rows
            # Second half: gate.
            out[:, N_pad : N_pad + N_un, :last_un] = gate_rows
            return out

        w13_padded = _stack_up_gate_w13(
            layer.w13_weight.data.view(torch.uint8), K_pad // 2, K_un // 2
        )
        w13_scale_padded = _stack_up_gate_w13(
            layer.w13_weight_scale.data,
            K_pad // sf_block_size,
            K_un // sf_block_size,
        )
        # Bias: same de-interleave on dim=-1.
        w13_bias_gate = layer.w13_weight_bias.data[:, 0::2]  # [E, N_un]
        w13_bias_up = layer.w13_weight_bias.data[:, 1::2]  # [E, N_un]
        w13_bias_padded = torch.zeros(E, 2 * N_pad, dtype=bias_dtype, device=device)
        w13_bias_padded[:, :N_un] = w13_bias_up
        w13_bias_padded[:, N_pad : N_pad + N_un] = w13_bias_gate

        def _pad_w2_3d(unpadded, last_pad, last_un):
            out = torch.zeros(E, K_pad, last_pad, dtype=unpadded.dtype, device=device)
            out[:, :K_un, :last_un] = unpadded[:, :K_un, :]
            return out

        # ---- w2 (no halving, just pad to [E, K_pad, N_pad/2]) ----------------
        w2_padded = _pad_w2_3d(
            layer.w2_weight.data.view(torch.uint8), N_pad // 2, N_un // 2
        )
        w2_scale_padded = _pad_w2_3d(
            layer.w2_weight_scale.data,
            N_pad // sf_block_size,
            N_un // sf_block_size,
        )
        w2_bias_padded = torch.zeros(E, K_pad, dtype=bias_dtype, device=device)
        w2_bias_padded[:, :K_un] = layer.w2_weight_bias.data

        # ---- Per-expert SwiGLU scalars (GPT-OSS defaults) ------------------
        layer.swiglu_alpha = Parameter(
            torch.full((E,), 1.702, dtype=torch.float32, device=device),
            requires_grad=False,
        )
```
**EN:** This segment of `Mxfp4MoEMethod._process_weights_for_sm90_cutlass()` implements reusable module logic for the quantization stack. It covers one portion of the full implementation.
**CN:** `Mxfp4MoEMethod._process_weights_for_sm90_cutlass()` 的这一段代码用于实现量化栈中的可复用模块逻辑，展示了完整实现中的一部分。

### Lines 965-1000: Mxfp4MoEMethod._process_weights_for_sm90_cutlass() (part 3/3)
```python
        layer.swiglu_beta = Parameter(
            torch.full((E,), 1.0, dtype=torch.float32, device=device),
            requires_grad=False,
        )
        layer.swiglu_limit = Parameter(
            torch.full((E,), 7.0, dtype=torch.float32, device=device),
            requires_grad=False,
        )

        # ---- FlashInfer SM90 byte / scale interleave -----------------------
        # The padded buffers above are contiguous by construction (allocated
        # via torch.zeros + slice assignment), so we feed them straight in.
        layer.w13_weight = Parameter(
            interleave_moe_weights_for_sm90_mixed_gemm(w13_padded, "fp4"),
            requires_grad=False,
        )
        layer.w2_weight = Parameter(
            interleave_moe_weights_for_sm90_mixed_gemm(w2_padded, "fp4"),
            requires_grad=False,
        )
        layer.w13_weight_scale = Parameter(
            interleave_moe_scales_for_sm90_mixed_gemm(
                w13_scale_padded, group_size=sf_block_size
            ),
            requires_grad=False,
        )
        layer.w2_weight_scale = Parameter(
            interleave_moe_scales_for_sm90_mixed_gemm(
                w2_scale_padded, group_size=sf_block_size
            ),
            requires_grad=False,
        )
        layer.w13_weight_bias = Parameter(w13_bias_padded, requires_grad=False)
        layer.w2_weight_bias = Parameter(w2_bias_padded, requires_grad=False)

        torch.cuda.empty_cache()
```
**EN:** This segment of `Mxfp4MoEMethod._process_weights_for_sm90_cutlass()` implements reusable module logic for the quantization stack. It covers one portion of the full implementation.
**CN:** `Mxfp4MoEMethod._process_weights_for_sm90_cutlass()` 的这一段代码用于实现量化栈中的可复用模块逻辑，展示了完整实现中的一部分。

### Lines 1002-1029: Mxfp4MoEMethod.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        self.moe_runner_config = moe_runner_config
        moe_runner_backend = get_moe_runner_backend()
        if moe_runner_backend.is_auto():
            # Must match apply() priority: _use_aiter before use_triton_kernels.
            if _use_aiter and get_moe_a2a_backend().supports_aiter():
                moe_runner_backend = MoeRunnerBackend.AITER
            elif self.use_triton_kernels:
                moe_runner_backend = MoeRunnerBackend.TRITON_KERNELS
            else:
                moe_runner_backend = MoeRunnerBackend.TRITON

        if moe_runner_backend.is_aiter():
            # MXFP4 hard-codes Swiglu in the AITER kernel path.
            self.runner = MoeRunner(
                moe_runner_backend, replace(moe_runner_config, activation="swiglu")
            )
        elif (
            moe_runner_backend.is_triton_kernels()
            or moe_runner_backend.is_triton()
            or moe_runner_backend.is_marlin()
        ):
            self.runner = MoeRunner(moe_runner_backend, moe_runner_config)
        else:
            # TODO(cwan): refactor other backends
            pass
```
**EN:** This block defines `Mxfp4MoEMethod.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `Mxfp4MoEMethod.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 1031-1097: Mxfp4MoEMethod._apply_sm90_cutlass()
```python
    def _apply_sm90_cutlass(self, layer, x, topk_output):
        """SM90 (Hopper) MXFP4 x BF16 MoE via FlashInfer's cutlass mixed-input
        path (PR #3084). The fused kernel does GEMM1 + SwiGLU + GEMM2 in one
        call; weights/scales were pre-interleaved at load time."""
        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput
        from sglang.srt.layers.moe.topk import TopKOutputChecker

        # Under ``--moe-runner-backend flashinfer_mxfp4`` the SGLang TopK layer
        # emits BypassedTopKOutput by default (the SM100 trtllm-gen kernel does
        # routing internally). The cutlass kernel needs explicit topk_ids /
        # topk_weights, so materialize them here when bypassed.
        if TopKOutputChecker.format_is_bypassed(topk_output):
            topk_output = topk_output.to_standard()
        topk_weights, topk_ids = topk_output.topk_weights, topk_output.topk_ids

        # Pad input hidden dim to the (already-padded) loaded weight width.
        origin_hidden = x.shape[-1]
        padded_hidden = self._padded_hidden
        if padded_hidden != origin_hidden:
            x = torch.nn.functional.pad(
                x,
                (0, padded_hidden - origin_hidden),
                mode="constant",
                value=0.0,
            )

        output_dtype = torch.bfloat16
        # Output is allocated at padded width (kernel writes padded_hidden
        # columns), then trimmed back to origin_hidden before returning.
        with use_symmetric_memory(
            get_tp_group(), disabled=not is_allocation_symmetric()
        ):
            out_padded = torch.empty(
                x.shape[0], padded_hidden, dtype=output_dtype, device=x.device
            )

        flashinfer_cutlass_fused_moe(
            input=x,
            token_selected_experts=topk_ids.to(torch.int),
            token_final_scales=topk_weights,
            fc1_expert_weights=layer.w13_weight,  # uint8 [E, 2*N, K/2] interleaved
            fc2_expert_weights=layer.w2_weight,  # uint8 [E, K, N/2] interleaved
            output_dtype=output_dtype,
            quant_scales=[
                layer.w13_weight_scale.view(torch.int32),
                layer.w2_weight_scale.view(torch.int32),
            ],
            fc1_expert_biases=layer.w13_weight_bias,  # bf16 [E, 2*N]
            fc2_expert_biases=layer.w2_weight_bias,  # bf16 [E, K]
            swiglu_alpha=layer.swiglu_alpha,
            swiglu_beta=layer.swiglu_beta,
            swiglu_limit=layer.swiglu_limit,
            tp_size=layer.moe_tp_size,
            tp_rank=layer.moe_tp_rank,
            ep_size=layer.moe_ep_size,
            ep_rank=layer.moe_ep_rank,
            use_w4_group_scaling=True,
            activation_type=ActivationType.Swiglu,
            tune_max_num_tokens=next_power_of_2(x.shape[0]),
            output=out_padded,
        )

        if padded_hidden != origin_hidden:
            out = out_padded[:, :origin_hidden].contiguous()
        else:
            out = out_padded
        return StandardCombineInput(hidden_states=out)
```
**EN:** This block defines `Mxfp4MoEMethod._apply_sm90_cutlass()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `Mxfp4MoEMethod._apply_sm90_cutlass()`，用于实现量化栈中的可复用模块逻辑。

### Lines 1099-1143: Mxfp4MoEMethod.apply() (part 1/4)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:

        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput
        from sglang.srt.layers.moe.topk import TopKOutputChecker

        x = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output

        if self.use_marlin:
            assert TopKOutputChecker.format_is_standard(topk_output)
            quant_info = MarlinMoeQuantInfo(
                w13_qweight=layer.w13_weight,
                w2_qweight=layer.w2_weight,
                w13_scales=layer.w13_weight_scale,
                w2_scales=layer.w2_weight_scale,
                w13_g_idx_sort_indices=None,
                w2_g_idx_sort_indices=None,
                weight_bits=4,
                is_k_full=True,
            )
            return self.runner.run(dispatch_output, quant_info)

        if self._fi_kernel == "cutlass_sm90":
            return self._apply_sm90_cutlass(layer, x, topk_output)
        if self.use_flashinfer:
            # When bf16 mode is enabled, we don't need to quantize the input,
            # TRT-LLM automatically handles quantization in the kernel implementation and pipelines it with GEMM operations,
            # which can theoretically improve performance
            origin_hidden_states_dim = x.shape[-1]
            if self.flashinfer_mxfp4_moe_precision == "bf16":
                assert x.dtype == torch.bfloat16
                x_quant = x
                x_scale = None

                # May be fused later if this code branch is frequently needed
                if self.hidden_size != origin_hidden_states_dim:
                    x_quant = torch.nn.functional.pad(
                        x_quant,
                        (0, self.hidden_size - origin_hidden_states_dim),
                        mode="constant",
                        value=0.0,
```
**EN:** This segment of `Mxfp4MoEMethod.apply()` applies quantized computation to runtime inputs. It covers one portion of the full implementation.
**CN:** `Mxfp4MoEMethod.apply()` 的这一段代码用于将量化计算应用到运行时输入上，展示了完整实现中的一部分。

### Lines 1144-1188: Mxfp4MoEMethod.apply() (part 2/4)
```python
                    )
            elif self.flashinfer_mxfp4_moe_precision == "default":
                x_quant, x_scale = mxfp8_quantize(x, False, alignment=self.hidden_size)
                x_scale = x_scale.view(torch.float8_e4m3fn).reshape(*x.shape[:-1], -1)
            else:
                raise NotImplementedError()

            assert x_quant.shape[-1] == self.hidden_size
            assert TopKOutputChecker.format_is_bypassed(topk_output)

            top_k = topk_output.topk_config.top_k
            router_logits = topk_output.router_logits

            with use_symmetric_memory(
                get_tp_group(), disabled=not is_allocation_symmetric()
            ):
                num_tokens = x_quant.shape[0]
                hidden_size = origin_hidden_states_dim
                symm_output = torch.empty(
                    num_tokens, hidden_size, dtype=torch.bfloat16, device=x_quant.device
                )
            trtllm_gen_output = trtllm_fp4_block_scale_moe(
                router_logits.to(torch.bfloat16),
                None,  # routing_bias
                x_quant,
                x_scale,
                layer.w13_weight,  # uint8 (e2m1 x 2)
                layer.w13_weight_scale,  # uint8 (e4m3 x 2)
                layer.w13_weight_bias,  # fp32 per expert per channel
                layer.gemm1_alpha,  # fp32 per expert
                layer.gemm1_beta,  # fp32 per expert
                layer.gemm1_clamp_limit,  # fp32 per expert
                layer.w2_weight,  # uint8 (e2m1 x 2)
                layer.w2_weight_scale,  # ue8m0
                layer.w2_weight_bias,  # fp32 per expert per channel
                None,  # output1_scale_scalar
                None,  # output1_scale_gate_scalar
                None,  # output2_scale_scalar
                layer.num_experts,
                top_k,
                None,  # n_group      # TODO: support n_group
                None,  # topk_group   # TODO: support topk_group
                self.intermediate_size_per_partition,  # padded to multiple of 256
                layer.moe_ep_rank * layer.num_local_experts,  # local_expert_offset
                layer.num_local_experts,  # local num experts
```
**EN:** This segment of `Mxfp4MoEMethod.apply()` applies quantized computation to runtime inputs. It covers one portion of the full implementation.
**CN:** `Mxfp4MoEMethod.apply()` 的这一段代码用于将量化计算应用到运行时输入上，展示了完整实现中的一部分。

### Lines 1189-1233: Mxfp4MoEMethod.apply() (part 3/4)
```python
                None,  # routed_scaling_factor
                1,  # routing_method_type, renormalize
                True,  # do finalize
                tune_max_num_tokens=next_power_of_2(x_quant.shape[0]),
                output=symm_output,
            )[0]
            return StandardCombineInput(hidden_states=trtllm_gen_output)
        if _use_aiter:
            from sglang.srt.layers.moe.moe_runner.aiter import (
                AiterMoeQuantInfo,
                AiterQuantType,
            )

            if hasattr(torch, "float4_e2m1fn_x2"):
                w13_weight = layer.w13_weight.view(torch.float4_e2m1fn_x2)
                w2_weight = layer.w2_weight.view(torch.float4_e2m1fn_x2)
            else:
                w13_weight = layer.w13_weight
                w2_weight = layer.w2_weight

            x_padded = torch.nn.functional.pad(
                x, (0, self.hidden_pad), mode="constant", value=0.0
            )
            quant_info = AiterMoeQuantInfo(
                w13_weight=w13_weight,
                w2_weight=w2_weight,
                quant_type=AiterQuantType.PER_1X32,
                w13_scale=layer.w13_weight_scale,
                w2_scale=layer.w2_weight_scale,
                b13=layer.w13_weight_bias,
                b2=layer.w2_weight_bias,
                expert_mask=layer.dispatcher.expert_mask_gpu,
                doweight_stage1=self.moe_runner_config.apply_router_weight_on_input,
                hidden_pad=self.hidden_pad,
                intermediate_pad=self.intermediate_pad,
            )
            return self.runner.run(
                dispatch_output._replace(hidden_states=x_padded), quant_info
            )

        backend = self.runner.runner_backend
        if backend.is_triton_kernels():
            from sglang.srt.layers.moe.moe_runner.triton_kernels import (
                TritonKernelsQuantInfo,
            )
```
**EN:** This segment of `Mxfp4MoEMethod.apply()` applies quantized computation to runtime inputs. It covers one portion of the full implementation.
**CN:** `Mxfp4MoEMethod.apply()` 的这一段代码用于将量化计算应用到运行时输入上，展示了完整实现中的一部分。

### Lines 1234-1261: Mxfp4MoEMethod.apply() (part 4/4)
```python

            assert (
                layer.moe_ep_size == 1
            ), "Expert parallel is not supported when using triton kernels"
            quant_info = TritonKernelsQuantInfo(
                w13_weight=(
                    self.w13_weight_triton_tensor
                    if self.w13_weight_triton_tensor is not None
                    else layer.w13_weight
                ),
                w2_weight=(
                    self.w2_weight_triton_tensor
                    if self.w2_weight_triton_tensor is not None
                    else layer.w2_weight
                ),
                w13_bias=getattr(layer, "w13_weight_bias", None),
                w2_bias=getattr(layer, "w2_weight_bias", None),
                w13_precision_config=getattr(self, "w13_precision_config", None),
                w2_precision_config=getattr(self, "w2_precision_config", None),
            )
        else:
            quant_info = TritonMoeQuantInfo(
                w13_weight=layer.w13_weight,
                w2_weight=layer.w2_weight,
                b13=getattr(layer, "w13_weight_bias", None),
                b2=getattr(layer, "w2_weight_bias", None),
            )
        return self.runner.run(dispatch_output, quant_info)
```
**EN:** This segment of `Mxfp4MoEMethod.apply()` applies quantized computation to runtime inputs. It covers one portion of the full implementation.
**CN:** `Mxfp4MoEMethod.apply()` 的这一段代码用于将量化计算应用到运行时输入上，展示了完整实现中的一部分。

### Lines 1264-1264: class Mxfp4DynamicQuantMoEMethod: definition
```python
class Mxfp4DynamicQuantMoEMethod(FusedMoEMethodBase):
```
**EN:** This block declares `Mxfp4DynamicQuantMoEMethod`, a runtime method class for the quantization stack. It organizes behaviors such as create_weights, mxfp4_quantize, process_weights_after_loading, create_moe_runner.
**CN:** 该代码块声明 `Mxfp4DynamicQuantMoEMethod`，它是量化栈中的运行方法类，组织了 create_weights, mxfp4_quantize, process_weights_after_loading, create_moe_runner 等行为。

### Lines 1265-1320: Mxfp4DynamicQuantMoEMethod.create_weights()
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):

        from sglang.srt.layers.moe.fused_moe_triton import FusedMoeWeightScaleSupported

        w13_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                2 * intermediate_size_per_partition,
                hidden_size,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        w2_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size,
                intermediate_size_per_partition,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )

        layer.register_parameter("w13_weight", w13_weight)
        set_weight_attrs(w13_weight, extra_weight_attrs)

        layer.register_parameter("w2_weight", w2_weight)
        set_weight_attrs(w2_weight, extra_weight_attrs)

        # Allocate 2 scales for w1 and w3 respectively.
        # They will be combined to a single scale after weight loading.
        w13_weight_scale = torch.nn.Parameter(
            torch.ones(num_experts, 2, dtype=torch.float32), requires_grad=False
        )
        w2_weight_scale = torch.nn.Parameter(
            torch.ones(num_experts, dtype=torch.float32), requires_grad=False
        )
        layer.register_parameter("w13_weight_scale", w13_weight_scale)
        layer.register_parameter("w2_weight_scale", w2_weight_scale)

        # Add the quantization method used (per tensor/grouped/channel)
        # to ensure the weight scales are loaded in properly
        extra_weight_attrs.update(
            {"quant_method": FusedMoeWeightScaleSupported.TENSOR.value}
        )

        layer.w13_input_scale = None
        layer.w2_input_scale = None
```
**EN:** This block defines `Mxfp4DynamicQuantMoEMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `Mxfp4DynamicQuantMoEMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 1322-1338: Mxfp4DynamicQuantMoEMethod.mxfp4_quantize()
```python
    def mxfp4_quantize(self, w):
        w_shape = w.shape
        w_need_reshape = True if w.dim() != 2 else False

        if w_need_reshape:
            w_last_dim_size = w_shape[-1]
            w = w.view(-1, w_last_dim_size)

        w, mx_scales = dynamic_mxfp4_quant(w)

        if w_need_reshape:
            w_new_shape = w_shape[:-1] + (w.shape[-1],)
            w = w.view(w_new_shape)

        mx_scales = e8m0_shuffle(mx_scales)

        return w, mx_scales
```
**EN:** This block defines `Mxfp4DynamicQuantMoEMethod.mxfp4_quantize()`, which handles tensor or weight quantization work.
**CN:** 该代码块定义了 `Mxfp4DynamicQuantMoEMethod.mxfp4_quantize()`，用于处理张量或权重量化逻辑。

### Lines 1340-1356: Mxfp4DynamicQuantMoEMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        w13, w13_mx_scales = self.mxfp4_quantize(layer.w13_weight.data)
        w2, w2_mx_scales = self.mxfp4_quantize(layer.w2_weight.data)

        # Pre-shuffle weight
        is_shuffled = _is_shuffle_moe_mxfp4
        if is_shuffled:
            w13 = shuffle_weight(w13.contiguous(), (16, 16))
            w2 = shuffle_weight(w2.contiguous(), (16, 16))

        layer.w13_weight = torch.nn.Parameter(w13, requires_grad=False)
        layer.w13_weight.is_shuffled = is_shuffled
        layer.w13_weight_scale = torch.nn.Parameter(w13_mx_scales, requires_grad=False)

        layer.w2_weight = torch.nn.Parameter(w2, requires_grad=False)
        layer.w2_weight.is_shuffled = is_shuffled
        layer.w2_weight_scale = torch.nn.Parameter(w2_mx_scales, requires_grad=False)
```
**EN:** This block defines `Mxfp4DynamicQuantMoEMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `Mxfp4DynamicQuantMoEMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 1358-1370: Mxfp4DynamicQuantMoEMethod.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        self.moe_runner_config = moe_runner_config
        moe_runner_backend = get_moe_runner_backend()
        if moe_runner_backend.is_auto() and get_moe_a2a_backend().supports_aiter():
            moe_runner_backend = MoeRunnerBackend.AITER

        if moe_runner_backend.is_aiter():
            self.runner = MoeRunner(moe_runner_backend, moe_runner_config)
        else:
            # TODO(cwan): refactor other backends
            pass
```
**EN:** This block defines `Mxfp4DynamicQuantMoEMethod.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `Mxfp4DynamicQuantMoEMethod.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 1372-1401: Mxfp4DynamicQuantMoEMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:
        from sglang.srt.layers.moe.moe_runner.aiter import (
            AiterMoeQuantInfo,
            AiterQuantType,
        )

        if hasattr(torch, "float4_e2m1fn_x2"):
            w13_weight = layer.w13_weight.view(torch.float4_e2m1fn_x2)
            w2_weight = layer.w2_weight.view(torch.float4_e2m1fn_x2)
        else:
            w13_weight = layer.w13_weight
            w2_weight = layer.w2_weight

        if hasattr(layer.w13_weight, "is_shuffled"):
            w13_weight.is_shuffled = True
            w2_weight.is_shuffled = True

        quant_info = AiterMoeQuantInfo(
            w13_weight=w13_weight,
            w2_weight=w2_weight,
            quant_type=AiterQuantType.PER_1X32,
            w13_scale=layer.w13_weight_scale,
            w2_scale=layer.w2_weight_scale,
            expert_mask=layer.dispatcher.expert_mask_gpu,
        )
        return self.runner.run(dispatch_output, quant_info)
```
**EN:** This block defines `Mxfp4DynamicQuantMoEMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `Mxfp4DynamicQuantMoEMethod.apply()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `dequant_mxfp4()` : A public function that restores low-precision data for downstream compute. / `dequant_mxfp4()`：一个公开函数，用于为后续计算恢复低精度数据。
- `quant_dequant_mxfp4()` : A public function that restores low-precision data for downstream compute. / `quant_dequant_mxfp4()`：一个公开函数，用于为后续计算恢复低精度数据。
- `Mxfp4Config`: A configuration class that structures file-level quantization behavior. / `Mxfp4Config` 是一个配置类，用于组织该文件中的量化行为。
- `Mxfp4MoEMethod`: A runtime method class that structures file-level quantization behavior. / `Mxfp4MoEMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `Mxfp4DynamicQuantMoEMethod`: A runtime method class that structures file-level quantization behavior. / `Mxfp4DynamicQuantMoEMethod` 是一个运行方法类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `aiter.ops.shuffle`, `aiter.ops.triton.quant`, `aiter.utility.fp4_utils`, `dataclasses`, `flashinfer`, `flashinfer.fused_moe`, `flashinfer.fused_moe.core`, `inspect`, `os`, `quark.torch.kernel`, `torch`
- **Internal / 内部**: `sglang.srt.distributed`, `sglang.srt.distributed.device_communicators.pynccl_allocator`, `sglang.srt.layers.dp_attention`, `sglang.srt.layers.linear`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.moe_runner.aiter`, `sglang.srt.layers.moe.moe_runner.marlin`, `sglang.srt.layers.moe.moe_runner.triton`, `sglang.srt.layers.moe.moe_runner.triton_kernels`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.moe.topk`, `sglang.srt.layers.moe.utils`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.quantization.marlin_utils`, `sglang.srt.layers.quantization.marlin_utils_fp4`
