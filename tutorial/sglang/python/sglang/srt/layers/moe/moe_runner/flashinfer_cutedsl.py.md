# flashinfer_cutedsl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/moe_runner/flashinfer_cutedsl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution with low-precision quantization data paths for the SGLang SRT runtime. It exposes symbols such as `interleave_w13_halves`, `cutedsl_quant_scale_to_scalar`, `resolve_cutedsl_standard_scales`, and `ensure_cutedsl_wrapper` and connects them to backend-specific paths such as `CUDA`, `FlashInfer`, and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行，并结合低精度量化数据路径。它提供了 `interleave_w13_halves`、`cutedsl_quant_scale_to_scalar`、`resolve_cutedsl_standard_scales` 以及 `ensure_cutedsl_wrapper` 等符号，并把这些符号连接到 `CUDA`、`FlashInfer` 以及 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-36: Imports, conditional backend setup, and runtime guards
```python
from __future__ import annotations

import logging
from dataclasses import dataclass
from typing import TYPE_CHECKING, Any, Optional

import torch

from sglang.srt.layers.moe.moe_runner.base import (
    MoeQuantInfo,
    MoeRunnerConfig,
    register_fused_func,
)
from sglang.srt.utils.common import log_info_on_rank0, print_warning_once

if TYPE_CHECKING:
    from sglang.srt.batch_overlap.single_batch_overlap import DownGemmOverlapArgs
    from sglang.srt.layers.moe.token_dispatcher import (
        DeepEPLLCombineInput,
        DeepEPLLDispatchOutput,
        StandardCombineInput,
        StandardDispatchOutput,
    )

logger = logging.getLogger(__name__)

_FP4_SF_VEC_SIZE = 16
_cutedsl_logged_scalarize: set = set()


# ---------------------------------------------------------------------------
# Weight / scale preparation utilities (called from modelopt_quant.py during
# process_weights_after_loading and lazy wrapper init)
# ---------------------------------------------------------------------------
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `logging`, `dataclasses.dataclass`, `typing.TYPE_CHECKING`, `typing.Any`, and `typing.Optional`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `logger`, `_FP4_SF_VEC_SIZE`, and `_cutedsl_logged_scalarize` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`logging`、`dataclasses.dataclass`、`typing.TYPE_CHECKING`、`typing.Any` 以及 `typing.Optional`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `logger`、`_FP4_SF_VEC_SIZE` 以及 `_cutedsl_logged_scalarize` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 37-64: Function `interleave_w13_halves` and its core logic
```python
def interleave_w13_halves(
    tensor: torch.Tensor, group_size: int = 64, dim: int = 1
) -> torch.Tensor:
    """Interleave the two logical W13 halves for CuteDSL's SwiGLU GEMM1 layout.

    The caller is responsible for loading W13 in the expected two-half order.
    This helper only rewrites the first and second halves into alternating
    `group_size` chunks along `dim`.
    """
    if tensor.shape[dim] % 2 != 0:
        raise ValueError(
            "Expected even size on interleave dimension for W13 half split."
        )
    split = tensor.shape[dim] // 2
    if split % group_size != 0:
        raise ValueError(
            f"Expected split dim divisible by group_size={group_size}, got {split}."
        )
    first_half = tensor.narrow(dim, 0, split)
    second_half = tensor.narrow(dim, split, split)
    first_half_groups = first_half.split(group_size, dim=dim)
    second_half_groups = second_half.split(group_size, dim=dim)
    interleaved = [
        item for pair in zip(first_half_groups, second_half_groups) for item in pair
    ]
    return torch.cat(interleaved, dim=dim)
