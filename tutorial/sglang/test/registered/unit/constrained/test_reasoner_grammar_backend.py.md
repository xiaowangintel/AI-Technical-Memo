# test_reasoner_grammar_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/constrained/test_reasoner_grammar_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates reasoner grammar backend behavior in SGLang's unit / constrained area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / constrained 领域中与 reasoner grammar backend 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: module imports and dependencies / 模块导入与依赖
```python
import os
import unittest
from types import SimpleNamespace
from unittest.mock import MagicMock

import torch

from sglang.srt.constrained.base_grammar_backend import BaseGrammarBackend
from sglang.srt.constrained.reasoner_grammar_backend import (
    ReasonerGrammarBackend,
    ReasonerGrammarObject,
)
from sglang.srt.constrained.torch_ops.token_filter_torch_ops import (
    set_token_filter_torch,
)
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `os`, `unittest`, `types`, `unittest.mock`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `os`, `unittest`, `types`, `unittest.mock`。

### Lines 18-18: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(2.0, "base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 21-21: class _DummyTokenizer declaration / 类 _DummyTokenizer 声明
```python
class _DummyTokenizer:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 22-23: method init / 方法 init
```python
    def __init__(self, token_map):
        self._token_map = token_map
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 25-26: method encode / 方法 encode
```python
    def encode(self, text, add_special_tokens=False):
        return list(self._token_map.get(text, []))
```
**EN:** This block implements `encode` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `encode`，承担模块行为中的一个聚焦逻辑片段。

### Lines 29-29: class _DummyGrammarBackend declaration / 类 _DummyGrammarBackend 声明
```python
class _DummyGrammarBackend(BaseGrammarBackend):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `BaseGrammarBackend`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `BaseGrammarBackend`。

### Lines 30-33: method init / 方法 init
```python
    def __init__(self, support_token_filter=True):
        super().__init__()
        self._support_token_filter = support_token_filter
        self._dispatch_result = None
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 35-37: method is support token filter / 方法 is support token filter
```python
    @property
    def is_support_token_filter(self):
        return self._support_token_filter
