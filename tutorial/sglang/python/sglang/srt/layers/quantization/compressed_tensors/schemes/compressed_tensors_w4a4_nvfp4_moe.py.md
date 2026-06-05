# compressed_tensors_w4a4_nvfp4_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/compressed_tensors/schemes/compressed_tensors_w4a4_nvfp4_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains supporting code for compressed tensors w4a4 nvfp4 moe quantization in the SGLang SRT layer stack. / 该模块包含 SGLang SRT 层栈中与 压缩 张量 w4a4 NVFP4 MoE 量化相关的支撑代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: module imports and setup
```python
from __future__ import annotations

import logging
from typing import TYPE_CHECKING

import torch

from sglang.srt.distributed import get_tp_group
from sglang.srt.distributed.device_communicators.pynccl_allocator import (
    use_symmetric_memory,
)
from sglang.srt.layers.dp_attention import is_allocation_symmetric
from sglang.srt.layers.moe import MoeRunner, MoeRunnerBackend, MoeRunnerConfig
from sglang.srt.layers.moe.cutlass_moe_params import CutlassMoEParams, CutlassMoEType
from sglang.srt.layers.moe.utils import RoutingMethodType, get_moe_runner_backend
from sglang.srt.layers.quantization.compressed_tensors.schemes import (
    CompressedTensorsMoEScheme,
)
from sglang.srt.layers.quantization.fp8_utils import is_blackwell_supported
from sglang.srt.layers.quantization.utils import (
    prepare_static_weights_for_trtllm_fp4_moe,
    reorder_w1w3_to_w3w1,
    replace_parameter,
    swizzle_blockscale,
)
from sglang.srt.utils import next_power_of_2, set_weight_attrs
```
**EN:** This block imports __future__, flashinfer, logging, torch, sglang.srt.distributed, sglang.srt.distributed.device_communicators.pynccl_allocator, sglang.srt.layers.dp_attention, sglang.srt.layers.moe and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, flashinfer, logging, torch, sglang.srt.distributed, sglang.srt.distributed.device_communicators.pynccl_allocator, sglang.srt.layers.dp_attention, sglang.srt.layers.moe 等依赖，并为当前量化实现准备模块命名空间。

### Lines 28-28: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 30-30: initialize __all__
```python
__all__ = ["CompressedTensorsW4A4Nvfp4MoE"]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as __all__.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 __all__。

### Lines 32-36: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        StandardDispatchOutput,
    )
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 39-40: class CompressedTensorsW4A4Nvfp4MoE: definition
```python
class CompressedTensorsW4A4Nvfp4MoE(CompressedTensorsMoEScheme):
```
**EN:** This block declares `CompressedTensorsW4A4Nvfp4MoE`, a scheme class for the quantization stack. It organizes behaviors such as __init__, get_min_capability, create_weights, process_weights_after_loading.
**CN:** 该代码块声明 `CompressedTensorsW4A4Nvfp4MoE`，它是量化栈中的执行方案类，组织了 __init__, get_min_capability, create_weights, process_weights_after_loading 等行为。

### Lines 41-49: CompressedTensorsW4A4Nvfp4MoE.__init__()
```python
    def __init__(self):
        if not is_blackwell_supported():
            raise ValueError(
                "Current platform does not support NVFP4"
                " quantization. Please use Blackwell and"
                " above."
            )
        self.group_size = 16
        self.use_flashinfer_trtllm = get_moe_runner_backend().is_flashinfer_trtllm()
