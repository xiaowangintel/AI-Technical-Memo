# flashinfer_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/utils/flashinfer_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Collects utility helpers for quantization backends, schemes, and utilities. / 汇总量化后端、方案与工具的工具函数。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 3-12)
```python
from enum import Enum
from typing import TYPE_CHECKING

import torch

from vllm import envs
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.platforms import current_platform
from vllm.utils.math_utils import round_up
```
**EN:** This opening block pulls in external dependencies such as `enum`, `typing`, `torch` and internal modules such as `vllm`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.platforms`, `vllm.utils.math_utils`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `enum`, `typing`, `torch`）以及内部模块（如 `vllm`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.platforms`, `vllm.utils.math_utils`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 17-17)
```python
logger = init_logger(__name__)
```
**EN:** This block defines module-level metadata or constants such as `logger`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `logger`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Function `rotate_weights_for_fi_trtllm_fp8_per_tensor_moe` (lines 50-95)
```python
def rotate_weights_for_fi_trtllm_fp8_per_tensor_moe(
    gemm1_weights: torch.Tensor, gemm2_weights: torch.Tensor, is_gated_activation: bool
):
    """Shuffle weights for FI TRT-LLM Format"""
    from flashinfer import reorder_rows_for_gated_act_gemm, shuffle_matrix_a

    epilogue_tile_m = 128
    num_experts = gemm1_weights.shape[0]
    hidden_size = gemm1_weights.shape[-1]
    intermediate_size = gemm1_weights.shape[1] // 2

    # Reorder rows of W1 for fused gated activation
    gemm1_weights_fp8_interleaved = []
    for i in range(num_experts):
        gemm1_weights_fp8_interleaved.append(
            reorder_rows_for_gated_act_gemm(gemm1_weights[i])
            if is_gated_activation
            else gemm1_weights[i]
        )

    # Stack weights and scales for all experts
    gemm1_weights_fp8_interleaved = torch.stack(gemm1_weights_fp8_interleaved).reshape(
        num_experts, 2 * intermediate_size, hidden_size
    )

    # Shuffle weights and scaling factors for transposed mma output
    gemm1_weights_fp8_shuffled = []
    gemm2_weights_fp8_shuffled = []
    for i in range(num_experts):
        gemm1_weights_fp8_shuffled.append(
            shuffle_matrix_a(
                gemm1_weights_fp8_interleaved[i].view(torch.uint8), epilogue_tile_m
            )
        )

        gemm2_weights_fp8_shuffled.append(
            shuffle_matrix_a(gemm2_weights[i].view(torch.uint8), epilogue_tile_m)
        )

    # Stack weights for all experts
    gemm1_weights.data = torch.stack(gemm1_weights_fp8_shuffled).view(
        torch.float8_e4m3fn
    )
    gemm2_weights.data = torch.stack(gemm2_weights_fp8_shuffled).view(
        torch.float8_e4m3fn
    )
```
**EN:** Defines function `rotate_weights_for_fi_trtllm_fp8_per_tensor_moe` with signature `rotate_weights_for_fi_trtllm_fp8_per_tensor_moe(gemm1_weights: torch.Tensor, gemm2_weights: torch.Tensor, is_gated_activation: bool)`. It mainly works with `gemm1_weights`, `gemm2_weights`, `is_gated_activation`; implements one step in the quantized-weight execution flow. The body uses branching, iteration, tensor/kernel operations. Key calls include `range`, `torch.stack.reshape`, `torch.stack.view`, `gemm1_weights_fp8_interleaved.append`, `gemm1_weights_fp8_shuffled.append`, `gemm2_weights_fp8_shuffled.append`.
**CN:** 定义函数 `rotate_weights_for_fi_trtllm_fp8_per_tensor_moe`，其签名为 `rotate_weights_for_fi_trtllm_fp8_per_tensor_moe(gemm1_weights: torch.Tensor, gemm2_weights: torch.Tensor, is_gated_activation: bool)`。它主要围绕 `gemm1_weights`, `gemm2_weights`, `is_gated_activation` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理、张量或内核操作。关键调用包括 `range`, `torch.stack.reshape`, `torch.stack.view`, `gemm1_weights_fp8_interleaved.append`, `gemm1_weights_fp8_shuffled.append`, `gemm2_weights_fp8_shuffled.append`。

