# test_marlin_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/quantization/test_marlin_gemm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / quantization / test_marlin_gemm, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / quantization / test_marlin_gemm 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-6)
```python
"""Tests for the marlin kernel.

Run `pytest tests/kernels/quantization/test_marlin_gemm.py`.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 8-46)
```python
import itertools

import pytest
import torch

from tests.kernels.utils import opcheck
from tests.quantization.utils import is_quant_method_supported
from vllm import _custom_ops as ops
from vllm.model_executor.layers.quantization.utils.int8_utils import (
    per_token_quant_int8,
)
from vllm.model_executor.layers.quantization.utils.marlin_utils import (
    marlin_make_empty_g_idx,
    marlin_make_workspace_new,
    marlin_permute_bias,
    query_marlin_supported_quant_types,
)
from vllm.model_executor.layers.quantization.utils.marlin_utils_fp4 import (
    rand_marlin_weight_mxfp4_like,
    rand_marlin_weight_nvfp4_like,
)
from vllm.model_executor.layers.quantization.utils.marlin_utils_fp8 import (
    marlin_quant_fp8_torch,
)
from vllm.model_executor.layers.quantization.utils.marlin_utils_test import (
    awq_marlin_quantize,
    get_weight_perm,
    marlin_quantize,
    marlin_weights,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    awq_pack,
    gptq_pack,
    gptq_quantize_weights,
    quantize_weights,
    sort_weights,
)
from vllm.platforms import current_platform
from vllm.scalar_type import scalar_types
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as itertools, pytest, torch; shared test helpers from tests.kernels.utils, tests.quantization.utils; and vLLM components like vllm, vllm.model_executor.layers.quantization.utils.int8_utils, vllm.model_executor.layers.quantization.utils.marlin_utils, vllm.model_executor.layers.quantization.utils.marlin_utils_fp4.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 itertools、pytest、torch；共享测试辅助模块，例如 tests.kernels.utils、tests.quantization.utils；vLLM 内部组件，例如 vllm、vllm.model_executor.layers.quantization.utils.int8_utils、vllm.model_executor.layers.quantization.utils.marlin_utils、vllm.model_executor.layers.quantization.utils.marlin_utils_fp4。

### Top-level block starting at line 48 (lines 48-52)
```python
if current_platform.is_rocm():
    pytest.skip(
        "These tests require marlin, which is not supported on ROCm.",
        allow_module_level=True,
    )
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Constants and module state (lines 54-133)
```python
ACT_ORDER_OPTS = [False, True]
K_FULL_OPTS = [False, True]
USE_ATOMIC_ADD_OPTS = [False, True]
USE_FP32_REDUCE_OPTS = [True]

MARLIN_K_CHUNKS = [128]
MARLIN_N_CHUNKS = [64, 256]

MARLIN_REPACK_NK_FACTORS = [
    (4, 8),
    (7, 5),
    (13, 11),
]

MNK_FACTORS = [
    (1, 1, 1),
    (1, 4, 8),
    (26, 37, 13),
    (257, 13, 11),
]

DTYPES = [torch.float16, torch.bfloat16]

