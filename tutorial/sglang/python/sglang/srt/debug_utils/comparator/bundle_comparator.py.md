# bundle_comparator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/bundle_comparator.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on comparison bundle orchestration. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于比较包编排。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Document the module intent / 说明模块意图
```python
"""Compare two tensor bundles."""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 3-46: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

from pathlib import Path
from typing import Any, Optional, Union

import torch

from sglang.srt.debug_utils.comparator.aligner.entrypoint.executor import (
    AlignerResult,
    execute_aligner_plan,
)
from sglang.srt.debug_utils.comparator.aligner.entrypoint.planner import (
    compute_aligner_plan,
)
from sglang.srt.debug_utils.comparator.aligner.entrypoint.types import AlignerPlan
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types import (
    TokenAlignerPlan,
)
from sglang.srt.debug_utils.comparator.dims_spec import (
    SEQ_DIM_NAME,
    TOKEN_DIM_NAME,
    ParallelAxis,
    apply_dim_names,
    parse_dims,
    resolve_dim_names,
)
from sglang.srt.debug_utils.comparator.dp_utils import filter_to_non_empty_dp_rank
from sglang.srt.debug_utils.comparator.log_sink import log_sink
from sglang.srt.debug_utils.comparator.meta_overrider import MetaOverrider
from sglang.srt.debug_utils.comparator.output_types import (
    BundleFileInfo,
    BundleSideInfo,
    ComparisonNonTensorRecord,
    ComparisonSkipRecord,
    ComparisonTensorRecord,
    ErrorLog,
    _split_logs,
)
from sglang.srt.debug_utils.comparator.tensor_comparator.comparator import (
    compare_tensor_pair,
    compute_tensor_info,
)
from sglang.srt.debug_utils.comparator.utils import Pair
from sglang.srt.debug_utils.dump_loader import LOAD_FAILED, ValueWithMeta
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 49-89: Implement helper `_build_skip_from_one_empty_side` / 实现辅助函数 `_build_skip_from_one_empty_side`
```python
def _build_skip_from_one_empty_side(
    *, name: str, pair: Pair[list[ValueWithMeta]]
) -> ComparisonSkipRecord:
    """Build a skip record when one side of *pair* is empty.

    The non-empty side's tensor info is attached to the record.
    """
    assert not pair.x or not pair.y
    if not pair.x:
        reason, available_side, available_items = (
            "baseline_load_failed",
            "target",
            pair.y,
        )
    else:
        reason, available_side, available_items = (
            "target_load_failed",
            "baseline",
            pair.x,
        )

    tensor_items: list[ValueWithMeta] = [
        it for it in available_items if isinstance(it.value, torch.Tensor)
    ]
    if not tensor_items:
        return ComparisonSkipRecord(name=name, reason=reason)

    first_tensor: torch.Tensor = tensor_items[0].value
    tensor_info = compute_tensor_info(first_tensor, include_sample=True)
    metas: list[dict[str, Any]] = [it.meta for it in tensor_items]
    bundle_info: BundleSideInfo = _collect_bundle_side_info(
        items=tensor_items, metas=metas
    )

    return ComparisonSkipRecord(
        name=name,
        reason=reason,
        available_side=available_side,  # type: ignore[arg-type]
        available_tensor_info=tensor_info,
        available_bundle_info=bundle_info,
    )
