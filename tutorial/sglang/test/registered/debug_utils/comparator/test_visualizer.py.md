# test_visualizer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/test_visualizer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on comparator visualizer in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 comparator visualizer 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Import dependencies
```python
import sys
from pathlib import Path

import pytest
import torch

from sglang.srt.debug_utils.comparator.visualizer.preprocessing import (
    _preprocess_tensor,
    _reshape_to_balanced_aspect,
)
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 13-13: Register CI metadata
```python
register_cpu_ci(est_time=30, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 16-16: Define class TestPreprocessTensor
```python
class TestPreprocessTensor:
```
**EN:** This declaration introduces the `TestPreprocessTensor` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestPreprocessTensor` 测试类，并说明它通过继承承担的职责。

### Lines 17-20: Run test: 1d becomes 2d
```python
    def test_1d_becomes_2d(self) -> None:
        t: torch.Tensor = torch.randn(100)
        result: torch.Tensor = _preprocess_tensor(t)
        assert result.ndim == 2
```
**EN:** This test method exercises 1d becomes 2d and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 1d becomes 2d 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 22-26: Run test: 3d becomes 2d
```python
    def test_3d_becomes_2d(self) -> None:
        t: torch.Tensor = torch.randn(2, 3, 4)
        result: torch.Tensor = _preprocess_tensor(t)
        assert result.ndim == 2
        assert result.numel() == t.numel()
```
**EN:** This test method exercises 3d becomes 2d and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 3d becomes 2d 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 28-32: Run test: high dim becomes 2d
```python
    def test_high_dim_becomes_2d(self) -> None:
        t: torch.Tensor = torch.randn(2, 3, 4, 5)
        result: torch.Tensor = _preprocess_tensor(t)
        assert result.ndim == 2
        assert result.numel() == t.numel()
```
**EN:** This test method exercises high dim becomes 2d and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 high dim becomes 2d 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 34-38: Run test: scalar becomes 2d
```python
    def test_scalar_becomes_2d(self) -> None:
        t: torch.Tensor = torch.tensor(3.14)
        result: torch.Tensor = _preprocess_tensor(t)
        assert result.ndim == 2
        assert result.numel() == 1
```
**EN:** This test method exercises scalar becomes 2d and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 scalar becomes 2d 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 40-44: Run test: already 2d preserves elements
```python
    def test_already_2d_preserves_elements(self) -> None:
        t: torch.Tensor = torch.randn(10, 20)
        result: torch.Tensor = _preprocess_tensor(t)
        assert result.ndim == 2
        assert result.numel() == 200
```
**EN:** This test method exercises already 2d preserves elements and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 already 2d preserves elements 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 47-47: Define class TestReshapeToBalancedAspect
```python
class TestReshapeToBalancedAspect:
```
**EN:** This declaration introduces the `TestReshapeToBalancedAspect` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestReshapeToBalancedAspect` 测试类，并说明它通过继承承担的职责。

### Lines 48-53: Run test: extreme wide gets fixed
```python
    def test_extreme_wide_gets_fixed(self) -> None:
        t: torch.Tensor = torch.randn(1, 10000)
        result: torch.Tensor = _reshape_to_balanced_aspect(t)
        h, w = result.shape
        ratio: float = max(h, w) / max(min(h, w), 1)
        assert ratio <= 5.0
```
**EN:** This test method exercises extreme wide gets fixed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 extreme wide gets fixed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 55-60: Run test: extreme tall gets fixed
```python
    def test_extreme_tall_gets_fixed(self) -> None:
        t: torch.Tensor = torch.randn(10000, 1)
        result: torch.Tensor = _reshape_to_balanced_aspect(t)
        h, w = result.shape
        ratio: float = max(h, w) / max(min(h, w), 1)
        assert ratio <= 5.0
```
**EN:** This test method exercises extreme tall gets fixed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 extreme tall gets fixed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 62-65: Run test: already balanced unchanged
```python
    def test_already_balanced_unchanged(self) -> None:
        t: torch.Tensor = torch.randn(100, 100)
        result: torch.Tensor = _reshape_to_balanced_aspect(t)
        assert result.shape == (100, 100)
```
**EN:** This test method exercises already balanced unchanged and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 already balanced unchanged 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 67-70: Run test: preserves numel
```python
    def test_preserves_numel(self) -> None:
        t: torch.Tensor = torch.randn(1, 7919)
        result: torch.Tensor = _reshape_to_balanced_aspect(t)
        assert result.numel() == t.numel()
```
**EN:** This test method exercises preserves numel and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 preserves numel 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 73-73: Define class TestGenerateComparisonFigure
```python
class TestGenerateComparisonFigure:
```
**EN:** This declaration introduces the `TestGenerateComparisonFigure` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestGenerateComparisonFigure` 测试类，并说明它通过继承承担的职责。

### Lines 74-76: Define helper: skip if no matplotlib
```python
    @pytest.fixture(autouse=True)
    def _skip_if_no_matplotlib(self) -> None:
        pytest.importorskip("matplotlib")
```
**EN:** This helper function encapsulates reusable logic inside `TestGenerateComparisonFigure` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGenerateComparisonFigure` 内部调用，从而让场景结构更清晰。

### Lines 78-92: Run test: nested output dir
```python
    def test_nested_output_dir(self, tmp_path: Path) -> None:
        from sglang.srt.debug_utils.comparator.visualizer import (
            generate_comparison_figure,
        )

        output_path: Path = tmp_path / "a" / "b" / "c" / "nested.png"

        generate_comparison_figure(
            baseline=torch.randn(10, 10),
            target=torch.randn(10, 10),
            name="nested",
            output_path=output_path,
        )

        assert output_path.exists()
```
**EN:** This test method exercises nested output dir and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 nested output dir 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.visualizer`, `sglang.srt.debug_utils.comparator.visualizer.preprocessing`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pathlib`, `pytest`, `sys`, `torch`
- Notable symbols / 关键符号: None / 无
