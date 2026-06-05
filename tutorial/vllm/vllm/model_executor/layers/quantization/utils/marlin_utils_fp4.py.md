# marlin_utils_fp4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/utils/marlin_utils_fp4.py`
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
    marlin_quant_input,
    should_use_atomic_add_reduce,
)
from vllm.platforms import current_platform
from vllm.scalar_type import scalar_types
```
**EN:** This opening block pulls in external dependencies such as `torch` and internal modules such as `vllm._custom_ops`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.platforms`, `vllm.scalar_type`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `torch`）以及内部模块（如 `vllm._custom_ops`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.platforms`, `vllm.scalar_type`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 21-23)
```python
FP4_MARLIN_SUPPORTED_GROUP_SIZES = [16]

logger = init_logger(__name__)
```
**EN:** This block defines module-level metadata or constants such as `FP4_MARLIN_SUPPORTED_GROUP_SIZES`, `logger`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `FP4_MARLIN_SUPPORTED_GROUP_SIZES`, `logger`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Function `nvfp4_marlin_process_scales` (lines 53-114)
```python
def nvfp4_marlin_process_scales(
    marlin_scales: torch.Tensor,
    scale_factor: float | None = None,
    a_dtype: torch.dtype | None = None,
) -> tuple[torch.Tensor, float]:
    """Process NVFP4 weight scales into the special S0E5M3 format for Marlin.

    Args:
        marlin_scales: Weight scales tensor in half precision, already
            permuted for the Marlin kernel layout.
        scale_factor: Optional power-of-2 rescaling factor. If None, the
            factor is computed automatically so that every non-zero scale
            satisfies ``scale * 2^7 >= 2`` (i.e., the MSB of the S0E5M3
            representation is always 1). When provided (e.g., for MoE
            layers where all experts must share the same factor), the
            given value is used directly. The caller is responsible for
            dividing ``global_scale`` by the returned ``scale_factor`` to
            preserve numerical correctness.

    Returns:
        A tuple of (processed_scales, scale_factor).
    """
    if not (marlin_scales >= 0).all():
        logger.warning_once(
            "NVFP4 Marlin assumes the scales to be >=0, but has encountered "
            "negative scales. Accuracy will likely be degraded. This is "
            "because it changes the scales from FP8-S1E4M3 to a special "
            "FP8-S0E5M3 format to speedup the dequantization."
        )

    # convert to half first, we would convert to fp8 later
    marlin_scales = marlin_scales.to(torch.half)

    # fit the layout of fp8 dequantization
    marlin_scales = marlin_scales.view(-1, 4)[:, [0, 2, 1, 3]].view(
        marlin_scales.size(0), -1
    )

# ... truncated for analysis ...
    # to fully utilize the E4M3 dynamic range (e.g., global_scale=1).
    # The caller must compensate by dividing global_scale by scale_factor.
    if scale_factor is None:
        scale_factor = _nvfp4_compute_scale_factor(marlin_scales, a_dtype)
    if scale_factor > 1.0:
        marlin_scales = (marlin_scales.float() * scale_factor).to(torch.half)

    marlin_scales = marlin_scales * (2**7)
    marlin_scales[marlin_scales < 2] = 0
    marlin_scales = marlin_scales.view(torch.int16) << 1
    marlin_scales = marlin_scales.view(torch.float8_e4m3fn)
    marlin_scales = marlin_scales[:, 1::2].contiguous()

    return marlin_scales, scale_factor
