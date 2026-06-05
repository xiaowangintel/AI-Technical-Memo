# marlin_utils_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/utils/marlin_utils_fp8.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Collects utility helpers for quantization backends, schemes, and utilities. / 汇总量化后端、方案与工具的工具函数。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 5-19)
```python
import torch

import vllm._custom_ops as ops
from vllm.logger import init_logger
from vllm.model_executor.layers.quantization.utils.marlin_utils import (
    USE_FP32_REDUCE_DEFAULT,
    get_marlin_input_dtype,
    marlin_make_workspace_new,
    marlin_permute_bias,
    marlin_permute_scales,
    should_use_atomic_add_reduce,
)
from vllm.model_executor.utils import replace_parameter
from vllm.platforms import current_platform
from vllm.scalar_type import scalar_types
```
**EN:** This opening block pulls in external dependencies such as `torch` and internal modules such as `vllm._custom_ops`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.model_executor.utils`, `vllm.platforms`, `vllm.scalar_type`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `torch`）以及内部模块（如 `vllm._custom_ops`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.model_executor.utils`, `vllm.platforms`, `vllm.scalar_type`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 21-21)
```python
logger = init_logger(__name__)
```
**EN:** This block defines module-level metadata or constants such as `logger`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `logger`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Function `apply_fp8_marlin_linear` (lines 42-89)
```python
def apply_fp8_marlin_linear(
    input: torch.Tensor,
    weight: torch.Tensor,
    weight_scale: torch.Tensor,
    workspace: torch.Tensor,
    size_n: int,
    size_k: int,
    bias: torch.Tensor | None,
    input_dtype: torch.dtype | None = None,
    use_fp32_reduce: bool = USE_FP32_REDUCE_DEFAULT,
) -> torch.Tensor:
    # For GPUs that lack FP8 hardware support, we can leverage the
    # Marlin kernel for fast weight-only FP8 quantization

    reshaped_x = input.reshape(-1, input.shape[-1])
    out_shape = input.shape[:-1] + (size_n,)

    use_atomic_add = should_use_atomic_add_reduce(
        m=reshaped_x.size(0), n=size_n, k=size_k, device=input.device, dtype=input.dtype
    )

    inputs = reshaped_x
    a_scales = None
    if input_dtype is not None and input_dtype.itemsize == 1:
        # inputs, a_scales = marlin_quant_input(inputs, torch.float8_e4m3fn)
        raise RuntimeError("Marlin W8A8 is not supported.")

    output = ops.marlin_gemm(
        a=inputs,
        c=None,
        b_q_weight=weight,
        b_bias=bias,
        b_scales=weight_scale,
        a_scales=a_scales,
        global_scale=None,
        b_zeros=None,
        g_idx=None,
        perm=None,
        workspace=workspace,
        b_q_type=scalar_types.float8_e4m3fn,
        size_m=reshaped_x.size(0),
        size_n=size_n,
        size_k=size_k,
        use_atomic_add=use_atomic_add,
        use_fp32_reduce=use_fp32_reduce,
    )

    return output.reshape(out_shape)
```
**EN:** Defines function `apply_fp8_marlin_linear` with signature `apply_fp8_marlin_linear(input: torch.Tensor, weight: torch.Tensor, weight_scale: torch.Tensor, workspace: torch.Tensor, size_n: int, size_k: int, bias: torch.Tensor | None, input_dtype: torch.dtype | None=None, use_fp32_reduce: bool=USE_FP32_REDUCE_DEFAULT) -> torch.Tensor`. It mainly works with `input`, `weight`, `weight_scale`, `workspace`, `size_n`, `size_k`, `bias`, `input_dtype`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `input.reshape`, `should_use_atomic_add_reduce`, `ops.marlin_gemm`, `output.reshape`, `RuntimeError`, `reshaped_x.size`.
**CN:** 定义函数 `apply_fp8_marlin_linear`，其签名为 `apply_fp8_marlin_linear(input: torch.Tensor, weight: torch.Tensor, weight_scale: torch.Tensor, workspace: torch.Tensor, size_n: int, size_k: int, bias: torch.Tensor | None, input_dtype: torch.dtype | None=None, use_fp32_reduce: bool=USE_FP32_REDUCE_DEFAULT) -> torch.Tensor`。它主要围绕 `input`, `weight`, `weight_scale`, `workspace`, `size_n`, `size_k`, `bias`, `input_dtype` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `input.reshape`, `should_use_atomic_add_reduce`, `ops.marlin_gemm`, `output.reshape`, `RuntimeError`, `reshaped_x.size`。

