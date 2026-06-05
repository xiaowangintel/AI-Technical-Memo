# test_grammar_manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/constrained/test_grammar_manager.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates grammar manager behavior in SGLang's unit / constrained area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / constrained 领域中与 grammar manager 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: supporting statements / 辅助语句
```python
"""
Unit tests for sglang.srt.constrained.grammar_manager.

Test Coverage:
- GrammarManager initialization, queue management, len, clear
- process_req_with_grammar: dispatch by constraint type (json, regex, ebnf,
  structural_tag), no-constraint requests, no-backend error, cache hits,
  cached invalid grammar abort
- abort_requests: single abort, abort all, future cancellation
- get_ready_grammar_requests: future completion, invalid grammar handling,
  timeout with max poll iterations, aborted request handling, queue cleanup

Usage:
    python -m pytest test_grammar_manager.py -v
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 17-28: module imports and dependencies / 模块导入与依赖
```python
import unittest
from concurrent.futures import Future
from unittest.mock import MagicMock, patch

from sglang.srt.constrained.base_grammar_backend import (
    BaseGrammarBackend,
    BaseGrammarObject,
    InvalidGrammarObject,
)
from sglang.srt.constrained.grammar_manager import GrammarManager
from sglang.srt.constrained.reasoner_grammar_backend import ReasonerGrammarObject
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `concurrent.futures`, `unittest.mock`, `sglang.srt.constrained.base_grammar_backend`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `concurrent.futures`, `unittest.mock`, `sglang.srt.constrained.base_grammar_backend`。

### Lines 30-30: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(2.0, "base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 33-48: function make scheduler / 函数 make scheduler
```python
def _make_scheduler(grammar_backend_name="none", skip_tokenizer=False):
    """Create a mock scheduler with necessary attributes."""
    scheduler = MagicMock()
    scheduler.server_args.grammar_backend = grammar_backend_name
    scheduler.server_args.skip_tokenizer_init = skip_tokenizer
    scheduler.server_args.reasoning_parser = None
    scheduler.server_args.constrained_json_whitespace_pattern = None
    scheduler.server_args.constrained_json_disable_any_whitespace = False

    # Distributed group mocks
    scheduler.dp_tp_cpu_group = MagicMock()
    scheduler.dp_tp_group.world_size = 1
    scheduler.dp_tp_group.first_rank = 0
    scheduler.dp_tp_group.is_first_rank = True

    return scheduler
```
**EN:** Create a mock scheduler with necessary attributes. This block implements `_make_scheduler` and captures one focused piece of the module's behavior.
**CN:** Create a mock scheduler with necessary attributes. 该代码块实现 `_make_scheduler`，承担模块行为中的一个聚焦逻辑片段。

### Lines 51-72: function make req / 函数 make req
```python
def _make_req(
    json_schema=None,
    regex=None,
    ebnf=None,
    structural_tag=None,
    rid="req-1",
    custom_params=None,
):
    """Create a mock request with sampling params."""
    req = MagicMock()
    req.rid = rid
    req.sampling_params.json_schema = json_schema
    req.sampling_params.regex = regex
    req.sampling_params.ebnf = ebnf
    req.sampling_params.structural_tag = structural_tag
    req.sampling_params.custom_params = custom_params
    req.require_reasoning = False
    req.grammar = None
    req.grammar_key = None
    req.grammar_wait_ct = 0
    req.finished.return_value = False
    return req
```
**EN:** Create a mock request with sampling params. This block implements `_make_req` and captures one focused piece of the module's behavior.
**CN:** Create a mock request with sampling params. 该代码块实现 `_make_req`，承担模块行为中的一个聚焦逻辑片段。

### Lines 75-75: class TestGrammarManagerInit declaration / 类 TestGrammarManagerInit 声明
```python
class TestGrammarManagerInit(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 76-76: supporting statements / 辅助语句
```python
    """Test GrammarManager initialization."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 78-86: test case init with backend / 测试用例 init with backend
```python
    @patch("sglang.srt.constrained.grammar_manager.create_grammar_backend")
    def test_init_with_backend(self, mock_create):
        mock_create.return_value = MagicMock(spec=BaseGrammarBackend)
        scheduler = _make_scheduler("xgrammar")
        scheduler.server_args.skip_tokenizer_init = False

        mgr = GrammarManager(scheduler)
        self.assertIsNotNone(mgr.grammar_backend)
        self.assertEqual(len(mgr), 0)
```
**EN:** This test exercises `test_init_with_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_with_backend`。

### Lines 88-91: test case init skip tokenizer / 测试用例 init skip tokenizer
```python
    def test_init_skip_tokenizer(self):
        scheduler = _make_scheduler(skip_tokenizer=True)
        mgr = GrammarManager(scheduler)
        self.assertIsNone(mgr.grammar_backend)
```
**EN:** This test exercises `test_init_skip_tokenizer` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_skip_tokenizer`。

### Lines 93-99: test case len and has waiting / 测试用例 len and has waiting
```python
    @patch("sglang.srt.constrained.grammar_manager.create_grammar_backend")
    def test_len_and_has_waiting(self, mock_create):
        mock_create.return_value = None
        scheduler = _make_scheduler()
        mgr = GrammarManager(scheduler)
        self.assertEqual(len(mgr), 0)
        self.assertFalse(mgr.has_waiting_grammars())
```
**EN:** This test exercises `test_len_and_has_waiting` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_len_and_has_waiting`。

### Lines 101-110: test case clear resets backend / 测试用例 clear resets backend
```python
    @patch("sglang.srt.constrained.grammar_manager.create_grammar_backend")
    def test_clear_resets_backend(self, mock_create):
        mock_backend = MagicMock(spec=BaseGrammarBackend)
        mock_create.return_value = mock_backend
        scheduler = _make_scheduler()
        scheduler.server_args.skip_tokenizer_init = False

        mgr = GrammarManager(scheduler)
        mgr.clear()
        mock_backend.reset.assert_called_once()
```
**EN:** This test exercises `test_clear_resets_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_clear_resets_backend`。

### Lines 112-117: test case clear no backend / 测试用例 clear no backend
```python
    @patch("sglang.srt.constrained.grammar_manager.create_grammar_backend")
    def test_clear_no_backend(self, mock_create):
        mock_create.return_value = None
        scheduler = _make_scheduler()
        mgr = GrammarManager(scheduler)
        mgr.clear()  # Should not raise
```
**EN:** This test exercises `test_clear_no_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_clear_no_backend`。

### Lines 120-120: class TestProcessReqWithGrammar declaration / 类 TestProcessReqWithGrammar 声明
```python
class TestProcessReqWithGrammar(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 121-121: supporting statements / 辅助语句
```python
    """Test process_req_with_grammar dispatch and caching."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 123-128: method make mgr / 方法 make mgr
