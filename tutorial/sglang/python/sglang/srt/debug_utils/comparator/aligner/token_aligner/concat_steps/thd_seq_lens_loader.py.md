# thd_seq_lens_loader.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/aligner/token_aligner/concat_steps/thd_seq_lens_loader.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on cross-step token concatenation alignment. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于跨步骤 token 拼接对齐。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

from pathlib import Path
from typing import Optional

import polars as pl

from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.aux_loader import (
    _detect_plugin,
    _load_and_align_aux_tensor,
)
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.aux_plugins import (
    _AuxFrameworkPlugin,
)
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 17-43: Implement function `load_thd_seq_lens_only` / 实现函数 `load_thd_seq_lens_only`
```python
def load_thd_seq_lens_only(
    dump_path: Path, df: pl.DataFrame
) -> Optional[dict[int, list[int]]]:
    plugin: Optional[_AuxFrameworkPlugin] = _detect_plugin(df, dump_path=dump_path)
    if plugin is None or not plugin.cp_sharded_names:
        return None

    non_cp_tensor_names: set[str] = (
        set(df["name"].unique().to_list()) & plugin.tensor_names
    ) - plugin.cp_sharded_names
    steps: list[int] = sorted(df["step"].unique().to_list())

    result: dict[int, list[int]] = {}
    for step in steps:
        step_data: dict[str, object] = {}
        for name in non_cp_tensor_names:
            tensor = _load_and_align_aux_tensor(
                name=name, step=step, df=df, dump_path=dump_path, plugin=plugin
            )
            if tensor is not None:
                step_data[name] = tensor

        seq_lens: Optional[list[int]] = plugin.extract_global_seq_lens(step_data)
        if seq_lens is not None:
            result[step] = seq_lens

    return result or None
```
**EN:** Function `load_thd_seq_lens_only` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `load_thd_seq_lens_only` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `load_thd_seq_lens_only`
- **Module role / 模块角色**: Cross-step token concatenation alignment / 跨步骤 token 拼接对齐
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `pathlib`, `typing`
- **Third-party / 第三方**: `polars`
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.aux_loader`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.aux_plugins`