DENSE_MARLIN_QUANT_TEST_CONFIGS = [
    # AWQ-INT4
    {"b_type": scalar_types.uint4, "group_blocks": [-1, 2, 4, 8]},
    # GPTQ-INT4
    {
        "b_type": scalar_types.uint4b8,
        "support_act_order": True,
        "group_blocks": [-1, 2, 4, 8],
    },
    # GPTQ-INT8
    {
        "b_type": scalar_types.uint8b128,
        "support_act_order": True,
        "group_blocks": [-1, 2, 4, 8],
    },
    # FP8
    {"b_type": scalar_types.float8_e4m3fn, "group_blocks": [-1, 8]},
    # NVFP4
    {"b_type": scalar_types.float4_e2m1f, "group_blocks": [1]},
    # MXFP4
    {
        "a_type": [scalar_types.bfloat16],
        "b_type": scalar_types.float4_e2m1f,
        "group_blocks": [2],
    },
    # AWQ-INT4 with INT8 activation
    {
        "a_type": [scalar_types.int8],
        "b_type": scalar_types.uint4,
        "group_blocks": [-1, 2, 4, 8],
    },
    # GPTQ-INT4 with INT8 activation
    {
        "a_type": [scalar_types.int8],
        "b_type": scalar_types.uint4b8,
        "group_blocks": [-1, 2, 4, 8],
    },
    # GPTQ-INT4 with FP8 activation
    {
        "a_type": [scalar_types.float8_e4m3fn],
        "b_type": scalar_types.uint4b8,
        "group_blocks": [-1, 2, 4, 8],
    },
    # AWQ-INT4 with FP8 activation
    {
        "a_type": [scalar_types.float8_e4m3fn],
        "b_type": scalar_types.uint4,
        "group_blocks": [-1, 2, 4, 8],
    },
    # MXFP4 with FP8 activation
    {
        "a_type": [scalar_types.float8_e4m3fn],
        "b_type": scalar_types.float4_e2m1f,
        "c_type": [scalar_types.bfloat16],
        "group_blocks": [2],
    },
]
```
**EN:** This block centralizes shared constants and parameter grids, including ACT_ORDER_OPTS, K_FULL_OPTS, USE_ATOMIC_ADD_OPTS, USE_FP32_REDUCE_OPTS, MARLIN_K_CHUNKS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 ACT_ORDER_OPTS、K_FULL_OPTS、USE_ATOMIC_ADD_OPTS、USE_FP32_REDUCE_OPTS、MARLIN_K_CHUNKS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `compute_max_diff` (lines 136-139)
```python
def compute_max_diff(output, output_ref):
    return torch.mean(torch.abs(output - output_ref)) / torch.mean(
        torch.abs(output_ref)
    )
```
**EN:** This helper function implements the shared logic for compute max diff. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 compute max diff 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `rand_data` (lines 142-143)
```python
def rand_data(shape, dtype=torch.float16):
    return torch.randn(shape, dtype=dtype, device="cuda")
```
**EN:** This helper function implements the shared logic for rand data. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 rand data 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_marlin_int4_fp8_preprocess_without_zp` (lines 146-165)
```python
@pytest.mark.skipif(
    not is_quant_method_supported("gptq_marlin"),
    reason="Marlin is not supported on this GPU type.",
)
def test_marlin_int4_fp8_preprocess_without_zp():
    qweight_unpacked = torch.randint(
        0, 16, size=(2048, 2048), dtype=torch.int32, device="cuda"
    )
    qweight_packed = qweight_unpacked[:, ::2] * 16 + qweight_unpacked[:, 1::2]
    qweight_packed = qweight_packed.to(torch.int8).view(torch.int32)

    cuda_res = ops.marlin_int4_fp8_preprocess(qweight_packed)

    torch_res = torch.where(
        qweight_unpacked >= 8, qweight_unpacked - 8, 15 - qweight_unpacked
    )
    torch_res = torch_res[:, ::2] * 16 + torch_res[:, 1::2]
    torch_res = torch_res.to(torch.int8).view(torch.int32)

    assert (cuda_res == torch_res).all()
```
**EN:** This pytest case verifies marlin int4 FP8 preprocess without zp. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 marlin int4 FP8 preprocess without zp 的行为。 不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

