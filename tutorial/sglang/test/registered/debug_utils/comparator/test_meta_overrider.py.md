# test_meta_overrider.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/test_meta_overrider.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on comparator meta overrider in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 comparator meta overrider 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Document the module
```python
"""Tests for meta_overrider — unit tests."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 3-17: Import dependencies
```python
from __future__ import annotations

import sys
import textwrap
from pathlib import Path

import pytest

from sglang.srt.debug_utils.comparator.meta_overrider import (
    MetaOverrider,
    MetaOverrideRule,
    _load_yaml_rules,
    _parse_cli_override_arg,
)
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 19-19: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 25-25: Define class TestMetaOverrideRule
```python
class TestMetaOverrideRule:
```
**EN:** This declaration introduces the `TestMetaOverrideRule` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestMetaOverrideRule` 测试类，并说明它通过继承承担的职责。

### Lines 26-26: Document the class `TestMetaOverrideRule`
```python
    """Pydantic validation for MetaOverrideRule."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestMetaOverrideRule`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestMetaOverrideRule`的设计意图。

### Lines 28-32: Run test: shared dims both
```python
    def test_shared_dims_both(self) -> None:
        """Default side='both' applies dims to both sides."""
        rule = MetaOverrideRule(match="hidden", dims="b s h d")
        assert rule.dims == "b s h d"
        assert rule.side == "both"
```
**EN:** This test method exercises shared dims both and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 shared dims both 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 34-38: Run test: side baseline
```python
    def test_side_baseline(self) -> None:
        """side='baseline' is accepted."""
        rule = MetaOverrideRule(match="logits", dims="b s v[tp]", side="baseline")
        assert rule.dims == "b s v[tp]"
        assert rule.side == "baseline"
```
**EN:** This test method exercises side baseline and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 side baseline 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 40-44: Run test: side target
```python
    def test_side_target(self) -> None:
        """side='target' is accepted."""
        rule = MetaOverrideRule(match="logits", dims="b s v[ep]", side="target")
        assert rule.dims == "b s v[ep]"
        assert rule.side == "target"
```
**EN:** This test method exercises side target and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 side target 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 46-49: Run test: invalid side rejected
```python
    def test_invalid_side_rejected(self) -> None:
        """Invalid side value is rejected."""
        with pytest.raises(Exception):
            MetaOverrideRule(match="x", dims="b s", side="invalid")
```
**EN:** This test method exercises invalid side rejected and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 invalid side rejected 场景，并验证观测到的行为是否符合预期契约。

### Lines 51-54: Run test: dims required
```python
    def test_dims_required(self) -> None:
        """Must specify dims."""
        with pytest.raises(Exception):
            MetaOverrideRule(match="x")
```
**EN:** This test method exercises dims required and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 dims required 场景，并验证观测到的行为是否符合预期契约。

### Lines 56-59: Run test: extra field rejected
```python
    def test_extra_field_rejected(self) -> None:
        """Extra fields are rejected by _StrictBase."""
        with pytest.raises(Exception):
            MetaOverrideRule(match="x", dims="b s", bogus="y")
```
**EN:** This test method exercises extra field rejected and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 extra field rejected 场景，并验证观测到的行为是否符合预期契约。

### Lines 65-65: Define class TestParseCLIOverrideArg
```python
class TestParseCLIOverrideArg:
```
**EN:** This declaration introduces the `TestParseCLIOverrideArg` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestParseCLIOverrideArg` 测试类，并说明它通过继承承担的职责。

### Lines 66-66: Document the class `TestParseCLIOverrideArg`
```python
    """CLI arg parsing for 'name:dims_string' format."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestParseCLIOverrideArg`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestParseCLIOverrideArg`的设计意图。

### Lines 68-72: Run test: basic
```python
    def test_basic(self) -> None:
        """Standard 'name:dims' parsing."""
        name, dims_str = _parse_cli_override_arg("hidden_states:b s h d")
        assert name == "hidden_states"
        assert dims_str == "b s h d"
```
**EN:** This test method exercises basic and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 basic 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 74-78: Run test: colon in dims
```python
    def test_colon_in_dims(self) -> None:
        """Extra colons in dims are kept (maxsplit=1)."""
        name, dims_str = _parse_cli_override_arg("x:a:b")
        assert name == "x"
        assert dims_str == "a:b"
```
**EN:** This test method exercises colon in dims and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 colon in dims 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 80-84: Run test: whitespace trimmed
```python
    def test_whitespace_trimmed(self) -> None:
        """Leading/trailing whitespace around name and dims is stripped."""
        name, dims_str = _parse_cli_override_arg("  foo  :  b s  ")
        assert name == "foo"
        assert dims_str == "b s"
```
**EN:** This test method exercises whitespace trimmed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 whitespace trimmed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 86-89: Run test: missing colon
```python
    def test_missing_colon(self) -> None:
        """No colon raises ValueError."""
        with pytest.raises(ValueError, match="Invalid override format"):
            _parse_cli_override_arg("no_colon_here")
```
**EN:** This test method exercises missing colon and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 missing colon 场景，并验证观测到的行为是否符合预期契约。