```
**EN:** Defines function `nvfp4_marlin_process_scales` with signature `nvfp4_marlin_process_scales(marlin_scales: torch.Tensor, scale_factor: float | None=None, a_dtype: torch.dtype | None=None) -> tuple[torch.Tensor, float]`. It mainly works with `marlin_scales`, `scale_factor`, `a_dtype`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `marlin_scales.to`, `marlin_scales.view.view`, `marlin_scales.view`, `marlin_scales.contiguous`, `all`, `logger.warning_once`.
**CN:** 定义函数 `nvfp4_marlin_process_scales`，其签名为 `nvfp4_marlin_process_scales(marlin_scales: torch.Tensor, scale_factor: float | None=None, a_dtype: torch.dtype | None=None) -> tuple[torch.Tensor, float]`。它主要围绕 `marlin_scales`, `scale_factor`, `a_dtype` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `marlin_scales.to`, `marlin_scales.view.view`, `marlin_scales.view`, `marlin_scales.contiguous`, `all`, `logger.warning_once`。

### Function `apply_fp4_marlin_linear` (lines 149-202)
```python
def apply_fp4_marlin_linear(
    input: torch.Tensor,
    weight: torch.Tensor,
    weight_scale: torch.Tensor,
    weight_global_scale: torch.Tensor | None,
    workspace: torch.Tensor,
    size_n: int,
    size_k: int,
    bias: torch.Tensor | None = None,
    input_dtype: torch.dtype | None = None,
    use_fp32_reduce: bool = USE_FP32_REDUCE_DEFAULT,
) -> torch.Tensor:
    # For GPUs that lack FP4 hardware support, we can leverage the
    # Marlin kernel for fast weight-only FP4 quantization

    reshaped_x = input.reshape(-1, input.shape[-1])
    out_shape = input.shape[:-1] + (size_n,)

    use_atomic_add = should_use_atomic_add_reduce(
        m=reshaped_x.size(0), n=size_n, k=size_k, device=input.device, dtype=input.dtype
    )

    inputs = reshaped_x
    a_scales = None
    is_nvfp4 = weight_global_scale is not None
    if input_dtype is not None and input_dtype.itemsize == 1:
        if is_nvfp4:
            raise RuntimeError("NVFP4 weight + INT8/FP8 activation is not supported.")
        elif input_dtype != torch.float8_e4m3fn:
            raise RuntimeError("MXFP4 weight + INT8 activation is not supported.")

        inputs, a_scales = marlin_quant_input(inputs, torch.float8_e4m3fn)

    output = ops.marlin_gemm(
        a=inputs,
        c=None,
        b_q_weight=weight,
        b_bias=bias,
        b_scales=weight_scale,
        a_scales=a_scales,
        global_scale=weight_global_scale,
        b_zeros=None,
        g_idx=None,
        perm=None,
        workspace=workspace,
        b_q_type=scalar_types.float4_e2m1f,
        size_m=reshaped_x.size(0),
        size_n=size_n,
        size_k=size_k,
        use_atomic_add=use_atomic_add,
        use_fp32_reduce=use_fp32_reduce,
    )

    return output.reshape(out_shape)
```
**EN:** Defines function `apply_fp4_marlin_linear` with signature `apply_fp4_marlin_linear(input: torch.Tensor, weight: torch.Tensor, weight_scale: torch.Tensor, weight_global_scale: torch.Tensor | None, workspace: torch.Tensor, size_n: int, size_k: int, bias: torch.Tensor | None=None, input_dtype: torch.dtype | None=None, use_fp32_reduce: bool=USE_FP32_REDUCE_DEFAULT) -> torch.Tensor`. It mainly works with `input`, `weight`, `weight_scale`, `weight_global_scale`, `workspace`, `size_n`, `size_k`, `bias`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `input.reshape`, `should_use_atomic_add_reduce`, `ops.marlin_gemm`, `output.reshape`, `marlin_quant_input`, `reshaped_x.size`.
**CN:** 定义函数 `apply_fp4_marlin_linear`，其签名为 `apply_fp4_marlin_linear(input: torch.Tensor, weight: torch.Tensor, weight_scale: torch.Tensor, weight_global_scale: torch.Tensor | None, workspace: torch.Tensor, size_n: int, size_k: int, bias: torch.Tensor | None=None, input_dtype: torch.dtype | None=None, use_fp32_reduce: bool=USE_FP32_REDUCE_DEFAULT) -> torch.Tensor`。它主要围绕 `input`, `weight`, `weight_scale`, `weight_global_scale`, `workspace`, `size_n`, `size_k`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `input.reshape`, `should_use_atomic_add_reduce`, `ops.marlin_gemm`, `output.reshape`, `marlin_quant_input`, `reshaped_x.size`。

### Function `prepare_fp4_layer_for_marlin` (lines 205-285)
```python
def prepare_fp4_layer_for_marlin(
    layer: torch.nn.Module, input_dtype: torch.dtype | None = None
) -> None:
    is_nvfp4 = hasattr(layer, "weight_global_scale")
    if input_dtype is not None and input_dtype.itemsize == 1:
        if is_nvfp4:
            raise RuntimeError("NVFP4 weight + INT8/FP8 activation is not supported.")
        elif input_dtype != torch.float8_e4m3fn:
            raise RuntimeError("MXFP4 weight + INT8 activation is not supported.")

    group_size = 16 if is_nvfp4 else 32

    part_size_n = layer.output_size_per_partition
    part_size_k = layer.input_size_per_partition
    param_dtype = layer.params_dtype

    assert layer.weight.shape == (part_size_n, part_size_k // 2)

    device = layer.weight.device

    # WORKSPACE
    layer.workspace = marlin_make_workspace_new(device)

    # WEIGHT
    # Repack weights to marlin format
    perm = torch.empty(0, dtype=torch.int, device=device)
    qweight = layer.weight.view(torch.int32).T.contiguous()

    is_a_8bit = input_dtype is not None and input_dtype.itemsize == 1
    marlin_qweight = ops.gptq_marlin_repack(
        b_q_weight=qweight,
        perm=perm,
        size_k=part_size_k,
        size_n=part_size_n,
        num_bits=4,
        is_a_8bit=is_a_8bit,
    )
    layer.weight = torch.nn.Parameter(marlin_qweight, requires_grad=False)
# ... truncated for analysis ...
            weight_global_scale, requires_grad=False
        )
    else:
        weight_scale = mxfp4_marlin_process_scales(
            weight_scale, input_dtype=input_dtype
        )
        layer.weight_scale = torch.nn.Parameter(weight_scale, requires_grad=False)

    if hasattr(layer, "bias") and layer.bias is not None:
        assert layer.bias.shape == (part_size_n,)
        bias = marlin_permute_bias(layer.bias)
        layer.bias = torch.nn.Parameter(bias, requires_grad=False)

    return