### Function `test_marlin_int4_fp8_preprocess_awq` (lines 168-195)
```python
@pytest.mark.skipif(
    not is_quant_method_supported("gptq_marlin"),
    reason="Marlin is not supported on this GPU type.",
)
def test_marlin_int4_fp8_preprocess_awq():
    group_size = 128

    qweight_unpacked = torch.randint(
        0, 16, size=(2048, 2048), dtype=torch.int32, device="cuda"
    )
    qzeros_unpacked = torch.randint(
        0, 16, size=(2048 // group_size, 2048), dtype=torch.int32, device="cuda"
    )

    qweight_packed = qweight_unpacked[:, ::2] * 16 + qweight_unpacked[:, 1::2]
    qweight_packed = qweight_packed.to(torch.int8).view(torch.int32)
    qzeros_packed = qzeros_unpacked[:, ::2] * 16 + qzeros_unpacked[:, 1::2]
    qzeros_packed = qzeros_packed.to(torch.int8).view(torch.int32)

    cuda_res = ops.marlin_int4_fp8_preprocess(qweight_packed, qzeros_packed)

    repeated_zp = qzeros_unpacked.repeat_interleave(group_size, 0)
    torch_res = qweight_unpacked - repeated_zp
    torch_res[torch_res < 0] = 15 - qweight_unpacked[torch_res < 0]
    torch_res = torch_res[:, ::2] * 16 + torch_res[:, 1::2]
    torch_res = torch_res.to(torch.int8).view(torch.int32)

    assert (cuda_res == torch_res).all()
```
**EN:** This pytest case verifies marlin int4 FP8 preprocess awq. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 marlin int4 FP8 preprocess awq 的行为。 不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

### Function `test_gptq_marlin_repack` (lines 198-265)
```python
@pytest.mark.skipif(
    not is_quant_method_supported("gptq_marlin"),
    reason="Marlin is not supported on this GPU type.",
)
@pytest.mark.parametrize("k_chunk", MARLIN_K_CHUNKS)
@pytest.mark.parametrize("n_chunk", MARLIN_N_CHUNKS)
@pytest.mark.parametrize("quant_type", query_marlin_supported_quant_types(False, False))
@pytest.mark.parametrize("act_order", ACT_ORDER_OPTS)
@pytest.mark.parametrize("is_a_8bit", [True, False])
@pytest.mark.parametrize("nk_factors", MARLIN_REPACK_NK_FACTORS)
def test_gptq_marlin_repack(
    k_chunk, n_chunk, quant_type, act_order, is_a_8bit, nk_factors
):
    n_factor, k_factor = nk_factors

    size_k = k_chunk * k_factor
    size_n = n_chunk * n_factor
    group_size = 128

    # Filter act_order
    if act_order:
        if group_size == -1:
            return
        if group_size == size_k:
            return
        if is_a_8bit:
            return

    # Normalize group_size
    if group_size == -1:
        group_size = size_k
    assert group_size <= size_k

    # Create input
    b_weight = rand_data((size_k, size_n))

    # Quantize (and apply act_order if provided)
    w_ref, q_w, s, g_idx, rand_perm = gptq_quantize_weights(
        b_weight, quant_type, group_size, act_order
    )

    # Pack to GPTQ format
    q_w_gptq = gptq_pack(q_w, quant_type.size_bits, size_k, size_n)

    # For act_order, sort the "weights" and "g_idx" so that group ids are
    # increasing
    sort_indices = torch.empty(0, dtype=torch.int, device=b_weight.device)
    if act_order:
        q_w, g_idx, sort_indices = sort_weights(q_w, g_idx)

    # Pack to Marlin format
    weight_perm = get_weight_perm(quant_type.size_bits, is_a_8bit)
    marlin_q_w_1 = marlin_weights(
        q_w, size_k, size_n, quant_type.size_bits, weight_perm, is_a_8bit
    )

    opcheck(
        torch.ops._C.gptq_marlin_repack,
        (q_w_gptq, sort_indices, size_k, size_n, quant_type.size_bits, is_a_8bit),
    )

    # Run Marlin repack GPU kernel
    marlin_q_w_2 = ops.gptq_marlin_repack(
        q_w_gptq, sort_indices, size_k, size_n, quant_type.size_bits, is_a_8bit
    )
    torch.accelerator.synchronize()

    torch.testing.assert_close(marlin_q_w_1, marlin_q_w_2)
```
**EN:** This pytest case verifies gptq marlin repack. It is parameterized across 6 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as k_chunk, n_chunk, quant_type, act_order. unsupported hardware, backend, or configuration combinations are skipped early. it also validates that the custom operator entry point is wired correctly.
**CN:** 该 pytest 用例验证 gptq marlin repack 的行为。 它通过 6 组参数化输入覆盖多种场景；它会使用诸如 k_chunk、n_chunk、quant_type、act_order 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；它还会校验自定义算子入口是否正确接线。

