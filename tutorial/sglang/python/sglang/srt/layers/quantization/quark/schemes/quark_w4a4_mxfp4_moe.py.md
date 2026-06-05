# quark_w4a4_mxfp4_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/quark/schemes/quark_w4a4_mxfp4_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains supporting code for quark w4a4 mxfp4 moe quantization in the SGLang SRT layer stack. / 该模块包含 SGLang SRT 层栈中与 Quark w4a4 MXFP4 MoE 量化相关的支撑代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: module imports and setup
```python
# SPDX-License-Identifier: Apache-2.0

from __future__ import annotations

import logging
from typing import TYPE_CHECKING, Any

import torch

from sglang.srt.layers.moe import MoeRunner, MoeRunnerBackend, MoeRunnerConfig
from sglang.srt.layers.moe.utils import get_moe_weight_sizes
from sglang.srt.layers.quantization.quark.schemes import QuarkMoEScheme
from sglang.srt.utils import (
    get_bool_env_var,
    is_gfx95_supported,
    is_hip,
    set_weight_attrs,
)
```
**EN:** This block imports __future__, aiter.ops.shuffle, aiter.utility.fp4_utils, logging, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.moe.moe_runner.aiter, sglang.srt.layers.moe.token_dispatcher and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, aiter.ops.shuffle, aiter.utility.fp4_utils, logging, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.moe.moe_runner.aiter, sglang.srt.layers.moe.token_dispatcher 等依赖，并为当前量化实现准备模块命名空间。

### Lines 20-24: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        StandardDispatchOutput,
    )
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 26-26: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 28-28: initialize _is_shuffle_moe_mxfp4
```python
_is_shuffle_moe_mxfp4 = is_gfx95_supported()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_shuffle_moe_mxfp4.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_shuffle_moe_mxfp4。

### Lines 30-30: initialize __all__
```python
__all__ = ["QuarkW4A4MXFp4MoE"]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as __all__.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 __all__。

### Lines 32-32: initialize _is_hip
```python
_is_hip = is_hip()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_hip.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_hip。

### Lines 33-33: initialize _use_aiter
```python
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _use_aiter.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _use_aiter。

### Lines 34-36: conditional logic for _use_aiter
```python
if _use_aiter:
    from aiter.ops.shuffle import shuffle_weight
    from aiter.utility.fp4_utils import e8m0_shuffle
```
**EN:** This block applies conditional logic controlled by `_use_aiter`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `_use_aiter` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 38-38: initialize OCP_MX_BLOCK_SIZE
```python
OCP_MX_BLOCK_SIZE = 32
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as OCP_MX_BLOCK_SIZE.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 OCP_MX_BLOCK_SIZE。

### Lines 41-42: class QuarkW4A4MXFp4MoE: definition
```python
class QuarkW4A4MXFp4MoE(QuarkMoEScheme):
```
**EN:** This block declares `QuarkW4A4MXFp4MoE`, a scheme class for the quantization stack. It organizes behaviors such as __init__, get_min_capability, create_weights, process_weights_after_loading.
**CN:** 该代码块声明 `QuarkW4A4MXFp4MoE`，它是量化栈中的执行方案类，组织了 __init__, get_min_capability, create_weights, process_weights_after_loading 等行为。

### Lines 43-57: QuarkW4A4MXFp4MoE.__init__()
```python
    def __init__(self, weight_config: dict[str, Any], input_config: dict[str, Any]):
        self.weight_quant = weight_config
        self.input_quant = input_config

        weight_qscheme = self.weight_quant.get("qscheme")
        input_qscheme = self.input_quant.get("qscheme")
        if not (weight_qscheme == "per_group" and input_qscheme == "per_group"):
            raise ValueError(
                "For MX(FP4) Fused MoE layers, only per-group scales "
                "for weights and activations are supported. Found "
                f"{weight_qscheme}, {input_qscheme}"
            )  # noqa E501

        self.static_input_scales = not self.input_quant.get("is_dynamic")
        self.with_bias = False
