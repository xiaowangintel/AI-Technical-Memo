# bundle_matcher.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/bundle_matcher.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on bundle matching logic. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于比较包匹配逻辑。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

import dataclasses
from dataclasses import dataclass
from typing import Any

import polars as pl

from sglang.srt.debug_utils.comparator.utils import Pair
from sglang.srt.debug_utils.dump_loader import filter_rows
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 14-14: Define class `TensorFileInfo` and class context / 定义类 `TensorFileInfo`及类上下文
```python
class TensorFileInfo:
```
**EN:** This section introduces `TensorFileInfo`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `TensorFileInfo`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 15-17: Declare fields for `TensorFileInfo` such as `filename`, `name`, `step` / 为 `TensorFileInfo` 声明字段，例如 `filename`, `name`, `step`
```python
    filename: str
    name: str
    step: int
```
**EN:** These lines declare the state carried by `TensorFileInfo`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `TensorFileInfo` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 20-20: Declare module-level symbols such as `TensorBundleInfo` / 声明模块级符号，例如 `TensorBundleInfo`
```python
TensorBundleInfo = list[TensorFileInfo]
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 23-38: Implement function `match_bundles` / 实现函数 `match_bundles`
```python
def match_bundles(
    *,
    dfs: Pair[pl.DataFrame],
    skip_keys: set[str],
) -> list[Pair[TensorBundleInfo]]:
    match_key_cols: list[str] = [c for c in dfs.y.columns if c not in skip_keys]
    unique_keys: pl.DataFrame = dfs.y.select(match_key_cols).unique(maintain_order=True)

    results: list[Pair[TensorBundleInfo]] = []
    for key_values in unique_keys.iter_rows(named=True):
        result = dfs.map(
            lambda df: _rows_to_tensor_infos(filter_rows(df, conditions=key_values))
        )
        results.append(result)

    return results
```
**EN:** Function `match_bundles` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `match_bundles` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 41-46: Implement helper `_rows_to_tensor_infos` / 实现辅助函数 `_rows_to_tensor_infos`
```python
def _rows_to_tensor_infos(rows: list[dict[str, Any]]) -> list[TensorFileInfo]:
    tensor_info_fields: set[str] = {f.name for f in dataclasses.fields(TensorFileInfo)}
    return [
        TensorFileInfo(**{k: v for k, v in row.items() if k in tensor_info_fields})
        for row in rows
    ]
```
**EN:** Function `_rows_to_tensor_infos` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_rows_to_tensor_infos` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `TensorFileInfo`, `match_bundles`, `_rows_to_tensor_infos`
- **Module role / 模块角色**: Bundle matching logic / 比较包匹配逻辑
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `dataclasses`, `typing`
- **Third-party / 第三方**: `polars`
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.utils`, `sglang.srt.debug_utils.dump_loader`
