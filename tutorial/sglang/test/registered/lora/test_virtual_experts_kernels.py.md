# test_virtual_experts_kernels.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/lora/test_virtual_experts_kernels.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates virtual experts kernels behavior in SGLang's lora area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 lora 领域中与 virtual experts kernels 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: supporting statements / 辅助语句
```python
"""Unit tests for the LoRA virtual-experts kernels under post-EP-dispatch
sentinel `topk_ids` (-1) and out-of-range expert IDs.

Covers two regression bugs that surface only with `--lora-use-virtual-experts`
+ `ep_size > 1`:

- `_fused_virtual_topk_ids` must preserve negative sentinel topk_ids. After
  EP dispatch, non-local experts arrive as `-1`; the pre-fix kernel mapped
  them onto a real virtual-expert slot belonging to another adapter and
  triggered OOB loads in downstream LoRA kernels.

- `_align_block_size_torch` / `_align_block_size_jit` (the `>= 1024`-expert
  fallback paths) must route `-1` and `>= num_experts` IDs into a sentinel
  bucket so they don't OOB-index `padded_offsets[sorted_expert_ids]` (negative
  wrap, or past-end) and don't get assigned to a real expert in the
  consumer-block table.

Both kernels run on CUDA. The fallback is gated on `virtual_num_experts >= 1024`
in production, but we exercise it directly here at smaller sizes for cheaper
iteration; one test sticks to the >1024 regime to mirror the production trigger.

Usage:
    python -m pytest test/registered/lora/test_virtual_experts_kernels.py -v
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 26-31: module imports and dependencies / 模块导入与依赖
```python
import unittest

import torch

