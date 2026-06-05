# test_token_filter_ops.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/constrained/test_token_filter_ops.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates token filter ops behavior in SGLang's unit / constrained area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / constrained 领域中与 token filter ops 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: supporting statements / 辅助语句
```python
"""
Unit tests for token filter operations (Triton and Torch paths).

Verifies that both implementations produce identical bitmask output
for the same inputs, ensuring parity across GPU and CPU paths.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 8-15: module imports and dependencies / 模块导入与依赖
```python
import unittest

import torch

from sglang.srt.constrained.torch_ops.token_filter_torch_ops import (
    set_token_filter_torch,
)
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `torch`, `sglang.srt.constrained.torch_ops.token_filter_torch_ops`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `torch`, `sglang.srt.constrained.torch_ops.token_filter_torch_ops`, `sglang.test.ci.ci_register`。

### Lines 17-24: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(2.0, "base-a-test-cpu")

# Conditionally import Triton path
_has_cuda = torch.cuda.is_available()
if _has_cuda:
    from sglang.srt.constrained.triton_ops.token_filter_ops import (
        set_token_filter_triton,
    )
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci, is_available.
**CN:** 该代码块通过 register_cpu_ci, is_available 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 27-36: function get allowed tokens / 函数 get allowed tokens
```python
def _get_allowed_tokens(vocab_mask, batch_idx, max_token_id):
    """Extract allowed token IDs from a bitmask row."""
    allowed = []
    for token_id in range(max_token_id):
        elem = token_id // 32
        bit = token_id % 32
        val = int(vocab_mask[batch_idx, elem].item())
        if val & (1 << bit):
            allowed.append(token_id)
    return allowed
```
**EN:** Extract allowed token IDs from a bitmask row. This block implements `_get_allowed_tokens` and captures one focused piece of the module's behavior.
**CN:** Extract allowed token IDs from a bitmask row. 该代码块实现 `_get_allowed_tokens`，承担模块行为中的一个聚焦逻辑片段。

### Lines 39-39: class TestSetTokenFilterTorch declaration / 类 TestSetTokenFilterTorch 声明
```python
class TestSetTokenFilterTorch(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 40-40: supporting statements / 辅助语句
```python
    """Tests for the Torch token filter implementation."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 42-47: test case allow tokens from blank mask / 测试用例 allow tokens from blank mask
```python
    def test_allow_tokens_from_blank_mask(self):
        vocab_mask = torch.zeros((1, 4), dtype=torch.int32)  # 128 tokens
        set_token_filter_torch(vocab_mask, [0, 5, 31, 32, 63], 0, is_allowed=True)

        allowed = _get_allowed_tokens(vocab_mask, 0, 64)
        self.assertEqual(allowed, [0, 5, 31, 32, 63])
```
**EN:** This test exercises `test_allow_tokens_from_blank_mask` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_allow_tokens_from_blank_mask`。

### Lines 49-59: test case block tokens from full mask / 测试用例 block tokens from full mask
```python
    def test_block_tokens_from_full_mask(self):
        vocab_mask = torch.full((1, 4), -1, dtype=torch.int32)  # all bits set
        set_token_filter_torch(
            vocab_mask, [3, 5], 0, is_allowed=False, reset_vocab_mask=False
        )

        allowed = _get_allowed_tokens(vocab_mask, 0, 64)
        self.assertNotIn(3, allowed)
        self.assertNotIn(5, allowed)
        self.assertIn(0, allowed)
        self.assertIn(1, allowed)
```
**EN:** This test exercises `test_block_tokens_from_full_mask` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_block_tokens_from_full_mask`。

### Lines 61-68: test case reset then allow / 测试用例 reset then allow
```python
    def test_reset_then_allow(self):
        vocab_mask = torch.full((1, 2), -1, dtype=torch.int32)
        set_token_filter_torch(
            vocab_mask, [7], 0, is_allowed=True, reset_vocab_mask=True
        )

        allowed = _get_allowed_tokens(vocab_mask, 0, 64)
        self.assertEqual(allowed, [7])
```
**EN:** This test exercises `test_reset_then_allow` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reset_then_allow`。

### Lines 70-81: test case reset then block / 测试用例 reset then block
```python
    def test_reset_then_block(self):
        vocab_mask = torch.zeros((1, 2), dtype=torch.int32)
        set_token_filter_torch(
            vocab_mask, [3, 5], 0, is_allowed=False, reset_vocab_mask=True
        )

        allowed = _get_allowed_tokens(vocab_mask, 0, 64)
        self.assertNotIn(3, allowed)
        self.assertNotIn(5, allowed)
        # All other tokens should be allowed (reset to -1 for block mode)
        self.assertIn(0, allowed)
        self.assertIn(7, allowed)
```
**EN:** This test exercises `test_reset_then_block` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reset_then_block`。

### Lines 83-90: test case empty token list / 测试用例 empty token list
```python
    def test_empty_token_list(self):
        vocab_mask = torch.zeros((1, 2), dtype=torch.int32)
        set_token_filter_torch(
            vocab_mask, [], 0, is_allowed=True, reset_vocab_mask=True
        )

        allowed = _get_allowed_tokens(vocab_mask, 0, 64)
        self.assertEqual(allowed, [])
```
**EN:** This test exercises `test_empty_token_list` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_token_list`。

### Lines 92-100: test case batch indexing / 测试用例 batch indexing
```python
    def test_batch_indexing(self):
        vocab_mask = torch.zeros((3, 2), dtype=torch.int32)
        set_token_filter_torch(vocab_mask, [1], 0, is_allowed=True)
        set_token_filter_torch(vocab_mask, [2], 1, is_allowed=True)
        set_token_filter_torch(vocab_mask, [3], 2, is_allowed=True)

        self.assertEqual(_get_allowed_tokens(vocab_mask, 0, 64), [1])
        self.assertEqual(_get_allowed_tokens(vocab_mask, 1, 64), [2])
        self.assertEqual(_get_allowed_tokens(vocab_mask, 2, 64), [3])
```
**EN:** This test exercises `test_batch_indexing` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_batch_indexing`。

### Lines 103-104: class TestTritonTorchParity declaration / 类 TestTritonTorchParity 声明
```python
@unittest.skipUnless(_has_cuda, "CUDA not available")
class TestTritonTorchParity(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 105-105: supporting statements / 辅助语句
```python
    """Tests that Triton and Torch produce identical output."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 107-133: method compare outputs / 方法 compare outputs
