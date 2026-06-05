# test_manually_verify.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/test_manually_verify.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on comparator manually verify in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 comparator manually verify 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Document the module
```python
"""Visual comparison figure tests — CI sanity check + human verification.

This file serves two purposes:
1. CI sanity check: ensures generate_comparison_figure() runs without errors
   across various tensor scenarios (registered via register_cpu_ci).
2. Human verification: all generated PNGs are copied to /tmp/comparator_manual_verify/
   so they can be pulled back to a local machine for visual inspection.

Run:
    python -m pytest test/registered/debug_utils/comparator/test_manually_verify.py -x -v

Human verification:
    After running, images are at /tmp/comparator_manual_verify/.
    Each test's docstring describes the expected visual appearance.
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 17-24: Import dependencies
```python
import shutil
import sys
from pathlib import Path

import pytest
import torch

from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 26-26: Register CI metadata
```python
register_cpu_ci(est_time=60, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 28-29: Define module constants
```python
_PUBLISH_DIR: Path = Path("/tmp/comparator_manual_verify")
_PNG_MAGIC: bytes = b"\x89PNG"
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 32-38: Define helper: publish dir
```python
@pytest.fixture(scope="session")
def publish_dir() -> Path:
    """Fixed output dir for human inspection — files are copied here after generation."""
    if _PUBLISH_DIR.exists():
        shutil.rmtree(_PUBLISH_DIR)
    _PUBLISH_DIR.mkdir(parents=True)
    return _PUBLISH_DIR
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 41-46: Define helper: assert valid png
```python
def _assert_valid_png(path: Path) -> None:
    assert path.exists(), f"PNG not created: {path}"
    assert path.stat().st_size > 0, f"PNG is empty: {path}"
    with open(path, "rb") as f:
        magic: bytes = f.read(4)
    assert magic == _PNG_MAGIC, f"Not a valid PNG: {path}"
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 49-71: Define helper: generate and publish
```python
def _generate_and_publish(
    *,
    baseline: torch.Tensor,
    target: torch.Tensor,
    name: str,
    tmp_path: Path,
    publish_dir: Path,
) -> Path:
    from sglang.srt.debug_utils.comparator.visualizer import (
        generate_comparison_figure,
    )

    output_path: Path = tmp_path / f"{name}.png"
    generate_comparison_figure(
        baseline=baseline,
        target=target,
        name=name,
        output_path=output_path,
    )

    _assert_valid_png(output_path)
    shutil.copy2(src=output_path, dst=publish_dir / output_path.name)
    return output_path
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 74-76: Define helper: skip if no matplotlib
```python
@pytest.fixture(autouse=True)
def _skip_if_no_matplotlib() -> None:
    pytest.importorskip("matplotlib")
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 79-79: Define class TestBundleDetailsManualVerify
```python
class TestBundleDetailsManualVerify:
```
**EN:** This declaration introduces the `TestBundleDetailsManualVerify` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestBundleDetailsManualVerify` 测试类，并说明它通过继承承担的职责。

### Lines 80-95: Run test: normal small diff
```python
    def test_normal_small_diff(self, tmp_path: Path, publish_dir: Path) -> None:
        """Two nearly-identical tensors (randn + 0.01 noise).

        Expected: All 6 panel rows visible. Diff heatmap nearly uniform light color.
        Hist2d tightly clustered along the red diagonal line.
        """
        baseline: torch.Tensor = torch.randn(32, 64)
        target: torch.Tensor = baseline + torch.randn(32, 64) * 0.01

        _generate_and_publish(
            baseline=baseline,
            target=target,
            name="normal_small_diff",
            tmp_path=tmp_path,
            publish_dir=publish_dir,
        )
```
**EN:** This test method exercises normal small diff and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 normal small diff 场景，并验证观测到的行为是否符合预期契约。

### Lines 97-112: Run test: significant diff
```python
    def test_significant_diff(self, tmp_path: Path, publish_dir: Path) -> None:
        """Two tensors with larger differences (randn + 0.5 noise).

        Expected: All 6 panel rows visible. Diff heatmap shows noticeable structure.
        Hist2d scatter is broader, spread away from the diagonal.
        """
        baseline: torch.Tensor = torch.randn(32, 64)
        target: torch.Tensor = baseline + torch.randn(32, 64) * 0.5

        _generate_and_publish(
            baseline=baseline,
            target=target,
            name="significant_diff",
            tmp_path=tmp_path,
            publish_dir=publish_dir,
        )
