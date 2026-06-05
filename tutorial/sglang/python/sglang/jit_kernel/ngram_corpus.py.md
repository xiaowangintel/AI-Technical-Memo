# ngram_corpus.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/ngram_corpus.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from __future__ import annotations". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from __future__ import annotations”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Imports and module setup
```python
from __future__ import annotations

from collections.abc import Iterable, Sequence
from typing import Dict, List, Tuple

import numpy as np
import torch
import tvm_ffi

from sglang.jit_kernel.utils import cache_once, load_jit

_MATCH_TYPE_MAP = {"BFS": 0, "PROB": 1}
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 15-26: Function `_to_csr`
```python
def _to_csr(batch_tokens: List[List[int]]) -> Tuple[torch.Tensor, torch.Tensor]:
    flat = []
    offsets = [0]
    for seq in batch_tokens:
        flat.extend(seq)
        offsets.append(len(flat))
    tokens_flat = torch.tensor(flat, dtype=torch.int32)
    offsets_t = torch.tensor(offsets, dtype=torch.int64)
    return tokens_flat, offsets_t


@cache_once
```
**EN:** This block defines `_to_csr`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_to_csr`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 27-140: Function `get_ngram_corpus_cls`
```python
def get_ngram_corpus_cls():
    module = load_jit(
        "ngram_corpus",
        cpp_files=[
            "ngram_corpus/result.cpp",
            "ngram_corpus/trie.cpp",
            "ngram_corpus/suffix_automaton.cpp",
            "ngram_corpus/ngram.cpp",
            "ngram_corpus/ngram_corpus_ffi.cpp",
        ],
        header_only=False,
    )
    module.register_once()

    @tvm_ffi.register_object("sgl.NgramCorpus")
    class NgramCorpusFFI(tvm_ffi.Object):
        __slots__ = ("__dict__",)

        def __init__(
            self,
            capacity: int,
            max_trie_depth: int,
            min_bfs_breadth: int,
            max_bfs_breadth: int,
            draft_token_num: int,
            match_type: str,
            external_sam_budget: int = 0,
            external_corpus_max_tokens: int = 10000000,
        ) -> None:
            mt = _MATCH_TYPE_MAP.get(match_type)
            if mt is None:
                raise ValueError(
                    f"Unknown match_type: '{match_type}'. Must be 'BFS' or 'PROB'."
                )
            self.__ffi_init__(
                capacity,
                max_trie_depth,
                min_bfs_breadth,
                max_bfs_breadth,
                draft_token_num,
                mt,
                external_sam_budget,
                external_corpus_max_tokens,
            )
            self._draft_token_num = draft_token_num

        def insert(self, batch_tokens: List[List[int]]) -> None:
            tokens_flat, offsets = _to_csr(batch_tokens)
            self.async_insert(tokens_flat, offsets)  # type: ignore

        def match_stateful(
            self,
            state_ids: List[int],
            batch_tokens: List[List[int]],
            total_lens: List[int],
        ) -> Tuple[np.ndarray, np.ndarray]:
            tokens_flat, offsets = _to_csr(batch_tokens)
            batch_size = len(batch_tokens)
            d = self._draft_token_num

            state_ids_t = torch.tensor(state_ids, dtype=torch.int64)
            total_lens_t = torch.tensor(total_lens, dtype=torch.int64)
            out_tokens = torch.zeros(batch_size * d, dtype=torch.int32)
            out_mask = torch.zeros(batch_size * d * d, dtype=torch.uint8)

            self.batch_match_stateful(  # type: ignore
                state_ids_t, tokens_flat, offsets, total_lens_t, out_tokens, out_mask
            )

            return out_tokens.numpy().astype(np.int64), out_mask.numpy().astype(
                np.int64
            )

        def erase_states(self, state_ids: List[int]) -> None:
            state_ids_t = torch.tensor(state_ids, dtype=torch.int64)
            self.erase_match_state(state_ids_t)  # type: ignore

        def load_external_corpus_named(
            self, corpus_id: str, chunks: Iterable[Sequence[int]], max_tokens: int
        ) -> Tuple[int, int]:
# ...
```
**EN:** This block defines `get_ngram_corpus_cls`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `get_ngram_corpus_cls`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `collections.abc -> Iterable`
- `typing -> Dict`
- `numpy as np`
- `torch`
- `tvm_ffi`
- `sglang.jit_kernel.utils -> cache_once`
