# quark_w8a8_fp8_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/quark/schemes/quark_w8a8_fp8_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains supporting code for quark w8a8 fp8 moe quantization in the SGLang SRT layer stack. / 该模块包含 SGLang SRT 层栈中与 Quark W8A8 FP8 MoE 量化相关的支撑代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: module imports and setup
```python
# SPDX-License-Identifier: Apache-2.0

from __future__ import annotations

import logging
from typing import TYPE_CHECKING, Any

import torch

from sglang.srt.layers.moe import MoeRunner, MoeRunnerBackend, MoeRunnerConfig
from sglang.srt.layers.moe.moe_runner.triton import TritonMoeQuantInfo
from sglang.srt.layers.quantization.fp8_kernel import is_fp8_fnuz, scaled_fp8_quant
from sglang.srt.layers.quantization.fp8_utils import normalize_e4m3fn_to_e4m3fnuz
from sglang.srt.layers.quantization.quark.schemes import QuarkMoEScheme
from sglang.srt.layers.quantization.utils import all_close_1d, per_tensor_dequantize
from sglang.srt.utils import get_bool_env_var, is_hip, set_weight_attrs
```
**EN:** This block imports __future__, aiter.ops.shuffle, logging, torch, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.moe.moe_runner.triton, sglang.srt.layers.moe.rocm_moe_utils and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, aiter.ops.shuffle, logging, torch, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.moe.moe_runner.triton, sglang.srt.layers.moe.rocm_moe_utils 等依赖，并为当前量化实现准备模块命名空间。

### Lines 18-22: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        StandardDispatchOutput,
    )
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 24-24: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 26-26: initialize __all__
```python
__all__ = ["QuarkW8A8FP8MoE"]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as __all__.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 __all__。

### Lines 28-28: initialize _is_fp8_fnuz
```python
_is_fp8_fnuz = is_fp8_fnuz()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_fp8_fnuz.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_fp8_fnuz。

### Lines 29-29: initialize _is_hip
```python
_is_hip = is_hip()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_hip.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_hip。

### Lines 30-30: initialize _use_aiter
```python
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _use_aiter.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _use_aiter。

### Lines 31-34: conditional logic for _use_aiter
```python
if _use_aiter:
    from aiter.ops.shuffle import shuffle_weight

    from sglang.srt.layers.moe.rocm_moe_utils import rocm_fused_experts_tkw1
```
**EN:** This block applies conditional logic controlled by `_use_aiter`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `_use_aiter` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 37-38: class QuarkW8A8FP8MoE: definition
```python
class QuarkW8A8FP8MoE(QuarkMoEScheme):
```
**EN:** This block declares `QuarkW8A8FP8MoE`, a scheme class for the quantization stack. It organizes behaviors such as __init__, get_min_capability, create_weights, process_weights_after_loading.
**CN:** 该代码块声明 `QuarkW8A8FP8MoE`，它是量化栈中的执行方案类，组织了 __init__, get_min_capability, create_weights, process_weights_after_loading 等行为。

### Lines 39-52: QuarkW8A8FP8MoE.__init__()
```python
    def __init__(self, weight_config: dict[str, Any], input_config: dict[str, Any]):
        self.is_static_input_scheme: bool = False
        self.input_qscheme = None

        if input_config is not None:
            self.is_static_input_scheme = not input_config.get("is_dynamic")
            self.input_qscheme = input_config.get("qscheme")

        self.input_per_token = (
            not self.is_static_input_scheme and self.input_qscheme == "per_channel"
        )
        self.weight_qscheme = weight_config.get("qscheme")
        self.is_weight_per_channel = self.weight_qscheme == "per_channel"
        self.out_dtype = torch.get_default_dtype()
```
**EN:** This block defines `QuarkW8A8FP8MoE.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `QuarkW8A8FP8MoE.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 54-57: QuarkW8A8FP8MoE.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        # lovelace and up
        return 89