```
**EN:** This block defines `interleave_w13_halves` and contains the main logic for this step. It mainly invokes `tensor.narrow`, `first_half.split`, `second_half.split`, `torch.cat`, and `ValueError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `split`, `first_half`, `second_half`, `first_half_groups`, and `second_half_groups` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `interleave_w13_halves`，并承载这一阶段的核心逻辑。 它主要调用 `tensor.narrow`、`first_half.split`、`second_half.split`、`torch.cat` 以及 `ValueError`，说明该流程会编排底层辅助函数或计算内核。 像 `split`、`first_half`、`second_half`、`first_half_groups` 以及 `second_half_groups` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 65-98: Function `cutedsl_quant_scale_to_scalar` and its core logic
```python
def cutedsl_quant_scale_to_scalar(
    quant_scale: torch.Tensor,
    *,
    name: str,
) -> torch.Tensor:
    """Reduce per-expert quant-domain scale vector to a single scalar.

    The quant domain is the reciprocal of the raw checkpoint scale:
        quant_scale = 1 / raw_scale

    Returns min(quant_scale) = 1/max(raw_scale), which is the TRTLLM CuteDSL
    convention for global scalar activation scales (see TRTLLM quantization.py
    lines 2137-2141: fc2_input_scale = tmp_fc2_input_scale.max().reciprocal()).

    If quant_scale is already scalar (numel==1), returns it unchanged.
    """
    quant_scale = quant_scale.to(torch.float32)
    if quant_scale.numel() == 0:
        print_warning_once(
            f"CuteDSL got empty {name}; using 1.0 fallback.",
        )
        return torch.ones(1, device=quant_scale.device, dtype=torch.float32)
    if quant_scale.numel() == 1:
        return quant_scale.reshape(1)
    if name not in _cutedsl_logged_scalarize:
        log_info_on_rank0(
            logger,
            f"CuteDSL: reducing per-expert {name} to scalar via "
            "min(quant_scale) = 1/max(raw_scale), matching TRTLLM convention.",
        )
        _cutedsl_logged_scalarize.add(name)
    return quant_scale.min().reshape(1)
```
**EN:** This block defines `cutedsl_quant_scale_to_scalar` and contains the main logic for this step. It mainly invokes `quant_scale.to`, `quant_scale.min.reshape`, `quant_scale.numel`, `print_warning_once`, and `torch.ones`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `quant_scale` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `cutedsl_quant_scale_to_scalar`，并承载这一阶段的核心逻辑。 它主要调用 `quant_scale.to`、`quant_scale.min.reshape`、`quant_scale.numel`、`print_warning_once` 以及 `torch.ones`，说明该流程会编排底层辅助函数或计算内核。 像 `quant_scale` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 99-219: Function `resolve_cutedsl_standard_scales` and its core logic
```python
def resolve_cutedsl_standard_scales(
    layer: torch.nn.Module,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]:
    """Resolve standard-path CuteDSL scales (baseline: scalar fc2/w13 input scales).

    Returns (w1_alpha, fc2_input_scale, w2_alpha, used_input_scale).
    used_input_scale is the scalarized w13 input scale for FP4 quantize and GEMM1.
    """

    def _to_fp32_tensor(x: torch.Tensor | float, ref: torch.Tensor) -> torch.Tensor:
        if not isinstance(x, torch.Tensor):
            x = torch.tensor(x, device=ref.device)
        return x.to(device=ref.device, dtype=torch.float32)

    def _align_scale_to_alpha(
        scale: torch.Tensor, alpha: torch.Tensor, scale_name: str
    ) -> torch.Tensor:
        scale = scale.to(device=alpha.device, dtype=torch.float32)
        alpha = alpha.to(torch.float32)
        if scale.ndim == 0:
            return scale
        # Gated weight scales may be (num_experts, 2) with separate gate/up
        # columns. Collapse to 1D by taking the first column (gate == up for
        # well-formed checkpoints; mismatch is warned in process_weights_after_loading).
        if scale.ndim == 2 and scale.shape[1] <= 2:
            scale = scale[:, 0]
        if scale.numel() == alpha.numel():
            return scale
        if scale.numel() == 1:
            return scale.reshape(())

        # Some EP setups may carry global-per-expert scale vectors while alphas are
        # local-per-expert vectors. Slice to this rank's local expert range.
        num_local_experts = getattr(layer, "num_local_experts", None)
        num_experts = getattr(layer, "num_experts", None)
        moe_ep_rank = getattr(layer, "moe_ep_rank", 0)
        if (
            num_local_experts is not None
            and num_experts is not None
            and scale.numel() == num_experts
            and alpha.numel() == num_local_experts
        ):
            start = moe_ep_rank * num_local_experts
            end = start + num_local_experts
            return scale[start:end]

        raise ValueError(
            f"Unable to align {scale_name} shape={tuple(scale.shape)} "
            f"to alpha shape={tuple(alpha.shape)} for CuteDSL standard scale resolution."
        )

    def _resolve_w1_alpha_from_scalar_input_scale(
        used_input_scale: torch.Tensor,
    ) -> torch.Tensor:
        """Resolve GEMM1 alpha consistent with scalarized activation quant scale.

        CuteDSL pre-quantizes x with a single scalar (used_input_scale), but
        g1_alphas was derived with per-expert activation scales:
            g1_alphas[e] = (1/w13_isq[e]) * w13_ws2[e]
        Correct alpha for scalar quantization:
            w1_alpha[e] = w13_ws2[e] / used_input_scale
                        = g1_alphas[e] * w13_isq[e] / used_input_scale
        When w13_isq is already scalar, this is a no-op (ratio = 1).
        """
        eps = 1e-12
        scalar = torch.clamp(used_input_scale.to(torch.float32).reshape(()), min=eps)

        if hasattr(layer, "w13_weight_scale_2"):
            w13_weight_scale_2 = _align_scale_to_alpha(
                layer.w13_weight_scale_2, layer.g1_alphas, "w13_weight_scale_2"
            )
            return w13_weight_scale_2.to(torch.float32) / scalar

        w13_isq = _align_scale_to_alpha(
            layer.w13_input_scale_quant, layer.g1_alphas, "w13_input_scale_quant"
        )
        w13_isq = torch.clamp(_to_fp32_tensor(w13_isq, layer.g1_alphas), min=eps)
        return (layer.g1_alphas.to(torch.float32) * w13_isq / scalar).to(torch.float32)

    def _resolve_w2_alpha_from_scalar_fc2_input_scale(
        fc2_input_scale: torch.Tensor,
    ) -> torch.Tensor:
        """Resolve GEMM2 alpha consistent with scalarized FC2 input scale.

        CuteDSL standard path uses a scalar global scale for GEMM1 FP4 output
        quantization (`fc2_input_scale`). GEMM2 alpha must use the same scalar
        convention: alpha2 = w2_weight_scale_2 / fc2_input_scale.
        """
        eps = 1e-12
        fc2_input_scale = fc2_input_scale.to(torch.float32)
        fc2_scalar = torch.clamp(fc2_input_scale.reshape(-1)[:1], min=eps).reshape(())

        if hasattr(layer, "w2_weight_scale_2"):
            w2_weight_scale_2 = _align_scale_to_alpha(
                layer.w2_weight_scale_2, layer.g2_alphas, "w2_weight_scale_2"
            )
            w2_weight_scale_2 = w2_weight_scale_2.to(torch.float32)
            return w2_weight_scale_2 / fc2_scalar

        w2_q_for_w2 = _align_scale_to_alpha(
            layer.w2_input_scale_quant, layer.g2_alphas, "w2_input_scale_quant"
        )
        w2_q_for_w2 = torch.clamp(
            _to_fp32_tensor(w2_q_for_w2, layer.g2_alphas), min=eps
        )
        w2_weight_scale_2 = layer.g2_alphas.to(torch.float32) * w2_q_for_w2
        return w2_weight_scale_2 / fc2_scalar

    fc2_input_scale = cutedsl_quant_scale_to_scalar(
        layer.w2_input_scale_quant,
        name="w2_input_scale_quant",
    )
    w2_alpha = _resolve_w2_alpha_from_scalar_fc2_input_scale(fc2_input_scale)
    used_input_scale = cutedsl_quant_scale_to_scalar(
        layer.w13_input_scale_quant,
        name="w13_input_scale_quant",
    )
    w1_alpha = _resolve_w1_alpha_from_scalar_input_scale(used_input_scale)
    return w1_alpha, fc2_input_scale, w2_alpha, used_input_scale
```
**EN:** This block defines `resolve_cutedsl_standard_scales` and contains the main logic for this step. It mainly invokes `cutedsl_quant_scale_to_scalar`, `_resolve_w2_alpha_from_scalar_fc2_input_scale`, `_resolve_w1_alpha_from_scalar_input_scale`, `x.to`, and `scale.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `fc2_input_scale`, `w2_alpha`, `used_input_scale`, `w1_alpha`, and `scale` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `resolve_cutedsl_standard_scales`，并承载这一阶段的核心逻辑。 它主要调用 `cutedsl_quant_scale_to_scalar`、`_resolve_w2_alpha_from_scalar_fc2_input_scale`、`_resolve_w1_alpha_from_scalar_input_scale`、`x.to` 以及 `scale.to`，说明该流程会编排底层辅助函数或计算内核。 像 `fc2_input_scale`、`w2_alpha`、`used_input_scale`、`w1_alpha` 以及 `scale` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 220-286: Function `ensure_cutedsl_wrapper` and its core logic
```python
def ensure_cutedsl_wrapper(layer: torch.nn.Module) -> None:
    """Lazily create CuteDslMoEWrapper and resolve scales on first forward.

    The wrapper is created lazily (not in __init__ / create_weights) because
    it depends on final weight shapes and EP configuration.  The wrapper's
    CUDA-graph buffers are allocated inside CuteDslMoEWrapper.__init__, which
    typically runs during the autotune dummy forward under inference_mode().
    We wrap the creation in inference_mode(False) so that those pre-allocated
    buffers are normal tensors -- inference tensors cannot be inplace-updated
    during later CUDA graph capture, which runs outside inference_mode.
    """
    if getattr(layer, "_cutedsl_wrapper", None) is not None:
        return

    try:
        from flashinfer import CuteDslMoEWrapper
    except ImportError as e:
        raise ImportError(
            "flashinfer_cutedsl backend requires FlashInfer with CuteDSL support. "
            "Install with: pip install flashinfer"
        ) from e

    from sglang.srt.server_args import get_global_server_args

    assert layer.intermediate_size_per_partition > 0, (
        f"CuteDSL MoE: intermediate_size_per_partition must be > 0, "
        f"got {layer.intermediate_size_per_partition}. Check EP/TP configuration."
    )

    server_args = get_global_server_args()
    use_cuda_graph = server_args is not None and not server_args.disable_cuda_graph
    max_num_tokens = max(
        getattr(server_args, "cuda_graph_max_bs", None) or 512,
        getattr(server_args, "chunked_prefill_size", None) or 8192,
    )
    top_k = layer.top_k if layer.top_k is not None else layer.moe_runner_config.top_k
    # inference_mode(False) ensures the wrapper's pre-allocated CUDA-graph
    # buffers are normal tensors.  This call typically happens inside
    # _dummy_run which runs under inference_mode(); inference tensors cannot
    # be inplace-updated during later CUDA graph capture (which runs outside
    # inference_mode), so we must opt out here.
    with torch.inference_mode(False):
        layer._cutedsl_wrapper = CuteDslMoEWrapper(
            num_experts=layer.num_experts,
            top_k=top_k,
            hidden_size=layer.hidden_size,
            intermediate_size=layer.intermediate_size_per_partition,
            use_cuda_graph=use_cuda_graph,
            max_num_tokens=max_num_tokens,
            num_local_experts=layer.num_local_experts,
            local_expert_offset=layer.moe_ep_rank * layer.num_local_experts,
            output_dtype=layer.moe_runner_config.params_dtype,
            device=str(layer.w13_weight.device),
        )

    w1_alpha, fc2_input_scale, w2_alpha, used_input_scale = (
        resolve_cutedsl_standard_scales(layer)
    )
    layer._cutedsl_scales = (w1_alpha, fc2_input_scale, w2_alpha)
    layer._cutedsl_input_scale = used_input_scale


