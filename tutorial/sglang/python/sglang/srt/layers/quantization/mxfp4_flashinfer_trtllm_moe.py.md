# mxfp4_flashinfer_trtllm_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/mxfp4_flashinfer_trtllm_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements runtime method classes that apply mxfp4 flashinfer trtllm moe quantized weights during inference. / 该模块实现了推理期运行方法类，用于应用 MXFP4 FlashInfer trtllm MoE 量化权重。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: module imports and setup
```python
from __future__ import annotations

import logging
from typing import TYPE_CHECKING

import torch
import triton
import triton.language as tl
from torch.nn import Module
from torch.nn.parameter import Parameter

from sglang.srt.distributed import get_tp_group
from sglang.srt.distributed.device_communicators.pynccl_allocator import (
    use_symmetric_memory,
)
from sglang.srt.layers.dp_attention import is_allocation_symmetric
from sglang.srt.layers.moe.utils import RoutingMethodType
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import (
    is_flashinfer_available,
    log_info_on_rank0,
    set_weight_attrs,
)
from sglang.srt.utils.common import next_power_of_2
```
**EN:** This block imports __future__, flashinfer, flashinfer.fp4_quantization, flashinfer.fused_moe, sglang.srt.distributed, sglang.srt.distributed.device_communicators.pynccl_allocator, sglang.srt.layers.dp_attention, sglang.srt.layers.moe.fused_moe_triton and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, flashinfer, flashinfer.fp4_quantization, flashinfer.fused_moe, sglang.srt.distributed, sglang.srt.distributed.device_communicators.pynccl_allocator, sglang.srt.layers.dp_attention, sglang.srt.layers.moe.fused_moe_triton 等依赖，并为当前量化实现准备模块命名空间。

### Lines 26-33: conditional logic for is_flashinfer_available()
```python
if is_flashinfer_available():
    from flashinfer import mxfp8_quantize, shuffle_matrix_a, shuffle_matrix_sf_a
    from flashinfer.fp4_quantization import block_scale_interleave
    from flashinfer.fused_moe import trtllm_fp4_block_scale_routed_moe
    from flashinfer.fused_moe.core import (
        _maybe_get_cached_w3_w1_permute_indices,
        get_w2_permute_indices_with_cache,
    )
```
**EN:** This block applies conditional logic controlled by `is_flashinfer_available()`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `is_flashinfer_available()` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 35-35: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 37-38: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import CombineInput, DispatchOutput
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 40-40: module imports and setup
```python
from sglang.srt.utils.common import get_bool_env_var
```
**EN:** This block imports __future__, flashinfer, flashinfer.fp4_quantization, flashinfer.fused_moe, sglang.srt.distributed, sglang.srt.distributed.device_communicators.pynccl_allocator, sglang.srt.layers.dp_attention, sglang.srt.layers.moe.fused_moe_triton and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, flashinfer, flashinfer.fp4_quantization, flashinfer.fused_moe, sglang.srt.distributed, sglang.srt.distributed.device_communicators.pynccl_allocator, sglang.srt.layers.dp_attention, sglang.srt.layers.moe.fused_moe_triton 等依赖，并为当前量化实现准备模块命名空间。

### Lines 42-44: initialize _USE_OFFICIAL_SHUFFLE
```python
_USE_OFFICIAL_SHUFFLE = get_bool_env_var(
    "SGLANG_MXFP4_USE_OFFICIAL_SHUFFLE", default="true"
)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _USE_OFFICIAL_SHUFFLE.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _USE_OFFICIAL_SHUFFLE。

### Lines 47-48: class PackTopkIds: definition
```python
class PackTopkIds:
```
**EN:** This block declares `PackTopkIds`, a supporting class for the quantization stack. It organizes behaviors such as execute, vanilla, triton.
**CN:** 该代码块声明 `PackTopkIds`，它是量化栈中的支撑类，组织了 execute, vanilla, triton 等行为。

### Lines 49-53: PackTopkIds.execute()
```python
    @classmethod
    def execute(
        cls, topk_ids: torch.Tensor, topk_weights: torch.Tensor
    ) -> torch.Tensor:
        return cls.triton(topk_ids, topk_weights)