```
**EN:** Defines function `prepare_fp4_layer_for_marlin` with signature `prepare_fp4_layer_for_marlin(layer: torch.nn.Module, input_dtype: torch.dtype | None=None) -> None`. It mainly works with `layer`, `input_dtype`; constructs runtime helpers from the current configuration. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `hasattr`, `marlin_make_workspace_new`, `torch.empty`, `layer.weight.view.T.contiguous`, `ops.gptq_marlin_repack`, `torch.nn.Parameter`.
**CN:** 定义函数 `prepare_fp4_layer_for_marlin`，其签名为 `prepare_fp4_layer_for_marlin(layer: torch.nn.Module, input_dtype: torch.dtype | None=None) -> None`。它主要围绕 `layer`, `input_dtype` 展开；根据当前配置构建运行时辅助对象。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `hasattr`, `marlin_make_workspace_new`, `torch.empty`, `layer.weight.view.T.contiguous`, `ops.gptq_marlin_repack`, `torch.nn.Parameter`。

### Function `prepare_nvfp4_moe_layer_for_marlin` (lines 288-394)
```python
def prepare_nvfp4_moe_layer_for_marlin(
    layer: torch.nn.Module,
    w13: torch.Tensor,
    w13_scale: torch.Tensor,
    w13_scale_2: torch.Tensor,
    w2: torch.Tensor,
    w2_scale: torch.Tensor,
    w2_scale_2: torch.Tensor,
    is_act_and_mul: bool,
) -> tuple[
    torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor
]:
    logger.warning_once(
        "Your GPU does not have native support for FP4 computation but "
        "FP4 quantization is being used. Weight-only FP4 compression will "
        "be used leveraging the Marlin kernel. This may degrade "
        "performance for compute-heavy workloads."
    )

    input_dtype = get_marlin_input_dtype(prefix="")
    if input_dtype is not None and input_dtype.itemsize == 1:
        raise RuntimeError("NVFP4 weight + INT8/FP8 activation is not supported.")

    GROUP_SIZE = 16
    E = layer.num_experts
    K = layer.hidden_size
    N = layer.intermediate_size_per_partition

    device = w13.device
    param_dtype = layer.params_dtype
    is_a_8bit = input_dtype is not None and input_dtype.itemsize == 1

    # WORKSPACE
    layer.workspace = marlin_make_workspace_new(device, 4)
    perm = torch.empty(0, dtype=torch.int, device=device)

    # WEIGHT
    # Repack weights to marlin format
# ... truncated for analysis ...
            marlin_scales, _ = nvfp4_marlin_process_scales(
                marlin_scales, scale_factor=combined_scale_factor, a_dtype=param_dtype
            )
            tensor_list.append(marlin_scales)

        scales = torch.cat([x.unsqueeze(0) for x in tensor_list], 0)
        g_scales = nvfp4_marlin_process_global_scale(g_scales, param_dtype)
        g_scales = g_scales / combined_scale_factor
        return scales, g_scales

    w13_scale, w13_scale_2 = premute_scales(w13_scale, w13_scale_2, "w13")
    w2_scale, w2_scale_2 = premute_scales(w2_scale, w2_scale_2, "w2")

    return w13, w13_scale, w13_scale_2, w2, w2_scale, w2_scale_2
