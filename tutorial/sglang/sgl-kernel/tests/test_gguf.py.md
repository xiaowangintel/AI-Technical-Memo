# test_gguf.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_gguf.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `GGUF quantization` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `GGUF quantization` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-2: Comments and local context
````python
# SPDX-License-Identifier: Apache-2.0

````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 3-19: Imports and module setup
````python
import random
import sys
from pathlib import Path

import numpy as np
import pytest
import torch
from gguf import GGMLQuantizationType, GGUFReader, ReaderTensor, dequantize
from huggingface_hub import snapshot_download
from sgl_kernel import (
    ggml_dequantize,
    ggml_moe_a8,
    ggml_moe_a8_vec,
    ggml_moe_get_block_size,
    ggml_mul_mat_a8,
    ggml_mul_mat_vec_a8,
)
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 21-22: Constants and configuration
````python
GGUF_SAMPLE = snapshot_download("Isotr0py/test-gguf-sample")
GGUF_SAMPLE_MOE = snapshot_download("SzymonOzog/test-gguf-moe-sample")
````
**EN:** This block defines shared constants or configuration values such as `GGUF_SAMPLE`, `GGUF_SAMPLE_MOE`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `GGUF_SAMPLE`, `GGUF_SAMPLE_MOE`），供后续函数或控制流程复用。

### Lines 25-31: `get_gguf_sample_tensors` definition
````python
def get_gguf_sample_tensors(
    hidden_size: int, quant_type: GGMLQuantizationType
) -> list[ReaderTensor]:
    sample_dir = GGUF_SAMPLE
    filename = f"Quant_{quant_type.name}_{hidden_size}.gguf"
    sample_file = Path(sample_dir) / filename
    return GGUFReader(sample_file).tensors
````
**EN:** This section defines `get_gguf_sample_tensors`. It retrieves or computes the `GGUF sample tensors` path used by the module.
**CN:** 该部分定义 `get_gguf_sample_tensors`。它负责获取或计算模块中与 `GGUF sample tensors` 相关的处理路径。

### Lines 34-40: `get_gguf_MoE_tensors` definition
````python
def get_gguf_MoE_tensors(
    hidden_size: int, quant_type: GGMLQuantizationType
) -> list[ReaderTensor]:
    sample_dir = GGUF_SAMPLE_MOE
    filename = f"Quant_{quant_type.name}_{hidden_size}.gguf"
    sample_file = Path(sample_dir) / filename
    return GGUFReader(sample_file).tensors
````
**EN:** This section defines `get_gguf_MoE_tensors`. It retrieves or computes the `GGUF Mo E tensors` path used by the module.
**CN:** 该部分定义 `get_gguf_MoE_tensors`。它负责获取或计算模块中与 `GGUF Mo E tensors` 相关的处理路径。

### Lines 43-43: Constants and configuration
````python
DTYPES = [torch.bfloat16]  # [torch.half, torch.bfloat16, torch.float32]
````
**EN:** This block defines shared constants or configuration values such as `DTYPES`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `DTYPES`），供后续函数或控制流程复用。

### Lines 44-45: Comments and local context
````python
# Hidden_size for testing, must match the sample file in HF repo,
# we have `hidden_size = 256, 1024` for test in HF repo currently.
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 46-69: Constants and configuration
````python
HIDDEN_SIZES = [256, 1024]
NUM_TOKENS = [7, 2050]  # Arbitrary values for testing
SEEDS = [0]
QUANT_TYPES = [
    # i-matrix
    GGMLQuantizationType.IQ1_M,
    GGMLQuantizationType.IQ1_S,
    GGMLQuantizationType.IQ2_S,
    GGMLQuantizationType.IQ2_XS,
    GGMLQuantizationType.IQ3_S,
    GGMLQuantizationType.IQ3_XXS,
    GGMLQuantizationType.IQ4_NL,
    GGMLQuantizationType.IQ4_XS,
    # k-quants
    GGMLQuantizationType.Q2_K,
    GGMLQuantizationType.Q3_K,
    GGMLQuantizationType.Q4_K,
    GGMLQuantizationType.Q5_K,
    GGMLQuantizationType.Q6_K,
    # standard quantization
    GGMLQuantizationType.Q4_0,
    GGMLQuantizationType.Q5_0,
    GGMLQuantizationType.Q8_0,
]
````
**EN:** This block defines shared constants or configuration values such as `HIDDEN_SIZES`, `NUM_TOKENS`, `SEEDS`, `QUANT_TYPES`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `HIDDEN_SIZES`, `NUM_TOKENS`, `SEEDS`, `QUANT_TYPES`），供后续函数或控制流程复用。

