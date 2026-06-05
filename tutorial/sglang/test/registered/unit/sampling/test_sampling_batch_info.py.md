# test_sampling_batch_info.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/sampling/test_sampling_batch_info.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates sampling batch info behavior in SGLang's unit / sampling area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / sampling 领域中与 sampling batch info 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for srt/sampling/sampling_batch_info.py — no server, no model loading."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-3: module imports and dependencies / 模块导入与依赖
```python
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.test.ci.ci_register`。

### Lines 5-5: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=9, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 7-17: module imports and dependencies / 模块导入与依赖
```python
import unittest
from unittest.mock import MagicMock, patch

import torch

from sglang.srt.sampling.sampling_batch_info import (
    SamplingBatchInfo,
    merge_bias_tensor,
)
from sglang.srt.sampling.sampling_params import TOP_K_ALL
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `unittest.mock`, `torch`, `sglang.srt.sampling.sampling_batch_info`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `unittest.mock`, `torch`, `sglang.srt.sampling.sampling_batch_info`。

### Lines 19-20: module-level constants and configuration / 模块级常量与配置
```python
VOCAB_SIZE = 32
DEVICE = "cpu"
```
**EN:** This block defines shared names such as `VOCAB_SIZE`, `DEVICE`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `VOCAB_SIZE`, `DEVICE` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 21-23: supporting source context / 辅助源码上下文
```python


# Helper: construct a minimal SamplingBatchInfo
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 24-40: function make info / 函数 make info
```python
def _make_info(batch_size=2, **overrides):
    """Create a SamplingBatchInfo with sane defaults for testing."""
    defaults = dict(
        temperatures=torch.ones(batch_size, 1),
        top_ps=torch.ones(batch_size),
        top_ks=torch.full((batch_size,), TOP_K_ALL, dtype=torch.int32),
        min_ps=torch.zeros(batch_size),
        is_all_greedy=False,
        need_top_p_sampling=False,
        need_top_k_sampling=False,
        need_min_p_sampling=False,
        vocab_size=VOCAB_SIZE,
        device=DEVICE,
        penalizer_orchestrator=MagicMock(is_required=False),
    )
    defaults.update(overrides)
    return SamplingBatchInfo(**defaults)
```
**EN:** Create a SamplingBatchInfo with sane defaults for testing. This block implements `_make_info` and captures one focused piece of the module's behavior.
**CN:** Create a SamplingBatchInfo with sane defaults for testing. 该代码块实现 `_make_info`，承担模块行为中的一个聚焦逻辑片段。

### Lines 43-44: class TestMergeBiasTensor declaration / 类 TestMergeBiasTensor 声明
```python
class TestMergeBiasTensor(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 45-48: test case both none returns none / 测试用例 both none returns none
```python
    def test_both_none_returns_none(self):
        """Test that merging two None tensors returns None."""
        result = merge_bias_tensor(None, None, 2, 3, DEVICE, 0.0)
        self.assertIsNone(result)
```
**EN:** Test that merging two None tensors returns None. This test exercises `test_both_none_returns_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that merging two None tensors returns None. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_both_none_returns_none`。

### Lines 50-57: test case both present concatenates / 测试用例 both present concatenates
```python
    def test_both_present_concatenates(self):
        """Test that two present tensors are concatenated along batch dim."""
        lhs = torch.ones(2, VOCAB_SIZE)
        rhs = torch.zeros(3, VOCAB_SIZE)
        result = merge_bias_tensor(lhs, rhs, 2, 3, DEVICE, 0.0)
        self.assertEqual(result.shape, (5, VOCAB_SIZE))
        self.assertEqual(result[0, 0].item(), 1.0)
        self.assertEqual(result[3, 0].item(), 0.0)
```
**EN:** Test that two present tensors are concatenated along batch dim. This test exercises `test_both_present_concatenates` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that two present tensors are concatenated along batch dim. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_both_present_concatenates`。

### Lines 59-67: test case lhs none fills default / 测试用例 lhs none fills default
```python
    def test_lhs_none_fills_default(self):
        """Test that missing lhs is filled with default value before concatenation."""
        rhs = torch.ones(3, VOCAB_SIZE)
        result = merge_bias_tensor(None, rhs, 2, 3, DEVICE, 0.0)
        self.assertEqual(result.shape, (5, VOCAB_SIZE))
        # First 2 rows filled with default (0.0)
        self.assertEqual(result[0, 0].item(), 0.0)
        # Last 3 rows from rhs
        self.assertEqual(result[2, 0].item(), 1.0)
```
**EN:** Test that missing lhs is filled with default value before concatenation. This test exercises `test_lhs_none_fills_default` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that missing lhs is filled with default value before concatenation. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lhs_none_fills_default`。

### Lines 69-76: test case rhs none fills default / 测试用例 rhs none fills default
```python
    def test_rhs_none_fills_default(self):
        """Test that missing rhs is filled with default value before concatenation."""
        lhs = torch.ones(2, VOCAB_SIZE)
        result = merge_bias_tensor(lhs, None, 2, 3, DEVICE, 0.0)
        self.assertEqual(result.shape, (5, VOCAB_SIZE))
        self.assertEqual(result[0, 0].item(), 1.0)
        # Last 3 rows filled with default (0.0)
        self.assertEqual(result[3, 0].item(), 0.0)