### Function `get_flashinfer_moe_backend` (lines 98-123)
```python
def get_flashinfer_moe_backend() -> FlashinferMoeBackend:
    backend_map = {
        "throughput": FlashinferMoeBackend.CUTLASS,
        "latency": FlashinferMoeBackend.TENSORRT_LLM,
        "masked_gemm": FlashinferMoeBackend.CUTEDSL,
    }

    flashinfer_moe_backend = envs.VLLM_FLASHINFER_MOE_BACKEND
    if flashinfer_moe_backend in backend_map:
        if (
            flashinfer_moe_backend == "latency"
            and not current_platform.is_device_capability_family(100)
        ):
            logger.info_once(
                "Flashinfer TRTLLM MOE backend is only supported on "
                "SM100 and later, using CUTLASS backend instead",
            )
            return FlashinferMoeBackend.CUTLASS
        return backend_map[flashinfer_moe_backend]
    elif current_platform.is_device_capability(90):
        return FlashinferMoeBackend.CUTLASS

    raise ValueError(
        f"Unknown flashinfer moe backend: {flashinfer_moe_backend!r}. "
        f"Expected one of {list(backend_map.keys())}."
    )
```
**EN:** Defines function `get_flashinfer_moe_backend` with signature `get_flashinfer_moe_backend() -> FlashinferMoeBackend`. It mainly works with object context only; returns a derived property or capability check. The body uses branching, validation/error handling. Key calls include `ValueError`, `current_platform.is_device_capability`, `logger.info_once`, `current_platform.is_device_capability_family`, `list`, `backend_map.keys`.
**CN:** 定义函数 `get_flashinfer_moe_backend`，其签名为 `get_flashinfer_moe_backend() -> FlashinferMoeBackend`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含分支判断、校验或报错逻辑。关键调用包括 `ValueError`, `current_platform.is_device_capability`, `logger.info_once`, `current_platform.is_device_capability_family`, `list`, `backend_map.keys`。