### Function `test_awq_marlin_repack` (lines 268-313)
```python
@pytest.mark.skipif(
    not is_quant_method_supported("gptq_marlin"),
    reason="Marlin is not supported on this GPU type.",
)
@pytest.mark.parametrize("k_chunk", MARLIN_K_CHUNKS)
@pytest.mark.parametrize("n_chunk", MARLIN_N_CHUNKS)
@pytest.mark.parametrize("quant_type", query_marlin_supported_quant_types(True))
@pytest.mark.parametrize("is_a_8bit", [True, False])
@pytest.mark.parametrize("nk_factors", MARLIN_REPACK_NK_FACTORS)
def test_awq_marlin_repack(k_chunk, n_chunk, quant_type, is_a_8bit, nk_factors):
    n_factor, k_factor = nk_factors

    size_k = k_chunk * k_factor
    size_n = n_chunk * n_factor

    group_size = 128

    # Create input
    b_weight = rand_data((size_k, size_n))

    # Quantize
    w_ref, q_w, s, zp = quantize_weights(
        b_weight, quant_type, group_size, zero_points=True
    )

    # Pack to AWQ format
    q_w_awq = awq_pack(q_w, quant_type.size_bits, size_k, size_n)

    # Pack to Marlin format
    weight_perm = get_weight_perm(quant_type.size_bits, is_a_8bit)
    marlin_q_w_1 = marlin_weights(
        q_w, size_k, size_n, quant_type.size_bits, weight_perm, is_a_8bit
    )

    opcheck(
        torch.ops._C.awq_marlin_repack,
        (q_w_awq, size_k, size_n, quant_type.size_bits, is_a_8bit),
    )

    # Run Marlin repack GPU kernel
    marlin_q_w_2 = ops.awq_marlin_repack(
        q_w_awq, size_k, size_n, quant_type.size_bits, is_a_8bit
    )
    torch.accelerator.synchronize()

    torch.testing.assert_close(marlin_q_w_1, marlin_q_w_2)
```
**EN:** This pytest case verifies awq marlin repack. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as k_chunk, n_chunk, quant_type, is_a_8bit. unsupported hardware, backend, or configuration combinations are skipped early. it also validates that the custom operator entry point is wired correctly.
**CN:** 该 pytest 用例验证 awq marlin repack 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 k_chunk、n_chunk、quant_type、is_a_8bit 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；它还会校验自定义算子入口是否正确接线。