```
**EN:** Test that missing rhs is filled with default value before concatenation. This test exercises `test_rhs_none_fills_default` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that missing rhs is filled with default value before concatenation. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rhs_none_fills_default`。

### Lines 78-84: test case custom default value / 测试用例 custom default value
```python
    def test_custom_default_value(self):
        """Test that a custom default (-1.0) fills the missing lhs rows."""
        rhs = torch.ones(1, VOCAB_SIZE)
        result = merge_bias_tensor(None, rhs, 2, 1, DEVICE, -1.0)
        self.assertEqual(result[0, 0].item(), -1.0)
        self.assertEqual(result[1, 0].item(), -1.0)
        self.assertEqual(result[2, 0].item(), 1.0)
```
**EN:** Test that a custom default (-1.0) fills the missing lhs rows. This test exercises `test_custom_default_value` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that a custom default (-1.0) fills the missing lhs rows. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_custom_default_value`。

### Lines 85-87: supporting source context / 辅助源码上下文
```python


# SamplingBatchInfo.__len__
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 88-89: class TestSamplingBatchInfoLen declaration / 类 TestSamplingBatchInfoLen 声明
```python
class TestSamplingBatchInfoLen(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 90-93: test case len matches batch size / 测试用例 len matches batch size
```python
    def test_len_matches_batch_size(self):
        """Test that __len__ returns batch size (number of temperature rows)."""
        info = _make_info(batch_size=5)
        self.assertEqual(len(info), 5)
```
**EN:** Test that __len__ returns batch size (number of temperature rows). This test exercises `test_len_matches_batch_size` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that __len__ returns batch size (number of temperature rows). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_len_matches_batch_size`。

### Lines 96-97: class TestMergeCustomLogitProcessor declaration / 类 TestMergeCustomLogitProcessor 声明
```python
class TestMergeCustomLogitProcessor(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 98-103: test case both none returns none / 测试用例 both none returns none
```python
    def test_both_none_returns_none(self):
        """Test that merging two None processor dicts returns None."""
        result = SamplingBatchInfo.merge_custom_logit_processor(
            None, None, 2, 3, DEVICE
        )
        self.assertIsNone(result)
```
**EN:** Test that merging two None processor dicts returns None. This test exercises `test_both_none_returns_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that merging two None processor dicts returns None. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_both_none_returns_none`。

### Lines 105-115: test case same key merges masks / 测试用例 same key merges masks
```python
    def test_same_key_merges_masks(self):
        """Test that same processor key concatenates the boolean masks."""
        proc = MagicMock()
        lhs = {42: (proc, torch.tensor([True, False]))}
        rhs = {42: (proc, torch.tensor([False, True, True]))}
        result = SamplingBatchInfo.merge_custom_logit_processor(lhs, rhs, 2, 3, DEVICE)
        self.assertIn(42, result)
        self.assertEqual(result[42][1].shape[0], 5)
        self.assertTrue(result[42][1][0].item())  # from lhs
        self.assertFalse(result[42][1][1].item())  # from lhs
        self.assertTrue(result[42][1][3].item())  # from rhs
```
**EN:** Test that same processor key concatenates the boolean masks. This test exercises `test_same_key_merges_masks` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that same processor key concatenates the boolean masks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_same_key_merges_masks`。

### Lines 117-131: test case disjoint keys / 测试用例 disjoint keys
```python
    def test_disjoint_keys(self):
        """Test that disjoint processor keys are merged with zero-filled padding."""
        proc_a = MagicMock()
        proc_b = MagicMock()
        lhs = {1: (proc_a, torch.tensor([True, False]))}
        rhs = {2: (proc_b, torch.tensor([True]))}
        result = SamplingBatchInfo.merge_custom_logit_processor(lhs, rhs, 2, 1, DEVICE)
        # Key 1: lhs mask [True, False] + zero-filled rhs [False]
        self.assertEqual(result[1][1].shape[0], 3)
        self.assertTrue(result[1][1][0].item())
        self.assertFalse(result[1][1][2].item())
        # Key 2: zero-filled lhs [False, False] + rhs mask [True]
        self.assertEqual(result[2][1].shape[0], 3)
        self.assertFalse(result[2][1][0].item())
        self.assertTrue(result[2][1][2].item())
```
**EN:** Test that disjoint processor keys are merged with zero-filled padding. This test exercises `test_disjoint_keys` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that disjoint processor keys are merged with zero-filled padding. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_disjoint_keys`。

### Lines 133-139: test case lhs none rhs present / 测试用例 lhs none rhs present
```python
    def test_lhs_none_rhs_present(self):
        """Test that None lhs is treated as empty dict and rhs mask is padded."""
        proc = MagicMock()
        rhs = {10: (proc, torch.tensor([True]))}
        result = SamplingBatchInfo.merge_custom_logit_processor(None, rhs, 2, 1, DEVICE)
        self.assertIn(10, result)
        self.assertEqual(result[10][1].shape[0], 3)
```
**EN:** Test that None lhs is treated as empty dict and rhs mask is padded. This test exercises `test_lhs_none_rhs_present` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that None lhs is treated as empty dict and rhs mask is padded. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lhs_none_rhs_present`。