```
**EN:** This block defines `PackTopkIds.execute()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `PackTopkIds.execute()`，用于实现量化栈中的可复用模块逻辑。

### Lines 55-62: PackTopkIds.vanilla()
```python
    @classmethod
    def vanilla(
        cls, topk_ids: torch.Tensor, topk_weights: torch.Tensor
    ) -> torch.Tensor:
        weight_bits = (
            topk_weights.to(torch.bfloat16).view(torch.int16).to(torch.int32) & 0xFFFF
        )
        return (topk_ids.to(torch.int32) << 16) | weight_bits
```
**EN:** This block defines `PackTopkIds.vanilla()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `PackTopkIds.vanilla()`，用于实现量化栈中的可复用模块逻辑。

### Lines 64-95: PackTopkIds.triton()
```python
    @classmethod
    def triton(cls, topk_ids: torch.Tensor, topk_weights: torch.Tensor) -> torch.Tensor:
        assert (
            topk_ids.shape == topk_weights.shape
        ), f"shape mismatch: {topk_ids.shape=} vs {topk_weights.shape=}"
        assert topk_ids.ndim >= 1, f"expected >=1D, got {topk_ids.shape=}"

        assert (
            topk_ids.dtype == torch.int32
        ), f"topk_ids must be int32, got {topk_ids.dtype}"
        assert (
            topk_weights.dtype == torch.float32
        ), f"topk_weights must be float32, got {topk_weights.dtype}"

        assert topk_ids.is_contiguous(), "topk_ids must be contiguous"
        assert topk_weights.is_contiguous(), "topk_weights must be contiguous"

        out = torch.empty_like(topk_ids, dtype=torch.int32)
        numel = out.numel()
        if numel == 0:
            return out

        BLOCK_SIZE = 1024
        grid = (triton.cdiv(numel, BLOCK_SIZE),)
        _pack_topk_ids_triton_kernel[grid](
            topk_ids,
            topk_weights,
            out,
            numel,
            BLOCK_SIZE=BLOCK_SIZE,
        )
        return out
```
**EN:** This block defines `PackTopkIds.triton()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `PackTopkIds.triton()`，用于实现量化栈中的可复用模块逻辑。

### Lines 98-120: _pack_topk_ids_triton_kernel()
```python
@triton.jit
def _pack_topk_ids_triton_kernel(
    topk_ids_ptr,
    topk_weights_ptr,
    out_ptr,
    numel,
    BLOCK_SIZE: tl.constexpr,
):
    pid = tl.program_id(0)
    offsets = pid * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
    mask = offsets < numel

    ids = tl.load(topk_ids_ptr + offsets, mask=mask, other=0)
    w = tl.load(topk_weights_ptr + offsets, mask=mask, other=0.0)

    w_bf16 = w.to(tl.bfloat16)
    w_i16 = w_bf16.to(tl.int16, bitcast=True)
    w_i32 = w_i16.to(tl.int32) & 0xFFFF

    ids_i32 = ids.to(tl.int32)
    packed = (ids_i32 << 16) | w_i32

    tl.store(out_ptr + offsets, packed, mask=mask)
```
**EN:** This block defines `_pack_topk_ids_triton_kernel()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `_pack_topk_ids_triton_kernel()`，用于实现量化栈中的可复用模块逻辑。

### Lines 123-124: class Mxfp4FlashinferTrtllmMoEMethod: definition
```python
class Mxfp4FlashinferTrtllmMoEMethod:
```
**EN:** This block declares `Mxfp4FlashinferTrtllmMoEMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_moe_runner, create_weights, process_weights_after_loading.
**CN:** 该代码块声明 `Mxfp4FlashinferTrtllmMoEMethod`，它是量化栈中的运行方法类，组织了 __init__, create_moe_runner, create_weights, process_weights_after_loading 等行为。

