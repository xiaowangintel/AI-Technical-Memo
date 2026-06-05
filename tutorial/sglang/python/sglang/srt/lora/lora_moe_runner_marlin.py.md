# lora_moe_runner_marlin.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/lora/lora_moe_runner_marlin.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Marlin MoE runner core with hook support for LoRA injection. / 该文件实现了核心 LoRA 运行时逻辑，例如适配器元数据、内存管理、层封装或整体编排。它属于 SGLang 支持动态适配器服务能力的基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-43: Module header, imports, and shared constants
```python
"""Marlin MoE runner core with hook support for LoRA injection.

Uses Marlin int4/int8 kernels for the base MoE projections.
LoRA deltas are injected via hooks.
"""

from __future__ import annotations

from typing import TYPE_CHECKING, Optional

import torch

from sglang.srt.layers.moe.moe_runner.base import MoeRunnerConfig
from sglang.srt.layers.moe.moe_runner.marlin import MarlinMoeQuantInfo
from sglang.srt.utils import is_cuda

if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import (
        StandardCombineInput,
        StandardDispatchOutput,
    )

_is_cuda = is_cuda()

if _is_cuda:
    from sgl_kernel import silu_and_mul

    from sglang.jit_kernel.moe_wna16_marlin import moe_wna16_marlin_gemm
    from sglang.srt.layers.moe.fused_moe_triton.fused_marlin_moe import (
        get_scalar_type,
    )
    from sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe import (
        moe_align_block_size,
    )
    from sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe_triton_kernels import (
        moe_sum_reduce_triton,
    )
    from sglang.srt.layers.quantization.marlin_utils import marlin_make_workspace


_MARLIN_WORKSPACE: Optional[torch.Tensor] = None
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 44-56: MarlinLoraRunnerCore class declaration
```python
class MarlinLoraRunnerCore:
    """
    MoE runner using Marlin kernels for base projections, with hooks for LoRA.

    Pipeline:
      1. moe_wna16_marlin_gemm (gate_up)
      1.5. hooks.after_gate_up
      2. silu_and_mul
      3. moe_wna16_marlin_gemm (down)
      3.5. hooks.after_down
      4. moe_sum_reduce
    """
```
**EN:** This block declares the `MarlinLoraRunnerCore` class, which exists to coordinate staged runtime execution. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `MarlinLoraRunnerCore` 类，其职责是协调分阶段运行时执行。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 57-58: MarlinLoraRunnerCore initializer
```python
    def __init__(self, config: MoeRunnerConfig):
        self.config = config