```
**EN:** This block defines `CompressedTensorsW4A4Nvfp4MoE.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `CompressedTensorsW4A4Nvfp4MoE.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 51-54: CompressedTensorsW4A4Nvfp4MoE.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        # Requires sm100(blackwell) architecture
        return 100
```
**EN:** This block defines `CompressedTensorsW4A4Nvfp4MoE.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `CompressedTensorsW4A4Nvfp4MoE.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 56-100: CompressedTensorsW4A4Nvfp4MoE.create_weights() (part 1/3)
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

        layer.params_dtype = params_dtype

        w13_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                2 * intermediate_size_per_partition,
                # 2 fp4 items are packed in the input dimension
                hidden_size // 2,
                requires_grad=False,
                dtype=torch.uint8,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight_packed", w13_weight)
        set_weight_attrs(w13_weight, extra_weight_attrs)

        w2_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size,
                # 2 fp4 items are packed in the input dimension
                intermediate_size_per_partition // 2,
                dtype=torch.uint8,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight_packed", w2_weight)
        set_weight_attrs(w2_weight, extra_weight_attrs)

        # Weight Scales
        w13_weight_scale = torch.nn.Parameter(
            torch.empty(
                num_experts,
                2 * intermediate_size_per_partition,
```
**EN:** This segment of `CompressedTensorsW4A4Nvfp4MoE.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `CompressedTensorsW4A4Nvfp4MoE.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 101-145: CompressedTensorsW4A4Nvfp4MoE.create_weights() (part 2/3)
```python
                # 2 fp4 items are packed in the input dimension
                hidden_size // self.group_size,
                dtype=torch.float8_e4m3fn,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight_scale", w13_weight_scale)
        extra_weight_attrs.update(
            {"quant_method": FusedMoeWeightScaleSupported.GROUP.value}
        )
        set_weight_attrs(w13_weight_scale, extra_weight_attrs)

        w2_weight_scale = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size,
                # 2 fp4 items are packed in the input dimension
                intermediate_size_per_partition // self.group_size,
                dtype=torch.float8_e4m3fn,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight_scale", w2_weight_scale)
        extra_weight_attrs.update(
            {"quant_method": FusedMoeWeightScaleSupported.GROUP.value}
        )
        set_weight_attrs(w2_weight_scale, extra_weight_attrs)

        # Weight Global Scales
        w13_weight_scale_2 = torch.nn.Parameter(
            torch.empty(num_experts, 2, dtype=torch.float32), requires_grad=False
        )
        layer.register_parameter("w13_weight_global_scale", w13_weight_scale_2)
        extra_weight_attrs.update(
            {"quant_method": FusedMoeWeightScaleSupported.TENSOR.value}
        )
        set_weight_attrs(w13_weight_scale_2, extra_weight_attrs)

        w2_weight_scale_2 = torch.nn.Parameter(
            torch.empty(num_experts, dtype=torch.float32), requires_grad=False
        )
        layer.register_parameter("w2_weight_global_scale", w2_weight_scale_2)
        extra_weight_attrs.update(
            {"quant_method": FusedMoeWeightScaleSupported.TENSOR.value}
        )
```
**EN:** This segment of `CompressedTensorsW4A4Nvfp4MoE.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `CompressedTensorsW4A4Nvfp4MoE.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 146-165: CompressedTensorsW4A4Nvfp4MoE.create_weights() (part 3/3)
```python
        set_weight_attrs(w2_weight_scale_2, extra_weight_attrs)

        # Input Global Scales
        w13_input_scale = torch.nn.Parameter(
            torch.empty(num_experts, 2, dtype=torch.float32), requires_grad=False
        )
        layer.register_parameter("w13_input_global_scale", w13_input_scale)
        extra_weight_attrs.update(
            {"quant_method": FusedMoeWeightScaleSupported.TENSOR.value}
        )
        set_weight_attrs(w13_input_scale, extra_weight_attrs)

        w2_input_scale = torch.nn.Parameter(
            torch.empty(num_experts, dtype=torch.float32), requires_grad=False
        )
        layer.register_parameter("w2_input_global_scale", w2_input_scale)
        extra_weight_attrs.update(
            {"quant_method": FusedMoeWeightScaleSupported.TENSOR.value}
        )
        set_weight_attrs(w2_input_scale, extra_weight_attrs)
```
**EN:** This segment of `CompressedTensorsW4A4Nvfp4MoE.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `CompressedTensorsW4A4Nvfp4MoE.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 167-211: CompressedTensorsW4A4Nvfp4MoE.process_weights_after_loading() (part 1/3)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        # From packed to weight
        layer.w13_weight = torch.nn.Parameter(
            layer.w13_weight_packed.data, requires_grad=False
        )
        delattr(layer, "w13_weight_packed")

        layer.w2_weight = torch.nn.Parameter(
            layer.w2_weight_packed.data, requires_grad=False
        )
        delattr(layer, "w2_weight_packed")

        if self.use_flashinfer_trtllm:
            w, s = reorder_w1w3_to_w3w1(
                layer.w13_weight.data, layer.w13_weight_scale.data, dim=-2
            )
            layer.w13_weight = torch.nn.Parameter(w, requires_grad=False)
            layer.w13_weight_scale = torch.nn.Parameter(s, requires_grad=False)

        if not torch.allclose(
            layer.w13_weight_global_scale[:, 0], layer.w13_weight_global_scale[:, 1]
        ):
            logger.warning_once(
                "w1_weight_global_scale must match w3_weight_global_scale. "
                "Accuracy may be affected."
            )

        # Take inverse of global scale saved to disk
        layer.w13_weight_scale_2 = torch.nn.Parameter(
            1 / layer.w13_weight_global_scale[:, 0], requires_grad=False
        )

        layer.w2_weight_scale_2 = torch.nn.Parameter(
            1 / layer.w2_weight_global_scale.data, requires_grad=False
        )

        # w13
        if self.use_flashinfer_trtllm:
            w13_input_global_scale = (
                layer.w13_input_global_scale.min()
                .to(torch.float32)
                .expand(layer.num_local_experts)
            )
        else:
            w13_input_global_scale = layer.w13_input_global_scale.min(dim=1).values.to(
```
**EN:** This segment of `CompressedTensorsW4A4Nvfp4MoE.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `CompressedTensorsW4A4Nvfp4MoE.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 212-256: CompressedTensorsW4A4Nvfp4MoE.process_weights_after_loading() (part 2/3)
```python
                torch.float32
            )
        layer.g1_alphas = torch.nn.Parameter(
            ((1 / w13_input_global_scale) * layer.w13_weight_scale_2),
            requires_grad=False,
        )

        layer.w13_input_scale_quant = torch.nn.Parameter(
            (w13_input_global_scale), requires_grad=False
        )

        # w2
        if self.use_flashinfer_trtllm:
            w2_input_global_scale = (
                layer.w2_input_global_scale.min()
                .to(torch.float32)
                .expand(layer.num_local_experts)
            )
        else:
            w2_input_global_scale = layer.w2_input_global_scale

        layer.g2_alphas = torch.nn.Parameter(
            ((1 / w2_input_global_scale) * layer.w2_weight_scale_2).to(torch.float32),
            requires_grad=False,
        )

        layer.w2_input_scale_quant = torch.nn.Parameter(
            (w2_input_global_scale), requires_grad=False
        )

        # TensorRT-LLM specific processing
        if self.use_flashinfer_trtllm:
            # Prepare static weights for TRT-LLM kernel
            (
                gemm1_weights_fp4_shuffled,
                gemm1_scales_fp4_shuffled,
                gemm2_weights_fp4_shuffled,
                gemm2_scales_fp4_shuffled,
            ) = prepare_static_weights_for_trtllm_fp4_moe(
                layer.w13_weight,
                layer.w2_weight,
                layer.w13_weight_scale,
                layer.w2_weight_scale,
                layer.w2_weight.size(-2),  # hidden_size
                layer.w13_weight.size(-2) // 2,  # intermediate_size
```
**EN:** This segment of `CompressedTensorsW4A4Nvfp4MoE.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `CompressedTensorsW4A4Nvfp4MoE.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 257-287: CompressedTensorsW4A4Nvfp4MoE.process_weights_after_loading() (part 3/3)
```python
                layer.w13_weight.size(0),  # num_experts
            )
            logger.debug("Finished shuffling weights for TRT-LLM MOE")

            replace_parameter(layer, "w13_weight", gemm1_weights_fp4_shuffled)
            replace_parameter(layer, "w2_weight", gemm2_weights_fp4_shuffled)
            replace_parameter(layer, "w13_weight_scale", gemm1_scales_fp4_shuffled)
            replace_parameter(layer, "w2_weight_scale", gemm2_scales_fp4_shuffled)

            # Additional parameter needed for TRT-LLM
            layer.g1_scale_c = torch.nn.Parameter(
                (layer.w2_input_scale_quant * layer.g1_alphas).to(torch.float32),
                requires_grad=False,
            )
        else:
            # swizzle weight scales
            layer.w13_weight_scale = torch.nn.Parameter(
                swizzle_blockscale(layer.w13_weight_scale), requires_grad=False
            )

            layer.w2_weight_scale = torch.nn.Parameter(
                swizzle_blockscale(layer.w2_weight_scale), requires_grad=False
            )

            layer.cutlass_moe_params = CutlassMoEParams(
                CutlassMoEType.BlockscaledFP4,
                layer.w13_weight.device,
                num_experts=layer.num_experts,
                intermediate_size_per_partition=layer.w2_weight.shape[2] * 2,
                hidden_size=layer.w13_weight.shape[2] * 2,
            )
```
**EN:** This segment of `CompressedTensorsW4A4Nvfp4MoE.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `CompressedTensorsW4A4Nvfp4MoE.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 289-293: CompressedTensorsW4A4Nvfp4MoE.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        self.moe_runner_config = moe_runner_config
        self.runner = MoeRunner(MoeRunnerBackend.TRITON, moe_runner_config)
```
**EN:** This block defines `CompressedTensorsW4A4Nvfp4MoE.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `CompressedTensorsW4A4Nvfp4MoE.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 295-339: CompressedTensorsW4A4Nvfp4MoE.apply_weights() (part 1/3)
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:

        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput

        x = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output

        if self.use_flashinfer_trtllm:
            from flashinfer import trtllm_fp4_block_scale_moe

            from sglang.srt.layers.quantization.fp4_utils import fp4_quantize

            router_logits = topk_output.router_logits
            topk_config = topk_output.topk_config

            # global_scale must be shape [1] (strict in cute-dsl backend).
            hs_fp4_bytes, hs_sf_bytes = fp4_quantize(
                x,
                layer.w13_input_scale_quant[:1],
                self.group_size,  # sf_vec_size
                False,  # use_ue8m0
                False,  # is_sf_swizzled_layout
            )
            hs_fp4 = hs_fp4_bytes.reshape(x.shape[0], x.shape[1] // 2)
            hs_scale = hs_sf_bytes.view(torch.float8_e4m3fn).reshape(
                *hs_sf_bytes.shape[:-1], -1
            )

            correction_bias = (
                None
                if topk_config.correction_bias is None
                else topk_config.correction_bias.to(x.dtype)
            )

            assert layer.routing_method_type is not None

            # DeepSeekV3 style routing requires float32 router logits
            if layer.routing_method_type == RoutingMethodType.DeepSeekV3:
                router_logits = router_logits.to(torch.float32)

            routed_scaling_factor = self.moe_runner_config.routed_scaling_factor
```
**EN:** This segment of `CompressedTensorsW4A4Nvfp4MoE.apply_weights()` applies quantized computation to runtime inputs. It covers one portion of the full implementation.
**CN:** `CompressedTensorsW4A4Nvfp4MoE.apply_weights()` 的这一段代码用于将量化计算应用到运行时输入上，展示了完整实现中的一部分。

### Lines 340-384: CompressedTensorsW4A4Nvfp4MoE.apply_weights() (part 2/3)
```python
            routed_scaling_factor = (
                routed_scaling_factor if routed_scaling_factor is not None else 1.0
            )

            with use_symmetric_memory(
                get_tp_group(), disabled=not is_allocation_symmetric()
            ):
                num_tokens = hs_fp4.shape[0]
                hidden_size = (
                    hs_fp4.shape[-1] * 2
                    if hs_fp4.dtype == torch.uint8
                    else hs_fp4.shape[-1]
                )
                symm_output = torch.empty(
                    num_tokens, hidden_size, dtype=torch.bfloat16, device=hs_fp4.device
                )

            output = trtllm_fp4_block_scale_moe(
                routing_logits=router_logits,
                routing_bias=correction_bias,
                hidden_states=hs_fp4,
                hidden_states_scale=hs_scale,
                gemm1_weights=layer.w13_weight,
                gemm1_weights_scale=layer.w13_weight_scale.view(torch.float8_e4m3fn),
                gemm1_bias=None,
                gemm1_alpha=None,
                gemm1_beta=None,
                gemm1_clamp_limit=None,
                gemm2_weights=layer.w2_weight,
                gemm2_weights_scale=layer.w2_weight_scale.view(torch.float8_e4m3fn),
                gemm2_bias=None,
                output1_scale_scalar=layer.g1_scale_c,
                output1_scale_gate_scalar=layer.g1_alphas,
                output2_scale_scalar=layer.g2_alphas,
                num_experts=layer.num_experts,
                top_k=topk_config.top_k,
                n_group=topk_config.num_expert_group,
                topk_group=topk_config.topk_group,
                intermediate_size=layer.intermediate_size_per_partition,
                local_expert_offset=layer.moe_ep_rank * layer.num_local_experts,
                local_num_experts=layer.num_local_experts,
                routed_scaling_factor=routed_scaling_factor,
                routing_method_type=layer.routing_method_type,
                do_finalize=True,
                tune_max_num_tokens=next_power_of_2(hs_fp4.shape[0]),
```
**EN:** This segment of `CompressedTensorsW4A4Nvfp4MoE.apply_weights()` applies quantized computation to runtime inputs. It covers one portion of the full implementation.
**CN:** `CompressedTensorsW4A4Nvfp4MoE.apply_weights()` 的这一段代码用于将量化计算应用到运行时输入上，展示了完整实现中的一部分。

### Lines 385-408: CompressedTensorsW4A4Nvfp4MoE.apply_weights() (part 3/3)
```python
                output=symm_output,
            )[0]
        else:
            from sglang.srt.layers.moe.cutlass_moe import cutlass_moe_fp4

            topk_weights, topk_ids = topk_output.topk_weights, topk_output.topk_ids

            output = cutlass_moe_fp4(
                a=x,
                a1_gscale=layer.w13_input_scale_quant,
                w1_fp4=layer.w13_weight,
                w1_blockscale=layer.w13_weight_scale,
                w1_alphas=layer.g1_alphas,
                a2_gscale=layer.w2_input_scale_quant,
                w2_fp4=layer.w2_weight,
                w2_blockscale=layer.w2_weight_scale,
                w2_alphas=layer.g2_alphas,
                topk_weights=topk_weights,
                topk_ids=topk_ids,
                params=layer.cutlass_moe_params,
                apply_router_weight_on_input=self.moe_runner_config.apply_router_weight_on_input,
            ).to(x.dtype)

        return StandardCombineInput(hidden_states=output)
```
**EN:** This segment of `CompressedTensorsW4A4Nvfp4MoE.apply_weights()` applies quantized computation to runtime inputs. It covers one portion of the full implementation.
**CN:** `CompressedTensorsW4A4Nvfp4MoE.apply_weights()` 的这一段代码用于将量化计算应用到运行时输入上，展示了完整实现中的一部分。

## Key Concepts / 关键概念
- `CompressedTensorsW4A4Nvfp4MoE`: A scheme class that structures file-level quantization behavior. / `CompressedTensorsW4A4Nvfp4MoE` 是一个执行方案类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `flashinfer`, `logging`, `torch`, `typing`
- **Internal / 内部**: `sglang.srt.distributed`, `sglang.srt.distributed.device_communicators.pynccl_allocator`, `sglang.srt.layers.dp_attention`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.cutlass_moe`, `sglang.srt.layers.moe.cutlass_moe_params`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.moe.utils`, `sglang.srt.layers.quantization.compressed_tensors.schemes`, `sglang.srt.layers.quantization.fp4_utils`, `sglang.srt.layers.quantization.fp8_utils`, `sglang.srt.layers.quantization.utils`, `sglang.srt.utils`
