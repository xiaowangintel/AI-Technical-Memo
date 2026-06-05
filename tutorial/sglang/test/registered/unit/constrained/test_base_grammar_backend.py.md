# test_base_grammar_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/constrained/test_base_grammar_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates base grammar backend behavior in SGLang's unit / constrained area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / constrained 领域中与 base grammar backend 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: supporting statements / 辅助语句
```python
"""
Unit tests for sglang.srt.constrained.base_grammar_backend.

Test Coverage:
- GrammarStats: default values, mutable default isolation
- BaseGrammarObject: default behavior
- InvalidGrammarObject: error message
- BaseGrammarBackend: caching, dispatch routing, unsupported fallback,
  thread pool execution, cache hit/miss
- create_grammar_backend: factory routing, "none" backend, invalid name,
  custom registry, reasoner wrapping
- register_grammar_backend: registration and lookup

Usage:
    python -m pytest test_base_grammar_backend.py -v
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 18-31: module imports and dependencies / 模块导入与依赖
```python
import unittest
from concurrent.futures import Future
from unittest.mock import MagicMock, patch

from sglang.srt.constrained.base_grammar_backend import (
    GRAMMAR_BACKEND_REGISTRY,
    BaseGrammarBackend,
    BaseGrammarObject,
    GrammarStats,
    InvalidGrammarObject,
    create_grammar_backend,
    register_grammar_backend,
)
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `concurrent.futures`, `unittest.mock`, `sglang.srt.constrained.base_grammar_backend`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `concurrent.futures`, `unittest.mock`, `sglang.srt.constrained.base_grammar_backend`。

### Lines 33-33: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(2.0, "base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 36-36: class TestGrammarStats declaration / 类 TestGrammarStats 声明
```python
class TestGrammarStats(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 37-37: supporting statements / 辅助语句
```python
    """Test GrammarStats dataclass."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 39-48: test case defaults / 测试用例 defaults
```python
    def test_defaults(self):
        stats = GrammarStats()
        self.assertIsNone(stats.compilation_time)
        self.assertIsNone(stats.schema_count)
        self.assertIsNone(stats.ebnf_size)
        self.assertFalse(stats.is_cache_hit)
        self.assertFalse(stats.is_grammar_aborted)
        self.assertEqual(stats.tree_traversal_time, [])
        self.assertIsNone(stats.dispatch_type)
        self.assertEqual(stats.num_timeout, 0)
```
**EN:** This test exercises `test_defaults` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_defaults`。

### Lines 50-55: test case tree traversal time mutable default / 测试用例 tree traversal time mutable default
```python
    def test_tree_traversal_time_mutable_default(self):
        """Ensure each instance gets its own list."""
        s1 = GrammarStats()
        s2 = GrammarStats()
        s1.tree_traversal_time.append(0.1)
        self.assertEqual(len(s2.tree_traversal_time), 0)
```
**EN:** Ensure each instance gets its own list. This test exercises `test_tree_traversal_time_mutable_default` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Ensure each instance gets its own list. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tree_traversal_time_mutable_default`。

### Lines 58-58: class TestBaseGrammarObject declaration / 类 TestBaseGrammarObject 声明
```python
class TestBaseGrammarObject(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 59-59: supporting statements / 辅助语句
```python
    """Test BaseGrammarObject base class."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 61-63: test case is terminated default / 测试用例 is terminated default
```python
    def test_is_terminated_default(self):
        obj = BaseGrammarObject()
        self.assertFalse(obj.is_terminated())
```
**EN:** This test exercises `test_is_terminated_default` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_is_terminated_default`。

### Lines 65-67: test case maybe init reasoning noop / 测试用例 maybe init reasoning noop
```python
    def test_maybe_init_reasoning_noop(self):
        obj = BaseGrammarObject()
        obj.maybe_init_reasoning(True)  # Should not raise
```
**EN:** This test exercises `test_maybe_init_reasoning_noop` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_maybe_init_reasoning_noop`。

