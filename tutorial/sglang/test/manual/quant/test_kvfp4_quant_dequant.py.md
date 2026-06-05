# test_kvfp4_quant_dequant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/quant/test_kvfp4_quant_dequant.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `kvfp4 quant dequant` scenario in `test/manual/quant`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual/quant` 中的 `kvfp4 quant dequant` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Imports and shared helpers / 导入与共享辅助项
```python
#!/usr/bin/env python3

import time

import numpy as np
import pytest
import torch

from sglang.srt.layers.quantization.kvfp4_tensor import KVFP4QuantizeUtil
```
**EN:** This range imports `time`, `numpy`, `pytest` and `torch`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。

### Lines 12-28: Helper routines around calculate_accuracy_metrics / 辅助例程
```python
def calculate_accuracy_metrics(
    original: torch.Tensor, reconstructed: torch.Tensor
) -> dict[str, float]:
    """Calculate accuracy metrics between original and reconstructed tensors."""
    mse = torch.mean((original - reconstructed) ** 2).item()
    mae = torch.mean(torch.abs(original - reconstructed)).item()

    # PSNR calculation
    max_val = torch.max(torch.abs(original)).item()
    psnr = 20 * np.log10(max_val / np.sqrt(mse)) if mse > 0 else float("inf")

    # Relative error
    rel_error = torch.mean(
        torch.abs(original - reconstructed) / (torch.abs(original) + 1e-8)
    ).item()

    return {"MSE": mse, "MAE": mae, "PSNR": psnr, "Relative Error": rel_error}
```
**EN:** This range implements helper routine(s) `calculate_accuracy_metrics` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `mean`, `item`, `abs` and `log10`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 31-55: Helper routines around run_benchmark / 辅助例程
```python
def run_benchmark(m, n, k, num_runs=100) -> dict[str, dict[str, float]]:
    """Run FP8 vs KVFP4 quantization benchmark and return metrics."""
    tensor_bf16 = torch.randn(m, n, k, dtype=torch.bfloat16, device="cuda")

    # --- FP8 ---
    for _ in range(3):  # warmup
        _ = tensor_bf16 * 2
    torch.cuda.synchronize()

    start = time.time()
    for _ in range(num_runs):
        tensor_fp8 = tensor_bf16.to(torch.float8_e4m3fn)
    torch.cuda.synchronize()
    fp8_quant_time = (time.time() - start) / num_runs

    start = time.time()
    for _ in range(num_runs):
        tensor_fp8_dequant = tensor_fp8.to(torch.bfloat16)
    torch.cuda.synchronize()
    fp8_dequant_time = (time.time() - start) / num_runs

    fp8_metrics = calculate_accuracy_metrics(tensor_bf16, tensor_fp8_dequant)

    # --- KVFP4 ---
    tensor_fp4, scale_factors = KVFP4QuantizeUtil.batched_quantize(tensor_bf16)
```
**EN:** This range implements helper routine(s) `run_benchmark` so setup, transformation, or validation logic can be reused cleanly. The code also ties this block to evaluation or benchmark execution. Representative call sites include `randn`, `synchronize`, `time` and `to`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 56-80: Scenario logic / 场景逻辑
```python
    _ = KVFP4QuantizeUtil.batched_dequantize(tensor_fp4, scale_factors)

    start = time.time()
    for _ in range(num_runs):
        tensor_fp4, scale_factors = KVFP4QuantizeUtil.batched_quantize(tensor_bf16)
    torch.cuda.synchronize()
    fp4_quant_time = (time.time() - start) / num_runs

    start = time.time()
    for _ in range(num_runs):
        tensor_fp4_dequant = KVFP4QuantizeUtil.batched_dequantize(
            tensor_fp4, scale_factors
        )
    torch.cuda.synchronize()
    fp4_dequant_time = (time.time() - start) / num_runs

    fp4_metrics = calculate_accuracy_metrics(tensor_bf16, tensor_fp4_dequant)

    return {
        "fp8": {
            "quant_time": fp8_quant_time,
            "dequant_time": fp8_dequant_time,
            **fp8_metrics,
        },
        "fp4": {
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `batched_dequantize`, `time`, `batched_quantize` and `synchronize`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 81-85: Scenario logic / 场景逻辑
```python
            "quant_time": fp4_quant_time,
            "dequant_time": fp4_dequant_time,
            **fp4_metrics,
        },
    }
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 86-105: Constants and scenario settings / 常量与场景配置
```python


# default tensor shapes (m, n, k)
# [M, 1, 576]: DeepSeekR1-FP4 MLA
# [M, 8, 64]: gpt-oss-20b MHA
MNK_FACTORS = [
    (64, 1, 576),
    (512, 1, 576),
    (1024, 1, 576),
    (4096, 1, 576),
    (2868672, 1, 576),
    (64, 8, 64),
    (512, 8, 64),
    (1024, 8, 64),
    (4096, 8, 64),
    (2868672, 8, 64),
]


@pytest.mark.parametrize("m,n,k", MNK_FACTORS)
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. Representative call sites include `shapes` and `parametrize`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 106-116: Test routines around test_kvfp4_quant_dequant / 测试例程
```python
def test_kvfp4_quant_dequant(m, n, k):
    """Benchmark FP8 vs KVFP4 for predefined tensor shapes."""
    print(f"\n=== Running benchmark for tensor shape: [{m}, {n}, {k}] ===")
    results = run_benchmark(m, n, k)

    print("FP8:", results["fp8"])
    print("FP4:", results["fp4"])

    # Basic assertions to make sure metrics are reasonable
    assert results["fp4"]["MSE"] < 1.0
    assert results["fp8"]["MSE"] < 1.0
```
**EN:** This range defines concrete test routine(s) `test_kvfp4_quant_dequant`. The logic drives the target scenario and encodes the expected acceptance criteria. The code also ties this block to evaluation or benchmark execution. Assertions in this block enforce the intended outcome. Representative call sites include `run_benchmark`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Pytest markers / Pytest 标记
- Accuracy evaluation / 精度评测
- Performance benchmarking / 性能基准测试
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `time`
- **Third-party / 第三方库**: `numpy`, `pytest`, `torch`
- **Project Modules / 项目模块**: `sglang.srt.layers.quantization.kvfp4_tensor`
