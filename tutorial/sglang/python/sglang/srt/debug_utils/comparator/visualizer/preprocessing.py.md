# preprocessing.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/visualizer/preprocessing.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on comparison result visualization. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于比较结果可视化。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Document the module intent / 说明模块意图
```python
"""Tensor preprocessing and utility functions for visualization."""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 3-9: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

import math
import re

import numpy as np
import torch
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 11-12: Declare module-level symbols such as `_DOWNSAMPLE_THRESHOLD`, `_SCATTER_SAMPLE_SIZE` / 声明模块级符号，例如 `_DOWNSAMPLE_THRESHOLD`, `_SCATTER_SAMPLE_SIZE`
```python
_DOWNSAMPLE_THRESHOLD: int = 10_000_000
_SCATTER_SAMPLE_SIZE: int = 10_000
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 15-24: Implement helper `_preprocess_tensor` / 实现辅助函数 `_preprocess_tensor`
```python
def _preprocess_tensor(tensor: torch.Tensor) -> torch.Tensor:
    t: torch.Tensor = tensor.squeeze()

    while t.ndim < 2:
        t = t.unsqueeze(0)
    if t.ndim > 2:
        t = t.reshape(-1, t.shape[-1])

    t = _reshape_to_balanced_aspect(t)
    return t
```
**EN:** Function `_preprocess_tensor` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_preprocess_tensor` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 27-48: Implement helper `_reshape_to_balanced_aspect` / 实现辅助函数 `_reshape_to_balanced_aspect`
```python
def _reshape_to_balanced_aspect(
    t: torch.Tensor, max_ratio: float = 5.0
) -> torch.Tensor:
    assert t.ndim == 2

    h, w = t.shape
    ratio: float = h / w if w > 0 else float("inf")

    if 1 / max_ratio <= ratio <= max_ratio:
        return t

    total: int = h * w
    target_side: int = int(math.sqrt(total))

    for new_h in range(target_side, 0, -1):
        if total % new_h == 0:
            new_w: int = total // new_h
            new_ratio: float = new_h / new_w
            if 1 / max_ratio <= new_ratio <= max_ratio:
                return t.reshape(new_h, new_w)

    return t.reshape(1, -1)
```
**EN:** Function `_reshape_to_balanced_aspect` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_reshape_to_balanced_aspect` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 54-55: Implement helper `_to_log10` / 实现辅助函数 `_to_log10`
```python
def _to_log10(t: torch.Tensor) -> torch.Tensor:
    return t.abs().clamp(min=1e-10).log10()
```
**EN:** Function `_to_log10` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_to_log10` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 58-67: Implement helper `_format_log_ticks` / 实现辅助函数 `_format_log_ticks`
```python
def _format_log_ticks(ax: object, axis: str = "both") -> None:
    from matplotlib.ticker import FuncFormatter

    formatter = FuncFormatter(
        lambda x, _: f"1e{int(x)}" if x == int(x) else f"1e{x:.1f}"
    )
    if axis in ("x", "both"):
        ax.xaxis.set_major_formatter(formatter)
    if axis in ("y", "both"):
        ax.yaxis.set_major_formatter(formatter)
```
**EN:** Function `_format_log_ticks` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_log_ticks` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 70-75: Implement helper `_format_stats` / 实现辅助函数 `_format_stats`
```python
def _format_stats(name: str, t: torch.Tensor) -> str:
    return (
        f"{name}: shape={tuple(t.shape)}, "
        f"min={t.min().item():.4g}, max={t.max().item():.4g}, "
        f"mean={t.mean().item():.4g}, std={t.std().item():.4g}"
    )
```
**EN:** Function `_format_stats` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_stats` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 78-85: Implement helper `_safe_hist` / 实现辅助函数 `_safe_hist`
```python
def _safe_hist(
    ax: object, data: np.ndarray, *, bins: int = 100, **kwargs: object
) -> None:
    data_f64: np.ndarray = data.astype(np.float64)
    try:
        ax.hist(data_f64, bins=bins, **kwargs)
    except ValueError:
        ax.hist(data_f64, bins=max(1, len(np.unique(data_f64[:1000]))), **kwargs)
```
**EN:** Function `_safe_hist` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_safe_hist` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 88-97: Implement helper `_maybe_downsample_numpy` / 实现辅助函数 `_maybe_downsample_numpy`
```python
def _maybe_downsample_numpy(
    t: torch.Tensor,
    max_elements: int = _DOWNSAMPLE_THRESHOLD,
) -> np.ndarray:
    if t.numel() <= max_elements:
        return t.numpy()

    rng: np.random.Generator = np.random.default_rng(seed=0)
    indices: np.ndarray = rng.choice(t.numel(), max_elements, replace=False)
    return t.numpy()[indices]
```
**EN:** Function `_maybe_downsample_numpy` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_maybe_downsample_numpy` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 100-101: Implement helper `_sanitize_filename` / 实现辅助函数 `_sanitize_filename`
```python
def _sanitize_filename(name: str) -> str:
    return re.sub(r"[/\.\s]+", "_", name).strip("_")
```
**EN:** Function `_sanitize_filename` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_sanitize_filename` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `_preprocess_tensor`, `_reshape_to_balanced_aspect`, `_to_log10`, `_format_log_ticks`, `_format_stats`, `_safe_hist`, `_maybe_downsample_numpy`, `_sanitize_filename`
- **Module role / 模块角色**: Comparison result visualization / 比较结果可视化
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `math`, `re`
- **Third-party / 第三方**: `numpy`, `torch`, `matplotlib`
- **Internal / 内部**: None / 无