### Lines 70-70: class TestInvalidGrammarObject declaration / 类 TestInvalidGrammarObject 声明
```python
class TestInvalidGrammarObject(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 71-71: supporting statements / 辅助语句
```python
    """Test InvalidGrammarObject."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 73-75: test case default error message / 测试用例 default error message
```python
    def test_default_error_message(self):
        obj = InvalidGrammarObject()
        self.assertEqual(obj.error_message, "Unknown grammar error")
```
**EN:** This test exercises `test_default_error_message` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_error_message`。

### Lines 77-79: test case custom error message / 测试用例 custom error message
```python
    def test_custom_error_message(self):
        obj = InvalidGrammarObject("Regex compilation failed")
        self.assertEqual(obj.error_message, "Regex compilation failed")
```
**EN:** This test exercises `test_custom_error_message` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_custom_error_message`。

### Lines 82-82: class TestBaseGrammarBackend declaration / 类 TestBaseGrammarBackend 声明
```python
class TestBaseGrammarBackend(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 83-83: supporting statements / 辅助语句
```python
    """Test BaseGrammarBackend caching and dispatch."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 85-86: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.backend = BaseGrammarBackend()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 88-89: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        self.backend.executor.shutdown(wait=True)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 91-96: test case set and get cache / 测试用例 set and get cache
```python
    def test_set_and_get_cache(self):
        obj = BaseGrammarObject()
        key = ("json", '{"type": "object"}')
        self.backend.set_cache(key, obj)
        self.assertIn(key, self.backend.cache)
        self.assertIs(self.backend.cache[key], obj)
```
**EN:** This test exercises `test_set_and_get_cache` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_set_and_get_cache`。

### Lines 98-101: test case reset clears cache / 测试用例 reset clears cache
```python
    def test_reset_clears_cache(self):
        self.backend.set_cache(("json", "schema"), BaseGrammarObject())
        self.backend.reset()
        self.assertEqual(len(self.backend.cache), 0)
```
**EN:** This test exercises `test_reset_clears_cache` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reset_clears_cache`。

### Lines 103-115: test case cache hit returns copy / 测试用例 cache hit returns copy
```python
    def test_cache_hit_returns_copy(self):
        """Cache hit should return a copy of the cached object."""
        mock_copy = BaseGrammarObject()
        obj = MagicMock(spec=BaseGrammarObject)
        obj.copy.return_value = mock_copy

        key = ("json", "schema")
        self.backend.set_cache(key, obj)
        result, cache_hit = self.backend.get_cached_or_future_value(key, False)

        self.assertTrue(cache_hit)
        obj.copy.assert_called_once()
        self.assertIs(result, mock_copy)
```
**EN:** Cache hit should return a copy of the cached object. This test exercises `test_cache_hit_returns_copy` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Cache hit should return a copy of the cached object. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_cache_hit_returns_copy`。

### Lines 117-125: test case cache hit inits reasoning / 测试用例 cache hit inits reasoning
```python
    def test_cache_hit_inits_reasoning(self):
        obj = MagicMock(spec=BaseGrammarObject)
        copied = MagicMock(spec=BaseGrammarObject)
        obj.copy.return_value = copied

        key = ("json", "schema")
        self.backend.set_cache(key, obj)
        self.backend.get_cached_or_future_value(key, True)
        copied.maybe_init_reasoning.assert_called_once_with(True)
```
**EN:** This test exercises `test_cache_hit_inits_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_cache_hit_inits_reasoning`。

### Lines 127-134: test case cache miss returns future / 测试用例 cache miss returns future
```python
    def test_cache_miss_returns_future(self):
        key = ("json", "schema")
        result, cache_hit = self.backend.get_cached_or_future_value(key, False)
        self.assertFalse(cache_hit)
        self.assertIsInstance(result, Future)
        # The future should complete (dispatch_json returns InvalidGrammarObject)
        value = result.result(timeout=5)
        self.assertIsInstance(value, InvalidGrammarObject)
```
**EN:** This test exercises `test_cache_miss_returns_future` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_cache_miss_returns_future`。

### Lines 136-147: test case all dispatch methods unsupported / 测试用例 all dispatch methods unsupported
```python
    def test_all_dispatch_methods_unsupported(self):
        """All dispatch methods on base class return InvalidGrammarObject."""
        cases = [
            ("dispatch_json", ("schema",)),
            ("dispatch_regex", ("[a-z]+",)),
            ("dispatch_ebnf", ("root ::= 'hello'",)),
            ("dispatch_structural_tag", ("{}",)),
        ]
        for method_name, args in cases:
            with self.subTest(method=method_name):
                result = getattr(self.backend, method_name)(*args)
                self.assertIsInstance(result, InvalidGrammarObject)
```
**EN:** All dispatch methods on base class return InvalidGrammarObject. This test exercises `test_all_dispatch_methods_unsupported` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** All dispatch methods on base class return InvalidGrammarObject. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_dispatch_methods_unsupported`。

### Lines 149-151: test case dispatch fallback raises / 测试用例 dispatch fallback raises
```python
    def test_dispatch_fallback_raises(self):
        with self.assertRaises(ValueError):
            self.backend.dispatch_fallback("unknown", "value")