```
**EN:** This test method exercises significant diff and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 significant diff 场景，并验证观测到的行为是否符合预期契约。

### Lines 114-129: Run test: shape mismatch
```python
    def test_shape_mismatch(self, tmp_path: Path, publish_dir: Path) -> None:
        """Baseline 32x64, target 16x32 — shapes do not match.

        Expected: Only 2 panel rows (baseline heatmap, target heatmap).
        No diff/histogram/hist2d/sampled panels since diff cannot be computed.
        """
        baseline: torch.Tensor = torch.randn(32, 64)
        target: torch.Tensor = torch.randn(16, 32)

        _generate_and_publish(
            baseline=baseline,
            target=target,
            name="shape_mismatch",
            tmp_path=tmp_path,
            publish_dir=publish_dir,
        )
```
**EN:** This test method exercises shape mismatch and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 shape mismatch 场景，并验证观测到的行为是否符合预期契约。

### Lines 131-146: Run test: large tensor
```python
    def test_large_tensor(self, tmp_path: Path, publish_dir: Path) -> None:
        """4000x4000 tensor — triggers internal downsampling.

        Expected: Figure renders normally without OOM. Downsampled panels
        should still look reasonable.
        """
        baseline: torch.Tensor = torch.randn(4000, 4000)
        target: torch.Tensor = baseline + torch.randn(4000, 4000) * 0.001

        _generate_and_publish(
            baseline=baseline,
            target=target,
            name="large_tensor",
            tmp_path=tmp_path,
            publish_dir=publish_dir,
        )
```
**EN:** This test method exercises large tensor and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 large tensor 场景，并验证观测到的行为是否符合预期契约。

### Lines 148-163: Run test: 1d tensor
```python
    def test_1d_tensor(self, tmp_path: Path, publish_dir: Path) -> None:
        """1D tensor (256,) — internally reshaped to 2D before plotting.

        Expected: All 6 panel rows visible. The heatmap shape reflects the
        reshaped 2D form, not the original 1D.
        """
        baseline: torch.Tensor = torch.randn(256)
        target: torch.Tensor = baseline + 0.01

        _generate_and_publish(
            baseline=baseline,
            target=target,
            name="1d_tensor",
            tmp_path=tmp_path,
            publish_dir=publish_dir,
        )
```
**EN:** This test method exercises 1d tensor and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 1d tensor 场景，并验证观测到的行为是否符合预期契约。

### Lines 165-180: Run test: constant tensor
```python
    def test_constant_tensor(self, tmp_path: Path, publish_dir: Path) -> None:
        """All-zero baseline, tiny-valued target.

        Expected: Colorbar range is extremely small. Histogram concentrates in
        a single bin. No rendering errors from near-zero variance.
        """
        baseline: torch.Tensor = torch.zeros(32, 64)
        target: torch.Tensor = torch.ones(32, 64) * 1e-8

        _generate_and_publish(
            baseline=baseline,
            target=target,
            name="constant_tensor",
            tmp_path=tmp_path,
            publish_dir=publish_dir,
        )
```
**EN:** This test method exercises constant tensor and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 constant tensor 场景，并验证观测到的行为是否符合预期契约。

### Lines 182-199: Run test: extreme values
```python
    def test_extreme_values(self, tmp_path: Path, publish_dir: Path) -> None:
        """Tensor containing values spanning 1e-10 to 1e10.

        Expected: Log10 panels handle the wide range gracefully. No inf/nan
        artifacts in the rendered figure.
        """
        baseline: torch.Tensor = torch.randn(32, 64).abs()
        baseline[0, 0] = 1e-10
        baseline[0, 1] = 1e10
        target: torch.Tensor = baseline + torch.randn(32, 64) * 0.01

        _generate_and_publish(
            baseline=baseline,
            target=target,
            name="extreme_values",
            tmp_path=tmp_path,
            publish_dir=publish_dir,
        )
