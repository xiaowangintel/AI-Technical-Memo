# test_awq_dequant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/quant/test_awq_dequant.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates awq dequant behavior in SGLang's quant area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 quant 领域中与 awq dequant 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: supporting source context / 辅助源码上下文
```python
# Adapted from https://github.com/vllm-project/vllm/blob/main/tests/kernels/quantization/test_awq_triton.py
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 4-9: supporting statements / 辅助语句
```python
"""
unittest version of the AWQ Triton kernel tests.

Run with:
    python -m unittest test_awq_dequant.py
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 11-22: module imports and dependencies / 模块导入与依赖
```python
import unittest

import torch

from sglang.srt.layers.quantization.awq.awq_triton import (
    AWQ_TRITON_SUPPORTED_GROUP_SIZES,
    awq_dequantize_triton,
    awq_gemm_triton,
)
from sglang.srt.utils import get_device
from sglang.test.ci.ci_register import register_amd_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `torch`, `sglang.srt.layers.quantization.awq.awq_triton`, `sglang.srt.utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `torch`, `sglang.srt.layers.quantization.awq.awq_triton`, `sglang.srt.utils`。

### Lines 24-26: CI registration and metadata / CI 注册与元数据
```python
register_amd_ci(est_time=2, suite="stage-a-test-1-gpu-small-amd")

device = get_device()
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_amd_ci, get_device.
**CN:** 该代码块通过 register_amd_ci, get_device 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 29-34: function reverse awq order / 函数 reverse awq order
```python
def reverse_awq_order(t: torch.Tensor) -> torch.Tensor:
    bits = 4
    AWQ_REVERSE_ORDER = [0, 4, 1, 5, 2, 6, 3, 7]
    idx = torch.arange(t.shape[-1], dtype=torch.int32, device=t.device)
    idx = idx.view(-1, 32 // bits)[:, AWQ_REVERSE_ORDER].view(-1)
    return (t[:, idx] & 0xF).contiguous()
```
**EN:** This block implements `reverse_awq_order` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `reverse_awq_order`，承担模块行为中的一个聚焦逻辑片段。

### Lines 37-64: function awq dequantize torch / 函数 awq dequantize torch
```python
def awq_dequantize_torch(
    qweight: torch.Tensor,
    scales: torch.Tensor,
    qzeros: torch.Tensor,
    group_size: int,
) -> torch.Tensor:
    if group_size == -1:
        group_size = qweight.shape[0]

    bits = 4
    shifts = torch.arange(0, 32, bits, device=qzeros.device)

    iweights = torch.bitwise_right_shift(qweight[:, :, None], shifts[None, None, :]).to(
        torch.int8
    )
    iweights = reverse_awq_order(iweights.view(iweights.shape[0], -1))

    zeros = torch.bitwise_right_shift(qzeros[:, :, None], shifts[None, None, :]).to(
        torch.int8
    )
    zeros = reverse_awq_order(zeros.view(qzeros.shape[0], -1))

    iweights = torch.bitwise_and(iweights, (2**bits) - 1)
    zeros = torch.bitwise_and(zeros, (2**bits) - 1)

    scales = scales.repeat_interleave(group_size, dim=0)
    zeros = zeros.repeat_interleave(group_size, dim=0)
    return (iweights - zeros) * scales
```
**EN:** This block implements `awq_dequantize_torch` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `awq_dequantize_torch`，承担模块行为中的一个聚焦逻辑片段。

### Lines 67-67: class TestAWQTriton declaration / 类 TestAWQTriton 声明
```python
class TestAWQTriton(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 68-82: test case dequantize / 测试用例 dequantize
```python
    def test_dequantize(self):
        rows_list = [3584, 18944, 128, 256, 512, 1024]
        cols_list = [448, 576, 4736, 16, 32, 64, 128]

        for qweight_rows in rows_list:
            for qweight_cols in cols_list:
                for group_size in AWQ_TRITON_SUPPORTED_GROUP_SIZES:
                    with self.subTest(
                        rows=qweight_rows, cols=qweight_cols, g=group_size
                    ):
                        self._run_dequant_case(
                            qweight_rows=qweight_rows,
                            qweight_cols=qweight_cols,
                            group_size=group_size,
                        )
```
**EN:** This test exercises `test_dequantize` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dequantize`。

### Lines 84-116: method run dequant case / 方法 run dequant case
```python
    def _run_dequant_case(self, qweight_rows, qweight_cols, group_size):
        if group_size == -1:
            group_size = qweight_rows

        torch.manual_seed(0)

        qweight = torch.randint(
            0,
            torch.iinfo(torch.int32).max,
            (qweight_rows, qweight_cols),
            dtype=torch.int32,
            device=device,
        )
        scales = torch.rand(
            qweight_rows // group_size,
            qweight_cols * 8,
            dtype=torch.float16,
            device=device,
        )
        zeros = torch.randint(
            0,
            torch.iinfo(torch.int32).max,
            (qweight_rows // group_size, qweight_cols),
            dtype=torch.int32,
            device=device,
        )

        ref = awq_dequantize_torch(qweight, scales, zeros, group_size)
        tri = awq_dequantize_triton(qweight, scales, zeros)

        # sanity
        self.assertFalse(torch.any(torch.isinf(tri)) or torch.any(torch.isnan(tri)))
        torch.testing.assert_close(ref, tri)
```
**EN:** This block implements `_run_dequant_case` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_dequant_case`，承担模块行为中的一个聚焦逻辑片段。

### Lines 117-118: supporting source context / 辅助源码上下文
```python

    # GEMM
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 119-137: test case gemm / 测试用例 gemm
```python
    def test_gemm(self):
        N_list = [1, 2, 4, 8, 14, 17, 23, 32]
        K_list = [128]
        M_list = [16, 24, 32]
        splitK_list = [1, 8]

        for N in N_list:
            for K in K_list:
                for M in M_list:
                    for group_size in AWQ_TRITON_SUPPORTED_GROUP_SIZES:
                        for splitK in splitK_list:
                            with self.subTest(N=N, K=K, M=M, g=group_size, sk=splitK):
                                self._run_gemm_case(
                                    N=N,
                                    K=K,
                                    M=M,
                                    group_size=group_size,
                                    splitK=splitK,
                                )
```
**EN:** This test exercises `test_gemm` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gemm`。

### Lines 139-176: method run gemm case / 方法 run gemm case
```python
    def _run_gemm_case(self, N, K, M, group_size, splitK):
        if group_size == -1:
            group_size = K

        torch.manual_seed(0)

        x = torch.rand((N, K), dtype=torch.float32, device=device)
        qweight = torch.randint(
            0,
            torch.iinfo(torch.int32).max,
            (K, M // 8),
            dtype=torch.int32,
            device=device,
        )
        qzeros = torch.randint(
            0,
            torch.iinfo(torch.int32).max,
            (K // group_size, M // 8),
            dtype=torch.int32,
            device=device,
        )
        scales = torch.rand((K // group_size, M), dtype=torch.float32, device=device)

        tri_out = awq_gemm_triton(x, qweight, scales, qzeros, splitK)

        self.assertFalse(
            torch.any(torch.isinf(tri_out)) or torch.any(torch.isnan(tri_out))
        )

        # dequantize & compare
        w_deq = awq_dequantize_triton(qweight, scales, qzeros)
        ref_out = torch.matmul(x, w_deq)

        self.assertFalse(
            torch.any(torch.isinf(ref_out)) or torch.any(torch.isnan(ref_out))
        )

        torch.testing.assert_close(tri_out.cpu(), ref_out.cpu(), atol=1e-1, rtol=1e-1)
```
**EN:** This block implements `_run_gemm_case` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_gemm_case`，承担模块行为中的一个聚焦逻辑片段。

### Lines 179-180: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=2)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `reverse_awq_order`: This block implements `reverse_awq_order` and captures one focused piece of the module's behavior. / 该代码块实现 `reverse_awq_order`，承担模块行为中的一个聚焦逻辑片段。
- `awq_dequantize_torch`: This block implements `awq_dequantize_torch` and captures one focused piece of the module's behavior. / 该代码块实现 `awq_dequantize_torch`，承担模块行为中的一个聚焦逻辑片段。
- `TestAWQTriton`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestAWQTriton.test_dequantize`: This test exercises `test_dequantize` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dequantize`。
- `TestAWQTriton._run_dequant_case`: This block implements `_run_dequant_case` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_dequant_case`，承担模块行为中的一个聚焦逻辑片段。
- `TestAWQTriton.test_gemm`: This test exercises `test_gemm` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gemm`。
- `TestAWQTriton._run_gemm_case`: This block implements `_run_gemm_case` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_gemm_case`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.layers.quantization.awq.awq_triton`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 180