```
**EN:** This test exercises `test_dispatch_fallback_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dispatch_fallback_raises`。

### Lines 153-164: test case init value dispatch routes all types / 测试用例 init value dispatch routes all types
```python
    def test_init_value_dispatch_routes_all_types(self):
        """_init_value_dispatch routes all grammar types to their dispatch methods."""
        cases = [
            ("json", "schema"),
            ("regex", "[a-z]+"),
            ("ebnf", "root ::= 'x'"),
            ("structural_tag", "{}"),
        ]
        for grammar_type, value in cases:
            with self.subTest(grammar_type=grammar_type):
                result = self.backend._init_value_dispatch((grammar_type, value), False)
                self.assertIsInstance(result, InvalidGrammarObject)
```
**EN:** _init_value_dispatch routes all grammar types to their dispatch methods. This test exercises `test_init_value_dispatch_routes_all_types` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** _init_value_dispatch routes all grammar types to their dispatch methods. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_value_dispatch_routes_all_types`。

### Lines 166-168: test case init value dispatch unknown type raises / 测试用例 init value dispatch unknown type raises
```python
    def test_init_value_dispatch_unknown_type_raises(self):
        with self.assertRaises(ValueError):
            self.backend._init_value_dispatch(("unknown_type", "value"), False)
```
**EN:** This test exercises `test_init_value_dispatch_unknown_type_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_value_dispatch_unknown_type_raises`。

### Lines 170-178: test case init value dispatch sets compilation time / 测试用例 init value dispatch sets compilation time
```python
    def test_init_value_dispatch_sets_compilation_time(self):
        """When grammar has stats, compilation_time should be set."""
        mock_grammar = MagicMock(spec=BaseGrammarObject)
        mock_grammar.grammar_stats = GrammarStats()
        self.backend.dispatch_json = MagicMock(return_value=mock_grammar)

        result = self.backend._init_value_dispatch(("json", "schema"), False)
        self.assertIsNotNone(result.grammar_stats.compilation_time)
        self.assertGreater(result.grammar_stats.compilation_time, 0)
```
**EN:** When grammar has stats, compilation_time should be set. This test exercises `test_init_value_dispatch_sets_compilation_time` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When grammar has stats, compilation_time should be set. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_value_dispatch_sets_compilation_time`。

### Lines 180-186: test case init value dispatch no stats / 测试用例 init value dispatch no stats
```python
    def test_init_value_dispatch_no_stats(self):
        """When grammar has no stats, should not crash."""
        mock_grammar = MagicMock(spec=BaseGrammarObject)
        mock_grammar.grammar_stats = None
        self.backend.dispatch_json = MagicMock(return_value=mock_grammar)
        # Should not raise
        self.backend._init_value_dispatch(("json", "schema"), False)
```
**EN:** When grammar has no stats, should not crash. This test exercises `test_init_value_dispatch_no_stats` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When grammar has no stats, should not crash. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_value_dispatch_no_stats`。

### Lines 188-201: test case reset then miss / 测试用例 reset then miss
```python
    def test_reset_then_miss(self):
        """After reset, previously cached keys should be misses."""
        key = ("json", "schema")
        obj = MagicMock(spec=BaseGrammarObject)
        obj.copy.return_value = obj
        self.backend.set_cache(key, obj)

        _, hit = self.backend.get_cached_or_future_value(key, False)
        self.assertTrue(hit)

        self.backend.reset()
        result, hit = self.backend.get_cached_or_future_value(key, False)
        self.assertFalse(hit)
        self.assertIsInstance(result, Future)
```
**EN:** After reset, previously cached keys should be misses. This test exercises `test_reset_then_miss` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** After reset, previously cached keys should be misses. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reset_then_miss`。

### Lines 203-208: test case dispatch fallback error message content / 测试用例 dispatch fallback error message content
```python
    def test_dispatch_fallback_error_message_content(self):
        """dispatch_fallback error should include the key type and value."""
        with self.assertRaises(ValueError) as ctx:
            self.backend.dispatch_fallback("custom_type", "custom_value")
        self.assertIn("custom_type", str(ctx.exception))
        self.assertIn("custom_value", str(ctx.exception))