### Function `prepare_fp8_layer_for_marlin` (lines 92-198)
```python
def prepare_fp8_layer_for_marlin(
    layer: torch.nn.Module,
    size_k_first: bool = True,
    input_dtype: torch.dtype | None = None,
) -> None:
    logger.warning_once(
        "Your GPU does not have native support for FP8 computation but "
        "FP8 quantization is being used. Weight-only FP8 compression will "
        "be used leveraging the Marlin kernel. This may degrade "
        "performance for compute-heavy workloads."
    )
    if input_dtype is not None and input_dtype.itemsize == 1:
        raise RuntimeError("Marlin W8A8 is not supported.")

    part_size_n = layer.output_size_per_partition
    part_size_k = layer.input_size_per_partition
    weight_block_size = getattr(layer, "weight_block_size", None)

    if size_k_first:
        assert layer.weight.shape == (part_size_k, part_size_n)
    else:
        assert layer.weight.shape == (part_size_n, part_size_k)

    device = layer.weight.device

    # WORKSPACE
    layer.workspace = marlin_make_workspace_new(device)

    # WEIGHT
    # Repack weights to marlin format
    perm = torch.empty(0, dtype=torch.int, device=device)
    qweight = pack_fp8_to_int32(layer.weight, size_k_first)
    if not size_k_first:
        qweight = qweight.T.contiguous()

    marlin_qweight = ops.gptq_marlin_repack(
        b_q_weight=qweight,
        perm=perm,
# ... truncated for analysis ...
    marlin_scales = marlin_permute_scales(
        s=scales, size_k=part_size_k, size_n=part_size_n, group_size=group_size
    )
    if input_dtype != torch.float8_e4m3fn:
        marlin_scales = fp8_fused_exponent_bias_into_scales(marlin_scales)
    if hasattr(layer, "weight_scale"):
        replace_parameter(layer, "weight_scale", marlin_scales)
    elif hasattr(layer, "weight_scale_inv"):
        replace_parameter(layer, "weight_scale_inv", marlin_scales)

    if hasattr(layer, "bias") and layer.bias is not None:
        assert layer.bias.shape == (part_size_n,)
        bias = marlin_permute_bias(layer.bias)
        replace_parameter(layer, "bias", bias)
```
**EN:** Defines function `prepare_fp8_layer_for_marlin` with signature `prepare_fp8_layer_for_marlin(layer: torch.nn.Module, size_k_first: bool=True, input_dtype: torch.dtype | None=None) -> None`. It mainly works with `layer`, `size_k_first`, `input_dtype`; constructs runtime helpers from the current configuration. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `logger.warning_once`, `getattr`, `marlin_make_workspace_new`, `torch.empty`, `pack_fp8_to_int32`, `ops.gptq_marlin_repack`.
**CN:** 定义函数 `prepare_fp8_layer_for_marlin`，其签名为 `prepare_fp8_layer_for_marlin(layer: torch.nn.Module, size_k_first: bool=True, input_dtype: torch.dtype | None=None) -> None`。它主要围绕 `layer`, `size_k_first`, `input_dtype` 展开；根据当前配置构建运行时辅助对象。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `logger.warning_once`, `getattr`, `marlin_make_workspace_new`, `torch.empty`, `pack_fp8_to_int32`, `ops.gptq_marlin_repack`。