```
**EN:** This block implements `is_support_token_filter` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `is_support_token_filter`，承担模块行为中的一个聚焦逻辑片段。

### Lines 39-41: method allocate vocab mask / 方法 allocate vocab mask
```python
    @staticmethod
    def allocate_vocab_mask(vocab_size, batch_size, device):
        return torch.zeros((batch_size, (vocab_size + 31) // 32), dtype=torch.int32)
```
**EN:** This block implements `allocate_vocab_mask` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `allocate_vocab_mask`，承担模块行为中的一个聚焦逻辑片段。

### Lines 43-45: method move vocab mask / 方法 move vocab mask
```python
    @staticmethod
    def move_vocab_mask(vocab_mask, device):
        return vocab_mask
```
**EN:** This block implements `move_vocab_mask` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `move_vocab_mask`，承担模块行为中的一个聚焦逻辑片段。

### Lines 47-49: method apply vocab mask / 方法 apply vocab mask
```python
    @staticmethod
    def apply_vocab_mask(logits, vocab_mask):
        return None
```
**EN:** This block implements `apply_vocab_mask` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `apply_vocab_mask`，承担模块行为中的一个聚焦逻辑片段。

### Lines 51-57: method set token filter / 方法 set token filter
```python
    @staticmethod
    def set_token_filter(
        vocab_mask, token_ids, batch_idx, is_allowed=True, reset_vocab_mask=True
    ):
        set_token_filter_torch(
            vocab_mask, token_ids, batch_idx, is_allowed, reset_vocab_mask
        )
```
**EN:** This block implements `set_token_filter` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `set_token_filter`，承担模块行为中的一个聚焦逻辑片段。

### Lines 59-60: method init value dispatch / 方法 init value dispatch
```python
    def _init_value_dispatch(self, key, reasoning):
        return self._dispatch_result
```
**EN:** This block implements `_init_value_dispatch` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_init_value_dispatch`，承担模块行为中的一个聚焦逻辑片段。

### Lines 63-70: function allowed token ids / 函数 allowed token ids
```python
def _allowed_token_ids(vocab_mask, token_ids):
    allowed = []
    for token_id in token_ids:
        elem = token_id // 32
        bit = token_id % 32
        if int(vocab_mask[0, elem].item()) & (1 << bit):
            allowed.append(token_id)
    return allowed
```
**EN:** This block implements `_allowed_token_ids` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_allowed_token_ids`，承担模块行为中的一个聚焦逻辑片段。

### Lines 73-73: class TestReasonerGrammarObject declaration / 类 TestReasonerGrammarObject 声明
```python
class TestReasonerGrammarObject(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 74-87: method make strict object / 方法 make strict object
```python
    def _make_strict_object(self):
        return ReasonerGrammarObject(
            grammar=None,
            think_end_id=7,
            think_excluded_token_ids=[3, 5],
            max_think_tokens=2,
            enable_token_filter=True,
            token_filter_fn=set_token_filter_torch,
            allocate_vocab_mask_fn=lambda vocab_size, batch_size, device: torch.zeros(
                (batch_size, (vocab_size + 31) // 32), dtype=torch.int32
            ),
            move_vocab_mask_fn=lambda vocab_mask, device: vocab_mask,
            apply_vocab_mask_fn=lambda logits, vocab_mask: None,
        )
```
**EN:** This block implements `_make_strict_object` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_strict_object`，承担模块行为中的一个聚焦逻辑片段。

### Lines 89-97: test case strict thinking phase excludes configured tokens / 测试用例 strict thinking phase excludes configured tokens
```python
    def test_strict_thinking_phase_excludes_configured_tokens(self):
        obj = self._make_strict_object()
        obj.maybe_init_reasoning(True)
        mask = obj.allocate_vocab_mask(64, 1, "cpu")

        obj.fill_vocab_mask(mask, 0)

        allowed = _allowed_token_ids(mask, [0, 1, 3, 5, 7, 8])
        self.assertEqual(allowed, [0, 1, 7, 8])
```
**EN:** This test exercises `test_strict_thinking_phase_excludes_configured_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_strict_thinking_phase_excludes_configured_tokens`。

### Lines 99-109: test case budget exhaustion allows only think end / 测试用例 budget exhaustion allows only think end
```python
    def test_budget_exhaustion_allows_only_think_end(self):
        obj = self._make_strict_object()
        obj.maybe_init_reasoning(True)
        obj.accept_token(10)
        obj.accept_token(11)
        mask = obj.allocate_vocab_mask(64, 1, "cpu")

        obj.fill_vocab_mask(mask, 0)

        allowed = _allowed_token_ids(mask, [0, 1, 3, 5, 7, 8, 10, 11])
        self.assertEqual(allowed, [7])
```
**EN:** This test exercises `test_budget_exhaustion_allows_only_think_end` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_budget_exhaustion_allows_only_think_end`。

### Lines 111-117: test case strict only wrapper exposes backend mask hooks / 测试用例 strict only wrapper exposes backend mask hooks
```python
    def test_strict_only_wrapper_exposes_backend_mask_hooks(self):
        obj = self._make_strict_object()
        mask = obj.allocate_vocab_mask(64, 2, "cpu")

        self.assertEqual(mask.shape, (2, 2))
        self.assertIs(obj.move_vocab_mask(mask, "cpu"), mask)
        self.assertIsNotNone(obj.apply_vocab_mask)
```
**EN:** This test exercises `test_strict_only_wrapper_exposes_backend_mask_hooks` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_strict_only_wrapper_exposes_backend_mask_hooks`。

### Lines 120-120: class TestReasonerGrammarBackend declaration / 类 TestReasonerGrammarBackend 声明
```python
class TestReasonerGrammarBackend(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 121-122: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self._prev_budget = os.environ.get("SGLANG_MAX_THINK_TOKENS")
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 124-128: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        if self._prev_budget is None:
            os.environ.pop("SGLANG_MAX_THINK_TOKENS", None)
        else:
            os.environ["SGLANG_MAX_THINK_TOKENS"] = self._prev_budget
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 130-136: method make parser / 方法 make parser
```python
    def _make_parser(self):
        detector = SimpleNamespace(
            think_start_token="<think>",
            think_end_token="</think>",
            think_excluded_tokens=["<tool_call>", "</tool_call>"],
        )
        return SimpleNamespace(detector=detector)
```
**EN:** This block implements `_make_parser` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_parser`，承担模块行为中的一个聚焦逻辑片段。

### Lines 138-146: method make tokenizer / 方法 make tokenizer
```python
    def _make_tokenizer(self, start_ids=None, end_ids=None):
        return _DummyTokenizer(
            {
                "<think>": [1] if start_ids is None else start_ids,
                "</think>": [2] if end_ids is None else end_ids,
                "<tool_call>": [3],
                "</tool_call>": [4],
            }
        )
```
**EN:** This block implements `_make_tokenizer` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_tokenizer`，承担模块行为中的一个聚焦逻辑片段。

### Lines 148-163: test case init strict reasoning grammar uses token filter and budget / 测试用例 init strict reasoning grammar uses token filter and budget
```python
    def test_init_strict_reasoning_grammar_uses_token_filter_and_budget(self):
        os.environ["SGLANG_MAX_THINK_TOKENS"] = "2"
        backend = _DummyGrammarBackend(support_token_filter=True)
        reasoner = ReasonerGrammarBackend(
            backend,
            self._make_parser(),
            self._make_tokenizer(),
            enable_strict_thinking=True,
        )

        obj = reasoner.init_strict_reasoning_grammar(reasoning=True)

        self.assertIsInstance(obj, ReasonerGrammarObject)
        self.assertTrue(obj.enable_token_filter)
        self.assertEqual(obj.max_think_tokens, 2)
        self.assertEqual(obj.think_excluded_token_ids, [3, 4])
```
**EN:** This test exercises `test_init_strict_reasoning_grammar_uses_token_filter_and_budget` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_strict_reasoning_grammar_uses_token_filter_and_budget`。

### Lines 165-174: test case init strict reasoning grammar none when strict disabled / 测试用例 init strict reasoning grammar none when strict disabled
```python
    def test_init_strict_reasoning_grammar_none_when_strict_disabled(self):
        backend = _DummyGrammarBackend(support_token_filter=True)
        reasoner = ReasonerGrammarBackend(
            backend,
            self._make_parser(),
            self._make_tokenizer(),
            enable_strict_thinking=False,
        )

        self.assertIsNone(reasoner.init_strict_reasoning_grammar(reasoning=True))
```
**EN:** This test exercises `test_init_strict_reasoning_grammar_none_when_strict_disabled` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_strict_reasoning_grammar_none_when_strict_disabled`。

### Lines 176-194: test case wraps inner grammar with reasoning state machine / 测试用例 wraps inner grammar with reasoning state machine
```python
    def test_wraps_inner_grammar_with_reasoning_state_machine(self):
        os.environ["SGLANG_MAX_THINK_TOKENS"] = "1"
        backend = _DummyGrammarBackend(support_token_filter=True)
        inner_grammar = MagicMock()
        backend._dispatch_result = inner_grammar
        reasoner = ReasonerGrammarBackend(
            backend,
            self._make_parser(),
            self._make_tokenizer(),
            enable_strict_thinking=True,
        )

        wrapped = reasoner._init_value_dispatch(("json", "{}"), reasoning=True)
        self.assertIsInstance(wrapped, ReasonerGrammarObject)
        wrapped.accept_token(10)
        inner_grammar.accept_token.assert_not_called()
        wrapped.accept_token(2)
        wrapped.accept_token(42)
        inner_grammar.accept_token.assert_called_once_with(42)
```
**EN:** This test exercises `test_wraps_inner_grammar_with_reasoning_state_machine` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_wraps_inner_grammar_with_reasoning_state_machine`。

### Lines 196-205: test case accepts multi token think start marker / 测试用例 accepts multi token think start marker
```python
    def test_accepts_multi_token_think_start_marker(self):
        """think_start_token can be multi-token (e.g., GPT-OSS) since it's not used."""
        backend = _DummyGrammarBackend(support_token_filter=True)
        reasoner = ReasonerGrammarBackend(
            backend,
            self._make_parser(),
            self._make_tokenizer(start_ids=[1, 2]),
            enable_strict_thinking=True,
        )
        self.assertIsNotNone(reasoner)
```
**EN:** think_start_token can be multi-token (e.g., GPT-OSS) since it's not used. This test exercises `test_accepts_multi_token_think_start_marker` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** think_start_token can be multi-token (e.g., GPT-OSS) since it's not used. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_accepts_multi_token_think_start_marker`。

### Lines 207-216: test case rejects multi token think end marker / 测试用例 rejects multi token think end marker
```python
    def test_rejects_multi_token_think_end_marker(self):
        backend = _DummyGrammarBackend(support_token_filter=True)

        with self.assertRaisesRegex(ValueError, "must encode to exactly one token"):
            ReasonerGrammarBackend(
                backend,
                self._make_parser(),
                self._make_tokenizer(end_ids=[2, 3]),
                enable_strict_thinking=True,
            )
```
**EN:** This test exercises `test_rejects_multi_token_think_end_marker` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rejects_multi_token_think_end_marker`。

### Lines 218-235: test case rejects unencodable excluded token / 测试用例 rejects unencodable excluded token
```python
    def test_rejects_unencodable_excluded_token(self):
        backend = _DummyGrammarBackend(support_token_filter=True)
        parser = self._make_parser()
        parser.detector.think_excluded_tokens = ["<unknown>"]
        tokenizer = _DummyTokenizer(
            {
                "<think>": [1],
                "</think>": [2],
            }
        )

        with self.assertRaisesRegex(ValueError, "could not be encoded"):
            ReasonerGrammarBackend(
                backend,
                parser,
                tokenizer,
                enable_strict_thinking=True,
            )
```
**EN:** This test exercises `test_rejects_unencodable_excluded_token` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rejects_unencodable_excluded_token`。

### Lines 237-246: test case strict mode fails when backend lacks token filter / 测试用例 strict mode fails when backend lacks token filter
```python
    def test_strict_mode_fails_when_backend_lacks_token_filter(self):
        backend = _DummyGrammarBackend(support_token_filter=False)

        with self.assertRaisesRegex(ValueError, "does not support token filtering"):
            ReasonerGrammarBackend(
                backend,
                self._make_parser(),
                self._make_tokenizer(),
                enable_strict_thinking=True,
            )
```
**EN:** This test exercises `test_strict_mode_fails_when_backend_lacks_token_filter` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_strict_mode_fails_when_backend_lacks_token_filter`。

### Lines 249-249: class TestReasonerGrammarObjectRollback declaration / 类 TestReasonerGrammarObjectRollback 声明
```python
class TestReasonerGrammarObjectRollback(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 250-250: supporting statements / 辅助语句
```python
    """Tests for rollback correctness at the THINKING→GENERATION boundary."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 252-268: method make object with mock grammar / 方法 make object with mock grammar
```python
    def _make_object_with_mock_grammar(self):
        inner_grammar = MagicMock()
        inner_grammar.is_terminated.return_value = False
        obj = ReasonerGrammarObject(
            grammar=inner_grammar,
            think_end_id=7,
            think_excluded_token_ids=[3, 5],
            max_think_tokens=-1,
            enable_token_filter=True,
            token_filter_fn=set_token_filter_torch,
            allocate_vocab_mask_fn=lambda vs, bs, d: torch.zeros(
                (bs, (vs + 31) // 32), dtype=torch.int32
            ),
            move_vocab_mask_fn=lambda vm, d: vm,
            apply_vocab_mask_fn=lambda l, vm: None,
        )
        return obj, inner_grammar
```
**EN:** This block implements `_make_object_with_mock_grammar` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_object_with_mock_grammar`，承担模块行为中的一个聚焦逻辑片段。

### Lines 270-289: test case rollback at generation boundary returns to thinking / 测试用例 rollback at generation boundary returns to thinking
```python
    def test_rollback_at_generation_boundary_returns_to_thinking(self):
        obj, inner_grammar = self._make_object_with_mock_grammar()
        obj.maybe_init_reasoning(True)

        # Accept 3 thinking tokens then think_end_id
        obj.accept_token(10)
        obj.accept_token(11)
        obj.accept_token(12)
        obj.accept_token(7)  # think_end_id → tokens_after_end = 0

        self.assertTrue(obj._is_generation())
        self.assertEqual(obj.tokens_after_end, 0)

        # Rollback 1 step: should return to THINKING
        obj.rollback(1)
        self.assertTrue(obj._is_thinking())
        self.assertEqual(obj.tokens_in_think, 3)
        self.assertEqual(obj.tokens_after_end, -1)
        # Grammar should not have been rolled back (no generation tokens were accepted)
        inner_grammar.rollback.assert_not_called()
```
**EN:** This test exercises `test_rollback_at_generation_boundary_returns_to_thinking` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rollback_at_generation_boundary_returns_to_thinking`。

### Lines 291-310: test case rollback spanning both phases / 测试用例 rollback spanning both phases
```python
    def test_rollback_spanning_both_phases(self):
        obj, inner_grammar = self._make_object_with_mock_grammar()
        obj.maybe_init_reasoning(True)

        # 2 thinking tokens + think_end + 3 generation tokens
        obj.accept_token(10)  # think
        obj.accept_token(11)  # think
        obj.accept_token(7)  # think_end_id
        obj.accept_token(20)  # gen 1
        obj.accept_token(21)  # gen 2
        obj.accept_token(22)  # gen 3

        self.assertEqual(obj.tokens_after_end, 3)

        # Rollback 5: should roll back 3 generation tokens + think_end + 1 thinking token
        obj.rollback(5)
        self.assertTrue(obj._is_thinking())
        self.assertEqual(obj.tokens_in_think, 1)
        # Grammar should be rolled back by 3 (only generation tokens)
        inner_grammar.rollback.assert_called_once_with(3)
```
**EN:** This test exercises `test_rollback_spanning_both_phases` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rollback_spanning_both_phases`。

### Lines 312-325: test case rollback generation tokens only / 测试用例 rollback generation tokens only
```python
    def test_rollback_generation_tokens_only(self):
        obj, inner_grammar = self._make_object_with_mock_grammar()
        obj.maybe_init_reasoning(True)

        obj.accept_token(10)  # think
        obj.accept_token(7)  # think_end_id
        obj.accept_token(20)  # gen 1
        obj.accept_token(21)  # gen 2

        # Rollback 1: should only roll back 1 generation token
        obj.rollback(1)
        self.assertTrue(obj._is_generation())
        self.assertEqual(obj.tokens_after_end, 1)
        inner_grammar.rollback.assert_called_once_with(1)
```
**EN:** This test exercises `test_rollback_generation_tokens_only` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rollback_generation_tokens_only`。

### Lines 327-339: test case rollback thinking tokens does not touch grammar / 测试用例 rollback thinking tokens does not touch grammar
```python
    def test_rollback_thinking_tokens_does_not_touch_grammar(self):
        obj, inner_grammar = self._make_object_with_mock_grammar()
        obj.maybe_init_reasoning(True)

        obj.accept_token(10)
        obj.accept_token(11)
        obj.accept_token(12)

        obj.rollback(2)
        self.assertTrue(obj._is_thinking())
        self.assertEqual(obj.tokens_in_think, 1)
        inner_grammar.rollback.assert_not_called()
        inner_grammar.accept_token.assert_not_called()
```
**EN:** This test exercises `test_rollback_thinking_tokens_does_not_touch_grammar` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rollback_thinking_tokens_does_not_touch_grammar`。

### Lines 341-358: test case copy preserves state / 测试用例 copy preserves state
```python
    def test_copy_preserves_state(self):
        obj, inner_grammar = self._make_object_with_mock_grammar()
        obj.maybe_init_reasoning(True)

        obj.accept_token(10)
        obj.accept_token(7)  # think_end_id → GENERATION
        obj.accept_token(20)

        self.assertEqual(obj.tokens_in_think, 1)
        self.assertEqual(obj.tokens_after_end, 1)

        copy = obj.copy()
        # State counters must be preserved for speculative decoding
        self.assertEqual(copy.tokens_in_think, 1)
        self.assertEqual(copy.tokens_after_end, 1)
        self.assertTrue(copy._is_generation())
        self.assertIsNotNone(copy.grammar)
        inner_grammar.copy.assert_called_once()
```
**EN:** This test exercises `test_copy_preserves_state` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_copy_preserves_state`。

### Lines 360-370: test case copy preserves thinking state / 测试用例 copy preserves thinking state
```python
    def test_copy_preserves_thinking_state(self):
        obj, inner_grammar = self._make_object_with_mock_grammar()
        obj.maybe_init_reasoning(True)

        obj.accept_token(10)
        obj.accept_token(11)

        copy = obj.copy()
        self.assertEqual(copy.tokens_in_think, 2)
        self.assertEqual(copy.tokens_after_end, -1)
        self.assertTrue(copy._is_thinking())
```
**EN:** This test exercises `test_copy_preserves_thinking_state` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_copy_preserves_thinking_state`。

### Lines 373-373: class TestReasonerGrammarObjectFillVocabMask declaration / 类 TestReasonerGrammarObjectFillVocabMask 声明
```python
class TestReasonerGrammarObjectFillVocabMask(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 374-374: supporting statements / 辅助语句
```python
    """Tests for fill_vocab_mask behavior in different states."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 376-404: test case thinking phase does not consult inner grammar / 测试用例 thinking phase does not consult inner grammar
```python
    def test_thinking_phase_does_not_consult_inner_grammar(self):
        inner_grammar = MagicMock()
        # Must return a real tensor for allocate_vocab_mask since fill_vocab_mask
        # delegates to allocate_vocab_mask via self.grammar when grammar is not None
        inner_grammar.allocate_vocab_mask.side_effect = lambda vs, bs, d: torch.zeros(
            (bs, (vs + 31) // 32), dtype=torch.int32
        )
        obj = ReasonerGrammarObject(
            grammar=inner_grammar,
            think_end_id=7,
            think_excluded_token_ids=[3, 5],
            max_think_tokens=-1,
            enable_token_filter=True,
            token_filter_fn=set_token_filter_torch,
            allocate_vocab_mask_fn=lambda vs, bs, d: torch.zeros(
                (bs, (vs + 31) // 32), dtype=torch.int32
            ),
            move_vocab_mask_fn=lambda vm, d: vm,
            apply_vocab_mask_fn=lambda l, vm: None,
        )
        obj.maybe_init_reasoning(True)
        mask = obj.allocate_vocab_mask(64, 1, "cpu")

        obj.fill_vocab_mask(mask, 0)

        inner_grammar.fill_vocab_mask.assert_not_called()
        # Excluded tokens (3, 5) should be blocked
        allowed = _allowed_token_ids(mask, [0, 1, 3, 5, 7, 8])
        self.assertEqual(allowed, [0, 1, 7, 8])
```
**EN:** This test exercises `test_thinking_phase_does_not_consult_inner_grammar` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_thinking_phase_does_not_consult_inner_grammar`。

### Lines 406-431: test case generation phase consults inner grammar / 测试用例 generation phase consults inner grammar
```python
    def test_generation_phase_consults_inner_grammar(self):
        inner_grammar = MagicMock()
        inner_grammar.allocate_vocab_mask.side_effect = lambda vs, bs, d: torch.zeros(
            (bs, (vs + 31) // 32), dtype=torch.int32
        )
        obj = ReasonerGrammarObject(
            grammar=inner_grammar,
            think_end_id=7,
            think_excluded_token_ids=[3, 5],
            max_think_tokens=-1,
            enable_token_filter=True,
            token_filter_fn=set_token_filter_torch,
            allocate_vocab_mask_fn=lambda vs, bs, d: torch.zeros(
                (bs, (vs + 31) // 32), dtype=torch.int32
            ),
            move_vocab_mask_fn=lambda vm, d: vm,
            apply_vocab_mask_fn=lambda l, vm: None,
        )
        obj.maybe_init_reasoning(True)
        obj.accept_token(10)
        obj.accept_token(7)  # think_end_id → GENERATION

        mask = obj.allocate_vocab_mask(64, 1, "cpu")
        obj.fill_vocab_mask(mask, 0)

        inner_grammar.fill_vocab_mask.assert_called_once_with(mask, 0)
```
**EN:** This test exercises `test_generation_phase_consults_inner_grammar` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_generation_phase_consults_inner_grammar`。

### Lines 433-450: test case non strict thinking is noop / 测试用例 non strict thinking is noop
```python
    def test_non_strict_thinking_is_noop(self):
        inner_grammar = MagicMock()
        obj = ReasonerGrammarObject(
            grammar=inner_grammar,
            think_end_id=7,
            think_excluded_token_ids=None,
            max_think_tokens=-1,
            enable_token_filter=False,
            token_filter_fn=None,
        )
        obj.maybe_init_reasoning(True)
        mask = torch.zeros((1, 2), dtype=torch.int32)

        obj.fill_vocab_mask(mask, 0)

        inner_grammar.fill_vocab_mask.assert_not_called()
        # Mask should remain all zeros (no filtering)
        self.assertTrue(torch.all(mask == 0))
```
**EN:** This test exercises `test_non_strict_thinking_is_noop` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_non_strict_thinking_is_noop`。

### Lines 453-454: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_DummyTokenizer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_DummyGrammarBackend`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_allowed_token_ids`: This block implements `_allowed_token_ids` and captures one focused piece of the module's behavior. / 该代码块实现 `_allowed_token_ids`，承担模块行为中的一个聚焦逻辑片段。
- `TestReasonerGrammarObject`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestReasonerGrammarBackend`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestReasonerGrammarObjectRollback`: Tests for rollback correctness at the THINKING→GENERATION boundary. / 用于组织相关测试、夹具或辅助方法。
- `TestReasonerGrammarObjectFillVocabMask`: Tests for fill_vocab_mask behavior in different states. / 用于组织相关测试、夹具或辅助方法。
- `_DummyTokenizer.__init__`: This block implements `__init__` and captures one focused piece of the module's behavior. / 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。
- `_DummyTokenizer.encode`: This block implements `encode` and captures one focused piece of the module's behavior. / 该代码块实现 `encode`，承担模块行为中的一个聚焦逻辑片段。
- `_DummyGrammarBackend.__init__`: This block implements `__init__` and captures one focused piece of the module's behavior. / 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。
- `_DummyGrammarBackend.is_support_token_filter`: This block implements `is_support_token_filter` and captures one focused piece of the module's behavior. / 该代码块实现 `is_support_token_filter`，承担模块行为中的一个聚焦逻辑片段。
- `_DummyGrammarBackend.allocate_vocab_mask`: This block implements `allocate_vocab_mask` and captures one focused piece of the module's behavior. / 该代码块实现 `allocate_vocab_mask`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `unittest`, `types`, `unittest.mock`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.constrained.base_grammar_backend`, `sglang.srt.constrained.reasoner_grammar_backend`, `sglang.srt.constrained.torch_ops.token_filter_torch_ops`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 454