### Function `convert_moe_weights_to_flashinfer_trtllm_block_layout` (lines 136-208)
```python
def convert_moe_weights_to_flashinfer_trtllm_block_layout(
    cache_permute_indices: dict[torch.Size, torch.Tensor],
    w13_weight: torch.Tensor,
    w2_weight: torch.Tensor,
) -> tuple[torch.Tensor, torch.Tensor]:
    """Convert expert weights to FlashInfer's block layout.

    This reorders W13 and W2 into the expected epilogue-tiled block layout and
    returns the shuffled weight tensors.
    """
    if w13_weight.dtype != torch.bfloat16 or w2_weight.dtype != torch.bfloat16:
        raise ValueError(
            "Unquantized Moe Backend FlashInfer TRTLLM requires bfloat16 weights"
        )

    from flashinfer.fused_moe.core import (
        _maybe_get_cached_w3_w1_permute_indices,
        convert_to_block_layout,
        get_w2_permute_indices_with_cache,
    )

    epilogue_tile_m = 128
    block_k = 128

    # Reorder rows of W13 and W2 for fused gated activation and convert to the
    # block layout expected by the FlashInfer kernel.
    num_experts = w13_weight.shape[0]
    device_w13 = w13_weight.device
    device_w2 = w2_weight.device

    w13_weights_shuffled: list[torch.Tensor] = []
    w2_weights_shuffled: list[torch.Tensor] = []

    for i in range(num_experts):
        permute_indices = _maybe_get_cached_w3_w1_permute_indices(
            cache_permute_indices,
            w13_weight[i].view(torch.uint8),
            epilogue_tile_m,
# ... truncated for analysis ...
        tmp_weights2 = convert_to_block_layout(tmp_weights2.view(torch.uint8), block_k)

        w13_weights_shuffled.append(tmp_weights1.view(torch.bfloat16))
        w2_weights_shuffled.append(tmp_weights2.view(torch.bfloat16))

    # Stack weights for all experts and return as BF16 tensors.
    w13_weights_shuffled_tensor = (
        torch.stack(w13_weights_shuffled).view(torch.bfloat16).contiguous()
    )
    w2_weights_shuffled_tensor = (
        torch.stack(w2_weights_shuffled).view(torch.bfloat16).contiguous()
    )

    return w13_weights_shuffled_tensor, w2_weights_shuffled_tensor
```
**EN:** Defines function `convert_moe_weights_to_flashinfer_trtllm_block_layout` with signature `convert_moe_weights_to_flashinfer_trtllm_block_layout(cache_permute_indices: dict[torch.Size, torch.Tensor], w13_weight: torch.Tensor, w2_weight: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `cache_permute_indices`, `w13_weight`, `w2_weight`; implements one step in the quantized-weight execution flow. The body uses branching, iteration, validation/error handling, tensor/kernel operations. Key calls include `range`, `torch.stack.view.contiguous`, `ValueError`, `_maybe_get_cached_w3_w1_permute_indices`, `w13_weight.clone.view.contiguous`, `get_w2_permute_indices_with_cache`.
**CN:** 定义函数 `convert_moe_weights_to_flashinfer_trtllm_block_layout`，其签名为 `convert_moe_weights_to_flashinfer_trtllm_block_layout(cache_permute_indices: dict[torch.Size, torch.Tensor], w13_weight: torch.Tensor, w2_weight: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `cache_permute_indices`, `w13_weight`, `w2_weight` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理、校验或报错逻辑、张量或内核操作。关键调用包括 `range`, `torch.stack.view.contiguous`, `ValueError`, `_maybe_get_cached_w3_w1_permute_indices`, `w13_weight.clone.view.contiguous`, `get_w2_permute_indices_with_cache`。

### Function `align_fp4_moe_weights_for_fi` (lines 211-263)
```python
def align_fp4_moe_weights_for_fi(
    w13: torch.Tensor,
    w13_scale: torch.Tensor,
    w2: torch.Tensor,
    w2_scale: torch.Tensor,
    is_act_and_mul: bool,
    min_alignment: int = 16,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor, int]:
    """Pad intermediate size so FlashInfer kernels' alignment constraints hold.

    Some FlashInfer FP4 MoE kernels require the intermediate size
    used for GEMM to be divisible by a small alignment value. When this is
    not satisfied (e.g. with certain tensor-parallel sizes), we pad the
    gate/up and down projection weights along the intermediate dim.
    """

    # Current local intermediate size (per partition) is the K dimension of
    # the down projection.
    num_experts, hidden_size, intermediate = w2.shape
    intermediate *= 2  # because of packed FP4

    padded_intermediate = round_up(intermediate, min_alignment)

    if padded_intermediate == intermediate:
        return w13, w13_scale, w2, w2_scale, intermediate

    logger.info_once(
        "Padding intermediate size from %d to %d for up/down projection weights.",
        intermediate,
        padded_intermediate,
    )

    up_mult = 2 if is_act_and_mul else 1
    padded_gate_up_dim = up_mult * padded_intermediate

    # Pad w13 and w2 along its intermediate dimension.
    padded_w13 = w13.new_zeros((num_experts, padded_gate_up_dim, hidden_size // 2))
    padded_w13[:, : w13.shape[1], :] = w13

    padded_w2 = w2.new_zeros((num_experts, hidden_size, padded_intermediate // 2))
    padded_w2[:, :, : w2.shape[2]] = w2

    padded_w13_scale = w13_scale.new_zeros(
        (num_experts, padded_gate_up_dim, hidden_size // 16)
    )
    padded_w13_scale[:, : w13_scale.shape[1], :] = w13_scale

    padded_w2_scale = w2_scale.new_zeros(
        (num_experts, hidden_size, padded_intermediate // 16)
    )
    padded_w2_scale[:, :, : w2_scale.shape[2]] = w2_scale

    return padded_w13, padded_w13_scale, padded_w2, padded_w2_scale, padded_intermediate
```
**EN:** Defines function `align_fp4_moe_weights_for_fi` with signature `align_fp4_moe_weights_for_fi(w13: torch.Tensor, w13_scale: torch.Tensor, w2: torch.Tensor, w2_scale: torch.Tensor, is_act_and_mul: bool, min_alignment: int=16) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor, int]`. It mainly works with `w13`, `w13_scale`, `w2`, `w2_scale`, `is_act_and_mul`, `min_alignment`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `round_up`, `logger.info_once`, `w13.new_zeros`, `w2.new_zeros`, `w13_scale.new_zeros`, `w2_scale.new_zeros`.
**CN:** 定义函数 `align_fp4_moe_weights_for_fi`，其签名为 `align_fp4_moe_weights_for_fi(w13: torch.Tensor, w13_scale: torch.Tensor, w2: torch.Tensor, w2_scale: torch.Tensor, is_act_and_mul: bool, min_alignment: int=16) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor, int]`。它主要围绕 `w13`, `w13_scale`, `w2`, `w2_scale`, `is_act_and_mul`, `min_alignment` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `round_up`, `logger.info_once`, `w13.new_zeros`, `w2.new_zeros`, `w13_scale.new_zeros`, `w2_scale.new_zeros`。

### Function `align_trtllm_fp4_moe_hidden_dim_for_fi` (lines 266-304)
```python
def align_trtllm_fp4_moe_hidden_dim_for_fi(
    w13: torch.Tensor,
    w13_scale: torch.Tensor,
    w2: torch.Tensor,
    w2_scale: torch.Tensor,
    min_alignment: int = 256,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor, int]:
    num_experts, gate_up_dim, packed_hidden_size = w13.shape
    hidden_size = packed_hidden_size * 2
    padded_hidden_size = round_up(hidden_size, min_alignment)

    if padded_hidden_size == hidden_size:
        return w13, w13_scale, w2, w2_scale, hidden_size

    logger.warning_once(
        "Padding hidden size from %d to %d for TRTLLM NVFP4 MoE weights. "
        "This requires activation slicing at runtime and may cause "
        "performance degradation.",
        hidden_size,
        padded_hidden_size,
    )

    padded_w13 = w13.new_zeros((num_experts, gate_up_dim, padded_hidden_size // 2))
    padded_w13[:, :, :packed_hidden_size] = w13

    padded_w13_scale = w13_scale.new_zeros(
        (num_experts, gate_up_dim, padded_hidden_size // 16)
    )
    padded_w13_scale[:, :, : w13_scale.shape[2]] = w13_scale

    padded_w2 = w2.new_zeros((num_experts, padded_hidden_size, w2.shape[2]))
    padded_w2[:, : w2.shape[1], :] = w2

    padded_w2_scale = w2_scale.new_zeros(
        (num_experts, padded_hidden_size, w2_scale.shape[2])
    )
    padded_w2_scale[:, : w2_scale.shape[1], :] = w2_scale

    return padded_w13, padded_w13_scale, padded_w2, padded_w2_scale, padded_hidden_size
```
**EN:** Defines function `align_trtllm_fp4_moe_hidden_dim_for_fi` with signature `align_trtllm_fp4_moe_hidden_dim_for_fi(w13: torch.Tensor, w13_scale: torch.Tensor, w2: torch.Tensor, w2_scale: torch.Tensor, min_alignment: int=256) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor, int]`. It mainly works with `w13`, `w13_scale`, `w2`, `w2_scale`, `min_alignment`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `round_up`, `logger.warning_once`, `w13.new_zeros`, `w13_scale.new_zeros`, `w2.new_zeros`, `w2_scale.new_zeros`.
**CN:** 定义函数 `align_trtllm_fp4_moe_hidden_dim_for_fi`，其签名为 `align_trtllm_fp4_moe_hidden_dim_for_fi(w13: torch.Tensor, w13_scale: torch.Tensor, w2: torch.Tensor, w2_scale: torch.Tensor, min_alignment: int=256) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor, int]`。它主要围绕 `w13`, `w13_scale`, `w2`, `w2_scale`, `min_alignment` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `round_up`, `logger.warning_once`, `w13.new_zeros`, `w13_scale.new_zeros`, `w2.new_zeros`, `w2_scale.new_zeros`。

### Function `_shuffle_mxfp8_moe_weights` (lines 382-454)
```python
def _shuffle_mxfp8_moe_weights(
    w13: torch.Tensor,
    w2: torch.Tensor,
    w13_scale: torch.Tensor,
    w2_scale: torch.Tensor,
    is_gated: bool,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]:
    """Preprocess MXFP8 weights and scales for the FlashInfer TRT-LLM kernel.

    Following flashinfer/tests/moe/test_trtllm_gen_fused_moe.py:
      1. reorder_rows_for_gated_act_gemm  (interleave gate/up rows)
      2. shuffle_matrix_a                 (weight data layout shuffle)
      3. shuffle_matrix_sf_a              (scale factor layout shuffle)
    """
    from flashinfer import (
        reorder_rows_for_gated_act_gemm,
        shuffle_matrix_a,
        shuffle_matrix_sf_a,
    )

    epilogue_tile_m = 128
    num_experts = w13.shape[0]
    intermediate_size = w13.shape[1] // 2
    hidden_size = w13.shape[2]

    w13_interleaved: list[torch.Tensor] = []
    w13_scale_interleaved: list[torch.Tensor] = []
    for i in range(num_experts):
        if is_gated:
            w13_interleaved.append(
                reorder_rows_for_gated_act_gemm(
                    w13[i].reshape(2 * intermediate_size, -1)
                )
            )
            w13_scale_interleaved.append(
                reorder_rows_for_gated_act_gemm(
                    w13_scale[i].reshape(2 * intermediate_size, -1)
                )
# ... truncated for analysis ...
        )
        w2_scale_shuffled.append(
            shuffle_matrix_sf_a(
                w2_scale[i].view(torch.uint8).reshape(hidden_size, -1),
                epilogue_tile_m,
            )
        )

    w13_out = torch.stack(w13_shuffled).view(torch.float8_e4m3fn)
    w2_out = torch.stack(w2_shuffled).view(torch.float8_e4m3fn)
    w13_scale_out = torch.stack(w13_scale_shuffled).reshape(w13_scale.shape)
    w2_scale_out = torch.stack(w2_scale_shuffled).reshape(w2_scale.shape)

    return w13_out, w2_out, w13_scale_out, w2_scale_out
```
**EN:** Defines function `_shuffle_mxfp8_moe_weights` with signature `_shuffle_mxfp8_moe_weights(w13: torch.Tensor, w2: torch.Tensor, w13_scale: torch.Tensor, w2_scale: torch.Tensor, is_gated: bool) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]`. It mainly works with `w13`, `w2`, `w13_scale`, `w2_scale`, `is_gated`; implements one step in the quantized-weight execution flow. The body uses branching, iteration, tensor/kernel operations. Key calls include `range`, `torch.stack.view`, `torch.stack.reshape`, `w13_shuffled.append`, `w2_shuffled.append`, `w13_scale_shuffled.append`.
**CN:** 定义函数 `_shuffle_mxfp8_moe_weights`，其签名为 `_shuffle_mxfp8_moe_weights(w13: torch.Tensor, w2: torch.Tensor, w13_scale: torch.Tensor, w2_scale: torch.Tensor, is_gated: bool) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]`。它主要围绕 `w13`, `w2`, `w13_scale`, `w2_scale`, `is_gated` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理、张量或内核操作。关键调用包括 `range`, `torch.stack.view`, `torch.stack.reshape`, `w13_shuffled.append`, `w2_shuffled.append`, `w13_scale_shuffled.append`。

### Function `prepare_fp8_moe_layer_for_fi` (lines 457-548)
```python
def prepare_fp8_moe_layer_for_fi(
    layer: torch.nn.Module,
    w13: torch.Tensor,
    w2: torch.Tensor,
    w13_scale: torch.Tensor,
    w13_input_scale: torch.Tensor | None,
    w2_scale: torch.Tensor,
    w2_input_scale: torch.Tensor | None,
    is_trtllm: bool = False,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]:
    """
    Convert Fp8 MoE weights to flashinfer kernel format

    Note that for trtllm we update the model state dict
    with the scale format needed for these kernels.

    Note that for per-tensor, we update the layer's
    intermediate size if the weights needed padding.
    """

    assert hasattr(layer.moe_config, "is_act_and_mul")
    block_quant = (
        hasattr(layer, "weight_block_size") and layer.weight_block_size is not None
    )
    is_mxfp8 = block_quant and w13_scale.dtype == torch.uint8
    is_deepseek_fp8 = block_quant and not is_mxfp8
    is_gated = layer.activation.is_gated

    # MXFP8 TRT-LLM requires W31 swap + reorder + shuffle.
    if is_mxfp8 and is_trtllm:
        # FlashInfer TRT-LLM SwiGLU expects [up; gate] but vLLM stores
        # [gate; up].  Swap both weights and scales before interleaving.
        if layer.moe_config.is_act_and_mul:
            w13 = swap_w13_to_w31(w13)
            # Scales may be 2D [E, flat] from _quantize_mxfp8_moe_weight;
            # reshape to 3D so swap_w13_to_w31 can flip the two halves,
            # then flatten back.
            if w13_scale.ndim == 2:
# ... truncated for analysis ...
        rotate_weights_for_fi_trtllm_fp8_per_tensor_moe(w13, w2, is_gated)

    # Clamp block scales to avoid NaN from the FlashInfer CUTLASS kernel.
    # Some FP8 models have near-zero block scales (~1e-23) for dead/unused
    # experts. The CUTLASS kernel doesn't handle these correctly on Hopper
    # (SM 9.0), producing NaN instead of near-zero output. Clamping to a
    # small minimum prevents this without affecting model accuracy since
    # these experts' effective weights are already zero.
    if block_quant:
        _FI_CUTLASS_MIN_BLOCK_SCALE = 1e-10
        w13_scale.clamp_(min=_FI_CUTLASS_MIN_BLOCK_SCALE)
        w2_scale.clamp_(min=_FI_CUTLASS_MIN_BLOCK_SCALE)

    return w13, w2, w13_scale, w2_scale
```
**EN:** Defines function `prepare_fp8_moe_layer_for_fi` with signature `prepare_fp8_moe_layer_for_fi(layer: torch.nn.Module, w13: torch.Tensor, w2: torch.Tensor, w13_scale: torch.Tensor, w13_input_scale: torch.Tensor | None, w2_scale: torch.Tensor, w2_input_scale: torch.Tensor | None, is_trtllm: bool=False) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]`. It mainly works with `layer`, `w13`, `w2`, `w13_scale`, `w13_input_scale`, `w2_scale`, `w2_input_scale`, `is_trtllm`; constructs runtime helpers from the current configuration. The body uses branching, validation/error handling. Key calls include `hasattr`, `_shuffle_mxfp8_moe_weights`, `align_fp8_moe_weights_for_fi`, `swap_w13_to_w31`, `_shuffle_deepseek_fp8_moe_weights`, `rotate_weights_for_fi_trtllm_fp8_per_tensor_moe`.
**CN:** 定义函数 `prepare_fp8_moe_layer_for_fi`，其签名为 `prepare_fp8_moe_layer_for_fi(layer: torch.nn.Module, w13: torch.Tensor, w2: torch.Tensor, w13_scale: torch.Tensor, w13_input_scale: torch.Tensor | None, w2_scale: torch.Tensor, w2_input_scale: torch.Tensor | None, is_trtllm: bool=False) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]`。它主要围绕 `layer`, `w13`, `w2`, `w13_scale`, `w13_input_scale`, `w2_scale`, `w2_input_scale`, `is_trtllm` 展开；根据当前配置构建运行时辅助对象。函数体包含分支判断、校验或报错逻辑。关键调用包括 `hasattr`, `_shuffle_mxfp8_moe_weights`, `align_fp8_moe_weights_for_fi`, `swap_w13_to_w31`, `_shuffle_deepseek_fp8_moe_weights`, `rotate_weights_for_fi_trtllm_fp8_per_tensor_moe`。

### Class `FlashinferMoeBackend` overview (lines 20-23)
```python
class FlashinferMoeBackend(Enum):
    TENSORRT_LLM = "TensorRT-LLM"
    CUTLASS = "CUTLASS"
    CUTEDSL = "CUTEDSL"
```
**EN:** Defines class `FlashinferMoeBackend` with base classes `Enum` and decorators none. It acts as a quantization-oriented module building block and exposes 0 direct methods, with notable entries no direct methods.
**CN:** 定义类 `FlashinferMoeBackend`，其基类为 `Enum`，装饰器为 无。它在整体实现中充当面向量化的模块构件，并直接暴露 0 个方法，较重要的包括 无直接方法。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `FlashinferMoeBackend`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `FlashinferMoeBackend`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `activation_to_flashinfer_int`, `activation_to_flashinfer_type`, `swap_w13_to_w31`, `rotate_weights_for_fi_trtllm_fp8_per_tensor_moe`, `get_flashinfer_moe_backend` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `activation_to_flashinfer_int`, `activation_to_flashinfer_type`, `swap_w13_to_w31`, `rotate_weights_for_fi_trtllm_fp8_per_tensor_moe`, `get_flashinfer_moe_backend` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `enum`, `typing`, `torch`
- **Internal / 内部**: `vllm`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.platforms`, `vllm.utils.math_utils`
