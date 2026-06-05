# test_penaltylib.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/sampling/test_penaltylib.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates penaltylib behavior in SGLang's unit / sampling area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / sampling 领域中与 penaltylib 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for srt/sampling/penaltylib/ — no server, no model loading."""
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

### Lines 7-24: module imports and dependencies / 模块导入与依赖
```python
import unittest
from unittest.mock import MagicMock

import torch

from sglang.srt.sampling.penaltylib.frequency_penalty import (
    BatchedFrequencyPenalizer,
)
from sglang.srt.sampling.penaltylib.min_new_tokens import (
    BatchedMinNewTokensPenalizer,
)
from sglang.srt.sampling.penaltylib.orchestrator import (
    BatchedPenalizerOrchestrator,
)
from sglang.srt.sampling.penaltylib.presence_penalty import (
    BatchedPresencePenalizer,
)
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `unittest.mock`, `torch`, `sglang.srt.sampling.penaltylib.frequency_penalty`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `unittest.mock`, `torch`, `sglang.srt.sampling.penaltylib.frequency_penalty`。

### Lines 26-27: module-level constants and configuration / 模块级常量与配置
```python
VOCAB_SIZE = 32
DEVICE = "cpu"
```
**EN:** This block defines shared names such as `VOCAB_SIZE`, `DEVICE`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `VOCAB_SIZE`, `DEVICE` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 28-30: supporting source context / 辅助源码上下文
```python


# Helpers: mock Req and ScheduleBatch
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 31-40: function make req / 函数 make req
```python
def _make_req(freq=0.0, presence=0.0, min_tokens=0, stop_ids=None, eos_id=2):
    """Create a mock request with sampling params."""
    req = MagicMock()
    req.sampling_params.frequency_penalty = freq
    req.sampling_params.presence_penalty = presence
    req.sampling_params.min_new_tokens = min_tokens
    req.sampling_params.stop_token_ids = stop_ids
    req.tokenizer.additional_stop_token_ids = None
    req.tokenizer.eos_token_id = eos_id
    return req
```
**EN:** Create a mock request with sampling params. This block implements `_make_req` and captures one focused piece of the module's behavior.
**CN:** Create a mock request with sampling params. 该代码块实现 `_make_req`，承担模块行为中的一个聚焦逻辑片段。

### Lines 43-49: function make batch / 函数 make batch
```python
def _make_batch(reqs):
    """Create a mock ScheduleBatch.
    Note: orchestrator accesses batch.reqs as an attribute (not a method call)."""
    batch = MagicMock()
    batch.reqs = reqs
    batch.device = DEVICE
    return batch
```
**EN:** Create a mock ScheduleBatch. This block implements `_make_batch` and captures one focused piece of the module's behavior.
**CN:** Create a mock ScheduleBatch. 该代码块实现 `_make_batch`，承担模块行为中的一个聚焦逻辑片段。