```python
    def _compare_outputs(self, token_ids, is_allowed, reset):
        vocab_size = 128
        num_elements = (vocab_size + 31) // 32

        torch_mask = torch.zeros((1, num_elements), dtype=torch.int32)
        triton_mask = torch.zeros((1, num_elements), dtype=torch.int32, device="cuda")

        set_token_filter_torch(
            torch_mask,
            token_ids,
            0,
            is_allowed=is_allowed,
            reset_vocab_mask=reset,
        )
        set_token_filter_triton(
            triton_mask,
            token_ids,
            0,
            is_allowed=is_allowed,
            reset_vocab_mask=reset,
        )

        triton_cpu = triton_mask.cpu()
        self.assertTrue(
            torch.equal(torch_mask, triton_cpu),
            f"Mismatch: torch={torch_mask} triton={triton_cpu}",
        )
```
**EN:** This block implements `_compare_outputs` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_compare_outputs`，承担模块行为中的一个聚焦逻辑片段。

### Lines 135-136: test case parity allow tokens / 测试用例 parity allow tokens
```python
    def test_parity_allow_tokens(self):
        self._compare_outputs([0, 5, 31, 32, 63, 100], is_allowed=True, reset=True)
```
**EN:** This test exercises `test_parity_allow_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parity_allow_tokens`。

### Lines 138-139: test case parity block tokens / 测试用例 parity block tokens
```python
    def test_parity_block_tokens(self):
        self._compare_outputs([3, 5, 10], is_allowed=False, reset=True)
```
**EN:** This test exercises `test_parity_block_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parity_block_tokens`。

### Lines 141-142: test case parity empty tokens / 测试用例 parity empty tokens
```python
    def test_parity_empty_tokens(self):
        self._compare_outputs([], is_allowed=True, reset=True)
```
**EN:** This test exercises `test_parity_empty_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parity_empty_tokens`。

### Lines 145-146: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_get_allowed_tokens`: Extract allowed token IDs from a bitmask row. / 该代码块实现 `_get_allowed_tokens`，承担模块行为中的一个聚焦逻辑片段。
- `TestSetTokenFilterTorch`: Tests for the Torch token filter implementation. / 用于组织相关测试、夹具或辅助方法。
- `TestTritonTorchParity`: Tests that Triton and Torch produce identical output. / 用于组织相关测试、夹具或辅助方法。
- `TestSetTokenFilterTorch.test_allow_tokens_from_blank_mask`: This test exercises `test_allow_tokens_from_blank_mask` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_allow_tokens_from_blank_mask`。
- `TestSetTokenFilterTorch.test_block_tokens_from_full_mask`: This test exercises `test_block_tokens_from_full_mask` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_block_tokens_from_full_mask`。
- `TestSetTokenFilterTorch.test_reset_then_allow`: This test exercises `test_reset_then_allow` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reset_then_allow`。
- `TestSetTokenFilterTorch.test_reset_then_block`: This test exercises `test_reset_then_block` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reset_then_block`。
- `TestSetTokenFilterTorch.test_empty_token_list`: This test exercises `test_empty_token_list` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_token_list`。
- `TestSetTokenFilterTorch.test_batch_indexing`: This test exercises `test_batch_indexing` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_batch_indexing`。
- `TestTritonTorchParity._compare_outputs`: This block implements `_compare_outputs` and captures one focused piece of the module's behavior. / 该代码块实现 `_compare_outputs`，承担模块行为中的一个聚焦逻辑片段。
- `TestTritonTorchParity.test_parity_allow_tokens`: This test exercises `test_parity_allow_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parity_allow_tokens`。
- `TestTritonTorchParity.test_parity_block_tokens`: This test exercises `test_parity_block_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parity_block_tokens`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.constrained.torch_ops.token_filter_torch_ops`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 146