```
**EN:** This block defines `QuarkW8A8FP8MoE.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuarkW8A8FP8MoE.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 59-103: QuarkW8A8FP8MoE.create_weights() (part 1/3)
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

        params_dtype = torch.float8_e4m3fn

        # WEIGHTS
        w13_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                2 * intermediate_size_per_partition,
                hidden_size,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight", w13_weight)
        set_weight_attrs(w13_weight, extra_weight_attrs)

        w2_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size,
                intermediate_size_per_partition,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight", w2_weight)
        set_weight_attrs(w2_weight, extra_weight_attrs)

        # WEIGHT_SCALES
        # per-tensor quantization
        if self.weight_qscheme == "per_tensor":
            # Allocate 2 scales for w1 and w3 respectively.
            # They will be combined to a single scale after weight loading.
            w13_weight_scale = torch.nn.Parameter(
                torch.ones(num_experts, 2, dtype=torch.float32), requires_grad=False
```
**EN:** This segment of `QuarkW8A8FP8MoE.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `QuarkW8A8FP8MoE.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 104-148: QuarkW8A8FP8MoE.create_weights() (part 2/3)
```python
            )
            w2_weight_scale = torch.nn.Parameter(
                torch.ones(num_experts, dtype=torch.float32), requires_grad=False
            )
            weight_quant_method = FusedMoeWeightScaleSupported.TENSOR.value
        elif self.weight_qscheme == "per_channel":
            w13_weight_scale = torch.nn.Parameter(
                torch.ones(
                    num_experts,
                    2 * intermediate_size_per_partition,
                    dtype=torch.float32,
                ),
                requires_grad=False,
            )
            w2_weight_scale = torch.nn.Parameter(
                torch.ones(num_experts, hidden_size, dtype=torch.float32),
                requires_grad=False,
            )
            weight_quant_method = FusedMoeWeightScaleSupported.CHANNEL.value
        else:
            raise ValueError(
                f"Unsupported weight quantization strategy: {self.weight_qscheme}."
            )

        layer.register_parameter("w13_weight_scale", w13_weight_scale)
        layer.register_parameter("w2_weight_scale", w2_weight_scale)
        # Add the quantization method used (per tensor/grouped/channel)
        # to ensure the weight scales are loaded in properly
        extra_weight_attrs.update({"quant_method": weight_quant_method})
        set_weight_attrs(w13_weight_scale, extra_weight_attrs)
        set_weight_attrs(w2_weight_scale, extra_weight_attrs)

        # INPUT_SCALES
        if self.is_static_input_scheme:
            assert (
                self.input_qscheme == "per_tensor"
            ), "Only per-tensor quantization is supported for static input scales"
            w13_input_scale = torch.nn.Parameter(
                torch.ones(num_experts, dtype=torch.float32), requires_grad=False
            )
            layer.register_parameter("w13_input_scale", w13_input_scale)
            set_weight_attrs(w13_input_scale, extra_weight_attrs)

            w2_input_scale = torch.nn.Parameter(
                torch.ones(num_experts, dtype=torch.float32), requires_grad=False
```
**EN:** This segment of `QuarkW8A8FP8MoE.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `QuarkW8A8FP8MoE.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 149-154: QuarkW8A8FP8MoE.create_weights() (part 3/3)
```python
            )
            layer.register_parameter("w2_input_scale", w2_input_scale)
            set_weight_attrs(w2_input_scale, extra_weight_attrs)
        else:
            layer.w13_input_scale = None
            layer.w2_input_scale = None