```
**EN:** dispatch_fallback error should include the key type and value. This test exercises `test_dispatch_fallback_error_message_content` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** dispatch_fallback error should include the key type and value. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dispatch_fallback_error_message_content`。

### Lines 210-214: test case init value dispatch none grammar / 测试用例 init value dispatch none grammar
```python
    def test_init_value_dispatch_none_grammar(self):
        """When dispatch returns None, should not crash on stats check."""
        self.backend.dispatch_json = MagicMock(return_value=None)
        result = self.backend._init_value_dispatch(("json", "schema"), False)
        self.assertIsNone(result)
```
**EN:** When dispatch returns None, should not crash on stats check. This test exercises `test_init_value_dispatch_none_grammar` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When dispatch returns None, should not crash on stats check. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_value_dispatch_none_grammar`。

### Lines 216-236: test case cache miss duplicate key submits separate futures / 测试用例 cache miss duplicate key submits separate futures
```python
    def test_cache_miss_duplicate_key_submits_separate_futures(self):
        """Two cache misses for the same key each get their own Future.

        The backend does not deduplicate in-flight compilations — that is
        handled at the GrammarManager level via grammar_queue. Each call
        to get_cached_or_future_value with an uncached key submits a new
        task to the executor."""
        key = ("json", "schema")
        result1, hit1 = self.backend.get_cached_or_future_value(key, False)
        result2, hit2 = self.backend.get_cached_or_future_value(key, False)

        self.assertFalse(hit1)
        self.assertFalse(hit2)
        self.assertIsInstance(result1, Future)
        self.assertIsInstance(result2, Future)
        # They are independent futures, not shared
        self.assertIsNot(result1, result2)

        # Both should complete successfully
        self.assertIsInstance(result1.result(timeout=5), InvalidGrammarObject)
        self.assertIsInstance(result2.result(timeout=5), InvalidGrammarObject)
```
**EN:** Two cache misses for the same key each get their own Future. This test exercises `test_cache_miss_duplicate_key_submits_separate_futures` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Two cache misses for the same key each get their own Future. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_cache_miss_duplicate_key_submits_separate_futures`。

### Lines 239-239: class TestRegisterGrammarBackend declaration / 类 TestRegisterGrammarBackend 声明
```python
class TestRegisterGrammarBackend(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 240-240: supporting statements / 辅助语句
```python
    """Test grammar backend registry."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 242-243: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self._saved = dict(GRAMMAR_BACKEND_REGISTRY)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 245-247: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        GRAMMAR_BACKEND_REGISTRY.clear()
        GRAMMAR_BACKEND_REGISTRY.update(self._saved)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 249-252: test case register and use / 测试用例 register and use
```python
    def test_register_and_use(self):
        mock_init = MagicMock(return_value="custom_backend")
        register_grammar_backend("my_backend", mock_init)
        self.assertIn("my_backend", GRAMMAR_BACKEND_REGISTRY)
```
**EN:** This test exercises `test_register_and_use` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_register_and_use`。

### Lines 254-259: test case overwrite registration / 测试用例 overwrite registration
```python
    def test_overwrite_registration(self):
        register_grammar_backend("dup", lambda *a: "first")
        register_grammar_backend("dup", lambda *a: "second")
        self.assertEqual(
            GRAMMAR_BACKEND_REGISTRY["dup"](None, None, None, None), "second"
        )
```
**EN:** This test exercises `test_overwrite_registration` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_overwrite_registration`。

### Lines 262-262: class TestCreateGrammarBackend declaration / 类 TestCreateGrammarBackend 声明
```python
class TestCreateGrammarBackend(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 263-263: supporting statements / 辅助语句
```python
    """Test create_grammar_backend factory function."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 265-266: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self._saved = dict(GRAMMAR_BACKEND_REGISTRY)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 268-270: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        GRAMMAR_BACKEND_REGISTRY.clear()
        GRAMMAR_BACKEND_REGISTRY.update(self._saved)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 272-281: method make server args / 方法 make server args