### Lines 91-94: Run test: empty name
```python
    def test_empty_name(self) -> None:
        """Empty name raises ValueError."""
        with pytest.raises(ValueError, match="Invalid override format"):
            _parse_cli_override_arg(":b s h")
```
**EN:** This test method exercises empty name and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 empty name 场景，并验证观测到的行为是否符合预期契约。

### Lines 96-99: Run test: empty dims
```python
    def test_empty_dims(self) -> None:
        """Empty dims raises ValueError."""
        with pytest.raises(ValueError, match="Invalid override format"):
            _parse_cli_override_arg("foo:")
```
**EN:** This test method exercises empty dims and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 empty dims 场景，并验证观测到的行为是否符合预期契约。

### Lines 105-105: Define class TestMetaOverrider
```python
class TestMetaOverrider:
```
**EN:** This declaration introduces the `TestMetaOverrider` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestMetaOverrider` 测试类，并说明它通过继承承担的职责。

### Lines 106-106: Document the class `TestMetaOverrider`
```python
    """MetaOverrider logic: matching, priority, apply_to_meta."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestMetaOverrider`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestMetaOverrider`的设计意图。

### Lines 108-121: Run test: first match wins
```python
    def test_first_match_wins(self) -> None:
        """First matching rule takes effect; later rules ignored."""
        overrider = MetaOverrider(
            rules=[
                MetaOverrideRule(match="hidden", dims="FIRST"),
                MetaOverrideRule(match="hidden", dims="SECOND"),
            ]
        )
        result: dict = overrider.apply_to_meta(
            name="hidden_states",
            meta={"dims": "old"},
            side="baseline",
        )
        assert result["dims"] == "FIRST"
```
**EN:** This test method exercises first match wins and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 first match wins 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 123-133: Run test: regex contains match
```python
    def test_regex_contains_match(self) -> None:
        """match is a regex contains search, not exact match."""
        overrider = MetaOverrider(
            rules=[MetaOverrideRule(match=r"\.q_proj\.", dims="h d")]
        )
        result: dict = overrider.apply_to_meta(
            name="layers.0.q_proj.weight",
            meta={"dims": "old"},
            side="baseline",
        )
        assert result["dims"] == "h d"
```
**EN:** This test method exercises regex contains match and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 regex contains match 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 135-145: Run test: no match preserves original
```python
    def test_no_match_preserves_original(self) -> None:
        """No matching rule leaves meta untouched."""
        overrider = MetaOverrider(
            rules=[MetaOverrideRule(match="logits", dims="b s v")]
        )
        result: dict = overrider.apply_to_meta(
            name="hidden_states",
            meta={"dims": "original"},
            side="baseline",
        )
        assert result["dims"] == "original"
```
**EN:** This test method exercises no match preserves original and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no match preserves original 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 147-170: Run test: side filtering
```python
    @pytest.mark.parametrize(
        "rule_side,apply_side,should_match",
        [
            ("baseline", "baseline", True),
            ("baseline", "target", False),
            ("target", "target", True),
            ("target", "baseline", False),
            ("both", "baseline", True),
            ("both", "target", True),
        ],
    )
    def test_side_filtering(
        self, rule_side: str, apply_side: str, should_match: bool
    ) -> None:
        """Rule only applies when its side matches the apply side."""
        overrider = MetaOverrider(
            rules=[MetaOverrideRule(match="logits", dims="NEW", side=rule_side)]
        )
        result: dict = overrider.apply_to_meta(
            name="logits",
            meta={"dims": "old"},
            side=apply_side,
        )
        assert result["dims"] == ("NEW" if should_match else "old")
```
**EN:** This test method exercises side filtering and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 side filtering 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 172-175: Run test: is empty
```python
    def test_is_empty(self) -> None:
        """Empty overrider reports is_empty=True."""
        assert MetaOverrider(rules=[]).is_empty
        assert not MetaOverrider(rules=[MetaOverrideRule(match="x", dims="d")]).is_empty
```
**EN:** This test method exercises is empty and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 is empty 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 177-185: Run test: meta without dims key
```python
    def test_meta_without_dims_key(self) -> None:
        """Override adds 'dims' even if original meta lacks it."""
        overrider = MetaOverrider(rules=[MetaOverrideRule(match="hidden", dims="NEW")])
        result: dict = overrider.apply_to_meta(
            name="hidden",
            meta={"other": "val"},
            side="baseline",
        )
        assert result["dims"] == "NEW"
```
**EN:** This test method exercises meta without dims key and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 meta without dims key 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 191-191: Define class TestFromArgsAndConfig
```python
class TestFromArgsAndConfig:
```
**EN:** This declaration introduces the `TestFromArgsAndConfig` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFromArgsAndConfig` 测试类，并说明它通过继承承担的职责。

### Lines 192-192: Document the class `TestFromArgsAndConfig`
```python
    """MetaOverrider.from_args_and_config merges CLI + YAML rules."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestFromArgsAndConfig`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestFromArgsAndConfig`的设计意图。