### Function `prepare_fp8_moe_layer_for_marlin` (lines 201-319)
```python
def prepare_fp8_moe_layer_for_marlin(
    layer: torch.nn.Module,
    w13_weight: torch.Tensor,
    w2_weight: torch.Tensor,
    w13_weight_scale: torch.Tensor,
    w2_weight_scale: torch.Tensor,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]:
    """
    Shuffle weights and scales into marlin format.

    Note that this function has the side effect of adding a `workspace`
    attribute to the layer. This `workspace` does not need to be
    registered as a Parameter as it is not used during weight reloading.
    """

    logger.warning_once(
        "Your GPU does not have native support for FP8 computation but "
        "FP8 quantization is being used. Weight-only FP8 compression will "
        "be used leveraging the Marlin kernel. This may degrade "
        "performance for compute-heavy workloads."
    )
    input_dtype = get_marlin_input_dtype()
    if input_dtype is not None and input_dtype.itemsize == 1:
        raise NotImplementedError("Marlin W8A8 is not supported.")

    e = layer.num_experts
    k = layer.hidden_size
    n = layer.intermediate_size_per_partition
    w13_n = w13_weight.size(1)
    weight_block_size = getattr(layer, "weight_block_size", None)

    # WORKSPACE
    device = layer.w13_weight.device
    # NOTE(rob): we do not need to register the workspace as a param
    # because it is not used as part of the weight reloading process.
    layer.workspace = marlin_make_workspace_new(device, 4)
    perm = torch.empty(0, dtype=torch.int, device=device)

# ... truncated for analysis ...
            marlin_scales = marlin_permute_scales(
                s=scales[i], size_k=size_k, size_n=size_n, group_size=group_size
            )
            tensor_list.append(marlin_scales)

        scales = torch.cat([x.unsqueeze(0) for x in tensor_list], 0)
        if input_dtype != torch.float8_e4m3fn:
            scales = fp8_fused_exponent_bias_into_scales(scales)
        return scales

    w13_weight_scale = permute_scales(w13_weight_scale, "w13")
    w2_weight_scale = permute_scales(w2_weight_scale, "w2")

    return w13_weight, w2_weight, w13_weight_scale, w2_weight_scale
```
**EN:** Defines function `prepare_fp8_moe_layer_for_marlin` with signature `prepare_fp8_moe_layer_for_marlin(layer: torch.nn.Module, w13_weight: torch.Tensor, w2_weight: torch.Tensor, w13_weight_scale: torch.Tensor, w2_weight_scale: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]`. It mainly works with `layer`, `w13_weight`, `w2_weight`, `w13_weight_scale`, `w2_weight_scale`; constructs runtime helpers from the current configuration. The body uses branching, iteration, comprehensions, validation/error handling, tensor/kernel operations. Key calls include `logger.warning_once`, `get_marlin_input_dtype`, `w13_weight.size`, `getattr`, `marlin_make_workspace_new`, `torch.empty`.
**CN:** 定义函数 `prepare_fp8_moe_layer_for_marlin`，其签名为 `prepare_fp8_moe_layer_for_marlin(layer: torch.nn.Module, w13_weight: torch.Tensor, w2_weight: torch.Tensor, w13_weight_scale: torch.Tensor, w2_weight_scale: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]`。它主要围绕 `layer`, `w13_weight`, `w2_weight`, `w13_weight_scale`, `w2_weight_scale` 展开；根据当前配置构建运行时辅助对象。函数体包含分支判断、循环处理、推导式、校验或报错逻辑、张量或内核操作。关键调用包括 `logger.warning_once`, `get_marlin_input_dtype`, `w13_weight.size`, `getattr`, `marlin_make_workspace_new`, `torch.empty`。