### Lines 125-130: Mxfp4FlashinferTrtllmMoEMethod.__init__()
```python
    def __init__(self, fp8_method, prefix: str):
        self._fp8 = fp8_method
        self.prefix = prefix
        self.flashinfer_mxfp4_moe_precision = (
            get_global_server_args().flashinfer_mxfp4_moe_precision
        )
```
**EN:** This block defines `Mxfp4FlashinferTrtllmMoEMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `Mxfp4FlashinferTrtllmMoEMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 132-148: Mxfp4FlashinferTrtllmMoEMethod.create_moe_runner()
```python
    def create_moe_runner(self, layer, moe_runner_config):
        self.moe_runner_config = moe_runner_config

        swiglu_limit = moe_runner_config.swiglu_limit
        assert (
            swiglu_limit is not None
        ), f"swiglu_limit must be non-None for DeepSeek V4 (got {swiglu_limit!r})"
        self._gemm1_clamp_limit_tensor = (
            torch.full(
                (layer.num_local_experts,),
                swiglu_limit,
                dtype=torch.float32,
                device=layer.w13_weight.device,
            )
            if swiglu_limit is not None
            else None
        )
```
**EN:** This block defines `Mxfp4FlashinferTrtllmMoEMethod.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `Mxfp4FlashinferTrtllmMoEMethod.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 150-211: Mxfp4FlashinferTrtllmMoEMethod.create_weights()
```python
    def create_weights(
        self,
        layer,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype,
        **extra_weight_attrs,
    ):
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoeWeightScaleSupported

        fp4_block_k = 32

        w13_weight = Parameter(
            torch.empty(
                num_experts,
                2 * intermediate_size_per_partition,
                hidden_size // 2,
                dtype=torch.int8,
            ),
            requires_grad=False,
        )
        w2_weight = Parameter(
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

        w13_weight_scale = Parameter(
            torch.ones(
                num_experts,
                2 * intermediate_size_per_partition,
                hidden_size // fp4_block_k,
                dtype=torch.float32,
            ),
            requires_grad=False,
        )
        w2_weight_scale = Parameter(
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
**EN:** This block defines `Mxfp4FlashinferTrtllmMoEMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `Mxfp4FlashinferTrtllmMoEMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 213-257: Mxfp4FlashinferTrtllmMoEMethod.process_weights_after_loading() (part 1/3)
```python
    def process_weights_after_loading(self, layer: Module) -> None:
        from sglang.srt.layers.quantization.utils import reorder_w1w3_to_w3w1

        self._fp8.process_weights_after_loading(layer)

        if getattr(layer, "_mega_moe_weights_built", False):
            return

        w13_w, w13_s = reorder_w1w3_to_w3w1(
            layer.w13_weight.data, layer.w13_weight_scale_inv.data
        )
        layer.w13_weight = Parameter(w13_w, requires_grad=False)
        layer.w13_weight_scale_inv = Parameter(w13_s, requires_grad=False)

        log_info_on_rank0(
            logger,
            f"Shuffling FP4 expert weights for TRT-LLM MxFP4 kernel "
            f"(layer: {self.prefix})...",
        )

        w13 = layer.w13_weight.data
        w2 = layer.w2_weight.data
        w13_scale = layer.w13_weight_scale_inv.data
        w2_scale = layer.w2_weight_scale_inv.data
        num_experts = w13.shape[0]

        if w13_scale.dtype == torch.float32:
            w13_scale = w13_scale.to(torch.float8_e8m0fnu)
            w2_scale = w2_scale.to(torch.float8_e8m0fnu)

        epilogue_tile_m = 128
        g1_w, g1_s, g2_w, g2_s = [], [], [], []
        if _USE_OFFICIAL_SHUFFLE:
            cache: dict = {}
            for i in range(num_experts):
                w13_u8 = w13[i].view(torch.uint8)
                w13_s_u8 = w13_scale[i].view(torch.uint8)
                w2_u8 = w2[i].view(torch.uint8)
                w2_s_u8 = w2_scale[i].view(torch.uint8)

                perm = _maybe_get_cached_w3_w1_permute_indices(
                    cache,
                    w13_u8,
                    epilogue_tile_m,
                )