```
**EN:** This block initializes the `MarlinLoraRunnerCore` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `MarlinLoraRunnerCore` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 60-119: MarlinLoraRunnerCore.run_from_dispatch method (part 1/3)
```python
    def run_from_dispatch(
        self,
        dispatch_output: StandardDispatchOutput,
        quant_info: MarlinMoeQuantInfo,
        runner_config: MoeRunnerConfig,
        hooks=None,
    ) -> StandardCombineInput:
        global _MARLIN_WORKSPACE
        from sglang.srt.layers.moe.token_dispatcher.standard import StandardCombineInput

        assert hooks is not None, "hooks must be provided for MarlinLoraRunnerCore"

        hidden_states = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output
        topk_weights = topk_output.topk_weights
        topk_ids = topk_output.topk_ids

        assert runner_config.activation == "silu", "Only SiLU activation is supported."
        assert (
            torch.cuda.get_device_capability(hidden_states.device)[0] >= 9
        ), "MarlinLoraRunnerCore requires CUDA compute capability >= 9"
        inplace = runner_config.inplace
        routed_scaling_factor = runner_config.routed_scaling_factor

        M, K = hidden_states.shape
        E = quant_info.w13_qweight.shape[0]
        N = quant_info.w2_qweight.shape[1] * 16
        topk = topk_ids.shape[1]
        num_bits = quant_info.weight_bits

        for block_size_m in [8, 16, 32, 48, 64]:
            if M * topk / E / block_size_m < 0.9:
                break

        sorted_token_ids, expert_ids, num_tokens_post_padded = moe_align_block_size(
            topk_ids, block_size_m, E
        )

        if (
            _MARLIN_WORKSPACE is None
            or _MARLIN_WORKSPACE.device != hidden_states.device
        ):
            _MARLIN_WORKSPACE = marlin_make_workspace(
                hidden_states.device, max_blocks_per_sm=4
            )
        workspace = _MARLIN_WORKSPACE

        scalar_type1 = get_scalar_type(num_bits, quant_info.w13_qzeros is not None)
        scalar_type2 = get_scalar_type(num_bits, quant_info.w2_qzeros is not None)

        # Stage 1: Gate/Up (Marlin)
        intermediate_cache1 = torch.empty(
            (M * topk, 2 * N), device=hidden_states.device, dtype=hidden_states.dtype
        )
        intermediate_cache1 = moe_wna16_marlin_gemm(
            hidden_states,
            intermediate_cache1,
            quant_info.w13_qweight,
            None,
            quant_info.w13_scales,
```
**EN:** This block uses `MarlinLoraRunnerCore.run_from_dispatch` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `MarlinLoraRunnerCore.run_from_dispatch` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 120-179: MarlinLoraRunnerCore.run_from_dispatch method (part 2/3)
```python
            None,
            quant_info.w13_qzeros,
            quant_info.w13_g_idx,
            quant_info.w13_g_idx_sort_indices,
            workspace,
            sorted_token_ids,
            expert_ids,
            num_tokens_post_padded,
            topk_weights,
            moe_block_size=block_size_m,
            top_k=topk,
            mul_topk_weights=False,
            is_ep=quant_info.expert_map is not None,
            b_q_type=scalar_type1,
            size_m=M,
            size_n=2 * N,
            size_k=K,
            is_k_full=quant_info.is_k_full,
            use_atomic_add=True,
            use_fp32_reduce=True,
            is_zp_float=False,
        )

        # Hook: after gate_up
        if hooks.after_gate_up:
            intermediate_cache1_3d = intermediate_cache1.view(M, topk, 2 * N)
            hooks.after_gate_up(
                hidden_states, intermediate_cache1_3d, topk_weights, topk_ids
            )

        # Stage 2: Activation
        intermediate_cache2 = torch.empty(
            (M * topk, N), device=hidden_states.device, dtype=hidden_states.dtype
        )
        silu_and_mul(intermediate_cache1.view(-1, 2 * N), intermediate_cache2)

        # Stage 3: Down (Marlin)
        intermediate_cache3 = torch.empty(
            (M * topk, K), device=hidden_states.device, dtype=hidden_states.dtype
        )
        if quant_info.expert_map is not None:
            intermediate_cache3.zero_()

        intermediate_cache3 = moe_wna16_marlin_gemm(
            intermediate_cache2,
            intermediate_cache3,
            quant_info.w2_qweight,
            None,
            quant_info.w2_scales,
            None,
            quant_info.w2_qzeros,
            quant_info.w2_g_idx,
            quant_info.w2_g_idx_sort_indices,
            workspace,
            sorted_token_ids,
            expert_ids,
            num_tokens_post_padded,
            topk_weights,
            moe_block_size=block_size_m,
            top_k=1,
```
**EN:** This block uses `MarlinLoraRunnerCore.run_from_dispatch` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk continues the routine's middle-stage logic and data movement.
**CN:** 该代码块通过 `MarlinLoraRunnerCore.run_from_dispatch` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分继续展示例程中段的逻辑与数据流动。

### Lines 180-206: MarlinLoraRunnerCore.run_from_dispatch method (part 3/3)
```python
            mul_topk_weights=True,
            is_ep=quant_info.expert_map is not None,
            b_q_type=scalar_type2,
            size_m=M * topk,
            size_n=K,
            size_k=N,
            is_k_full=quant_info.is_k_full,
            use_atomic_add=True,
            use_fp32_reduce=True,
            is_zp_float=False,
        )
        intermediate_cache3 = intermediate_cache3.view(M, topk, K)

        # Hook: after down
        if hooks.after_down:
            hooks.after_down(
                intermediate_cache2, intermediate_cache3, topk_weights, topk_ids
            )

        # Stage 4: Reduction
        output = hidden_states if inplace else torch.empty_like(hidden_states)
        if routed_scaling_factor is None:
            routed_scaling_factor = 1.0
        # NOTE: fusion opportunity here
        moe_sum_reduce_triton(intermediate_cache3, output, routed_scaling_factor)

        return StandardCombineInput(hidden_states=output)
```
**EN:** This block uses `MarlinLoraRunnerCore.run_from_dispatch` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `MarlinLoraRunnerCore.run_from_dispatch` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

## Key Concepts / 关键概念
- LoRA adapter execution / LoRA 适配器执行
- Mixture-of-Experts support / 混合专家支持
- Tensor computation with PyTorch / 基于 PyTorch 的张量计算

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.jit_kernel.moe_wna16_marlin`
- `sglang.srt.layers.moe.fused_moe_triton.fused_marlin_moe`
- `sglang.srt.layers.moe.moe_runner.base`
- `sglang.srt.layers.moe.moe_runner.marlin`
- `sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe`
- `sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe_triton_kernels`
- `sglang.srt.layers.moe.token_dispatcher`
- `sglang.srt.layers.moe.token_dispatcher.standard`
- `sglang.srt.layers.quantization.marlin_utils`
- `sglang.srt.utils`
### External / 外部
- `__future__`
- `sgl_kernel`
- `torch`
- `typing` (stdlib)
