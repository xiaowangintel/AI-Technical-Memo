# flashinfer_trtllm_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/flashinfer_trtllm_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution with low-precision quantization data paths for the SGLang SRT runtime. It exposes symbols such as `_fake_fp8_block_scale_moe`, `trtllm_fp8_block_scale_moe_wrapper`, `_fake_fp8_block_scale_routed_moe`, and `trtllm_fp8_block_scale_routed_moe_wrapper` and connects them to backend-specific paths such as `FlashInfer` and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行，并结合低精度量化数据路径。它提供了 `_fake_fp8_block_scale_moe`、`trtllm_fp8_block_scale_moe_wrapper`、`_fake_fp8_block_scale_routed_moe` 以及 `trtllm_fp8_block_scale_routed_moe_wrapper` 等符号，并把这些符号连接到 `FlashInfer` 和 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Module imports and dependency wiring
```python
from typing import Optional

import torch

from sglang.srt.utils.custom_op import register_custom_op
```
**EN:** This section prepares the module namespace. It imports `typing.Optional`, `torch`, and `sglang.srt.utils.custom_op.register_custom_op`, so later blocks can reuse runtime, tensor, or backend helpers.
**CN:** 该部分负责准备模块命名空间。 它导入了 `typing.Optional`、`torch` 以及 `sglang.srt.utils.custom_op.register_custom_op`，让后续代码可以复用运行时、张量或后端辅助逻辑。

### Lines 8-37: Internal helper `_fake_fp8_block_scale_moe`
```python
def _fake_fp8_block_scale_moe(
    routing_logits: torch.Tensor,
    routing_bias: Optional[torch.Tensor],
    hidden_states: torch.Tensor,
    hidden_states_scale: torch.Tensor,
    gemm1_weights: torch.Tensor,
    gemm1_weights_scale: torch.Tensor,
    gemm2_weights: torch.Tensor,
    gemm2_weights_scale: torch.Tensor,
    num_experts: int,
    top_k: int,
    n_group: Optional[int],
    topk_group: Optional[int],
    intermediate_size: int,
    local_expert_offset: int,
    local_num_experts: int,
    routed_scaling_factor: Optional[float],
    routing_method_type: int = 0,
    use_shuffled_weight: bool = False,
    weight_layout: int = 0,
    enable_pdl: Optional[bool] = None,
    tune_max_num_tokens: int = 8192,
    fp8_quantization_type: Optional[int] = None,
    activation_type: Optional[int] = None,
) -> torch.Tensor:
    return torch.empty(
        hidden_states.shape, dtype=torch.bfloat16, device=hidden_states.device
    )
```
**EN:** This block defines `_fake_fp8_block_scale_moe` and contains the main logic for this step. It mainly invokes `torch.empty`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_fake_fp8_block_scale_moe`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 38-106: Function `trtllm_fp8_block_scale_moe_wrapper` and its core logic
```python
@register_custom_op(fake_impl=_fake_fp8_block_scale_moe)
def trtllm_fp8_block_scale_moe_wrapper(
    routing_logits: torch.Tensor,
    routing_bias: Optional[torch.Tensor],
    hidden_states: torch.Tensor,
    hidden_states_scale: torch.Tensor,
    gemm1_weights: torch.Tensor,
    gemm1_weights_scale: torch.Tensor,
    gemm2_weights: torch.Tensor,
    gemm2_weights_scale: torch.Tensor,
    num_experts: int,
    top_k: int,
    n_group: Optional[int],
    topk_group: Optional[int],
    intermediate_size: int,
    local_expert_offset: int,
    local_num_experts: int,
    routed_scaling_factor: Optional[float],
    routing_method_type: int = 0,
    use_shuffled_weight: bool = False,
    weight_layout: int = 0,
    enable_pdl: Optional[bool] = None,
    tune_max_num_tokens: int = 8192,
    fp8_quantization_type: Optional[int] = None,
    activation_type: Optional[int] = None,
) -> torch.Tensor:
    try:
        from flashinfer.fused_moe import trtllm_fp8_block_scale_moe
    except ImportError as e:
        raise ImportError(
            "Can't import trtllm_fp8_block_scale_moe from flashinfer. "
            "Please check flashinfer version."
        ) from e
    kwargs = {
        "routing_logits": routing_logits,
        "routing_bias": routing_bias,
        "hidden_states": hidden_states,
        "hidden_states_scale": hidden_states_scale,
        "gemm1_weights": gemm1_weights,
        "gemm1_weights_scale": gemm1_weights_scale,
        "gemm2_weights": gemm2_weights,
        "gemm2_weights_scale": gemm2_weights_scale,
        "num_experts": num_experts,
        "top_k": top_k,
        "n_group": n_group,
        "topk_group": topk_group,
        "intermediate_size": intermediate_size,
        "local_expert_offset": local_expert_offset,
        "local_num_experts": local_num_experts,
        "routed_scaling_factor": routed_scaling_factor,
        "routing_method_type": routing_method_type,
        "use_shuffled_weight": use_shuffled_weight,
        "weight_layout": weight_layout,
        "enable_pdl": enable_pdl,
        "tune_max_num_tokens": tune_max_num_tokens,
    }
    if fp8_quantization_type is not None:
        from flashinfer.fused_moe import Fp8QuantizationType

        kwargs["fp8_quantization_type"] = Fp8QuantizationType(fp8_quantization_type)

    if activation_type is not None:
        from flashinfer.fused_moe.core import ActivationType

        kwargs["activation_type"] = ActivationType(activation_type)

    return trtllm_fp8_block_scale_moe(**kwargs)