from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `torch`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `torch`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`。

### Lines 33-33: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=15, stage="base-b", runner_config="1-gpu-small")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 35-40: module imports and dependencies / 模块导入与依赖
```python
from sglang.srt.lora.triton_ops.virtual_experts import (
    _align_block_size_jit,
    _align_block_size_torch,
    _fused_virtual_topk_ids,
    fused_sanitize_expert_ids,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.srt.lora.triton_ops.virtual_experts`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.srt.lora.triton_ops.virtual_experts`。

### Lines 43-43: class TestFusedVirtualTopkIdsPreservesSentinels declaration / 类 TestFusedVirtualTopkIdsPreservesSentinels 声明
```python
class TestFusedVirtualTopkIdsPreservesSentinels(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 44-44: supporting statements / 辅助语句
```python
    """Item B regression: post-EP-dispatch -1 sentinels must NOT be remapped."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 46-50: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        if not torch.cuda.is_available():
            raise unittest.SkipTest("CUDA required")
        cls.device = "cuda:0"
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 52-94: test case negative sentinels preserved / 测试用例 negative sentinels preserved
```python
    def test_negative_sentinels_preserved(self):
        # Mix of valid topk_ids in [0, num_experts), -1 sentinels (typical
        # post-EP-dispatch), and a synthetic -2 to ensure the fix doesn't
        # depend on the exact -1 value.
        topk_ids = torch.tensor(
            [
                [3, -1],
                [-1, 5],
                [0, 7],
                [-1, -1],
                [2, 9],
                [11, -2],
                [-1, 4],
                [6, -1],
            ],
            dtype=torch.int32,
            device=self.device,
        )
        token_lora_mapping = torch.tensor(
            [0, 1, 0, 2, -1, 1, 0, 1], dtype=torch.int32, device=self.device
        )
        num_experts = 16
        max_loras = 4

        virtual_ids, _, _ = _fused_virtual_topk_ids(
            topk_ids,
            token_lora_mapping,
            num_experts,
            shared_outer=False,
            max_loras=max_loras,
        )

        # Every negative input must stay negative (and equal) in the output.
        for m in range(topk_ids.shape[0]):
            for k in range(topk_ids.shape[1]):
                base = topk_ids[m, k].item()
                if base < 0:
                    self.assertEqual(
                        virtual_ids[m, k].item(),
                        base,
                        f"negative sentinel at ({m},{k}) was remapped: "
                        f"{base} -> {virtual_ids[m, k].item()}",
                    )
```
**EN:** This test exercises `test_negative_sentinels_preserved` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_negative_sentinels_preserved`。

### Lines 96-117: test case positive topk remapped correctly / 测试用例 positive topk remapped correctly
```python
    def test_positive_topk_remapped_correctly(self):
        """Sanity: valid (non-negative) IDs follow the
        `base + safe_lora * num_experts` rule."""
        topk_ids = torch.tensor(
            [[3, 1], [0, 7], [2, 9]], dtype=torch.int32, device=self.device
        )
        token_lora_mapping = torch.tensor(
            [0, 1, 2], dtype=torch.int32, device=self.device
        )
        num_experts = 16
        max_loras = 4

        virtual_ids, _, _ = _fused_virtual_topk_ids(
            topk_ids, token_lora_mapping, num_experts, False, max_loras
        )

        for m in range(topk_ids.shape[0]):
            lora = token_lora_mapping[m].item()
            for k in range(topk_ids.shape[1]):
                base = topk_ids[m, k].item()
                expected = base + max(lora, 0) * num_experts
                self.assertEqual(virtual_ids[m, k].item(), expected)
```
**EN:** Sanity: valid (non-negative) IDs follow the `base + safe_lora * num_experts` rule. This test exercises `test_positive_topk_remapped_correctly` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Sanity: valid (non-negative) IDs follow the `base + safe_lora * num_experts` rule. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_positive_topk_remapped_correctly`。

### Lines 119-131: test case no lora token does not shift base / 测试用例 no lora token does not shift base
```python
    def test_no_lora_token_does_not_shift_base(self):
        """`token_lora_mapping[m] == -1` (no LoRA) keeps `safe_lora=0`,
        so positive bases pass through unchanged and the row mask is False."""
        topk_ids = torch.tensor([[3, 5]], dtype=torch.int32, device=self.device)
        token_lora_mapping = torch.tensor([-1], dtype=torch.int32, device=self.device)
        num_experts = 16

        virtual_ids, mask, _ = _fused_virtual_topk_ids(
            topk_ids, token_lora_mapping, num_experts, False, max_loras=4
        )
        self.assertEqual(virtual_ids[0, 0].item(), 3)
        self.assertEqual(virtual_ids[0, 1].item(), 5)
        self.assertFalse(bool(mask[0].item()))
```
**EN:** `token_lora_mapping[m] == -1` (no LoRA) keeps `safe_lora=0`, so positive bases pass through unchanged and the row mask is False. This test exercises `test_no_lora_token_does_not_shift_base` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** `token_lora_mapping[m] == -1` (no LoRA) keeps `safe_lora=0`, so positive bases pass through unchanged and the row mask is False. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_lora_token_does_not_shift_base`。

### Lines 134-134: class _AlignBlockSizeSentinelBucketBase declaration / 类 _AlignBlockSizeSentinelBucketBase 声明
```python
class _AlignBlockSizeSentinelBucketBase(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 135-138: supporting statements / 辅助语句
```python
    """Shared tests for both the torch.compile and JIT align_block_size paths.

    Subclasses override ``_align`` to select the concrete implementation.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 140-146: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        if not torch.cuda.is_available():
            raise unittest.SkipTest("CUDA required")
        if cls is _AlignBlockSizeSentinelBucketBase:
            raise unittest.SkipTest("Base class")
        cls.device = "cuda:0"
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 148-149: method align / 方法 align
```python
    def _align(self, topk_ids, block_size, num_experts):
        raise NotImplementedError
```
**EN:** This block implements `_align` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_align`，承担模块行为中的一个聚焦逻辑片段。

### Lines 151-155: method assigned experts / 方法 assigned experts
```python
    @staticmethod
    def _assigned_experts(expert_ids: torch.Tensor) -> list:
        """Return the list of real expert ids assigned to blocks (filtering
        out -1 sentinels for padding/exclusion)."""
        return expert_ids[expert_ids != -1].cpu().tolist()
```
**EN:** Return the list of real expert ids assigned to blocks (filtering out -1 sentinels for padding/exclusion). This block implements `_assigned_experts` and captures one focused piece of the module's behavior.
**CN:** Return the list of real expert ids assigned to blocks (filtering out -1 sentinels for padding/exclusion). 该代码块实现 `_assigned_experts`，承担模块行为中的一个聚焦逻辑片段。

### Lines 157-162: method assert only real or sentinel / 方法 assert only real or sentinel
```python
    def _assert_only_real_or_sentinel(self, expert_ids: torch.Tensor, num_experts: int):
        for eid in expert_ids.cpu().tolist():
            self.assertTrue(
                eid == -1 or 0 <= eid < num_experts,
                f"expert_ids contains invalid value {eid}",
            )
```
**EN:** This block implements `_assert_only_real_or_sentinel` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_assert_only_real_or_sentinel`，承担模块行为中的一个聚焦逻辑片段。

### Lines 164-179: test case all valid baseline / 测试用例 all valid baseline
```python
    def test_all_valid_baseline(self):
        """Sanity: with no invalid IDs, every present real expert appears
        in the assignment, and no junk values leak through."""
        num_experts = 8
        block_size = 16
        topk_ids = torch.tensor(
            [[0, 3], [4, 7], [1, 2], [5, 6]],
            dtype=torch.int32,
            device=self.device,
        )

        _, expert_ids, _ = self._align(topk_ids, block_size, num_experts)

        self._assert_only_real_or_sentinel(expert_ids, num_experts)
        assigned = set(self._assigned_experts(expert_ids))
        self.assertEqual(assigned, set(range(num_experts)))
```
**EN:** Sanity: with no invalid IDs, every present real expert appears in the assignment, and no junk values leak through. This test exercises `test_all_valid_baseline` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Sanity: with no invalid IDs, every present real expert appears in the assignment, and no junk values leak through. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_valid_baseline`。

### Lines 181-197: test case negative ids routed to sentinel / 测试用例 negative ids routed to sentinel
```python
    def test_negative_ids_routed_to_sentinel(self):
        """`-1` tokens must not appear as real expert assignments and must
        not corrupt the assignment of real IDs."""
        num_experts = 8
        block_size = 16
        topk_ids = torch.tensor(
            [[0, -1], [-1, 7], [1, -1], [-1, -1]],
            dtype=torch.int32,
            device=self.device,
        )

        _, expert_ids, _ = self._align(topk_ids, block_size, num_experts)

        self._assert_only_real_or_sentinel(expert_ids, num_experts)
        assigned = self._assigned_experts(expert_ids)
        for valid_eid in (0, 1, 7):
            self.assertIn(valid_eid, assigned)
```
**EN:** `-1` tokens must not appear as real expert assignments and must not corrupt the assignment of real IDs. This test exercises `test_negative_ids_routed_to_sentinel` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** `-1` tokens must not appear as real expert assignments and must not corrupt the assignment of real IDs. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_negative_ids_routed_to_sentinel`。

### Lines 199-219: test case oor ids routed to sentinel / 测试用例 oor ids routed to sentinel
```python
    def test_oor_ids_routed_to_sentinel(self):
        """IDs `>= num_experts` (e.g. virtual-experts remap when combined
        with non-local sentinels) must not break cumsum/searchsorted and
        must not show up as real assignments."""
        num_experts = 8
        block_size = 16
        topk_ids = torch.tensor(
            [[0, 100], [50, 7], [1, 200]],
            dtype=torch.int32,
            device=self.device,
        )

        _, expert_ids, _ = self._align(topk_ids, block_size, num_experts)

        self._assert_only_real_or_sentinel(expert_ids, num_experts)
        assigned = self._assigned_experts(expert_ids)
        for valid_eid in (0, 1, 7, 50):
            if valid_eid >= num_experts:
                self.assertNotIn(valid_eid, assigned)
            else:
                self.assertIn(valid_eid, assigned)
```
**EN:** IDs `>= num_experts` (e.g. This test exercises `test_oor_ids_routed_to_sentinel` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** IDs `>= num_experts` (e.g. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_oor_ids_routed_to_sentinel`。

### Lines 221-243: test case mixed invalid at production size / 测试用例 mixed invalid at production size
```python
    def test_mixed_invalid_at_production_size(self):
        """Mirror the production trigger: `num_experts >= 1024` (only path
        where the large-expert fallback is invoked instead of the native
        align kernel)."""
        num_experts = 1500
        block_size = 16
        topk_ids = torch.tensor(
            [
                [-1, 500],
                [num_experts + 7, 1000],
                [num_experts * 2, 100],
                [-1, 0],
            ],
            dtype=torch.int32,
            device=self.device,
        )

        _, expert_ids, _ = self._align(topk_ids, block_size, num_experts)

        self._assert_only_real_or_sentinel(expert_ids, num_experts)
        assigned = self._assigned_experts(expert_ids)
        for valid_eid in (0, 100, 500, 1000):
            self.assertIn(valid_eid, assigned)
```
**EN:** Mirror the production trigger: `num_experts >= 1024` (only path where the large-expert fallback is invoked instead of the native align kernel). This test exercises `test_mixed_invalid_at_production_size` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Mirror the production trigger: `num_experts >= 1024` (only path where the large-expert fallback is invoked instead of the native align kernel). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mixed_invalid_at_production_size`。

### Lines 245-257: test case empty topk ids does not crash / 测试用例 empty topk ids does not crash
```python
    def test_empty_topk_ids_does_not_crash(self):
        """Edge: empty input. Should return empty/zero outputs without
        OOB indexing on the sentinel bucket."""
        num_experts = 8
        block_size = 16
        topk_ids = torch.empty((0, 2), dtype=torch.int32, device=self.device)

        sorted_token_ids, expert_ids, num_post_padded = self._align(
            topk_ids, block_size, num_experts
        )

        self.assertEqual(num_post_padded.item(), 0)
        self.assertEqual(self._assigned_experts(expert_ids), [])
```
**EN:** Edge: empty input. This test exercises `test_empty_topk_ids_does_not_crash` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Edge: empty input. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_topk_ids_does_not_crash`。

### Lines 260-260: class TestAlignBlockSizeTorchSentinelBucket declaration / 类 TestAlignBlockSizeTorchSentinelBucket 声明
```python
class TestAlignBlockSizeTorchSentinelBucket(_AlignBlockSizeSentinelBucketBase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `_AlignBlockSizeSentinelBucketBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `_AlignBlockSizeSentinelBucketBase`。

### Lines 261-261: supporting statements / 辅助语句
```python
    """Test the pure-PyTorch torch.compile fallback path (AMD/ROCm compatible)."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 263-264: method align / 方法 align
```python
    def _align(self, topk_ids, block_size, num_experts):
        return _align_block_size_torch(topk_ids, block_size, num_experts)
```
**EN:** This block implements `_align` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_align`，承担模块行为中的一个聚焦逻辑片段。

### Lines 267-267: class TestAlignBlockSizeJitSentinelBucket declaration / 类 TestAlignBlockSizeJitSentinelBucket 声明
```python
class TestAlignBlockSizeJitSentinelBucket(_AlignBlockSizeSentinelBucketBase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `_AlignBlockSizeSentinelBucketBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `_AlignBlockSizeSentinelBucketBase`。

### Lines 268-269: supporting statements / 辅助语句
```python
    """Test the CUDA JIT kernel path (with fused_sanitize_expert_ids, as in
    production)."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 271-276: method align / 方法 align
```python
    def _align(self, topk_ids, block_size, num_experts):
        sorted_token_ids, expert_ids, num_tokens_post_padded = _align_block_size_jit(
            topk_ids, block_size, num_experts
        )
        expert_ids = fused_sanitize_expert_ids(expert_ids, num_experts)
        return sorted_token_ids, expert_ids, num_tokens_post_padded
```
**EN:** This block implements `_align` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_align`，承担模块行为中的一个聚焦逻辑片段。

### Lines 279-280: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=2)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestFusedVirtualTopkIdsPreservesSentinels`: Item B regression: post-EP-dispatch -1 sentinels must NOT be remapped. / 用于组织相关测试、夹具或辅助方法。
- `_AlignBlockSizeSentinelBucketBase`: Shared tests for both the torch.compile and JIT align_block_size paths. / 用于组织相关测试、夹具或辅助方法。
- `TestAlignBlockSizeTorchSentinelBucket`: Test the pure-PyTorch torch.compile fallback path (AMD/ROCm compatible). / 用于组织相关测试、夹具或辅助方法。
- `TestAlignBlockSizeJitSentinelBucket`: Test the CUDA JIT kernel path (with fused_sanitize_expert_ids, as in production). / 用于组织相关测试、夹具或辅助方法。
- `TestFusedVirtualTopkIdsPreservesSentinels.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestFusedVirtualTopkIdsPreservesSentinels.test_negative_sentinels_preserved`: This test exercises `test_negative_sentinels_preserved` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_negative_sentinels_preserved`。
- `TestFusedVirtualTopkIdsPreservesSentinels.test_positive_topk_remapped_correctly`: Sanity: valid (non-negative) IDs follow the `base + safe_lora * num_experts` rule. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_positive_topk_remapped_correctly`。
- `TestFusedVirtualTopkIdsPreservesSentinels.test_no_lora_token_does_not_shift_base`: `token_lora_mapping[m] == -1` (no LoRA) keeps `safe_lora=0`, so positive bases pass through unchanged and the row mask is False. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_lora_token_does_not_shift_base`。
- `_AlignBlockSizeSentinelBucketBase.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `_AlignBlockSizeSentinelBucketBase._align`: This block implements `_align` and captures one focused piece of the module's behavior. / 该代码块实现 `_align`，承担模块行为中的一个聚焦逻辑片段。
- `_AlignBlockSizeSentinelBucketBase._assigned_experts`: Return the list of real expert ids assigned to blocks (filtering out -1 sentinels for padding/exclusion). / 该代码块实现 `_assigned_experts`，承担模块行为中的一个聚焦逻辑片段。
- `_AlignBlockSizeSentinelBucketBase._assert_only_real_or_sentinel`: This block implements `_assert_only_real_or_sentinel` and captures one focused piece of the module's behavior. / 该代码块实现 `_assert_only_real_or_sentinel`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.test_utils`, `sglang.srt.lora.triton_ops.virtual_experts`

- **Total lines / 总行数**: 280