# ---------------------------------------------------------------------------
# Dataclass + fused function for moe_runner dispatch
# ---------------------------------------------------------------------------
```
**EN:** This block defines `ensure_cutedsl_wrapper` and contains the main logic for this step. It mainly invokes `get_global_server_args`, `max`, `resolve_cutedsl_standard_scales`, `getattr`, and `torch.inference_mode`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `server_args`, `use_cuda_graph`, `max_num_tokens`, `top_k`, and `w1_alpha` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `ensure_cutedsl_wrapper`，并承载这一阶段的核心逻辑。 它主要调用 `get_global_server_args`、`max`、`resolve_cutedsl_standard_scales`、`getattr` 以及 `torch.inference_mode`，说明该流程会编排底层辅助函数或计算内核。 像 `server_args`、`use_cuda_graph`、`max_num_tokens`、`top_k` 以及 `w1_alpha` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 287-333: Class `CuteDslFp4MoeQuantInfo` declaration and shared state
```python
@dataclass
class CuteDslFp4MoeQuantInfo(MoeQuantInfo):
    """Quantization payload for FlashInfer CuteDSL FP4 MoE kernels.

    Shared by the two CuteDSL runner entries:

    * "v2" standard path (a2a=``none``/``flashinfer``): consumed by the
      ``@register_fused_func("none", "flashinfer_cutedsl")`` entry, which
      drives ``CuteDslMoEWrapper.run``. Weights are ``[Up, Gate]``
      interleaved with MMA-layout blockscales. ``wrapper`` is set;
      ``w*_scale`` are scalarized.

    * "v1" DeepEP low-latency path (a2a=``deepep``): consumed by the
      ``@register_fused_func("deepep", "flashinfer_cutedsl")`` entry,
      which drives ``flashinfer_cutedsl_moe_masked``. Weights are
      ``[Gate, Up]`` non-interleaved with swizzled blockscales.
      ``wrapper`` is ``None``; ``w*_scale`` are per-expert.
    """

    # FP4 packed weights (uint8)
    w13_weight: torch.Tensor
    w2_weight: torch.Tensor

    # Block-scale factors (MMA layout for v2, swizzled for v1)
    w13_weight_sf: torch.Tensor
    w2_weight_sf: torch.Tensor

    # Per-expert GEMM dequant alphas (scalarized for v2, per-expert for v1)
    w1_alpha: torch.Tensor
    w2_alpha: torch.Tensor

    # Activation quant scales (1 / raw_input_scale).
    #   - a1_scale: quantizes hidden_states before GEMM1
    #   - a2_scale: quantizes GEMM1 output before GEMM2 (a.k.a. fc2 input)
    a1_scale: torch.Tensor
    a2_scale: torch.Tensor

    # v2 only: lazily-created CuteDslMoEWrapper (``None`` on the v1 path).
    wrapper: Optional[Any] = None

    # v1 only: ``True`` when DeepEP pre-quantizes activations to NVFP4.
    use_nvfp4_dispatch: bool = False

    # v1 only: SBO down-GEMM overlap args.
    down_gemm_overlap_args: Optional["DownGemmOverlapArgs"] = None