```
**EN:** This segment of `QuarkW8A8FP8MoE.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `QuarkW8A8FP8MoE.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 156-200: QuarkW8A8FP8MoE.process_weights_after_loading() (part 1/3)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        # Fp8 moe kernels require a single activation scale.
        # We take the max of all the scales in case they differ.
        if self.is_static_input_scheme:
            if layer.w13_input_scale is None or layer.w2_input_scale is None:
                raise ValueError(
                    "QuantConfig has static quantization, but found "
                    "activation scales are None."
                )
            if not all_close_1d(layer.w13_input_scale) or not all_close_1d(
                layer.w2_input_scale
            ):
                logger.warning(
                    "Found input_scales that are not equal for "
                    "fp8 MoE layer. Using the maximum across experts "
                    "for each layer."
                )
            layer.w13_input_scale = torch.nn.Parameter(
                layer.w13_input_scale.max(), requires_grad=False
            )
            layer.w2_input_scale = torch.nn.Parameter(
                layer.w2_input_scale.max(), requires_grad=False
            )

        if _is_fp8_fnuz:
            # Normalize the weights and scales
            w13_weight, w13_weight_scale, w13_input_scale = (
                normalize_e4m3fn_to_e4m3fnuz(
                    layer.w13_weight, layer.w13_weight_scale, layer.w13_input_scale
                )
            )
            w2_weight, w2_weight_scale, w2_input_scale = normalize_e4m3fn_to_e4m3fnuz(
                layer.w2_weight, layer.w2_weight_scale, layer.w2_input_scale
            )
            # Reset the parameter
            layer.w13_weight = torch.nn.Parameter(w13_weight, requires_grad=False)
            layer.w13_weight_scale = torch.nn.Parameter(
                w13_weight_scale, requires_grad=False
            )
            if w13_input_scale is not None:
                layer.w13_input_scale = torch.nn.Parameter(
                    w13_input_scale, requires_grad=False
                )
            layer.w2_weight = torch.nn.Parameter(w2_weight, requires_grad=False)
            layer.w2_weight_scale = torch.nn.Parameter(
```
**EN:** This segment of `QuarkW8A8FP8MoE.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `QuarkW8A8FP8MoE.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 201-245: QuarkW8A8FP8MoE.process_weights_after_loading() (part 2/3)
```python
                w2_weight_scale, requires_grad=False
            )
            if w2_input_scale is not None:
                layer.w2_input_scale = torch.nn.Parameter(
                    w2_input_scale, requires_grad=False
                )
        if self.weight_qscheme == "per_tensor":
            # Fp8 moe kernel needs single weight scale for w13 per expert.
            # We take the max then dequant and requant each expert.
            assert layer.w13_weight_scale is not None
            shard_size = layer.intermediate_size_per_partition
            max_w13_scales = layer.w13_weight_scale.max(dim=1).values
            for expert_id in range(layer.num_local_experts):
                start = 0
                for shard_id in range(2):
                    dq_weight = per_tensor_dequantize(
                        layer.w13_weight[expert_id][start : start + shard_size, :],
                        layer.w13_weight_scale[expert_id][shard_id],
                    )
                    (
                        layer.w13_weight[expert_id][start : start + shard_size, :],
                        _,
                    ) = scaled_fp8_quant(dq_weight, max_w13_scales[expert_id])

                    start += shard_size

            layer.w13_weight_scale = torch.nn.Parameter(
                max_w13_scales, requires_grad=False
            )
        elif self.weight_qscheme == "per_channel":
            layer.w13_weight_scale = torch.nn.Parameter(
                layer.w13_weight_scale.unsqueeze(-1), requires_grad=False
            )
            layer.w2_weight_scale = torch.nn.Parameter(
                layer.w2_weight_scale.unsqueeze(-1), requires_grad=False
            )
        else:
            raise ValueError(
                f"Unsupported weight quantization strategy: {self.weight_qscheme}."
            )

        if (
            _use_aiter
            and self.is_weight_per_channel
            and self.moe_runner_config.apply_router_weight_on_input
```
**EN:** This segment of `QuarkW8A8FP8MoE.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `QuarkW8A8FP8MoE.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 246-258: QuarkW8A8FP8MoE.process_weights_after_loading() (part 3/3)
```python
        ):
            with torch.no_grad():
                # Pre-shuffle weights
                layer.w13_weight = torch.nn.Parameter(
                    shuffle_weight(layer.w13_weight.data, (16, 16)),
                    requires_grad=False,
                )
                torch.cuda.empty_cache()
                layer.w2_weight = torch.nn.Parameter(
                    shuffle_weight(layer.w2_weight.data, (16, 16)),
                    requires_grad=False,
                )
                torch.cuda.empty_cache()
```
**EN:** This segment of `QuarkW8A8FP8MoE.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `QuarkW8A8FP8MoE.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 260-264: QuarkW8A8FP8MoE.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        self.moe_runner_config = moe_runner_config
        self.runner = MoeRunner(MoeRunnerBackend.TRITON, moe_runner_config)
```
**EN:** This block defines `QuarkW8A8FP8MoE.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `QuarkW8A8FP8MoE.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 266-312: QuarkW8A8FP8MoE.apply_weights()
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:

        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput

        x = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output

        moe_runner_config = self.moe_runner_config

        if (
            _use_aiter
            and self.is_weight_per_channel
            and moe_runner_config.apply_router_weight_on_input
        ):
            topk_weights, topk_ids, _ = topk_output
            output = rocm_fused_experts_tkw1(
                hidden_states=x,
                w1=layer.w13_weight,
                w2=layer.w2_weight,
                topk_weights=topk_weights,
                topk_ids=topk_ids,
                activation=moe_runner_config.activation,
                apply_router_weight_on_input=moe_runner_config.apply_router_weight_on_input,
                use_fp8_w8a8=True,
                per_channel_quant=self.is_weight_per_channel,
                w1_scale=layer.w13_weight_scale,
                w2_scale=layer.w2_weight_scale,
                a1_scale=layer.w13_input_scale,
                a2_scale=layer.w2_input_scale,
            )
            return StandardCombineInput(hidden_states=output)
        else:
            quant_info = TritonMoeQuantInfo(
                w13_weight=layer.w13_weight,
                w2_weight=layer.w2_weight,
                use_fp8_w8a8=True,
                per_channel_quant=self.is_weight_per_channel,
                w13_scale=layer.w13_weight_scale,
                w2_scale=layer.w2_weight_scale,
                a13_scale=layer.w13_input_scale,
                a2_scale=layer.w2_input_scale,
            )
            return self.runner.run(dispatch_output, quant_info)
```
**EN:** This block defines `QuarkW8A8FP8MoE.apply_weights()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `QuarkW8A8FP8MoE.apply_weights()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `QuarkW8A8FP8MoE`: A scheme class that structures file-level quantization behavior. / `QuarkW8A8FP8MoE` 是一个执行方案类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `aiter.ops.shuffle`, `logging`, `torch`, `typing`
- **Internal / 内部**: `sglang.srt.layers.moe`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.moe_runner.triton`, `sglang.srt.layers.moe.rocm_moe_utils`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.quantization.fp8_kernel`, `sglang.srt.layers.quantization.fp8_utils`, `sglang.srt.layers.quantization.quark.schemes`, `sglang.srt.layers.quantization.utils`, `sglang.srt.utils`