### Function `marlin_generate_valid_test_cases` (lines 316-391)
```python
def marlin_generate_valid_test_cases():
    all_combinations = itertools.product(
        DENSE_MARLIN_QUANT_TEST_CONFIGS,
        MNK_FACTORS,
        MARLIN_N_CHUNKS,
        MARLIN_K_CHUNKS,
        ACT_ORDER_OPTS,
        K_FULL_OPTS,
        USE_ATOMIC_ADD_OPTS,
        USE_FP32_REDUCE_OPTS,
    )

    def is_invalid(
        a_type,
        b_type,
        c_type,
        group_blocks,
        size_m,
        size_n,
        size_k,
        act_order,
        is_k_full,
        use_atomic_add,
        use_fp32_reduce,
    ):
        if use_atomic_add:
            if use_fp32_reduce:
                return False
            if (
                c_type == scalar_types.bfloat16
                and torch.cuda.get_device_capability()[0] < 9
            ):
                return False

        group_size = group_blocks if group_blocks <= 0 else group_blocks * 16
        if group_size > 0 and size_k % group_size != 0:
            return False

        if act_order and group_size in [-1, size_k]:
            return False
        if group_size == size_k:
            return False
        if not act_order and is_k_full:
            return False

        return a_type.size_bits < 16 or a_type is c_type

    cases = []
    for case in all_combinations:
        quant_test_config, mnk_factors, n_chunk, k_chunk, act_order, *_ = case
        size_m = mnk_factors[0]
        size_n = mnk_factors[1] * n_chunk
        size_k = mnk_factors[2] * k_chunk

        if act_order and not quant_test_config.get("support_act_order", False):
            continue

        f16_types = [scalar_types.float16, scalar_types.bfloat16]
        inner_combinations = itertools.product(
            quant_test_config.get("a_type", f16_types),
            [quant_test_config["b_type"]],
            quant_test_config.get("c_type", f16_types),
            quant_test_config["group_blocks"],
        )

        for sub_case in inner_combinations:
            if (
                sub_case[0] == scalar_types.float8_e4m3fn
                and not current_platform.is_device_capability(89)
                and not current_platform.is_device_capability_family(120)
            ):
                continue
            args = sub_case + (size_m, size_n, size_k) + case[4:]
            if is_invalid(*args):
                cases.append(args)
    return cases
```
**EN:** This helper function implements the shared logic for marlin generate valid test cases. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 marlin generate valid test cases 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_marlin_gemm` (lines 394-524)
```python
@pytest.mark.skipif(
    not is_quant_method_supported("gptq_marlin"),
    reason="Marlin is not supported on this GPU type.",
)
@pytest.mark.parametrize(
    (
        "a_type, b_type, c_type, group_blocks,"
        "size_m, size_n, size_k, act_order, is_k_full,"
        "use_atomic_add, use_fp32_reduce"
    ),
    marlin_generate_valid_test_cases(),
)
def test_marlin_gemm(
    a_type,
    b_type,
    c_type,
    group_blocks,
    size_m,
    size_n,
    size_k,
    act_order,
    is_k_full,
    use_atomic_add,
    use_fp32_reduce,
):
    has_zp = b_type in [scalar_types.uint4, scalar_types.uint8]

    group_size = group_blocks if group_blocks <= 0 else group_blocks * 16

    if c_type == scalar_types.float16:
        dtype = torch.float16
    elif c_type == scalar_types.bfloat16:
        dtype = torch.bfloat16
    else:
        raise RuntimeError("unsupported c_type")

    if a_type == scalar_types.int8:
        a_dtype = torch.int8
    elif a_type == scalar_types.float8_e4m3fn:
        a_dtype = torch.float8_e4m3fn
# ... excerpt ...
            a_scales = a_scales.float()
            marlin_s = marlin_s / marlin_s.max() * 4096
            marlin_s = marlin_s.round().to(torch.int16).view(dtype)
    elif a_type == scalar_types.float8_e4m3fn:
        a_input, a_scales = ops.scaled_fp8_quant(a_input, use_per_token_if_dynamic=True)
        a_input_ref = a_input.to(a_scales.dtype) * a_scales.view(-1, 1)
        a_input_ref = a_input_ref.to(dtype)
    else:
        assert a_type.size_bits == 16
        a_input_ref = a_input
        a_scales = None

    output = torch.empty((size_m, size_n), dtype=dtype, device=a_input.device)

    output = ops.marlin_gemm(
        a_input,
        output,
        marlin_q_w,
        None,
        marlin_s,
        a_scales,
        marlin_s2,
        marlin_zp,
        g_idx,
        sort_indices,
        workspace,
        b_type,
        a_input.shape[0],
        b_weight.shape[1],
        a_input.shape[1],
        is_k_full=is_k_full,
        use_atomic_add=use_atomic_add,
        use_fp32_reduce=use_fp32_reduce,
        is_zp_float=False,
    )
    output_ref = torch.matmul(a_input_ref, w_ref)

    max_diff = compute_max_diff(output, output_ref)
    assert max_diff < 0.04