```
**EN:** This block introduces class `CuteDslFp4MoeQuantInfo` and the state shared by its methods. It inherits from `MoeQuantInfo`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Quantization payload for FlashInfer CuteDSL FP4 MoE kernels.
**CN:** 该代码块引入类 `CuteDslFp4MoeQuantInfo`，并定义其方法共享的状态。 它继承自 `MoeQuantInfo`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 334-379: Function `fused_experts_none_to_flashinfer_cutedsl_fp4` and its core logic
```python
@register_fused_func("none", "flashinfer_cutedsl")
def fused_experts_none_to_flashinfer_cutedsl_fp4(
    dispatch_output: StandardDispatchOutput,
    quant_info: CuteDslFp4MoeQuantInfo,
    runner_config: MoeRunnerConfig,
) -> StandardCombineInput:
    from sglang.srt.layers.moe.token_dispatcher.standard import StandardCombineInput
    from sglang.srt.layers.moe.topk import TopKOutputChecker
    from sglang.srt.layers.quantization.fp4_utils import fp4_quantize

    assert runner_config.activation == "silu", "Only silu is supported for CuteDSL MoE."
    assert quant_info.wrapper is not None, "CuteDSL v2 path requires CuteDslMoEWrapper."

    hidden_states = dispatch_output.hidden_states
    topk_output = dispatch_output.topk_output
    assert TopKOutputChecker.format_is_standard(topk_output)

    topk_ids = topk_output.topk_ids
    topk_weights = topk_output.topk_weights
    if topk_ids.dtype != torch.int32:
        topk_ids = topk_ids.to(torch.int32)

    x_fp4, x_sf = fp4_quantize(
        hidden_states,
        quant_info.a1_scale,
        sf_vec_size=_FP4_SF_VEC_SIZE,
        is_sf_swizzled_layout=False,
    )

    output = quant_info.wrapper.run(
        x=x_fp4,
        x_sf=x_sf,
        token_selected_experts=topk_ids,
        token_final_scales=topk_weights,
        w1_weight=quant_info.w13_weight,
        w1_weight_sf=quant_info.w13_weight_sf,
        w1_alpha=quant_info.w1_alpha,
        fc2_input_scale=quant_info.a2_scale,
        w2_weight=quant_info.w2_weight,
        w2_weight_sf=quant_info.w2_weight_sf,
        w2_alpha=quant_info.w2_alpha,
    )

    return StandardCombineInput(hidden_states=output)
