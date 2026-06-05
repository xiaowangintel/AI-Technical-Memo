# aux_loader.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/aligner/token_aligner/smart/aux_loader.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on heuristic token alignment. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于启发式 token 对齐。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-36: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

from pathlib import Path
from typing import Any, Optional

import polars as pl
import torch

from sglang.srt.debug_utils.comparator.aligner.entrypoint.executor import (
    execute_sub_plans,
)
from sglang.srt.debug_utils.comparator.aligner.entrypoint.planner import (
    compute_per_step_sub_plans,
)
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.aux_plugins import (
    AUX_NAMES,
    _AuxFrameworkPlugin,
    _plugins,
)
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types import (
    TokenAlignerGlobalAux,
    TokenAlignerStepAux,
)
from sglang.srt.debug_utils.comparator.aligner.unsharder.parallel_info import (
    normalize_parallel_info,
)
from sglang.srt.debug_utils.comparator.dims_spec import (
    ParallelAxis,
    TokenLayout,
    apply_dim_names,
    resolve_dim_names,
)
from sglang.srt.debug_utils.comparator.dp_utils import filter_to_non_empty_dp_rank
from sglang.srt.debug_utils.comparator.log_sink import log_sink
from sglang.srt.debug_utils.comparator.output_types import ErrorLog, InfoLog
from sglang.srt.debug_utils.dump_loader import ValueWithMeta, filter_rows
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 39-43: Declare module-level symbols such as `__all__` / 声明模块级符号，例如 `__all__`
```python
__all__ = [
    "AUX_NAMES",
    "has_aux_tensors",
    "load_and_normalize_aux",
]
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 46-87: Implement function `load_and_normalize_aux` / 实现函数 `load_and_normalize_aux`
```python
def load_and_normalize_aux(
    dump_path: Path, df: pl.DataFrame
) -> Optional[TokenAlignerGlobalAux]:
    """Bootstrap: load, unshard, and normalize auxiliary tensors for one side."""
    plugin: Optional[_AuxFrameworkPlugin] = _detect_plugin(df, dump_path=dump_path)
    if plugin is None:
        return None

    available_names: set[str] = set(df["name"].unique().to_list()) & plugin.all_names
    steps: list[int] = sorted(df["step"].unique().to_list())
    tensor_names: set[str] = available_names & plugin.tensor_names
    non_tensor_names: set[str] = available_names & plugin.non_tensor_names

    steps_data: dict[int, dict[str, object]] = {}
    thd_seq_lens_by_step: dict[int, list[int]] = {}
    for step in steps:
        step_data, thd_seq_lens = _load_step_data(
            step=step,
            tensor_names=tensor_names,
            non_tensor_names=non_tensor_names,
            df=df,
            dump_path=dump_path,
            plugin=plugin,
        )
        if step_data:
            steps_data[step] = step_data
        if thd_seq_lens is not None:
            thd_seq_lens_by_step[step] = thd_seq_lens

    layout: TokenLayout = plugin.detect_layout(steps_data)

    step_auxs: dict[int, TokenAlignerStepAux] = {
        step: plugin.compute_step_aux(step_data, layout=layout, step=step)
        for step, step_data in steps_data.items()
    }

    return TokenAlignerGlobalAux(
        step_auxs=step_auxs,
        framework=plugin.name,
        layout=layout,
        thd_seq_lens_by_step=thd_seq_lens_by_step or None,
    )
```
**EN:** Function `load_and_normalize_aux` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `load_and_normalize_aux` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 90-93: Implement function `has_aux_tensors` / 实现函数 `has_aux_tensors`
```python
def has_aux_tensors(df: pl.DataFrame) -> bool:
    """Check if the DataFrame contains the minimum auxiliary tensors for alignment."""
    names: set[str] = set(df["name"].unique().to_list())
    return any(plugin.has_required_names(names) for plugin in _plugins)