### Function `apply_mxfp8_marlin_linear` (lines 349-390)
```python
def apply_mxfp8_marlin_linear(
    input: torch.Tensor,
    weight: torch.Tensor,
    weight_scale: torch.Tensor,
    workspace: torch.Tensor,
    size_n: int,
    size_k: int,
    bias: torch.Tensor | None = None,
    use_fp32_reduce: bool = USE_FP32_REDUCE_DEFAULT,
) -> torch.Tensor:
    reshaped_x = input.reshape(-1, input.shape[-1])
    out_shape = input.shape[:-1] + (size_n,)

    use_atomic_add = should_use_atomic_add_reduce(
        m=reshaped_x.size(0),
        n=size_n,
        k=size_k,
        device=input.device,
        dtype=input.dtype,
    )

    output = ops.marlin_gemm(
        a=reshaped_x,
        c=None,
        b_q_weight=weight,
        b_bias=bias,
        b_scales=weight_scale,
        a_scales=None,
        global_scale=None,
        b_zeros=None,
        g_idx=None,
        perm=None,
        workspace=workspace,
        b_q_type=scalar_types.float8_e4m3fn,
        size_m=reshaped_x.size(0),
        size_n=size_n,
        size_k=size_k,
        use_atomic_add=use_atomic_add,
        use_fp32_reduce=use_fp32_reduce,
    )

    return output.reshape(out_shape)
```
**EN:** Defines function `apply_mxfp8_marlin_linear` with signature `apply_mxfp8_marlin_linear(input: torch.Tensor, weight: torch.Tensor, weight_scale: torch.Tensor, workspace: torch.Tensor, size_n: int, size_k: int, bias: torch.Tensor | None=None, use_fp32_reduce: bool=USE_FP32_REDUCE_DEFAULT) -> torch.Tensor`. It mainly works with `input`, `weight`, `weight_scale`, `workspace`, `size_n`, `size_k`, `bias`, `use_fp32_reduce`; implements one step in the quantized-weight execution flow. The body uses tensor/kernel operations. Key calls include `input.reshape`, `should_use_atomic_add_reduce`, `ops.marlin_gemm`, `output.reshape`, `reshaped_x.size`.
**CN:** 定义函数 `apply_mxfp8_marlin_linear`，其签名为 `apply_mxfp8_marlin_linear(input: torch.Tensor, weight: torch.Tensor, weight_scale: torch.Tensor, workspace: torch.Tensor, size_n: int, size_k: int, bias: torch.Tensor | None=None, use_fp32_reduce: bool=USE_FP32_REDUCE_DEFAULT) -> torch.Tensor`。它主要围绕 `input`, `weight`, `weight_scale`, `workspace`, `size_n`, `size_k`, `bias`, `use_fp32_reduce` 展开；实现量化权重执行流程中的一个步骤。函数体包含张量或内核操作。关键调用包括 `input.reshape`, `should_use_atomic_add_reduce`, `ops.marlin_gemm`, `output.reshape`, `reshaped_x.size`。

### Function `prepare_mxfp8_layer_for_marlin` (lines 393-449)
```python
def prepare_mxfp8_layer_for_marlin(layer: torch.nn.Module) -> None:
    """Repack MXFP8 weights and scales into Marlin kernel format.

    Expects the layer to have:
      - weight: [N, K] float8_e4m3fn
      - weight_scale: [N, K//32] uint8 (e8m0 encoded)
      - input_size_per_partition / output_size_per_partition
    """
    part_size_n = layer.output_size_per_partition
    part_size_k = layer.input_size_per_partition
    group_size = 32  # MX standard block size

    device = layer.weight.device

    # WORKSPACE
    layer.workspace = marlin_make_workspace_new(device)

    # WEIGHT - repack FP8 weights to Marlin format
    perm = torch.empty(0, dtype=torch.int, device=device)
    qweight = pack_fp8_to_int32(layer.weight, size_k_first=False)
    qweight = qweight.T.contiguous()

    marlin_qweight = ops.gptq_marlin_repack(
        b_q_weight=qweight,
        perm=perm,
        size_k=part_size_k,
        size_n=part_size_n,
        num_bits=8,
    )
    replace_parameter(layer, "weight", marlin_qweight)

    # WEIGHT SCALES
    # Convert uint8 scales -> e8m0fnu -> param_dtype for permutation
    # Scales are [N, K//32], need [K//32, N] for marlin_permute_scales
    param_dtype = torch.get_default_dtype()
    scales = layer.weight_scale.data[:part_size_n, : part_size_k // group_size]
    scales = scales.contiguous()
    scales = scales.view(torch.float8_e8m0fnu).to(param_dtype)
    scales = scales.T.contiguous()

    # Permute scales to Marlin layout
    marlin_scales = marlin_permute_scales(
        s=scales,
        size_k=part_size_k,
        size_n=part_size_n,
        group_size=group_size,
    )

    # Reorder for e8m0 kernel layout and convert back to e8m0fnu
    marlin_scales = mxfp8_marlin_process_scales(marlin_scales)
    replace_parameter(layer, "weight_scale", marlin_scales)

    # BIAS
    if hasattr(layer, "bias") and layer.bias is not None:
        assert layer.bias.shape == (part_size_n,)
        bias = marlin_permute_bias(layer.bias)
        replace_parameter(layer, "bias", bias)
```
**EN:** Defines function `prepare_mxfp8_layer_for_marlin` with signature `prepare_mxfp8_layer_for_marlin(layer: torch.nn.Module) -> None`. It mainly works with `layer`; constructs runtime helpers from the current configuration. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `marlin_make_workspace_new`, `torch.empty`, `pack_fp8_to_int32`, `qweight.T.contiguous`, `ops.gptq_marlin_repack`, `replace_parameter`.
**CN:** 定义函数 `prepare_mxfp8_layer_for_marlin`，其签名为 `prepare_mxfp8_layer_for_marlin(layer: torch.nn.Module) -> None`。它主要围绕 `layer` 展开；根据当前配置构建运行时辅助对象。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `marlin_make_workspace_new`, `torch.empty`, `pack_fp8_to_int32`, `qweight.T.contiguous`, `ops.gptq_marlin_repack`, `replace_parameter`。