```python
    def _make_server_args(
        self, backend="none", reasoning_parser=None, enable_strict_thinking=False
    ):
        args = MagicMock()
        args.grammar_backend = backend
        args.reasoning_parser = reasoning_parser
        args.enable_strict_thinking = enable_strict_thinking
        args.constrained_json_whitespace_pattern = None
        args.constrained_json_disable_any_whitespace = False
        return args
```
**EN:** This block implements `_make_server_args` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_server_args`，承担模块行为中的一个聚焦逻辑片段。

### Lines 283-286: test case none backend returns none / 测试用例 none backend returns none
```python
    def test_none_backend_returns_none(self):
        args = self._make_server_args("none")
        result = create_grammar_backend(args, None, 32000)
        self.assertIsNone(result)
```
**EN:** This test exercises `test_none_backend_returns_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_none_backend_returns_none`。

### Lines 288-291: test case none backend with strict thinking raises / 测试用例 none backend with strict thinking raises
```python
    def test_none_backend_with_strict_thinking_raises(self):
        args = self._make_server_args("none", enable_strict_thinking=True)
        with self.assertRaisesRegex(ValueError, "enable-strict-thinking"):
            create_grammar_backend(args, None, 32000)
```
**EN:** This test exercises `test_none_backend_with_strict_thinking_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_none_backend_with_strict_thinking_raises`。

### Lines 293-296: test case invalid backend raises / 测试用例 invalid backend raises
```python
    def test_invalid_backend_raises(self):
        args = self._make_server_args("nonexistent_backend")
        with self.assertRaises(ValueError):
            create_grammar_backend(args, None, 32000)
```
**EN:** This test exercises `test_invalid_backend_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_invalid_backend_raises`。

### Lines 298-303: test case custom registered backend / 测试用例 custom registered backend
```python
    def test_custom_registered_backend(self):
        mock_backend = MagicMock()
        register_grammar_backend("test_custom", lambda *a: mock_backend)
        args = self._make_server_args("test_custom")
        result = create_grammar_backend(args, "tok", 32000, {1, 2})
        self.assertIs(result, mock_backend)
```
**EN:** This test exercises `test_custom_registered_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_custom_registered_backend`。

### Lines 305-320: test case custom backend receives args / 测试用例 custom backend receives args
```python
    def test_custom_backend_receives_args(self):
        received = {}

        def init_fn(server_args, tokenizer, vocab_size, eos_token_ids):
            received["server_args"] = server_args
            received["tokenizer"] = tokenizer
            received["vocab_size"] = vocab_size
            received["eos_token_ids"] = eos_token_ids
            return MagicMock()

        register_grammar_backend("capture", init_fn)
        args = self._make_server_args("capture")
        create_grammar_backend(args, "my_tok", 50000, {0, 2})
        self.assertEqual(received["tokenizer"], "my_tok")
        self.assertEqual(received["vocab_size"], 50000)
        self.assertEqual(received["eos_token_ids"], {0, 2})
```
**EN:** This test exercises `test_custom_backend_receives_args` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_custom_backend_receives_args`。

### Lines 322-332: test case custom backend skips reasoner wrapping / 测试用例 custom backend skips reasoner wrapping
```python
    def test_custom_backend_skips_reasoner_wrapping(self):
        """Custom registered backends return directly, bypassing reasoner wrapping."""
        mock_inner = MagicMock(spec=BaseGrammarBackend)
        register_grammar_backend("inner_r", lambda *a: mock_inner)

        args = self._make_server_args("inner_r", reasoning_parser="deepseek-r1")
        tokenizer = MagicMock()

        result = create_grammar_backend(args, tokenizer, 32000)
        # Custom backends return early, no reasoner wrapping applied
        self.assertIs(result, mock_inner)
```
**EN:** Custom registered backends return directly, bypassing reasoner wrapping. This test exercises `test_custom_backend_skips_reasoner_wrapping` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Custom registered backends return directly, bypassing reasoner wrapping. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_custom_backend_skips_reasoner_wrapping`。

### Lines 334-343: test case outlines backend / 测试用例 outlines backend
```python
    @patch("sglang.srt.constrained.outlines_backend.OutlinesGrammarBackend")
    def test_outlines_backend(self, mock_outlines_cls):
        mock_backend = MagicMock(spec=BaseGrammarBackend)
        mock_outlines_cls.return_value = mock_backend
        args = self._make_server_args("outlines")
        args.constrained_json_whitespace_pattern = r"\s*"

        result = create_grammar_backend(args, "tok", 32000)
        mock_outlines_cls.assert_called_once_with("tok", whitespace_pattern=r"\s*")
        self.assertIs(result, mock_backend)