### Lines 140-142: supporting source context / 辅助源码上下文
```python


# apply_logits_bias
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 143-144: class TestApplyLogitsBias declaration / 类 TestApplyLogitsBias 声明
```python
class TestApplyLogitsBias(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 145-151: test case applies additive penalties / 测试用例 applies additive penalties
```python
    def test_applies_additive_penalties(self):
        """Test that pre-accumulated additive penalties are added to logits."""
        info = _make_info(batch_size=1)
        info.acc_additive_penalties = torch.tensor([[-1.0] * VOCAB_SIZE])
        logits = torch.zeros(1, VOCAB_SIZE)
        info.apply_logits_bias(logits)
        self.assertAlmostEqual(logits[0, 0].item(), -1.0, places=5)
```
**EN:** Test that pre-accumulated additive penalties are added to logits. This test exercises `test_applies_additive_penalties` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that pre-accumulated additive penalties are added to logits. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_applies_additive_penalties`。

### Lines 153-162: test case applies logit bias / 测试用例 applies logit bias
```python
    def test_applies_logit_bias(self):
        """Test that per-token logit_bias is added to logits."""
        info = _make_info(batch_size=1)
        bias = torch.zeros(1, VOCAB_SIZE)
        bias[0, 5] = 10.0
        info.logit_bias = bias
        logits = torch.zeros(1, VOCAB_SIZE)
        info.apply_logits_bias(logits)
        self.assertAlmostEqual(logits[0, 5].item(), 10.0, places=5)
        self.assertAlmostEqual(logits[0, 0].item(), 0.0, places=5)
```
**EN:** Test that per-token logit_bias is added to logits. This test exercises `test_applies_logit_bias` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that per-token logit_bias is added to logits. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_applies_logit_bias`。

### Lines 164-171: test case applies vocab mask / 测试用例 applies vocab mask
```python
    def test_applies_vocab_mask(self):
        """Test that vocab_mask triggers the apply_mask_func callback."""
        info = _make_info(batch_size=1)
        info.vocab_mask = torch.ones(1, VOCAB_SIZE)
        info.apply_mask_func = MagicMock()
        logits = torch.zeros(1, VOCAB_SIZE)
        info.apply_logits_bias(logits)
        info.apply_mask_func.assert_called_once()
```
**EN:** Test that vocab_mask triggers the apply_mask_func callback. This test exercises `test_applies_vocab_mask` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that vocab_mask triggers the apply_mask_func callback. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_applies_vocab_mask`。

### Lines 173-179: test case applies penalizer orchestrator / 测试用例 applies penalizer orchestrator
```python
    def test_applies_penalizer_orchestrator(self):
        """Test that a required orchestrator's apply() is called on logits."""
        orch = MagicMock(is_required=True)
        info = _make_info(batch_size=1, penalizer_orchestrator=orch)
        logits = torch.zeros(1, VOCAB_SIZE)
        info.apply_logits_bias(logits)
        orch.apply.assert_called_once_with(logits)
```
**EN:** Test that a required orchestrator's apply() is called on logits. This test exercises `test_applies_penalizer_orchestrator` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that a required orchestrator's apply() is called on logits. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_applies_penalizer_orchestrator`。

### Lines 181-190: test case no bias no change / 测试用例 no bias no change
```python
    def test_no_bias_no_change(self):
        """Test that logits stay unchanged when no bias sources are set."""
        info = _make_info(batch_size=1)
        info.acc_additive_penalties = None
        info.logit_bias = None
        info.vocab_mask = None
        logits = torch.zeros(1, VOCAB_SIZE)
        original = logits.clone()
        info.apply_logits_bias(logits)
        self.assertTrue(torch.equal(logits, original))
```
**EN:** Test that logits stay unchanged when no bias sources are set. This test exercises `test_no_bias_no_change` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that logits stay unchanged when no bias sources are set. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_bias_no_change`。

### Lines 191-193: supporting source context / 辅助源码上下文
```python


# update_penalties
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 194-195: class TestUpdatePenalties declaration / 类 TestUpdatePenalties 声明
```python
class TestUpdatePenalties(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 196-207: test case required creates penalties tensor / 测试用例 required creates penalties tensor
```python
    def test_required_creates_penalties_tensor(self):
        """Test that update_penalties allocates a zero tensor and calls orchestrator methods."""
        orch = MagicMock(is_required=True)
        orch.accumulate_scaling_penalties.return_value = None
        info = _make_info(batch_size=2, penalizer_orchestrator=orch)
        info.update_penalties()
        self.assertIsNotNone(info.acc_additive_penalties)
        self.assertEqual(info.acc_additive_penalties.shape, (2, VOCAB_SIZE))
        orch.accumulate_additive_penalties.assert_called_once_with(
            info.acc_additive_penalties
        )
        orch.accumulate_scaling_penalties.assert_called_once()
```
**EN:** Test that update_penalties allocates a zero tensor and calls orchestrator methods. This test exercises `test_required_creates_penalties_tensor` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that update_penalties allocates a zero tensor and calls orchestrator methods. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_required_creates_penalties_tensor`。

### Lines 209-214: test case not required sets none / 测试用例 not required sets none
```python
    def test_not_required_sets_none(self):
        """Test that update_penalties sets acc_additive_penalties to None when not required."""
        orch = MagicMock(is_required=False)
        info = _make_info(batch_size=2, penalizer_orchestrator=orch)
        info.update_penalties()
        self.assertIsNone(info.acc_additive_penalties)
```
**EN:** Test that update_penalties sets acc_additive_penalties to None when not required. This test exercises `test_not_required_sets_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that update_penalties sets acc_additive_penalties to None when not required. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_not_required_sets_none`。

### Lines 215-217: supporting source context / 辅助源码上下文
```python


# update_regex_vocab_mask
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 218-219: class TestUpdateRegexVocabMask declaration / 类 TestUpdateRegexVocabMask 声明
```python
class TestUpdateRegexVocabMask(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 220-226: test case no grammars clears mask / 测试用例 no grammars clears mask
```python
    def test_no_grammars_clears_mask(self):
        """Test that None grammars clears both vocab_mask and apply_mask_func."""
        info = _make_info(batch_size=1)
        info.grammars = None
        info.update_regex_vocab_mask()
        self.assertIsNone(info.vocab_mask)
        self.assertIsNone(info.apply_mask_func)
```
**EN:** Test that None grammars clears both vocab_mask and apply_mask_func. This test exercises `test_no_grammars_clears_mask` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that None grammars clears both vocab_mask and apply_mask_func. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_grammars_clears_mask`。

### Lines 228-233: test case empty grammars clears mask / 测试用例 empty grammars clears mask
```python
    def test_empty_grammars_clears_mask(self):
        """Test that empty grammars list clears vocab_mask."""
        info = _make_info(batch_size=1)
        info.grammars = []
        info.update_regex_vocab_mask()
        self.assertIsNone(info.vocab_mask)
```
**EN:** Test that empty grammars list clears vocab_mask. This test exercises `test_empty_grammars_clears_mask` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that empty grammars list clears vocab_mask. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_grammars_clears_mask`。

### Lines 235-247: test case with grammars allocates and fills / 测试用例 with grammars allocates and fills
```python
    def test_with_grammars_allocates_and_fills(self):
        """Test that an active grammar gets allocate, fill, and move called."""
        grammar = MagicMock()
        grammar.finished = False
        grammar.is_terminated.return_value = False
        grammar.allocate_vocab_mask.return_value = torch.zeros(1, VOCAB_SIZE)
        grammar.move_vocab_mask.return_value = torch.zeros(1, VOCAB_SIZE)
        info = _make_info(batch_size=1)
        info.grammars = [grammar]
        info.update_regex_vocab_mask()
        grammar.allocate_vocab_mask.assert_called_once()
        grammar.fill_vocab_mask.assert_called_once()
        grammar.move_vocab_mask.assert_called_once()
```
**EN:** Test that an active grammar gets allocate, fill, and move called. This test exercises `test_with_grammars_allocates_and_fills` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that an active grammar gets allocate, fill, and move called. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_with_grammars_allocates_and_fills`。

### Lines 249-270: test case mixed grammars only active fills / 测试用例 mixed grammars only active fills
```python
    def test_mixed_grammars_only_active_fills(self):
        """Test that finished, terminated, and None grammars are skipped."""
        active = MagicMock()
        active.finished = False
        active.is_terminated.return_value = False
        active.allocate_vocab_mask.return_value = torch.zeros(3, VOCAB_SIZE)
        active.move_vocab_mask.return_value = torch.zeros(3, VOCAB_SIZE)

        finished = MagicMock()
        finished.finished = True

        terminated = MagicMock()
        terminated.finished = False
        terminated.is_terminated.return_value = True

        info = _make_info(batch_size=3)
        info.grammars = [active, finished, terminated]
        info.update_regex_vocab_mask()

        active.fill_vocab_mask.assert_called_once()
        finished.fill_vocab_mask.assert_not_called()
        terminated.fill_vocab_mask.assert_not_called()
```
**EN:** Test that finished, terminated, and None grammars are skipped. This test exercises `test_mixed_grammars_only_active_fills` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that finished, terminated, and None grammars are skipped. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mixed_grammars_only_active_fills`。

### Lines 271-273: supporting source context / 辅助源码上下文
```python


# filter_batch
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 274-275: class TestFilterBatch declaration / 类 TestFilterBatch 声明
```python
class TestFilterBatch(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 276-291: test case filter keeps correct indices / 测试用例 filter keeps correct indices
```python
    def test_filter_keeps_correct_indices(self):
        """Test that filter retains rows at indices 0 and 2, dropping index 1."""
        info = _make_info(batch_size=3)
        info.temperatures = torch.tensor([[1.0], [2.0], [3.0]])
        info.top_ps = torch.tensor([0.9, 0.8, 0.7])
        info.top_ks = torch.tensor([10, 20, 30], dtype=torch.int32)
        info.min_ps = torch.tensor([0.0, 0.1, 0.2])
        info.logit_bias = torch.ones(3, VOCAB_SIZE)
        keep = torch.tensor([0, 2])
        info.filter_batch([0, 2], keep)
        self.assertEqual(len(info), 2)
        self.assertAlmostEqual(info.temperatures[0, 0].item(), 1.0)
        self.assertAlmostEqual(info.temperatures[1, 0].item(), 3.0)
        self.assertAlmostEqual(info.top_ps[1].item(), 0.7)
        # logit_bias should also be filtered
        self.assertEqual(info.logit_bias.shape, (2, VOCAB_SIZE))
```
**EN:** Test that filter retains rows at indices 0 and 2, dropping index 1. This test exercises `test_filter_keeps_correct_indices` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that filter retains rows at indices 0 and 2, dropping index 1. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_filter_keeps_correct_indices`。

### Lines 293-304: test case filter with custom logit processor / 测试用例 filter with custom logit processor
```python
    def test_filter_with_custom_logit_processor(self):
        """Test that filter updates both custom_params list and processor mask."""
        proc = MagicMock()
        info = _make_info(batch_size=3)
        info.has_custom_logit_processor = True
        info.custom_logit_processor = {42: (proc, torch.tensor([True, False, True]))}
        info.custom_params = [{"a": 1}, {"b": 2}, {"c": 3}]
        keep = torch.tensor([0, 2])
        info.filter_batch([0, 2], keep)
        self.assertEqual(info.custom_params, [{"a": 1}, {"c": 3}])
        mask = info.custom_logit_processor[42][1]
        self.assertEqual(mask.shape[0], 2)
```
**EN:** Test that filter updates both custom_params list and processor mask. This test exercises `test_filter_with_custom_logit_processor` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that filter updates both custom_params list and processor mask. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_filter_with_custom_logit_processor`。

### Lines 306-317: test case filter removes all custom processors / 测试用例 filter removes all custom processors
```python
    def test_filter_removes_all_custom_processors(self):
        """Test cleanup when filter removes all requests using a processor."""
        proc = MagicMock()
        info = _make_info(batch_size=3)
        info.has_custom_logit_processor = True
        info.custom_logit_processor = {42: (proc, torch.tensor([False, True, False]))}
        info.custom_params = [None, {"x": 1}, None]
        # Keep only index 0 and 2 — processor 42's mask becomes [False, False]
        keep = torch.tensor([0, 2])
        info.filter_batch([0, 2], keep)
        self.assertFalse(info.has_custom_logit_processor)
        self.assertIsNone(info.custom_logit_processor)
```
**EN:** Test cleanup when filter removes all requests using a processor. This test exercises `test_filter_removes_all_custom_processors` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test cleanup when filter removes all requests using a processor. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_filter_removes_all_custom_processors`。

### Lines 319-325: test case filter with none sampling seed / 测试用例 filter with none sampling seed
```python
    def test_filter_with_none_sampling_seed(self):
        """Test that filter preserves None sampling_seed without error."""
        info = _make_info(batch_size=3)
        info.sampling_seed = None
        keep = torch.tensor([1])
        info.filter_batch([1], keep)
        self.assertIsNone(info.sampling_seed)
```
**EN:** Test that filter preserves None sampling_seed without error. This test exercises `test_filter_with_none_sampling_seed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that filter preserves None sampling_seed without error. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_filter_with_none_sampling_seed`。

### Lines 326-328: supporting source context / 辅助源码上下文
```python


# merge_batch
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 329-330: class TestMergeBatch declaration / 类 TestMergeBatch 声明
```python
class TestMergeBatch(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 331-339: test case merge concatenates tensors / 测试用例 merge concatenates tensors
```python
    def test_merge_concatenates_tensors(self):
        """Test that merge concatenates temperature tensors from both batches."""
        info1 = _make_info(batch_size=2)
        info1.temperatures = torch.tensor([[1.0], [2.0]])
        info2 = _make_info(batch_size=1)
        info2.temperatures = torch.tensor([[3.0]])
        info1.merge_batch(info2)
        self.assertEqual(len(info1), 3)
        self.assertAlmostEqual(info1.temperatures[2, 0].item(), 3.0)
```
**EN:** Test that merge concatenates temperature tensors from both batches. This test exercises `test_merge_concatenates_tensors` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that merge concatenates temperature tensors from both batches. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_merge_concatenates_tensors`。

### Lines 341-359: test case merge combines flags / 测试用例 merge combines flags
```python
    def test_merge_combines_flags(self):
        """Test that merge ANDs is_all_greedy and ORs need_*_sampling flags."""
        info1 = _make_info(
            is_all_greedy=True,
            need_top_p_sampling=False,
            need_top_k_sampling=False,
            need_min_p_sampling=False,
        )
        info2 = _make_info(
            is_all_greedy=False,
            need_top_p_sampling=True,
            need_top_k_sampling=True,
            need_min_p_sampling=True,
        )
        info1.merge_batch(info2)
        self.assertFalse(info1.is_all_greedy)  # AND semantics
        self.assertTrue(info1.need_top_p_sampling)  # OR semantics
        self.assertTrue(info1.need_top_k_sampling)  # OR semantics
        self.assertTrue(info1.need_min_p_sampling)  # OR semantics
```
**EN:** Test that merge ANDs is_all_greedy and ORs need_*_sampling flags. This test exercises `test_merge_combines_flags` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that merge ANDs is_all_greedy and ORs need_*_sampling flags. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_merge_combines_flags`。

### Lines 361-368: test case merge with logit bias / 测试用例 merge with logit bias
```python
    def test_merge_with_logit_bias(self):
        """Test that merge pads missing logit_bias with zeros before concatenation."""
        info1 = _make_info(batch_size=1)
        info1.logit_bias = torch.ones(1, VOCAB_SIZE)
        info2 = _make_info(batch_size=1)
        info2.logit_bias = None
        info1.merge_batch(info2)
        self.assertEqual(info1.logit_bias.shape, (2, VOCAB_SIZE))
```
**EN:** Test that merge pads missing logit_bias with zeros before concatenation. This test exercises `test_merge_with_logit_bias` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that merge pads missing logit_bias with zeros before concatenation. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_merge_with_logit_bias`。

### Lines 370-383: test case merge with custom logit processor / 测试用例 merge with custom logit processor
```python
    def test_merge_with_custom_logit_processor(self):
        """Test that merge combines processors when only one side has them."""
        proc = MagicMock()
        info1 = _make_info(batch_size=1)
        info1.has_custom_logit_processor = True
        info1.custom_logit_processor = {1: (proc, torch.tensor([True]))}
        info1.custom_params = [{"a": 1}]
        info2 = _make_info(batch_size=1)
        info2.has_custom_logit_processor = False
        info2.custom_logit_processor = None
        info2.custom_params = None
        info1.merge_batch(info2)
        self.assertTrue(info1.has_custom_logit_processor)
        self.assertEqual(len(info1.custom_params), 2)
```
**EN:** Test that merge combines processors when only one side has them. This test exercises `test_merge_with_custom_logit_processor` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that merge combines processors when only one side has them. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_merge_with_custom_logit_processor`。

### Lines 385-392: test case merge with none sampling seed / 测试用例 merge with none sampling seed
```python
    def test_merge_with_none_sampling_seed(self):
        """Test that merge preserves None when both sampling_seeds are None."""
        info1 = _make_info(batch_size=1)
        info1.sampling_seed = None
        info2 = _make_info(batch_size=1)
        info2.sampling_seed = None
        info1.merge_batch(info2)
        self.assertIsNone(info1.sampling_seed)
```
**EN:** Test that merge preserves None when both sampling_seeds are None. This test exercises `test_merge_with_none_sampling_seed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that merge preserves None when both sampling_seeds are None. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_merge_with_none_sampling_seed`。

### Lines 394-404: test case merge with both sampling seeds / 测试用例 merge with both sampling seeds
```python
    def test_merge_with_both_sampling_seeds(self):
        """Test that merge concatenates both sampling_seed tensors."""
        info1 = _make_info(batch_size=2)
        info1.sampling_seed = torch.tensor([10, 20], dtype=torch.int64)
        info2 = _make_info(batch_size=1)
        info2.sampling_seed = torch.tensor([30], dtype=torch.int64)
        info1.merge_batch(info2)
        self.assertEqual(info1.sampling_seed.shape[0], 3)
        self.assertEqual(info1.sampling_seed[0].item(), 10)
        self.assertEqual(info1.sampling_seed[1].item(), 20)
        self.assertEqual(info1.sampling_seed[2].item(), 30)
```
**EN:** Test that merge concatenates both sampling_seed tensors. This test exercises `test_merge_with_both_sampling_seeds` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that merge concatenates both sampling_seed tensors. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_merge_with_both_sampling_seeds`。

### Lines 405-407: supporting source context / 辅助源码上下文
```python


# copy_for_forward
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 408-409: class TestCopyForForward declaration / 类 TestCopyForForward 声明
```python
class TestCopyForForward(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 410-417: test case returns copy without orchestrator / 测试用例 returns copy without orchestrator
```python
    def test_returns_copy_without_orchestrator(self):
        """Test that copy_for_forward returns a copy with orchestrator set to None."""
        orch = MagicMock(is_required=False)
        info = _make_info(batch_size=1, penalizer_orchestrator=orch)
        copied = info.copy_for_forward()
        self.assertIsNone(copied.penalizer_orchestrator)
        # Original should still have orchestrator
        self.assertIsNotNone(info.penalizer_orchestrator)
```
**EN:** Test that copy_for_forward returns a copy with orchestrator set to None. This test exercises `test_returns_copy_without_orchestrator` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that copy_for_forward returns a copy with orchestrator set to None. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_returns_copy_without_orchestrator`。

### Lines 418-420: supporting source context / 辅助源码上下文
```python


# from_schedule_batch
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 421-422: class TestFromScheduleBatch declaration / 类 TestFromScheduleBatch 声明
```python
class TestFromScheduleBatch(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 423-452: method make req / 方法 make req
```python
    def _make_req(
        self,
        temp=1.0,
        top_p=1.0,
        top_k=-1,
        min_p=0.0,
        freq=0.0,
        presence=0.0,
        min_tokens=0,
        logit_bias=None,
        seed=None,
        stop_ids=None,
        eos_id=2,
    ):
        req = MagicMock()
        req.sampling_params.temperature = temp
        req.sampling_params.top_p = top_p
        req.sampling_params.top_k = top_k
        req.sampling_params.min_p = min_p
        req.sampling_params.frequency_penalty = freq
        req.sampling_params.presence_penalty = presence
        req.sampling_params.min_new_tokens = min_tokens
        req.sampling_params.logit_bias = logit_bias
        req.sampling_params.sampling_seed = seed
        req.sampling_params.stop_token_ids = stop_ids
        req.sampling_params.custom_params = None
        req.custom_logit_processor = None
        req.tokenizer.additional_stop_token_ids = None
        req.tokenizer.eos_token_id = eos_id
        return req
```
**EN:** This block implements `_make_req` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_req`，承担模块行为中的一个聚焦逻辑片段。

### Lines 454-469: test case basic construction / 测试用例 basic construction
```python
    @patch("sglang.srt.sampling.sampling_batch_info.get_global_server_args")
    def test_basic_construction(self, mock_server_args):
        """Test that from_schedule_batch correctly extracts sampling params from requests."""
        mock_server_args.return_value.enable_deterministic_inference = False
        mock_server_args.return_value.enable_custom_logit_processor = False

        reqs = [self._make_req(temp=0.8, top_p=0.9, top_k=50, min_p=0.1)]
        batch = MagicMock()
        batch.reqs = reqs
        batch.device = DEVICE

        info = SamplingBatchInfo.from_schedule_batch(batch, VOCAB_SIZE)
        self.assertEqual(len(info), 1)
        self.assertAlmostEqual(info.temperatures[0, 0].item(), 0.8, places=5)
        self.assertAlmostEqual(info.top_ps[0].item(), 0.9, places=5)
        self.assertEqual(info.top_ks[0].item(), 50)
```
**EN:** Test that from_schedule_batch correctly extracts sampling params from requests. This test exercises `test_basic_construction` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that from_schedule_batch correctly extracts sampling params from requests. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_basic_construction`。

### Lines 471-482: test case greedy detection / 测试用例 greedy detection
```python
    @patch("sglang.srt.sampling.sampling_batch_info.get_global_server_args")
    def test_greedy_detection(self, mock_server_args):
        """Test that top_k=1 sets is_all_greedy=True."""
        mock_server_args.return_value.enable_deterministic_inference = False
        mock_server_args.return_value.enable_custom_logit_processor = False

        reqs = [self._make_req(top_k=1)]
        batch = MagicMock()
        batch.reqs = reqs
        batch.device = DEVICE
        info = SamplingBatchInfo.from_schedule_batch(batch, VOCAB_SIZE)
        self.assertTrue(info.is_all_greedy)
```
**EN:** Test that top_k=1 sets is_all_greedy=True. This test exercises `test_greedy_detection` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that top_k=1 sets is_all_greedy=True. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_greedy_detection`。

### Lines 484-498: test case logit bias construction / 测试用例 logit bias construction
```python
    @patch("sglang.srt.sampling.sampling_batch_info.get_global_server_args")
    def test_logit_bias_construction(self, mock_server_args):
        """Test that logit_bias dict is converted to a tensor with correct values."""
        mock_server_args.return_value.enable_deterministic_inference = False
        mock_server_args.return_value.enable_custom_logit_processor = False

        reqs = [self._make_req(logit_bias={"5": 2.0, "10": -1.0})]
        batch = MagicMock()
        batch.reqs = reqs
        batch.device = DEVICE
        info = SamplingBatchInfo.from_schedule_batch(batch, VOCAB_SIZE)
        self.assertIsNotNone(info.logit_bias)
        self.assertAlmostEqual(info.logit_bias[0, 5].item(), 2.0)
        self.assertAlmostEqual(info.logit_bias[0, 10].item(), -1.0)
        self.assertAlmostEqual(info.logit_bias[0, 0].item(), 0.0)
```
**EN:** Test that logit_bias dict is converted to a tensor with correct values. This test exercises `test_logit_bias_construction` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that logit_bias dict is converted to a tensor with correct values. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_logit_bias_construction`。

### Lines 500-513: test case deterministic seed / 测试用例 deterministic seed
```python
    @patch("sglang.srt.sampling.sampling_batch_info.get_global_server_args")
    def test_deterministic_seed(self, mock_server_args):
        """Test that explicit seed=123 is kept and missing seed defaults to 42."""
        mock_server_args.return_value.enable_deterministic_inference = True
        mock_server_args.return_value.enable_custom_logit_processor = False

        reqs = [self._make_req(seed=123), self._make_req(seed=None)]
        batch = MagicMock()
        batch.reqs = reqs
        batch.device = DEVICE
        info = SamplingBatchInfo.from_schedule_batch(batch, VOCAB_SIZE)
        self.assertIsNotNone(info.sampling_seed)
        self.assertEqual(info.sampling_seed[0].item(), 123)
        self.assertEqual(info.sampling_seed[1].item(), 42)  # default
```
**EN:** Test that explicit seed=123 is kept and missing seed defaults to 42. This test exercises `test_deterministic_seed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that explicit seed=123 is kept and missing seed defaults to 42. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_deterministic_seed`。

### Lines 515-529: test case from schedule batch sampling flags / 测试用例 from schedule batch sampling flags
```python
    @patch("sglang.srt.sampling.sampling_batch_info.get_global_server_args")
    def test_from_schedule_batch_sampling_flags(self, mock_server_args):
        """Test that sampling flags (need_top_p/top_k/min_p) are set correctly."""
        mock_server_args.return_value.enable_deterministic_inference = False
        mock_server_args.return_value.enable_custom_logit_processor = False

        reqs = [self._make_req(top_p=0.9, top_k=50, min_p=0.1)]
        batch = MagicMock()
        batch.reqs = reqs
        batch.device = DEVICE
        info = SamplingBatchInfo.from_schedule_batch(batch, VOCAB_SIZE)
        self.assertTrue(info.need_top_p_sampling)  # 0.9 != 1.0
        self.assertTrue(info.need_top_k_sampling)  # 50 != TOP_K_ALL
        self.assertTrue(info.need_min_p_sampling)  # 0.1 > 0
        self.assertFalse(info.is_all_greedy)  # top_k=50 > 1
```
**EN:** Test that sampling flags (need_top_p/top_k/min_p) are set correctly. This test exercises `test_from_schedule_batch_sampling_flags` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that sampling flags (need_top_p/top_k/min_p) are set correctly. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_from_schedule_batch_sampling_flags`。

### Lines 531-542: test case no logit bias when all none / 测试用例 no logit bias when all none
```python
    @patch("sglang.srt.sampling.sampling_batch_info.get_global_server_args")
    def test_no_logit_bias_when_all_none(self, mock_server_args):
        """Test that logit_bias stays None when no request has logit_bias set."""
        mock_server_args.return_value.enable_deterministic_inference = False
        mock_server_args.return_value.enable_custom_logit_processor = False

        reqs = [self._make_req(), self._make_req()]
        batch = MagicMock()
        batch.reqs = reqs
        batch.device = DEVICE
        info = SamplingBatchInfo.from_schedule_batch(batch, VOCAB_SIZE)
        self.assertIsNone(info.logit_bias)
```
**EN:** Test that logit_bias stays None when no request has logit_bias set. This test exercises `test_no_logit_bias_when_all_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that logit_bias stays None when no request has logit_bias set. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_logit_bias_when_all_none`。

### Lines 544-577: test case custom logit processor merging / 测试用例 custom logit processor merging
```python
    @patch("sglang.srt.sampling.sampling_batch_info.get_global_server_args")
    def test_custom_logit_processor_merging(self, mock_server_args):
        """Test deserialization and merging of custom logit processors."""
        from sglang.srt.sampling.custom_logit_processor import (
            DisallowedTokensLogitsProcessor,
        )

        mock_server_args.return_value.enable_deterministic_inference = False
        mock_server_args.return_value.enable_custom_logit_processor = True

        proc_str = DisallowedTokensLogitsProcessor.to_str()
        req1 = self._make_req()
        req1.custom_logit_processor = proc_str
        req1.sampling_params.custom_params = {"token_ids": [1]}
        req2 = self._make_req()
        req2.custom_logit_processor = None  # no processor
        req2.sampling_params.custom_params = None

        batch = MagicMock()
        batch.reqs = [req1, req2]
        batch.device = DEVICE
        info = SamplingBatchInfo.from_schedule_batch(batch, VOCAB_SIZE)

        self.assertTrue(info.has_custom_logit_processor)
        self.assertIsNotNone(info.custom_logit_processor)
        self.assertEqual(len(info.custom_logit_processor), 1)
        # Check the mask: req1 has processor (True), req2 doesn't (False)
        key = list(info.custom_logit_processor.keys())[0]
        proc, mask = info.custom_logit_processor[key]
        self.assertIsInstance(proc, DisallowedTokensLogitsProcessor)
        self.assertTrue(mask[0].item())
        self.assertFalse(mask[1].item())
        # custom_params should be collected for all reqs
        self.assertEqual(len(info.custom_params), 2)
```
**EN:** Test deserialization and merging of custom logit processors. This test exercises `test_custom_logit_processor_merging` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test deserialization and merging of custom logit processors. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_custom_logit_processor_merging`。

### Lines 580-581: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_make_info`: Create a SamplingBatchInfo with sane defaults for testing. / 该代码块实现 `_make_info`，承担模块行为中的一个聚焦逻辑片段。
- `TestMergeBiasTensor`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestSamplingBatchInfoLen`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestMergeCustomLogitProcessor`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestApplyLogitsBias`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestUpdatePenalties`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestUpdateRegexVocabMask`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestFilterBatch`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestMergeBiasTensor.test_both_none_returns_none`: Test that merging two None tensors returns None. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_both_none_returns_none`。
- `TestMergeBiasTensor.test_both_present_concatenates`: Test that two present tensors are concatenated along batch dim. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_both_present_concatenates`。
- `TestMergeBiasTensor.test_lhs_none_fills_default`: Test that missing lhs is filled with default value before concatenation. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lhs_none_fills_default`。
- `TestMergeBiasTensor.test_rhs_none_fills_default`: Test that missing rhs is filled with default value before concatenation. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rhs_none_fills_default`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `unittest.mock`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.srt.sampling.sampling_batch_info`, `sglang.srt.sampling.sampling_params`, `sglang.test.test_utils`

- **Total lines / 总行数**: 581