```
**EN:** Function `has_aux_tensors` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `has_aux_tensors` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 96-110: Implement helper `_detect_plugin` / 实现辅助函数 `_detect_plugin`
```python
def _detect_plugin(df: pl.DataFrame, dump_path: Path) -> Optional[_AuxFrameworkPlugin]:
    names: set[str] = set(df["name"].unique().to_list())

    for plugin in _plugins:
        if names & plugin.discriminating_names:
            return plugin

    first_row: dict = df.row(0, named=True)
    value: ValueWithMeta = ValueWithMeta.load(dump_path / first_row["filename"])

    for plugin in _plugins:
        if f"{plugin.name}_parallel_info" in value.meta:
            return plugin

    return None
```
**EN:** Function `_detect_plugin` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_detect_plugin` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 113-164: Implement helper `_load_step_data` / 实现辅助函数 `_load_step_data`
```python
def _load_step_data(
    *,
    step: int,
    tensor_names: set[str],
    non_tensor_names: set[str],
    df: pl.DataFrame,
    dump_path: Path,
    plugin: _AuxFrameworkPlugin,
) -> tuple[dict[str, object], Optional[list[int]]]:
    """Load all tensor and non-tensor aux values for a single step.

    Two-pass loading: non-CP-sharded tensors first (to obtain cu_seqlens_q
    for seq_lens), then CP-sharded tensors with seq_lens for THD unshard/reorder.

    Returns (step_data, thd_global_seq_lens).
    """
    result: dict[str, object] = {}

    # Pass 0: non-tensor values
    for name in non_tensor_names:
        value = _load_non_tensor_aux(name=name, step=step, df=df, dump_path=dump_path)
        if value is not None:
            result[name] = value

    # Pass 1: non-CP-sharded tensors (e.g. cu_seqlens_q, seq_lens)
    non_cp_tensor_names: set[str] = tensor_names - plugin.cp_sharded_names
    cp_tensor_names: set[str] = tensor_names & plugin.cp_sharded_names

    for name in non_cp_tensor_names:
        tensor = _load_and_align_aux_tensor(
            name=name, step=step, df=df, dump_path=dump_path, plugin=plugin
        )
        if tensor is not None:
            result[name] = tensor

    # Derive global seq_lens for THD unshard (framework-specific extraction)
    thd_global_seq_lens: Optional[list[int]] = plugin.extract_global_seq_lens(result)

    # Pass 2: CP-sharded tensors (input_ids, position_ids, etc.)
    for name in cp_tensor_names:
        tensor = _load_and_align_aux_tensor(
            name=name,
            step=step,
            df=df,
            dump_path=dump_path,
            plugin=plugin,
            thd_global_seq_lens=thd_global_seq_lens,
        )
        if tensor is not None:
            result[name] = tensor

    return result, thd_global_seq_lens
```
**EN:** Function `_load_step_data` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_load_step_data` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 167-195: Implement helper `_load_non_tensor_aux` / 实现辅助函数 `_load_non_tensor_aux`
```python
def _load_non_tensor_aux(
    *, name: str, step: int, df: pl.DataFrame, dump_path: Path
) -> Optional[object]:
    """Load a non-tensor auxiliary value for a step, validating consistency across ranks."""
    rows = filter_rows(df, conditions={"name": name, "step": step})
    if not rows:
        return None

    loaded: list[ValueWithMeta] = [
        ValueWithMeta.load(dump_path / r["filename"]) for r in rows
    ]
    loaded = filter_to_non_empty_dp_rank(loaded, dp_axis=ParallelAxis.DP)

    if len(loaded) > 1:
        first_value = loaded[0].value
        for i, item in enumerate(loaded[1:], start=1):
            if item.value != first_value:
                log_sink.add(
                    ErrorLog(
                        category=f"{name}_mismatch",
                        message=(
                            f"{name} mismatch across ranks: rank 0 has {first_value}, "
                            f"rank {i} has {item.value}"
                        ),
                    )
                )
                break

    return loaded[0].value