```
**EN:** This test exercises `test_outlines_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_outlines_backend`。

### Lines 345-356: test case xgrammar backend / 测试用例 xgrammar backend
```python
    @patch("sglang.srt.constrained.xgrammar_backend.XGrammarGrammarBackend")
    def test_xgrammar_backend(self, mock_xgrammar_cls):
        mock_backend = MagicMock(spec=BaseGrammarBackend)
        mock_xgrammar_cls.return_value = mock_backend
        args = self._make_server_args("xgrammar")
        args.constrained_json_disable_any_whitespace = True

        result = create_grammar_backend(args, "tok", 32000, {1, 2})
        mock_xgrammar_cls.assert_called_once_with(
            "tok", vocab_size=32000, model_eos_token_ids=[1, 2], any_whitespace=False
        )
        self.assertIs(result, mock_backend)
```
**EN:** This test exercises `test_xgrammar_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_xgrammar_backend`。

### Lines 358-369: test case xgrammar unsupported tokenizer falls back to none / 测试用例 xgrammar unsupported tokenizer falls back to none
```python
    @patch("sglang.srt.constrained.xgrammar_backend.XGrammarGrammarBackend")
    def test_xgrammar_unsupported_tokenizer_falls_back_to_none(self, mock_xgrammar_cls):
        from sglang.srt.constrained.xgrammar_backend import TokenizerNotSupportedError

        mock_xgrammar_cls.side_effect = TokenizerNotSupportedError(
            "unsupported tokenizer"
        )
        args = self._make_server_args("xgrammar")

        result = create_grammar_backend(args, "tok", 32000, {1})
        self.assertIsNone(result)
        self.assertEqual(args.grammar_backend, "none")
```
**EN:** This test exercises `test_xgrammar_unsupported_tokenizer_falls_back_to_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_xgrammar_unsupported_tokenizer_falls_back_to_none`。

### Lines 371-383: test case llguidance backend / 测试用例 llguidance backend
```python
    @patch("sglang.srt.constrained.llguidance_backend.GuidanceBackend")
    def test_llguidance_backend(self, mock_guidance_cls):
        mock_backend = MagicMock(spec=BaseGrammarBackend)
        mock_guidance_cls.return_value = mock_backend
        args = self._make_server_args("llguidance")
        args.constrained_json_disable_any_whitespace = False
        args.constrained_json_whitespace_pattern = r"\s+"

        result = create_grammar_backend(args, "tok", 32000)
        mock_guidance_cls.assert_called_once_with(
            tokenizer="tok", any_whitespace=True, whitespace_pattern=r"\s+"
        )
        self.assertIs(result, mock_backend)
```
**EN:** This test exercises `test_llguidance_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_llguidance_backend`。

### Lines 385-402: test case reasoner wrapping on builtin backend / 测试用例 reasoner wrapping on builtin backend
```python
    @patch("sglang.srt.constrained.outlines_backend.OutlinesGrammarBackend")
    def test_reasoner_wrapping_on_builtin_backend(self, mock_outlines_cls):
        """Non-custom backends get wrapped with ReasonerGrammarBackend."""
        from sglang.srt.constrained.reasoner_grammar_backend import (
            ReasonerGrammarBackend,
        )

        mock_backend = MagicMock(spec=BaseGrammarBackend)
        mock_backend.is_support_token_filter = False
        mock_outlines_cls.return_value = mock_backend
        args = self._make_server_args("outlines", reasoning_parser="deepseek-r1")
        tokenizer = MagicMock()
        # encode must return a single-token list for think_start/end tokens
        tokenizer.encode.return_value = [42]

        result = create_grammar_backend(args, tokenizer, 32000, think_end_id=42)
        self.assertIsInstance(result, ReasonerGrammarBackend)
        self.assertIs(result.grammar_backend, mock_backend)
