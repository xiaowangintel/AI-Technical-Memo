# entrypoint.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/aligner/token_aligner/entrypoint.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on token alignment orchestration. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于token 对齐编排。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-29: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

from dataclasses import dataclass
from pathlib import Path
from typing import Literal, Optional

import polars as pl

from sglang.srt.debug_utils.comparator.aligner.token_aligner.concat_steps.thd_seq_lens_loader import (
    load_thd_seq_lens_only,
)
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.aux_loader import (
    has_aux_tensors,
    load_and_normalize_aux,
)
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.planner import (
    compute_token_aligner_plan,
)
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.seq_info_builder import (
    build_seqs_info,
)
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types import (
    TokenAlignerGlobalAux,
    TokenAlignerPlan,
    TokenAlignerSeqsInfo,
)
from sglang.srt.debug_utils.comparator.log_sink import log_sink
from sglang.srt.debug_utils.comparator.output_types import InfoLog
from sglang.srt.debug_utils.comparator.utils import Pair
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 31-34: Declare module-level symbols such as `_NONE_THD`, `TokenAlignerMode` / 声明模块级符号，例如 `_NONE_THD`, `TokenAlignerMode`
```python
_NONE_THD: Pair[Optional[dict[int, list[int]]]] = Pair(x=None, y=None)


TokenAlignerMode = Literal["concat_steps", "smart"]
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 38-38: Define class `TokenAlignerResult` and class context / 定义类 `TokenAlignerResult`及类上下文
```python
class TokenAlignerResult:
```
**EN:** This section introduces `TokenAlignerResult`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `TokenAlignerResult`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 39-39: Document the module intent / 说明模块意图
```python
    """Result of token aligner computation, bundling mode + plan with THD metadata."""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 41-43: Declare fields for `TokenAlignerResult` such as `mode`, `plan`, `thd_seq_lens_by_step_pair` / 为 `TokenAlignerResult` 声明字段，例如 `mode`, `plan`, `thd_seq_lens_by_step_pair`
```python
    mode: Optional[TokenAlignerMode]
    plan: Optional[TokenAlignerPlan]
    thd_seq_lens_by_step_pair: Pair[Optional[dict[int, list[int]]]]
```
**EN:** These lines declare the state carried by `TokenAlignerResult`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `TokenAlignerResult` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 46-78: Implement function `compute_maybe_token_aligner_result` / 实现函数 `compute_maybe_token_aligner_result`
```python
def compute_maybe_token_aligner_result(
    *,
    dir_pair: Pair[Path],
    dfs: Pair[pl.DataFrame],
    token_aligner_mode: Optional[TokenAlignerMode],
) -> TokenAlignerResult:
    if token_aligner_mode is None:
        return TokenAlignerResult(
            mode=None, plan=None, thd_seq_lens_by_step_pair=_NONE_THD
        )

    if token_aligner_mode == "concat_steps":
        thd_pair: Pair[Optional[dict[int, list[int]]]] = _load_thd_seq_lens_pair(
            dir_pair=dir_pair, dfs=dfs
        )
        return TokenAlignerResult(
            mode="concat_steps", plan=None, thd_seq_lens_by_step_pair=thd_pair
        )
    elif token_aligner_mode == "smart":
        if not (has_aux_tensors(dfs.x) and has_aux_tensors(dfs.y)):
            log_sink.add(
                InfoLog(
                    category="aux_tensors_missing",
                    message="Aux tensors missing, skipping token alignment",
                )
            )
            return TokenAlignerResult(
                mode=None, plan=None, thd_seq_lens_by_step_pair=_NONE_THD
            )

        return _build_smart_result(dir_pair=dir_pair, dfs=dfs)
    else:
        raise NotImplementedError(f"Unknown {token_aligner_mode=}")
```
**EN:** Function `compute_maybe_token_aligner_result` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `compute_maybe_token_aligner_result` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 81-120: Implement helper `_build_smart_result` / 实现辅助函数 `_build_smart_result`
```python
def _build_smart_result(
    *,
    dir_pair: Pair[Path],
    dfs: Pair[pl.DataFrame],
) -> TokenAlignerResult:
    """Load aux tensors, build token indices, and compute the alignment plan."""
    aux_pair: Pair[Optional[TokenAlignerGlobalAux]] = Pair(
        x=load_and_normalize_aux(dump_path=dir_pair.x, df=dfs.x),
        y=load_and_normalize_aux(dump_path=dir_pair.y, df=dfs.y),
    )

    thd_seq_lens_by_step_pair: Pair[Optional[dict[int, list[int]]]] = aux_pair.map(
        lambda aux: aux.thd_seq_lens_by_step if aux is not None else None
    )

    if aux_pair.x is None or aux_pair.y is None:
        log_sink.add(
            InfoLog(
                category="framework_detection_failed",
                message="Framework detection failed, skipping token alignment",
            )
        )
        return TokenAlignerResult(
            mode=None,
            plan=None,
            thd_seq_lens_by_step_pair=thd_seq_lens_by_step_pair,
        )

    global_aux: Pair[TokenAlignerGlobalAux] = Pair(x=aux_pair.x, y=aux_pair.y)

    seqs_info: Pair[TokenAlignerSeqsInfo] = global_aux.map(build_seqs_info)

    plan: Optional[TokenAlignerPlan] = compute_token_aligner_plan(
        seqs_info_pair=seqs_info
    )
    return TokenAlignerResult(
        mode="smart",
        plan=plan,
        thd_seq_lens_by_step_pair=thd_seq_lens_by_step_pair,
    )
```
**EN:** Function `_build_smart_result` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_build_smart_result` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 123-132: Implement helper `_load_thd_seq_lens_pair` / 实现辅助函数 `_load_thd_seq_lens_pair`
```python
def _load_thd_seq_lens_pair(
    *,
    dir_pair: Pair[Path],
    dfs: Pair[pl.DataFrame],
) -> Pair[Optional[dict[int, list[int]]]]:
    """Load only thd_seq_lens for each side (lightweight, no full aux loading)."""
    return Pair(
        x=load_thd_seq_lens_only(dump_path=dir_pair.x, df=dfs.x),
        y=load_thd_seq_lens_only(dump_path=dir_pair.y, df=dfs.y),
    )
```
**EN:** Function `_load_thd_seq_lens_pair` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_load_thd_seq_lens_pair` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `TokenAlignerResult`, `compute_maybe_token_aligner_result`, `_build_smart_result`, `_load_thd_seq_lens_pair`
- **Module role / 模块角色**: Token alignment orchestration / token 对齐编排
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `dataclasses`, `pathlib`, `typing`
- **Third-party / 第三方**: `polars`
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.aligner.token_aligner.concat_steps.thd_seq_lens_loader`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.aux_loader`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.planner`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.seq_info_builder`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types`, `sglang.srt.debug_utils.comparator.log_sink`, `sglang.srt.debug_utils.comparator.output_types`, `sglang.srt.debug_utils.comparator.utils`