```
**EN:** Function `_build_skip_from_one_empty_side` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_build_skip_from_one_empty_side` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 92-121: Implement helper `_collect_bundle_side_info` / 实现辅助函数 `_collect_bundle_side_info`
```python
def _collect_bundle_side_info(
    items: list[ValueWithMeta],
    metas: list[dict[str, Any]],
) -> BundleSideInfo:
    from sglang.srt.debug_utils.comparator.display import (
        PARALLEL_INFO_KEYS,
        _extract_parallel_info,
    )

    files: list[BundleFileInfo] = []
    for item, meta in zip(items, metas):
        assert isinstance(item.value, torch.Tensor)
        tensor: torch.Tensor = item.value

        parallel_info: dict[str, str] = {}
        for key in PARALLEL_INFO_KEYS:
            _extract_parallel_info(row_data=parallel_info, info=meta.get(key, {}))

        files.append(
            BundleFileInfo(
                shape=list(tensor.shape),
                dtype=str(tensor.dtype),
                rank=meta.get("rank"),
                parallel_info=parallel_info if parallel_info else None,
                filename=meta.get("filename"),
            )
        )

    dims: Optional[str] = metas[0].get("dims") if metas else None
    return BundleSideInfo(num_files=len(files), files=files, dims=dims)
```
**EN:** Function `_collect_bundle_side_info` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_collect_bundle_side_info` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 124-154: Implement function `compare_bundle_pair` / 实现函数 `compare_bundle_pair`
```python
def compare_bundle_pair(
    *,
    name: str,
    filenames_pair: Pair[list[str]],
    dir_pair: Pair[Path],
    token_aligner_mode: Optional[str],
    token_aligner_plan: Optional[TokenAlignerPlan],
    diff_threshold: float,
    thd_seq_lens_by_step_pair: Pair[Optional[dict[int, list[int]]]] = Pair(
        x=None, y=None
    ),
    viz_output_dir: Optional[Path] = None,
    compute_per_token: bool = False,
    meta_overrider: Optional[MetaOverrider] = None,
) -> Union[ComparisonTensorRecord, ComparisonSkipRecord, ComparisonNonTensorRecord]:
    with log_sink.context() as collected_logs:
        result = _compare_bundle_pair_inner(
            name=name,
            filenames_pair=filenames_pair,
            dir_pair=dir_pair,
            token_aligner_mode=token_aligner_mode,
            token_aligner_plan=token_aligner_plan,
            diff_threshold=diff_threshold,
            thd_seq_lens_by_step_pair=thd_seq_lens_by_step_pair,
            viz_output_dir=viz_output_dir,
            compute_per_token=compute_per_token,
            meta_overrider=meta_overrider,
        )

    errors, infos = _split_logs(collected_logs)
    return result.model_copy(update={"errors": errors, "infos": infos})
```
**EN:** Function `compare_bundle_pair` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `compare_bundle_pair` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 157-224: Implement helper `_compare_bundle_pair_inner` / 实现辅助函数 `_compare_bundle_pair_inner`
```python
def _compare_bundle_pair_inner(
    *,
    name: str,
    filenames_pair: Pair[list[str]],
    dir_pair: Pair[Path],
    token_aligner_mode: Optional[str],
    token_aligner_plan: Optional[TokenAlignerPlan],
    diff_threshold: float,
    thd_seq_lens_by_step_pair: Pair[Optional[dict[int, list[int]]]] = Pair(
        x=None, y=None
    ),
    viz_output_dir: Optional[Path] = None,
    compute_per_token: bool = False,
    meta_overrider: Optional[MetaOverrider] = None,
) -> Union[ComparisonTensorRecord, ComparisonSkipRecord, ComparisonNonTensorRecord]:
    # 1. Load all successfully loaded values
    all_pair: Pair[list[ValueWithMeta]] = Pair(
        x=_load_all_values(filenames=filenames_pair.x, base_path=dir_pair.x),
        y=_load_all_values(filenames=filenames_pair.y, base_path=dir_pair.y),
    )

    if not all_pair.x or not all_pair.y:
        return _build_skip_from_one_empty_side(name=name, pair=all_pair)

    # 1b. Dims override: patch meta["dims"] before DP filter reads it
    # (--override-dims may add ``# dp:=moe_dp``, so it must run first)
    if meta_overrider is not None and not meta_overrider.is_empty:
        _apply = meta_overrider.apply_to_meta
        all_pair = Pair(
            x=[
                ValueWithMeta(
                    value=v.value, meta=_apply(name=name, meta=v.meta, side="baseline")
                )
                for v in all_pair.x
            ],
            y=[
                ValueWithMeta(
                    value=v.value, meta=_apply(name=name, meta=v.meta, side="target")
                )
                for v in all_pair.y
            ],
        )

    # 1c. DP filter: keep only the non-empty dp_rank
    all_pair = all_pair.map(
        lambda items: filter_to_non_empty_dp_rank(
            items, dp_axis=_extract_dp_axis_from_items(items)
        )
    )

    # 2. Check if any side has non-tensor values → non-tensor display path
    has_non_tensor: bool = any(
        not isinstance(it.value, torch.Tensor) for it in [*all_pair.x, *all_pair.y]
    )
    if has_non_tensor:
        return _compare_bundle_pair_non_tensor_type(name=name, value_pair=all_pair)

    # 3. All values are tensors → tensor comparison path
    return _compare_bundle_pair_tensor_type(
        name=name,
        valid_pair=all_pair,
        token_aligner_mode=token_aligner_mode,
        token_aligner_plan=token_aligner_plan,
        diff_threshold=diff_threshold,
        thd_seq_lens_by_step_pair=thd_seq_lens_by_step_pair,
        viz_output_dir=viz_output_dir,
        compute_per_token=compute_per_token,
    )
