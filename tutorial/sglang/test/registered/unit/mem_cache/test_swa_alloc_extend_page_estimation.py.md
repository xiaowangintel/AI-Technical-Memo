# test_swa_alloc_extend_page_estimation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/mem_cache/test_swa_alloc_extend_page_estimation.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates swa alloc extend page estimation behavior in SGLang's unit / mem cache area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 内存缓存 领域中与 swa alloc extend page estimation 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: supporting statements / 辅助语句
```python
"""Regression for SWA alloc_extend page estimation.

Old gate in SWATokenToKVPoolAllocator.alloc_extend added one full page_size
per request unconditionally, refusing extends that fit inside the request's
last partial page. Fix replaces with get_num_new_pages-based gating.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 8-16: module imports and dependencies / 模块导入与依赖
```python
import unittest
from types import SimpleNamespace
from unittest.mock import MagicMock

import torch

from sglang.srt.mem_cache.swa_memory_pool import SWATokenToKVPoolAllocator
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `types`, `unittest.mock`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `types`, `unittest.mock`, `torch`。

### Lines 18-18: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=2, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 21-36: function make self / 函数 make self
```python
def _make_self(*, page_size: int, full_available: int, swa_available: int):
    full_indices = torch.tensor([10, 11], dtype=torch.int64)
    swa_indices = torch.tensor([20, 21], dtype=torch.int64)
    return SimpleNamespace(
        page_size=page_size,
        full_attn_allocator=SimpleNamespace(
            available_size=lambda: full_available,
            alloc_extend=MagicMock(return_value=full_indices),
        ),
        swa_attn_allocator=SimpleNamespace(
            available_size=lambda: swa_available,
            alloc_extend=MagicMock(return_value=swa_indices),
        ),
        translate_loc_from_full_to_swa=lambda last_loc: last_loc,
        full_to_swa_index_mapping=torch.zeros(64, dtype=torch.int64),
    )
```
**EN:** This block implements `_make_self` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_self`，承担模块行为中的一个聚焦逻辑片段。

### Lines 39-50: function call / 函数 call
```python
def _call(stub, *, prefix_lens_cpu, seq_lens_cpu, extend_num_tokens):
    return SWATokenToKVPoolAllocator.alloc_extend(
        stub,
        prefix_lens=prefix_lens_cpu,
        prefix_lens_cpu=prefix_lens_cpu,
        seq_lens=seq_lens_cpu,
        seq_lens_cpu=seq_lens_cpu,
        last_loc=torch.tensor(
            [int(p) - 1 for p in prefix_lens_cpu.tolist()], dtype=torch.int64
        ),
        extend_num_tokens=extend_num_tokens,
    )
```
**EN:** This block implements `_call` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_call`，承担模块行为中的一个聚焦逻辑片段。

### Lines 53-53: class TestSWAAllocExtendPageEstimation declaration / 类 TestSWAAllocExtendPageEstimation 声明
```python
class TestSWAAllocExtendPageEstimation(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 54-66: test case zero new pages must succeed / 测试用例 zero new pages must succeed
```python
    def test_zero_new_pages_must_succeed(self):
        # Old: 2 + 2*8 = 18 > 16 -> would refuse.
        # New: prefix 5 -> 6 stays in page 0, 0 new pages.
        stub = _make_self(page_size=8, full_available=16, swa_available=16)
        result = _call(
            stub,
            prefix_lens_cpu=torch.tensor([5, 5], dtype=torch.int64),
            seq_lens_cpu=torch.tensor([6, 6], dtype=torch.int64),
            extend_num_tokens=2,
        )
        self.assertIsNotNone(result)
        stub.full_attn_allocator.alloc_extend.assert_called_once()
        stub.swa_attn_allocator.alloc_extend.assert_called_once()
```
**EN:** This test exercises `test_zero_new_pages_must_succeed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_zero_new_pages_must_succeed`。

### Lines 68-77: test case one new page fits / 测试用例 one new page fits
```python
    def test_one_new_page_fits(self):
        # Old: 6 + 2*8 = 22 > 16. New: 2 new pages == 16 // 8.
        stub = _make_self(page_size=8, full_available=16, swa_available=16)
        result = _call(
            stub,
            prefix_lens_cpu=torch.tensor([7, 7], dtype=torch.int64),
            seq_lens_cpu=torch.tensor([10, 10], dtype=torch.int64),
            extend_num_tokens=6,
        )
        self.assertIsNotNone(result)
```
**EN:** This test exercises `test_one_new_page_fits` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_one_new_page_fits`。

