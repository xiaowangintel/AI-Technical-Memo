# test_adaptive_spec_params.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/spec/test_adaptive_spec_params.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates adaptive spec params behavior in SGLang's unit / spec area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 推测解码 领域中与 adaptive spec params 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.srt.speculative.adaptive_spec_params import AdaptiveSpeculativeParams
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.srt.speculative.adaptive_spec_params`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.srt.speculative.adaptive_spec_params`, `sglang.test.ci.ci_register`。

### Lines 6-6: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=6, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 9-9: class TestAdaptiveSpeculativeParams declaration / 类 TestAdaptiveSpeculativeParams 声明
```python
class TestAdaptiveSpeculativeParams(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 10-18: test case initial steps added to candidates when missing / 测试用例 initial steps added to candidates when missing
```python
    def test_initial_steps_added_to_candidates_when_missing(self):
        params = AdaptiveSpeculativeParams(
            initial_steps=2,
            config={"candidate_steps": [1, 3, 7]},
        )

        self.assertEqual(params.candidate_steps, [1, 2, 3, 7])
        self.assertEqual(params.current_steps, 2)
        self.assertEqual(params.ema_accept_len, 1.0)
```
**EN:** This test exercises `test_initial_steps_added_to_candidates_when_missing` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_initial_steps_added_to_candidates_when_missing`。

### Lines 20-38: test case update respects warmup and interval / 测试用例 update respects warmup and interval
```python
    def test_update_respects_warmup_and_interval(self):
        params = AdaptiveSpeculativeParams(
            initial_steps=3,
            config={
                "candidate_steps": [1, 3, 7],
                "ema_alpha": 1.0,
                "warmup_batches": 1,
                "update_interval": 2,
            },
        )

        self.assertFalse(params.update([0, 0]))
        self.assertEqual(params.current_steps, 3)

        self.assertFalse(params.update([0, 0]))
        self.assertEqual(params.current_steps, 3)

        self.assertTrue(params.update([0, 0]))
        self.assertEqual(params.current_steps, 1)
```
**EN:** This test exercises `test_update_respects_warmup_and_interval` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_respects_warmup_and_interval`。

### Lines 40-59: test case empty batches do not consume warmup or shift steps / 测试用例 empty batches do not consume warmup or shift steps
```python
    def test_empty_batches_do_not_consume_warmup_or_shift_steps(self):
        params = AdaptiveSpeculativeParams(
            initial_steps=3,
            config={
                "candidate_steps": [1, 3, 7],
                "ema_alpha": 1.0,
                "warmup_batches": 1,
                "update_interval": 1,
            },
        )

        self.assertFalse(params.update([]))
        self.assertEqual(params.current_steps, 3)
        self.assertEqual(params.ema_accept_len, 2.0)

        self.assertFalse(params.update([0, 0]))
        self.assertEqual(params.current_steps, 3)

        self.assertTrue(params.update([0, 0]))
        self.assertEqual(params.current_steps, 1)
```
**EN:** This test exercises `test_empty_batches_do_not_consume_warmup_or_shift_steps` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_batches_do_not_consume_warmup_or_shift_steps`。

### Lines 61-77: test case update scales up across candidates / 测试用例 update scales up across candidates
```python
    def test_update_scales_up_across_candidates(self):
        params = AdaptiveSpeculativeParams(
            initial_steps=1,
            config={
                "candidate_steps": [1, 3, 7],
                "ema_alpha": 1.0,
                "warmup_batches": 0,
                "update_interval": 1,
                "up_hysteresis": 0.0,
            },
        )

        self.assertTrue(params.update([1, 1]))
        self.assertEqual(params.current_steps, 3)

        self.assertTrue(params.update([3, 3]))
        self.assertEqual(params.current_steps, 7)
```
**EN:** This test exercises `test_update_scales_up_across_candidates` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_scales_up_across_candidates`。

### Lines 79-91: test case update can scale down across candidates in one recompute / 测试用例 update can scale down across candidates in one recompute
```python
    def test_update_can_scale_down_across_candidates_in_one_recompute(self):
        params = AdaptiveSpeculativeParams(
            initial_steps=7,
            config={
                "candidate_steps": [1, 3, 7],
                "ema_alpha": 1.0,
                "warmup_batches": 0,
                "update_interval": 1,
            },
        )

        self.assertTrue(params.update([0, 0]))
        self.assertEqual(params.current_steps, 1)
