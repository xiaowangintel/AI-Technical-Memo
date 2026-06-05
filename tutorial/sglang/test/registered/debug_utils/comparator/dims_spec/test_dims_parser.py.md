# test_dims_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/dims_spec/test_dims_parser.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on dims spec dims parser in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 dims spec dims parser 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Import dependencies
```python
import sys

import pytest

from sglang.srt.debug_utils.comparator.dims_spec import (
    SQUEEZE_DIM_NAME,
    DimSpec,
    DimsSpec,
    Ordering,
    ParallelAxis,
    ParallelModifier,
    _SingletonDimUtil,
    parse_dims,
    resolve_dim_names,
)
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 18-18: Register CI metadata
```python
register_cpu_ci(est_time=5, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 21-21: Define class TestSingletonDimUtilFilterOut
```python
class TestSingletonDimUtilFilterOut:
```
**EN:** This declaration introduces the `TestSingletonDimUtilFilterOut` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestSingletonDimUtilFilterOut` 测试类，并说明它通过继承承担的职责。

### Lines 22-24: Run test: no squeeze
```python
    def test_no_squeeze(self) -> None:
        specs: list[DimSpec] = parse_dims("t h d").dims
        assert _SingletonDimUtil.filter_out(specs) == specs
```
**EN:** This test method exercises no squeeze and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no squeeze 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 26-31: Run test: with squeeze
```python
    def test_with_squeeze(self) -> None:
        specs: list[DimSpec] = parse_dims("t 1 h").dims
        filtered: list[DimSpec] = _SingletonDimUtil.filter_out(specs)
        assert len(filtered) == 2
        assert filtered[0].name == "t"
        assert filtered[1].name == "h"
```
**EN:** This test method exercises with squeeze and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with squeeze 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 33-35: Run test: all squeeze
```python
    def test_all_squeeze(self) -> None:
        specs: list[DimSpec] = parse_dims("1 1").dims
        assert _SingletonDimUtil.filter_out(specs) == []
```
**EN:** This test method exercises all squeeze and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 all squeeze 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 38-38: Define class TestSingletonDimUtilIsSqueeze
```python
class TestSingletonDimUtilIsSqueeze:
```
**EN:** This declaration introduces the `TestSingletonDimUtilIsSqueeze` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestSingletonDimUtilIsSqueeze` 测试类，并说明它通过继承承担的职责。

### Lines 39-40: Run test: squeeze
```python
    def test_squeeze(self) -> None:
        assert _SingletonDimUtil.is_squeeze(DimSpec(name=SQUEEZE_DIM_NAME)) is True
```
**EN:** This test method exercises squeeze and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 squeeze 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 42-43: Run test: non squeeze
```python
    def test_non_squeeze(self) -> None:
        assert _SingletonDimUtil.is_squeeze(DimSpec(name="t")) is False
```
**EN:** This test method exercises non squeeze and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 non squeeze 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 46-46: Define class TestSingletonDimUtilMakeName
```python
class TestSingletonDimUtilMakeName:
```
**EN:** This declaration introduces the `TestSingletonDimUtilMakeName` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestSingletonDimUtilMakeName` 测试类，并说明它通过继承承担的职责。

### Lines 47-50: Run test: indices
```python
    def test_indices(self) -> None:
        assert _SingletonDimUtil.make_name(0) == "singleton0"
        assert _SingletonDimUtil.make_name(1) == "singleton1"
        assert _SingletonDimUtil.make_name(99) == "singleton99"
```
**EN:** This test method exercises indices and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 indices 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 53-53: Define class TestSingletonDimUtilSanitizeNames
```python
class TestSingletonDimUtilSanitizeNames:
```
**EN:** This declaration introduces the `TestSingletonDimUtilSanitizeNames` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestSingletonDimUtilSanitizeNames` 测试类，并说明它通过继承承担的职责。

### Lines 54-55: Run test: no squeeze
```python
    def test_no_squeeze(self) -> None:
        assert _SingletonDimUtil.sanitize_names(["t", "h", "d"]) == ["t", "h", "d"]
```
**EN:** This test method exercises no squeeze and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no squeeze 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 57-62: Run test: single squeeze
```python
    def test_single_squeeze(self) -> None:
        assert _SingletonDimUtil.sanitize_names(["t", "1", "h"]) == [
            "t",
            "singleton0",
            "h",
        ]
```
**EN:** This test method exercises single squeeze and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single squeeze 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 64-70: Run test: multiple squeeze
```python
    def test_multiple_squeeze(self) -> None:
        assert _SingletonDimUtil.sanitize_names(["1", "t", "1", "h"]) == [
            "singleton0",
            "t",
            "singleton1",
            "h",
        ]
```
**EN:** This test method exercises multiple squeeze and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multiple squeeze 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 72-73: Run test: empty
```python
    def test_empty(self) -> None:
        assert _SingletonDimUtil.sanitize_names([]) == []
```
**EN:** This test method exercises empty and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 76-76: Define class TestParseDims
```python
class TestParseDims:
```
**EN:** This declaration introduces the `TestParseDims` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestParseDims` 测试类，并说明它通过继承承担的职责。

### Lines 77-83: Run test: multi dims
```python
    def test_multi_dims(self) -> None:
        assert parse_dims("b s h d").dims == [
            DimSpec(name="b"),
            DimSpec(name="s"),
            DimSpec(name="h"),
            DimSpec(name="d"),
        ]
```
**EN:** This test method exercises multi dims and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multi dims 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 85-86: Run test: single dim
```python
    def test_single_dim(self) -> None:
        assert parse_dims("t").dims == [DimSpec(name="t")]
```
**EN:** This test method exercises single dim and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single dim 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 88-102: Run test: mixed annotated
```python
    def test_mixed_annotated(self) -> None:
        assert parse_dims("b s[cp:zigzag] h[tp] d").dims == [
            DimSpec(name="b"),
            DimSpec(
                name="s",
                parallel_modifiers=[
                    ParallelModifier(axis=ParallelAxis.CP, ordering=Ordering.ZIGZAG),
                ],
            ),
            DimSpec(
                name="h",
                parallel_modifiers=[ParallelModifier(axis=ParallelAxis.TP)],
            ),
            DimSpec(name="d"),
        ]
```
**EN:** This test method exercises mixed annotated and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 mixed annotated 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 104-106: Run test: empty string raises
```python
    def test_empty_string_raises(self) -> None:
        with pytest.raises(ValueError, match="empty"):
            parse_dims("")
```
**EN:** This test method exercises empty string raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 empty string raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 108-110: Run test: whitespace only raises
```python
    def test_whitespace_only_raises(self) -> None:
        with pytest.raises(ValueError, match="empty"):
            parse_dims("   ")
```
**EN:** This test method exercises whitespace only raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 whitespace only raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 112-114: Run test: duplicate name raises
```python
    def test_duplicate_name_raises(self) -> None:
        with pytest.raises(ValueError, match="Duplicate"):
            parse_dims("h h")
```
**EN:** This test method exercises duplicate name raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 duplicate name raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 116-121: Run test: with squeeze dims
```python
    def test_with_squeeze_dims(self) -> None:
        dims: list[DimSpec] = parse_dims("t 1 h").dims
        assert len(dims) == 3
        assert dims[0] == DimSpec(name="t")
        assert dims[1] == DimSpec(name="1")
        assert dims[2] == DimSpec(name="h")
```
**EN:** This test method exercises with squeeze dims and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with squeeze dims 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 123-127: Run test: multiple squeeze dims no duplicate error
```python
    def test_multiple_squeeze_dims_no_duplicate_error(self) -> None:
        dims: list[DimSpec] = parse_dims("t 1 h 1 d").dims
        assert len(dims) == 5
        assert dims[1] == DimSpec(name="1")
        assert dims[3] == DimSpec(name="1")
```
**EN:** This test method exercises multiple squeeze dims no duplicate error and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multiple squeeze dims no duplicate error 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 130-130: Define class TestParseDimsWithFused
```python
class TestParseDimsWithFused:
```
**EN:** This declaration introduces the `TestParseDimsWithFused` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestParseDimsWithFused` 测试类，并说明它通过继承承担的职责。

### Lines 131-136: Run test: fused in dims
```python
    def test_fused_in_dims(self) -> None:
        result: DimsSpec = parse_dims("t (num_heads*head_dim)[tp]")
        assert len(result.dims) == 2
        assert result.dims[0] == DimSpec(name="t")
        assert result.dims[1].is_fused
        assert result.dims[1].name == "num_heads*head_dim"
```
**EN:** This test method exercises fused in dims and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fused in dims 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 138-143: Run test: fused and regular mixed
```python
    def test_fused_and_regular_mixed(self) -> None:
        result: DimsSpec = parse_dims("t (num_heads*head_dim)[tp] d")
        assert len(result.dims) == 3
        assert not result.dims[0].is_fused
        assert result.dims[1].is_fused
        assert not result.dims[2].is_fused
```
**EN:** This test method exercises fused and regular mixed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fused and regular mixed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 145-147: Run test: fused sub name conflicts with regular raises
```python
    def test_fused_sub_name_conflicts_with_regular_raises(self) -> None:
        with pytest.raises(ValueError, match="Duplicate"):
            parse_dims("t num_heads (num_heads*head_dim)")
```
**EN:** This test method exercises fused sub name conflicts with regular raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 fused sub name conflicts with regular raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 149-153: Run test: multiple fused dims
```python
    def test_multiple_fused_dims(self) -> None:
        result: DimsSpec = parse_dims("(a*b) (c*d)")
        assert len(result.dims) == 2
        assert result.dims[0].is_fused
        assert result.dims[1].is_fused
```
**EN:** This test method exercises multiple fused dims and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multiple fused dims 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 155-157: Run test: cross fused duplicate sub name raises
```python
    def test_cross_fused_duplicate_sub_name_raises(self) -> None:
        with pytest.raises(ValueError, match="Duplicate"):
            parse_dims("(a*b) (c*a)")
```
**EN:** This test method exercises cross fused duplicate sub name raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 cross fused duplicate sub name raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 160-160: Define class TestParseDimsWithHash
```python
class TestParseDimsWithHash:
```
**EN:** This declaration introduces the `TestParseDimsWithHash` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestParseDimsWithHash` 测试类，并说明它通过继承承担的职责。

### Lines 161-161: Document the class `TestParseDimsWithHash`
```python
    """parse_dims strips the ``#`` declaration section from dims."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestParseDimsWithHash`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestParseDimsWithHash`的设计意图。

### Lines 163-164: Run test: shape dims unchanged
```python
    def test_shape_dims_unchanged(self) -> None:
        assert parse_dims("b s h[tp] # dp:=moe_dp").dims == parse_dims("b s h[tp]").dims
```
**EN:** This test method exercises shape dims unchanged and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 shape dims unchanged 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 166-167: Run test: dp group alias extracted
```python
    def test_dp_group_alias_extracted(self) -> None:
        assert parse_dims("b s h[tp] # dp:=moe_dp").dp_group_alias == "moe_dp"
```
**EN:** This test method exercises dp group alias extracted and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp group alias extracted 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 169-170: Run test: no hash no alias
```python
    def test_no_hash_no_alias(self) -> None:
        assert parse_dims("b s h[tp]").dp_group_alias is None
```
**EN:** This test method exercises no hash no alias and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no hash no alias 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 172-174: Run test: whitespace around hash
```python
    def test_whitespace_around_hash(self) -> None:
        assert parse_dims("t h #   dp:=foo  ").dims == parse_dims("t h").dims
        assert parse_dims("t h #   dp:=foo  ").dp_group_alias == "foo"
```
**EN:** This test method exercises whitespace around hash and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 whitespace around hash 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 176-180: Run test: multiple declarations picks dp
```python
    def test_multiple_declarations_picks_dp(self) -> None:
        result: DimsSpec = parse_dims("t h[tp] # dp:=moe_dp ep:replicated")
        assert result.dims == parse_dims("t h[tp]").dims
        assert result.dp_group_alias == "moe_dp"
        assert result.replicated_axes == frozenset({ParallelAxis.EP})
```
**EN:** This test method exercises multiple declarations picks dp and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multiple declarations picks dp 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 182-185: Run test: no dp alias token
```python
    def test_no_dp_alias_token(self) -> None:
        result: DimsSpec = parse_dims("t h[tp] # ep:replicated")
        assert result.dp_group_alias is None
        assert result.replicated_axes == frozenset({ParallelAxis.EP})
```
**EN:** This test method exercises no dp alias token and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no dp alias token 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 188-188: Define class TestDpGroupAlias
```python
class TestDpGroupAlias:
```
**EN:** This declaration introduces the `TestDpGroupAlias` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDpGroupAlias` 测试类，并说明它通过继承承担的职责。

### Lines 189-190: Run test: basic
```python
    def test_basic(self) -> None:
        assert parse_dims("b s h[tp] # dp:=moe_dp").dp_group_alias == "moe_dp"
```
**EN:** This test method exercises basic and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 basic 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 192-193: Run test: no hash returns none
```python
    def test_no_hash_returns_none(self) -> None:
        assert parse_dims("t h").dp_group_alias is None
```
**EN:** This test method exercises no hash returns none and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no hash returns none 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 195-196: Run test: no dp alias token
```python
    def test_no_dp_alias_token(self) -> None:
        assert parse_dims("t h[tp] # ep:replicated").dp_group_alias is None
```
**EN:** This test method exercises no dp alias token and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no dp alias token 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 198-202: Run test: multiple tokens picks dp
```python
    def test_multiple_tokens_picks_dp(self) -> None:
        assert (
            parse_dims("b s # ep:replicated dp:=custom_dp").dp_group_alias
            == "custom_dp"
        )
```
**EN:** This test method exercises multiple tokens picks dp and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multiple tokens picks dp 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 205-205: Define class TestExplicitReplicatedAxes
```python
class TestExplicitReplicatedAxes:
```
**EN:** This declaration introduces the `TestExplicitReplicatedAxes` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestExplicitReplicatedAxes` 测试类，并说明它通过继承承担的职责。

### Lines 206-208: Run test: single replicated
```python
    def test_single_replicated(self) -> None:
        result: DimsSpec = parse_dims("b s h[tp] d # ep:replicated")
        assert result.replicated_axes == frozenset({ParallelAxis.EP})
```
**EN:** This test method exercises single replicated and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single replicated 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 210-211: Run test: explicit sharded equivalent
```python
    def test_explicit_sharded_equivalent(self) -> None:
        assert parse_dims("b s h[tp:sharded] d").dims == parse_dims("b s h[tp] d").dims
```
**EN:** This test method exercises explicit sharded equivalent and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 explicit sharded equivalent 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 213-215: Run test: multiple replicated
```python
    def test_multiple_replicated(self) -> None:
        result: DimsSpec = parse_dims("b s h[tp] d # ep:replicated cp:replicated")
        assert result.replicated_axes == frozenset({ParallelAxis.EP, ParallelAxis.CP})
```
**EN:** This test method exercises multiple replicated and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multiple replicated 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 217-220: Run test: dp alias and replicated coexist
```python
    def test_dp_alias_and_replicated_coexist(self) -> None:
        result: DimsSpec = parse_dims("b s h[tp] d # dp:=moe_dp ep:replicated")
        assert result.dp_group_alias == "moe_dp"
        assert result.replicated_axes == frozenset({ParallelAxis.EP})
```
**EN:** This test method exercises dp alias and replicated coexist and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp alias and replicated coexist 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 222-224: Run test: no hash replicated empty
```python
    def test_no_hash_replicated_empty(self) -> None:
        result: DimsSpec = parse_dims("b s h[tp] d")
        assert result.replicated_axes == frozenset()
```
**EN:** This test method exercises no hash replicated empty and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no hash replicated empty 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 226-228: Run test: hash without replicated
```python
    def test_hash_without_replicated(self) -> None:
        result: DimsSpec = parse_dims("b s h[tp] d # dp:=moe_dp")
        assert result.replicated_axes == frozenset()
```
**EN:** This test method exercises hash without replicated and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 hash without replicated 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 230-232: Run test: replicated conflicts with sharded raises
```python
    def test_replicated_conflicts_with_sharded_raises(self) -> None:
        with pytest.raises(ValueError, match="both sharded.*and replicated"):
            parse_dims("b s h[tp] d # tp:replicated")
```
**EN:** This test method exercises replicated conflicts with sharded raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 replicated conflicts with sharded raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 234-236: Run test: unknown axis in replicated raises
```python
    def test_unknown_axis_in_replicated_raises(self) -> None:
        with pytest.raises(ValueError, match="Unknown axis"):
            parse_dims("b s h[tp] d # xyz:replicated")
```
**EN:** This test method exercises unknown axis in replicated raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 unknown axis in replicated raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 238-240: Run test: duplicate replicated declaration raises
```python
    def test_duplicate_replicated_declaration_raises(self) -> None:
        with pytest.raises(ValueError, match="Duplicate replicated"):
            parse_dims("b s h d # ep:replicated ep:replicated")
```
**EN:** This test method exercises duplicate replicated declaration raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 duplicate replicated declaration raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 242-244: Run test: unrecognized token in comment raises
```python
    def test_unrecognized_token_in_comment_raises(self) -> None:
        with pytest.raises(ValueError, match="Unrecognized token"):
            parse_dims("b s h[tp] d # ep:replicatd")
```
**EN:** This test method exercises unrecognized token in comment raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 unrecognized token in comment raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 246-248: Run test: duplicate dp alias raises
```python
    def test_duplicate_dp_alias_raises(self) -> None:
        with pytest.raises(ValueError, match="Duplicate dp alias"):
            parse_dims("b s h d # dp:=foo dp:=bar")
```
**EN:** This test method exercises duplicate dp alias raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 duplicate dp alias raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 251-251: Define class TestResolveDimNames
```python
class TestResolveDimNames:
```
**EN:** This declaration introduces the `TestResolveDimNames` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestResolveDimNames` 测试类，并说明它通过继承承担的职责。

### Lines 252-253: Run test: no squeeze
```python
    def test_no_squeeze(self) -> None:
        assert resolve_dim_names("t h d") == ["t", "h", "d"]
```
**EN:** This test method exercises no squeeze and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no squeeze 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 255-256: Run test: single squeeze
```python
    def test_single_squeeze(self) -> None:
        assert resolve_dim_names("t 1 h") == ["t", "singleton0", "h"]
```
**EN:** This test method exercises single squeeze and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single squeeze 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 258-264: Run test: multiple squeeze
```python
    def test_multiple_squeeze(self) -> None:
        assert resolve_dim_names("1 t 1 h") == [
            "singleton0",
            "t",
            "singleton1",
            "h",
        ]
```
**EN:** This test method exercises multiple squeeze and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multiple squeeze 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 267-267: Define class TestResolveDimNamesWithFused
```python
class TestResolveDimNamesWithFused:
```
**EN:** This declaration introduces the `TestResolveDimNamesWithFused` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestResolveDimNamesWithFused` 测试类，并说明它通过继承承担的职责。

### Lines 268-272: Run test: fused dim uses triple underscore
```python
    def test_fused_dim_uses_triple_underscore(self) -> None:
        assert resolve_dim_names("t (num_heads*head_dim)") == [
            "t",
            "num_heads___head_dim",
        ]
```
**EN:** This test method exercises fused dim uses triple underscore and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fused dim uses triple underscore 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 274-279: Run test: fused with regular dims
```python
    def test_fused_with_regular_dims(self) -> None:
        assert resolve_dim_names("t (num_heads*head_dim)[tp] d") == [
            "t",
            "num_heads___head_dim",
            "d",
        ]
```
**EN:** This test method exercises fused with regular dims and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fused with regular dims 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 281-282: Run test: three way fused
```python
    def test_three_way_fused(self) -> None:
        assert resolve_dim_names("(a*b*c)") == ["a___b___c"]
```
**EN:** This test method exercises three way fused and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 three way fused 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 284-285: Run test: fused with squeeze
```python
    def test_fused_with_squeeze(self) -> None:
        assert resolve_dim_names("t 1 (a*b)") == ["t", "singleton0", "a___b"]
```
**EN:** This test method exercises fused with squeeze and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fused with squeeze 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 288-288: Define class TestResolveDimNamesWithHash
```python
class TestResolveDimNamesWithHash:
```
**EN:** This declaration introduces the `TestResolveDimNamesWithHash` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestResolveDimNamesWithHash` 测试类，并说明它通过继承承担的职责。

### Lines 289-290: Run test: hash stripped
```python
    def test_hash_stripped(self) -> None:
        assert resolve_dim_names("t h # dp:=moe_dp") == ["t", "h"]
```
**EN:** This test method exercises hash stripped and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 hash stripped 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 293-294: Expose unittest entrypoint
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
