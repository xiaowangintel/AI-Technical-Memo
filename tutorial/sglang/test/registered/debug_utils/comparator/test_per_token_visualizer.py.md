# test_per_token_visualizer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/test_per_token_visualizer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on comparator per token visualizer in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 comparator per token visualizer 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Document the module
```python
"""Layer 2: PNG generation tests for per-token heatmap visualizer.

Requires matplotlib — uses pytest.importorskip to gracefully skip if absent.
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 6-16: Import dependencies
```python
import sys
from pathlib import Path

import pytest
import torch

from sglang.srt.debug_utils.comparator.output_types import ComparisonTensorRecord
from sglang.srt.debug_utils.comparator.tensor_comparator.comparator import (
    compare_tensor_pair,
)
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 18-18: Register CI metadata
```python
register_cpu_ci(est_time=30, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 20-20: Define module constants
```python
_PNG_MAGIC: bytes = b"\x89PNG"
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 23-25: Define helper: skip if no matplotlib
```python
@pytest.fixture(autouse=True)
def _skip_if_no_matplotlib() -> None:
    pytest.importorskip("matplotlib")
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 28-43: Define helper: make comparison record
```python
def _make_comparison_record(
    *,
    name: str,
    baseline: torch.Tensor,
    target: torch.Tensor,
    seq_dim: int = 0,
) -> ComparisonTensorRecord:
    """Build a ComparisonTensorRecord with per-token data from raw tensors."""
    info = compare_tensor_pair(
        x_baseline=baseline,
        x_target=target,
        name=name,
        diff_threshold=1e-3,
        seq_dim=seq_dim,
    )
    return ComparisonTensorRecord(**info.model_dump())
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 46-46: Define class TestPerTokenVisualizer
```python
class TestPerTokenVisualizer:
```
**EN:** This declaration introduces the `TestPerTokenVisualizer` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestPerTokenVisualizer` 测试类，并说明它通过继承承担的职责。

### Lines 47-57: Run test: no data returns none
```python
    def test_no_data_returns_none(self, tmp_path: Path) -> None:
        """Empty records list → None returned, no file created."""
        from sglang.srt.debug_utils.comparator.per_token_visualizer import (
            generate_per_token_heatmap,
        )

        output_path: Path = tmp_path / "empty.png"
        result = generate_per_token_heatmap(records=[], output_path=output_path)

        assert result is None
        assert not output_path.exists()
```
**EN:** This test method exercises no data returns none and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no data returns none 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 59-76: Run test: no per token data returns none
```python
    def test_no_per_token_data_returns_none(self, tmp_path: Path) -> None:
        """Records without per_token_rel_diff → None."""
        from sglang.srt.debug_utils.comparator.per_token_visualizer import (
            generate_per_token_heatmap,
        )

        info = compare_tensor_pair(
            x_baseline=torch.randn(4, 8),
            x_target=torch.randn(4, 8),
            name="no_per_token",
            diff_threshold=1e-3,
        )
        record = ComparisonTensorRecord(**info.model_dump())

        output_path: Path = tmp_path / "no_data.png"
        result = generate_per_token_heatmap(records=[record], output_path=output_path)

        assert result is None
```
**EN:** This test method exercises no per token data returns none and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no per token data returns none 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 78-102: Run test: generates valid png
```python
    def test_generates_valid_png(self, tmp_path: Path) -> None:
        """Records with per-token data → valid PNG file."""
        from sglang.srt.debug_utils.comparator.per_token_visualizer import (
            generate_per_token_heatmap,
        )

        torch.manual_seed(42)
        records: list[ComparisonTensorRecord] = [
            _make_comparison_record(
                name=f"tensor_{i}",
                baseline=torch.randn(16, 32),
                target=torch.randn(16, 32),
            )
            for i in range(3)
        ]

        output_path: Path = tmp_path / "heatmap.png"
        result = generate_per_token_heatmap(records=records, output_path=output_path)

        assert result == output_path
        assert output_path.exists()
        assert output_path.stat().st_size > 0
        with open(output_path, "rb") as f:
            magic: bytes = f.read(4)
        assert magic == _PNG_MAGIC
```
**EN:** This test method exercises generates valid png and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 generates valid png 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 104-136: Run test: variable length sequences
```python
    def test_variable_length_sequences(self, tmp_path: Path) -> None:
        """Records with different token lengths → NaN padding, no crash."""
        from sglang.srt.debug_utils.comparator.per_token_visualizer import (
            generate_per_token_heatmap,
        )

        torch.manual_seed(42)
        records: list[ComparisonTensorRecord] = [
            _make_comparison_record(
                name="short",
                baseline=torch.randn(4, 8),
                target=torch.randn(4, 8),
            ),
            _make_comparison_record(
                name="medium",
                baseline=torch.randn(16, 8),
                target=torch.randn(16, 8),
            ),
            _make_comparison_record(
                name="long",
                baseline=torch.randn(64, 8),
                target=torch.randn(64, 8),
            ),
        ]

        output_path: Path = tmp_path / "variable.png"
        result = generate_per_token_heatmap(records=records, output_path=output_path)

        assert result == output_path
        assert output_path.exists()
        with open(output_path, "rb") as f:
            magic: bytes = f.read(4)
        assert magic == _PNG_MAGIC
```
**EN:** This test method exercises variable length sequences and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 variable length sequences 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 138-155: Run test: creates parent dirs
```python
    def test_creates_parent_dirs(self, tmp_path: Path) -> None:
        """Output path with non-existent parent dirs → dirs created automatically."""
        from sglang.srt.debug_utils.comparator.per_token_visualizer import (
            generate_per_token_heatmap,
        )

        torch.manual_seed(42)
        record = _make_comparison_record(
            name="test",
            baseline=torch.randn(8, 16),
            target=torch.randn(8, 16),
        )

        output_path: Path = tmp_path / "nested" / "deep" / "heatmap.png"
        result = generate_per_token_heatmap(records=[record], output_path=output_path)

        assert result == output_path
        assert output_path.exists()
```
**EN:** This test method exercises creates parent dirs and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 creates parent dirs 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 158-159: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.output_types`, `sglang.srt.debug_utils.comparator.per_token_visualizer`, `sglang.srt.debug_utils.comparator.tensor_comparator.comparator`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pathlib`, `pytest`, `sys`, `torch`
- Notable symbols / 关键符号: None / 无
