# test_hook_registry.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/plugins/test_hook_registry.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates hook registry behavior in SGLang's unit / plugins area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 插件 领域中与 hook registry 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: supporting statements / 辅助语句
```python
"""
Unit tests for the hook registry system.

Covers: basic hooks (AROUND/BEFORE/AFTER/REPLACE), descriptor preservation
(classmethod/staticmethod), hook ordering, cross-target conflict detection,
patch propagation, and edge cases.

Run:  python -m pytest test/registered/unit/plugins/test_hook_registry.py -v
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 11-17: module imports and dependencies / 模块导入与依赖
```python
import sys
import types
import uuid

from sglang.srt.plugins.hook_registry import HookRegistry, HookType, plugin_hook
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `sys`, `types`, `uuid`, `sglang.srt.plugins.hook_registry`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sys`, `types`, `uuid`, `sglang.srt.plugins.hook_registry`。

### Lines 19-25: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=7, suite="base-a-test-cpu")

# ---------------------------------------------------------------------------
# Helpers: synthetic module creation
# ---------------------------------------------------------------------------

_SYNTH_MODULE_PREFIX = "_synth_hook_test_"
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 28-35: function make module / 函数 make module
```python
def _make_module(**attrs):
    """Create a throwaway module registered in sys.modules."""
    name = f"{_SYNTH_MODULE_PREFIX}{uuid.uuid4().hex[:8]}"
    mod = types.ModuleType(name)
    for k, v in attrs.items():
        setattr(mod, k, v)
    sys.modules[name] = mod
    return mod, name
```
**EN:** Create a throwaway module registered in sys.modules. This block implements `_make_module` and captures one focused piece of the module's behavior.
**CN:** Create a throwaway module registered in sys.modules. 该代码块实现 `_make_module`，承担模块行为中的一个聚焦逻辑片段。

### Lines 38-42: function cleanup synth modules / 函数 cleanup synth modules
```python
def _cleanup_synth_modules():
    """Remove all synthetic modules from sys.modules."""
    to_del = [k for k in sys.modules if k.startswith(_SYNTH_MODULE_PREFIX)]
    for k in to_del:
        del sys.modules[k]