```
**EN:** This test exercises `test_update_can_scale_down_across_candidates_in_one_recompute` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_can_scale_down_across_candidates_in_one_recompute`。

### Lines 93-110: test case exact rise threshold does not upshift / 测试用例 exact rise threshold does not upshift
```python
    def test_exact_rise_threshold_does_not_upshift(self):
        params = AdaptiveSpeculativeParams(
            initial_steps=3,
            config={
                "candidate_steps": [1, 3, 7],
                "ema_alpha": 1.0,
                "warmup_batches": 0,
                "update_interval": 1,
                "up_hysteresis": 0.0,
            },
        )

        self.assertFalse(params.update([2, 3]))
        self.assertEqual(params.current_steps, 3)
        self.assertEqual(params.ema_accept_len, 2.5)

        self.assertTrue(params.update([3, 3]))
        self.assertEqual(params.current_steps, 7)
```
**EN:** This test exercises `test_exact_rise_threshold_does_not_upshift` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_exact_rise_threshold_does_not_upshift`。

### Lines 112-127: test case exact drop threshold does downshift / 测试用例 exact drop threshold does downshift
```python
    def test_exact_drop_threshold_does_downshift(self):
        params = AdaptiveSpeculativeParams(
            initial_steps=3,
            config={
                "candidate_steps": [1, 3, 7],
                "ema_alpha": 1.0,
                "warmup_batches": 0,
                "update_interval": 1,
                "down_hysteresis": 0.0,
                "up_hysteresis": 0.5,
            },
        )

        self.assertTrue(params.update([0, 1]))
        self.assertEqual(params.current_steps, 1)
        self.assertEqual(params.ema_accept_len, 0.5)
```
**EN:** This test exercises `test_exact_drop_threshold_does_downshift` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_exact_drop_threshold_does_downshift`。

### Lines 129-145: test case hysteresis can prevent premature upshift / 测试用例 hysteresis can prevent premature upshift
```python
    def test_hysteresis_can_prevent_premature_upshift(self):
        params = AdaptiveSpeculativeParams(
            initial_steps=3,
            config={
                "candidate_steps": [1, 3, 7],
                "ema_alpha": 1.0,
                "warmup_batches": 0,
                "update_interval": 1,
                "up_hysteresis": 0.75,
            },
        )

        self.assertFalse(params.update([3, 3]))
        self.assertEqual(params.current_steps, 3)

        self.assertTrue(params.update([4, 4]))
        self.assertEqual(params.current_steps, 7)
```
**EN:** This test exercises `test_hysteresis_can_prevent_premature_upshift` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hysteresis_can_prevent_premature_upshift`。

### Lines 147-163: test case down hysteresis can prevent premature downshift / 测试用例 down hysteresis can prevent premature downshift
```python
    def test_down_hysteresis_can_prevent_premature_downshift(self):
        params = AdaptiveSpeculativeParams(
            initial_steps=7,
            config={
                "candidate_steps": [1, 3, 7],
                "ema_alpha": 1.0,
                "warmup_batches": 0,
                "update_interval": 1,
                "down_hysteresis": -0.75,
            },
        )

        self.assertFalse(params.update([2, 2]))
        self.assertEqual(params.current_steps, 7)

        self.assertTrue(params.update([1, 1]))
        self.assertEqual(params.current_steps, 3)
