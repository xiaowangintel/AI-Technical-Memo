# ngram_corpus.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/speculative/cpp_ngram/ngram_corpus.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file supports the n-gram speculative decoding path, especially corpus access or related auxiliary configuration for the C++ n-gram module. It helps the runtime reuse external text statistics during drafting. / 该文件服务于 n-gram 推测解码路径，尤其是 C++ n-gram 模块所需的语料访问或相关辅助配置。它帮助运行时在起草阶段复用外部文本统计信息。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13: Module header, imports, and shared constants
```python
# -*- coding: utf-8 -*-

import logging
from collections.abc import Iterable, Sequence
from typing import Dict, List, Tuple

import numpy as np

from sglang.jit_kernel.ngram_corpus import get_ngram_corpus_cls

logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 14-14: NgramCorpus class declaration
```python
class NgramCorpus:
```
**EN:** This block declares the `NgramCorpus` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `NgramCorpus` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 15-42: NgramCorpus initializer
```python
    def __init__(
        self,
        max_trie_depth=18,
        min_bfs_breadth=1,
        max_bfs_breadth=8,
        draft_token_num=8,
        match_type="BFS",
        capacity=1000000,
        external_sam_budget=0,
        external_corpus_max_tokens=10000000,
    ) -> None:
        cls = get_ngram_corpus_cls()
        self._obj = cls(
            capacity=capacity,
            max_trie_depth=max_trie_depth,
            min_bfs_breadth=min_bfs_breadth,
            max_bfs_breadth=max_bfs_breadth,
            draft_token_num=draft_token_num,
            match_type=match_type,
            external_sam_budget=external_sam_budget,
            external_corpus_max_tokens=external_corpus_max_tokens,
        )
        self.draft_token_num = draft_token_num
        self.external_corpus_max_tokens = external_corpus_max_tokens
        self._req_id_to_state_id: Dict[str, int] = {}
        self._next_state_id: int = 0
        self._corpus_token_counts: Dict[str, int] = {}
        self._total_loaded_tokens: int = 0
```
**EN:** This block initializes the `NgramCorpus` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `NgramCorpus` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 44-50: NgramCorpus._get_state_id method
```python
    def _get_state_id(self, req_id: str) -> int:
        sid = self._req_id_to_state_id.get(req_id)
        if sid is None:
            sid = self._next_state_id
            self._next_state_id += 1
            self._req_id_to_state_id[req_id] = sid
        return sid
```
**EN:** This block uses `NgramCorpus._get_state_id` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NgramCorpus._get_state_id` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 52-53: NgramCorpus.batch_put method
```python
    def batch_put(self, batch_tokens: List[List[int]]):
        self._obj.insert(batch_tokens)
```
**EN:** This block uses `NgramCorpus.batch_put` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NgramCorpus.batch_put` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 55-56: NgramCorpus.synchronize method
```python
    def synchronize(self):
        self._obj.synchronize()  # type: ignore
```
**EN:** This block uses `NgramCorpus.synchronize` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NgramCorpus.synchronize` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 58-60: NgramCorpus.remaining_token_budget method
```python
    @property
    def remaining_token_budget(self) -> int:
        return self.external_corpus_max_tokens - self._total_loaded_tokens
```
**EN:** This block uses `NgramCorpus.remaining_token_budget` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NgramCorpus.remaining_token_budget` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 62-77: NgramCorpus.load_external_corpus_named method
```python
    def load_external_corpus_named(
        self, corpus_id: str, chunks: Iterable[Sequence[int]]
    ) -> int:
        if corpus_id in self._corpus_token_counts:
            raise ValueError(
                f"External corpus '{corpus_id}' already exists. Remove it before "
                f"adding a new corpus with the same id."
            )
        # Note(kpham-sgl): remaining_token_budget is stale (e.g if there are removes
        # during the load), which makes the budget more conservative than it should be.
        # This is acceptable because otherwise load_external_corpus_named would need to check the budget after each chunk,
        # which would be inefficient.
        _, loaded_token_count = self._obj.load_external_corpus_named(
            corpus_id, chunks, self.remaining_token_budget
        )
        return loaded_token_count
```
**EN:** This block uses `NgramCorpus.load_external_corpus_named` to load resources or weights. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NgramCorpus.load_external_corpus_named` 来加载资源或权重。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 78-80: Class-level supporting statements
```python

    # Commit corpus bookkeeping after successful load. Call only at background thread join.
    # (or after synchronous load_external_corpus_named returns)
```
**EN:** This block contains supporting statements for the `NgramCorpus` class, such as constants, attributes, registrations, or small helper expressions that shape the surrounding control flow.
**CN:** 该代码块包含`NgramCorpus` 类的配套语句，例如常量、属性、注册逻辑或小型辅助表达式，它们共同塑造了周边控制流程。

### Lines 81-85: NgramCorpus.commit_external_corpus_load method
```python
    def commit_external_corpus_load(
        self, corpus_id: str, loaded_token_count: int
    ) -> None:
        self._corpus_token_counts[corpus_id] = loaded_token_count
        self._total_loaded_tokens += loaded_token_count
```
**EN:** This block uses `NgramCorpus.commit_external_corpus_load` to load resources or weights. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NgramCorpus.commit_external_corpus_load` 来加载资源或权重。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 87-90: NgramCorpus.remove_external_corpus method
```python
    def remove_external_corpus(self, corpus_id: str) -> None:
        self._obj.remove_corpus(corpus_id)
        old_count = self._corpus_token_counts.pop(corpus_id, 0)
        self._total_loaded_tokens -= old_count