```
**EN:** This block defines `trtllm_fp8_block_scale_moe_wrapper` and contains the main logic for this step. Decorators like `register_custom_op` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_custom_op`, `trtllm_fp8_block_scale_moe`, `Fp8QuantizationType`, `ActivationType`, and `ImportError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `kwargs` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `trtllm_fp8_block_scale_moe_wrapper`，并承载这一阶段的核心逻辑。 像 `register_custom_op` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_custom_op`、`trtllm_fp8_block_scale_moe`、`Fp8QuantizationType`、`ActivationType` 以及 `ImportError`，说明该流程会编排底层辅助函数或计算内核。 像 `kwargs` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 107-136: Internal helper `_fake_fp8_block_scale_routed_moe`
```python
def _fake_fp8_block_scale_routed_moe(
    topk_ids: torch.Tensor,
    routing_bias: Optional[torch.Tensor],
    hidden_states: torch.Tensor,
    hidden_states_scale: torch.Tensor,
    gemm1_weights: torch.Tensor,
    gemm1_weights_scale: torch.Tensor,
    gemm2_weights: torch.Tensor,
    gemm2_weights_scale: torch.Tensor,
    num_experts: int,
    top_k: int,
    n_group: Optional[int],
    topk_group: Optional[int],
    intermediate_size: int,
    local_expert_offset: int,
    local_num_experts: int,
    routed_scaling_factor: Optional[float],
    routing_method_type: int = 0,
    use_shuffled_weight: bool = False,
    weight_layout: int = 0,
    enable_pdl: Optional[bool] = None,
    tune_max_num_tokens: int = 8192,
    fp8_quantization_type: Optional[int] = None,
    activation_type: Optional[int] = None,
) -> torch.Tensor:
    return torch.empty(
        hidden_states.shape, dtype=torch.bfloat16, device=hidden_states.device
    )
```
**EN:** This block defines `_fake_fp8_block_scale_routed_moe` and contains the main logic for this step. It mainly invokes `torch.empty`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_fake_fp8_block_scale_routed_moe`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 137-205: Function `trtllm_fp8_block_scale_routed_moe_wrapper` and its core logic
```python
@register_custom_op(fake_impl=_fake_fp8_block_scale_routed_moe)
def trtllm_fp8_block_scale_routed_moe_wrapper(
    topk_ids: torch.Tensor,
    routing_bias: Optional[torch.Tensor],
    hidden_states: torch.Tensor,
    hidden_states_scale: torch.Tensor,
    gemm1_weights: torch.Tensor,
    gemm1_weights_scale: torch.Tensor,
    gemm2_weights: torch.Tensor,
    gemm2_weights_scale: torch.Tensor,
    num_experts: int,
    top_k: int,
    n_group: Optional[int],
    topk_group: Optional[int],
    intermediate_size: int,
    local_expert_offset: int,
    local_num_experts: int,
    routed_scaling_factor: Optional[float],
    routing_method_type: int = 0,
    use_shuffled_weight: bool = False,
    weight_layout: int = 0,
    enable_pdl: Optional[bool] = None,
    tune_max_num_tokens: int = 8192,
    fp8_quantization_type: Optional[int] = None,
    activation_type: Optional[int] = None,
) -> torch.Tensor:
    try:
        from flashinfer.fused_moe import trtllm_fp8_block_scale_routed_moe
    except ImportError as e:
        raise ImportError(
            "Can't import trtllm_fp8_block_scale_routed_moe from flashinfer. "
            "Please check flashinfer version."
        ) from e
    kwargs = {
        "topk_ids": topk_ids,
        "routing_bias": routing_bias,
        "hidden_states": hidden_states,
        "hidden_states_scale": hidden_states_scale,
        "gemm1_weights": gemm1_weights,
        "gemm1_weights_scale": gemm1_weights_scale,
        "gemm2_weights": gemm2_weights,
        "gemm2_weights_scale": gemm2_weights_scale,
        "num_experts": num_experts,
        "top_k": top_k,
        "n_group": n_group,
        "topk_group": topk_group,
        "intermediate_size": intermediate_size,
        "local_expert_offset": local_expert_offset,
        "local_num_experts": local_num_experts,
        "routed_scaling_factor": routed_scaling_factor,
        "routing_method_type": routing_method_type,
        "use_shuffled_weight": use_shuffled_weight,
        "weight_layout": weight_layout,
        "enable_pdl": enable_pdl,
        "tune_max_num_tokens": tune_max_num_tokens,
    }
    if fp8_quantization_type is not None:
        from flashinfer.fused_moe import Fp8QuantizationType

        kwargs["fp8_quantization_type"] = Fp8QuantizationType(fp8_quantization_type)

    if activation_type is not None:
        from flashinfer.fused_moe.core import ActivationType

        kwargs["activation_type"] = ActivationType(activation_type)

    return trtllm_fp8_block_scale_routed_moe(**kwargs)
