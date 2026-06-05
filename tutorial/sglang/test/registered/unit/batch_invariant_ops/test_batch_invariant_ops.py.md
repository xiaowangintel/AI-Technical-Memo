# test_batch_invariant_ops.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/batch_invariant_ops/test_batch_invariant_ops.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates batch invariant ops behavior in SGLang's unit / batch invariant ops area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / batch invariant ops 领域中与 batch invariant ops 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting source context / 辅助源码上下文
```python
# Adapted from https://github.com/thinking-machines-lab/batch_invariant_ops/blob/main/test_batch_invariance.py
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 2-10: module imports and dependencies / 模块导入与依赖
```python
import math
import unittest

import torch

from sglang.srt.batch_invariant_ops import batch_invariant_ops
from sglang.srt.batch_invariant_ops.batch_invariant_ops import set_batch_invariant_mode
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `math`, `unittest`, `torch`, `sglang.srt.batch_invariant_ops`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `math`, `unittest`, `torch`, `sglang.srt.batch_invariant_ops`。

### Lines 11-13: supporting source context / 辅助源码上下文
```python

# Note: MI300 (gfx942) has 64KB shared memory limit but kernel needs 66KB
# MI35x (gfx950/CDNA4) may have different limits - testing on MI35x only
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 14-22: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=10, suite="nightly-1-gpu", nightly=True)
register_amd_ci(est_time=10, suite="nightly-amd-1-gpu-mi35x", nightly=True)

device_type = getattr(torch.accelerator.current_accelerator(), "type", "cpu")
torch.set_default_device(device_type)

# Just to get the logging out of the way
with set_batch_invariant_mode(True):
    pass
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci, getattr.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci, getattr 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 25-25: class TestBatchInvariantOps declaration / 类 TestBatchInvariantOps 声明
```python
class TestBatchInvariantOps(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 26-28: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        batch_invariant_ops._ENABLE_MM_COMPARISON_TEST = True
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 30-32: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        batch_invariant_ops._ENABLE_MM_COMPARISON_TEST = False
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 34-55: method test batch invariance / 方法 test batch invariance
```python
    def _test_batch_invariance(self, M, K, N, dtype):
        """
        Test that matrix operations produce identical results for:
        - Method 1: Matrix-vector multiplication (batch size 1)
        - Method 2: Matrix-matrix multiplication, then slice (full batch)
        """
        a = torch.linspace(-100, 100, M * K, dtype=dtype).reshape(M, K)

        # Create non-contiguous tensor
        b = torch.linspace(-100, 100, K * N, dtype=dtype).reshape(N, K)
        b = b.transpose(0, 1)

        # Method 1: Matrix-vector multiplication (batch size 1)
        out1 = torch.mm(a[:1], b)

        # Method 2: Matrix-matrix multiplication, then slice (full batch)
        out2_pre = torch.mm(a, b)
        out2 = out2_pre[:1]

        # Check if results are identical
        diff = (out1 - out2).abs().max()
        return diff.item()
```
**EN:** Test that matrix operations produce identical results for: - Method 1: Matrix-vector multiplication (batch size 1) - Method 2: Matrix-matrix multiplication, then slice (full batch) This block implements `_test_batch_invariance` and captures one focused piece of the module's behavior.
**CN:** Test that matrix operations produce identical results for: - Method 1: Matrix-vector multiplication (batch size 1) - Method 2: Matrix-matrix multiplication, then slice (full batch) 该代码块实现 `_test_batch_invariance`，承担模块行为中的一个聚焦逻辑片段。

### Lines 57-63: method run multiple iterations / 方法 run multiple iterations
```python
    def _run_multiple_iterations(self, iters, M, K, N, dtype):
        """Run multiple iterations and collect diff statistics"""
        difflist = []
        for _ in range(iters):
            diff = self._test_batch_invariance(M, K, N, dtype)
            difflist.append(diff)
        return difflist
```
**EN:** Run multiple iterations and collect diff statistics This block implements `_run_multiple_iterations` and captures one focused piece of the module's behavior.
**CN:** Run multiple iterations and collect diff statistics 该代码块实现 `_run_multiple_iterations`，承担模块行为中的一个聚焦逻辑片段。

### Lines 65-102: method assert batch invariant results / 方法 assert batch invariant results
```python
    def _assert_batch_invariant_results(self, difflist, dtype, test_name):
        """
        Assert that in batch-invariant mode:
        1. All diffs must not be NaN
        2. All diffs must be exactly 0
        3. Max, min, and diff of diffs must all be 0
        """
        max_diff = max(difflist)
        min_diff = min(difflist)
        diff_range = max_diff - min_diff

        # Check for NaN values
        self.assertFalse(
            math.isnan(max_diff), f"{test_name}: max_diff is NaN for {dtype}"
        )
        self.assertFalse(
            math.isnan(min_diff), f"{test_name}: min_diff is NaN for {dtype}"
        )
        self.assertFalse(
            math.isnan(diff_range), f"{test_name}: diff_range is NaN for {dtype}"
        )

        # Check that all diffs are exactly 0
        self.assertEqual(
            max_diff,
            0.0,
            f"{test_name}: max_diff must be 0 in batch-invariant mode, got {max_diff} for {dtype}",
        )
        self.assertEqual(
            min_diff,
            0.0,
            f"{test_name}: min_diff must be 0 in batch-invariant mode, got {min_diff} for {dtype}",
        )
        self.assertEqual(
            diff_range,
            0.0,
            f"{test_name}: diff_range must be 0 in batch-invariant mode, got {diff_range} for {dtype}",
        )
```
**EN:** Assert that in batch-invariant mode: 1. This block implements `_assert_batch_invariant_results` and captures one focused piece of the module's behavior.
**CN:** Assert that in batch-invariant mode: 1. 该代码块实现 `_assert_batch_invariant_results`，承担模块行为中的一个聚焦逻辑片段。

### Lines 104-121: test case small matrices / 测试用例 small matrices
```python
    def test_small_matrices(self):
        """Test batch invariance with small matrix sizes"""
        test_cases = [
            ("Small-1", 8, 64, 128),
            ("Small-2", 16, 128, 256),
            ("Small-3", 4, 32, 64),
        ]

        for name, M, K, N in test_cases:
            with self.subTest(name=name, M=M, K=K, N=N):
                for dtype in [torch.float32, torch.bfloat16]:
                    with self.subTest(dtype=dtype):
                        # Run with batch-invariant mode
                        with set_batch_invariant_mode(True):
                            difflist = self._run_multiple_iterations(
                                iters=5, M=M, K=K, N=N, dtype=dtype
                            )
                            self._assert_batch_invariant_results(difflist, dtype, name)
```
**EN:** Test batch invariance with small matrix sizes This test exercises `test_small_matrices` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test batch invariance with small matrix sizes 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_small_matrices`。

### Lines 123-140: test case medium matrices / 测试用例 medium matrices
```python
    def test_medium_matrices(self):
        """Test batch invariance with medium matrix sizes"""
        test_cases = [
            ("Medium-1", 32, 128, 1024),
            ("Medium-2", 64, 512, 2048),
            ("Medium-3", 24, 192, 768),
        ]

        for name, M, K, N in test_cases:
            with self.subTest(name=name, M=M, K=K, N=N):
                for dtype in [torch.float32, torch.bfloat16]:
                    with self.subTest(dtype=dtype):
                        # Run with batch-invariant mode
                        with set_batch_invariant_mode(True):
                            difflist = self._run_multiple_iterations(
                                iters=5, M=M, K=K, N=N, dtype=dtype
                            )
                            self._assert_batch_invariant_results(difflist, dtype, name)
```
**EN:** Test batch invariance with medium matrix sizes This test exercises `test_medium_matrices` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test batch invariance with medium matrix sizes 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_medium_matrices`。

### Lines 142-159: test case large matrices / 测试用例 large matrices
```python
    def test_large_matrices(self):
        """Test batch invariance with large matrix sizes"""
        test_cases = [
            ("Large-1", 128, 1024, 4096),
            ("Large-2", 256, 2048, 8192),
            ("Large-3", 96, 768, 3072),
        ]

        for name, M, K, N in test_cases:
            with self.subTest(name=name, M=M, K=K, N=N):
                for dtype in [torch.float32, torch.bfloat16]:
                    with self.subTest(dtype=dtype):
                        # Run with batch-invariant mode
                        with set_batch_invariant_mode(True):
                            difflist = self._run_multiple_iterations(
                                iters=5, M=M, K=K, N=N, dtype=dtype
                            )
                            self._assert_batch_invariant_results(difflist, dtype, name)
```
**EN:** Test batch invariance with large matrix sizes This test exercises `test_large_matrices` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test batch invariance with large matrix sizes 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_large_matrices`。

### Lines 161-174: test case without batch invariant mode / 测试用例 without batch invariant mode
```python
    def test_without_batch_invariant_mode(self):
        """
        Test that without batch-invariant mode, results may differ.
        This test demonstrates the difference batch-invariant mode makes.
        """
        M, K, N = 32, 128, 1024
        dtype = torch.float32

        # Run without batch-invariant mode
        with set_batch_invariant_mode(False):
            difflist = self._run_multiple_iterations(
                iters=5, M=M, K=K, N=N, dtype=dtype
            )
            print(f"Without batch-invariant mode, we get diffs: {difflist}")
```
**EN:** Test that without batch-invariant mode, results may differ. This test exercises `test_without_batch_invariant_mode` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that without batch-invariant mode, results may differ. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_without_batch_invariant_mode`。

### Lines 176-195: method test bmm batch invariance / 方法 test bmm batch invariance
```python
    def _test_bmm_batch_invariance(self, B, M, K, N, dtype):
        """
        Test that BMM operations produce identical results for:
        - Method 1: BMM with subset of batches
        - Method 2: BMM with all batches, then slice
        """
        a = torch.linspace(-100, 100, B * M * K, dtype=dtype).reshape(B, M, K)
        b = torch.linspace(-100, 100, B * K * N, dtype=dtype).reshape(B, K, N)

        # Method 1: BMM with subset (first 2 batches)
        subset_size = min(2, B)
        out1 = torch.bmm(a[:subset_size], b[:subset_size])

        # Method 2: BMM with all batches, then slice
        out2_pre = torch.bmm(a, b)
        out2 = out2_pre[:subset_size]

        # Check if results are identical
        diff = (out1 - out2).abs().max()
        return diff.item()
```
**EN:** Test that BMM operations produce identical results for: - Method 1: BMM with subset of batches - Method 2: BMM with all batches, then slice This block implements `_test_bmm_batch_invariance` and captures one focused piece of the module's behavior.
**CN:** Test that BMM operations produce identical results for: - Method 1: BMM with subset of batches - Method 2: BMM with all batches, then slice 该代码块实现 `_test_bmm_batch_invariance`，承担模块行为中的一个聚焦逻辑片段。

### Lines 197-203: method run bmm multiple iterations / 方法 run bmm multiple iterations
```python
    def _run_bmm_multiple_iterations(self, iters, B, M, K, N, dtype):
        """Run multiple BMM iterations and collect diff statistics"""
        difflist = []
        for _ in range(iters):
            diff = self._test_bmm_batch_invariance(B, M, K, N, dtype)
            difflist.append(diff)
        return difflist
```
**EN:** Run multiple BMM iterations and collect diff statistics This block implements `_run_bmm_multiple_iterations` and captures one focused piece of the module's behavior.
**CN:** Run multiple BMM iterations and collect diff statistics 该代码块实现 `_run_bmm_multiple_iterations`，承担模块行为中的一个聚焦逻辑片段。

### Lines 205-222: test case bmm small matrices / 测试用例 bmm small matrices
```python
    def test_bmm_small_matrices(self):
        """Test BMM batch invariance with small matrix sizes"""
        test_cases = [
            ("BMM-Small-1", 4, 8, 64, 128),
            ("BMM-Small-2", 8, 16, 128, 256),
            ("BMM-Small-3", 6, 4, 32, 64),
        ]

        for name, B, M, K, N in test_cases:
            with self.subTest(name=name, B=B, M=M, K=K, N=N):
                for dtype in [torch.float32, torch.bfloat16]:
                    with self.subTest(dtype=dtype):
                        # Run with batch-invariant mode
                        with set_batch_invariant_mode(True):
                            difflist = self._run_bmm_multiple_iterations(
                                iters=5, B=B, M=M, K=K, N=N, dtype=dtype
                            )
                            self._assert_batch_invariant_results(difflist, dtype, name)
```
**EN:** Test BMM batch invariance with small matrix sizes This test exercises `test_bmm_small_matrices` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test BMM batch invariance with small matrix sizes 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bmm_small_matrices`。

### Lines 224-241: test case bmm medium matrices / 测试用例 bmm medium matrices
```python
    def test_bmm_medium_matrices(self):
        """Test BMM batch invariance with medium matrix sizes"""
        test_cases = [
            ("BMM-Medium-1", 8, 32, 128, 1024),
            ("BMM-Medium-2", 16, 64, 512, 2048),
            ("BMM-Medium-3", 12, 24, 192, 768),
        ]

        for name, B, M, K, N in test_cases:
            with self.subTest(name=name, B=B, M=M, K=K, N=N):
                for dtype in [torch.float32, torch.bfloat16]:
                    with self.subTest(dtype=dtype):
                        # Run with batch-invariant mode
                        with set_batch_invariant_mode(True):
                            difflist = self._run_bmm_multiple_iterations(
                                iters=5, B=B, M=M, K=K, N=N, dtype=dtype
                            )
                            self._assert_batch_invariant_results(difflist, dtype, name)
```
**EN:** Test BMM batch invariance with medium matrix sizes This test exercises `test_bmm_medium_matrices` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test BMM batch invariance with medium matrix sizes 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bmm_medium_matrices`。

### Lines 243-260: test case bmm large matrices / 测试用例 bmm large matrices
```python
    def test_bmm_large_matrices(self):
        """Test BMM batch invariance with large matrix sizes"""
        test_cases = [
            ("BMM-Large-1", 16, 128, 1024, 4096),
            ("BMM-Large-2", 32, 256, 2048, 8192),
            ("BMM-Large-3", 24, 96, 768, 3072),
        ]

        for name, B, M, K, N in test_cases:
            with self.subTest(name=name, B=B, M=M, K=K, N=N):
                for dtype in [torch.float32, torch.bfloat16]:
                    with self.subTest(dtype=dtype):
                        # Run with batch-invariant mode
                        with set_batch_invariant_mode(True):
                            difflist = self._run_bmm_multiple_iterations(
                                iters=5, B=B, M=M, K=K, N=N, dtype=dtype
                            )
                            self._assert_batch_invariant_results(difflist, dtype, name)
```
**EN:** Test BMM batch invariance with large matrix sizes This test exercises `test_bmm_large_matrices` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test BMM batch invariance with large matrix sizes 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bmm_large_matrices`。

### Lines 263-264: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestBatchInvariantOps`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestBatchInvariantOps.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestBatchInvariantOps.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestBatchInvariantOps._test_batch_invariance`: Test that matrix operations produce identical results for: - Method 1: Matrix-vector multiplication (batch size 1) - Method 2: Matrix-matrix multiplication, then slice (full batch) / 该代码块实现 `_test_batch_invariance`，承担模块行为中的一个聚焦逻辑片段。
- `TestBatchInvariantOps._run_multiple_iterations`: Run multiple iterations and collect diff statistics / 该代码块实现 `_run_multiple_iterations`，承担模块行为中的一个聚焦逻辑片段。
- `TestBatchInvariantOps._assert_batch_invariant_results`: Assert that in batch-invariant mode: 1. / 该代码块实现 `_assert_batch_invariant_results`，承担模块行为中的一个聚焦逻辑片段。
- `TestBatchInvariantOps.test_small_matrices`: Test batch invariance with small matrix sizes / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_small_matrices`。
- `TestBatchInvariantOps.test_medium_matrices`: Test batch invariance with medium matrix sizes / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_medium_matrices`。
- `TestBatchInvariantOps.test_large_matrices`: Test batch invariance with large matrix sizes / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_large_matrices`。
- `TestBatchInvariantOps.test_without_batch_invariant_mode`: Test that without batch-invariant mode, results may differ. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_without_batch_invariant_mode`。
- `TestBatchInvariantOps._test_bmm_batch_invariance`: Test that BMM operations produce identical results for: - Method 1: BMM with subset of batches - Method 2: BMM with all batches, then slice / 该代码块实现 `_test_bmm_batch_invariance`，承担模块行为中的一个聚焦逻辑片段。
- `TestBatchInvariantOps._run_bmm_multiple_iterations`: Run multiple BMM iterations and collect diff statistics / 该代码块实现 `_run_bmm_multiple_iterations`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `math`, `unittest`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.batch_invariant_ops`, `sglang.srt.batch_invariant_ops.batch_invariant_ops`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 264