```
**EN:** Function `_compare_bundle_pair_inner` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_compare_bundle_pair_inner` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 227-234: Implement helper `_extract_dp_axis_from_items` / 实现辅助函数 `_extract_dp_axis_from_items`
```python
def _extract_dp_axis_from_items(items: list[ValueWithMeta]) -> ParallelAxis:
    """Extract dp axis from the first item's ``meta["dims"]``."""
    if not items:
        return ParallelAxis.DP
    dims_str: Optional[str] = items[0].meta.get("dims")
    if dims_str is None:
        return ParallelAxis.DP
    return parse_dims(dims_str).dp_axis
```
**EN:** Function `_extract_dp_axis_from_items` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_extract_dp_axis_from_items` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 237-326: Implement helper `_compare_bundle_pair_tensor_type` / 实现辅助函数 `_compare_bundle_pair_tensor_type`
```python
def _compare_bundle_pair_tensor_type(
    *,
    name: str,
    valid_pair: Pair[list[ValueWithMeta]],
    token_aligner_mode: Optional[str],
    token_aligner_plan: Optional[TokenAlignerPlan],
    diff_threshold: float,
    thd_seq_lens_by_step_pair: Pair[Optional[dict[int, list[int]]]] = Pair(
        x=None, y=None
    ),
    viz_output_dir: Optional[Path] = None,
    compute_per_token: bool = False,
) -> Union[ComparisonTensorRecord, ComparisonSkipRecord]:
    if not valid_pair.x or not valid_pair.y:
        return _build_skip_from_one_empty_side(name=name, pair=valid_pair)

    # Plan (meta only, no tensor)
    metas_pair: Pair[list[dict[str, Any]]] = valid_pair.map(
        lambda items: [it.meta for it in items]
    )
    plan: AlignerPlan = compute_aligner_plan(
        metas_pair=metas_pair,
        token_aligner_mode=token_aligner_mode,
        token_aligner_plan=token_aligner_plan,
        thd_seq_lens_by_step_pair=thd_seq_lens_by_step_pair,
    )

    # Collect raw bundle info before alignment
    raw_bundle_info: Pair[BundleSideInfo] = Pair(
        x=_collect_bundle_side_info(items=valid_pair.x, metas=metas_pair.x),
        y=_collect_bundle_side_info(items=valid_pair.y, metas=metas_pair.y),
    )

    # Apply dim names to tensors, then execute
    tensors_pair: Pair[list[torch.Tensor]] = Pair(
        x=_apply_dim_names_from_meta(
            tensors=[it.value for it in valid_pair.x],
            metas=metas_pair.x,
        ),
        y=_apply_dim_names_from_meta(
            tensors=[it.value for it in valid_pair.y],
            metas=metas_pair.y,
        ),
    )
    aligner_result: AlignerResult = execute_aligner_plan(
        tensors_pair=tensors_pair, plan=plan
    )
    replicated_checks = aligner_result.replicated_checks

    if aligner_result.tensors is None:
# ... truncated for brevity ...
        name=name,
        diff_threshold=diff_threshold,
        seq_dim=seq_dim,
    )
    record = ComparisonTensorRecord(
        **info.model_dump(),
        traced_plan=aligner_result.traced_plan,
        replicated_checks=replicated_checks,
        raw_bundle_info=raw_bundle_info,
    )

    if viz_output_dir is not None:
        _try_generate_viz(
            baseline=aligned_baseline,
            target=aligned_target,
            name=name,
            viz_output_dir=viz_output_dir,
        )

    return record
```
**EN:** Function `_compare_bundle_pair_tensor_type` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps. The snippet is abbreviated to keep the analysis readable.
**CN:** 函数 `_compare_bundle_pair_tensor_type` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。 为了保持分析可读性，这里的代码片段做了节选。

### Lines 329-359: Implement helper `_try_generate_viz` / 实现辅助函数 `_try_generate_viz`
```python
def _try_generate_viz(
    *,
    baseline: torch.Tensor,
    target: torch.Tensor,
    name: str,
    viz_output_dir: Path,
) -> None:
    from sglang.srt.debug_utils.comparator.visualizer import (
        generate_comparison_figure,
    )
    from sglang.srt.debug_utils.comparator.visualizer.preprocessing import (
        _sanitize_filename,
    )

    filename: str = _sanitize_filename(name) + ".png"
    output_path: Path = viz_output_dir / filename

    try:
        generate_comparison_figure(
            baseline=baseline,
            target=target,
            name=name,
            output_path=output_path,
        )
    except Exception as exc:
        log_sink.add(
            ErrorLog(
                category="visualizer",
                message=f"Visualization failed for {name}: {exc}",
            )
        )