```python
    def _make_mgr(self):
        scheduler = _make_scheduler()
        scheduler.server_args.skip_tokenizer_init = True
        mgr = GrammarManager(scheduler)
        mgr.grammar_backend = MagicMock(spec=BaseGrammarBackend)
        return mgr
```
**EN:** This block implements `_make_mgr` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_mgr`，承担模块行为中的一个聚焦逻辑片段。

### Lines 130-135: test case no constraint returns false / 测试用例 no constraint returns false
```python
    def test_no_constraint_returns_false(self):
        mgr = self._make_mgr()
        req = _make_req()  # No constraints
        result = mgr.process_req_with_grammar(req)
        self.assertFalse(result)
        self.assertEqual(len(mgr.grammar_queue), 0)
```
**EN:** This test exercises `test_no_constraint_returns_false` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_constraint_returns_false`。

### Lines 137-147: test case json schema cache miss / 测试用例 json schema cache miss
```python
    def test_json_schema_cache_miss(self):
        mgr = self._make_mgr()
        future = Future()
        mgr.grammar_backend.get_cached_or_future_value.return_value = (future, False)

        req = _make_req(json_schema='{"type": "object"}')
        result = mgr.process_req_with_grammar(req)

        self.assertTrue(result)
        self.assertEqual(len(mgr.grammar_queue), 1)
        self.assertEqual(req.grammar_key, ("json", '{"type": "object"}'))
```
**EN:** This test exercises `test_json_schema_cache_miss` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_json_schema_cache_miss`。

### Lines 149-158: test case regex cache miss / 测试用例 regex cache miss
```python
    def test_regex_cache_miss(self):
        mgr = self._make_mgr()
        future = Future()
        mgr.grammar_backend.get_cached_or_future_value.return_value = (future, False)

        req = _make_req(regex="[a-z]+")
        result = mgr.process_req_with_grammar(req)

        self.assertTrue(result)
        self.assertEqual(req.grammar_key, ("regex", "[a-z]+"))
```
**EN:** This test exercises `test_regex_cache_miss` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_regex_cache_miss`。

### Lines 160-169: test case ebnf cache miss / 测试用例 ebnf cache miss
```python
    def test_ebnf_cache_miss(self):
        mgr = self._make_mgr()
        future = Future()
        mgr.grammar_backend.get_cached_or_future_value.return_value = (future, False)

        req = _make_req(ebnf="root ::= 'hello'")
        result = mgr.process_req_with_grammar(req)

        self.assertTrue(result)
        self.assertEqual(req.grammar_key, ("ebnf", "root ::= 'hello'"))
```
**EN:** This test exercises `test_ebnf_cache_miss` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ebnf_cache_miss`。

### Lines 171-183: test case structural tag cache miss / 测试用例 structural tag cache miss
```python
    def test_structural_tag_cache_miss(self):
        mgr = self._make_mgr()
        future = Future()
        mgr.grammar_backend.get_cached_or_future_value.return_value = (future, False)

        req = _make_req(structural_tag='{"structures": [], "triggers": []}')
        result = mgr.process_req_with_grammar(req)

        self.assertTrue(result)
        self.assertEqual(
            req.grammar_key,
            ("structural_tag", '{"structures": [], "triggers": []}'),
        )
```
**EN:** This test exercises `test_structural_tag_cache_miss` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_structural_tag_cache_miss`。

### Lines 185-199: test case cache hit returns false / 测试用例 cache hit returns false
```python
    def test_cache_hit_returns_false(self):
        """Cache hit should NOT add to grammar queue."""
        mgr = self._make_mgr()
        grammar_obj = MagicMock(spec=BaseGrammarObject)
        mgr.grammar_backend.get_cached_or_future_value.return_value = (
            grammar_obj,
            True,
        )

        req = _make_req(json_schema='{"type": "object"}')
        result = mgr.process_req_with_grammar(req)

        self.assertFalse(result)
        self.assertEqual(len(mgr.grammar_queue), 0)
        self.assertIs(req.grammar, grammar_obj)
```
**EN:** Cache hit should NOT add to grammar queue. This test exercises `test_cache_hit_returns_false` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Cache hit should NOT add to grammar queue. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_cache_hit_returns_false`。

### Lines 201-212: test case cache hit invalid grammar aborts / 测试用例 cache hit invalid grammar aborts
```python
    def test_cache_hit_invalid_grammar_aborts(self):
        """Cache hit with InvalidGrammarObject should abort the request."""
        mgr = self._make_mgr()
        invalid = InvalidGrammarObject("bad schema")
        mgr.grammar_backend.get_cached_or_future_value.return_value = (invalid, True)

        req = _make_req(json_schema="bad")
        result = mgr.process_req_with_grammar(req)

        self.assertFalse(result)
        req.set_finish_with_abort.assert_called_once()
        self.assertIn("bad schema", req.set_finish_with_abort.call_args[0][0])