```
**EN:** This test exercises `test_down_hysteresis_can_prevent_premature_downshift` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_down_hysteresis_can_prevent_premature_downshift`。

### Lines 165-192: test case multi batch sequence can ramp up then back down / 测试用例 multi batch sequence can ramp up then back down
```python
    def test_multi_batch_sequence_can_ramp_up_then_back_down(self):
        params = AdaptiveSpeculativeParams(
            initial_steps=3,
            config={
                "candidate_steps": [1, 3, 7],
                "ema_alpha": 0.5,
                "warmup_batches": 0,
                "update_interval": 1,
                "up_hysteresis": 0.0,
                "down_hysteresis": 0.0,
            },
        )

        self.assertTrue(params.update([4, 4]))
        self.assertEqual(params.current_steps, 7)
        self.assertEqual(params.ema_accept_len, 3.0)

        self.assertTrue(params.update([0, 0]))
        self.assertEqual(params.current_steps, 3)
        self.assertEqual(params.ema_accept_len, 1.5)

        self.assertFalse(params.update([0, 0]))
        self.assertEqual(params.current_steps, 3)
        self.assertEqual(params.ema_accept_len, 0.75)

        self.assertTrue(params.update([0, 0]))
        self.assertEqual(params.current_steps, 1)
        self.assertEqual(params.ema_accept_len, 0.375)
```
**EN:** This test exercises `test_multi_batch_sequence_can_ramp_up_then_back_down` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multi_batch_sequence_can_ramp_up_then_back_down`。

### Lines 195-196: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestAdaptiveSpeculativeParams`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestAdaptiveSpeculativeParams.test_initial_steps_added_to_candidates_when_missing`: This test exercises `test_initial_steps_added_to_candidates_when_missing` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_initial_steps_added_to_candidates_when_missing`。
- `TestAdaptiveSpeculativeParams.test_update_respects_warmup_and_interval`: This test exercises `test_update_respects_warmup_and_interval` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_respects_warmup_and_interval`。
- `TestAdaptiveSpeculativeParams.test_empty_batches_do_not_consume_warmup_or_shift_steps`: This test exercises `test_empty_batches_do_not_consume_warmup_or_shift_steps` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_batches_do_not_consume_warmup_or_shift_steps`。
- `TestAdaptiveSpeculativeParams.test_update_scales_up_across_candidates`: This test exercises `test_update_scales_up_across_candidates` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_scales_up_across_candidates`。
- `TestAdaptiveSpeculativeParams.test_update_can_scale_down_across_candidates_in_one_recompute`: This test exercises `test_update_can_scale_down_across_candidates_in_one_recompute` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_can_scale_down_across_candidates_in_one_recompute`。
- `TestAdaptiveSpeculativeParams.test_exact_rise_threshold_does_not_upshift`: This test exercises `test_exact_rise_threshold_does_not_upshift` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_exact_rise_threshold_does_not_upshift`。
- `TestAdaptiveSpeculativeParams.test_exact_drop_threshold_does_downshift`: This test exercises `test_exact_drop_threshold_does_downshift` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_exact_drop_threshold_does_downshift`。
- `TestAdaptiveSpeculativeParams.test_hysteresis_can_prevent_premature_upshift`: This test exercises `test_hysteresis_can_prevent_premature_upshift` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hysteresis_can_prevent_premature_upshift`。
- `TestAdaptiveSpeculativeParams.test_down_hysteresis_can_prevent_premature_downshift`: This test exercises `test_down_hysteresis_can_prevent_premature_downshift` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_down_hysteresis_can_prevent_premature_downshift`。
- `TestAdaptiveSpeculativeParams.test_multi_batch_sequence_can_ramp_up_then_back_down`: This test exercises `test_multi_batch_sequence_can_ramp_up_then_back_down` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multi_batch_sequence_can_ramp_up_then_back_down`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.srt.speculative.adaptive_spec_params`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 196