```
**EN:** This pytest case verifies marlin gemm. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as a_type, b_type, c_type, group_blocks. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 marlin gemm 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 a_type、b_type、c_type、group_blocks 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。 由于该代码块较大，这里只展示关键片段。

### Function `test_marlin_gemm_subset_input` (lines 527-572)
```python
def test_marlin_gemm_subset_input():
    quant_type = scalar_types.uint4b8
    group_size = 128

    size_m, size_k, size_n = 32, 1024, 2048
    big_m = size_m * 2
    big_k = size_k * 2

    a_input = rand_data((big_m, big_k))[8 : size_m + 8, 8 : size_k + 8]
    b_weight = rand_data((size_k, size_n))

    w_ref, marlin_q_w, marlin_s, g_idx, sort_indices, _ = marlin_quantize(
        b_weight, quant_type, group_size, False
    )

    marlin_zp = marlin_make_empty_g_idx(marlin_s.device)
    workspace = marlin_make_workspace_new(a_input.device)

    output = ops.marlin_gemm(
        a_input,
        None,
        marlin_q_w,
        None,
        marlin_s,
        None,
        None,
        marlin_zp,
        g_idx,
        sort_indices,
        workspace,
        quant_type,
        a_input.shape[0],
        b_weight.shape[1],
        a_input.shape[1],
        is_k_full=True,
        use_atomic_add=False,
        use_fp32_reduce=True,
        is_zp_float=False,
    )
    output_ref = torch.matmul(a_input, w_ref)

    torch.accelerator.synchronize()

    max_diff = compute_max_diff(output, output_ref)

    assert max_diff < 0.04
```
**EN:** This pytest case verifies marlin gemm subset input. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 marlin gemm subset input 的行为。 结尾处的断言会固定预期行为或计算图形态。

### Function `test_marlin_gemm_with_bias` (lines 575-621)
```python
@pytest.mark.parametrize("size_m", [1, 256])
def test_marlin_gemm_with_bias(size_m):
    quant_type = scalar_types.uint4b8
    group_size = 128

    size_k, size_n = 1024, 2048
    a_input = rand_data((size_m, size_k))
    b_weight = rand_data((size_k, size_n))
    b_bias = rand_data((size_n,)) * 10

    marlin_bias = marlin_permute_bias(b_bias)

    w_ref, marlin_q_w, marlin_s, g_idx, sort_indices, _ = marlin_quantize(
        b_weight, quant_type, group_size, False
    )

    marlin_zp = marlin_make_empty_g_idx(marlin_s.device)
    workspace = marlin_make_workspace_new(a_input.device)

    output = ops.marlin_gemm(
        a_input,
        None,
        marlin_q_w,
        marlin_bias,
        marlin_s,
        None,
        None,
        marlin_zp,
        g_idx,
        sort_indices,
        workspace,
        quant_type,
        a_input.shape[0],
        b_weight.shape[1],
        a_input.shape[1],
        is_k_full=True,
        use_atomic_add=False,
        use_fp32_reduce=True,
        is_zp_float=False,
    )
    output_ref = torch.matmul(a_input, w_ref) + b_bias.view(1, -1)

    torch.accelerator.synchronize()

    max_diff = compute_max_diff(output, output_ref)

    assert max_diff < 0.04
```
**EN:** This pytest case verifies marlin gemm with bias. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as size_m. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 marlin gemm with bias 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 size_m 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `itertools`
- `pytest`
- `torch`
- `tests.kernels.utils -> opcheck`
- `tests.quantization.utils -> is_quant_method_supported`
- `vllm -> _custom_ops`
- `vllm.model_executor.layers.quantization.utils.int8_utils -> per_token_quant_int8`
- `vllm.model_executor.layers.quantization.utils.marlin_utils -> marlin_make_empty_g_idx, marlin_make_workspace_new, marlin_permute_bias, query_marlin_supported_quant_types`
- `vllm.model_executor.layers.quantization.utils.marlin_utils_fp4 -> rand_marlin_weight_mxfp4_like, rand_marlin_weight_nvfp4_like`
- `vllm.model_executor.layers.quantization.utils.marlin_utils_fp8 -> marlin_quant_fp8_torch`
- `vllm.model_executor.layers.quantization.utils.marlin_utils_test -> awq_marlin_quantize, get_weight_perm, marlin_quantize, marlin_weights`
- `vllm.model_executor.layers.quantization.utils.quant_utils -> awq_pack, gptq_pack, gptq_quantize_weights, quantize_weights, sort_weights`
- `vllm.platforms -> current_platform`
- `vllm.scalar_type -> scalar_types`
