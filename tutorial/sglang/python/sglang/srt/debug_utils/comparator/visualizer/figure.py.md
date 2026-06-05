# figure.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/visualizer/figure.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on comparison result visualization. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于比较结果可视化。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Document the module intent / 说明模块意图
```python
"""Main orchestration logic for comparison figure generation."""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 3-14: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

from dataclasses import dataclass
from pathlib import Path
from typing import Callable, Optional

import numpy as np
import torch

from sglang.srt.debug_utils.comparator.visualizer.preprocessing import (
    _preprocess_tensor,
)
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 18-18: Define class `_PanelContext` and class context / 定义类 `_PanelContext`及类上下文
```python
class _PanelContext:
```
**EN:** This section introduces `_PanelContext`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_PanelContext`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 19-22: Declare fields for `_PanelContext` such as `baseline_2d`, `target_2d`, `diff`, `name` / 为 `_PanelContext` 声明字段，例如 `baseline_2d`, `target_2d`, `diff`, `name`
```python
    baseline_2d: torch.Tensor
    target_2d: torch.Tensor
    diff: Optional[torch.Tensor]  # None when shapes differ
    name: str
```
**EN:** These lines declare the state carried by `_PanelContext`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `_PanelContext` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 26-26: Define class `_Panel` and class context / 定义类 `_Panel`及类上下文
```python
class _Panel:
```
**EN:** This section introduces `_Panel`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_Panel`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 27-29: Declare fields for `_Panel` such as `label`, `requires_diff`, `draw` / 为 `_Panel` 声明字段，例如 `label`, `requires_diff`, `draw`
```python
    label: str
    requires_diff: bool
    draw: Callable[[np.ndarray, int, _PanelContext], Optional[str]]
```
**EN:** These lines declare the state carried by `_Panel`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `_Panel` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 32-51: Implement helper `_build_panels` / 实现辅助函数 `_build_panels`
```python
def _build_panels() -> list[_Panel]:
    from sglang.srt.debug_utils.comparator.visualizer.panels import (
        _draw_baseline_heatmap,
        _draw_diff_heatmap,
        _draw_diff_histogram,
        _draw_hist2d,
        _draw_sampled,
        _draw_target_heatmap,
    )

    return [
        _Panel(
            label="Baseline Heatmap", requires_diff=False, draw=_draw_baseline_heatmap
        ),
        _Panel(label="Target Heatmap", requires_diff=False, draw=_draw_target_heatmap),
        _Panel(label="Abs Diff Heatmap", requires_diff=True, draw=_draw_diff_heatmap),
        _Panel(label="Abs Diff Hist", requires_diff=True, draw=_draw_diff_histogram),
        _Panel(label="Hist2D", requires_diff=True, draw=_draw_hist2d),
        _Panel(label="Sampled", requires_diff=True, draw=_draw_sampled),
    ]
```
**EN:** Function `_build_panels` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_build_panels` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 54-116: Implement function `generate_comparison_figure` / 实现函数 `generate_comparison_figure`
```python
def generate_comparison_figure(
    *,
    baseline: torch.Tensor,
    target: torch.Tensor,
    name: str,
    output_path: Path,
) -> None:
    """Generate a multi-panel comparison PNG for a baseline/target tensor pair.

    Panels (6 rows x 2 cols, left=normal, right=log10):
      Row 0: Baseline heatmap
      Row 1: Target heatmap
      Row 2: Abs Diff heatmap
      Row 3: Abs Diff histogram
      Row 4: Hist2D scatter (baseline vs target density)
      Row 5: Sampled scatter (10k sampled mini-heatmap)
    """
    import matplotlib.pyplot as plt

    baseline_f: torch.Tensor = baseline.detach().cpu().float()
    target_f: torch.Tensor = target.detach().cpu().float()

    can_diff: bool = baseline_f.shape == target_f.shape

    baseline_2d: torch.Tensor = _preprocess_tensor(baseline_f)
    target_2d: torch.Tensor = _preprocess_tensor(target_f)

    diff: Optional[torch.Tensor] = (baseline_2d - target_2d).abs() if can_diff else None

    ctx = _PanelContext(
        baseline_2d=baseline_2d,
        target_2d=target_2d,
        diff=diff,
        name=name,
    )

    panels: list[_Panel] = _build_panels()
    active: list[_Panel] = [p for p in panels if not p.requires_diff or can_diff]

    nrows: int = len(active)
    ncols: int = 2
    fig, axes = plt.subplots(nrows, ncols, figsize=(5 * ncols, 3.5 * nrows))
    if nrows == 1:
        axes = axes.reshape(1, -1)

    stats_lines: list[str] = []
    for i, panel in enumerate(active):
        stats_line: Optional[str] = panel.draw(axes, i, ctx)
        if stats_line is not None:
            stats_lines.append(stats_line)

    num_stats: int = len(stats_lines)
    title_height: float = 0.015 * num_stats + 0.015
    fig.suptitle(
        "\n".join(stats_lines),
        fontsize=9,
        family="monospace",
        y=1 - title_height / 2,
    )
    plt.tight_layout(rect=[0, 0, 1, 1 - title_height])
    output_path.parent.mkdir(parents=True, exist_ok=True)
    plt.savefig(str(output_path), dpi=150, bbox_inches="tight")
    plt.close(fig)
```
**EN:** Function `generate_comparison_figure` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `generate_comparison_figure` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `_PanelContext`, `_Panel`, `_build_panels`, `generate_comparison_figure`
- **Module role / 模块角色**: Comparison result visualization / 比较结果可视化
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `dataclasses`, `pathlib`, `typing`
- **Third-party / 第三方**: `numpy`, `torch`, `matplotlib`
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.visualizer.preprocessing`, `sglang.srt.debug_utils.comparator.visualizer.panels`