```
**EN:** This block uses `NgramCorpus.remove_external_corpus` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NgramCorpus.remove_external_corpus` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 92-93: NgramCorpus.list_external_corpora method
```python
    def list_external_corpora(self) -> Dict[str, int]:
        return self._obj.list_corpora()
```
**EN:** This block uses `NgramCorpus.list_external_corpora` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NgramCorpus.list_external_corpora` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 95-98: NgramCorpus.reset method
```python
    def reset(self):
        self._obj.reset()  # type: ignore
        self._req_id_to_state_id.clear()
        self._next_state_id = 0
```
**EN:** This block uses `NgramCorpus.reset` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NgramCorpus.reset` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 100-107: NgramCorpus.batch_get method
```python
    def batch_get(
        self,
        req_ids: List[str],
        batch_tokens: List[List[int]],
        total_lens: List[int],
    ) -> Tuple[np.ndarray, np.ndarray]:
        state_ids = [self._get_state_id(rid) for rid in req_ids]
        return self._obj.match_stateful(state_ids, batch_tokens, total_lens)
```
**EN:** This block uses `NgramCorpus.batch_get` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NgramCorpus.batch_get` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 109-116: NgramCorpus.erase_match_state method
```python
    def erase_match_state(self, req_ids: List[str]):
        state_ids = []
        for rid in req_ids:
            sid = self._req_id_to_state_id.pop(rid, None)
            if sid is not None:
                state_ids.append(sid)
        if state_ids:
            self._obj.erase_states(state_ids)
```
**EN:** This block uses `NgramCorpus.erase_match_state` to store configuration or metadata. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NgramCorpus.erase_match_state` 来存储配置或元数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 118-151: NgramCorpus.leaf_paths_from_mask method
```python
    def leaf_paths_from_mask(
        self, tokens: List[int], tree_mask: List[List[int]]
    ) -> List[List[int]]:
        """
        Find all leaf paths according to the binary tree_mask (i.e., paths that are not prefixes of any other path).

        Args:
            mask   : List[List[int]]   # nxn binary matrix
            tokens : List[int]         # token list corresponding to columns

        Returns:
            List[List[int]]            # token lists of only the leaf paths, preserving their order of appearance
        """

        row_sets = [
            (i, {idx for idx, v in enumerate(row) if v == 1})
            for i, row in enumerate(tree_mask)
        ]
        leaf_sets = []
        leaf_rows = []

        for i, cur_set in reversed(row_sets):
            if any(cur_set <= kept for kept in leaf_sets):
                continue
            leaf_sets.append(cur_set)
            leaf_rows.append(i)

        leaf_rows.reverse()
        result = []
        for r in leaf_rows:
            path = [tokens[col] for col in range(len(tokens)) if tree_mask[r][col] == 1]
            result.append(path)

        return result
```
**EN:** This block uses `NgramCorpus.leaf_paths_from_mask` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NgramCorpus.leaf_paths_from_mask` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 153-172: NgramCorpus.debug_result method
```python
    def debug_result(
        self, decoding_ids: np.ndarray, decoding_masks: np.ndarray, tokenizer=None
    ):
        decoding_ids = decoding_ids.reshape(-1, self.draft_token_num)
        decoding_masks = decoding_masks.reshape(
            -1, self.draft_token_num, self.draft_token_num
        )
        logger.info(f"\n{decoding_ids=}\n{decoding_masks=}")
        for i in range(decoding_ids.shape[0]):
            leaf_paths = self.leaf_paths_from_mask(
                decoding_ids[i].tolist(), decoding_masks[i].tolist()
            )
            if tokenizer is None:
                logger.info(f"draft path {i}: {leaf_paths}")
            else:
                logger.info(f"result {i}:")
                for leaf_path in leaf_paths:
                    logger.info(
                        f"draft path {i}: {leaf_path} -> {tokenizer.decode(leaf_path, ensure_ascii=False)}"
                    )
```
**EN:** This block uses `NgramCorpus.debug_result` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NgramCorpus.debug_result` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 173-200: Module-level supporting statements
```python


# main function
if __name__ == "__main__":
    format = f"%(levelname)s %(asctime)s %(filename)s:%(lineno)d] %(message)s"
    logging.basicConfig(
        level=logging.DEBUG,
        format=format,
        datefmt="%Y-%m-%d %H:%M:%S",
        force=True,
    )

    token_ids = [
        [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
        [1, 2, 3, 44, 55, 66, 77, 88, 99, 100],
    ]
    corpus = NgramCorpus(max_trie_depth=12, draft_token_num=8)
    corpus.batch_put(token_ids)

    corpus.synchronize()
    queries = [[1, 2, 3], [3, 44], [3, 6, 999]]
    decoding_ids, decoding_masks = corpus.batch_get(
        req_ids=[f"query-{i}" for i in range(len(queries))],
        batch_tokens=queries,
        total_lens=[len(q) for q in queries],
    )

    corpus.debug_result(decoding_ids, decoding_masks)
```
**EN:** This block contains supporting statements for the module, such as constants, attributes, registrations, or small helper expressions that shape the surrounding control flow.
**CN:** 该代码块包含模块的配套语句，例如常量、属性、注册逻辑或小型辅助表达式，它们共同塑造了周边控制流程。

## Key Concepts / 关键概念
- Speculative decoding / 推测解码
- N-gram drafting / N-gram 起草
- Corpus management / 语料管理

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.jit_kernel.ngram_corpus`
### External / 外部
- `numpy`
- `collections` (stdlib)
- `logging` (stdlib)
- `typing` (stdlib)