```
**EN:** This block defines `trtllm_fp8_block_scale_routed_moe_wrapper` and contains the main logic for this step. Decorators like `register_custom_op` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_custom_op`, `trtllm_fp8_block_scale_routed_moe`, `Fp8QuantizationType`, `ActivationType`, and `ImportError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `kwargs` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `trtllm_fp8_block_scale_routed_moe_wrapper`，并承载这一阶段的核心逻辑。 像 `register_custom_op` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_custom_op`、`trtllm_fp8_block_scale_routed_moe`、`Fp8QuantizationType`、`ActivationType` 以及 `ImportError`，说明该流程会编排底层辅助函数或计算内核。 像 `kwargs` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 206-233: Internal helper `_fake_fp8_per_tensor_scale_moe`
```python
def _fake_fp8_per_tensor_scale_moe(
    routing_logits: torch.Tensor,
    routing_bias: Optional[torch.Tensor],
    hidden_states: torch.Tensor,
    gemm1_weights: torch.Tensor,
    output1_scales_scalar: torch.Tensor,
    output1_scales_gate_scalar: torch.Tensor,
    gemm2_weights: torch.Tensor,
    output2_scales_scalar: torch.Tensor,
    num_experts: int,
    top_k: int,
    n_group: Optional[int],
    topk_group: Optional[int],
    intermediate_size: int,
    local_expert_offset: int,
    local_num_experts: int,
    routed_scaling_factor: Optional[float],
    use_routing_scales_on_input: bool,
    routing_method_type: int = 0,
    enable_pdl: Optional[bool] = None,
    tune_max_num_tokens: int = 8192,
    activation_type: Optional[int] = None,
) -> torch.Tensor:
    return torch.empty(
        hidden_states.shape, dtype=torch.bfloat16, device=hidden_states.device
    )
```
**EN:** This block defines `_fake_fp8_per_tensor_scale_moe` and contains the main logic for this step. It mainly invokes `torch.empty`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_fake_fp8_per_tensor_scale_moe`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 234-295: Function `trtllm_fp8_per_tensor_scale_moe_wrapper` and its core logic
```python
@register_custom_op(fake_impl=_fake_fp8_per_tensor_scale_moe)
def trtllm_fp8_per_tensor_scale_moe_wrapper(
    routing_logits: torch.Tensor,
    routing_bias: Optional[torch.Tensor],
    hidden_states: torch.Tensor,
    gemm1_weights: torch.Tensor,
    output1_scales_scalar: torch.Tensor,
    output1_scales_gate_scalar: torch.Tensor,
    gemm2_weights: torch.Tensor,
    output2_scales_scalar: torch.Tensor,
    num_experts: int,
    top_k: int,
    n_group: Optional[int],
    topk_group: Optional[int],
    intermediate_size: int,
    local_expert_offset: int,
    local_num_experts: int,
    routed_scaling_factor: Optional[float],
    use_routing_scales_on_input: bool,
    routing_method_type: int = 0,
    enable_pdl: Optional[bool] = None,
    tune_max_num_tokens: int = 8192,
    activation_type: Optional[int] = None,
) -> torch.Tensor:
    # lazy import
    try:
        from flashinfer.fused_moe import trtllm_fp8_per_tensor_scale_moe
    except ImportError as e:
        raise ImportError(
            "Can't import trtllm_fp8_per_tensor_scale_moe from flashinfer. "
            "Please check flashinfer version."
        ) from e

    kwargs = {
        "routing_logits": routing_logits,
        "routing_bias": routing_bias,
        "hidden_states": hidden_states,
        "gemm1_weights": gemm1_weights,
        "output1_scales_scalar": output1_scales_scalar,
        "output1_scales_gate_scalar": output1_scales_gate_scalar,
        "gemm2_weights": gemm2_weights,
        "output2_scales_scalar": output2_scales_scalar,
        "num_experts": num_experts,
        "top_k": top_k,
        "n_group": n_group,
        "topk_group": topk_group,
        "intermediate_size": intermediate_size,
        "local_expert_offset": local_expert_offset,
        "local_num_experts": local_num_experts,
        "routed_scaling_factor": routed_scaling_factor,
        "use_routing_scales_on_input": use_routing_scales_on_input,
        "routing_method_type": routing_method_type,
        "enable_pdl": enable_pdl,
        "tune_max_num_tokens": tune_max_num_tokens,
    }

    if activation_type is not None:
        from flashinfer.fused_moe.core import ActivationType

        kwargs["activation_type"] = ActivationType(activation_type)

    return trtllm_fp8_per_tensor_scale_moe(**kwargs)