### Lines 50-52: supporting source context / 辅助源码上下文
```python


# BatchedPenalizerOrchestrator
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 53-54: class TestBatchedPenalizerOrchestrator declaration / 类 TestBatchedPenalizerOrchestrator 声明
```python
class TestBatchedPenalizerOrchestrator(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 55-62: test case init detects required penalizers / 测试用例 init detects required penalizers
```python
    def test_init_detects_required_penalizers(self):
        """Test that orchestrator marks is_required=True when any request has nonzero penalty."""
        reqs = [_make_req(freq=1.0)]
        batch = _make_batch(reqs)
        orch = BatchedPenalizerOrchestrator(
            VOCAB_SIZE, batch, {BatchedFrequencyPenalizer}
        )
        self.assertTrue(orch.is_required)
```
**EN:** Test that orchestrator marks is_required=True when any request has nonzero penalty. This test exercises `test_init_detects_required_penalizers` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that orchestrator marks is_required=True when any request has nonzero penalty. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_detects_required_penalizers`。

### Lines 64-71: test case init not required when no penalties / 测试用例 init not required when no penalties
```python
    def test_init_not_required_when_no_penalties(self):
        """Test that orchestrator marks is_required=False when all penalties are zero."""
        reqs = [_make_req()]  # all defaults (0.0)
        batch = _make_batch(reqs)
        orch = BatchedPenalizerOrchestrator(
            VOCAB_SIZE, batch, {BatchedFrequencyPenalizer}
        )
        self.assertFalse(orch.is_required)
```
**EN:** Test that orchestrator marks is_required=False when all penalties are zero. This test exercises `test_init_not_required_when_no_penalties` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that orchestrator marks is_required=False when all penalties are zero. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_not_required_when_no_penalties`。

### Lines 73-78: test case batch property via weakref / 测试用例 batch property via weakref
```python
    def test_batch_property_via_weakref(self):
        """Test that batch property returns the original batch via weakref."""
        reqs = [_make_req()]
        batch = _make_batch(reqs)
        orch = BatchedPenalizerOrchestrator(VOCAB_SIZE, batch, set())
        self.assertIs(orch.batch, batch)
```
**EN:** Test that batch property returns the original batch via weakref. This test exercises `test_batch_property_via_weakref` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that batch property returns the original batch via weakref. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_batch_property_via_weakref`。

### Lines 80-86: test case batch setter none / 测试用例 batch setter none
```python
    def test_batch_setter_none(self):
        """Test that setting batch to None breaks the weakref cleanly."""
        reqs = [_make_req()]
        batch = _make_batch(reqs)
        orch = BatchedPenalizerOrchestrator(VOCAB_SIZE, batch, set())
        orch.batch = None
        self.assertIsNone(orch.batch)
```
**EN:** Test that setting batch to None breaks the weakref cleanly. This test exercises `test_batch_setter_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that setting batch to None breaks the weakref cleanly. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_batch_setter_none`。

### Lines 88-95: test case batch setter new batch / 测试用例 batch setter new batch
```python
    def test_batch_setter_new_batch(self):
        """Test that batch can be reassigned to a different ScheduleBatch."""
        reqs = [_make_req()]
        batch1 = _make_batch(reqs)
        batch2 = _make_batch(reqs)
        orch = BatchedPenalizerOrchestrator(VOCAB_SIZE, batch1, set())
        orch.batch = batch2
        self.assertIs(orch.batch, batch2)
```
**EN:** Test that batch can be reassigned to a different ScheduleBatch. This test exercises `test_batch_setter_new_batch` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that batch can be reassigned to a different ScheduleBatch. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_batch_setter_new_batch`。

### Lines 97-106: test case context manager releases / 测试用例 context manager releases
```python
    def test_context_manager_releases(self):
        """Test that exiting the context manager releases all penalizers."""
        reqs = [_make_req(freq=1.0)]
        batch = _make_batch(reqs)
        with BatchedPenalizerOrchestrator(
            VOCAB_SIZE, batch, {BatchedFrequencyPenalizer}
        ) as orch:
            self.assertTrue(orch.is_required)
        self.assertFalse(orch.is_required)
        self.assertEqual(len(orch.penalizers), 0)
```
**EN:** Test that exiting the context manager releases all penalizers. This test exercises `test_context_manager_releases` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that exiting the context manager releases all penalizers. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_context_manager_releases`。

### Lines 108-116: test case filter empty indices releases / 测试用例 filter empty indices releases
```python
    def test_filter_empty_indices_releases(self):
        """Test that filtering with no indices left fully releases the orchestrator."""
        reqs = [_make_req(freq=1.0)]
        batch = _make_batch(reqs)
        orch = BatchedPenalizerOrchestrator(
            VOCAB_SIZE, batch, {BatchedFrequencyPenalizer}
        )
        orch.filter(torch.tensor([], dtype=torch.long))
        self.assertFalse(orch.is_required)
```
**EN:** Test that filtering with no indices left fully releases the orchestrator. This test exercises `test_filter_empty_indices_releases` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that filtering with no indices left fully releases the orchestrator. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_filter_empty_indices_releases`。

### Lines 118-126: test case filter not required is noop / 测试用例 filter not required is noop
```python
    def test_filter_not_required_is_noop(self):
        """Test that filter on a not-required orchestrator does nothing."""
        reqs = [_make_req()]
        batch = _make_batch(reqs)
        orch = BatchedPenalizerOrchestrator(
            VOCAB_SIZE, batch, {BatchedFrequencyPenalizer}
        )
        self.assertFalse(orch.is_required)
        orch.filter(torch.tensor([0]))  # should not raise
```
**EN:** Test that filter on a not-required orchestrator does nothing. This test exercises `test_filter_not_required_is_noop` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that filter on a not-required orchestrator does nothing. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_filter_not_required_is_noop`。

### Lines 128-139: test case merge both not required is noop / 测试用例 merge both not required is noop
```python
    def test_merge_both_not_required_is_noop(self):
        """Test that merging two not-required orchestrators stays not-required."""
        reqs = [_make_req()]
        batch = _make_batch(reqs)
        orch1 = BatchedPenalizerOrchestrator(
            VOCAB_SIZE, batch, {BatchedFrequencyPenalizer}
        )
        orch2 = BatchedPenalizerOrchestrator(
            VOCAB_SIZE, batch, {BatchedFrequencyPenalizer}
        )
        orch1.merge(orch2)  # should not raise
        self.assertFalse(orch1.is_required)
```
**EN:** Test that merging two not-required orchestrators stays not-required. This test exercises `test_merge_both_not_required_is_noop` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that merging two not-required orchestrators stays not-required. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_merge_both_not_required_is_noop`。

### Lines 140-142: supporting source context / 辅助源码上下文
```python


# BatchedFrequencyPenalizer
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 143-144: class TestBatchedFrequencyPenalizer declaration / 类 TestBatchedFrequencyPenalizer 声明
```python
class TestBatchedFrequencyPenalizer(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 145-152: method setup / 方法 setup
```python
    def _setup(self, freq_values):
        reqs = [_make_req(freq=f) for f in freq_values]
        batch = _make_batch(reqs)
        orch = BatchedPenalizerOrchestrator(
            VOCAB_SIZE, batch, {BatchedFrequencyPenalizer}
        )
        pen = orch.penalizers[BatchedFrequencyPenalizer]
        return orch, pen
```
**EN:** This block implements `_setup` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_setup`，承担模块行为中的一个聚焦逻辑片段。

### Lines 154-157: test case is required with nonzero penalty / 测试用例 is required with nonzero penalty
```python
    def test_is_required_with_nonzero_penalty(self):
        """Test that nonzero frequency_penalty makes the penalizer required."""
        _, pen = self._setup([1.5])
        self.assertTrue(pen.is_required())
```
**EN:** Test that nonzero frequency_penalty makes the penalizer required. This test exercises `test_is_required_with_nonzero_penalty` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that nonzero frequency_penalty makes the penalizer required. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_is_required_with_nonzero_penalty`。

### Lines 159-162: test case is not required with zero penalty / 测试用例 is not required with zero penalty
```python
    def test_is_not_required_with_zero_penalty(self):
        """Test that zero frequency_penalty makes the penalizer not required."""
        _, pen = self._setup([0.0])
        self.assertFalse(pen.is_required())
```
**EN:** Test that zero frequency_penalty makes the penalizer not required. This test exercises `test_is_not_required_with_zero_penalty` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that zero frequency_penalty makes the penalizer not required. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_is_not_required_with_zero_penalty`。

### Lines 164-174: test case cumulate and apply / 测试用例 cumulate and apply
```python
    def test_cumulate_and_apply(self):
        """Test that cumulating a token applies frequency penalty to its logit."""
        orch, pen = self._setup([2.0])
        output_ids = torch.tensor([5])
        pen.cumulate_output_tokens(output_ids)

        logits = torch.zeros(1, VOCAB_SIZE)
        pen.apply(logits)
        self.assertAlmostEqual(logits[0, 5].item(), -2.0, places=5)
        # Other tokens unaffected
        self.assertAlmostEqual(logits[0, 0].item(), 0.0, places=5)
```
**EN:** Test that cumulating a token applies frequency penalty to its logit. This test exercises `test_cumulate_and_apply` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that cumulating a token applies frequency penalty to its logit. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_cumulate_and_apply`。

### Lines 176-184: test case cumulate twice doubles penalty / 测试用例 cumulate twice doubles penalty
```python
    def test_cumulate_twice_doubles_penalty(self):
        """Test that frequency penalty scales linearly with occurrence count."""
        orch, pen = self._setup([1.0])
        pen.cumulate_output_tokens(torch.tensor([3]))
        pen.cumulate_output_tokens(torch.tensor([3]))

        logits = torch.zeros(1, VOCAB_SIZE)
        pen.apply(logits)
        self.assertAlmostEqual(logits[0, 3].item(), -2.0, places=5)
```
**EN:** Test that frequency penalty scales linearly with occurrence count. This test exercises `test_cumulate_twice_doubles_penalty` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that frequency penalty scales linearly with occurrence count. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_cumulate_twice_doubles_penalty`。

### Lines 186-192: test case filter keeps subset / 测试用例 filter keeps subset
```python
    def test_filter_keeps_subset(self):
        """Test that filter retains only the selected batch indices."""
        orch, pen = self._setup([1.0, 2.0])
        keep = torch.tensor([1])
        pen.filter(keep)
        self.assertEqual(pen.frequency_penalties.shape[0], 1)
        self.assertAlmostEqual(pen.frequency_penalties[0, 0].item(), 2.0, places=5)
```
**EN:** Test that filter retains only the selected batch indices. This test exercises `test_filter_keeps_subset` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that filter retains only the selected batch indices. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_filter_keeps_subset`。

### Lines 194-199: test case merge concatenates / 测试用例 merge concatenates
```python
    def test_merge_concatenates(self):
        """Test that merge concatenates penalty tensors from two penalizers."""
        _, pen1 = self._setup([1.0])
        _, pen2 = self._setup([2.0])
        pen1.merge(pen2)
        self.assertEqual(pen1.frequency_penalties.shape[0], 2)
```
**EN:** Test that merge concatenates penalty tensors from two penalizers. This test exercises `test_merge_concatenates` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that merge concatenates penalty tensors from two penalizers. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_merge_concatenates`。

### Lines 201-207: test case teardown cleans attributes / 测试用例 teardown cleans attributes
```python
    def test_teardown_cleans_attributes(self):
        """Test that teardown deletes internal tensors and resets prepared state."""
        _, pen = self._setup([1.0])
        pen.teardown()
        self.assertFalse(hasattr(pen, "frequency_penalties"))
        self.assertFalse(hasattr(pen, "cumulated_frequency_penalties"))
        self.assertFalse(pen.is_prepared())
```
**EN:** Test that teardown deletes internal tensors and resets prepared state. This test exercises `test_teardown_cleans_attributes` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that teardown deletes internal tensors and resets prepared state. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_teardown_cleans_attributes`。

### Lines 209-213: test case cumulate when not prepared is noop / 测试用例 cumulate when not prepared is noop
```python
    def test_cumulate_when_not_prepared_is_noop(self):
        """Test that cumulate before prepare does not crash."""
        _, pen = self._setup([0.0])
        # pen is not prepared (is_required=False)
        pen.cumulate_output_tokens(torch.tensor([1]))  # should not raise
```
**EN:** Test that cumulate before prepare does not crash. This test exercises `test_cumulate_when_not_prepared_is_noop` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that cumulate before prepare does not crash. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_cumulate_when_not_prepared_is_noop`。

### Lines 215-221: test case apply when not prepared is noop / 测试用例 apply when not prepared is noop
```python
    def test_apply_when_not_prepared_is_noop(self):
        """Test that apply on an unprepared penalizer leaves logits unchanged."""
        _, pen = self._setup([0.0])
        logits = torch.zeros(1, VOCAB_SIZE)
        original = logits.clone()
        pen.apply(logits)
        self.assertTrue(torch.equal(logits, original))
```
**EN:** Test that apply on an unprepared penalizer leaves logits unchanged. This test exercises `test_apply_when_not_prepared_is_noop` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that apply on an unprepared penalizer leaves logits unchanged. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_apply_when_not_prepared_is_noop`。

### Lines 222-224: supporting source context / 辅助源码上下文
```python


# BatchedPresencePenalizer
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 225-226: class TestBatchedPresencePenalizer declaration / 类 TestBatchedPresencePenalizer 声明
```python
class TestBatchedPresencePenalizer(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 227-234: method setup / 方法 setup
```python
    def _setup(self, presence_values):
        reqs = [_make_req(presence=p) for p in presence_values]
        batch = _make_batch(reqs)
        orch = BatchedPenalizerOrchestrator(
            VOCAB_SIZE, batch, {BatchedPresencePenalizer}
        )
        pen = orch.penalizers[BatchedPresencePenalizer]
        return orch, pen
```
**EN:** This block implements `_setup` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_setup`，承担模块行为中的一个聚焦逻辑片段。

### Lines 236-239: test case is required with nonzero penalty / 测试用例 is required with nonzero penalty
```python
    def test_is_required_with_nonzero_penalty(self):
        """Test that nonzero presence_penalty makes the penalizer required."""
        _, pen = self._setup([0.5])
        self.assertTrue(pen.is_required())
```
**EN:** Test that nonzero presence_penalty makes the penalizer required. This test exercises `test_is_required_with_nonzero_penalty` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that nonzero presence_penalty makes the penalizer required. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_is_required_with_nonzero_penalty`。

### Lines 241-250: test case presence penalty does not scale / 测试用例 presence penalty does not scale
```python
    def test_presence_penalty_does_not_scale(self):
        """Test that presence penalty is flat (same value regardless of count)."""
        orch, pen = self._setup([1.0])
        pen.cumulate_output_tokens(torch.tensor([7]))
        pen.cumulate_output_tokens(torch.tensor([7]))  # same token again

        logits = torch.zeros(1, VOCAB_SIZE)
        pen.apply(logits)
        # scatter_ overwrites (not adds), so penalty should be 1.0, not 2.0
        self.assertAlmostEqual(logits[0, 7].item(), -1.0, places=5)
```
**EN:** Test that presence penalty is flat (same value regardless of count). This test exercises `test_presence_penalty_does_not_scale` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that presence penalty is flat (same value regardless of count). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_presence_penalty_does_not_scale`。

### Lines 252-258: test case filter keeps subset / 测试用例 filter keeps subset
```python
    def test_filter_keeps_subset(self):
        """Test that filter retains the first request's presence penalty."""
        orch, pen = self._setup([1.0, 2.0])
        keep = torch.tensor([0])
        pen.filter(keep)
        self.assertEqual(pen.presence_penalties.shape[0], 1)
        self.assertAlmostEqual(pen.presence_penalties[0, 0].item(), 1.0, places=5)
```
**EN:** Test that filter retains the first request's presence penalty. This test exercises `test_filter_keeps_subset` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that filter retains the first request's presence penalty. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_filter_keeps_subset`。

### Lines 260-265: test case merge concatenates / 测试用例 merge concatenates
```python
    def test_merge_concatenates(self):
        """Test that merge concatenates presence penalty tensors."""
        _, pen1 = self._setup([1.0])
        _, pen2 = self._setup([2.0])
        pen1.merge(pen2)
        self.assertEqual(pen1.presence_penalties.shape[0], 2)
```
**EN:** Test that merge concatenates presence penalty tensors. This test exercises `test_merge_concatenates` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that merge concatenates presence penalty tensors. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_merge_concatenates`。

### Lines 267-271: test case teardown cleans attributes / 测试用例 teardown cleans attributes
```python
    def test_teardown_cleans_attributes(self):
        """Test that teardown removes the presence_penalties tensor."""
        _, pen = self._setup([1.0])
        pen.teardown()
        self.assertFalse(hasattr(pen, "presence_penalties"))
```
**EN:** Test that teardown removes the presence_penalties tensor. This test exercises `test_teardown_cleans_attributes` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that teardown removes the presence_penalties tensor. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_teardown_cleans_attributes`。

### Lines 272-274: supporting source context / 辅助源码上下文
```python


# BatchedMinNewTokensPenalizer
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 275-276: class TestBatchedMinNewTokensPenalizer declaration / 类 TestBatchedMinNewTokensPenalizer 声明
```python
class TestBatchedMinNewTokensPenalizer(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 277-285: method setup / 方法 setup
```python
    def _setup(self, configs):
        """configs: list of (min_tokens, stop_ids, eos_id)."""
        reqs = [_make_req(min_tokens=c[0], stop_ids=c[1], eos_id=c[2]) for c in configs]
        batch = _make_batch(reqs)
        orch = BatchedPenalizerOrchestrator(
            VOCAB_SIZE, batch, {BatchedMinNewTokensPenalizer}
        )
        pen = orch.penalizers[BatchedMinNewTokensPenalizer]
        return orch, pen
```
**EN:** configs: list of (min_tokens, stop_ids, eos_id). This block implements `_setup` and captures one focused piece of the module's behavior.
**CN:** configs: list of (min_tokens, stop_ids, eos_id). 该代码块实现 `_setup`，承担模块行为中的一个聚焦逻辑片段。

### Lines 287-290: test case is required with positive min tokens / 测试用例 is required with positive min tokens
```python
    def test_is_required_with_positive_min_tokens(self):
        """Test that positive min_new_tokens makes the penalizer required."""
        _, pen = self._setup([(5, None, 2)])
        self.assertTrue(pen.is_required())
```
**EN:** Test that positive min_new_tokens makes the penalizer required. This test exercises `test_is_required_with_positive_min_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that positive min_new_tokens makes the penalizer required. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_is_required_with_positive_min_tokens`。

### Lines 292-295: test case is not required with zero min tokens / 测试用例 is not required with zero min tokens
```python
    def test_is_not_required_with_zero_min_tokens(self):
        """Test that min_new_tokens=0 makes the penalizer not required."""
        _, pen = self._setup([(0, None, 2)])
        self.assertFalse(pen.is_required())
```
**EN:** Test that min_new_tokens=0 makes the penalizer not required. This test exercises `test_is_not_required_with_zero_min_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that min_new_tokens=0 makes the penalizer not required. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_is_not_required_with_zero_min_tokens`。

### Lines 297-305: test case blocks eos before min tokens / 测试用例 blocks eos before min tokens
```python
    def test_blocks_eos_before_min_tokens(self):
        """Test that EOS token is blocked before min_new_tokens is reached."""
        orch, pen = self._setup([(3, None, 2)])
        # Before any output: len=0 < min=3 → block EOS (token 2)
        logits = torch.zeros(1, VOCAB_SIZE)
        pen.apply(logits)
        self.assertTrue(torch.isinf(logits[0, 2]) and logits[0, 2] < 0)
        # Non-stop tokens should be fine
        self.assertEqual(logits[0, 0].item(), 0.0)
```
**EN:** Test that EOS token is blocked before min_new_tokens is reached. This test exercises `test_blocks_eos_before_min_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that EOS token is blocked before min_new_tokens is reached. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_blocks_eos_before_min_tokens`。

### Lines 307-316: test case allows eos after min tokens / 测试用例 allows eos after min tokens
```python
    def test_allows_eos_after_min_tokens(self):
        """Test that EOS is allowed after generating min_new_tokens."""
        orch, pen = self._setup([(2, None, 2)])
        # Generate 2 tokens
        pen.cumulate_output_tokens(torch.tensor([10]))
        pen.cumulate_output_tokens(torch.tensor([11]))
        # Now len=2 >= min=2 → EOS should NOT be blocked
        logits = torch.zeros(1, VOCAB_SIZE)
        pen.apply(logits)
        self.assertEqual(logits[0, 2].item(), 0.0)
```
**EN:** Test that EOS is allowed after generating min_new_tokens. This test exercises `test_allows_eos_after_min_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that EOS is allowed after generating min_new_tokens. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_allows_eos_after_min_tokens`。

### Lines 318-326: test case blocks custom stop tokens / 测试用例 blocks custom stop tokens
```python
    def test_blocks_custom_stop_tokens(self):
        """Test that custom stop_token_ids are also blocked before min_new_tokens."""
        orch, pen = self._setup([(3, {5, 10}, 2)])
        logits = torch.zeros(1, VOCAB_SIZE)
        pen.apply(logits)
        # EOS (2), stop token 5, stop token 10 should all be blocked
        self.assertTrue(torch.isinf(logits[0, 2]) and logits[0, 2] < 0)
        self.assertTrue(torch.isinf(logits[0, 5]) and logits[0, 5] < 0)
        self.assertTrue(torch.isinf(logits[0, 10]) and logits[0, 10] < 0)
```
**EN:** Test that custom stop_token_ids are also blocked before min_new_tokens. This test exercises `test_blocks_custom_stop_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that custom stop_token_ids are also blocked before min_new_tokens. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_blocks_custom_stop_tokens`。

### Lines 328-347: test case blocks additional stop tokens / 测试用例 blocks additional stop tokens
```python
    def test_blocks_additional_stop_tokens(self):
        """Test that tokenizer's additional_stop_token_ids are also blocked."""
        req = _make_req(min_tokens=3, stop_ids=None, eos_id=2)
        req.tokenizer.additional_stop_token_ids = {7, 8}
        batch = _make_batch([req])
        orch = BatchedPenalizerOrchestrator(
            VOCAB_SIZE, batch, {BatchedMinNewTokensPenalizer}
        )
        pen = orch.penalizers[BatchedMinNewTokensPenalizer]

        logits = torch.zeros(1, VOCAB_SIZE)
        pen.apply(logits)
        # EOS (2) + additional stops (7, 8) should all be blocked
        for tok in [2, 7, 8]:
            self.assertTrue(
                torch.isinf(logits[0, tok]) and logits[0, tok] < 0,
                f"token {tok} should be blocked before min_new_tokens",
            )
        # Non-stop tokens should be fine
        self.assertEqual(logits[0, 0].item(), 0.0)
```
**EN:** Test that tokenizer's additional_stop_token_ids are also blocked. This test exercises `test_blocks_additional_stop_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that tokenizer's additional_stop_token_ids are also blocked. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_blocks_additional_stop_tokens`。

### Lines 349-355: test case filter keeps subset / 测试用例 filter keeps subset
```python
    def test_filter_keeps_subset(self):
        """Test that filter keeps the second request (min_tokens=5) and drops the first."""
        orch, pen = self._setup([(3, None, 2), (5, None, 2)])
        keep = torch.tensor([1])
        pen.filter(keep)
        self.assertEqual(pen.min_new_tokens.shape[0], 1)
        self.assertEqual(pen.min_new_tokens[0, 0].item(), 5)
```
**EN:** Test that filter keeps the second request (min_tokens=5) and drops the first. This test exercises `test_filter_keeps_subset` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that filter keeps the second request (min_tokens=5) and drops the first. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_filter_keeps_subset`。

### Lines 357-362: test case merge concatenates / 测试用例 merge concatenates
```python
    def test_merge_concatenates(self):
        """Test that merge combines min_new_tokens tensors from two penalizers."""
        _, pen1 = self._setup([(3, None, 2)])
        _, pen2 = self._setup([(5, None, 2)])
        pen1.merge(pen2)
        self.assertEqual(pen1.min_new_tokens.shape[0], 2)
```
**EN:** Test that merge combines min_new_tokens tensors from two penalizers. This test exercises `test_merge_concatenates` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that merge combines min_new_tokens tensors from two penalizers. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_merge_concatenates`。

### Lines 364-370: test case teardown cleans attributes / 测试用例 teardown cleans attributes
```python
    def test_teardown_cleans_attributes(self):
        """Test that teardown removes min_new_tokens, stop_token_penalties, and len_output_tokens."""
        _, pen = self._setup([(3, None, 2)])
        pen.teardown()
        self.assertFalse(hasattr(pen, "min_new_tokens"))
        self.assertFalse(hasattr(pen, "stop_token_penalties"))
        self.assertFalse(hasattr(pen, "len_output_tokens"))
```
**EN:** Test that teardown removes min_new_tokens, stop_token_penalties, and len_output_tokens. This test exercises `test_teardown_cleans_attributes` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that teardown removes min_new_tokens, stop_token_penalties, and len_output_tokens. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_teardown_cleans_attributes`。

### Lines 371-373: supporting source context / 辅助源码上下文
```python


# _BatchedPenalizer base class edge cases
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 374-375: class TestBatchedPenalizerBase declaration / 类 TestBatchedPenalizerBase 声明
```python
class TestBatchedPenalizerBase(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 376-385: test case filter when not prepared is noop / 测试用例 filter when not prepared is noop
```python
    def test_filter_when_not_prepared_is_noop(self):
        """Test that filter on an unprepared penalizer does not crash."""
        reqs = [_make_req()]
        batch = _make_batch(reqs)
        orch = BatchedPenalizerOrchestrator(
            VOCAB_SIZE, batch, {BatchedFrequencyPenalizer}
        )
        pen = orch.penalizers[BatchedFrequencyPenalizer]
        # pen is not prepared (frequency_penalty=0 → not required)
        pen.filter(torch.tensor([0]))  # should not raise
```
**EN:** Test that filter on an unprepared penalizer does not crash. This test exercises `test_filter_when_not_prepared_is_noop` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that filter on an unprepared penalizer does not crash. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_filter_when_not_prepared_is_noop`。

### Lines 387-406: test case merge prepares both if needed / 测试用例 merge prepares both if needed
```python
    def test_merge_prepares_both_if_needed(self):
        """Test that merge prepares unprepared side before concatenating."""
        reqs_a = [_make_req(freq=0.0)]  # not required
        reqs_b = [_make_req(freq=1.0)]  # required
        batch_a = _make_batch(reqs_a)
        batch_b = _make_batch(reqs_b)
        orch_a = BatchedPenalizerOrchestrator(
            VOCAB_SIZE, batch_a, {BatchedFrequencyPenalizer}
        )
        orch_b = BatchedPenalizerOrchestrator(
            VOCAB_SIZE, batch_b, {BatchedFrequencyPenalizer}
        )
        pen_a = orch_a.penalizers[BatchedFrequencyPenalizer]
        pen_b = orch_b.penalizers[BatchedFrequencyPenalizer]
        self.assertFalse(pen_a.is_prepared())
        self.assertTrue(pen_b.is_prepared())
        # Merge should prepare pen_a first
        pen_a.merge(pen_b)
        self.assertTrue(pen_a.is_prepared())
        self.assertEqual(pen_a.frequency_penalties.shape[0], 2)
```
**EN:** Test that merge prepares unprepared side before concatenating. This test exercises `test_merge_prepares_both_if_needed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that merge prepares unprepared side before concatenating. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_merge_prepares_both_if_needed`。

### Lines 408-421: test case merge both unprepared is noop / 测试用例 merge both unprepared is noop
```python
    def test_merge_both_unprepared_is_noop(self):
        """Test that merging two unprepared penalizers keeps them unprepared."""
        reqs = [_make_req()]
        batch = _make_batch(reqs)
        orch1 = BatchedPenalizerOrchestrator(
            VOCAB_SIZE, batch, {BatchedFrequencyPenalizer}
        )
        orch2 = BatchedPenalizerOrchestrator(
            VOCAB_SIZE, batch, {BatchedFrequencyPenalizer}
        )
        pen1 = orch1.penalizers[BatchedFrequencyPenalizer]
        pen2 = orch2.penalizers[BatchedFrequencyPenalizer]
        pen1.merge(pen2)  # both not prepared → noop
        self.assertFalse(pen1.is_prepared())
```
**EN:** Test that merging two unprepared penalizers keeps them unprepared. This test exercises `test_merge_both_unprepared_is_noop` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that merging two unprepared penalizers keeps them unprepared. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_merge_both_unprepared_is_noop`。

### Lines 423-434: test case prepare is idempotent / 测试用例 prepare is idempotent
```python
    def test_prepare_is_idempotent(self):
        """Test that calling prepare() multiple times does not crash."""
        reqs = [_make_req(freq=1.0)]
        batch = _make_batch(reqs)
        orch = BatchedPenalizerOrchestrator(
            VOCAB_SIZE, batch, {BatchedFrequencyPenalizer}
        )
        pen = orch.penalizers[BatchedFrequencyPenalizer]
        self.assertTrue(pen.is_prepared())
        # Calling prepare again should not crash or reinitialize
        pen.prepare()
        self.assertTrue(pen.is_prepared())
```
**EN:** Test that calling prepare() multiple times does not crash. This test exercises `test_prepare_is_idempotent` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that calling prepare() multiple times does not crash. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prepare_is_idempotent`。

### Lines 435-437: supporting source context / 辅助源码上下文
```python


# Orchestrator with multiple penalizer types
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 438-439: class TestOrchestratorMultiplePenalizers declaration / 类 TestOrchestratorMultiplePenalizers 声明
```python
class TestOrchestratorMultiplePenalizers(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 440-466: test case all three penalizers / 测试用例 all three penalizers
```python
    def test_all_three_penalizers(self):
        """Test orchestrator managing frequency, presence, and min_new_tokens together."""
        reqs = [_make_req(freq=1.0, presence=0.5, min_tokens=2, eos_id=2)]
        batch = _make_batch(reqs)
        orch = BatchedPenalizerOrchestrator(
            VOCAB_SIZE,
            batch,
            {
                BatchedFrequencyPenalizer,
                BatchedPresencePenalizer,
                BatchedMinNewTokensPenalizer,
            },
        )
        self.assertTrue(orch.is_required)

        # Cumulate one token
        output_ids = torch.tensor([5])
        orch.cumulate_output_tokens(output_ids)

        # Apply all penalties
        logits = torch.zeros(1, VOCAB_SIZE)
        orch.apply(logits)

        # Token 5: freq_penalty=1.0 (cumulated once) + pres_penalty=0.5
        self.assertAlmostEqual(logits[0, 5].item(), -1.5, places=4)
        # EOS (token 2): blocked by min_new_tokens (len=1 < min=2)
        self.assertTrue(torch.isinf(logits[0, 2]) and logits[0, 2] < 0)
```
**EN:** Test orchestrator managing frequency, presence, and min_new_tokens together. This test exercises `test_all_three_penalizers` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test orchestrator managing frequency, presence, and min_new_tokens together. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_three_penalizers`。

### Lines 468-483: test case filter with penalizer no longer required / 测试用例 filter with penalizer no longer required
```python
    def test_filter_with_penalizer_no_longer_required(self):
        """Test that penalizer is torn down when no longer required after filter."""
        reqs = [_make_req(freq=0.0), _make_req(freq=1.0)]
        batch = _make_batch(reqs)
        orch = BatchedPenalizerOrchestrator(
            VOCAB_SIZE, batch, {BatchedFrequencyPenalizer}
        )
        self.assertTrue(orch.is_required)

        # Keep only the request with freq=0 (index 0)
        batch.reqs = [reqs[0]]
        orch.filter(torch.tensor([0]))

        pen = orch.penalizers[BatchedFrequencyPenalizer]
        # After filter, only req with freq=0 remains → penalizer not required
        self.assertFalse(pen.is_required())
```
**EN:** Test that penalizer is torn down when no longer required after filter. This test exercises `test_filter_with_penalizer_no_longer_required` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that penalizer is torn down when no longer required after filter. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_filter_with_penalizer_no_longer_required`。

### Lines 485-496: test case filter keeps required penalizer / 测试用例 filter keeps required penalizer
```python
    def test_filter_keeps_required_penalizer(self):
        """Test that filter keeps penalizer active when still required."""
        reqs = [_make_req(freq=1.0), _make_req(freq=2.0)]
        batch = _make_batch(reqs)
        orch = BatchedPenalizerOrchestrator(
            VOCAB_SIZE, batch, {BatchedFrequencyPenalizer}
        )
        self.assertTrue(orch.is_required)

        batch.reqs = [reqs[1]]
        orch.filter(torch.tensor([1]))
        self.assertTrue(orch.is_required)
```
**EN:** Test that filter keeps penalizer active when still required. This test exercises `test_filter_keeps_required_penalizer` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that filter keeps penalizer active when still required. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_filter_keeps_required_penalizer`。

### Lines 498-516: test case merge one required / 测试用例 merge one required
```python
    def test_merge_one_required(self):
        """Test that merge marks orchestrator as required when one side is."""
        reqs_a = [_make_req(freq=0.0)]
        reqs_b = [_make_req(freq=1.0)]
        batch_a = _make_batch(reqs_a)
        batch_b = _make_batch(reqs_b)
        orch_a = BatchedPenalizerOrchestrator(
            VOCAB_SIZE, batch_a, {BatchedFrequencyPenalizer}
        )
        orch_b = BatchedPenalizerOrchestrator(
            VOCAB_SIZE, batch_b, {BatchedFrequencyPenalizer}
        )
        self.assertFalse(orch_a.is_required)
        self.assertTrue(orch_b.is_required)

        orch_a.merge(orch_b)
        self.assertTrue(orch_a.is_required)
        pen = orch_a.penalizers[BatchedFrequencyPenalizer]
        self.assertEqual(pen.frequency_penalties.shape[0], 2)
```
**EN:** Test that merge marks orchestrator as required when one side is. This test exercises `test_merge_one_required` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that merge marks orchestrator as required when one side is. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_merge_one_required`。

### Lines 519-520: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_make_req`: Create a mock request with sampling params. / 该代码块实现 `_make_req`，承担模块行为中的一个聚焦逻辑片段。
- `_make_batch`: Create a mock ScheduleBatch. / 该代码块实现 `_make_batch`，承担模块行为中的一个聚焦逻辑片段。
- `TestBatchedPenalizerOrchestrator`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestBatchedFrequencyPenalizer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestBatchedPresencePenalizer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestBatchedMinNewTokensPenalizer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestBatchedPenalizerBase`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestOrchestratorMultiplePenalizers`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestBatchedPenalizerOrchestrator.test_init_detects_required_penalizers`: Test that orchestrator marks is_required=True when any request has nonzero penalty. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_detects_required_penalizers`。
- `TestBatchedPenalizerOrchestrator.test_init_not_required_when_no_penalties`: Test that orchestrator marks is_required=False when all penalties are zero. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_not_required_when_no_penalties`。
- `TestBatchedPenalizerOrchestrator.test_batch_property_via_weakref`: Test that batch property returns the original batch via weakref. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_batch_property_via_weakref`。
- `TestBatchedPenalizerOrchestrator.test_batch_setter_none`: Test that setting batch to None breaks the weakref cleanly. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_batch_setter_none`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `unittest.mock`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.srt.sampling.penaltylib.frequency_penalty`, `sglang.srt.sampling.penaltylib.min_new_tokens`, `sglang.srt.sampling.penaltylib.orchestrator`, `sglang.srt.sampling.penaltylib.presence_penalty`, `sglang.test.test_utils`

- **Total lines / 总行数**: 520