### Lines 72-91: `test_dequantize` definition
````python
@pytest.mark.parametrize("hidden_size", HIDDEN_SIZES)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("quant_type", QUANT_TYPES)
@torch.inference_mode()
def test_dequantize(
    hidden_size: int, dtype: torch.dtype, quant_type: GGMLQuantizationType
):
    tensors = get_gguf_sample_tensors(hidden_size, quant_type)
    for tensor in tensors:
        shape_str = tensor.name.split("_")[-1]
        shape = map(int, shape_str.split("x"))

        ref_output = torch.tensor(
            dequantize(tensor.data, quant_type), device="cuda"
        ).to(dtype)
        output = ggml_dequantize(
            torch.tensor(tensor.data, device="cuda"), quant_type, *list(shape), dtype
        )

        torch.testing.assert_close(output, ref_output, atol=1e-2, rtol=4e-2)
````
**EN:** This section defines the test `test_dequantize`. It sets up inputs, runs the target path, and checks the expected result. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_dequantize`。它会准备输入、执行目标路径，并检查预期结果。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 94-117: `test_mmvq` definition
````python
@pytest.mark.parametrize("hidden_size", HIDDEN_SIZES)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("quant_type", QUANT_TYPES)
@torch.inference_mode()
def test_mmvq(hidden_size: int, dtype: torch.dtype, quant_type: GGMLQuantizationType):

    tensors = get_gguf_sample_tensors(hidden_size, quant_type)
    x = torch.rand((1, hidden_size), dtype=dtype, device="cuda")
    for tensor in tensors:
        weight = torch.tensor(dequantize(tensor.data, quant_type), device="cuda").to(
            dtype
        )
        ref_output = x @ weight.T

        qweight = torch.tensor(tensor.data, device="cuda")
        output = ggml_mul_mat_vec_a8(qweight, x, quant_type, qweight.shape[0]).to(dtype)

        # NOTE(FlamingoPg): There can be occasional errors, Loosen the granularity of gguf bf16 verification.
        atols = {torch.half: 1, torch.bfloat16: 1.5, torch.float: 1}
        rtols = {torch.half: 1e-1, torch.bfloat16: 3e1, torch.float: 1e-1}

        torch.testing.assert_close(
            output, ref_output, atol=atols[dtype], rtol=rtols[dtype]
        )
````
**EN:** This section defines the test `test_mmvq`. It sets up inputs, runs the target path, and checks the expected result. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_mmvq`。它会准备输入、执行目标路径，并检查预期结果。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 120-163: `test_mmq` definition
````python
@pytest.mark.parametrize("num_tokens", NUM_TOKENS)
@pytest.mark.parametrize("hidden_size", HIDDEN_SIZES)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize(
    "quant_type",
    [
        # k-quants
        GGMLQuantizationType.Q2_K,
        GGMLQuantizationType.Q3_K,
        GGMLQuantizationType.Q4_K,
        GGMLQuantizationType.Q5_K,
        GGMLQuantizationType.Q6_K,
        # standard quants
        GGMLQuantizationType.Q4_0,
        GGMLQuantizationType.Q5_0,
        GGMLQuantizationType.Q8_0,
    ],
)
@torch.inference_mode()
def test_mmq(
    num_tokens: int,
    hidden_size: int,
    dtype: torch.dtype,
    quant_type: GGMLQuantizationType,
):

    tensors = get_gguf_sample_tensors(hidden_size, quant_type)
    x = torch.rand((num_tokens, hidden_size), dtype=dtype, device="cuda")
    for tensor in tensors:
        weight = torch.tensor(dequantize(tensor.data, quant_type), device="cuda").to(
            dtype
        )
        ref_output = x @ weight.T

        qweight = torch.tensor(tensor.data, device="cuda")
        output = ggml_mul_mat_a8(qweight, x, quant_type, qweight.shape[0])
        atols = {torch.half: 1, torch.bfloat16: 1.5, torch.float: 1.2}
        # test matrix has inputs centered around 0 and lower precision from
        # bfloat16 tends to accumulate and can greatly inflate rtol
        # since outputs are also very close to 0
        rtols = {torch.half: 1e-1, torch.bfloat16: 1e4, torch.float: 2e1}
        torch.testing.assert_close(
            output, ref_output, atol=atols[dtype], rtol=rtols[dtype]
        )
````
**EN:** This section defines the test `test_mmq`. It sets up inputs, runs the target path, and checks the expected result. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_mmq`。它会准备输入、执行目标路径，并检查预期结果。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 166-167: Command-line entry point
````python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `get_gguf_sample_tensors`, `get_gguf_MoE_tensors`, `test_dequantize`, `test_mmvq`, `test_mmq`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`
- **External / 外部**: `gguf`, `huggingface_hub`, `numpy`, `pathlib`, `pytest`, `random`, `sys`, `torch`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