```
**EN:** This block defines `fused_experts_none_to_flashinfer_cutedsl_fp4` and contains the main logic for this step. Decorators like `register_fused_func` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_fused_func`, `TopKOutputChecker.format_is_standard`, `fp4_quantize`, `quant_info.wrapper.run`, and `StandardCombineInput`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `topk_output`, `topk_ids`, `topk_weights`, and `x_fp4` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fused_experts_none_to_flashinfer_cutedsl_fp4`，并承载这一阶段的核心逻辑。 像 `register_fused_func` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_fused_func`、`TopKOutputChecker.format_is_standard`、`fp4_quantize`、`quant_info.wrapper.run` 以及 `StandardCombineInput`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`topk_output`、`topk_ids`、`topk_weights` 以及 `x_fp4` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 380-442: Function `fused_experts_deepep_to_flashinfer_cutedsl_fp4` and its core logic
```python
@register_fused_func("deepep", "flashinfer_cutedsl")
def fused_experts_deepep_to_flashinfer_cutedsl_fp4(
    dispatch_output: DeepEPLLDispatchOutput,
    quant_info: CuteDslFp4MoeQuantInfo,
    runner_config: MoeRunnerConfig,
) -> DeepEPLLCombineInput:
    from sglang.srt.layers.moe.flashinfer_cutedsl_moe import (
        flashinfer_cutedsl_moe_masked,
    )
    from sglang.srt.layers.moe.token_dispatcher.deepep import DeepEPLLCombineInput

    assert runner_config.activation == "silu", "Only silu is supported for CuteDSL MoE."
    assert (
        not runner_config.apply_router_weight_on_input
    ), "apply_router_weight_on_input is not supported for Flashinfer"

    hidden_states, hidden_states_scale, _, _, masked_m, _ = dispatch_output

    # flashinfer_cutedsl_moe_masked reinterprets scales as float8_e4m3fn.
    # Same-dtype .view is a no-op; only wider dtypes (e.g. int32-packed
    # UE8M0) need stride(-1)==1.
    if (
        quant_info.use_nvfp4_dispatch
        and hidden_states_scale is not None
        and hidden_states_scale.element_size() != 1
        and hidden_states_scale.stride(-1) != 1
    ):
        raise AssertionError(
            f"NVFP4 dispatch scale has stride(-1)={hidden_states_scale.stride(-1)}, "
            f"dtype={hidden_states_scale.dtype}; .view(float8_e4m3fn) requires stride(-1)==1. "
            "Try SGLANG_MOE_NVFP4_DISPATCH=0 or check DeepEP version."
        )

    overlap = quant_info.down_gemm_overlap_args
    output = flashinfer_cutedsl_moe_masked(
        hidden_states=(hidden_states, hidden_states_scale),
        input_global_scale=(
            None if quant_info.use_nvfp4_dispatch else quant_info.a1_scale
        ),
        w1=quant_info.w13_weight,
        w1_blockscale=quant_info.w13_weight_sf,
        w1_alpha=quant_info.w1_alpha,
        w2=quant_info.w2_weight,
        a2_global_scale=quant_info.a2_scale,
        w2_blockscale=quant_info.w2_weight_sf,
        w2_alpha=quant_info.w2_alpha,
        masked_m=masked_m,
        **(
            dict(
                down_sm_count=overlap.num_sms,
                down_signals=overlap.signal,
                down_start_event=overlap.start_event,
            )
            if overlap is not None
            else {}
        ),
    )

    return DeepEPLLCombineInput(
        hidden_states=output,
        topk_ids=dispatch_output.topk_ids,
        topk_weights=dispatch_output.topk_weights,
    )
```
**EN:** This block defines `fused_experts_deepep_to_flashinfer_cutedsl_fp4` and contains the main logic for this step. Decorators like `register_fused_func` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_fused_func`, `flashinfer_cutedsl_moe_masked`, `DeepEPLLCombineInput`, `AssertionError`, and `hidden_states_scale.element_size`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `hidden_states_scale`, `_`, `masked_m`, and `overlap` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fused_experts_deepep_to_flashinfer_cutedsl_fp4`，并承载这一阶段的核心逻辑。 像 `register_fused_func` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_fused_func`、`flashinfer_cutedsl_moe_masked`、`DeepEPLLCombineInput`、`AssertionError` 以及 `hidden_states_scale.element_size`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`hidden_states_scale`、`_`、`masked_m` 以及 `overlap` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `interleave_w13_halves`, `cutedsl_quant_scale_to_scalar`, `resolve_cutedsl_standard_scales`, `ensure_cutedsl_wrapper`, and `CuteDslFp4MoeQuantInfo`. / **主要符号**：核心入口包括 `interleave_w13_halves`、`cutedsl_quant_scale_to_scalar`、`resolve_cutedsl_standard_scales`、`ensure_cutedsl_wrapper` 以及 `CuteDslFp4MoeQuantInfo`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `logging`, `dataclasses.dataclass`, `typing.TYPE_CHECKING`, `typing.Any`, and `typing.Optional` / **标准库**：`__future__.annotations`、`logging`、`dataclasses.dataclass`、`typing.TYPE_CHECKING`、`typing.Any` 以及 `typing.Optional`
- **Third-party**: `torch` and `flashinfer.CuteDslMoEWrapper` / **第三方依赖**：`torch` 和 `flashinfer.CuteDslMoEWrapper`
- **Internal SGLang modules**: `sglang.srt.layers.moe.moe_runner.base.MoeQuantInfo`, `sglang.srt.layers.moe.moe_runner.base.MoeRunnerConfig`, `sglang.srt.layers.moe.moe_runner.base.register_fused_func`, `sglang.srt.utils.common.log_info_on_rank0`, `sglang.srt.utils.common.print_warning_once`, `sglang.srt.batch_overlap.single_batch_overlap.DownGemmOverlapArgs`, `sglang.srt.layers.moe.token_dispatcher.DeepEPLLCombineInput`, `sglang.srt.layers.moe.token_dispatcher.DeepEPLLDispatchOutput`, `sglang.srt.layers.moe.token_dispatcher.StandardCombineInput`, `sglang.srt.layers.moe.token_dispatcher.StandardDispatchOutput`, `sglang.srt.server_args.get_global_server_args`, and `sglang.srt.layers.moe.token_dispatcher.standard.StandardCombineInput` / **SGLang 内部模块**：`sglang.srt.layers.moe.moe_runner.base.MoeQuantInfo`、`sglang.srt.layers.moe.moe_runner.base.MoeRunnerConfig`、`sglang.srt.layers.moe.moe_runner.base.register_fused_func`、`sglang.srt.utils.common.log_info_on_rank0`、`sglang.srt.utils.common.print_warning_once`、`sglang.srt.batch_overlap.single_batch_overlap.DownGemmOverlapArgs`、`sglang.srt.layers.moe.token_dispatcher.DeepEPLLCombineInput`、`sglang.srt.layers.moe.token_dispatcher.DeepEPLLDispatchOutput`、`sglang.srt.layers.moe.token_dispatcher.StandardCombineInput`、`sglang.srt.layers.moe.token_dispatcher.StandardDispatchOutput`、`sglang.srt.server_args.get_global_server_args` 以及 `sglang.srt.layers.moe.token_dispatcher.standard.StandardCombineInput`