```
**EN:** Non-custom backends get wrapped with ReasonerGrammarBackend. This test exercises `test_reasoner_wrapping_on_builtin_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Non-custom backends get wrapped with ReasonerGrammarBackend. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reasoner_wrapping_on_builtin_backend`。

### Lines 404-413: test case no reasoner wrapping without think end id / 测试用例 no reasoner wrapping without think end id
```python
    @patch("sglang.srt.constrained.outlines_backend.OutlinesGrammarBackend")
    def test_no_reasoner_wrapping_without_think_end_id(self, mock_outlines_cls):
        """Without think_end_id passed in, no reasoner wrapping."""
        mock_backend = MagicMock(spec=BaseGrammarBackend)
        mock_outlines_cls.return_value = mock_backend
        args = self._make_server_args("outlines", reasoning_parser="deepseek-r1")
        tokenizer = MagicMock(spec=[])  # No think_end_id attribute

        result = create_grammar_backend(args, tokenizer, 32000, think_end_id=None)
        self.assertIs(result, mock_backend)
```
**EN:** Without think_end_id passed in, no reasoner wrapping. This test exercises `test_no_reasoner_wrapping_without_think_end_id` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Without think_end_id passed in, no reasoner wrapping. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_reasoner_wrapping_without_think_end_id`。

### Lines 415-424: test case no reasoner wrapping without reasoning parser / 测试用例 no reasoner wrapping without reasoning parser
```python
    @patch("sglang.srt.constrained.outlines_backend.OutlinesGrammarBackend")
    def test_no_reasoner_wrapping_without_reasoning_parser(self, mock_outlines_cls):
        """Without reasoning_parser, no reasoner wrapping even with think_end_id."""
        mock_backend = MagicMock(spec=BaseGrammarBackend)
        mock_outlines_cls.return_value = mock_backend
        args = self._make_server_args("outlines", reasoning_parser=None)
        tokenizer = MagicMock()

        result = create_grammar_backend(args, tokenizer, 32000, think_end_id=42)
        self.assertIs(result, mock_backend)
```
**EN:** Without reasoning_parser, no reasoner wrapping even with think_end_id. This test exercises `test_no_reasoner_wrapping_without_reasoning_parser` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Without reasoning_parser, no reasoner wrapping even with think_end_id. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_reasoner_wrapping_without_reasoning_parser`。

### Lines 426-434: test case xgrammar eos none / 测试用例 xgrammar eos none
```python
    @patch("sglang.srt.constrained.xgrammar_backend.XGrammarGrammarBackend")
    def test_xgrammar_eos_none(self, mock_xgrammar_cls):
        """eos_token_ids=None should pass None, not an empty list."""
        mock_xgrammar_cls.return_value = MagicMock(spec=BaseGrammarBackend)
        args = self._make_server_args("xgrammar")

        create_grammar_backend(args, "tok", 32000, None)
        _, kwargs = mock_xgrammar_cls.call_args
        self.assertIsNone(kwargs["model_eos_token_ids"])
```
**EN:** eos_token_ids=None should pass None, not an empty list. This test exercises `test_xgrammar_eos_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** eos_token_ids=None should pass None, not an empty list. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_xgrammar_eos_none`。

### Lines 437-438: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestGrammarStats`: Test GrammarStats dataclass. / 用于组织相关测试、夹具或辅助方法。
- `TestBaseGrammarObject`: Test BaseGrammarObject base class. / 用于组织相关测试、夹具或辅助方法。
- `TestInvalidGrammarObject`: Test InvalidGrammarObject. / 用于组织相关测试、夹具或辅助方法。
- `TestBaseGrammarBackend`: Test BaseGrammarBackend caching and dispatch. / 用于组织相关测试、夹具或辅助方法。
- `TestRegisterGrammarBackend`: Test grammar backend registry. / 用于组织相关测试、夹具或辅助方法。
- `TestCreateGrammarBackend`: Test create_grammar_backend factory function. / 用于组织相关测试、夹具或辅助方法。
- `TestGrammarStats.test_defaults`: This test exercises `test_defaults` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_defaults`。
- `TestGrammarStats.test_tree_traversal_time_mutable_default`: Ensure each instance gets its own list. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tree_traversal_time_mutable_default`。
- `TestBaseGrammarObject.test_is_terminated_default`: This test exercises `test_is_terminated_default` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_is_terminated_default`。
- `TestBaseGrammarObject.test_maybe_init_reasoning_noop`: This test exercises `test_maybe_init_reasoning_noop` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_maybe_init_reasoning_noop`。
- `TestInvalidGrammarObject.test_default_error_message`: This test exercises `test_default_error_message` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_error_message`。
- `TestInvalidGrammarObject.test_custom_error_message`: This test exercises `test_custom_error_message` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_custom_error_message`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `concurrent.futures`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.srt.constrained.base_grammar_backend`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 438