```
**EN:** This segment of `Mxfp4FlashinferTrtllmMoEMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `Mxfp4FlashinferTrtllmMoEMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 258-302: Mxfp4FlashinferTrtllmMoEMethod.process_weights_after_loading() (part 2/3)
```python
                g1_w.append(w13_u8[perm.to(w13_u8.device)].contiguous())
                perm_sf = _maybe_get_cached_w3_w1_permute_indices(
                    cache,
                    w13_s_u8,
                    epilogue_tile_m,
                    num_elts_per_sf=16,
                )
                g1_s.append(
                    block_scale_interleave(
                        w13_s_u8[perm_sf.to(w13_s_u8.device)].contiguous()
                    )
                )

                perm = get_w2_permute_indices_with_cache(
                    cache,
                    w2_u8,
                    epilogue_tile_m,
                )
                g2_w.append(w2_u8[perm.to(w2_u8.device)].contiguous())
                perm_sf = get_w2_permute_indices_with_cache(
                    cache,
                    w2_s_u8,
                    epilogue_tile_m,
                    num_elts_per_sf=16,
                )
                g2_s.append(
                    block_scale_interleave(
                        w2_s_u8[perm_sf.to(w2_s_u8.device)].contiguous()
                    )
                )
        else:
            for i in range(num_experts):
                g1_w.append(shuffle_matrix_a(w13[i].view(torch.uint8), epilogue_tile_m))
                g1_s.append(
                    shuffle_matrix_sf_a(w13_scale[i].view(torch.uint8), epilogue_tile_m)
                )
                g2_w.append(shuffle_matrix_a(w2[i].view(torch.uint8), epilogue_tile_m))
                g2_s.append(
                    shuffle_matrix_sf_a(w2_scale[i].view(torch.uint8), epilogue_tile_m)
                )

        layer.w13_weight = Parameter(torch.stack(g1_w), requires_grad=False)
        layer.w13_weight_scale_inv = Parameter(
            torch.stack(g1_s)
            .view(torch.float8_e4m3fn)
```
**EN:** This segment of `Mxfp4FlashinferTrtllmMoEMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `Mxfp4FlashinferTrtllmMoEMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 303-315: Mxfp4FlashinferTrtllmMoEMethod.process_weights_after_loading() (part 3/3)
```python
            .reshape(num_experts, w13.shape[1], -1),
            requires_grad=False,
        )
        layer.w2_weight = Parameter(torch.stack(g2_w), requires_grad=False)
        layer.w2_weight_scale_inv = Parameter(
            torch.stack(g2_s)
            .view(torch.float8_e4m3fn)
            .reshape(num_experts, w2.shape[1], -1),
            requires_grad=False,
        )

        self._register_static_scale_ones(layer)
        torch.cuda.empty_cache()
```
**EN:** This segment of `Mxfp4FlashinferTrtllmMoEMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `Mxfp4FlashinferTrtllmMoEMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 317-328: Mxfp4FlashinferTrtllmMoEMethod._register_static_scale_ones()
```python
    def _register_static_scale_ones(self, layer: Module) -> None:
        device = layer.w13_weight.device
        for name in (
            "output1_scale_scalar",
            "output1_scale_gate_scalar",
            "output2_scale_scalar",
        ):
            layer.register_buffer(
                name,
                torch.ones(layer.num_local_experts, device=device, dtype=torch.float32),
                persistent=False,
            )