```
**EN:** Cache hit with InvalidGrammarObject should abort the request. This test exercises `test_cache_hit_invalid_grammar_aborts` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Cache hit with InvalidGrammarObject should abort the request. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_cache_hit_invalid_grammar_aborts`。

### Lines 214-226: test case no backend aborts / 测试用例 no backend aborts
```python
    def test_no_backend_aborts(self):
        """No grammar backend should abort request."""
        scheduler = _make_scheduler()
        scheduler.server_args.skip_tokenizer_init = True
        mgr = GrammarManager(scheduler)
        mgr.grammar_backend = None

        req = _make_req(json_schema='{"type": "object"}')
        result = mgr.process_req_with_grammar(req)

        self.assertFalse(result)
        req.set_finish_with_abort.assert_called_once()
        self.assertIn("not supported", req.set_finish_with_abort.call_args[0][0])
```
**EN:** No grammar backend should abort request. This test exercises `test_no_backend_aborts` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** No grammar backend should abort request. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_backend_aborts`。

### Lines 228-236: test case json takes priority over other constraints / 测试用例 json takes priority over other constraints
```python
    def test_json_takes_priority_over_other_constraints(self):
        """When json_schema is set, it should be used regardless of other fields."""
        mgr = self._make_mgr()
        future = Future()
        mgr.grammar_backend.get_cached_or_future_value.return_value = (future, False)

        req = _make_req(json_schema='{"type": "object"}', regex="[a-z]+")
        mgr.process_req_with_grammar(req)
        self.assertEqual(req.grammar_key, ("json", '{"type": "object"}'))
```
**EN:** When json_schema is set, it should be used regardless of other fields. This test exercises `test_json_takes_priority_over_other_constraints` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When json_schema is set, it should be used regardless of other fields. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_json_takes_priority_over_other_constraints`。

### Lines 238-253: test case require reasoning forwarded to backend / 测试用例 require reasoning forwarded to backend
```python
    def test_require_reasoning_forwarded_to_backend(self):
        """require_reasoning from the request should be passed to the backend."""
        mgr = self._make_mgr()
        grammar_obj = MagicMock(spec=BaseGrammarObject)
        mgr.grammar_backend.get_cached_or_future_value.return_value = (
            grammar_obj,
            True,
        )

        req = _make_req(json_schema="schema")
        req.require_reasoning = True
        mgr.process_req_with_grammar(req)

        mgr.grammar_backend.get_cached_or_future_value.assert_called_once_with(
            ("json", "schema"), True
        )
```
**EN:** require_reasoning from the request should be passed to the backend. This test exercises `test_require_reasoning_forwarded_to_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** require_reasoning from the request should be passed to the backend. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_require_reasoning_forwarded_to_backend`。

### Lines 255-264: test case has waiting grammars after enqueue / 测试用例 has waiting grammars after enqueue
```python
    def test_has_waiting_grammars_after_enqueue(self):
        mgr = self._make_mgr()
        future = Future()
        mgr.grammar_backend.get_cached_or_future_value.return_value = (future, False)

        self.assertFalse(mgr.has_waiting_grammars())
        req = _make_req(json_schema="schema")
        mgr.process_req_with_grammar(req)
        self.assertTrue(mgr.has_waiting_grammars())
        self.assertEqual(len(mgr), 1)
```
**EN:** This test exercises `test_has_waiting_grammars_after_enqueue` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_waiting_grammars_after_enqueue`。

### Lines 266-282: test case cache hit applies request thinking budget / 测试用例 cache hit applies request thinking budget
```python
    def test_cache_hit_applies_request_thinking_budget(self):
        mgr = self._make_mgr()
        grammar_obj = ReasonerGrammarObject(
            grammar=None, think_end_id=0, max_think_tokens=99
        )
        mgr.grammar_backend.get_cached_or_future_value.return_value = (
            grammar_obj,
            True,
        )

        req = _make_req(
            json_schema="schema",
            custom_params={"thinking_budget": 7},
        )
        mgr.process_req_with_grammar(req)

        self.assertEqual(req.grammar.max_think_tokens, 7)
```
**EN:** This test exercises `test_cache_hit_applies_request_thinking_budget` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_cache_hit_applies_request_thinking_budget`。

