# mxfp4_marlin_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/mxfp4_marlin_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements runtime method classes that apply mxfp4 marlin moe quantized weights during inference. / 该模块实现了推理期运行方法类，用于应用 MXFP4 Marlin MoE 量化权重。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: module imports and setup
```python
from __future__ import annotations

import logging
from typing import TYPE_CHECKING

import torch
from torch.nn import Module

from sglang.srt.layers.moe.moe_runner.marlin import MarlinMoeQuantInfo
from sglang.srt.layers.moe.utils import MoeRunnerBackend
from sglang.srt.utils import log_info_on_rank0, set_weight_attrs
from sglang.srt.utils.common import is_sm90_supported
```
**EN:** This block imports __future__, logging, torch, torch.nn, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.moe.moe_runner, sglang.srt.layers.moe.moe_runner.marlin, sglang.srt.layers.moe.token_dispatcher and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, logging, torch, torch.nn, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.moe.moe_runner, sglang.srt.layers.moe.moe_runner.marlin, sglang.srt.layers.moe.token_dispatcher 等依赖，并为当前量化实现准备模块命名空间。

### Lines 14-15: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import CombineInput, DispatchOutput
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 17-17: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 20-22: class Mxfp4MarlinMoEMethod: definition
```python
class Mxfp4MarlinMoEMethod:
    """MXFP4 (E8M0 scales) MoE quantization method using the Marlin backend."""
```
**EN:** This block declares `Mxfp4MarlinMoEMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_moe_runner, create_weights, process_weights_after_loading.
**CN:** 该代码块声明 `Mxfp4MarlinMoEMethod`，它是量化栈中的运行方法类，组织了 __init__, create_moe_runner, create_weights, process_weights_after_loading 等行为。

### Lines 23-25: Mxfp4MarlinMoEMethod.__init__()
```python
    def __init__(self, fp8_method, prefix: str):
        self._fp8 = fp8_method
        self.prefix = prefix
```
**EN:** This block defines `Mxfp4MarlinMoEMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `Mxfp4MarlinMoEMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 27-30: Mxfp4MarlinMoEMethod.create_moe_runner()
```python
    def create_moe_runner(self, layer, moe_runner_config):
        from sglang.srt.layers.moe.moe_runner import MoeRunner

        self.runner = MoeRunner(MoeRunnerBackend.MARLIN, moe_runner_config)
```
**EN:** This block defines `Mxfp4MarlinMoEMethod.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `Mxfp4MarlinMoEMethod.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 32-95: Mxfp4MarlinMoEMethod.create_weights()
```python
    def create_weights(
        self,
        layer: Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        from sglang.srt.layers.moe.fused_moe_triton import (
            FusedMoeWeightScaleSupported,
        )

        fp4_block_k = 32

        w13_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                2 * intermediate_size_per_partition,
                hidden_size // 2,
                dtype=torch.int8,
            ),
            requires_grad=False,
        )
        w2_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size,
                intermediate_size_per_partition // 2,
                dtype=torch.int8,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight", w13_weight)
        set_weight_attrs(w13_weight, extra_weight_attrs)
        layer.register_parameter("w2_weight", w2_weight)
        set_weight_attrs(w2_weight, extra_weight_attrs)

        w13_weight_scale = torch.nn.Parameter(
            torch.ones(
                num_experts,
                2 * intermediate_size_per_partition,
                hidden_size // fp4_block_k,
                dtype=torch.float32,
            ),
            requires_grad=False,
        )
        w2_weight_scale = torch.nn.Parameter(
            torch.ones(
                num_experts,
                hidden_size,
                intermediate_size_per_partition // fp4_block_k,
                dtype=torch.float32,
            ),
            requires_grad=False,
        )
        w13_weight_scale.format_ue8m0 = False
        w2_weight_scale.format_ue8m0 = False
        scale_attrs = dict(extra_weight_attrs)
        scale_attrs["quant_method"] = FusedMoeWeightScaleSupported.BLOCK.value
        layer.register_parameter("w13_weight_scale_inv", w13_weight_scale)
        set_weight_attrs(w13_weight_scale, scale_attrs)
        layer.register_parameter("w2_weight_scale_inv", w2_weight_scale)
        set_weight_attrs(w2_weight_scale, scale_attrs)
```
**EN:** This block defines `Mxfp4MarlinMoEMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `Mxfp4MarlinMoEMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 97-133: Mxfp4MarlinMoEMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: Module) -> None:
        from sglang.srt.layers.quantization.marlin_utils import (
            check_moe_marlin_supports_layer,
        )
        from sglang.srt.layers.quantization.marlin_utils_fp4 import (
            prepare_moe_mxfp4_layer_for_marlin,
        )

        # Let the FP8 base method handle ROCm normalization, etc.
        self._fp8.process_weights_after_loading(layer)

        if getattr(layer, "_mega_moe_weights_built", False):
            return

        if not is_sm90_supported():
            raise RuntimeError(
                "DeepSeekV4 MXFP4 Marlin fallback requires Hopper/SM90 or above."
            )
        if not check_moe_marlin_supports_layer(layer, 32):
            raise RuntimeError(
                "Current DeepSeekV4 MoE layer does not satisfy Marlin constraints."
            )

        # NOTE: the Marlin MoE runner consumes w13 in the checkpoint's
        # native ``[w1; w3]`` order -- see ``silu_and_mul`` in
        # fused_marlin_moe.py which expects ``gate = intermediate[:, :N]``
        # (first half) and ``up = intermediate[:, N:]`` (second half).
        # Unlike the flashinfer trtllm_fp4 kernel (which wants [w3, w1]),
        # we must *not* call ``reorder_w1w3_to_w3w1`` here.

        log_info_on_rank0(
            logger,
            f"Preparing DeepSeekV4 MXFP4 experts for Marlin backend "
            f"(layer: {self.prefix})...",
        )
        prepare_moe_mxfp4_layer_for_marlin(layer)
        layer._dsv4_mxfp4_backend = "marlin"
```
**EN:** This block defines `Mxfp4MarlinMoEMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `Mxfp4MarlinMoEMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 135-159: Mxfp4MarlinMoEMethod.apply()
```python
    def apply(
        self,
        layer: Module,
        dispatch_output: DispatchOutput,
    ) -> CombineInput:
        from sglang.srt.layers.moe.token_dispatcher.standard import StandardCombineInput
        from sglang.srt.layers.moe.topk import TopKOutputChecker

        topk_output = dispatch_output.topk_output
        if not TopKOutputChecker.format_is_standard(topk_output):
            raise ValueError(f"Unsupported topk output format: {topk_output.format}")

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
        runner_output = self.runner.run(dispatch_output, quant_info=quant_info)

        return StandardCombineInput(hidden_states=runner_output.hidden_states)
```
**EN:** This block defines `Mxfp4MarlinMoEMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `Mxfp4MarlinMoEMethod.apply()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `Mxfp4MarlinMoEMethod`: A runtime method class that structures file-level quantization behavior. / `Mxfp4MarlinMoEMethod` 是一个运行方法类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `logging`, `torch`, `torch.nn`, `typing`
- **Internal / 内部**: `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.moe_runner`, `sglang.srt.layers.moe.moe_runner.marlin`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.moe.token_dispatcher.standard`, `sglang.srt.layers.moe.topk`, `sglang.srt.layers.moe.utils`, `sglang.srt.layers.quantization.marlin_utils`, `sglang.srt.layers.quantization.marlin_utils_fp4`, `sglang.srt.utils`, `sglang.srt.utils.common`