```
**EN:** This block defines `Mxfp4FlashinferTrtllmMoEMethod._register_static_scale_ones()`, which registers hooks, kernels, or helper behavior.
**CN:** 该代码块定义了 `Mxfp4FlashinferTrtllmMoEMethod._register_static_scale_ones()`，用于注册钩子、内核或辅助行为。

### Lines 330-374: Mxfp4FlashinferTrtllmMoEMethod.apply() (part 1/3)
```python
    def apply(
        self,
        layer: Module,
        dispatch_output: DispatchOutput,
    ) -> CombineInput:
        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput
        from sglang.srt.layers.moe.topk import TopKOutputChecker

        hidden_states = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output

        w13 = layer.w13_weight
        w2 = layer.w2_weight
        w13_scale = layer.w13_weight_scale_inv
        w2_scale = layer.w2_weight_scale_inv

        intermediate_size = w2.shape[2] * 2 if w2.dtype == torch.uint8 else w2.shape[2]
        hidden_size = w13.shape[2] * 2 if w13.dtype == torch.uint8 else w13.shape[2]

        num_local_experts = layer.num_local_experts
        if w13_scale.dim() == 2:
            w13_scale = w13_scale.reshape(num_local_experts, 2 * intermediate_size, -1)
        if w2_scale.dim() == 2:
            w2_scale = w2_scale.reshape(num_local_experts, hidden_size, -1)

        if TopKOutputChecker.format_is_standard(topk_output):
            topk_ids = topk_output.topk_ids
            topk_weights = topk_output.topk_weights
        elif TopKOutputChecker.format_is_bypassed(topk_output):
            raise NotImplementedError(
                "the old code in this branch is WRONG. e.g. it does not consider HashTopK, and may miss args"
            )
        else:
            raise ValueError(f"Unsupported topk output format: {topk_output.format}")

        packed_topk = PackTopkIds.execute(topk_ids, topk_weights)

        precision = self.flashinfer_mxfp4_moe_precision
        if precision == "bf16":
            assert hidden_states.dtype == torch.bfloat16
            x_quant = hidden_states
            x_scale = None
            origin_dim = x_quant.shape[-1]
            if hidden_size != origin_dim:
                x_quant = torch.nn.functional.pad(
```
**EN:** This segment of `Mxfp4FlashinferTrtllmMoEMethod.apply()` applies quantized computation to runtime inputs. It covers one portion of the full implementation.
**CN:** `Mxfp4FlashinferTrtllmMoEMethod.apply()` 的这一段代码用于将量化计算应用到运行时输入上，展示了完整实现中的一部分。

### Lines 375-419: Mxfp4FlashinferTrtllmMoEMethod.apply() (part 2/3)
```python
                    x_quant,
                    (0, hidden_size - origin_dim),
                    mode="constant",
                    value=0.0,
                )
        elif precision == "default":
            x_quant, x_scale = mxfp8_quantize(
                hidden_states, False, alignment=hidden_size
            )
            x_scale = x_scale.view(torch.float8_e4m3fn).reshape(
                *hidden_states.shape[:-1], -1
            )
        else:
            raise NotImplementedError(f"Unsupported mxfp4 moe precision: {precision}")

        with use_symmetric_memory(
            get_tp_group(), disabled=not is_allocation_symmetric()
        ):
            num_tokens = x_quant.shape[0]
            out_hidden_size = (
                x_quant.shape[-1] * 2
                if x_quant.dtype == torch.uint8
                else x_quant.shape[-1]
            )
            symm_output = torch.empty(
                num_tokens, out_hidden_size, dtype=torch.bfloat16, device=x_quant.device
            )

        output = trtllm_fp4_block_scale_routed_moe(
            topk_ids=packed_topk,
            routing_bias=None,
            hidden_states=x_quant,
            hidden_states_scale=x_scale,
            gemm1_weights=w13,
            gemm1_weights_scale=w13_scale,
            gemm1_bias=None,
            gemm1_alpha=None,
            gemm1_beta=None,
            gemm1_clamp_limit=self._gemm1_clamp_limit_tensor,
            gemm2_weights=w2,
            gemm2_weights_scale=w2_scale,
            gemm2_bias=None,
            output1_scale_scalar=layer.output1_scale_scalar,
            output1_scale_gate_scalar=layer.output1_scale_gate_scalar,
            output2_scale_scalar=layer.output2_scale_scalar,
```
**EN:** This segment of `Mxfp4FlashinferTrtllmMoEMethod.apply()` applies quantized computation to runtime inputs. It covers one portion of the full implementation.
**CN:** `Mxfp4FlashinferTrtllmMoEMethod.apply()` 的这一段代码用于将量化计算应用到运行时输入上，展示了完整实现中的一部分。

### Lines 420-434: Mxfp4FlashinferTrtllmMoEMethod.apply() (part 3/3)
```python
            num_experts=layer.num_experts,
            top_k=packed_topk.shape[1],
            n_group=1,
            topk_group=1,
            intermediate_size=intermediate_size,
            local_expert_offset=layer.moe_ep_rank * layer.num_local_experts,
            local_num_experts=num_local_experts,
            routed_scaling_factor=1.0,
            routing_method_type=int(RoutingMethodType.TopK),
            do_finalize=True,
            tune_max_num_tokens=next_power_of_2(x_quant.shape[0]),
            output=symm_output,
        )[0]

        return StandardCombineInput(hidden_states=output)