```
**EN:** This block defines `trtllm_fp8_per_tensor_scale_moe_wrapper` and contains the main logic for this step. Decorators like `register_custom_op` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_custom_op`, `trtllm_fp8_per_tensor_scale_moe`, `ActivationType`, and `ImportError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `kwargs` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `trtllm_fp8_per_tensor_scale_moe_wrapper`，并承载这一阶段的核心逻辑。 像 `register_custom_op` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_custom_op`、`trtllm_fp8_per_tensor_scale_moe`、`ActivationType` 以及 `ImportError`，说明该流程会编排底层辅助函数或计算内核。 像 `kwargs` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `_fake_fp8_block_scale_moe`, `trtllm_fp8_block_scale_moe_wrapper`, `_fake_fp8_block_scale_routed_moe`, `trtllm_fp8_block_scale_routed_moe_wrapper`, and `_fake_fp8_per_tensor_scale_moe`. / **主要符号**：核心入口包括 `_fake_fp8_block_scale_moe`、`trtllm_fp8_block_scale_moe_wrapper`、`_fake_fp8_block_scale_routed_moe`、`trtllm_fp8_block_scale_routed_moe_wrapper` 以及 `_fake_fp8_per_tensor_scale_moe`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `typing.Optional` / **标准库**：`typing.Optional`
- **Third-party**: `torch`, `flashinfer.fused_moe.trtllm_fp8_block_scale_moe`, `flashinfer.fused_moe.Fp8QuantizationType`, `flashinfer.fused_moe.core.ActivationType`, `flashinfer.fused_moe.trtllm_fp8_block_scale_routed_moe`, and `flashinfer.fused_moe.trtllm_fp8_per_tensor_scale_moe` / **第三方依赖**：`torch`、`flashinfer.fused_moe.trtllm_fp8_block_scale_moe`、`flashinfer.fused_moe.Fp8QuantizationType`、`flashinfer.fused_moe.core.ActivationType`、`flashinfer.fused_moe.trtllm_fp8_block_scale_routed_moe` 以及 `flashinfer.fused_moe.trtllm_fp8_per_tensor_scale_moe`
- **Internal SGLang modules**: `sglang.srt.utils.custom_op.register_custom_op` / **SGLang 内部模块**：`sglang.srt.utils.custom_op.register_custom_op`