```
**EN:** Defines function `prepare_nvfp4_moe_layer_for_marlin` with signature `prepare_nvfp4_moe_layer_for_marlin(layer: torch.nn.Module, w13: torch.Tensor, w13_scale: torch.Tensor, w13_scale_2: torch.Tensor, w2: torch.Tensor, w2_scale: torch.Tensor, w2_scale_2: torch.Tensor, is_act_and_mul: bool) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]`. It mainly works with `layer`, `w13`, `w13_scale`, `w13_scale_2`, `w2`, `w2_scale`, `w2_scale_2`, `is_act_and_mul`; constructs runtime helpers from the current configuration. The body uses branching, iteration, comprehensions, validation/error handling, tensor/kernel operations. Key calls include `logger.warning_once`, `get_marlin_input_dtype`, `marlin_make_workspace_new`, `torch.empty`, `repack_weight`, `premute_scales`.
**CN:** 定义函数 `prepare_nvfp4_moe_layer_for_marlin`，其签名为 `prepare_nvfp4_moe_layer_for_marlin(layer: torch.nn.Module, w13: torch.Tensor, w13_scale: torch.Tensor, w13_scale_2: torch.Tensor, w2: torch.Tensor, w2_scale: torch.Tensor, w2_scale_2: torch.Tensor, is_act_and_mul: bool) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]`。它主要围绕 `layer`, `w13`, `w13_scale`, `w13_scale_2`, `w2`, `w2_scale`, `w2_scale_2`, `is_act_and_mul` 展开；根据当前配置构建运行时辅助对象。函数体包含分支判断、循环处理、推导式、校验或报错逻辑、张量或内核操作。关键调用包括 `logger.warning_once`, `get_marlin_input_dtype`, `marlin_make_workspace_new`, `torch.empty`, `repack_weight`, `premute_scales`。

### Function `prepare_moe_fp4_layer_for_marlin` (lines 397-519)
```python
def prepare_moe_fp4_layer_for_marlin(
    layer: torch.nn.Module, input_dtype: torch.dtype | None = None
) -> None:
    is_nvfp4 = hasattr(layer, "w13_weight_scale_2")
    if input_dtype is not None and input_dtype.itemsize == 1:
        if is_nvfp4:
            raise RuntimeError("NVFP4 weight + INT8/FP8 activation is not supported.")
        elif input_dtype != torch.float8_e4m3fn:
            raise RuntimeError("MXFP4 weight + INT8 activation is not supported.")

    group_size = 16 if is_nvfp4 else 32

    e = layer.moe_config.num_experts
    k = layer.moe_config.hidden_dim
    n = layer.moe_config.intermediate_size_per_partition

    # WORKSPACE
    device = layer.w13_weight.device
    param_dtype = layer.params_dtype
    layer.workspace = marlin_make_workspace_new(device, 4)
    perm = torch.empty(0, dtype=torch.int, device=device)
    is_a_8bit = input_dtype is not None and input_dtype.itemsize == 1

    # WEIGHT
    # Repack weights to marlin format
    for name in ["w13_weight", "w2_weight"]:
        weight = getattr(layer, name)
        tensor_list = []
        if "w13" in name:
            size_n, size_k = n * 2, k
        else:
            size_n, size_k = k, n

        assert weight.shape == (e, size_n, size_k // 2)

        for i in range(e):
            qweight = weight[i].view(torch.int32).T.contiguous()

# ... truncated for analysis ...
    for name in ["w13_bias", "w2_bias"]:
        if not hasattr(layer, name):
            continue
        bias = getattr(layer, name).to(param_dtype)

        tensor_list = []
        for i in range(e):
            expert_bias = bias[i]

            tensor_list.append(marlin_permute_bias(expert_bias))

        bias = torch.cat([x.unsqueeze(0) for x in tensor_list], 0)
        bias = torch.nn.Parameter(bias, requires_grad=False)
        setattr(layer, name, bias)
```
**EN:** Defines function `prepare_moe_fp4_layer_for_marlin` with signature `prepare_moe_fp4_layer_for_marlin(layer: torch.nn.Module, input_dtype: torch.dtype | None=None) -> None`. It mainly works with `layer`, `input_dtype`; constructs runtime helpers from the current configuration. The body uses branching, iteration, comprehensions, validation/error handling, tensor/kernel operations. Key calls include `hasattr`, `marlin_make_workspace_new`, `torch.empty`, `getattr`, `range`, `torch.cat`.
**CN:** 定义函数 `prepare_moe_fp4_layer_for_marlin`，其签名为 `prepare_moe_fp4_layer_for_marlin(layer: torch.nn.Module, input_dtype: torch.dtype | None=None) -> None`。它主要围绕 `layer`, `input_dtype` 展开；根据当前配置构建运行时辅助对象。函数体包含分支判断、循环处理、推导式、校验或报错逻辑、张量或内核操作。关键调用包括 `hasattr`, `marlin_make_workspace_new`, `torch.empty`, `getattr`, `range`, `torch.cat`。

### Function `prepare_moe_mxfp4_layer_for_marlin` (lines 522-633)
```python
def prepare_moe_mxfp4_layer_for_marlin(
    layer: torch.nn.Module,
    w13: torch.Tensor,
    w2: torch.Tensor,
    w13_scale: torch.Tensor,
    w2_scale: torch.Tensor,
    w13_bias: torch.Tensor | None,
    w2_bias: torch.Tensor | None,
) -> tuple[
    torch.Tensor,
    torch.Tensor,
    torch.Tensor,
    torch.Tensor,
    torch.Tensor | None,
    torch.Tensor | None,
]:
    """Pure-function version of prepare_moe_fp4_layer_for_marlin for MXFP4.

    Takes weight tensors as inputs and returns transformed tensors.
    Does NOT modify the layer in-place.
    """
    input_dtype = get_marlin_input_dtype()
    if (
        input_dtype is not None
        and input_dtype.itemsize == 1
        and input_dtype != torch.float8_e4m3fn
    ):
        raise RuntimeError("MXFP4 weight + INT8 activation is not supported.")

    group_size = 32  # MXFP4 block size

    # Derive dimensions from actual weight shapes to handle rounded/padded
    # sizes correctly (e.g., Mxfp4MoEMethod rounds up hidden_dim).
    # w13 shape: (E, 2*N, K//2)
    e = w13.shape[0]
    n = w13.shape[1] // 2  # intermediate_size_per_partition
    k = w13.shape[2] * 2  # hidden_size

# ... truncated for analysis ...
    # BIAS: Permute bias
    def permute_bias(bias: torch.Tensor | None) -> torch.Tensor | None:
        if bias is None:
            return None
        bias = bias.to(param_dtype)
        tensor_list = []
        for i in range(e):
            tensor_list.append(marlin_permute_bias(bias[i]))
        return torch.cat([x.unsqueeze(0) for x in tensor_list], 0)

    w13_bias = permute_bias(w13_bias)
    w2_bias = permute_bias(w2_bias)

    return w13, w2, w13_scale, w2_scale, w13_bias, w2_bias
```
**EN:** Defines function `prepare_moe_mxfp4_layer_for_marlin` with signature `prepare_moe_mxfp4_layer_for_marlin(layer: torch.nn.Module, w13: torch.Tensor, w2: torch.Tensor, w13_scale: torch.Tensor, w2_scale: torch.Tensor, w13_bias: torch.Tensor | None, w2_bias: torch.Tensor | None) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor | None, torch.Tensor | None]`. It mainly works with `layer`, `w13`, `w2`, `w13_scale`, `w2_scale`, `w13_bias`, `w2_bias`; constructs runtime helpers from the current configuration. The body uses branching, iteration, comprehensions, validation/error handling, tensor/kernel operations. Key calls include `get_marlin_input_dtype`, `torch.empty`, `repack_weight`, `permute_scales`, `permute_bias`, `RuntimeError`.
**CN:** 定义函数 `prepare_moe_mxfp4_layer_for_marlin`，其签名为 `prepare_moe_mxfp4_layer_for_marlin(layer: torch.nn.Module, w13: torch.Tensor, w2: torch.Tensor, w13_scale: torch.Tensor, w2_scale: torch.Tensor, w13_bias: torch.Tensor | None, w2_bias: torch.Tensor | None) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor | None, torch.Tensor | None]`。它主要围绕 `layer`, `w13`, `w2`, `w13_scale`, `w2_scale`, `w13_bias`, `w2_bias` 展开；根据当前配置构建运行时辅助对象。函数体包含分支判断、循环处理、推导式、校验或报错逻辑、张量或内核操作。关键调用包括 `get_marlin_input_dtype`, `torch.empty`, `repack_weight`, `permute_scales`, `permute_bias`, `RuntimeError`。

### Function `rand_marlin_weight_mxfp4_like` (lines 693-751)
```python
def rand_marlin_weight_mxfp4_like(weight, group_size, input_dtype=None):
    is_a_8bit = input_dtype is not None and input_dtype.itemsize == 1
    if is_a_8bit:
        assert input_dtype == torch.float8_e4m3fn, (
            "MXFP4 weight + INT8 activation is not supported."
        )

    assert group_size > 0
    size_n, size_k = weight.shape
    device = weight.device

    scales = torch.randint(
        110,
        120,
        (size_n, size_k // group_size),
        dtype=torch.uint8,
        device=weight.device,
    )
    scales = scales.view(torch.float8_e8m0fnu)

    fp4_weight = torch.randint(
        0, 256, (size_n, size_k // 2), dtype=torch.uint8, device=weight.device
    )
    fp4_weight_part_1 = (fp4_weight & 0b10000000) | ((fp4_weight & 0b01110000) >> 2)
    fp4_weight_part_1 = fp4_weight_part_1.view(torch.float8_e4m3fn)
    fp4_weight_part_1 = fp4_weight_part_1.to(weight.dtype) * (2**6)

    fp4_weight2 = fp4_weight << 4
    fp4_weight_part_2 = (fp4_weight2 & 0b10000000) | ((fp4_weight2 & 0b01110000) >> 2)
    fp4_weight_part_2 = fp4_weight_part_2.view(torch.float8_e4m3fn)
    fp4_weight_part_2 = fp4_weight_part_2.to(weight.dtype) * (2**6)

    weight_ref = torch.cat(
        [fp4_weight_part_2.unsqueeze(2), fp4_weight_part_1.unsqueeze(2)], 2
    ).view(size_n, size_k)
    weight_ref = weight_ref * scales.repeat_interleave(group_size, 1).to(weight.dtype)

    perm = torch.empty(0, dtype=torch.int, device=device)
    fp4_weight = fp4_weight.view(torch.int32).T.contiguous()
    marlin_qweight = ops.gptq_marlin_repack(
        b_q_weight=fp4_weight,
        perm=perm,
        size_k=size_k,
        size_n=size_n,
        num_bits=4,
        is_a_8bit=is_a_8bit,
    )

    marlin_scales = marlin_permute_scales(
        s=scales.T.to(weight.dtype),
        size_k=size_k,
        size_n=size_n,
        group_size=group_size,
        is_a_8bit=is_a_8bit,
    )

    marlin_scales = mxfp4_marlin_process_scales(marlin_scales, input_dtype=input_dtype)

    return weight_ref.T, marlin_qweight, marlin_scales.to(torch.float8_e8m0fnu)
```
**EN:** Defines function `rand_marlin_weight_mxfp4_like` with signature `rand_marlin_weight_mxfp4_like(weight, group_size, input_dtype=None)`. It mainly works with `weight`, `group_size`, `input_dtype`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `torch.randint`, `scales.view`, `fp4_weight_part_1.view`, `fp4_weight_part_2.view`, `torch.cat.view`, `torch.empty`.
**CN:** 定义函数 `rand_marlin_weight_mxfp4_like`，其签名为 `rand_marlin_weight_mxfp4_like(weight, group_size, input_dtype=None)`。它主要围绕 `weight`, `group_size`, `input_dtype` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `torch.randint`, `scales.view`, `fp4_weight_part_1.view`, `fp4_weight_part_2.view`, `torch.cat.view`, `torch.empty`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level helpers such as `is_fp4_marlin_supported`, `_nvfp4_compute_scale_factor`, `nvfp4_marlin_process_scales`, `mxfp4_marlin_process_scales`, `nvfp4_marlin_process_global_scale` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `is_fp4_marlin_supported`, `_nvfp4_compute_scale_factor`, `nvfp4_marlin_process_scales`, `mxfp4_marlin_process_scales`, `nvfp4_marlin_process_global_scale` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `torch`
- **Internal / 内部**: `vllm._custom_ops`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.platforms`, `vllm.scalar_type`