### Lines 79-88: test case full pool genuinely insufficient / 测试用例 full pool genuinely insufficient
```python
    def test_full_pool_genuinely_insufficient(self):
        stub = _make_self(page_size=8, full_available=8, swa_available=64)
        result = _call(
            stub,
            prefix_lens_cpu=torch.tensor([8, 8, 8, 8, 8], dtype=torch.int64),
            seq_lens_cpu=torch.tensor([9, 9, 9, 9, 9], dtype=torch.int64),
            extend_num_tokens=5,
        )
        self.assertIsNone(result)
        stub.full_attn_allocator.alloc_extend.assert_not_called()
```
**EN:** This test exercises `test_full_pool_genuinely_insufficient` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_full_pool_genuinely_insufficient`。

### Lines 90-99: test case swa pool genuinely insufficient / 测试用例 swa pool genuinely insufficient
```python
    def test_swa_pool_genuinely_insufficient(self):
        stub = _make_self(page_size=8, full_available=64, swa_available=8)
        result = _call(
            stub,
            prefix_lens_cpu=torch.tensor([8, 8, 8, 8, 8], dtype=torch.int64),
            seq_lens_cpu=torch.tensor([9, 9, 9, 9, 9], dtype=torch.int64),
            extend_num_tokens=5,
        )
        self.assertIsNone(result)
        stub.swa_attn_allocator.alloc_extend.assert_not_called()
```
**EN:** This test exercises `test_swa_pool_genuinely_insufficient` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_swa_pool_genuinely_insufficient`。

### Lines 101-109: test case exactly at capacity succeeds / 测试用例 exactly at capacity succeeds
```python
    def test_exactly_at_capacity_succeeds(self):
        stub = _make_self(page_size=8, full_available=16, swa_available=16)
        result = _call(
            stub,
            prefix_lens_cpu=torch.tensor([8, 8], dtype=torch.int64),
            seq_lens_cpu=torch.tensor([9, 9], dtype=torch.int64),
            extend_num_tokens=2,
        )
        self.assertIsNotNone(result)
```
**EN:** This test exercises `test_exactly_at_capacity_succeeds` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_exactly_at_capacity_succeeds`。

### Lines 111-119: test case one over capacity refuses / 测试用例 one over capacity refuses
```python
    def test_one_over_capacity_refuses(self):
        stub = _make_self(page_size=8, full_available=16, swa_available=16)
        result = _call(
            stub,
            prefix_lens_cpu=torch.tensor([8, 8, 8], dtype=torch.int64),
            seq_lens_cpu=torch.tensor([9, 9, 9], dtype=torch.int64),
            extend_num_tokens=3,
        )
        self.assertIsNone(result)
```
**EN:** This test exercises `test_one_over_capacity_refuses` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_one_over_capacity_refuses`。

### Lines 121-135: test case zero new pages across page sizes / 测试用例 zero new pages across page sizes
```python
    def test_zero_new_pages_across_page_sizes(self):
        # Over-estimation gap grows with page_size; sweep to confirm fix
        # doesn't depend on the page_size=8 numbers above.
        for page_size in (16, 32, 64, 128):
            stub = _make_self(
                page_size=page_size,
                full_available=page_size * 2,
                swa_available=page_size * 2,
            )
            prefix = torch.tensor([page_size - 2] * 4, dtype=torch.int64)
            seq = torch.tensor([page_size - 1] * 4, dtype=torch.int64)
            result = _call(
                stub, prefix_lens_cpu=prefix, seq_lens_cpu=seq, extend_num_tokens=4
            )
            self.assertIsNotNone(result, f"page_size={page_size}")
```
**EN:** This test exercises `test_zero_new_pages_across_page_sizes` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_zero_new_pages_across_page_sizes`。

### Lines 138-139: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_make_self`: This block implements `_make_self` and captures one focused piece of the module's behavior. / 该代码块实现 `_make_self`，承担模块行为中的一个聚焦逻辑片段。
- `_call`: This block implements `_call` and captures one focused piece of the module's behavior. / 该代码块实现 `_call`，承担模块行为中的一个聚焦逻辑片段。
- `TestSWAAllocExtendPageEstimation`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestSWAAllocExtendPageEstimation.test_zero_new_pages_must_succeed`: This test exercises `test_zero_new_pages_must_succeed` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_zero_new_pages_must_succeed`。
- `TestSWAAllocExtendPageEstimation.test_one_new_page_fits`: This test exercises `test_one_new_page_fits` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_one_new_page_fits`。
- `TestSWAAllocExtendPageEstimation.test_full_pool_genuinely_insufficient`: This test exercises `test_full_pool_genuinely_insufficient` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_full_pool_genuinely_insufficient`。
- `TestSWAAllocExtendPageEstimation.test_swa_pool_genuinely_insufficient`: This test exercises `test_swa_pool_genuinely_insufficient` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_swa_pool_genuinely_insufficient`。
- `TestSWAAllocExtendPageEstimation.test_exactly_at_capacity_succeeds`: This test exercises `test_exactly_at_capacity_succeeds` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_exactly_at_capacity_succeeds`。
- `TestSWAAllocExtendPageEstimation.test_one_over_capacity_refuses`: This test exercises `test_one_over_capacity_refuses` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_one_over_capacity_refuses`。
- `TestSWAAllocExtendPageEstimation.test_zero_new_pages_across_page_sizes`: This test exercises `test_zero_new_pages_across_page_sizes` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_zero_new_pages_across_page_sizes`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `types`, `unittest.mock`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.mem_cache.swa_memory_pool`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 139
