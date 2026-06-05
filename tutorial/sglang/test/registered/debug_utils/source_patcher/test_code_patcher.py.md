# test_code_patcher.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/source_patcher/test_code_patcher.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on source patcher code patcher in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 source patcher code patcher 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Import dependencies
```python
from types import ModuleType

import pytest

from sglang.srt.debug_utils.source_patcher.code_patcher import (
    CodePatcher,
    _resolve_target,
    patch_function,
)
from sglang.srt.debug_utils.source_patcher.types import EditSpec, PatchSpec
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 13-13: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 15-15: Define module constants
```python
SAMPLE_MODULE_NAME = "_source_patcher_test_fixtures.sample_module"
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 18-18: Define class TestPatchFunction
```python
class TestPatchFunction:
```
**EN:** This declaration introduces the `TestPatchFunction` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestPatchFunction` 测试类，并说明它通过继承承担的职责。

### Lines 19-38: Run test: basic patch changes behavior
```python
    def test_basic_patch_changes_behavior(self, sample_module: ModuleType) -> None:
        cls = sample_module.SampleClass
        obj = cls()
        assert obj.greet("world") == "hello world"

        state = patch_function(
            target=cls.greet,
            edits=[
                EditSpec(
                    match='greeting = f"hello {name}"',
                    replacement='greeting = f"patched {name}"',
                )
            ],
        )
        try:
            assert obj.greet("world") == "patched world"
        finally:
            state.restore()

        assert obj.greet("world") == "hello world"
```
**EN:** This test method exercises basic patch changes behavior and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 basic patch changes behavior 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 40-57: Run test: globals preserved after patch
```python
    def test_globals_preserved_after_patch(self, sample_module: ModuleType) -> None:
        cls = sample_module.SampleClass
        obj = cls()
        assert obj.uses_global() == "value=global_value"

        state = patch_function(
            target=cls.uses_global,
            edits=[
                EditSpec(
                    match='return f"value={GLOBAL_VAR}"',
                    replacement='return f"patched_value={GLOBAL_VAR}"',
                )
            ],
        )
        try:
            assert obj.uses_global() == "patched_value=global_value"
        finally:
            state.restore()
```
**EN:** This test method exercises globals preserved after patch and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 globals preserved after patch 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 59-75: Run test: function identity preserved
```python
    def test_function_identity_preserved(self, sample_module: ModuleType) -> None:
        cls = sample_module.SampleClass
        fn_id_before = id(cls.greet)

        state = patch_function(
            target=cls.greet,
            edits=[
                EditSpec(
                    match='greeting = f"hello {name}"',
                    replacement='greeting = f"patched {name}"',
                )
            ],
        )
        try:
            assert id(cls.greet) == fn_id_before
        finally:
            state.restore()
```
**EN:** This test method exercises function identity preserved and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 function identity preserved 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 77-95: Run test: patch standalone function
```python
    def test_patch_standalone_function(self, sample_module: ModuleType) -> None:
        fn = sample_module.standalone_function
        assert fn(2, 3) == 5

        state = patch_function(
            target=fn,
            edits=[
                EditSpec(
                    match="return a + b",
                    replacement="return a * b",
                )
            ],
        )
        try:
            assert fn(2, 3) == 6
        finally:
            state.restore()

        assert fn(2, 3) == 5
```
**EN:** This test method exercises patch standalone function and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 patch standalone function 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 97-116: Run test: patched code can reference global variable
```python
    def test_patched_code_can_reference_global_variable(
        self, sample_module: ModuleType
    ) -> None:
        """Replacement code that references a module-level global should work."""
        cls = sample_module.SampleClass
        obj = cls()

        state = patch_function(
            target=cls.greet,
            edits=[
                EditSpec(
                    match='greeting = f"hello {name}"',
                    replacement='greeting = f"{GLOBAL_VAR} {name}"',
                )
            ],
        )
        try:
            assert obj.greet("world") == "global_value world"
        finally:
            state.restore()
```
**EN:** This test method exercises patched code can reference global variable and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 patched code can reference global variable 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 118-137: Run test: patched code can call another class method
```python
    def test_patched_code_can_call_another_class_method(
        self, sample_module: ModuleType
    ) -> None:
        """Replacement code that calls HelperClass.format_value should work."""
        cls = sample_module.SampleClass
        obj = cls()

        state = patch_function(
            target=cls.greet,
            edits=[
                EditSpec(
                    match='greeting = f"hello {name}"',
                    replacement="greeting = HelperClass.format_value(name)",
                )
            ],
        )
        try:
            assert obj.greet("world") == "[world]"
        finally:
            state.restore()
