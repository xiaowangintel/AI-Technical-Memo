# planner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/aligner/token_aligner/smart/planner.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on heuristic token alignment. It mainly computes reusable plans that later stages can execute deterministically. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于启发式 token 对齐。它主要用于计算可复用的计划对象，供后续阶段确定性执行。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

from collections import defaultdict
from typing import NamedTuple, Optional

from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types import (
    SeqId,
    TokenAlignerPlan,
    TokenAlignerSeqInfo,
    TokenAlignerSeqsInfo,
    TokenLocator,
)
from sglang.srt.debug_utils.comparator.utils import Pair
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 16-54: Implement function `compute_token_aligner_plan` / 实现函数 `compute_token_aligner_plan`
```python
def compute_token_aligner_plan(
    seqs_info_pair: Pair[TokenAlignerSeqsInfo],
) -> TokenAlignerPlan:
    """Compute a token alignment plan from two side token seqs_info_pair."""
    matched_pairs: list[tuple[SeqId, SeqId]] = _match_sequences(
        seqs=Pair(x=seqs_info_pair.x.sequences, y=seqs_info_pair.y.sequences)
    )

    _empty = TokenLocator(steps=[], token_index_in_step=[])
    locator_x: TokenLocator = _empty
    locator_y: TokenLocator = _empty

    for seq_id_x, seq_id_y in matched_pairs:
        rec: Pair[TokenAlignerSeqInfo] = Pair(
            x=seqs_info_pair.x.sequences[seq_id_x],
            y=seqs_info_pair.y.sequences[seq_id_y],
        )

        # positions is validated to be [0, 1, ..., N-1], so position == index
        # and the common range is simply [0, min(len_x, len_y)).
        common_len: int = min(len(rec.x.positions), len(rec.y.positions))

        x_ids = rec.x.input_ids[:common_len]
        y_ids = rec.y.input_ids[:common_len]
        assert x_ids == y_ids, f"{seq_id_x=} {seq_id_y=} {x_ids=} {y_ids=}"

        locator_x = locator_x + TokenLocator(
            steps=rec.x.locator.steps[:common_len],
            token_index_in_step=rec.x.locator.token_index_in_step[:common_len],
        )
        locator_y = locator_y + TokenLocator(
            steps=rec.y.locator.steps[:common_len],
            token_index_in_step=rec.y.locator.token_index_in_step[:common_len],
        )

    return TokenAlignerPlan(
        locators=Pair(x=locator_x, y=locator_y),
        layouts=seqs_info_pair.map(lambda s: s.layout),
    )
```
**EN:** Function `compute_token_aligner_plan` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `compute_token_aligner_plan` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 60-89: Implement helper `_match_sequences` / 实现辅助函数 `_match_sequences`
```python
def _match_sequences(
    seqs: Pair[dict[SeqId, TokenAlignerSeqInfo]],
) -> list[tuple[SeqId, SeqId]]:
    """For each y (target) sequence, find a matching x (baseline) sequence.

    Two-pass: exact match first, then prefix match for remaining.
    """
    x_lookup: dict[tuple[int, ...], list[SeqId]] = defaultdict(list)
    for seq_id, rec in seqs.x.items():
        x_lookup[tuple(rec.input_ids)].append(seq_id)

    claimed_x_ids: set[SeqId] = set()
    matched_seq_id_pairs: list[tuple[SeqId, SeqId]] = []

    for seq_id_y in sorted(seqs.y.keys()):
        seq_y: TokenAlignerSeqInfo = seqs.y[seq_id_y]

        matched_x: Optional[SeqId] = _find_matching_x_exact(
            seq_y=seq_y, x_lookup=x_lookup, claimed_x_ids=claimed_x_ids
        )
        if matched_x is None:
            matched_x = _find_matching_x_prefix(
                seq_y=seq_y, x_seqs=seqs.x, claimed_x_ids=claimed_x_ids
            )

        if matched_x is not None:
            matched_seq_id_pairs.append((matched_x, seq_id_y))
            claimed_x_ids.add(matched_x)

    return matched_seq_id_pairs
```
**EN:** Function `_match_sequences` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_match_sequences` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 92-104: Implement helper `_find_matching_x_exact` / 实现辅助函数 `_find_matching_x_exact`
```python
def _find_matching_x_exact(
    *,
    seq_y: TokenAlignerSeqInfo,
    x_lookup: dict[tuple[int, ...], list[SeqId]],
    claimed_x_ids: set[SeqId],
) -> Optional[SeqId]:
    """Find an x sequence with identical input_ids."""
    ids_y_key: tuple[int, ...] = tuple(seq_y.input_ids)
    candidates: list[SeqId] = x_lookup.get(ids_y_key, [])
    for candidate in candidates:
        if candidate not in claimed_x_ids:
            return candidate
    return None
```
**EN:** Function `_find_matching_x_exact` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_find_matching_x_exact` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 107-107: Define class `_PrefixCandidate` and class context / 定义类 `_PrefixCandidate`及类上下文
```python
class _PrefixCandidate(NamedTuple):
```
**EN:** This section introduces `_PrefixCandidate`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_PrefixCandidate`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 108-109: Declare fields for `_PrefixCandidate` such as `seq_id_x`, `overlap_len` / 为 `_PrefixCandidate` 声明字段，例如 `seq_id_x`, `overlap_len`
```python
    seq_id_x: SeqId
    overlap_len: int
```
**EN:** These lines declare the state carried by `_PrefixCandidate`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `_PrefixCandidate` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 112-129: Implement helper `_find_matching_x_prefix` / 实现辅助函数 `_find_matching_x_prefix`
```python
def _find_matching_x_prefix(
    *,
    seq_y: TokenAlignerSeqInfo,
    x_seqs: dict[SeqId, TokenAlignerSeqInfo],
    claimed_x_ids: set[SeqId],
) -> Optional[SeqId]:
    """Find the x sequence with the longest prefix relationship to y."""
    ids_y: list[int] = seq_y.input_ids
    candidates: list[_PrefixCandidate] = [
        _PrefixCandidate(
            seq_id_x=seq_id_x, overlap_len=min(len(seq_x.input_ids), len(ids_y))
        )
        for seq_id_x, seq_x in x_seqs.items()
        if seq_id_x not in claimed_x_ids and _is_prefix_pair(seq_x.input_ids, ids_y)
    ]
    if not candidates:
        return None
    return max(candidates, key=lambda c: c.overlap_len).seq_id_x
```
**EN:** Function `_find_matching_x_prefix` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_find_matching_x_prefix` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 132-135: Implement helper `_is_prefix_pair` / 实现辅助函数 `_is_prefix_pair`
```python
def _is_prefix_pair(a: list[int], b: list[int]) -> bool:
    """True if a is a prefix of b, or b is a prefix of a."""
    shorter_len: int = min(len(a), len(b))
    return a[:shorter_len] == b[:shorter_len]
```
**EN:** Function `_is_prefix_pair` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_is_prefix_pair` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `_PrefixCandidate`, `compute_token_aligner_plan`, `_match_sequences`, `_find_matching_x_exact`, `_find_matching_x_prefix`, `_is_prefix_pair`
- **Module role / 模块角色**: Heuristic token alignment / 启发式 token 对齐
- **Implementation focus / 实现重点**: Computes reusable plans that later stages can execute deterministically / 计算可复用的计划对象，供后续阶段确定性执行

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `collections`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types`, `sglang.srt.debug_utils.comparator.utils`
