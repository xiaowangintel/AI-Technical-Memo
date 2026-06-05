# test_dim_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/dims_spec/test_dim_parser.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on dims spec dim parser in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 dims spec dim parser 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Import dependencies
```python
import sys

import pytest

from sglang.srt.debug_utils.comparator.dims_spec import (
    DimSpec,
    Ordering,
    ParallelAxis,
    ParallelModifier,
    Reduction,
    parse_dim,
)
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 15-15: Register CI metadata
```python
register_cpu_ci(est_time=5, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 18-18: Define class TestParseDim
```python
class TestParseDim:
```
**EN:** This declaration introduces the `TestParseDim` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestParseDim` 测试类，并说明它通过继承承担的职责。

### Lines 19-20: Run test: plain name
```python
    def test_plain_name(self) -> None:
        assert parse_dim("b") == DimSpec(name="b")
```
**EN:** This test method exercises plain name and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 plain name 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 22-26: Run test: parallel axis
```python
    def test_parallel_axis(self) -> None:
        assert parse_dim("h[tp]") == DimSpec(
            name="h",
            parallel_modifiers=[ParallelModifier(axis=ParallelAxis.TP)],
        )
```
**EN:** This test method exercises parallel axis and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 parallel axis 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 28-32: Run test: all parallel axes
```python
    def test_all_parallel_axes(self) -> None:
        assert parse_dim("a[tp]").parallel_modifiers[0].axis == ParallelAxis.TP
        assert parse_dim("a[cp]").parallel_modifiers[0].axis == ParallelAxis.CP
        assert parse_dim("a[ep]").parallel_modifiers[0].axis == ParallelAxis.EP
        assert parse_dim("a[sp]").parallel_modifiers[0].axis == ParallelAxis.SP
```
**EN:** This test method exercises all parallel axes and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 all parallel axes 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 34-41: Run test: ordering
```python
    def test_ordering(self) -> None:
        assert (
            parse_dim("s[cp:zigzag]").parallel_modifiers[0].ordering == Ordering.ZIGZAG
        )
        assert (
            parse_dim("s[cp:natural]").parallel_modifiers[0].ordering
            == Ordering.NATURAL
        )
```
**EN:** This test method exercises ordering and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 ordering 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 43-47: Run test: reduction
```python
    def test_reduction(self) -> None:
        assert (
            parse_dim("h[tp:partial]").parallel_modifiers[0].reduction
            == Reduction.PARTIAL
        )
```
**EN:** This test method exercises reduction and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 reduction 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 49-59: Run test: all qualifiers
```python
    def test_all_qualifiers(self) -> None:
        assert parse_dim("s[cp:zigzag+partial]") == DimSpec(
            name="s",
            parallel_modifiers=[
                ParallelModifier(
                    axis=ParallelAxis.CP,
                    ordering=Ordering.ZIGZAG,
                    reduction=Reduction.PARTIAL,
                ),
            ],
        )
```
**EN:** This test method exercises all qualifiers and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 all qualifiers 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 61-68: Run test: multi axis
```python
    def test_multi_axis(self) -> None:
        result: DimSpec = parse_dim("t[cp:zigzag,sp]")
        assert result.name == "t"
        assert len(result.parallel_modifiers) == 2
        assert result.parallel_modifiers[0] == ParallelModifier(
            axis=ParallelAxis.CP, ordering=Ordering.ZIGZAG
        )
        assert result.parallel_modifiers[1] == ParallelModifier(axis=ParallelAxis.SP)
```
**EN:** This test method exercises multi axis and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multi axis 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 70-74: Run test: invalid token raises
```python
    def test_invalid_token_raises(self) -> None:
        with pytest.raises(ValueError, match="Invalid dim token"):
            parse_dim("h[]")
        with pytest.raises(ValueError, match="Invalid dim token"):
            parse_dim("h[tp[x]]")
```
**EN:** This test method exercises invalid token raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 invalid token raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 76-78: Run test: unknown axis raises
```python
    def test_unknown_axis_raises(self) -> None:
        with pytest.raises(ValueError, match="Unknown axis"):
            parse_dim("h[xyz]")
```
**EN:** This test method exercises unknown axis raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 unknown axis raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 80-82: Run test: unknown qualifier raises
```python
    def test_unknown_qualifier_raises(self) -> None:
        with pytest.raises(ValueError, match="Unknown qualifier"):
            parse_dim("h[tp:foobar]")
```
**EN:** This test method exercises unknown qualifier raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 unknown qualifier raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 84-86: Run test: multiple ordering raises
```python
    def test_multiple_ordering_raises(self) -> None:
        with pytest.raises(ValueError, match="Multiple ordering"):
            parse_dim("s[cp:zigzag+natural]")
```
**EN:** This test method exercises multiple ordering raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 multiple ordering raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 88-90: Run test: multiple reduction raises
```python
    def test_multiple_reduction_raises(self) -> None:
        with pytest.raises(ValueError, match="Multiple reduction"):
            parse_dim("h[tp:partial+partial]")
```
**EN:** This test method exercises multiple reduction raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 multiple reduction raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 92-94: Run test: duplicate axis raises
```python
    def test_duplicate_axis_raises(self) -> None:
        with pytest.raises(ValueError, match="Duplicate axis"):
            parse_dim("h[tp,tp]")
```
**EN:** This test method exercises duplicate axis raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 duplicate axis raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 96-97: Run test: squeeze dim
```python
    def test_squeeze_dim(self) -> None:
        assert parse_dim("1") == DimSpec(name="1")
```
**EN:** This test method exercises squeeze dim and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 squeeze dim 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 99-101: Run test: squeeze dim rejects modifiers
```python
    def test_squeeze_dim_rejects_modifiers(self) -> None:
        with pytest.raises(ValueError, match="Invalid dim token"):
            parse_dim("1[tp]")
```
**EN:** This test method exercises squeeze dim rejects modifiers and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 squeeze dim rejects modifiers 场景，并验证观测到的行为是否符合预期契约。

### Lines 104-104: Define class TestParseFusedDim
```python
class TestParseFusedDim:
```
**EN:** This declaration introduces the `TestParseFusedDim` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestParseFusedDim` 测试类，并说明它通过继承承担的职责。

### Lines 105-110: Run test: basic fused
```python
    def test_basic_fused(self) -> None:
        result: DimSpec = parse_dim("(num_heads*head_dim)")
        assert result.name == "num_heads*head_dim"
        assert result.parallel_modifiers == []
        assert result.is_fused
        assert result.sub_dims == ["num_heads", "head_dim"]
```
**EN:** This test method exercises basic fused and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 basic fused 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 112-116: Run test: fused with modifier
```python
    def test_fused_with_modifier(self) -> None:
        result: DimSpec = parse_dim("(num_heads*head_dim)[tp]")
        assert result.name == "num_heads*head_dim"
        assert result.parallel_modifiers == [ParallelModifier(axis=ParallelAxis.TP)]
        assert result.sub_dims == ["num_heads", "head_dim"]
```
**EN:** This test method exercises fused with modifier and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fused with modifier 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 118-122: Run test: three way fused
```python
    def test_three_way_fused(self) -> None:
        result: DimSpec = parse_dim("(a*b*c)")
        assert result.name == "a*b*c"
        assert len(result.sub_dims) == 3
        assert result.sub_dims == ["a", "b", "c"]
```
**EN:** This test method exercises three way fused and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 three way fused 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 124-127: Run test: three way fused with modifier
```python
    def test_three_way_fused_with_modifier(self) -> None:
        result: DimSpec = parse_dim("(a*b*c)[tp]")
        assert result.parallel_modifiers == [ParallelModifier(axis=ParallelAxis.TP)]
        assert len(result.sub_dims) == 3
```
**EN:** This test method exercises three way fused with modifier and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 three way fused with modifier 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 129-134: Run test: fused with complex modifier
```python
    def test_fused_with_complex_modifier(self) -> None:
        result: DimSpec = parse_dim("(a*b)[cp:zigzag]")
        assert result.parallel_modifiers == [
            ParallelModifier(axis=ParallelAxis.CP, ordering=Ordering.ZIGZAG)
        ]
        assert result.sub_dims == ["a", "b"]
```
**EN:** This test method exercises fused with complex modifier and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fused with complex modifier 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 136-139: Run test: regular dim not fused
```python
    def test_regular_dim_not_fused(self) -> None:
        result: DimSpec = parse_dim("h[tp]")
        assert not result.is_fused
        assert result.sub_dims == ["h"]
```
**EN:** This test method exercises regular dim not fused and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 regular dim not fused 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 141-143: Run test: fused duplicate sub names raises
```python
    def test_fused_duplicate_sub_names_raises(self) -> None:
        with pytest.raises(ValueError, match="Duplicate sub-dim"):
            parse_dim("(a*a)")
```
**EN:** This test method exercises fused duplicate sub names raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 fused duplicate sub names raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 145-147: Run test: fused invalid sub dim raises
```python
    def test_fused_invalid_sub_dim_raises(self) -> None:
        with pytest.raises(ValueError, match="Invalid sub-dim"):
            parse_dim("(a*1)")
```
**EN:** This test method exercises fused invalid sub dim raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 fused invalid sub dim raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 150-151: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pytest`, `sys`
- Notable symbols / 关键符号: None / 无
