# test_custom_logit_processor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/sampling/test_custom_logit_processor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates custom logit processor behavior in SGLang's unit / sampling area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / sampling 领域中与 custom logit processor 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for srt/sampling/custom_logit_processor.py — no server, no model loading."""
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
register_cpu_ci(est_time=7, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 7-21: module imports and dependencies / 模块导入与依赖
```python
import json
import unittest
from unittest.mock import MagicMock

import torch

from sglang.srt.sampling.custom_logit_processor import (
    CustomLogitProcessor,
    DeepseekOCRNoRepeatNGramLogitProcessor,
    DeepSeekR1ThinkingBudgetLogitProcessor,
    DisallowedTokensLogitsProcessor,
    Qwen3ThinkingBudgetLogitProcessor,
    _cache_from_str,
)
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `unittest`, `unittest.mock`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `unittest`, `unittest.mock`, `torch`。

### Lines 22-24: supporting source context / 辅助源码上下文
```python


# Helper: mock a Req object (used by ThinkingBudget and NGram processors)
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 25-29: function make req / 函数 make req
```python
def _make_req(origin_input_ids=None, output_ids=None):
    req = MagicMock()
    req.origin_input_ids = origin_input_ids or []
    req.output_ids = output_ids or []
    return req
```
**EN:** This block implements `_make_req` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_req`，承担模块行为中的一个聚焦逻辑片段。

### Lines 30-32: supporting source context / 辅助源码上下文
```python


# Serialization round-trip
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 33-34: class TestCustomLogitProcessorSerialization declaration / 类 TestCustomLogitProcessorSerialization 声明
```python
class TestCustomLogitProcessorSerialization(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 35-40: test case to str produces valid json / 测试用例 to str produces valid json
```python
    def test_to_str_produces_valid_json(self):
        """Test that to_str() produces valid JSON with a 'callable' field."""
        s = DisallowedTokensLogitsProcessor.to_str()
        data = json.loads(s)
        self.assertIn("callable", data)
        self.assertIsInstance(data["callable"], str)
```
**EN:** Test that to_str() produces valid JSON with a 'callable' field. This test exercises `test_to_str_produces_valid_json` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that to_str() produces valid JSON with a 'callable' field. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_to_str_produces_valid_json`。

### Lines 42-46: test case round trip serialization / 测试用例 round trip serialization
```python
    def test_round_trip_serialization(self):
        """Test serialize then deserialize produces a usable processor."""
        s = DisallowedTokensLogitsProcessor.to_str()
        processor = CustomLogitProcessor.from_str(s)
        self.assertIsInstance(processor, DisallowedTokensLogitsProcessor)
```
**EN:** Test serialize then deserialize produces a usable processor. This test exercises `test_round_trip_serialization` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test serialize then deserialize produces a usable processor. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_round_trip_serialization`。

### Lines 48-54: test case from str is cached / 测试用例 from str is cached
```python
    def test_from_str_is_cached(self):
        """Test that from_str uses LRU cache for repeated calls."""
        _cache_from_str.cache_clear()
        s = DisallowedTokensLogitsProcessor.to_str()
        cls1 = _cache_from_str(s)
        cls2 = _cache_from_str(s)
        self.assertIs(cls1, cls2)
```
**EN:** Test that from_str uses LRU cache for repeated calls. This test exercises `test_from_str_is_cached` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that from_str uses LRU cache for repeated calls. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_from_str_is_cached`。

### Lines 55-57: supporting source context / 辅助源码上下文
```python


# DisallowedTokensLogitsProcessor
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 58-58: class TestDisallowedTokensLogitsProcessor declaration / 类 TestDisallowedTokensLogitsProcessor 声明
```python
class TestDisallowedTokensLogitsProcessor(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 59-60: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.processor = DisallowedTokensLogitsProcessor()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 62-69: test case disallowed tokens set to neg inf / 测试用例 disallowed tokens set to neg inf
```python
    def test_disallowed_tokens_set_to_neg_inf(self):
        """Test that disallowed token positions are set to -inf for all batch items."""
        logits = torch.zeros(2, 10)
        params = [{"token_ids": [2, 5]}, {"token_ids": [2, 5]}]
        result = self.processor(logits, params)
        self.assertTrue(torch.isinf(result[0, 2]) and result[0, 2] < 0)
        self.assertTrue(torch.isinf(result[0, 5]) and result[0, 5] < 0)
        self.assertTrue(torch.isinf(result[1, 2]) and result[1, 2] < 0)
```
**EN:** Test that disallowed token positions are set to -inf for all batch items. This test exercises `test_disallowed_tokens_set_to_neg_inf` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that disallowed token positions are set to -inf for all batch items. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_disallowed_tokens_set_to_neg_inf`。

### Lines 71-78: test case allowed tokens unchanged / 测试用例 allowed tokens unchanged
```python
    def test_allowed_tokens_unchanged(self):
        """Test that non-disallowed tokens keep their original logit values."""
        logits = torch.ones(1, 10)
        params = [{"token_ids": [3]}]
        result = self.processor(logits, params)
        self.assertEqual(result[0, 0].item(), 1.0)
        self.assertEqual(result[0, 4].item(), 1.0)
        self.assertTrue(torch.isinf(result[0, 3]) and result[0, 3] < 0)
```
**EN:** Test that non-disallowed tokens keep their original logit values. This test exercises `test_allowed_tokens_unchanged` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that non-disallowed tokens keep their original logit values. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_allowed_tokens_unchanged`。

### Lines 80-85: test case mismatched params raises / 测试用例 mismatched params raises
```python
    def test_mismatched_params_raises(self):
        """Test that mismatched token_ids across batch items raises AssertionError."""
        logits = torch.zeros(2, 10)
        params = [{"token_ids": [1, 2]}, {"token_ids": [3, 4]}]
        with self.assertRaises(AssertionError):
            self.processor(logits, params)
```
**EN:** Test that mismatched token_ids across batch items raises AssertionError. This test exercises `test_mismatched_params_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that mismatched token_ids across batch items raises AssertionError. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mismatched_params_raises`。

### Lines 86-88: supporting source context / 辅助源码上下文
```python


# ThinkingBudgetLogitProcessor (using Qwen3 variant)
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 89-89: class TestThinkingBudgetLogitProcessor declaration / 类 TestThinkingBudgetLogitProcessor 声明
```python
class TestThinkingBudgetLogitProcessor(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 90-101: class-level constants and configuration for `TestThinkingBudgetLogitProcessor` / 类级常量与配置
```python
    """Test thinking budget enforcement using Qwen3 token IDs.

    Qwen3 tokens:
        THINKING_START = 151667
        THINKING_END   = 151668
        NEW_LINE       = 198
    """

    START = Qwen3ThinkingBudgetLogitProcessor.THINKING_START_TOKEN_ID
    END = Qwen3ThinkingBudgetLogitProcessor.THINKING_END_TOKEN_ID
    NL = Qwen3ThinkingBudgetLogitProcessor.NEW_LINE_TOKEN_ID
    VOCAB = 200000
```
**EN:** This block defines shared names such as `START`, `END`, `NL`, `VOCAB`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `START`, `END`, `NL`, `VOCAB` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 103-104: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.processor = Qwen3ThinkingBudgetLogitProcessor()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 106-107: method logits / 方法 logits
```python
    def _logits(self, batch_size=1):
        return torch.zeros(batch_size, self.VOCAB)
```
**EN:** This block implements `_logits` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_logits`，承担模块行为中的一个聚焦逻辑片段。

### Lines 109-118: test case budget not exceeded no change / 测试用例 budget not exceeded no change
```python
    def test_budget_not_exceeded_no_change(self):
        """Test no modification when thinking tokens are within budget."""
        req = _make_req(
            origin_input_ids=[self.START],
            output_ids=[100, 101],  # 2 tokens after start
        )
        params = [{"thinking_budget": 10, "__req__": req}]
        logits = self._logits()
        result = self.processor(logits, params)
        self.assertEqual(result[0, 0].item(), 0.0)  # unchanged
```
**EN:** Test no modification when thinking tokens are within budget. This test exercises `test_budget_not_exceeded_no_change` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test no modification when thinking tokens are within budget. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_budget_not_exceeded_no_change`。

### Lines 120-131: test case budget exceeded forces newline first / 测试用例 budget exceeded forces newline first
```python
    def test_budget_exceeded_forces_newline_first(self):
        """Test forcing newline when budget exceeded and last token is not newline."""
        req = _make_req(
            origin_input_ids=[self.START],
            output_ids=[100] * 5,  # 5 tokens, budget=5 → exceeded
        )
        params = [{"thinking_budget": 5, "__req__": req}]
        logits = self._logits()
        result = self.processor(logits, params)
        # newline should be the only non-neg-inf token
        self.assertEqual(result[0, self.NL].item(), 0.0)
        self.assertTrue(torch.isinf(result[0, 0]) and result[0, 0] < 0)
```
**EN:** Test forcing newline when budget exceeded and last token is not newline. This test exercises `test_budget_exceeded_forces_newline_first` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test forcing newline when budget exceeded and last token is not newline. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_budget_exceeded_forces_newline_first`。

### Lines 133-143: test case budget exceeded with newline forces end token / 测试用例 budget exceeded with newline forces end token
```python
    def test_budget_exceeded_with_newline_forces_end_token(self):
        """Test forcing end token when budget exceeded and last token is newline."""
        req = _make_req(
            origin_input_ids=[self.START],
            output_ids=[100] * 5 + [self.NL],  # 6 tokens, last is newline
        )
        params = [{"thinking_budget": 5, "__req__": req}]
        logits = self._logits()
        result = self.processor(logits, params)
        self.assertEqual(result[0, self.END].item(), 0.0)
        self.assertTrue(torch.isinf(result[0, 0]) and result[0, 0] < 0)
```
**EN:** Test forcing end token when budget exceeded and last token is newline. This test exercises `test_budget_exceeded_with_newline_forces_end_token` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test forcing end token when budget exceeded and last token is newline. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_budget_exceeded_with_newline_forces_end_token`。

### Lines 145-152: test case skips when not in thinking / 测试用例 skips when not in thinking
```python
    def test_skips_when_not_in_thinking(self):
        """Test skip when THINKING_START is absent (no thinking phase)."""
        req = _make_req(origin_input_ids=[100, 101], output_ids=[102])
        params = [{"thinking_budget": 0, "__req__": req}]
        logits = self._logits()
        original = logits.clone()
        result = self.processor(logits, params)
        self.assertTrue(torch.equal(result, original))
```
**EN:** Test skip when THINKING_START is absent (no thinking phase). This test exercises `test_skips_when_not_in_thinking` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test skip when THINKING_START is absent (no thinking phase). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_skips_when_not_in_thinking`。

### Lines 154-164: test case skips when thinking already ended / 测试用例 skips when thinking already ended
```python
    def test_skips_when_thinking_already_ended(self):
        """Test skip when THINKING_END already appeared."""
        req = _make_req(
            origin_input_ids=[self.START],
            output_ids=[100, self.END, 200],
        )
        params = [{"thinking_budget": 0, "__req__": req}]
        logits = self._logits()
        original = logits.clone()
        result = self.processor(logits, params)
        self.assertTrue(torch.equal(result, original))
```
**EN:** Test skip when THINKING_END already appeared. This test exercises `test_skips_when_thinking_already_ended` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test skip when THINKING_END already appeared. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_skips_when_thinking_already_ended`。

### Lines 166-173: test case skips when budget is none / 测试用例 skips when budget is none
```python
    def test_skips_when_budget_is_none(self):
        """Test that thinking_budget=None is ignored even during thinking phase."""
        req = _make_req(origin_input_ids=[self.START], output_ids=[100] * 10)
        params = [{"thinking_budget": None, "__req__": req}]
        logits = self._logits()
        original = logits.clone()
        result = self.processor(logits, params)
        self.assertTrue(torch.equal(result, original))
```
**EN:** Test that thinking_budget=None is ignored even during thinking phase. This test exercises `test_skips_when_budget_is_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that thinking_budget=None is ignored even during thinking phase. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_skips_when_budget_is_none`。

### Lines 175-182: test case skips when budget is negative / 测试用例 skips when budget is negative
```python
    def test_skips_when_budget_is_negative(self):
        """Test that negative thinking_budget is treated as disabled (no enforcement)."""
        req = _make_req(origin_input_ids=[self.START], output_ids=[100] * 10)
        params = [{"thinking_budget": -1, "__req__": req}]
        logits = self._logits()
        original = logits.clone()
        result = self.processor(logits, params)
        self.assertTrue(torch.equal(result, original))
```
**EN:** Test that negative thinking_budget is treated as disabled (no enforcement). This test exercises `test_skips_when_budget_is_negative` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that negative thinking_budget is treated as disabled (no enforcement). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_skips_when_budget_is_negative`。

### Lines 184-189: test case none params returns unchanged / 测试用例 none params returns unchanged
```python
    def test_none_params_returns_unchanged(self):
        """Test that passing None as param list returns logits unchanged."""
        logits = self._logits()
        original = logits.clone()
        result = self.processor(logits, None)
        self.assertTrue(torch.equal(result, original))
```
**EN:** Test that passing None as param list returns logits unchanged. This test exercises `test_none_params_returns_unchanged` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that passing None as param list returns logits unchanged. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_none_params_returns_unchanged`。

### Lines 191-196: test case empty params returns unchanged / 测试用例 empty params returns unchanged
```python
    def test_empty_params_returns_unchanged(self):
        """Test that passing empty param list returns logits unchanged."""
        logits = self._logits()
        original = logits.clone()
        result = self.processor(logits, [])
        self.assertTrue(torch.equal(result, original))
```
**EN:** Test that passing empty param list returns logits unchanged. This test exercises `test_empty_params_returns_unchanged` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that passing empty param list returns logits unchanged. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_params_returns_unchanged`。

### Lines 198-208: test case budget zero forces immediate end / 测试用例 budget zero forces immediate end
```python
    def test_budget_zero_forces_immediate_end(self):
        """Test that budget=0 forces thinking to end immediately."""
        req = _make_req(
            origin_input_ids=[self.START],
            output_ids=[100],  # 1 token after start > budget=0
        )
        params = [{"thinking_budget": 0, "__req__": req}]
        logits = self._logits()
        result = self.processor(logits, params)
        # Should force newline since last token (100) is not newline
        self.assertEqual(result[0, self.NL].item(), 0.0)
```
**EN:** Test that budget=0 forces thinking to end immediately. This test exercises `test_budget_zero_forces_immediate_end` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that budget=0 forces thinking to end immediately. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_budget_zero_forces_immediate_end`。

### Lines 210-223: test case none param dict in list skipped / 测试用例 none param dict in list skipped
```python
    def test_none_param_dict_in_list_skipped(self):
        """Test that None entry in param list is skipped gracefully."""
        req = _make_req(
            origin_input_ids=[self.START],
            output_ids=[100] * 10,
        )
        params = [None, {"thinking_budget": 0, "__req__": req}]
        logits = self._logits(batch_size=2)
        result = self.processor(logits, params)
        # Batch 0 (None param) should be unchanged
        self.assertEqual(result[0, 0].item(), 0.0)
        # Batch 1 should have been modified (budget exceeded)
        self.assertEqual(result[1, self.NL].item(), 0.0)
        self.assertTrue(torch.isinf(result[1, 0]) and result[1, 0] < 0)
```
**EN:** Test that None entry in param list is skipped gracefully. This test exercises `test_none_param_dict_in_list_skipped` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that None entry in param list is skipped gracefully. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_none_param_dict_in_list_skipped`。

### Lines 225-238: test case multiple thinking start counts from first / 测试用例 multiple thinking start counts from first
```python
    def test_multiple_thinking_start_counts_from_first(self):
        """Test that budget counts from the first THINKING_START occurrence."""
        req = _make_req(
            origin_input_ids=[self.START, 100, 101],
            output_ids=[self.START, 200, 201],  # second START in output
        )
        # cur_ids = [START, 100, 101, START, 200, 201]
        # First START at index 0, tokens_after_start = 5
        # Budget=10 → 5 < 10 → no modification
        params = [{"thinking_budget": 10, "__req__": req}]
        logits = self._logits()
        original = logits.clone()
        result = self.processor(logits, params)
        self.assertTrue(torch.equal(result, original))
```
**EN:** Test that budget counts from the first THINKING_START occurrence. This test exercises `test_multiple_thinking_start_counts_from_first` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that budget counts from the first THINKING_START occurrence. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_thinking_start_counts_from_first`。

### Lines 240-253: test case deepseek r1 variant forces end / 测试用例 deepseek r1 variant forces end
```python
    def test_deepseek_r1_variant_forces_end(self):
        """Test DeepSeekR1 variant with its own token IDs."""
        proc = DeepSeekR1ThinkingBudgetLogitProcessor()
        START = proc.THINKING_START_TOKEN_ID  # 128798
        NL = proc.NEW_LINE_TOKEN_ID  # 201
        VOCAB = 200000

        req = _make_req(origin_input_ids=[START], output_ids=[100] * 5)
        params = [{"thinking_budget": 5, "__req__": req}]
        logits = torch.zeros(1, VOCAB)
        result = proc(logits, params)
        # Budget exceeded, last token (100) is not newline → force newline
        self.assertEqual(result[0, NL].item(), 0.0)
        self.assertTrue(torch.isinf(result[0, 0]) and result[0, 0] < 0)
```
**EN:** Test DeepSeekR1 variant with its own token IDs. This test exercises `test_deepseek_r1_variant_forces_end` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test DeepSeekR1 variant with its own token IDs. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_deepseek_r1_variant_forces_end`。

### Lines 254-256: supporting source context / 辅助源码上下文
```python


# DeepseekOCRNoRepeatNGramLogitProcessor
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 257-257: class TestDeepseekOCRNoRepeatNGramLogitProcessor declaration / 类 TestDeepseekOCRNoRepeatNGramLogitProcessor 声明
```python
class TestDeepseekOCRNoRepeatNGramLogitProcessor(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 258-258: class-level constants and configuration for `TestDeepseekOCRNoRepeatNGramLogitProcessor` / 类级常量与配置
```python
    VOCAB = 100
```
**EN:** This block defines shared names such as `VOCAB`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `VOCAB` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 260-261: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.processor = DeepseekOCRNoRepeatNGramLogitProcessor()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 263-264: method logits / 方法 logits
```python
    def _logits(self, batch_size=1):
        return torch.zeros(batch_size, self.VOCAB)
```
**EN:** This block implements `_logits` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_logits`，承担模块行为中的一个聚焦逻辑片段。

### Lines 266-278: test case bans repeated bigrams / 测试用例 bans repeated bigrams
```python
    def test_bans_repeated_bigrams(self):
        """Test banning token that completes a repeated bigram."""
        req = _make_req(origin_input_ids=[1, 2, 3, 1, 2])
        params = [
            {
                "__req__": req,
                "ngram_size": 2,
                "window_size": 100,
            }
        ]
        logits = self._logits()
        result = self.processor(logits, params)
        self.assertTrue(torch.isinf(result[0, 3]) and result[0, 3] < 0)
```
**EN:** Test banning token that completes a repeated bigram. This test exercises `test_bans_repeated_bigrams` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test banning token that completes a repeated bigram. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bans_repeated_bigrams`。

### Lines 280-287: test case non repeated tokens unchanged / 测试用例 non repeated tokens unchanged
```python
    def test_non_repeated_tokens_unchanged(self):
        """Test that tokens not completing a repeated ngram are unchanged."""
        req = _make_req(origin_input_ids=[1, 2, 3, 1, 2])
        params = [{"__req__": req, "ngram_size": 2, "window_size": 100}]
        logits = self._logits()
        result = self.processor(logits, params)
        # Token 1 is not banned (prefix (2) was followed by 3, not 1)
        self.assertEqual(result[0, 1].item(), 0.0)
```
**EN:** Test that tokens not completing a repeated ngram are unchanged. This test exercises `test_non_repeated_tokens_unchanged` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that tokens not completing a repeated ngram are unchanged. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_non_repeated_tokens_unchanged`。

### Lines 289-299: test case window size limits search / 测试用例 window size limits search
```python
    def test_window_size_limits_search(self):
        """Test that ngrams outside the window are not considered."""
        # Sequence: [1,2,3,...,1,2] but window only covers the last 3 tokens
        req = _make_req(origin_input_ids=[1, 2, 3, 4, 5, 1, 2])
        params = [{"__req__": req, "ngram_size": 2, "window_size": 3}]
        logits = self._logits()
        result = self.processor(logits, params)
        # Window covers [5, 1, 2]. The bigram (1,2) from index 0-1 is outside.
        # Within window: bigrams are (5,1), (1,2). Current prefix is (2).
        # No bigram starting with prefix (2) in window → nothing banned.
        self.assertEqual(result[0, 3].item(), 0.0)
```
**EN:** Test that ngrams outside the window are not considered. This test exercises `test_window_size_limits_search` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that ngrams outside the window are not considered. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_window_size_limits_search`。

### Lines 301-315: test case whitelist protects tokens / 测试用例 whitelist protects tokens
```python
    def test_whitelist_protects_tokens(self):
        """Test that whitelisted tokens are not banned despite repeated ngrams."""
        req = _make_req(origin_input_ids=[1, 2, 3, 1, 2])
        params = [
            {
                "__req__": req,
                "ngram_size": 2,
                "window_size": 100,
                "whitelist_token_ids": [3],
            }
        ]
        logits = self._logits()
        result = self.processor(logits, params)
        # Token 3 would be banned but is whitelisted
        self.assertEqual(result[0, 3].item(), 0.0)
```
**EN:** Test that whitelisted tokens are not banned despite repeated ngrams. This test exercises `test_whitelist_protects_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that whitelisted tokens are not banned despite repeated ngrams. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_whitelist_protects_tokens`。

### Lines 317-324: test case ngram size zero skips / 测试用例 ngram size zero skips
```python
    def test_ngram_size_zero_skips(self):
        """ngram_size=0 is invalid and should be skipped (no modification)."""
        req = _make_req(origin_input_ids=[1, 2, 1, 2])
        params = [{"__req__": req, "ngram_size": 0, "window_size": 100}]
        logits = self._logits()
        original = logits.clone()
        result = self.processor(logits, params)
        self.assertTrue(torch.equal(result, original))
```
**EN:** ngram_size=0 is invalid and should be skipped (no modification). This test exercises `test_ngram_size_zero_skips` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** ngram_size=0 is invalid and should be skipped (no modification). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ngram_size_zero_skips`。

### Lines 326-333: test case window size zero skips / 测试用例 window size zero skips
```python
    def test_window_size_zero_skips(self):
        """Test that window_size=0 disables ngram checking (no modification)."""
        req = _make_req(origin_input_ids=[1, 2, 1, 2])
        params = [{"__req__": req, "ngram_size": 2, "window_size": 0}]
        logits = self._logits()
        original = logits.clone()
        result = self.processor(logits, params)
        self.assertTrue(torch.equal(result, original))
```
**EN:** Test that window_size=0 disables ngram checking (no modification). This test exercises `test_window_size_zero_skips` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that window_size=0 disables ngram checking (no modification). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_window_size_zero_skips`。

### Lines 335-340: test case empty params returns unchanged / 测试用例 empty params returns unchanged
```python
    def test_empty_params_returns_unchanged(self):
        """Test that None param list returns logits unchanged (early return)."""
        logits = self._logits()
        original = logits.clone()
        result = self.processor(logits, None)
        self.assertTrue(torch.equal(result, original))
```
**EN:** Test that None param list returns logits unchanged (early return). This test exercises `test_empty_params_returns_unchanged` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that None param list returns logits unchanged (early return). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_params_returns_unchanged`。

### Lines 342-349: test case short sequence skips / 测试用例 short sequence skips
```python
    def test_short_sequence_skips(self):
        """Sequence shorter than ngram_size should be skipped."""
        req = _make_req(origin_input_ids=[1])
        params = [{"__req__": req, "ngram_size": 3, "window_size": 100}]
        logits = self._logits()
        original = logits.clone()
        result = self.processor(logits, params)
        self.assertTrue(torch.equal(result, original))
```
**EN:** Sequence shorter than ngram_size should be skipped. This test exercises `test_short_sequence_skips` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Sequence shorter than ngram_size should be skipped. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_short_sequence_skips`。

### Lines 351-362: test case unigram mode / 测试用例 unigram mode
```python
    def test_unigram_mode(self):
        """ngram_size=1 bans any token already seen in the window."""
        req = _make_req(origin_input_ids=[5, 10, 15])
        params = [{"__req__": req, "ngram_size": 1, "window_size": 100}]
        logits = self._logits()
        result = self.processor(logits, params)
        # All tokens in [5, 10, 15] should be banned
        self.assertTrue(torch.isinf(result[0, 5]) and result[0, 5] < 0)
        self.assertTrue(torch.isinf(result[0, 10]) and result[0, 10] < 0)
        self.assertTrue(torch.isinf(result[0, 15]) and result[0, 15] < 0)
        # Other tokens should be fine
        self.assertEqual(result[0, 0].item(), 0.0)
```
**EN:** ngram_size=1 bans any token already seen in the window. This test exercises `test_unigram_mode` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** ngram_size=1 bans any token already seen in the window. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unigram_mode`。

### Lines 364-370: test case none req skips / 测试用例 none req skips
```python
    def test_none_req_skips(self):
        """If __req__ is missing, the batch item should be skipped."""
        params = [{"ngram_size": 2, "window_size": 100}]
        logits = self._logits()
        original = logits.clone()
        result = self.processor(logits, params)
        self.assertTrue(torch.equal(result, original))
```
**EN:** If __req__ is missing, the batch item should be skipped. This test exercises `test_none_req_skips` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** If __req__ is missing, the batch item should be skipped. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_none_req_skips`。

### Lines 372-379: test case invalid ngram size type skips / 测试用例 invalid ngram size type skips
```python
    def test_invalid_ngram_size_type_skips(self):
        """Non-numeric ngram_size should be handled gracefully."""
        req = _make_req(origin_input_ids=[1, 2, 1, 2])
        params = [{"__req__": req, "ngram_size": "invalid", "window_size": 100}]
        logits = self._logits()
        original = logits.clone()
        result = self.processor(logits, params)
        self.assertTrue(torch.equal(result, original))
```
**EN:** Non-numeric ngram_size should be handled gracefully. This test exercises `test_invalid_ngram_size_type_skips` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Non-numeric ngram_size should be handled gracefully. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_invalid_ngram_size_type_skips`。

### Lines 381-390: test case falsy params in list skipped / 测试用例 falsy params in list skipped
```python
    def test_falsy_params_in_list_skipped(self):
        """A falsy entry (None, {}, 0) in param list should be skipped."""
        req = _make_req(origin_input_ids=[1, 2, 1, 2])
        params = [None, {"__req__": req, "ngram_size": 2, "window_size": 100}]
        logits = self._logits(batch_size=2)
        result = self.processor(logits, params)
        # Batch 0 (None) unchanged
        self.assertEqual(result[0, 0].item(), 0.0)
        # Batch 1 has ban applied
        self.assertTrue(torch.isinf(result[1, 1]) and result[1, 1] < 0)
```
**EN:** A falsy entry (None, {}, 0) in param list should be skipped. This test exercises `test_falsy_params_in_list_skipped` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** A falsy entry (None, {}, 0) in param list should be skipped. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_falsy_params_in_list_skipped`。

### Lines 392-403: test case search end leq search start skips / 测试用例 search end leq search start skips
```python
    def test_search_end_leq_search_start_skips(self):
        """Test skip when window is too small for the ngram_size."""
        # sequence length=4, ngram_size=3, window_size=2
        # search_start = max(0, 4-2) = 2
        # search_end = 4 - 3 + 1 = 2
        # search_end (2) <= search_start (2) → skip
        req = _make_req(origin_input_ids=[1, 2, 3, 4])
        params = [{"__req__": req, "ngram_size": 3, "window_size": 2}]
        logits = self._logits()
        original = logits.clone()
        result = self.processor(logits, params)
        self.assertTrue(torch.equal(result, original))
```
**EN:** Test skip when window is too small for the ngram_size. This test exercises `test_search_end_leq_search_start_skips` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test skip when window is too small for the ngram_size. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_search_end_leq_search_start_skips`。

### Lines 405-419: test case invalid whitelist type handled / 测试用例 invalid whitelist type handled
```python
    def test_invalid_whitelist_type_handled(self):
        """Test graceful handling of non-iterable whitelist_token_ids."""
        req = _make_req(origin_input_ids=[1, 2, 1, 2])
        params = [
            {
                "__req__": req,
                "ngram_size": 2,
                "window_size": 100,
                "whitelist_token_ids": 999,  # int, not iterable
            }
        ]
        logits = self._logits()
        result = self.processor(logits, params)
        # Should still ban token 1 (whitelist parse fails, falls back to empty set)
        self.assertTrue(torch.isinf(result[0, 1]) and result[0, 1] < 0)
```
**EN:** Test graceful handling of non-iterable whitelist_token_ids. This test exercises `test_invalid_whitelist_type_handled` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test graceful handling of non-iterable whitelist_token_ids. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_invalid_whitelist_type_handled`。

### Lines 421-441: test case batch processing / 测试用例 batch processing
```python
    def test_batch_processing(self):
        """Test that multiple batch items are processed independently."""
        req1 = _make_req(
            origin_input_ids=[1, 2, 1, 2]
        )  # will ban token 2 (bigram repeat)
        req2 = _make_req(origin_input_ids=[3, 4, 5])  # no repeat
        params = [
            {"__req__": req1, "ngram_size": 2, "window_size": 100},
            {"__req__": req2, "ngram_size": 2, "window_size": 100},
        ]
        logits = self._logits(batch_size=2)
        result = self.processor(logits, params)
        # Batch 0: bigram (1,2) appeared, prefix is (2) → ban token that followed (2) = 1
        # Also (2,1) appeared, prefix is (2) → already covered
        # Actually: sequence is [1,2,1,2], prefix is last (ngram_size-1)=1 token = (2)
        # Scanning: index 0: (1,2) prefix=(1); index 1: (2,1) prefix=(2)→bans 1; index 2: (1,2) prefix=(1)
        # So prefix (2) appeared at index 1, followed by token 1. Ban token 1.
        self.assertTrue(torch.isinf(result[0, 1]) and result[0, 1] < 0)
        # Batch 1: prefix is (5), no matching prefix in window → no bans
        self.assertEqual(result[1, 3].item(), 0.0)
        self.assertEqual(result[1, 4].item(), 0.0)
```
**EN:** Test that multiple batch items are processed independently. This test exercises `test_batch_processing` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that multiple batch items are processed independently. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_batch_processing`。

### Lines 444-445: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_make_req`: This block implements `_make_req` and captures one focused piece of the module's behavior. / 该代码块实现 `_make_req`，承担模块行为中的一个聚焦逻辑片段。
- `TestCustomLogitProcessorSerialization`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDisallowedTokensLogitsProcessor`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestThinkingBudgetLogitProcessor`: Test thinking budget enforcement using Qwen3 token IDs. / 用于组织相关测试、夹具或辅助方法。
- `TestDeepseekOCRNoRepeatNGramLogitProcessor`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestCustomLogitProcessorSerialization.test_to_str_produces_valid_json`: Test that to_str() produces valid JSON with a 'callable' field. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_to_str_produces_valid_json`。
- `TestCustomLogitProcessorSerialization.test_round_trip_serialization`: Test serialize then deserialize produces a usable processor. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_round_trip_serialization`。
- `TestCustomLogitProcessorSerialization.test_from_str_is_cached`: Test that from_str uses LRU cache for repeated calls. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_from_str_is_cached`。
- `TestDisallowedTokensLogitsProcessor.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestDisallowedTokensLogitsProcessor.test_disallowed_tokens_set_to_neg_inf`: Test that disallowed token positions are set to -inf for all batch items. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_disallowed_tokens_set_to_neg_inf`。
- `TestDisallowedTokensLogitsProcessor.test_allowed_tokens_unchanged`: Test that non-disallowed tokens keep their original logit values. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_allowed_tokens_unchanged`。
- `TestDisallowedTokensLogitsProcessor.test_mismatched_params_raises`: Test that mismatched token_ids across batch items raises AssertionError. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mismatched_params_raises`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `unittest`, `unittest.mock`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.srt.sampling.custom_logit_processor`, `sglang.test.test_utils`

- **Total lines / 总行数**: 445