### Function `prepare_mxfp8_moe_layer_for_marlin` (lines 452-532)
```python
def prepare_mxfp8_moe_layer_for_marlin(
    layer: torch.nn.Module,
    w13: torch.Tensor,
    w2: torch.Tensor,
    w13_scale: torch.Tensor,
    w2_scale: torch.Tensor,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]:
    """Repack MXFP8 MoE weights and scales into Marlin kernel format.

    Args:
        layer: MoE layer (used to read params_dtype and attach workspace).
        w13: [E, 2*N, K] float8_e4m3fn weights.
        w2:  [E, K, N] float8_e4m3fn weights.
        w13_scale: [E, 2*N, K//32] uint8 e8m0 scales.
        w2_scale:  [E, K, N//32] uint8 e8m0 scales.

    Returns:
        (w13, w2, w13_scale, w2_scale) in Marlin format.
    """
    group_size = 32
    e = w13.shape[0]
    w13_n = w13.shape[1]
    k = w13.shape[2]
    n = w2.shape[2]

    device = w13.device
    param_dtype = torch.get_default_dtype()
    perm = torch.empty(0, dtype=torch.int, device=device)

    layer.workspace = marlin_make_workspace_new(device, 4)

    def repack_weight(weight: torch.Tensor, name: str) -> torch.Tensor:
        if "w13" in name:
            size_n, size_k = w13_n, k
        else:
            size_n, size_k = k, n

        assert weight.shape == (e, size_n, size_k)
# ... truncated for analysis ...
            marlin_s = marlin_permute_scales(
                s=s,
                size_k=size_k,
                size_n=size_n,
                group_size=group_size,
            )
            marlin_s = mxfp8_marlin_process_scales(marlin_s)
            tensor_list.append(marlin_s)
        return torch.cat([x.unsqueeze(0) for x in tensor_list], 0)

    w13_scale = permute_scales(w13_scale, "w13")
    w2_scale = permute_scales(w2_scale, "w2")

    return w13, w2, w13_scale, w2_scale
```
**EN:** Defines function `prepare_mxfp8_moe_layer_for_marlin` with signature `prepare_mxfp8_moe_layer_for_marlin(layer: torch.nn.Module, w13: torch.Tensor, w2: torch.Tensor, w13_scale: torch.Tensor, w2_scale: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]`. It mainly works with `layer`, `w13`, `w2`, `w13_scale`, `w2_scale`; constructs runtime helpers from the current configuration. The body uses branching, iteration, comprehensions, validation/error handling, tensor/kernel operations. Key calls include `torch.get_default_dtype`, `torch.empty`, `marlin_make_workspace_new`, `repack_weight`, `permute_scales`, `range`.
**CN:** 定义函数 `prepare_mxfp8_moe_layer_for_marlin`，其签名为 `prepare_mxfp8_moe_layer_for_marlin(layer: torch.nn.Module, w13: torch.Tensor, w2: torch.Tensor, w13_scale: torch.Tensor, w2_scale: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]`。它主要围绕 `layer`, `w13`, `w2`, `w13_scale`, `w2_scale` 展开；根据当前配置构建运行时辅助对象。函数体包含分支判断、循环处理、推导式、校验或报错逻辑、张量或内核操作。关键调用包括 `torch.get_default_dtype`, `torch.empty`, `marlin_make_workspace_new`, `repack_weight`, `permute_scales`, `range`。