```
**EN:** This test method exercises patched code can call another class method and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 patched code can call another class method 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 139-161: Run test: patched code uses helper via existing method
```python
    def test_patched_code_uses_helper_via_existing_method(
        self, sample_module: ModuleType
    ) -> None:
        """The uses_helper method already calls HelperClass; verify it survives patching."""
        cls = sample_module.SampleClass
        obj = cls()
        assert obj.uses_helper("test") == "[test]"

        state = patch_function(
            target=cls.uses_helper,
            edits=[
                EditSpec(
                    match="return HelperClass.format_value(value)",
                    replacement='return HelperClass.format_value("patched_" + value)',
                )
            ],
        )
        try:
            assert obj.uses_helper("test") == "[patched_test]"
        finally:
            state.restore()

        assert obj.uses_helper("test") == "[test]"
```
**EN:** This test method exercises patched code uses helper via existing method and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 patched code uses helper via existing method 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 164-164: Define class TestResolveTarget
```python
class TestResolveTarget:
```
**EN:** This declaration introduces the `TestResolveTarget` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestResolveTarget` 测试类，并说明它通过继承承担的职责。

### Lines 165-167: Run test: resolve class method
```python
    def test_resolve_class_method(self, sample_module: ModuleType) -> None:
        target = _resolve_target(f"{SAMPLE_MODULE_NAME}.SampleClass.greet")
        assert target is sample_module.SampleClass.greet
```
**EN:** This test method exercises resolve class method and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 resolve class method 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 169-171: Run test: resolve standalone function
```python
    def test_resolve_standalone_function(self, sample_module: ModuleType) -> None:
        target = _resolve_target(f"{SAMPLE_MODULE_NAME}.standalone_function")
        assert target is sample_module.standalone_function
```
**EN:** This test method exercises resolve standalone function and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 resolve standalone function 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 173-175: Run test: resolve nonexistent raises
```python
    def test_resolve_nonexistent_raises(self, sample_module: ModuleType) -> None:
        with pytest.raises((ImportError, AttributeError)):
            _resolve_target(f"{SAMPLE_MODULE_NAME}.NonexistentClass.method")
```
**EN:** This test method exercises resolve nonexistent raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 resolve nonexistent raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 178-178: Define class TestCodePatcher
```python
class TestCodePatcher:
```
**EN:** This declaration introduces the `TestCodePatcher` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestCodePatcher` 测试类，并说明它通过继承承担的职责。

### Lines 179-201: Run test: context manager patches and restores
```python
    def test_context_manager_patches_and_restores(
        self, sample_module: ModuleType
    ) -> None:
        cls = sample_module.SampleClass
        obj = cls()
        assert obj.greet("world") == "hello world"

        patches = [
            PatchSpec(
                target=f"{SAMPLE_MODULE_NAME}.SampleClass.greet",
                edits=[
                    EditSpec(
                        match='greeting = f"hello {name}"',
                        replacement='greeting = f"ctx_patched {name}"',
                    )
                ],
            )
        ]

        with CodePatcher(patches=patches):
            assert obj.greet("world") == "ctx_patched world"

        assert obj.greet("world") == "hello world"
```
**EN:** This test method exercises context manager patches and restores and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 context manager patches and restores 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 203-233: Run test: context manager multiple patches
```python
    def test_context_manager_multiple_patches(self, sample_module: ModuleType) -> None:
        cls = sample_module.SampleClass
        obj = cls()

        patches = [
            PatchSpec(
                target=f"{SAMPLE_MODULE_NAME}.SampleClass.greet",
                edits=[
                    EditSpec(
                        match='greeting = f"hello {name}"',
                        replacement='greeting = f"p1 {name}"',
                    )
                ],
            ),
            PatchSpec(
                target=f"{SAMPLE_MODULE_NAME}.SampleClass.compute",
                edits=[
                    EditSpec(
                        match="result = x * 2 + 1",
                        replacement="result = x * 100",
                    )
                ],
            ),
        ]

        with CodePatcher(patches=patches):
            assert obj.greet("world") == "p1 world"
            assert obj.compute(5) == 500

        assert obj.greet("world") == "hello world"
        assert obj.compute(5) == 11
```
**EN:** This test method exercises context manager multiple patches and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 context manager multiple patches 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 235-256: Run test: restores on exception
```python
    def test_restores_on_exception(self, sample_module: ModuleType) -> None:
        cls = sample_module.SampleClass
        obj = cls()

        patches = [
            PatchSpec(
                target=f"{SAMPLE_MODULE_NAME}.SampleClass.greet",
                edits=[
                    EditSpec(
                        match='greeting = f"hello {name}"',
                        replacement='greeting = f"err_patched {name}"',
                    )
                ],
            )
        ]

        with pytest.raises(RuntimeError):
            with CodePatcher(patches=patches):
                assert obj.greet("world") == "err_patched world"
                raise RuntimeError("test error")

        assert obj.greet("world") == "hello world"
```
**EN:** This test method exercises restores on exception and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 restores on exception 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 259-262: Expose unittest entrypoint
```python
if __name__ == "__main__":
    import sys

    sys.exit(pytest.main([__file__, "-v"]))
```
**EN:** This conditional keeps the module executable as a standalone unittest target.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.debug_utils.source_patcher.code_patcher`, `sglang.srt.debug_utils.source_patcher.types`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pytest`, `sys`, `types`
- Notable symbols / 关键符号: None / 无
