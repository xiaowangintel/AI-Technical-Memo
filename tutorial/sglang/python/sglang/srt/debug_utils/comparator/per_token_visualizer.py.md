# per_token_visualizer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/per_token_visualizer.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on comparison result visualization. It mainly turns raw comparison state into readable debug output. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于比较结果可视化。它主要用于把原始比较状态转换为可读的调试输出。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: Document the module intent / 说明模块意图
```python
"""Per-token relative difference heatmap generator.

Produces a single PNG with rows = tensor names, columns = token positions,
color = log10(rel_diff).
"""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 7-12: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

from pathlib import Path
from typing import Optional

from sglang.srt.debug_utils.comparator.output_types import ComparisonTensorRecord
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 15-29: Implement function `generate_per_token_heatmap` / 实现函数 `generate_per_token_heatmap`
```python
def generate_per_token_heatmap(
    *,
    records: list[ComparisonTensorRecord],
    output_path: Path,
) -> Optional[Path]:
    """Generate a per-token relative difference heatmap PNG.

    Returns the output path if a file was written, or None if no data was available.
    """
    rows_data: list[tuple[str, list[float]]] = _collect_per_token_data(records=records)
    if not rows_data:
        return None

    _render_heatmap(rows_data=rows_data, output_path=output_path)
    return output_path
```
**EN:** Function `generate_per_token_heatmap` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `generate_per_token_heatmap` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 32-41: Implement helper `_collect_per_token_data` / 实现辅助函数 `_collect_per_token_data`
```python
def _collect_per_token_data(
    *,
    records: list[ComparisonTensorRecord],
) -> list[tuple[str, list[float]]]:
    rows: list[tuple[str, list[float]]] = []
    for record in records:
        if record.diff is None or record.diff.per_token_rel_diff is None:
            continue
        rows.append((record.name, record.diff.per_token_rel_diff))
    return rows
```
**EN:** Function `_collect_per_token_data` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_collect_per_token_data` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 44-83: Implement helper `_render_heatmap` / 实现辅助函数 `_render_heatmap`
```python
def _render_heatmap(
    *,
    rows_data: list[tuple[str, list[float]]],
    output_path: Path,
) -> None:
    import matplotlib
    import numpy as np

    matplotlib.use("Agg")
    import matplotlib.pyplot as plt

    max_len: int = max(len(vals) for _, vals in rows_data)
    labels: list[str] = [label for label, _ in rows_data]

    matrix: np.ndarray = np.full((len(rows_data), max_len), np.nan, dtype=np.float64)
    for i, (_, vals) in enumerate(rows_data):
        matrix[i, : len(vals)] = vals

    fig_width: float = max(12.0, max_len * 0.15)
    fig_height: float = max(6.0, len(rows_data) * 0.3)
    fig, ax = plt.subplots(figsize=(fig_width, fig_height))

    im = ax.imshow(
        np.log10(matrix + 1e-10), aspect="auto", cmap="hot", interpolation="nearest"
    )

    ax.set_xlabel("Token Position")
    ax.set_ylabel("Tensor")
    ax.set_yticks(range(len(labels)))
    ax.set_yticklabels(labels, fontsize=8)

    colorbar = fig.colorbar(im, ax=ax)
    colorbar.set_label("log10(rel_diff)")

    ax.set_title("Per-Token Relative Difference Heatmap")
    fig.tight_layout()

    output_path.parent.mkdir(parents=True, exist_ok=True)
    fig.savefig(str(output_path), dpi=150)
    plt.close(fig)
```
**EN:** Function `_render_heatmap` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_render_heatmap` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `generate_per_token_heatmap`, `_collect_per_token_data`, `_render_heatmap`
- **Module role / 模块角色**: Comparison result visualization / 比较结果可视化
- **Implementation focus / 实现重点**: Turns raw comparison state into readable debug output / 把原始比较状态转换为可读的调试输出

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `pathlib`, `typing`
- **Third-party / 第三方**: `matplotlib`, `numpy`
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.output_types`
