# test_tensor_naming.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/dims_spec/test_tensor_naming.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on dims spec tensor naming in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 dims spec tensor naming 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Import dependencies
```python
import sys

import pytest
import torch

from sglang.srt.debug_utils.comparator.dims_spec import (
    DimSpec,
    apply_dim_names,
    find_dim_index,
    parse_dims,
    resolve_dim_by_name,
    strip_dim_names,
)
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 16-16: Register CI metadata
```python
register_cpu_ci(est_time=5, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 19-19: Define class TestFindDimIndex
```python
class TestFindDimIndex:
```
**EN:** This declaration introduces the `TestFindDimIndex` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFindDimIndex` 测试类，并说明它通过继承承担的职责。

### Lines 20-22: Run test: found
```python
    def test_found(self) -> None:
        specs: list[DimSpec] = parse_dims("b s h d").dims
        assert find_dim_index(specs, "s") == 1
```
**EN:** This test method exercises found and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 found 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 24-26: Run test: not found
```python
    def test_not_found(self) -> None:
        specs: list[DimSpec] = parse_dims("b s h d").dims
        assert find_dim_index(specs, "t") is None
```
**EN:** This test method exercises not found and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 not found 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 28-30: Run test: first dim
```python
    def test_first_dim(self) -> None:
        specs: list[DimSpec] = parse_dims("t h d").dims
        assert find_dim_index(specs, "t") == 0
```
**EN:** This test method exercises first dim and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 first dim 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 32-34: Run test: last dim
```python
    def test_last_dim(self) -> None:
        specs: list[DimSpec] = parse_dims("b s h d").dims
        assert find_dim_index(specs, "d") == 3
```
**EN:** This test method exercises last dim and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 last dim 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 36-38: Run test: with modifiers
```python
    def test_with_modifiers(self) -> None:
        specs: list[DimSpec] = parse_dims("b s[cp:zigzag] h[tp] d").dims
        assert find_dim_index(specs, "h") == 2
```
**EN:** This test method exercises with modifiers and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with modifiers 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 40-41: Run test: empty list
```python
    def test_empty_list(self) -> None:
        assert find_dim_index([], "t") is None
```
**EN:** This test method exercises empty list and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty list 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 44-44: Define class TestResolveDimByName
```python
class TestResolveDimByName:
```
**EN:** This declaration introduces the `TestResolveDimByName` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestResolveDimByName` 测试类，并说明它通过继承承担的职责。

### Lines 45-49: Run test: resolve found
```python
    def test_resolve_found(self) -> None:
        tensor: torch.Tensor = torch.randn(2, 3, 4).refine_names("b", "s", "h")
        assert resolve_dim_by_name(tensor, "b") == 0
        assert resolve_dim_by_name(tensor, "s") == 1
        assert resolve_dim_by_name(tensor, "h") == 2
```
**EN:** This test method exercises resolve found and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 resolve found 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 51-54: Run test: resolve not found raises
```python
    def test_resolve_not_found_raises(self) -> None:
        tensor: torch.Tensor = torch.randn(2, 3).refine_names("b", "s")
        with pytest.raises(ValueError, match="not in tensor names"):
            resolve_dim_by_name(tensor, "h")
```
**EN:** This test method exercises resolve not found raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 resolve not found raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 56-59: Run test: resolve unnamed raises
```python
    def test_resolve_unnamed_raises(self) -> None:
        tensor: torch.Tensor = torch.randn(2, 3)
        with pytest.raises(ValueError, match="no names"):
            resolve_dim_by_name(tensor, "b")
```
**EN:** This test method exercises resolve unnamed raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 resolve unnamed raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 62-62: Define class TestApplyDimNames
```python
class TestApplyDimNames:
```
**EN:** This declaration introduces the `TestApplyDimNames` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestApplyDimNames` 测试类，并说明它通过继承承担的职责。

### Lines 63-67: Run test: apply
```python
    def test_apply(self) -> None:
        tensor: torch.Tensor = torch.randn(2, 3, 4)
        named: torch.Tensor = apply_dim_names(tensor, ["b", "s", "h"])
        assert named.names == ("b", "s", "h")
        assert named.shape == (2, 3, 4)
```
**EN:** This test method exercises apply and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 apply 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 69-72: Run test: apply preserves data
```python
    def test_apply_preserves_data(self) -> None:
        tensor: torch.Tensor = torch.randn(2, 3)
        named: torch.Tensor = apply_dim_names(tensor, ["x", "y"])
        assert torch.equal(strip_dim_names(named), tensor)
```
**EN:** This test method exercises apply preserves data and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 apply preserves data 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 74-80: Run test: ndim mismatch gives clear error
```python
    def test_ndim_mismatch_gives_clear_error(self) -> None:
        tensor: torch.Tensor = torch.randn(10, 1, 128)
        with pytest.raises(
            ValueError,
            match=r"dims metadata mismatch.*3 dims.*shape \[10, 1, 128\].*2 names \['t', 'num_experts'\].*fix the dims string",
        ):
            apply_dim_names(tensor, ["t", "num_experts"])
```
**EN:** This test method exercises ndim mismatch gives clear error and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 ndim mismatch gives clear error 场景，并验证观测到的行为是否符合预期契约。

### Lines 83-83: Define class TestStripDimNames
```python
class TestStripDimNames:
```
**EN:** This declaration introduces the `TestStripDimNames` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestStripDimNames` 测试类，并说明它通过继承承担的职责。

### Lines 84-87: Run test: strip
```python
    def test_strip(self) -> None:
        tensor: torch.Tensor = torch.randn(2, 3).refine_names("a", "b")
        stripped: torch.Tensor = strip_dim_names(tensor)
        assert stripped.names == (None, None)
```
**EN:** This test method exercises strip and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 strip 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 89-92: Run test: strip already unnamed
```python
    def test_strip_already_unnamed(self) -> None:
        tensor: torch.Tensor = torch.randn(2, 3)
        stripped: torch.Tensor = strip_dim_names(tensor)
        assert stripped.names == (None, None)
```
**EN:** This test method exercises strip already unnamed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 strip already unnamed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 95-96: Expose unittest entrypoint
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
- External and stdlib modules / 外部与标准库模块: `pytest`, `sys`, `torch`
- Notable symbols / 关键符号: None / 无