```
**EN:** This segment of `Mxfp4FlashinferTrtllmMoEMethod.apply()` applies quantized computation to runtime inputs. It covers one portion of the full implementation.
**CN:** `Mxfp4FlashinferTrtllmMoEMethod.apply()` 的这一段代码用于将量化计算应用到运行时输入上，展示了完整实现中的一部分。

### Lines 437-469: maybe_fuse_routed_scale_and_shared_add()
```python
def maybe_fuse_routed_scale_and_shared_add(
    experts,
    routed: torch.Tensor,
    shared: torch.Tensor | None,
    routed_scaling_factor: float,
) -> torch.Tensor:
    # When MxFP4 fusion is on, the upstream `routed *= scale` is skipped and
    # the scaling is folded into the shared-add via `shared.add_(routed,
    # alpha=scale)`. With no shared output, the missing scale is applied
    # in-place. Otherwise `routed` is already scale-final and we just add
    # `shared` (or pass through if there is none).
    from sglang.srt.layers.quantization.mxfp4_flashinfer_cutlass_moe import (
        Mxfp4FlashinferCutlassMoEMethod,
    )
    from sglang.srt.layers.quantization.mxfp4_marlin_moe import (
        Mxfp4MarlinMoEMethod,
    )

    fused = isinstance(
        experts.quant_method,
        (
            Mxfp4FlashinferTrtllmMoEMethod,
            Mxfp4FlashinferCutlassMoEMethod,
            Mxfp4MarlinMoEMethod,
        ),
    )
    if fused:
        if shared is not None:
            return shared.add_(routed, alpha=routed_scaling_factor)
        return routed.mul_(routed_scaling_factor)
    if shared is not None:
        routed += shared
    return routed
```
**EN:** This block defines `maybe_fuse_routed_scale_and_shared_add()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `maybe_fuse_routed_scale_and_shared_add()`，用于实现量化栈中的可复用模块逻辑。

## Key Concepts / 关键概念
- `_USE_OFFICIAL_SHUFFLE`: A module-level constant or registry. / `_USE_OFFICIAL_SHUFFLE`：模块级常量或注册表。
- `PackTopkIds`: A supporting class that structures file-level quantization behavior. / `PackTopkIds` 是一个支撑类，用于组织该文件中的量化行为。
- `Mxfp4FlashinferTrtllmMoEMethod`: A runtime method class that structures file-level quantization behavior. / `Mxfp4FlashinferTrtllmMoEMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `maybe_fuse_routed_scale_and_shared_add()` : A public function that implements reusable module logic for the quantization stack. / `maybe_fuse_routed_scale_and_shared_add()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `flashinfer`, `flashinfer.fp4_quantization`, `flashinfer.fused_moe`, `flashinfer.fused_moe.core`, `logging`, `torch`, `torch.nn`, `torch.nn.parameter`, `triton`, `triton.language`, `typing`
- **Internal / 内部**: `sglang.srt.distributed`, `sglang.srt.distributed.device_communicators.pynccl_allocator`, `sglang.srt.layers.dp_attention`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.moe.topk`, `sglang.srt.layers.moe.utils`, `sglang.srt.layers.quantization.mxfp4_flashinfer_cutlass_moe`, `sglang.srt.layers.quantization.mxfp4_marlin_moe`, `sglang.srt.layers.quantization.utils`, `sglang.srt.server_args`, `sglang.srt.utils`, `sglang.srt.utils.common`