```
**EN:** This block defines `QuarkW4A4MXFp4MoE.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `QuarkW4A4MXFp4MoE.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 59-61: QuarkW4A4MXFp4MoE.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 70
```
**EN:** This block defines `QuarkW4A4MXFp4MoE.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuarkW4A4MXFp4MoE.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 63-107: QuarkW4A4MXFp4MoE.create_weights() (part 1/2)
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

        w13_up_dim, w2_down_dim, weight_padded = get_moe_weight_sizes(
            intermediate_size_per_partition,
            is_aiter_moe=_use_aiter,
            is_concat=True,
            is_packed=True,
        )

        # Add the quantization method used (per tensor/grouped/channel)
        # to ensure the weight scales are loaded in properly
        extra_weight_attrs.update(
            {
                "quant_method": FusedMoeWeightScaleSupported.BLOCK.value,
                "weight_padded": weight_padded,
            },
        )

        params_dtype = torch.uint8

        # WEIGHTS
        w13_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                w13_up_dim,
                hidden_size // 2,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight", w13_weight)

        set_weight_attrs(w13_weight, extra_weight_attrs)

        w2_weight = torch.nn.Parameter(
```
**EN:** This segment of `QuarkW4A4MXFp4MoE.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `QuarkW4A4MXFp4MoE.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 108-147: QuarkW4A4MXFp4MoE.create_weights() (part 2/2)
```python
            torch.empty(
                num_experts,
                hidden_size,
                w2_down_dim,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight", w2_weight)

        set_weight_attrs(w2_weight, extra_weight_attrs)

        # WEIGHT_SCALES
        w13_weight_scale = torch.nn.Parameter(
            torch.ones(
                num_experts,
                w13_up_dim,
                hidden_size // OCP_MX_BLOCK_SIZE,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )

        # 1. w2 scale is floor division of inter_dim by blockscale.
        # 2. w2 scale needs to scale up just as w2.
        # We combine 1. and 2. to keep the integer precision.
        w2_weight_scale = torch.nn.Parameter(
            torch.ones(
                num_experts,
                hidden_size,
                (w2_down_dim * 2) // OCP_MX_BLOCK_SIZE,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        set_weight_attrs(w2_weight_scale, extra_weight_attrs)
        set_weight_attrs(w13_weight_scale, extra_weight_attrs)

        layer.register_parameter("w13_weight_scale", w13_weight_scale)
        layer.register_parameter("w2_weight_scale", w2_weight_scale)
```
**EN:** This segment of `QuarkW4A4MXFp4MoE.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `QuarkW4A4MXFp4MoE.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 149-178: QuarkW4A4MXFp4MoE.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        float_dtype = torch.get_default_dtype()

        # Pre-shuffle weight scales
        s0, s1, _ = layer.w13_weight_scale.shape
        w13_weight_scale = layer.w13_weight_scale.view(s0 * s1, -1)
        w13_weight_scale = e8m0_shuffle(w13_weight_scale)
        # layer.w13_weight_scale = torch.nn.Parameter(w13_weight_scale, requires_grad=False)
        layer.w13_weight_scale.data = w13_weight_scale.view(s0, s1, -1)

        s0, s1, _ = layer.w2_weight_scale.shape
        w2_weight_scale = layer.w2_weight_scale.view(s0 * s1, -1)
        w2_weight_scale = e8m0_shuffle(w2_weight_scale)
        # layer.w2_weight_scale = torch.nn.Parameter(w2_weight_scale, requires_grad=False)
        layer.w2_weight_scale.data = w2_weight_scale.view(s0, s1, -1)

        # Pre-shuffle weight
        if _is_shuffle_moe_mxfp4:
            layer.w13_weight.data = shuffle_weight(
                layer.w13_weight.contiguous(), (16, 16)
            )
            layer.w2_weight.data = shuffle_weight(
                layer.w2_weight.contiguous(), (16, 16)
            )
            layer.w13_weight.is_shuffled = True
            layer.w2_weight.is_shuffled = True

        if hasattr(layer, "dispatcher"):
            # Weights are stored as torch.uint8 but semantically MXFP4
            layer.dispatcher.set_quant_config({"weight_dtype": torch.float4_e2m1fn_x2})
```
**EN:** This block defines `QuarkW4A4MXFp4MoE.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `QuarkW4A4MXFp4MoE.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 180-197: QuarkW4A4MXFp4MoE.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        from sglang.srt.layers.moe.utils import (
            get_moe_a2a_backend,
            get_moe_runner_backend,
        )

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
**EN:** This block defines `QuarkW4A4MXFp4MoE.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `QuarkW4A4MXFp4MoE.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 199-228: QuarkW4A4MXFp4MoE.apply_weights()
```python
    def apply_weights(
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
**EN:** This block defines `QuarkW4A4MXFp4MoE.apply_weights()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `QuarkW4A4MXFp4MoE.apply_weights()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `OCP_MX_BLOCK_SIZE`: A module-level constant or registry. / `OCP_MX_BLOCK_SIZE`：模块级常量或注册表。
- `QuarkW4A4MXFp4MoE`: A scheme class that structures file-level quantization behavior. / `QuarkW4A4MXFp4MoE` 是一个执行方案类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `aiter.ops.shuffle`, `aiter.utility.fp4_utils`, `logging`, `torch`, `typing`
- **Internal / 内部**: `sglang.srt.layers.moe`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.moe_runner.aiter`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.moe.utils`, `sglang.srt.layers.quantization.quark.schemes`, `sglang.srt.utils`
