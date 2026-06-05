# seq_info_builder.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/aligner/token_aligner/smart/seq_info_builder.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on heuristic token alignment. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于启发式 token 对齐。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

from dataclasses import dataclass, field

from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types import (
    SeqId,
    TokenAlignerGlobalAux,
    TokenAlignerSeqInfo,
    TokenAlignerSeqsInfo,
    TokenAlignerStepAux,
    TokenLocator,
)
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 16-16: Define class `_SeqInfoAccumulator` and class context / 定义类 `_SeqInfoAccumulator`及类上下文
```python
class _SeqInfoAccumulator:
```
**EN:** This section introduces `_SeqInfoAccumulator`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_SeqInfoAccumulator`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 17-17: Document the module intent / 说明模块意图
```python
    """Mutable accumulator for building TokenAlignerSeqInfo without per-step validation."""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 19-22: Declare fields for `_SeqInfoAccumulator` such as `input_ids`, `positions`, `steps`, `token_index_in_step` / 为 `_SeqInfoAccumulator` 声明字段，例如 `input_ids`, `positions`, `steps`, `token_index_in_step`
```python
    input_ids: list[int] = field(default_factory=list)
    positions: list[int] = field(default_factory=list)
    steps: list[int] = field(default_factory=list)
    token_index_in_step: list[int] = field(default_factory=list)
```
**EN:** These lines declare the state carried by `_SeqInfoAccumulator`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `_SeqInfoAccumulator` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 24-35: Implement method `extend` for `_SeqInfoAccumulator` / 为 `_SeqInfoAccumulator` 实现方法 `extend`
```python
    def extend(
        self,
        *,
        input_ids: list[int],
        positions: list[int],
        steps: list[int],
        token_index_in_step: list[int],
    ) -> None:
        self.input_ids.extend(input_ids)
        self.positions.extend(positions)
        self.steps.extend(steps)
        self.token_index_in_step.extend(token_index_in_step)
```
**EN:** Method `extend` implements behavior on `_SeqInfoAccumulator`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `extend` 为 `_SeqInfoAccumulator` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 37-45: Implement method `build` for `_SeqInfoAccumulator` / 为 `_SeqInfoAccumulator` 实现方法 `build`
```python
    def build(self) -> TokenAlignerSeqInfo:
        return TokenAlignerSeqInfo(
            input_ids=self.input_ids,
            positions=self.positions,
            locator=TokenLocator(
                steps=self.steps,
                token_index_in_step=self.token_index_in_step,
            ),
        )
```
**EN:** Method `build` implements behavior on `_SeqInfoAccumulator`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `build` 为 `_SeqInfoAccumulator` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 48-53: Implement function `build_seqs_info` / 实现函数 `build_seqs_info`
```python
def build_seqs_info(global_aux: TokenAlignerGlobalAux) -> TokenAlignerSeqsInfo:
    """Build sequence info for one side from its auxiliary tensors."""
    return TokenAlignerSeqsInfo(
        sequences=_build_token_aligner_seq_infos(global_aux),
        layout=global_aux.layout,
    )
```
**EN:** Function `build_seqs_info` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `build_seqs_info` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 56-81: Implement helper `_build_token_aligner_seq_infos` / 实现辅助函数 `_build_token_aligner_seq_infos`
```python
def _build_token_aligner_seq_infos(
    global_aux: TokenAlignerGlobalAux,
) -> dict[SeqId, TokenAlignerSeqInfo]:
    """Build token index for any framework/layout using seq_ids for identity tracking."""
    accum: dict[SeqId, _SeqInfoAccumulator] = {}

    for step in sorted(global_aux.step_auxs.keys()):
        aux: TokenAlignerStepAux = global_aux.step_auxs[step]

        offset: int = 0
        for seq_index, seq_len in enumerate(aux.seq_lens):
            seq_id: SeqId = aux.seq_ids[seq_index]

            if seq_id not in accum:
                accum[seq_id] = _SeqInfoAccumulator()

            accum[seq_id].extend(
                input_ids=aux.input_ids[offset : offset + seq_len],
                positions=aux.positions[offset : offset + seq_len],
                steps=[step] * seq_len,
                token_index_in_step=list(range(offset, offset + seq_len)),
            )

            offset += seq_len

    return {seq_id: acc.build() for seq_id, acc in accum.items()}
```
**EN:** Function `_build_token_aligner_seq_infos` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_build_token_aligner_seq_infos` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `_SeqInfoAccumulator`, `build_seqs_info`, `_build_token_aligner_seq_infos`
- **Module role / 模块角色**: Heuristic token alignment / 启发式 token 对齐
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `dataclasses`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types`