```
**EN:** This test method exercises extreme values and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 extreme values 场景，并验证观测到的行为是否符合预期契约。

### Lines 202-202: Define class TestPerTokenHeatmapManualVerify
```python
class TestPerTokenHeatmapManualVerify:
```
**EN:** This declaration introduces the `TestPerTokenHeatmapManualVerify` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestPerTokenHeatmapManualVerify` 测试类，并说明它通过继承承担的职责。

### Lines 203-247: Run test: increasing diff
```python
    def test_increasing_diff(self, tmp_path: Path, publish_dir: Path) -> None:
        """Per-token heatmap with linearly increasing diff across token positions.

        Expected: Heatmap shows a clear left-to-right gradient — dark/cold on
        the left (small diff), bright/hot on the right (large diff). Multiple
        rows for different tensor names. Colorbar shows log10 scale.
        """
        from sglang.srt.debug_utils.comparator.output_types import (
            ComparisonTensorRecord,
        )
        from sglang.srt.debug_utils.comparator.per_token_visualizer import (
            generate_per_token_heatmap,
        )
        from sglang.srt.debug_utils.comparator.tensor_comparator.comparator import (
            compare_tensor_pair,
        )

        torch.manual_seed(42)
        seq_len: int = 64
        hidden_dim: int = 128
        num_tensors: int = 5

        records: list[ComparisonTensorRecord] = []
        for i in range(num_tensors):
            baseline: torch.Tensor = torch.randn(seq_len, hidden_dim)
            noise_scale: torch.Tensor = torch.linspace(
                1e-6, 0.5, steps=seq_len
            ).unsqueeze(1)
            target: torch.Tensor = baseline + torch.randn_like(baseline) * noise_scale

            info = compare_tensor_pair(
                x_baseline=baseline,
                x_target=target,
                name=f"layer_{i}_hidden_states",
                diff_threshold=1e-3,
                seq_dim=0,
            )
            records.append(ComparisonTensorRecord(**info.model_dump()))

        output_path: Path = tmp_path / "per_token_increasing_diff.png"
        result = generate_per_token_heatmap(records=records, output_path=output_path)

        assert result is not None
        _assert_valid_png(output_path)
        shutil.copy2(src=output_path, dst=publish_dir / output_path.name)
```
**EN:** This test method exercises increasing diff and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 increasing diff 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 249-291: Run test: single spike
```python
    def test_single_spike(self, tmp_path: Path, publish_dir: Path) -> None:
        """Per-token heatmap where only one token position has large diff.

        Expected: Heatmap shows one bright vertical stripe at the spike position,
        rest is dark/cold.
        """
        from sglang.srt.debug_utils.comparator.output_types import (
            ComparisonTensorRecord,
        )
        from sglang.srt.debug_utils.comparator.per_token_visualizer import (
            generate_per_token_heatmap,
        )
        from sglang.srt.debug_utils.comparator.tensor_comparator.comparator import (
            compare_tensor_pair,
        )

        torch.manual_seed(42)
        seq_len: int = 64
        hidden_dim: int = 128
        spike_pos: int = 32
        num_tensors: int = 4

        records: list[ComparisonTensorRecord] = []
        for i in range(num_tensors):
            baseline: torch.Tensor = torch.randn(seq_len, hidden_dim)
            target: torch.Tensor = baseline.clone()
            target[spike_pos, :] += torch.randn(hidden_dim) * 5.0

            info = compare_tensor_pair(
                x_baseline=baseline,
                x_target=target,
                name=f"layer_{i}_attn_output",
                diff_threshold=1e-3,
                seq_dim=0,
            )
            records.append(ComparisonTensorRecord(**info.model_dump()))

        output_path: Path = tmp_path / "per_token_single_spike.png"
        result = generate_per_token_heatmap(records=records, output_path=output_path)

        assert result is not None
        _assert_valid_png(output_path)
        shutil.copy2(src=output_path, dst=publish_dir / output_path.name)
```
**EN:** This test method exercises single spike and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single spike 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 294-295: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.output_types`, `sglang.srt.debug_utils.comparator.per_token_visualizer`, `sglang.srt.debug_utils.comparator.tensor_comparator.comparator`, `sglang.srt.debug_utils.comparator.visualizer`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pathlib`, `pytest`, `shutil`, `sys`, `torch`
- Notable symbols / 关键符号: None / 无