### Lines 284-297: test case strict reasoning grammar applies request thinking budget / 测试用例 strict reasoning grammar applies request thinking budget
```python
    def test_strict_reasoning_grammar_applies_request_thinking_budget(self):
        mgr = self._make_mgr()
        mgr._enable_strict_thinking = True
        grammar_obj = ReasonerGrammarObject(
            grammar=None, think_end_id=0, max_think_tokens=99
        )
        mgr.grammar_backend.init_strict_reasoning_grammar.return_value = grammar_obj

        req = _make_req(custom_params={"thinking_budget": 3})
        req.require_reasoning = True
        mgr.process_req_with_grammar(req)

        self.assertIs(req.grammar, grammar_obj)
        self.assertEqual(req.grammar.max_think_tokens, 3)
```
**EN:** This test exercises `test_strict_reasoning_grammar_applies_request_thinking_budget` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_strict_reasoning_grammar_applies_request_thinking_budget`。

### Lines 300-300: class TestAbortRequests declaration / 类 TestAbortRequests 声明
```python
class TestAbortRequests(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 301-301: supporting statements / 辅助语句
```python
    """Test abort_requests handling."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 303-308: method make mgr with queue / 方法 make mgr with queue
```python
    def _make_mgr_with_queue(self):
        scheduler = _make_scheduler()
        scheduler.server_args.skip_tokenizer_init = True
        mgr = GrammarManager(scheduler)
        mgr.grammar_backend = MagicMock(spec=BaseGrammarBackend)
        return mgr
```
**EN:** This block implements `_make_mgr_with_queue` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_mgr_with_queue`，承担模块行为中的一个聚焦逻辑片段。

### Lines 310-323: test case abort by rid prefix / 测试用例 abort by rid prefix
```python
    def test_abort_by_rid_prefix(self):
        mgr = self._make_mgr_with_queue()
        req = _make_req(rid="req-123")
        future = MagicMock(spec=Future)
        req.grammar = future
        mgr.grammar_queue.append(req)

        abort_req = MagicMock()
        abort_req.abort_all = False
        abort_req.rid = "req-123"

        mgr.abort_requests(abort_req)
        future.cancel.assert_called_once()
        req.set_finish_with_abort.assert_called_once()
```
**EN:** This test exercises `test_abort_by_rid_prefix` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_abort_by_rid_prefix`。

### Lines 325-336: test case abort non matching rid / 测试用例 abort non matching rid
```python
    def test_abort_non_matching_rid(self):
        mgr = self._make_mgr_with_queue()
        req = _make_req(rid="req-999")
        req.grammar = MagicMock(spec=Future)
        mgr.grammar_queue.append(req)

        abort_req = MagicMock()
        abort_req.abort_all = False
        abort_req.rid = "req-123"

        mgr.abort_requests(abort_req)
        req.set_finish_with_abort.assert_not_called()
```
**EN:** This test exercises `test_abort_non_matching_rid` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_abort_non_matching_rid`。

### Lines 338-353: test case abort all / 测试用例 abort all
```python
    def test_abort_all(self):
        mgr = self._make_mgr_with_queue()
        reqs = []
        for i in range(3):
            req = _make_req(rid=f"req-{i}")
            req.grammar = MagicMock(spec=Future)
            mgr.grammar_queue.append(req)
            reqs.append(req)

        abort_req = MagicMock()
        abort_req.abort_all = True
        abort_req.rid = ""

        mgr.abort_requests(abort_req)
        for req in reqs:
            req.set_finish_with_abort.assert_called_once()
```
**EN:** This test exercises `test_abort_all` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_abort_all`。

### Lines 355-361: test case abort empty queue / 测试用例 abort empty queue
```python
    def test_abort_empty_queue(self):
        """Aborting on an empty queue should not raise."""
        mgr = self._make_mgr_with_queue()
        abort_req = MagicMock()
        abort_req.abort_all = True
        abort_req.rid = ""
        mgr.abort_requests(abort_req)  # Should not raise
```
**EN:** Aborting on an empty queue should not raise. This test exercises `test_abort_empty_queue` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Aborting on an empty queue should not raise. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_abort_empty_queue`。

### Lines 363-375: test case abort prefix match / 测试用例 abort prefix match
```python
    def test_abort_prefix_match(self):
        """rid.startswith means prefix matching, not exact matching."""
        mgr = self._make_mgr_with_queue()
        req = _make_req(rid="req-123-suffix")
        req.grammar = MagicMock(spec=Future)
        mgr.grammar_queue.append(req)

        abort_req = MagicMock()
        abort_req.abort_all = False
        abort_req.rid = "req-123"

        mgr.abort_requests(abort_req)
        req.set_finish_with_abort.assert_called_once()
```
**EN:** rid.startswith means prefix matching, not exact matching. This test exercises `test_abort_prefix_match` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** rid.startswith means prefix matching, not exact matching. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_abort_prefix_match`。

### Lines 378-378: class TestGetReadyGrammarRequests declaration / 类 TestGetReadyGrammarRequests 声明
```python
class TestGetReadyGrammarRequests(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 379-379: supporting statements / 辅助语句
```python
    """Test get_ready_grammar_requests polling and result handling."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 381-389: method make mgr / 方法 make mgr
```python
    def _make_mgr(self):
        scheduler = _make_scheduler()
        scheduler.server_args.skip_tokenizer_init = True
        mgr = GrammarManager(scheduler)
        mgr.grammar_backend = MagicMock(spec=BaseGrammarBackend)
        # Use very short poll interval for tests
        mgr.SGLANG_GRAMMAR_POLL_INTERVAL = 0.01
        mgr.SGLANG_GRAMMAR_MAX_POLL_ITERATIONS = 3
        return mgr
```
**EN:** This block implements `_make_mgr` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_mgr`，承担模块行为中的一个聚焦逻辑片段。

### Lines 391-411: test case ready future returns req / 测试用例 ready future returns req
```python
    def test_ready_future_returns_req(self):
        mgr = self._make_mgr()

        grammar_obj = MagicMock(spec=BaseGrammarObject)
        grammar_obj.copy.return_value = grammar_obj
        future = Future()
        future.set_result(grammar_obj)

        req = _make_req(json_schema="schema")
        req.grammar = future
        req.grammar_key = ("json", "schema")
        mgr.grammar_queue.append(req)

        result = mgr.get_ready_grammar_requests()
        self.assertEqual(len(result), 1)
        self.assertIs(result[0], req)
        self.assertIs(req.grammar, grammar_obj)
        # Cache should be set
        mgr.grammar_backend.set_cache.assert_called_once()
        # Queue should be empty
        self.assertEqual(len(mgr.grammar_queue), 0)
```
**EN:** This test exercises `test_ready_future_returns_req` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ready_future_returns_req`。

### Lines 413-430: test case invalid grammar aborts req / 测试用例 invalid grammar aborts req
```python
    def test_invalid_grammar_aborts_req(self):
        mgr = self._make_mgr()

        invalid = InvalidGrammarObject("compile error")
        invalid_copy = InvalidGrammarObject("compile error")
        invalid.copy = MagicMock(return_value=invalid_copy)
        future = Future()
        future.set_result(invalid)

        req = _make_req(json_schema="bad")
        req.grammar = future
        req.grammar_key = ("json", "bad")
        mgr.grammar_queue.append(req)

        result = mgr.get_ready_grammar_requests()
        self.assertEqual(len(result), 1)
        req.set_finish_with_abort.assert_called_once()
        self.assertIn("compile error", req.set_finish_with_abort.call_args[0][0])
```
**EN:** This test exercises `test_invalid_grammar_aborts_req` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_invalid_grammar_aborts_req`。

### Lines 432-442: test case aborted req removed from queue / 测试用例 aborted req removed from queue
```python
    def test_aborted_req_removed_from_queue(self):
        mgr = self._make_mgr()

        req = _make_req(json_schema="schema")
        req.finished.return_value = True  # Already aborted
        req.grammar = None
        mgr.grammar_queue.append(req)

        result = mgr.get_ready_grammar_requests()
        self.assertEqual(len(result), 1)
        self.assertEqual(len(mgr.grammar_queue), 0)
```
**EN:** This test exercises `test_aborted_req_removed_from_queue` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_aborted_req_removed_from_queue`。

### Lines 444-465: test case timeout aborts req / 测试用例 timeout aborts req
```python
    def test_timeout_aborts_req(self):
        mgr = self._make_mgr()
        mgr.SGLANG_GRAMMAR_MAX_POLL_ITERATIONS = 1

        future = Future()  # Never completes
        req = _make_req(json_schema="slow")
        req.grammar = future
        req.grammar_key = ("json", "slow")
        req.grammar_wait_ct = 0
        mgr.grammar_queue.append(req)

        # First call: not ready, increments wait_ct to 1 (== max_poll)
        result = mgr.get_ready_grammar_requests()
        # Should timeout and abort
        self.assertEqual(len(result), 1)
        req.set_finish_with_abort.assert_called_once()
        self.assertIn("timed out", req.set_finish_with_abort.call_args[0][0])
        # Cache should store InvalidGrammarObject for timeout
        mgr.grammar_backend.set_cache.assert_called_once()
        cached_key, cached_val = mgr.grammar_backend.set_cache.call_args[0]
        self.assertEqual(cached_key, ("json", "slow"))
        self.assertIsInstance(cached_val, InvalidGrammarObject)
```
**EN:** This test exercises `test_timeout_aborts_req` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_timeout_aborts_req`。

### Lines 467-482: test case pending future stays in queue / 测试用例 pending future stays in queue
```python
    def test_pending_future_stays_in_queue(self):
        """Futures that aren't done stay in the queue."""
        mgr = self._make_mgr()
        mgr.SGLANG_GRAMMAR_MAX_POLL_ITERATIONS = 100  # High to avoid timeout

        future = Future()  # Never completes
        req = _make_req(json_schema="pending")
        req.grammar = future
        req.grammar_key = ("json", "pending")
        req.grammar_wait_ct = 0
        mgr.grammar_queue.append(req)

        result = mgr.get_ready_grammar_requests()
        self.assertEqual(len(result), 0)
        self.assertEqual(len(mgr.grammar_queue), 1)
        self.assertEqual(req.grammar_wait_ct, 1)
```
**EN:** Futures that aren't done stay in the queue. This test exercises `test_pending_future_stays_in_queue` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Futures that aren't done stay in the queue. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pending_future_stays_in_queue`。

### Lines 484-510: test case mixed ready and pending / 测试用例 mixed ready and pending
```python
    def test_mixed_ready_and_pending(self):
        mgr = self._make_mgr()
        mgr.SGLANG_GRAMMAR_MAX_POLL_ITERATIONS = 100

        # Ready request
        grammar_obj = MagicMock(spec=BaseGrammarObject)
        grammar_obj.copy.return_value = grammar_obj
        done_future = Future()
        done_future.set_result(grammar_obj)
        ready_req = _make_req(json_schema="ready", rid="r1")
        ready_req.grammar = done_future
        ready_req.grammar_key = ("json", "ready")

        # Pending request
        pending_future = Future()
        pending_req = _make_req(json_schema="pending", rid="r2")
        pending_req.grammar = pending_future
        pending_req.grammar_key = ("json", "pending")
        pending_req.grammar_wait_ct = 0

        mgr.grammar_queue = [ready_req, pending_req]

        result = mgr.get_ready_grammar_requests()
        self.assertEqual(len(result), 1)
        self.assertIs(result[0], ready_req)
        self.assertEqual(len(mgr.grammar_queue), 1)
        self.assertIs(mgr.grammar_queue[0], pending_req)
```
**EN:** This test exercises `test_mixed_ready_and_pending` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mixed_ready_and_pending`。

### Lines 512-517: test case empty queue / 测试用例 empty queue
```python
    def test_empty_queue(self):
        """get_ready_grammar_requests on empty queue should return empty list."""
        mgr = self._make_mgr()
        result = mgr.get_ready_grammar_requests()
        self.assertEqual(len(result), 0)
        self.assertEqual(len(mgr.grammar_queue), 0)
```
**EN:** get_ready_grammar_requests on empty queue should return empty list. This test exercises `test_empty_queue` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** get_ready_grammar_requests on empty queue should return empty list. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_queue`。

### Lines 519-535: test case progressive timeout / 测试用例 progressive timeout
```python
    def test_progressive_timeout(self):
        """Request with partial wait_ct should timeout after remaining iterations."""
        mgr = self._make_mgr()
        mgr.SGLANG_GRAMMAR_MAX_POLL_ITERATIONS = 3

        future = Future()  # Never completes
        req = _make_req(json_schema="slow")
        req.grammar = future
        req.grammar_key = ("json", "slow")
        req.grammar_wait_ct = 2  # Already waited 2 iterations
        mgr.grammar_queue.append(req)

        # wait_ct increments to 3 (== max), should timeout
        result = mgr.get_ready_grammar_requests()
        self.assertEqual(len(result), 1)
        req.set_finish_with_abort.assert_called_once()
        self.assertIn("timed out", req.set_finish_with_abort.call_args[0][0])
```
**EN:** Request with partial wait_ct should timeout after remaining iterations. This test exercises `test_progressive_timeout` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Request with partial wait_ct should timeout after remaining iterations. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_progressive_timeout`。

### Lines 537-552: test case future exception creates invalid grammar object / 测试用例 future exception creates invalid grammar object
```python
    def test_future_exception_creates_invalid_grammar_object(self):
        """A future that raised an exception should create InvalidGrammarObject, not crash."""
        mgr = self._make_mgr()

        future = Future()
        future.set_exception(RuntimeError("compilation crashed"))

        req = _make_req(json_schema="crash")
        req.grammar = future
        req.grammar_key = ("json", "crash")
        mgr.grammar_queue.append(req)

        result = mgr.get_ready_grammar_requests()
        self.assertEqual(len(result), 1)
        self.assertIsInstance(result[0].grammar, InvalidGrammarObject)
        req.set_finish_with_abort.assert_called_once()
```
**EN:** A future that raised an exception should create InvalidGrammarObject, not crash. This test exercises `test_future_exception_creates_invalid_grammar_object` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** A future that raised an exception should create InvalidGrammarObject, not crash. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_future_exception_creates_invalid_grammar_object`。

### Lines 554-574: test case ready future applies request budget without polluting cache / 测试用例 ready future applies request budget without polluting cache
```python
    def test_ready_future_applies_request_budget_without_polluting_cache(self):
        mgr = self._make_mgr()

        grammar_obj = ReasonerGrammarObject(
            grammar=None, think_end_id=0, max_think_tokens=99
        )
        future = Future()
        future.set_result(grammar_obj)

        req = _make_req(json_schema="schema", custom_params={"thinking_budget": 4})
        req.grammar = future
        req.grammar_key = ("json", "schema")
        mgr.grammar_queue.append(req)

        result = mgr.get_ready_grammar_requests()

        self.assertEqual(len(result), 1)
        self.assertEqual(req.grammar.max_think_tokens, 4)
        cached_key, cached_value = mgr.grammar_backend.set_cache.call_args[0]
        self.assertEqual(cached_key, ("json", "schema"))
        self.assertEqual(cached_value.max_think_tokens, 99)
```
**EN:** This test exercises `test_ready_future_applies_request_budget_without_polluting_cache` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ready_future_applies_request_budget_without_polluting_cache`。

### Lines 576-614: test case multi rank sync intersects ready unions failed / 测试用例 multi rank sync intersects ready unions failed
```python
    @patch("sglang.srt.constrained.grammar_manager.torch.distributed.all_gather_object")
    def test_multi_rank_sync_intersects_ready_unions_failed(self, mock_all_gather):
        """With multiple ranks, ready = intersection, failed = union."""
        mgr = self._make_mgr()
        mgr.grammar_sync_size = 2  # Enable multi-rank path

        # Two requests: idx 0 ready on both ranks, idx 1 ready only on rank 0
        grammar_obj = MagicMock(spec=BaseGrammarObject)
        grammar_obj.copy.return_value = grammar_obj
        done_future = Future()
        done_future.set_result(grammar_obj)

        req0 = _make_req(json_schema="s0", rid="r0")
        req0.grammar = done_future
        req0.grammar_key = ("json", "s0")

        pending_future = Future()
        req1 = _make_req(json_schema="s1", rid="r1")
        req1.grammar = pending_future
        req1.grammar_key = ("json", "s1")
        req1.grammar_wait_ct = 0

        mgr.grammar_queue = [req0, req1]
        mgr.SGLANG_GRAMMAR_MAX_POLL_ITERATIONS = 100

        # Simulate all_gather: rank 0 has {0} ready, rank 1 has {0,1} ready
        def fake_all_gather(output_list, _obj, group=None):  # noqa: ARG001
            output_list[0] = ({0}, set())  # rank 0: only idx 0 ready
            output_list[1] = ({0, 1}, set())  # rank 1: both ready

        mock_all_gather.side_effect = fake_all_gather

        result = mgr.get_ready_grammar_requests()
        # Intersection of ready: {0} ∩ {0,1} = {0}
        self.assertEqual(len(result), 1)
        self.assertIs(result[0], req0)
        # req1 stays in queue
        self.assertEqual(len(mgr.grammar_queue), 1)
        self.assertIs(mgr.grammar_queue[0], req1)
```
**EN:** With multiple ranks, ready = intersection, failed = union. This test exercises `test_multi_rank_sync_intersects_ready_unions_failed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** With multiple ranks, ready = intersection, failed = union. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multi_rank_sync_intersects_ready_unions_failed`。

### Lines 616-643: test case multi rank sync unions failed / 测试用例 multi rank sync unions failed
```python
    @patch("sglang.srt.constrained.grammar_manager.torch.distributed.all_gather_object")
    def test_multi_rank_sync_unions_failed(self, mock_all_gather):
        """Failed requests from any rank should be unioned."""
        mgr = self._make_mgr()
        mgr.grammar_sync_size = 2
        mgr.SGLANG_GRAMMAR_MAX_POLL_ITERATIONS = 1

        pending_future = Future()  # Never completes
        req = _make_req(json_schema="slow", rid="r0")
        req.grammar = pending_future
        req.grammar_key = ("json", "slow")
        req.grammar_wait_ct = 0

        mgr.grammar_queue = [req]

        # Simulate: rank 0 has no ready and idx 0 failed, rank 1 has no ready/failed
        def fake_all_gather(output_list, _obj, group=None):  # noqa: ARG001
            output_list[0] = (set(), {0})  # rank 0: idx 0 timed out
            output_list[1] = (set(), set())  # rank 1: nothing

        mock_all_gather.side_effect = fake_all_gather

        result = mgr.get_ready_grammar_requests()
        # Union of failed: {} ∪ {0} = {0}
        self.assertEqual(len(result), 1)
        req.set_finish_with_abort.assert_called_once()
        self.assertIn("timed out", req.set_finish_with_abort.call_args[0][0])
        self.assertEqual(len(mgr.grammar_queue), 0)
```
**EN:** Failed requests from any rank should be unioned. This test exercises `test_multi_rank_sync_unions_failed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Failed requests from any rank should be unioned. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multi_rank_sync_unions_failed`。

### Lines 646-646: class TestStrictReasoningPaths declaration / 类 TestStrictReasoningPaths 声明
```python
class TestStrictReasoningPaths(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 647-647: supporting statements / 辅助语句
```python
    """Test _enable_strict_thinking code paths in GrammarManager."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 649-655: method make mgr / 方法 make mgr
```python
    def _make_mgr(self):
        scheduler = _make_scheduler()
        scheduler.server_args.skip_tokenizer_init = True
        mgr = GrammarManager(scheduler)
        mgr.grammar_backend = MagicMock(spec=BaseGrammarBackend)
        mgr._enable_strict_thinking = True
        return mgr
```
**EN:** This block implements `_make_mgr` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_mgr`，承担模块行为中的一个聚焦逻辑片段。

### Lines 657-669: test case strict unconstrained request gets strict grammar / 测试用例 strict unconstrained request gets strict grammar
```python
    def test_strict_unconstrained_request_gets_strict_grammar(self):
        """Request without json_schema/regex/ebnf should get strict-only grammar."""
        mgr = self._make_mgr()
        grammar_obj = MagicMock()
        mgr.grammar_backend.init_strict_reasoning_grammar.return_value = grammar_obj

        req = _make_req()  # No constraint
        req.require_reasoning = True
        result = mgr.process_req_with_grammar(req)

        self.assertFalse(result)  # Not added to grammar queue
        self.assertIs(req.grammar, grammar_obj)
        mgr.grammar_backend.init_strict_reasoning_grammar.assert_called_once_with(True)
```
**EN:** Request without json_schema/regex/ebnf should get strict-only grammar. This test exercises `test_strict_unconstrained_request_gets_strict_grammar` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Request without json_schema/regex/ebnf should get strict-only grammar. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_strict_unconstrained_request_gets_strict_grammar`。

### Lines 671-682: test case strict unconstrained no reasoning flag / 测试用例 strict unconstrained no reasoning flag
```python
    def test_strict_unconstrained_no_reasoning_flag(self):
        """Unconstrained request with require_reasoning=False still gets strict grammar."""
        mgr = self._make_mgr()
        grammar_obj = MagicMock()
        mgr.grammar_backend.init_strict_reasoning_grammar.return_value = grammar_obj

        req = _make_req()
        req.require_reasoning = False
        mgr.process_req_with_grammar(req)

        self.assertIs(req.grammar, grammar_obj)
        mgr.grammar_backend.init_strict_reasoning_grammar.assert_called_once_with(False)
```
**EN:** Unconstrained request with require_reasoning=False still gets strict grammar. This test exercises `test_strict_unconstrained_no_reasoning_flag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Unconstrained request with require_reasoning=False still gets strict grammar. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_strict_unconstrained_no_reasoning_flag`。

### Lines 684-693: test case strict unconstrained none grammar is fine / 测试用例 strict unconstrained none grammar is fine
```python
    def test_strict_unconstrained_none_grammar_is_fine(self):
        """If init_strict_reasoning_grammar returns None, req.grammar stays None."""
        mgr = self._make_mgr()
        mgr.grammar_backend.init_strict_reasoning_grammar.return_value = None

        req = _make_req()
        req.require_reasoning = True
        mgr.process_req_with_grammar(req)

        self.assertIsNone(req.grammar)
```
**EN:** If init_strict_reasoning_grammar returns None, req.grammar stays None. This test exercises `test_strict_unconstrained_none_grammar_is_fine` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** If init_strict_reasoning_grammar returns None, req.grammar stays None. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_strict_unconstrained_none_grammar_is_fine`。

### Lines 695-706: test case strict constrained request uses normal dispatch / 测试用例 strict constrained request uses normal dispatch
```python
    def test_strict_constrained_request_uses_normal_dispatch(self):
        """Request with json_schema should go through normal dispatch, not strict path."""
        mgr = self._make_mgr()
        future = MagicMock(spec=Future)
        mgr.grammar_backend.get_cached_or_future_value.return_value = (future, False)

        req = _make_req(json_schema='{"type": "object"}')
        req.require_reasoning = True
        result = mgr.process_req_with_grammar(req)

        self.assertTrue(result)  # Added to grammar queue
        mgr.grammar_backend.init_strict_reasoning_grammar.assert_not_called()
```
**EN:** Request with json_schema should go through normal dispatch, not strict path. This test exercises `test_strict_constrained_request_uses_normal_dispatch` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Request with json_schema should go through normal dispatch, not strict path. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_strict_constrained_request_uses_normal_dispatch`。

### Lines 708-718: test case strict not set skips strict path / 测试用例 strict not set skips strict path
```python
    def test_strict_not_set_skips_strict_path(self):
        """When _enable_strict_thinking=False, unconstrained requests get no grammar."""
        mgr = self._make_mgr()
        mgr._enable_strict_thinking = False

        req = _make_req()
        req.require_reasoning = True
        mgr.process_req_with_grammar(req)

        self.assertIsNone(req.grammar)
        mgr.grammar_backend.init_strict_reasoning_grammar.assert_not_called()
```
**EN:** When _enable_strict_thinking=False, unconstrained requests get no grammar. This test exercises `test_strict_not_set_skips_strict_path` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When _enable_strict_thinking=False, unconstrained requests get no grammar. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_strict_not_set_skips_strict_path`。

### Lines 720-738: test case future exception creates invalid grammar / 测试用例 future exception creates invalid grammar
```python
    def test_future_exception_creates_invalid_grammar(self):
        """Future.result() raising should create InvalidGrammarObject, not crash."""
        mgr = self._make_mgr()

        future = Future()
        future.set_exception(RuntimeError("compilation failed"))

        req = _make_req(json_schema='{"type": "object"}')
        req.require_reasoning = True
        req.grammar = future
        req.grammar_key = ("json", '{"type": "object"}')
        mgr.grammar_queue.append(req)

        mgr.SGLANG_GRAMMAR_POLL_INTERVAL = 0.001
        result = mgr.get_ready_grammar_requests()

        self.assertEqual(len(result), 1)
        self.assertIsInstance(result[0].grammar, InvalidGrammarObject)
        req.set_finish_with_abort.assert_called_once()
```
**EN:** Future.result() raising should create InvalidGrammarObject, not crash. This test exercises `test_future_exception_creates_invalid_grammar` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Future.result() raising should create InvalidGrammarObject, not crash. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_future_exception_creates_invalid_grammar`。

### Lines 741-742: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_make_scheduler`: Create a mock scheduler with necessary attributes. / 该代码块实现 `_make_scheduler`，承担模块行为中的一个聚焦逻辑片段。
- `_make_req`: Create a mock request with sampling params. / 该代码块实现 `_make_req`，承担模块行为中的一个聚焦逻辑片段。
- `TestGrammarManagerInit`: Test GrammarManager initialization. / 用于组织相关测试、夹具或辅助方法。
- `TestProcessReqWithGrammar`: Test process_req_with_grammar dispatch and caching. / 用于组织相关测试、夹具或辅助方法。
- `TestAbortRequests`: Test abort_requests handling. / 用于组织相关测试、夹具或辅助方法。
- `TestGetReadyGrammarRequests`: Test get_ready_grammar_requests polling and result handling. / 用于组织相关测试、夹具或辅助方法。
- `TestStrictReasoningPaths`: Test _enable_strict_thinking code paths in GrammarManager. / 用于组织相关测试、夹具或辅助方法。
- `TestGrammarManagerInit.test_init_with_backend`: This test exercises `test_init_with_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_with_backend`。
- `TestGrammarManagerInit.test_init_skip_tokenizer`: This test exercises `test_init_skip_tokenizer` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_skip_tokenizer`。
- `TestGrammarManagerInit.test_len_and_has_waiting`: This test exercises `test_len_and_has_waiting` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_len_and_has_waiting`。
- `TestGrammarManagerInit.test_clear_resets_backend`: This test exercises `test_clear_resets_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_clear_resets_backend`。
- `TestGrammarManagerInit.test_clear_no_backend`: This test exercises `test_clear_no_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_clear_no_backend`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `concurrent.futures`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.srt.constrained.base_grammar_backend`, `sglang.srt.constrained.grammar_manager`, `sglang.srt.constrained.reasoner_grammar_backend`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 742
