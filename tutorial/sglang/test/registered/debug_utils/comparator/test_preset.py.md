# test_preset.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/test_preset.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on comparator preset in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 comparator preset 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Import dependencies
```python
import pytest

from sglang.srt.debug_utils.comparator.preset import PRESETS, expand_preset
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 6-6: Register CI metadata
```python
register_cpu_ci(est_time=5, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 9-9: Define class TestExpandPreset
```python
class TestExpandPreset:
```
**EN:** This declaration introduces the `TestExpandPreset` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestExpandPreset` 测试类，并说明它通过继承承担的职责。

### Lines 10-10: Document the class `TestExpandPreset`
```python
    """Test preset expansion logic."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestExpandPreset`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestExpandPreset`的设计意图。

### Lines 12-27: Run test: explicit preset
```python
    def test_explicit_preset(self):
        """--preset sglang_megatron expands into its argv."""
        argv = [
            "--baseline-path",
            "/a",
            "--preset",
            "sglang_megatron",
            "--diff-threshold",
            "0.01",
        ]
        result = expand_preset(argv, presets=PRESETS)
        assert "--preset" not in result
        assert "--grouping-skip-keys" in result
        assert "concat_steps" in result
        assert "--baseline-path" in result
        assert "--diff-threshold" in result
```
**EN:** This test method exercises explicit preset and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 explicit preset 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 29-33: Run test: default preset applied
```python
    def test_default_preset_applied(self):
        """No --preset and no --grouping-skip-keys triggers default preset."""
        argv = ["--baseline-path", "/a"]
        result = expand_preset(argv, presets=PRESETS)
        assert "--grouping-skip-keys" in result
```
**EN:** This test method exercises default preset applied and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 default preset applied 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 35-39: Run test: explicit skip keys prevents default
```python
    def test_explicit_skip_keys_prevents_default(self):
        """Explicit --grouping-skip-keys prevents default preset injection."""
        argv = ["--grouping-skip-keys", "rank", "--baseline-path", "/a"]
        result = expand_preset(argv, presets=PRESETS)
        assert result == argv
```
**EN:** This test method exercises explicit skip keys prevents default and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 explicit skip keys prevents default 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 41-44: Run test: unknown preset raises
```python
    def test_unknown_preset_raises(self):
        """Unknown preset name raises ValueError."""
        with pytest.raises(ValueError, match="Unknown value for --preset"):
            expand_preset(["--preset", "nonexistent"], presets=PRESETS)
```
**EN:** This test method exercises unknown preset raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 unknown preset raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 47-50: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.preset`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pytest`, `sys`
- Notable symbols / 关键符号: None / 无