```
**EN:** Function `_load_non_tensor_aux` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_load_non_tensor_aux` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 198-258: Implement helper `_load_and_align_aux_tensor` / 实现辅助函数 `_load_and_align_aux_tensor`
```python
def _load_and_align_aux_tensor(
    *,
    name: str,
    step: int,
    df: pl.DataFrame,
    dump_path: Path,
    plugin: _AuxFrameworkPlugin,
    thd_global_seq_lens: Optional[list[int]] = None,
) -> Optional[torch.Tensor]:
    """Load an auxiliary tensor for (name, step), align if needed."""
    rows = filter_rows(df, conditions={"name": name, "step": step})
    if not rows:
        return None

    loaded: list[ValueWithMeta] = [
        ValueWithMeta.load(dump_path / r["filename"]) for r in rows
    ]
    loaded = filter_to_non_empty_dp_rank(loaded, dp_axis=ParallelAxis.DP)

    tensors: list[torch.Tensor] = [
        item.value for item in loaded if isinstance(item.value, torch.Tensor)
    ]
    if not tensors:
        return None

    if len(tensors) == 1:
        return tensors[0]

    metas: list[dict[str, Any]] = [item.meta for item in loaded]
    metas = _ensure_dims_in_metas(
        name=name, plugin=plugin, metas=metas, ndim=tensors[0].ndim
    )

    sub_plans = compute_per_step_sub_plans(
        metas=metas,
        thd_global_seq_lens=(
            thd_global_seq_lens if name in plugin.cp_sharded_names else None
        ),
    )
    if sub_plans:
        dims_str: Optional[str] = metas[0].get("dims")
        if dims_str is not None:
            dim_names: list[str] = resolve_dim_names(dims_str)
            tensors = [apply_dim_names(t, dim_names) for t in tensors]

        sub_result = execute_sub_plans(tensors=tensors, plans=sub_plans)
        assert sub_result.tensor is not None
        return sub_result.tensor.rename(
            None
        )  # strip named dims before returning to plugin

    log_sink.add(
        InfoLog(
            category="aux_no_dims",
            message=(
                f"aux tensor '{name}' has {len(tensors)} ranks "
                f"but no dims metadata, using rank 0 only"
            ),
        )
    )
    return tensors[0]
```
**EN:** Function `_load_and_align_aux_tensor` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_load_and_align_aux_tensor` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 261-285: Implement helper `_ensure_dims_in_metas` / 实现辅助函数 `_ensure_dims_in_metas`
```python
def _ensure_dims_in_metas(
    *,
    name: str,
    plugin: _AuxFrameworkPlugin,
    metas: list[dict[str, Any]],
    ndim: int,
) -> list[dict[str, Any]]:
    """Inject inferred dims into metas if not already present.

    Returns metas unchanged if dims is already set, or a new list with dims
    injected if inference succeeds for CP-sharded tensors.
    """
    if metas[0].get("dims") is not None:
        return metas

    parallel_infos = [normalize_parallel_info(m) for m in metas]
    has_cp: bool = any(ParallelAxis.CP in info for info in parallel_infos)
    if not has_cp:
        return metas

    if name in plugin.cp_sharded_names:
        inferred_dims: str = plugin.infer_cp_sharded_dims(name=name, ndim=ndim)
        return [{**m, "dims": inferred_dims} for m in metas]

    return metas
```
**EN:** Function `_ensure_dims_in_metas` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_ensure_dims_in_metas` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `load_and_normalize_aux`, `has_aux_tensors`, `_detect_plugin`, `_load_step_data`, `_load_non_tensor_aux`, `_load_and_align_aux_tensor`, `_ensure_dims_in_metas`
- **Module role / 模块角色**: Heuristic token alignment / 启发式 token 对齐
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `pathlib`, `typing`
- **Third-party / 第三方**: `polars`, `torch`
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.aligner.entrypoint.executor`, `sglang.srt.debug_utils.comparator.aligner.entrypoint.planner`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.aux_plugins`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types`, `sglang.srt.debug_utils.comparator.aligner.unsharder.parallel_info`, `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.srt.debug_utils.comparator.dp_utils`, `sglang.srt.debug_utils.comparator.log_sink`, `sglang.srt.debug_utils.comparator.output_types`, `sglang.srt.debug_utils.dump_loader`
