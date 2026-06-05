# testing_helpers.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/testing_helpers.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on comparator testing helpers in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 comparator testing helpers 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Document the module
```python
"""Shared test helpers for comparator tests."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 3-11: Import dependencies
```python
from __future__ import annotations

import re
from io import StringIO
from typing import Optional

from rich.console import Console

from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 13-18: Register CI metadata
```python
register_cpu_ci(
    est_time=0,
    suite="base-a-test-cpu",
    nightly=True,
    disabled="helper module, no tests",
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 20-24: Import dependencies
```python
from sglang.srt.debug_utils.comparator.tensor_comparator.types import (
    DiffInfo,
    TensorInfo,
    TensorStats,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 26-40: Define module constants
```python
DEFAULT_PERCENTILES: dict[int, float] = {
    1: -1.8,
    5: -1.5,
    50: 0.0,
    95: 1.5,
    99: 1.8,
}

DEFAULT_ABS_DIFF_PERCENTILES: dict[int, float] = {
    1: 0.0001,
    5: 0.0001,
    50: 0.0002,
    95: 0.0004,
    99: 0.0005,
}
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 43-58: Define helper: make stats
```python
def make_stats(
    mean: float = 0.0,
    abs_mean: float = 0.8,
    std: float = 1.0,
    min: float = -2.0,
    max: float = 2.0,
    percentiles: Optional[dict[int, float]] = None,
) -> TensorStats:
    return TensorStats(
        mean=mean,
        abs_mean=abs_mean,
        std=std,
        min=min,
        max=max,
        percentiles=percentiles if percentiles is not None else DEFAULT_PERCENTILES,
    )
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 61-83: Define helper: make diff
```python
def make_diff(
    rel_diff: float = 0.0001,
    max_abs_diff: float = 0.0005,
    mean_abs_diff: float = 0.0002,
    abs_diff_percentiles: Optional[dict[int, float]] = None,
    diff_threshold: float = 1e-3,
    passed: bool = True,
) -> DiffInfo:
    return DiffInfo(
        rel_diff=rel_diff,
        max_abs_diff=max_abs_diff,
        mean_abs_diff=mean_abs_diff,
        abs_diff_percentiles=(
            abs_diff_percentiles
            if abs_diff_percentiles is not None
            else DEFAULT_ABS_DIFF_PERCENTILES
        ),
        max_diff_coord=[2, 3],
        baseline_at_max=1.0,
        target_at_max=1.0005,
        diff_threshold=diff_threshold,
        passed=passed,
    )
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 86-86: Define module constants
```python
_ANSI_ESCAPE_RE = re.compile(r"\033\[([0-9;]*)m")
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 89-115: Define helper: assert rich tags balanced
```python
def assert_rich_tags_balanced(markup: str) -> None:
    """Render Rich markup to ANSI and verify no styles are active at the end.

    Tracks ANSI style state through the output. A ``\\033[0m`` (reset)
    clears all active styles; any other ``\\033[Nm`` sets a style.
    At the end of the output, no style should remain active.
    """
    buf = StringIO()
    console = Console(file=buf, force_terminal=True, width=10000, highlight=False)
    console.print(markup, end="")
    ansi_output: str = buf.getvalue()

    if "\033[" not in ansi_output:
        return

    styled = False
    for match in _ANSI_ESCAPE_RE.finditer(ansi_output):
        params: str = match.group(1)
        if params == "0" or params == "":
            styled = False
        else:
            styled = True

    assert not styled, (
        f"ANSI styles still active at end of output — likely unclosed Rich tag.\n"
        f"Last 200 chars of ANSI output: {ansi_output[-200:]!r}"
    )
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 118-129: Define helper: make tensor info
```python
def make_tensor_info(
    shape: Optional[list[int]] = None,
    dtype: str = "torch.float32",
    stats: Optional[TensorStats] = None,
    sample: Optional[str] = None,
) -> TensorInfo:
    return TensorInfo(
        shape=shape if shape is not None else [4, 8],
        dtype=dtype,
        stats=stats if stats is not None else make_stats(),
        sample=sample,
    )
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.tensor_comparator.types`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `__future__`, `io`, `re`, `rich.console`, `typing`
- Notable symbols / 关键符号: None / 无