### Function `marlin_quant_fp8_torch` (lines 535-575)
```python
def marlin_quant_fp8_torch(weight, group_size, input_dtype=None):
    is_a_8bit = input_dtype is not None and input_dtype.itemsize == 1
    if is_a_8bit:
        assert input_dtype == torch.float8_e4m3fn

    size_n, size_k = weight.shape
    device = weight.device

    if group_size != -1:
        scales = weight.view(size_n, -1, group_size).abs().max(-1)[0] / 448
        repeated_scales = scales.repeat_interleave(group_size, 1)
        fp8_weight = (weight / repeated_scales).to(torch.float8_e4m3fn)
        weight_ref = fp8_weight.to(weight.dtype) * repeated_scales
    else:
        scales = weight.view(size_n, 1, group_size).abs().max(-1)[0] / 448
        repeated_scales = scales.repeat_interleave(size_k, 1)
        fp8_weight = (weight / repeated_scales).to(torch.float8_e4m3fn)
        weight_ref = fp8_weight.to(weight.dtype) * repeated_scales

    packed_weight = pack_fp8_to_int32(fp8_weight, False).T.contiguous()
    perm = torch.empty(0, dtype=torch.int, device=device)
    marlin_qweight = ops.gptq_marlin_repack(
        b_q_weight=packed_weight,
        perm=perm,
        size_k=size_k,
        size_n=size_n,
        num_bits=8,
        is_a_8bit=is_a_8bit,
    )

    marlin_scales = marlin_permute_scales(
        s=scales.T,
        size_k=size_k,
        size_n=size_n,
        group_size=group_size,
        is_a_8bit=is_a_8bit,
    )

    marlin_scales = fp8_fused_exponent_bias_into_scales(marlin_scales)

    return weight_ref.T, marlin_qweight, marlin_scales
```
**EN:** Defines function `marlin_quant_fp8_torch` with signature `marlin_quant_fp8_torch(weight, group_size, input_dtype=None)`. It mainly works with `weight`, `group_size`, `input_dtype`; handles quantization-related transformation logic. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `pack_fp8_to_int32.T.contiguous`, `torch.empty`, `ops.gptq_marlin_repack`, `marlin_permute_scales`, `fp8_fused_exponent_bias_into_scales`, `scales.repeat_interleave`.
**CN:** 定义函数 `marlin_quant_fp8_torch`，其签名为 `marlin_quant_fp8_torch(weight, group_size, input_dtype=None)`。它主要围绕 `weight`, `group_size`, `input_dtype` 展开；处理量化相关的变换逻辑。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `pack_fp8_to_int32.T.contiguous`, `torch.empty`, `ops.gptq_marlin_repack`, `marlin_permute_scales`, `fp8_fused_exponent_bias_into_scales`, `scales.repeat_interleave`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level helpers such as `is_fp8_marlin_supported`, `fp8_fused_exponent_bias_into_scales`, `apply_fp8_marlin_linear`, `prepare_fp8_layer_for_marlin`, `prepare_fp8_moe_layer_for_marlin` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `is_fp8_marlin_supported`, `fp8_fused_exponent_bias_into_scales`, `apply_fp8_marlin_linear`, `prepare_fp8_layer_for_marlin`, `prepare_fp8_moe_layer_for_marlin` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `torch`
- **Internal / 内部**: `vllm._custom_ops`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.model_executor.utils`, `vllm.platforms`, `vllm.scalar_type`