### Lines 194-215: Run test: cli before yaml
```python
    def test_cli_before_yaml(self, tmp_path: Path) -> None:
        """CLI rules are ordered before YAML rules (CLI wins on conflict)."""
        yaml_path = tmp_path / "override.yaml"
        yaml_path.write_text(textwrap.dedent("""\
            overrides:
              - match: "hidden"
                dims: "FROM_YAML"
        """))

        overrider = MetaOverrider.from_args_and_config(
            override_dims=["hidden:FROM_CLI"],
            override_baseline_dims=[],
            override_target_dims=[],
            override_config=yaml_path,
        )

        result: dict = overrider.apply_to_meta(
            name="hidden",
            meta={"dims": "old"},
            side="baseline",
        )
        assert result["dims"] == "FROM_CLI"
```
**EN:** This test method exercises cli before yaml and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cli before yaml 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 217-225: Run test: no config no cli
```python
    def test_no_config_no_cli(self) -> None:
        """Empty CLI + no YAML yields empty overrider."""
        overrider = MetaOverrider.from_args_and_config(
            override_dims=[],
            override_baseline_dims=[],
            override_target_dims=[],
            override_config=None,
        )
        assert overrider.is_empty
```
**EN:** This test method exercises no config no cli and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no config no cli 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 227-247: Run test: per side cli produces separate rules
```python
    def test_per_side_cli_produces_separate_rules(self) -> None:
        """--override-baseline-dims and --override-target-dims produce separate rules with side field."""
        overrider = MetaOverrider.from_args_and_config(
            override_dims=[],
            override_baseline_dims=["hidden:b s h[tp]"],
            override_target_dims=["hidden:b s h[ep]"],
            override_config=None,
        )

        baseline: dict = overrider.apply_to_meta(
            name="hidden",
            meta={"dims": "old"},
            side="baseline",
        )
        target: dict = overrider.apply_to_meta(
            name="hidden",
            meta={"dims": "old"},
            side="target",
        )
        assert baseline["dims"] == "b s h[tp]"
        assert target["dims"] == "b s h[ep]"
```
**EN:** This test method exercises per side cli produces separate rules and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 per side cli produces separate rules 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 253-253: Define class TestLoadYamlRules
```python
class TestLoadYamlRules:
```
**EN:** This declaration introduces the `TestLoadYamlRules` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestLoadYamlRules` 测试类，并说明它通过继承承担的职责。

### Lines 254-254: Document the class `TestLoadYamlRules`
```python
    """YAML loading and validation."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestLoadYamlRules`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestLoadYamlRules`的设计意图。

### Lines 256-272: Run test: valid yaml
```python
    def test_valid_yaml(self, tmp_path: Path) -> None:
        """Valid YAML with override rules loads correctly."""
        yaml_path = tmp_path / "override.yaml"
        yaml_path.write_text(textwrap.dedent("""\
            overrides:
              - match: "hidden"
                dims: "b s h d"
              - match: "logits"
                dims: "b s v[tp]"
                side: baseline
        """))
        rules = _load_yaml_rules(yaml_path)
        assert len(rules) == 2
        assert rules[0].dims == "b s h d"
        assert rules[0].side == "both"
        assert rules[1].dims == "b s v[tp]"
        assert rules[1].side == "baseline"
```
**EN:** This test method exercises valid yaml and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 valid yaml 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 274-279: Run test: empty yaml
```python
    def test_empty_yaml(self, tmp_path: Path) -> None:
        """Empty YAML file returns no rules."""
        yaml_path = tmp_path / "empty.yaml"
        yaml_path.write_text("")
        rules = _load_yaml_rules(yaml_path)
        assert rules == []
```
**EN:** This test method exercises empty yaml and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty yaml 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 281-286: Run test: unknown top key rejected
```python
    def test_unknown_top_key_rejected(self, tmp_path: Path) -> None:
        """Unknown top-level key is rejected by OverrideConfig."""
        yaml_path = tmp_path / "bad.yaml"
        yaml_path.write_text("unknown_key: 42\n")
        with pytest.raises(Exception):
            _load_yaml_rules(yaml_path)
```
**EN:** This test method exercises unknown top key rejected and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 unknown top key rejected 场景，并验证观测到的行为是否符合预期契约。

### Lines 288-293: Run test: overrides empty list
```python
    def test_overrides_empty_list(self, tmp_path: Path) -> None:
        """Only 'overrides' key with no entries returns empty list."""
        yaml_path = tmp_path / "minimal.yaml"
        yaml_path.write_text("overrides: []\n")
        rules = _load_yaml_rules(yaml_path)
        assert rules == []
```
**EN:** This test method exercises overrides empty list and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 overrides empty list 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 296-297: Expose unittest entrypoint
```python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
```
**EN:** This conditional keeps the module executable as a standalone unittest target.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.meta_overrider`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `__future__`, `pathlib`, `pytest`, `sys`, `textwrap`
- Notable symbols / 关键符号: None / 无