```
**EN:** Remove all synthetic modules from sys.modules. This block implements `_cleanup_synth_modules` and captures one focused piece of the module's behavior.
**CN:** Remove all synthetic modules from sys.modules. 该代码块实现 `_cleanup_synth_modules`，承担模块行为中的一个聚焦逻辑片段。

### Lines 43-49: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# Base class for hook tests (shared setUp/tearDown)
# ---------------------------------------------------------------------------


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 50-50: class _HookTestCase declaration / 类 _HookTestCase 声明
```python
class _HookTestCase(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 51-51: supporting statements / 辅助语句
```python
    """Base class that resets HookRegistry and cleans up synth modules."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 53-55: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        HookRegistry.reset()
        _cleanup_synth_modules()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 57-59: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        HookRegistry.reset()
        _cleanup_synth_modules()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 60-66: supporting source context / 辅助源码上下文
```python


# ===========================================================================
# TestBasicHooks
# ===========================================================================


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 67-67: class TestBasicHooks declaration / 类 TestBasicHooks 声明
```python
class TestBasicHooks(_HookTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `_HookTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `_HookTestCase`。

### Lines 68-69: supporting statements / 辅助语句
```python
    """AROUND / BEFORE / AFTER / REPLACE on plain functions, class REPLACE,
    and the @plugin_hook decorator."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 71-82: test case around function / 测试用例 around function
```python
    def test_around_function(self):
        def orig(x):
            return x * 2

        mod, name = _make_module(orig=orig)

        def add_one(original_fn, x):
            return original_fn(x) + 1

        HookRegistry.register(f"{name}.orig", add_one, HookType.AROUND)
        HookRegistry.apply_hooks()
        self.assertEqual(mod.orig(3), 7)  # 3*2 + 1
```
**EN:** This test exercises `test_around_function` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_around_function`。

### Lines 84-97: test case before modifies args / 测试用例 before modifies args
```python
    def test_before_modifies_args(self):
        """BEFORE hook returns (args, kwargs) to modify arguments."""

        def orig(x, y=0):
            return x + y

        mod, name = _make_module(orig=orig)

        def double_x(x, y=0):
            return (x * 2,), {"y": y + 1}

        HookRegistry.register(f"{name}.orig", double_x, HookType.BEFORE)
        HookRegistry.apply_hooks()
        self.assertEqual(mod.orig(3), 7)  # x=3*2=6, y=0+1=1, 6+1=7
```
**EN:** BEFORE hook returns (args, kwargs) to modify arguments. This test exercises `test_before_modifies_args` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** BEFORE hook returns (args, kwargs) to modify arguments. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_before_modifies_args`。

### Lines 99-112: test case before returning none / 测试用例 before returning none
```python
    def test_before_returning_none(self):
        """BEFORE hook returning None leaves arguments unchanged."""

        def orig(x):
            return x * 2

        mod, name = _make_module(orig=orig)

        def before_noop(x):
            return None  # leave args unchanged

        HookRegistry.register(f"{name}.orig", before_noop, HookType.BEFORE)
        HookRegistry.apply_hooks()
        self.assertEqual(mod.orig(3), 6)  # args unchanged
```
**EN:** BEFORE hook returning None leaves arguments unchanged. This test exercises `test_before_returning_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** BEFORE hook returning None leaves arguments unchanged. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_before_returning_none`。

### Lines 114-125: test case after function / 测试用例 after function
```python
    def test_after_function(self):
        def orig(x):
            return x * 2

        mod, name = _make_module(orig=orig)

        def add_ten(result, x):
            return result + 10

        HookRegistry.register(f"{name}.orig", add_ten, HookType.AFTER)
        HookRegistry.apply_hooks()
        self.assertEqual(mod.orig(3), 16)  # 3*2 + 10
```
**EN:** This test exercises `test_after_function` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_after_function`。

### Lines 127-138: test case replace function / 测试用例 replace function
```python
    def test_replace_function(self):
        def orig(x):
            return x * 2

        mod, name = _make_module(orig=orig)

        def replacement(x):
            return x * 100

        HookRegistry.register(f"{name}.orig", replacement, HookType.REPLACE)
        HookRegistry.apply_hooks()
        self.assertEqual(mod.orig(3), 300)
```
**EN:** This test exercises `test_replace_function` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_replace_function`。

### Lines 140-156: test case class replace / 测试用例 class replace
```python
    def test_class_replace(self):
        class Original:
            def greet(self):
                return "original"

        mod, name = _make_module(Original=Original)

        class Replacement(Original):
            def greet(self):
                return "replaced"

        HookRegistry.register(f"{name}.Original", Replacement, HookType.REPLACE)
        HookRegistry.apply_hooks()

        self.assertIs(mod.Original, Replacement)
        self.assertIsInstance(mod.Original(), Replacement)
        self.assertEqual(mod.Original().greet(), "replaced")
```
**EN:** This test exercises `test_class_replace` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_class_replace`。

### Lines 158-169: test case plugin hook decorator / 测试用例 plugin hook decorator
```python
    def test_plugin_hook_decorator(self):
        def orig(x):
            return x

        mod, name = _make_module(orig=orig)

        @plugin_hook(f"{name}.orig", type=HookType.REPLACE)
        def my_replace(x):
            return x + 42

        HookRegistry.apply_hooks()
        self.assertEqual(mod.orig(0), 42)
```
**EN:** This test exercises `test_plugin_hook_decorator` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_plugin_hook_decorator`。

### Lines 170-176: supporting source context / 辅助源码上下文
```python


# ===========================================================================
# TestDescriptorPreservation  (Bug B regression tests)
# ===========================================================================


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 177-177: class TestDescriptorPreservation declaration / 类 TestDescriptorPreservation 声明
```python
class TestDescriptorPreservation(_HookTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `_HookTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `_HookTestCase`。

### Lines 178-178: supporting statements / 辅助语句
```python
    """Hooks on classmethod/staticmethod must preserve descriptor semantics."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 180-191: method make cls module / 方法 make cls module
```python
    def _make_cls_module(self):
        class MyClass:
            @classmethod
            def cm(cls, x):
                return ("cm", cls.__name__, x)

            @staticmethod
            def sm(x):
                return ("sm", x)

        mod, name = _make_module(MyClass=MyClass)
        return mod, name, MyClass
```
**EN:** This block implements `_make_cls_module` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_cls_module`，承担模块行为中的一个聚焦逻辑片段。

### Lines 193-203: test case around classmethod / 测试用例 around classmethod
```python
    def test_around_classmethod(self):
        mod, name, MyClass = self._make_cls_module()

        def add_tag(original_fn, cls, x):
            return original_fn(cls, x) + ("around",)

        HookRegistry.register(f"{name}.MyClass.cm", add_tag, HookType.AROUND)
        HookRegistry.apply_hooks()

        result = mod.MyClass.cm(1)
        self.assertEqual(result, ("cm", "MyClass", 1, "around"))
```
**EN:** This test exercises `test_around_classmethod` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_around_classmethod`。

### Lines 205-215: test case replace classmethod / 测试用例 replace classmethod
```python
    def test_replace_classmethod(self):
        mod, name, MyClass = self._make_cls_module()

        def new_cm(cls, x):
            return ("replaced_cm", cls.__name__, x)

        HookRegistry.register(f"{name}.MyClass.cm", new_cm, HookType.REPLACE)
        HookRegistry.apply_hooks()

        result = mod.MyClass.cm(1)
        self.assertEqual(result, ("replaced_cm", "MyClass", 1))
```
**EN:** This test exercises `test_replace_classmethod` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_replace_classmethod`。

### Lines 217-227: test case around staticmethod / 测试用例 around staticmethod
```python
    def test_around_staticmethod(self):
        mod, name, MyClass = self._make_cls_module()

        def wrap_sm(original_fn, x):
            return original_fn(x) + ("around",)

        HookRegistry.register(f"{name}.MyClass.sm", wrap_sm, HookType.AROUND)
        HookRegistry.apply_hooks()

        result = mod.MyClass.sm(1)
        self.assertEqual(result, ("sm", 1, "around"))
```
**EN:** This test exercises `test_around_staticmethod` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_around_staticmethod`。

### Lines 229-239: test case replace staticmethod / 测试用例 replace staticmethod
```python
    def test_replace_staticmethod(self):
        mod, name, MyClass = self._make_cls_module()

        def new_sm(x):
            return ("replaced_sm", x)

        HookRegistry.register(f"{name}.MyClass.sm", new_sm, HookType.REPLACE)
        HookRegistry.apply_hooks()

        result = mod.MyClass.sm(1)
        self.assertEqual(result, ("replaced_sm", 1))
```
**EN:** This test exercises `test_replace_staticmethod` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_replace_staticmethod`。

### Lines 241-254: test case classmethod subclass cls / 测试用例 classmethod subclass cls
```python
    def test_classmethod_subclass_cls(self):
        mod, name, MyClass = self._make_cls_module()

        def add_tag(original_fn, cls, x):
            return original_fn(cls, x) + ("around",)

        HookRegistry.register(f"{name}.MyClass.cm", add_tag, HookType.AROUND)
        HookRegistry.apply_hooks()

        class Sub(mod.MyClass):
            pass

        result = Sub.cm(1)
        self.assertEqual(result, ("cm", "Sub", 1, "around"))
```
**EN:** This test exercises `test_classmethod_subclass_cls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_classmethod_subclass_cls`。

### Lines 255-261: supporting source context / 辅助源码上下文
```python


# ===========================================================================
# TestHookOrdering
# ===========================================================================


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 262-262: class TestHookOrdering declaration / 类 TestHookOrdering 声明
```python
class TestHookOrdering(_HookTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `_HookTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `_HookTestCase`。

### Lines 263-263: supporting statements / 辅助语句
```python
    """Verify REPLACE is applied first, then other hooks wrap it."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 265-281: test case replace then around / 测试用例 replace then around
```python
    def test_replace_then_around(self):
        def orig(x):
            return x

        mod, name = _make_module(orig=orig)

        def repl(x):
            return x * 10

        def add_one(original_fn, x):
            return original_fn(x) + 1

        HookRegistry.register(f"{name}.orig", repl, HookType.REPLACE)
        HookRegistry.register(f"{name}.orig", add_one, HookType.AROUND)
        HookRegistry.apply_hooks()
        # REPLACE first: x*10, then AROUND: +1  => 31
        self.assertEqual(mod.orig(3), 31)
```
**EN:** This test exercises `test_replace_then_around` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_replace_then_around`。

### Lines 283-303: test case replace before after / 测试用例 replace before after
```python
    def test_replace_before_after(self):
        def orig(x):
            return x

        mod, name = _make_module(orig=orig)

        def repl(x):
            return x * 10

        def double_arg(x):
            return (x * 2,), {}

        def add_hundred(result, x):
            return result + 100

        HookRegistry.register(f"{name}.orig", repl, HookType.REPLACE)
        HookRegistry.register(f"{name}.orig", double_arg, HookType.BEFORE)
        HookRegistry.register(f"{name}.orig", add_hundred, HookType.AFTER)
        HookRegistry.apply_hooks()
        # BEFORE doubles x: 3*2=6 → REPLACE: 6*10=60 → AFTER: 60+100=160
        self.assertEqual(mod.orig(3), 160)
```
**EN:** This test exercises `test_replace_before_after` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_replace_before_after`。

### Lines 304-310: supporting source context / 辅助源码上下文
```python


# ===========================================================================
# TestCrossTargetConflict
# ===========================================================================


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 311-311: class TestCrossTargetConflict declaration / 类 TestCrossTargetConflict 声明
```python
class TestCrossTargetConflict(_HookTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `_HookTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `_HookTestCase`。

### Lines 312-312: supporting statements / 辅助语句
```python
    """Verify warning for class REPLACE + method REPLACE combo."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 314-335: test case class replace then method replace warns / 测试用例 class replace then method replace warns
```python
    def test_class_replace_then_method_replace_warns(self):
        class Original:
            def foo(self):
                return "orig"

        mod, name = _make_module(Original=Original)

        class Replacement(Original):
            def foo(self):
                return "class_replaced"

        HookRegistry.register(f"{name}.Original", Replacement, HookType.REPLACE)

        def method_repl(self):
            return "method_replaced"

        HookRegistry.register(f"{name}.Original.foo", method_repl, HookType.REPLACE)

        with self.assertLogs("sglang.srt.plugins.hook_registry", level="WARNING") as cm:
            HookRegistry.apply_hooks()

        self.assertTrue(any("will override" in msg for msg in cm.output))
```
**EN:** This test exercises `test_class_replace_then_method_replace_warns` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_class_replace_then_method_replace_warns`。

### Lines 336-342: supporting source context / 辅助源码上下文
```python


# ===========================================================================
# TestPatchPropagation
# ===========================================================================


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 343-343: class TestPatchPropagation declaration / 类 TestPatchPropagation 声明
```python
class TestPatchPropagation(_HookTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `_HookTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `_HookTestCase`。

### Lines 344-344: supporting statements / 辅助语句
```python
    """Verify that patches propagate to other modules that imported the target."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 346-360: test case same reference propagates / 测试用例 same reference propagates
```python
    def test_same_reference_propagates(self):
        def orig(x):
            return x * 2

        source_mod, source_name = _make_module(orig=orig)
        importer_mod, _ = _make_module(orig=orig)  # same reference

        def add_one(fn, x):
            return fn(x) + 1

        HookRegistry.register(f"{source_name}.orig", add_one, HookType.AROUND)
        HookRegistry.apply_hooks()

        self.assertEqual(source_mod.orig(3), 7)
        self.assertEqual(importer_mod.orig(3), 7)
```
**EN:** This test exercises `test_same_reference_propagates` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_same_reference_propagates`。

### Lines 361-367: supporting source context / 辅助源码上下文
```python


# ===========================================================================
# TestEdgeCases
# ===========================================================================


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 368-368: class TestEdgeCases declaration / 类 TestEdgeCases 声明
```python
class TestEdgeCases(_HookTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `_HookTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `_HookTestCase`。

### Lines 369-369: supporting statements / 辅助语句
```python
    """Reset, type validation, multi-AROUND onion, idempotent apply."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 371-384: test case reset / 测试用例 reset
```python
    def test_reset(self):
        def orig(x):
            return x

        mod, name = _make_module(orig=orig)

        def noop(fn, x):
            return fn(x)

        HookRegistry.register(f"{name}.orig", noop, HookType.AROUND)
        HookRegistry.reset()

        HookRegistry.apply_hooks()
        self.assertEqual(mod.orig(3), 3)
```
**EN:** This test exercises `test_reset` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reset`。

### Lines 386-392: test case register class with wrong type / 测试用例 register class with wrong type
```python
    def test_register_class_with_wrong_type(self):
        class BadHook:
            pass

        for ht in (HookType.BEFORE, HookType.AFTER, HookType.AROUND):
            with self.assertRaises(TypeError):
                HookRegistry.register("some.target", BadHook, ht)
```
**EN:** This test exercises `test_register_class_with_wrong_type` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_register_class_with_wrong_type`。

### Lines 394-423: test case multi around onion / 测试用例 multi around onion
```python
    def test_multi_around_onion(self):
        call_order = []

        def orig(x):
            call_order.append("orig")
            return x

        mod, name = _make_module(orig=orig)

        def around1(fn, x):
            call_order.append("a1_before")
            result = fn(x)
            call_order.append("a1_after")
            return result + 1

        def around2(fn, x):
            call_order.append("a2_before")
            result = fn(x)
            call_order.append("a2_after")
            return result + 10

        HookRegistry.register(f"{name}.orig", around1, HookType.AROUND)
        HookRegistry.register(f"{name}.orig", around2, HookType.AROUND)
        HookRegistry.apply_hooks()

        result = mod.orig(0)
        self.assertEqual(result, 11)
        self.assertEqual(
            call_order, ["a2_before", "a1_before", "orig", "a1_after", "a2_after"]
        )
```
**EN:** This test exercises `test_multi_around_onion` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multi_around_onion`。

### Lines 425-442: test case apply idempotent / 测试用例 apply idempotent
```python
    def test_apply_idempotent(self):
        call_count = [0]

        def orig(x):
            return x

        mod, name = _make_module(orig=orig)

        def counter(fn, x):
            call_count[0] += 1
            return fn(x)

        HookRegistry.register(f"{name}.orig", counter, HookType.AROUND)
        HookRegistry.apply_hooks()
        HookRegistry.apply_hooks()  # second apply should be no-op

        mod.orig(1)
        self.assertEqual(call_count[0], 1)
```
**EN:** This test exercises `test_apply_idempotent` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_apply_idempotent`。

### Lines 445-448: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    import unittest

    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_make_module`: Create a throwaway module registered in sys.modules. / 该代码块实现 `_make_module`，承担模块行为中的一个聚焦逻辑片段。
- `_cleanup_synth_modules`: Remove all synthetic modules from sys.modules. / 该代码块实现 `_cleanup_synth_modules`，承担模块行为中的一个聚焦逻辑片段。
- `_HookTestCase`: Base class that resets HookRegistry and cleans up synth modules. / 用于组织相关测试、夹具或辅助方法。
- `TestBasicHooks`: AROUND / BEFORE / AFTER / REPLACE on plain functions, class REPLACE, and the @plugin_hook decorator. / 用于组织相关测试、夹具或辅助方法。
- `TestDescriptorPreservation`: Hooks on classmethod/staticmethod must preserve descriptor semantics. / 用于组织相关测试、夹具或辅助方法。
- `TestHookOrdering`: Verify REPLACE is applied first, then other hooks wrap it. / 用于组织相关测试、夹具或辅助方法。
- `TestCrossTargetConflict`: Verify warning for class REPLACE + method REPLACE combo. / 用于组织相关测试、夹具或辅助方法。
- `TestPatchPropagation`: Verify that patches propagate to other modules that imported the target. / 用于组织相关测试、夹具或辅助方法。
- `_HookTestCase.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `_HookTestCase.tearDown`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestBasicHooks.test_around_function`: This test exercises `test_around_function` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_around_function`。
- `TestBasicHooks.test_before_modifies_args`: BEFORE hook returns (args, kwargs) to modify arguments. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_before_modifies_args`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `sys`, `types`, `uuid`
- **Internal modules / 内部模块**: `sglang.srt.plugins.hook_registry`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 448