```
**EN:** Function `_try_generate_viz` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_try_generate_viz` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 362-372: Implement helper `_resolve_seq_dim` / 实现辅助函数 `_resolve_seq_dim`
```python
def _resolve_seq_dim(tensor: torch.Tensor) -> Optional[int]:
    """Find the token/seq dimension index from the tensor's named dims."""
    if tensor.names[0] is None:
        return None

    names: tuple[Optional[str], ...] = tensor.names
    for target_name in (TOKEN_DIM_NAME, SEQ_DIM_NAME):
        if target_name in names:
            return list(names).index(target_name)

    return None
```
**EN:** Function `_resolve_seq_dim` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_resolve_seq_dim` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 375-395: Implement helper `_compare_bundle_pair_non_tensor_type` / 实现辅助函数 `_compare_bundle_pair_non_tensor_type`
```python
def _compare_bundle_pair_non_tensor_type(
    *,
    name: str,
    value_pair: Pair[list[ValueWithMeta]],
) -> ComparisonNonTensorRecord:
    baseline_value: Any = value_pair.x[0].value
    target_value: Any = value_pair.y[0].value

    try:
        values_equal: bool = bool(baseline_value == target_value)
    except Exception:
        values_equal = False

    return ComparisonNonTensorRecord(
        name=name,
        baseline_value=repr(baseline_value),
        target_value=repr(target_value),
        baseline_type=type(baseline_value).__name__,
        target_type=type(target_value).__name__,
        values_equal=values_equal,
    )
```
**EN:** Function `_compare_bundle_pair_non_tensor_type` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_compare_bundle_pair_non_tensor_type` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 398-411: Implement helper `_apply_dim_names_from_meta` / 实现辅助函数 `_apply_dim_names_from_meta`
```python
def _apply_dim_names_from_meta(
    *,
    tensors: list[torch.Tensor],
    metas: list[dict[str, Any]],
) -> list[torch.Tensor]:
    if not metas:
        return tensors

    dims_str: Optional[str] = metas[0].get("dims")
    if dims_str is None:
        return tensors

    dim_names: list[str] = resolve_dim_names(dims_str)
    return [apply_dim_names(t, dim_names) for t in tensors]
```
**EN:** Function `_apply_dim_names_from_meta` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_apply_dim_names_from_meta` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 414-427: Implement helper `_load_all_values` / 实现辅助函数 `_load_all_values`
```python
def _load_all_values(filenames: list[str], base_path: Path) -> list[ValueWithMeta]:
    result: list[ValueWithMeta] = []
    for f in filenames:
        item: ValueWithMeta = ValueWithMeta.load(base_path / f)
        if item.value is LOAD_FAILED:
            log_sink.add(
                ErrorLog(
                    category="load_failed",
                    message=f"Failed to load tensor file: {f}",
                )
            )
            continue
        result.append(item)
    return result
```
**EN:** Function `_load_all_values` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_load_all_values` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `_build_skip_from_one_empty_side`, `_collect_bundle_side_info`, `compare_bundle_pair`, `_compare_bundle_pair_inner`, `_extract_dp_axis_from_items`, `_compare_bundle_pair_tensor_type`, `_try_generate_viz`, `_resolve_seq_dim`, `_compare_bundle_pair_non_tensor_type`, `_apply_dim_names_from_meta`, `_load_all_values`
- **Module role / 模块角色**: Comparison bundle orchestration / 比较包编排
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `pathlib`, `typing`
- **Third-party / 第三方**: `torch`
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.aligner.entrypoint.executor`, `sglang.srt.debug_utils.comparator.aligner.entrypoint.planner`, `sglang.srt.debug_utils.comparator.aligner.entrypoint.types`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types`, `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.srt.debug_utils.comparator.dp_utils`, `sglang.srt.debug_utils.comparator.log_sink`, `sglang.srt.debug_utils.comparator.meta_overrider`, `sglang.srt.debug_utils.comparator.output_types`, `sglang.srt.debug_utils.comparator.tensor_comparator.comparator`, `sglang.srt.debug_utils.comparator.utils`, `sglang.srt.debug_utils.dump_loader`, `sglang.srt.debug_utils.comparator.display`, `sglang.srt.debug_utils.comparator.visualizer`, `sglang.srt.debug_utils.comparator.visualizer.preprocessing`
