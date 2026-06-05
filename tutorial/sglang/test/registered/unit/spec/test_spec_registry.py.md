# test_spec_registry.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/spec/test_spec_registry.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates spec registry behavior in SGLang's unit / spec area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 推测解码 领域中与 spec registry 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for the speculative algorithm plugin registry."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-13: module imports and dependencies / 模块导入与依赖
```python
import unittest
from unittest.mock import MagicMock

from sglang.srt.speculative.spec_info import SpeculativeAlgorithm
from sglang.srt.speculative.spec_registry import (
    _REGISTRY,
    _RESERVED_NAMES,
    CustomSpecAlgo,
)
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `unittest.mock`, `sglang.srt.speculative.spec_info`, `sglang.srt.speculative.spec_registry`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `unittest.mock`, `sglang.srt.speculative.spec_info`, `sglang.srt.speculative.spec_registry`。

### Lines 15-15: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=5, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 18-18: class _RegistryIsolated declaration / 类 _RegistryIsolated 声明
```python
class _RegistryIsolated(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 19-19: supporting statements / 辅助语句
```python
    """Snapshot and restore the global registry so tests don't leak."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 21-23: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self._snapshot = _REGISTRY.copy()
        _REGISTRY.clear()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 25-27: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        _REGISTRY.clear()
        _REGISTRY.update(self._snapshot)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 30-30: class TestFromString declaration / 类 TestFromString 声明
```python
class TestFromString(_RegistryIsolated):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `_RegistryIsolated`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `_RegistryIsolated`。

### Lines 31-32: test case none input returns none member / 测试用例 none input returns none member
```python
    def test_none_input_returns_none_member(self):
        self.assertIs(SpeculativeAlgorithm.from_string(None), SpeculativeAlgorithm.NONE)
```
**EN:** This test exercises `test_none_input_returns_none_member` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_none_input_returns_none_member`。

### Lines 34-40: test case builtin name returns enum / 测试用例 builtin name returns enum
```python
    def test_builtin_name_returns_enum(self):
        self.assertIs(
            SpeculativeAlgorithm.from_string("EAGLE"), SpeculativeAlgorithm.EAGLE
        )
        self.assertIs(
            SpeculativeAlgorithm.from_string("NGRAM"), SpeculativeAlgorithm.NGRAM
        )
```
**EN:** This test exercises `test_builtin_name_returns_enum` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_builtin_name_returns_enum`。

### Lines 42-45: test case builtin name is case insensitive / 测试用例 builtin name is case insensitive
```python
    def test_builtin_name_is_case_insensitive(self):
        self.assertIs(
            SpeculativeAlgorithm.from_string("eagle"), SpeculativeAlgorithm.EAGLE
        )
```
**EN:** This test exercises `test_builtin_name_is_case_insensitive` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_builtin_name_is_case_insensitive`。

### Lines 47-49: test case unknown name raises / 测试用例 unknown name raises
```python
    def test_unknown_name_raises(self):
        with self.assertRaisesRegex(ValueError, "Unknown speculative algorithm"):
            SpeculativeAlgorithm.from_string("NOT_REGISTERED")
```
**EN:** This test exercises `test_unknown_name_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unknown_name_raises`。

### Lines 51-58: test case registered plugin returns custom spec / 测试用例 registered plugin returns custom spec
```python
    def test_registered_plugin_returns_custom_spec(self):
        @SpeculativeAlgorithm.register("MY_FOO")
        def _factory(server_args):
            return MagicMock

        algo = SpeculativeAlgorithm.from_string("MY_FOO")
        self.assertIsInstance(algo, CustomSpecAlgo)
        self.assertEqual(algo.name, "MY_FOO")
```
**EN:** This test exercises `test_registered_plugin_returns_custom_spec` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_registered_plugin_returns_custom_spec`。

### Lines 60-68: test case registered plugin lookup is case insensitive / 测试用例 registered plugin lookup is case insensitive
```python
    def test_registered_plugin_lookup_is_case_insensitive(self):
        @SpeculativeAlgorithm.register("MY_FOO")
        def _factory(server_args):
            return MagicMock

        self.assertIs(
            SpeculativeAlgorithm.from_string("my_foo"),
            SpeculativeAlgorithm.from_string("MY_FOO"),
        )
```
**EN:** This test exercises `test_registered_plugin_lookup_is_case_insensitive` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_registered_plugin_lookup_is_case_insensitive`。

### Lines 71-71: class TestRegister declaration / 类 TestRegister 声明
```python
class TestRegister(_RegistryIsolated):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `_RegistryIsolated`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `_RegistryIsolated`。

### Lines 72-77: test case register returns factory unchanged / 测试用例 register returns factory unchanged
```python
    def test_register_returns_factory_unchanged(self):
        def _factory(server_args):
            return MagicMock

        decorated = SpeculativeAlgorithm.register("MY_FOO")(_factory)
        self.assertIs(decorated, _factory)
```
**EN:** This test exercises `test_register_returns_factory_unchanged` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_register_returns_factory_unchanged`。

### Lines 79-93: test case two distinct registrations are independent / 测试用例 two distinct registrations are independent
```python
    def test_two_distinct_registrations_are_independent(self):
        @SpeculativeAlgorithm.register("FOO")
        def _foo_factory(server_args):
            return MagicMock

        @SpeculativeAlgorithm.register("BAR")
        def _bar_factory(server_args):
            return MagicMock

        foo = SpeculativeAlgorithm.from_string("FOO")
        bar = SpeculativeAlgorithm.from_string("BAR")
        self.assertIsNot(foo, bar)
        self.assertNotEqual(foo, bar)
        self.assertEqual(foo.name, "FOO")
        self.assertEqual(bar.name, "BAR")
```
**EN:** This test exercises `test_two_distinct_registrations_are_independent` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_two_distinct_registrations_are_independent`。

### Lines 95-104: test case duplicate name raises / 测试用例 duplicate name raises
```python
    def test_duplicate_name_raises(self):
        @SpeculativeAlgorithm.register("MY_FOO")
        def _factory(server_args):
            return MagicMock

        with self.assertRaisesRegex(ValueError, "already registered"):

            @SpeculativeAlgorithm.register("MY_FOO")
            def _factory2(server_args):
                return MagicMock
```
**EN:** This test exercises `test_duplicate_name_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_duplicate_name_raises`。

### Lines 106-109: test case reserved name raises / 测试用例 reserved name raises
```python
    def test_reserved_name_raises(self):
        for reserved in _RESERVED_NAMES:
            with self.assertRaisesRegex(ValueError, "reserved"):
                SpeculativeAlgorithm.register(reserved)
```
**EN:** This test exercises `test_reserved_name_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reserved_name_raises`。

### Lines 111-120: test case register is case insensitive on collision / 测试用例 register is case insensitive on collision
```python
    def test_register_is_case_insensitive_on_collision(self):
        @SpeculativeAlgorithm.register("MY_FOO")
        def _factory(server_args):
            return MagicMock

        with self.assertRaisesRegex(ValueError, "already registered"):

            @SpeculativeAlgorithm.register("my_foo")
            def _factory2(server_args):
                return MagicMock
```
**EN:** This test exercises `test_register_is_case_insensitive_on_collision` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_register_is_case_insensitive_on_collision`。

### Lines 123-123: class TestCustomSpecAlgoInterface declaration / 类 TestCustomSpecAlgoInterface 声明
```python
class TestCustomSpecAlgoInterface(_RegistryIsolated):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `_RegistryIsolated`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `_RegistryIsolated`。

### Lines 124-124: supporting statements / 辅助语句
```python
    """CustomSpecAlgo must duck-type SpeculativeAlgorithm enum values."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 126-133: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        super().setUp()

        @SpeculativeAlgorithm.register("MY_FOO", supports_overlap=False)
        def _factory(server_args):
            return MagicMock

        self.algo = SpeculativeAlgorithm.from_string("MY_FOO")
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 135-142: test case is predicates all false except speculative / 测试用例 is predicates all false except speculative
```python
    def test_is_predicates_all_false_except_speculative(self):
        self.assertFalse(self.algo.is_none())
        self.assertFalse(self.algo.is_eagle())
        self.assertFalse(self.algo.is_eagle3())
        self.assertFalse(self.algo.is_dflash())
        self.assertFalse(self.algo.is_standalone())
        self.assertFalse(self.algo.is_ngram())
        self.assertTrue(self.algo.is_speculative())
```
**EN:** This test exercises `test_is_predicates_all_false_except_speculative` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_is_predicates_all_false_except_speculative`。

### Lines 144-153: test case supports spec v2 follows supports overlap / 测试用例 supports spec v2 follows supports overlap
```python
    def test_supports_spec_v2_follows_supports_overlap(self):
        # Plugin registered with supports_overlap=False -> not spec_v2.
        self.assertFalse(self.algo.supports_spec_v2())

        @SpeculativeAlgorithm.register("MY_V2", supports_overlap=True)
        def _factory(server_args):
            return MagicMock

        v2 = SpeculativeAlgorithm.from_string("MY_V2")
        self.assertTrue(v2.supports_spec_v2())
```
**EN:** This test exercises `test_supports_spec_v2_follows_supports_overlap` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_supports_spec_v2_follows_supports_overlap`。

### Lines 155-159: test case create worker calls factory / 测试用例 create worker calls factory
```python
    def test_create_worker_calls_factory(self):
        server_args = MagicMock()
        server_args.disable_overlap_schedule = True
        worker_cls = self.algo.create_worker(server_args)
        self.assertIs(worker_cls, MagicMock)
```
**EN:** This test exercises `test_create_worker_calls_factory` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_create_worker_calls_factory`。

### Lines 161-165: test case create worker raises on overlap mismatch / 测试用例 create worker raises on overlap mismatch
```python
    def test_create_worker_raises_on_overlap_mismatch(self):
        server_args = MagicMock()
        server_args.disable_overlap_schedule = False
        with self.assertRaisesRegex(ValueError, "does not support overlap"):
            self.algo.create_worker(server_args)
```
**EN:** This test exercises `test_create_worker_raises_on_overlap_mismatch` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_create_worker_raises_on_overlap_mismatch`。

### Lines 168-168: class TestValidatorHook declaration / 类 TestValidatorHook 声明
```python
class TestValidatorHook(_RegistryIsolated):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `_RegistryIsolated`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `_RegistryIsolated`。

### Lines 169-180: test case validator invocation is caller driven / 测试用例 validator invocation is caller driven
```python
    def test_validator_invocation_is_caller_driven(self):
        validator = MagicMock()

        @SpeculativeAlgorithm.register("MY_FOO", validate_server_args=validator)
        def _factory(server_args):
            return MagicMock

        algo = SpeculativeAlgorithm.from_string("MY_FOO")
        self.assertIs(algo.validate_server_args, validator)
        # Callers (e.g. ServerArgs.__post_init__) must invoke the hook themselves;
        # CustomSpecAlgo does not call it from create_worker.
        validator.assert_not_called()
```
**EN:** This test exercises `test_validator_invocation_is_caller_driven` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_validator_invocation_is_caller_driven`。

### Lines 183-183: class TestSubclassOverride declaration / 类 TestSubclassOverride 声明
```python
class TestSubclassOverride(_RegistryIsolated):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `_RegistryIsolated`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `_RegistryIsolated`。

### Lines 184-184: supporting statements / 辅助语句
```python
    """Plugins can subclass CustomSpecAlgo to override is_*() / create_worker."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 186-203: test case subclass overrides is eagle / 测试用例 subclass overrides is eagle
```python
    def test_subclass_overrides_is_eagle(self):
        class EagleLike(CustomSpecAlgo):
            def is_eagle(self) -> bool:
                return True

        @SpeculativeAlgorithm.register(
            "MY_LIKE_EAGLE", supports_overlap=True, spec_class=EagleLike
        )
        def _factory(server_args):
            return MagicMock

        algo = SpeculativeAlgorithm.from_string("MY_LIKE_EAGLE")
        self.assertIsInstance(algo, EagleLike)
        self.assertIsInstance(algo, CustomSpecAlgo)
        self.assertTrue(algo.is_eagle())
        # Other predicates default to False
        self.assertFalse(algo.is_ngram())
        self.assertFalse(algo.is_dflash())
```
**EN:** This test exercises `test_subclass_overrides_is_eagle` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_subclass_overrides_is_eagle`。

### Lines 205-216: test case subclass overrides create worker / 测试用例 subclass overrides create worker
```python
    def test_subclass_overrides_create_worker(self):
        class CustomDispatch(CustomSpecAlgo):
            def create_worker(self, server_args):
                return "custom-dispatched"

        @SpeculativeAlgorithm.register("MY_CUSTOM", spec_class=CustomDispatch)
        def _factory(server_args):
            return MagicMock

        algo = SpeculativeAlgorithm.from_string("MY_CUSTOM")
        # Custom dispatch bypasses default overlap check
        self.assertEqual(algo.create_worker(MagicMock()), "custom-dispatched")
```
**EN:** This test exercises `test_subclass_overrides_create_worker` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_subclass_overrides_create_worker`。

### Lines 219-219: class TestCrossTypeIdentity declaration / 类 TestCrossTypeIdentity 声明
```python
class TestCrossTypeIdentity(_RegistryIsolated):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `_RegistryIsolated`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `_RegistryIsolated`。

### Lines 220-220: supporting statements / 辅助语句
```python
    """A plugin algo and a builtin enum value must never compare equal."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 222-230: test case plugin not equal to builtin / 测试用例 plugin not equal to builtin
```python
    def test_plugin_not_equal_to_builtin(self):
        @SpeculativeAlgorithm.register("MY_FOO")
        def _factory(server_args):
            return MagicMock

        algo = SpeculativeAlgorithm.from_string("MY_FOO")
        self.assertNotEqual(algo, SpeculativeAlgorithm.EAGLE)
        self.assertNotEqual(algo, SpeculativeAlgorithm.NONE)
        self.assertIsNot(algo, SpeculativeAlgorithm.EAGLE)
```
**EN:** This test exercises `test_plugin_not_equal_to_builtin` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_plugin_not_equal_to_builtin`。

### Lines 233-234: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=3)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_RegistryIsolated`: Snapshot and restore the global registry so tests don't leak. / 用于组织相关测试、夹具或辅助方法。
- `TestFromString`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestRegister`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestCustomSpecAlgoInterface`: CustomSpecAlgo must duck-type SpeculativeAlgorithm enum values. / 用于组织相关测试、夹具或辅助方法。
- `TestValidatorHook`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestSubclassOverride`: Plugins can subclass CustomSpecAlgo to override is_*() / create_worker. / 用于组织相关测试、夹具或辅助方法。
- `TestCrossTypeIdentity`: A plugin algo and a builtin enum value must never compare equal. / 用于组织相关测试、夹具或辅助方法。
- `_RegistryIsolated.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `_RegistryIsolated.tearDown`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestFromString.test_none_input_returns_none_member`: This test exercises `test_none_input_returns_none_member` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_none_input_returns_none_member`。
- `TestFromString.test_builtin_name_returns_enum`: This test exercises `test_builtin_name_returns_enum` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_builtin_name_returns_enum`。
- `TestFromString.test_builtin_name_is_case_insensitive`: This test exercises `test_builtin_name_is_case_insensitive` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_builtin_name_is_case_insensitive`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.srt.speculative.spec_info`, `sglang.srt.speculative.spec_registry`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 234
