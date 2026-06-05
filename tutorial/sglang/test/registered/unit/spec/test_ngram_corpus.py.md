# test_ngram_corpus.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/spec/test_ngram_corpus.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates ngram corpus behavior in SGLang's unit / spec area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 推测解码 领域中与 ngram corpus 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: module imports and dependencies / 模块导入与依赖
```python
import json
import os
import tempfile
import unittest
import uuid

import numpy as np

from sglang.srt.speculative.cpp_ngram.external_corpus import (
    iter_external_corpus_chunks,
)
from sglang.srt.speculative.cpp_ngram.ngram_corpus import NgramCorpus
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `os`, `tempfile`, `unittest`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `os`, `tempfile`, `unittest`。

### Lines 16-16: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=26, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 19-46: function make corpus / 函数 make corpus
```python
def _make_corpus(match_type="BFS", **kwargs):
    external_corpus_documents = kwargs.pop("external_corpus_documents", None)
    defaults = dict(
        max_trie_depth=12,
        min_bfs_breadth=1,
        max_bfs_breadth=8,
        draft_token_num=8,
        capacity=100000,
        external_sam_budget=0,
        external_corpus_max_tokens=10000000,
    )
    defaults.update(kwargs)
    defaults["match_type"] = match_type
    corpus = NgramCorpus(**defaults)
    if external_corpus_documents is not None:
        from sglang.srt.speculative.cpp_ngram.external_corpus import SEPARATOR_TOKEN

        chunks = []
        has_prev = False
        for doc in external_corpus_documents:
            if has_prev:
                chunks.append([SEPARATOR_TOKEN] + list(doc))
            else:
                chunks.append(list(doc))
            has_prev = True
        loaded_token_count = corpus.load_external_corpus_named("test_corpus", chunks)
        corpus.commit_external_corpus_load("test_corpus", loaded_token_count)
    return corpus
```
**EN:** This block implements `_make_corpus` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_corpus`，承担模块行为中的一个聚焦逻辑片段。

### Lines 49-57: function batch get / 函数 batch get
```python
def _batch_get(
    corpus: NgramCorpus,
    batch_tokens: list[list[int]],
):
    return corpus.batch_get(
        req_ids=[uuid.uuid4().hex for _ in range(len(batch_tokens))],
        batch_tokens=batch_tokens,
        total_lens=[len(tokens) for tokens in batch_tokens],
    )
```
**EN:** This block implements `_batch_get` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_batch_get`，承担模块行为中的一个聚焦逻辑片段。

### Lines 60-66: function batch get with state / 函数 batch get with state
```python
def _batch_get_with_state(
    corpus: NgramCorpus,
    req_id: str,
    current_tokens: list[int],
    total_len: int,
):
    return corpus.batch_get([req_id], [current_tokens], [total_len])
```
**EN:** This block implements `_batch_get_with_state` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_batch_get_with_state`，承担模块行为中的一个聚焦逻辑片段。

### Lines 69-69: class _IntTokenizer declaration / 类 _IntTokenizer 声明
```python
class _IntTokenizer:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 70-72: method encode / 方法 encode
```python
    def encode(self, text: str, add_special_tokens: bool = False):
        del add_special_tokens
        return [int(piece) for piece in text.split()]
```
**EN:** This block implements `encode` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `encode`，承担模块行为中的一个聚焦逻辑片段。

### Lines 75-154: module-level constants and configuration (part 1/2) / 模块级常量与配置（第 1/2 部分）
```python
SEED_SEQUENCES = [
    [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
    [1, 2, 3, 44, 55, 66, 77, 88, 99, 100],
]

QUERY_SEQUENCES = [[1, 2, 3], [3, 44], [3, 6, 999]]

EXPECTED_BFS_IDS = [
    [3, 4, 44, 5, 55, 6, 66, 77],
    [44, 55, 66, 77, 88, 99, 100, 0],
    [999, 0, 0, 0, 0, 0, 0, 0],
]

EXPECTED_PROB_IDS = [
    [3, 44, 4, 55, 5, 66, 6, 7],
    [44, 55, 66, 77, 88, 99, 100, 0],
    [999, 0, 0, 0, 0, 0, 0, 0],
]

EXPECTED_BFS_MASKS = [
    [
        [1, 0, 0, 0, 0, 0, 0, 0],
        [1, 1, 0, 0, 0, 0, 0, 0],
        [1, 0, 1, 0, 0, 0, 0, 0],
        [1, 1, 0, 1, 0, 0, 0, 0],
        [1, 0, 1, 0, 1, 0, 0, 0],
        [1, 1, 0, 1, 0, 1, 0, 0],
        [1, 0, 1, 0, 1, 0, 1, 0],
        [1, 0, 1, 0, 1, 0, 1, 1],
    ],
    [
        [1, 0, 0, 0, 0, 0, 0, 0],
        [1, 1, 0, 0, 0, 0, 0, 0],
        [1, 1, 1, 0, 0, 0, 0, 0],
        [1, 1, 1, 1, 0, 0, 0, 0],
        [1, 1, 1, 1, 1, 0, 0, 0],
        [1, 1, 1, 1, 1, 1, 0, 0],
        [1, 1, 1, 1, 1, 1, 1, 0],
        [1, 0, 0, 0, 0, 0, 0, 1],
    ],
    [
        [1, 0, 0, 0, 0, 0, 0, 0],
        [1, 1, 0, 0, 0, 0, 0, 0],
        [1, 0, 1, 0, 0, 0, 0, 0],
        [1, 0, 0, 1, 0, 0, 0, 0],
        [1, 0, 0, 0, 1, 0, 0, 0],
        [1, 0, 0, 0, 0, 1, 0, 0],
        [1, 0, 0, 0, 0, 0, 1, 0],
        [1, 0, 0, 0, 0, 0, 0, 1],
    ],
]

EXPECTED_PROB_MASKS = [
    [
        [1, 0, 0, 0, 0, 0, 0, 0],
        [1, 1, 0, 0, 0, 0, 0, 0],
        [1, 0, 1, 0, 0, 0, 0, 0],
        [1, 1, 0, 1, 0, 0, 0, 0],
        [1, 0, 1, 0, 1, 0, 0, 0],
        [1, 1, 0, 1, 0, 1, 0, 0],
        [1, 0, 1, 0, 1, 0, 1, 0],
        [1, 0, 1, 0, 1, 0, 1, 1],
    ],
    [
        [1, 0, 0, 0, 0, 0, 0, 0],
        [1, 1, 0, 0, 0, 0, 0, 0],
        [1, 1, 1, 0, 0, 0, 0, 0],
        [1, 1, 1, 1, 0, 0, 0, 0],
        [1, 1, 1, 1, 1, 0, 0, 0],
        [1, 1, 1, 1, 1, 1, 0, 0],
        [1, 1, 1, 1, 1, 1, 1, 0],
        [1, 0, 0, 0, 0, 0, 0, 1],
    ],
    [
        [1, 0, 0, 0, 0, 0, 0, 0],
        [1, 1, 0, 0, 0, 0, 0, 0],
        [1, 0, 1, 0, 0, 0, 0, 0],
        [1, 0, 0, 1, 0, 0, 0, 0],
        [1, 0, 0, 0, 1, 0, 0, 0],
        [1, 0, 0, 0, 0, 1, 0, 0],
```
**EN:** This block defines shared names such as `SEED_SEQUENCES`, `QUERY_SEQUENCES`, `EXPECTED_BFS_IDS`, `EXPECTED_PROB_IDS`, `EXPECTED_BFS_MASKS`. These values centralize configuration that later tests or helpers reuse. This chunk continues part 1 of the same logical block.
**CN:** 该代码块定义了 `SEED_SEQUENCES`, `QUERY_SEQUENCES`, `EXPECTED_BFS_IDS`, `EXPECTED_PROB_IDS`, `EXPECTED_BFS_MASKS` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。 这一段对应同一逻辑块的第 1 部分。

### Lines 155-158: module-level constants and configuration (part 2/2) / 模块级常量与配置（第 2/2 部分）
```python
        [1, 0, 0, 0, 0, 0, 1, 0],
        [1, 0, 0, 0, 0, 0, 0, 1],
    ],
]
```
**EN:** This block defines shared names such as `SEED_SEQUENCES`, `QUERY_SEQUENCES`, `EXPECTED_BFS_IDS`, `EXPECTED_PROB_IDS`, `EXPECTED_BFS_MASKS`. These values centralize configuration that later tests or helpers reuse. This chunk continues part 2 of the same logical block.
**CN:** 该代码块定义了 `SEED_SEQUENCES`, `QUERY_SEQUENCES`, `EXPECTED_BFS_IDS`, `EXPECTED_PROB_IDS`, `EXPECTED_BFS_MASKS` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。 这一段对应同一逻辑块的第 2 部分。

### Lines 161-161: class TestNgramCorpusBFS declaration / 类 TestNgramCorpusBFS 声明
```python
class TestNgramCorpusBFS(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 162-162: supporting statements / 辅助语句
```python
    """Golden-output tests for BFS matching mode."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 164-172: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.corpus = _make_corpus("BFS")
        cls.corpus.batch_put(SEED_SEQUENCES)
        cls.corpus.synchronize()
        ids, masks = _batch_get(cls.corpus, QUERY_SEQUENCES)
        draft = 8
        cls.ids = ids.reshape(-1, draft)
        cls.masks = masks.reshape(-1, draft, draft)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 174-175: test case token ids / 测试用例 token ids
```python
    def test_token_ids(self):
        np.testing.assert_array_equal(self.ids.tolist(), EXPECTED_BFS_IDS)
```
**EN:** This test exercises `test_token_ids` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_token_ids`。

### Lines 177-178: test case masks / 测试用例 masks
```python
    def test_masks(self):
        np.testing.assert_array_equal(self.masks.tolist(), EXPECTED_BFS_MASKS)
```
**EN:** This test exercises `test_masks` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_masks`。

### Lines 180-184: test case output shapes / 测试用例 output shapes
```python
    def test_output_shapes(self):
        n_queries = len(QUERY_SEQUENCES)
        draft = 8
        self.assertEqual(self.ids.shape, (n_queries, draft))
        self.assertEqual(self.masks.shape, (n_queries, draft, draft))
```
**EN:** This test exercises `test_output_shapes` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_output_shapes`。

### Lines 187-187: class TestNgramCorpusProb declaration / 类 TestNgramCorpusProb 声明
```python
class TestNgramCorpusProb(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 188-188: supporting statements / 辅助语句
```python
    """Golden-output tests for Prob matching mode."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 190-197: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.corpus = _make_corpus("PROB")
        cls.corpus.batch_put(SEED_SEQUENCES)
        cls.corpus.synchronize()
        ids, masks = _batch_get(cls.corpus, QUERY_SEQUENCES)
        cls.ids = ids.reshape(-1, 8)
        cls.masks = masks.reshape(-1, 8, 8)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 199-200: test case token ids / 测试用例 token ids
```python
    def test_token_ids(self):
        np.testing.assert_array_equal(self.ids.tolist(), EXPECTED_PROB_IDS)
```
**EN:** This test exercises `test_token_ids` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_token_ids`。

### Lines 202-203: test case masks / 测试用例 masks
```python
    def test_masks(self):
        np.testing.assert_array_equal(self.masks.tolist(), EXPECTED_PROB_MASKS)
```
**EN:** This test exercises `test_masks` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_masks`。

### Lines 205-208: test case output shapes / 测试用例 output shapes
```python
    def test_output_shapes(self):
        n_queries = len(QUERY_SEQUENCES)
        self.assertEqual(self.ids.shape, (n_queries, 8))
        self.assertEqual(self.masks.shape, (n_queries, 8, 8))
```
**EN:** This test exercises `test_output_shapes` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_output_shapes`。

### Lines 211-211: class TestNgramCorpusReset declaration / 类 TestNgramCorpusReset 声明
```python
class TestNgramCorpusReset(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 212-212: supporting statements / 辅助语句
```python
    """Verify reset clears all cached state."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 214-232: test case reset produces empty results / 测试用例 reset produces empty results
```python
    def test_reset_produces_empty_results(self):
        corpus = _make_corpus("BFS")
        corpus.batch_put(SEED_SEQUENCES)
        corpus.synchronize()

        ids_before, _ = _batch_get(corpus, [[1, 2, 3]])
        self.assertTrue(
            any(t != 0 for t in ids_before.tolist()[1:]),
            "Expected non-trivial draft tokens before reset",
        )

        corpus.reset()

        ids_after, _ = _batch_get(corpus, [[1, 2, 3]])
        self.assertEqual(
            ids_after.tolist(),
            [3, 0, 0, 0, 0, 0, 0, 0],
            "After reset, only last_token should be present (rest zero-padded)",
        )
```
**EN:** This test exercises `test_reset_produces_empty_results` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reset_produces_empty_results`。

### Lines 235-235: class TestNgramCorpusNoMatch declaration / 类 TestNgramCorpusNoMatch 声明
```python
class TestNgramCorpusNoMatch(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 236-236: supporting statements / 辅助语句
```python
    """Verify behavior when query has no match in the corpus."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 238-249: test case unmatched query / 测试用例 unmatched query
```python
    def test_unmatched_query(self):
        corpus = _make_corpus("BFS")
        corpus.batch_put([[10, 20, 30, 40, 50]])
        corpus.synchronize()

        ids, masks = _batch_get(corpus, [[999, 888, 777]])
        ids_list = ids.tolist()
        self.assertEqual(ids_list[0], 777, "First token should be last context token")
        self.assertTrue(
            all(t == 0 for t in ids_list[1:]),
            "No draft tokens expected when nothing matches",
        )
```
**EN:** This test exercises `test_unmatched_query` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unmatched_query`。

### Lines 251-256: test case empty corpus / 测试用例 empty corpus
```python
    def test_empty_corpus(self):
        corpus = _make_corpus("BFS")
        ids, masks = _batch_get(corpus, [[1, 2, 3]])
        ids_list = ids.tolist()
        self.assertEqual(ids_list[0], 3)
        self.assertTrue(all(t == 0 for t in ids_list[1:]))
```
**EN:** This test exercises `test_empty_corpus` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_corpus`。

### Lines 259-259: class TestNgramCorpusMultipleInserts declaration / 类 TestNgramCorpusMultipleInserts 声明
```python
class TestNgramCorpusMultipleInserts(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 260-260: supporting statements / 辅助语句
```python
    """Verify that multiple inserts accumulate correctly."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 262-274: test case incremental inserts / 测试用例 incremental inserts
```python
    def test_incremental_inserts(self):
        corpus = _make_corpus("BFS")
        corpus.batch_put([[1, 2, 3, 4, 5]])
        corpus.synchronize()

        corpus.batch_put([[1, 2, 3, 44, 55]])
        corpus.synchronize()

        ids, _ = _batch_get(corpus, [[1, 2, 3]])
        ids_list = ids.tolist()

        self.assertIn(4, ids_list, "Token 4 from first insert should still match")
        self.assertIn(44, ids_list, "Token 44 from second insert should also match")
```
**EN:** This test exercises `test_incremental_inserts` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_incremental_inserts`。

### Lines 277-277: class TestNgramCorpusSqueeze declaration / 类 TestNgramCorpusSqueeze 声明
```python
class TestNgramCorpusSqueeze(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 278-278: supporting statements / 辅助语句
```python
    """Verify cache eviction under memory pressure."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 280-287: test case small capacity does not crash / 测试用例 small capacity does not crash
```python
    def test_small_capacity_does_not_crash(self):
        corpus = _make_corpus("BFS", capacity=200)
        long_seq = list(range(1, 101))
        corpus.batch_put([long_seq])
        corpus.synchronize()

        ids, masks = _batch_get(corpus, [[50, 51, 52]])
        self.assertEqual(len(ids), 8, "Should still produce draft_token_num outputs")
```
**EN:** This test exercises `test_small_capacity_does_not_crash` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_small_capacity_does_not_crash`。

### Lines 289-303: test case eviction preserves recent / 测试用例 eviction preserves recent
```python
    def test_eviction_preserves_recent(self):
        corpus = _make_corpus("BFS", capacity=500, max_trie_depth=6)

        old_seq = list(range(1000, 1050))
        corpus.batch_put([old_seq])
        corpus.synchronize()

        recent_seq = list(range(2000, 2050))
        corpus.batch_put([recent_seq])
        corpus.synchronize()

        ids, _ = _batch_get(corpus, [[2000, 2001, 2002]])
        ids_list = ids.tolist()
        self.assertEqual(ids_list[0], 2002, "Last context token should be first")
        self.assertIn(2003, ids_list, "Recent sequence should still be matchable")
```
**EN:** This test exercises `test_eviction_preserves_recent` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_eviction_preserves_recent`。

### Lines 306-306: class TestNgramCorpusLeafPaths declaration / 类 TestNgramCorpusLeafPaths 声明
```python
class TestNgramCorpusLeafPaths(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 307-307: supporting statements / 辅助语句
```python
    """Verify the leaf_paths_from_mask utility."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 309-324: test case simple tree / 测试用例 simple tree
```python
    def test_simple_tree(self):
        corpus = _make_corpus("BFS")
        tokens = [3, 4, 44, 5, 55]
        mask = [
            [1, 0, 0, 0, 0],
            [1, 1, 0, 0, 0],
            [1, 0, 1, 0, 0],
            [1, 1, 0, 1, 0],
            [1, 0, 1, 0, 1],
        ]
        paths = corpus.leaf_paths_from_mask(tokens, mask)

        for path in paths:
            self.assertIn(3, path, "Root token should be in every path")

        self.assertEqual(len(paths), 2, "Two leaf paths expected for a binary tree")
```
**EN:** This test exercises `test_simple_tree` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_simple_tree`。

### Lines 326-336: test case single chain / 测试用例 single chain
```python
    def test_single_chain(self):
        corpus = _make_corpus("BFS")
        tokens = [10, 20, 30]
        mask = [
            [1, 0, 0],
            [1, 1, 0],
            [1, 1, 1],
        ]
        paths = corpus.leaf_paths_from_mask(tokens, mask)
        self.assertEqual(len(paths), 1)
        self.assertEqual(paths[0], [10, 20, 30])
```
**EN:** This test exercises `test_single_chain` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_chain`。

### Lines 339-339: class TestNgramCorpusBatchConsistency declaration / 类 TestNgramCorpusBatchConsistency 声明
```python
class TestNgramCorpusBatchConsistency(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 340-340: supporting statements / 辅助语句
```python
    """Verify batch queries produce same results as individual queries."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 342-366: test case batch vs individual / 测试用例 batch vs individual
```python
    def test_batch_vs_individual(self):
        corpus = _make_corpus("BFS")
        corpus.batch_put(SEED_SEQUENCES)
        corpus.synchronize()

        batch_ids, batch_masks = _batch_get(corpus, QUERY_SEQUENCES)
        draft = 8
        batch_ids = batch_ids.reshape(-1, draft)
        batch_masks = batch_masks.reshape(-1, draft, draft)

        for i, query in enumerate(QUERY_SEQUENCES):
            single_ids, single_masks = _batch_get(corpus, [query])
            single_ids = single_ids.reshape(-1, draft)
            single_masks = single_masks.reshape(-1, draft, draft)

            np.testing.assert_array_equal(
                batch_ids[i],
                single_ids[0],
                err_msg=f"Token mismatch for query {i}",
            )
            np.testing.assert_array_equal(
                batch_masks[i],
                single_masks[0],
                err_msg=f"Mask mismatch for query {i}",
            )
```
**EN:** This test exercises `test_batch_vs_individual` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_batch_vs_individual`。

### Lines 369-369: class TestMaskValidity declaration / 类 TestMaskValidity 声明
```python
class TestMaskValidity(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 370-370: supporting statements / 辅助语句
```python
    """Verify structural invariants of the output mask for any draft tree."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 372-376: method check mask / 方法 check mask
```python
    def _check_mask(self, masks_2d):
        n = len(masks_2d)
        for i in range(n):
            self.assertEqual(masks_2d[i][i], 1, f"Diagonal must be 1 at row {i}")
        self.assertEqual(masks_2d[0], [1] + [0] * (n - 1))
```
**EN:** This block implements `_check_mask` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_check_mask`，承担模块行为中的一个聚焦逻辑片段。

### Lines 378-385: test case bfs mask invariants / 测试用例 bfs mask invariants
```python
    def test_bfs_mask_invariants(self):
        corpus = _make_corpus("BFS")
        corpus.batch_put(SEED_SEQUENCES)
        corpus.synchronize()
        _, masks = _batch_get(corpus, QUERY_SEQUENCES)
        masks = masks.reshape(-1, 8, 8)
        for i in range(masks.shape[0]):
            self._check_mask(masks[i].tolist())
```
**EN:** This test exercises `test_bfs_mask_invariants` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bfs_mask_invariants`。

### Lines 387-394: test case prob mask invariants / 测试用例 prob mask invariants
```python
    def test_prob_mask_invariants(self):
        corpus = _make_corpus("PROB")
        corpus.batch_put(SEED_SEQUENCES)
        corpus.synchronize()
        _, masks = _batch_get(corpus, QUERY_SEQUENCES)
        masks = masks.reshape(-1, 8, 8)
        for i in range(masks.shape[0]):
            self._check_mask(masks[i].tolist())
```
**EN:** This test exercises `test_prob_mask_invariants` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prob_mask_invariants`。

### Lines 397-397: class TestFrequencyBoosting declaration / 类 TestFrequencyBoosting 声明
```python
class TestFrequencyBoosting(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 398-398: supporting statements / 辅助语句
```python
    """Verify that repeated insertions change Prob-mode selection."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 400-422: test case repeated insert promotes token / 测试用例 repeated insert promotes token
```python
    def test_repeated_insert_promotes_token(self):
        corpus = _make_corpus(
            "PROB",
            draft_token_num=2,
            max_bfs_breadth=1,
            min_bfs_breadth=1,
            max_trie_depth=5,
        )
        corpus.batch_put([[1, 2, 3, 10, 11]])
        corpus.synchronize()

        for _ in range(10):
            corpus.batch_put([[1, 2, 3, 20, 21]])
        corpus.synchronize()

        ids, _ = _batch_get(corpus, [[1, 2, 3]])
        ids_list = ids.tolist()

        self.assertEqual(
            ids_list[1],
            20,
            f"Token 20 should be selected over 10 after frequency boost, got {ids_list}",
        )
```
**EN:** This test exercises `test_repeated_insert_promotes_token` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_repeated_insert_promotes_token`。

### Lines 425-425: class TestRecencyOrdering declaration / 类 TestRecencyOrdering 声明
```python
class TestRecencyOrdering(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 426-426: supporting statements / 辅助语句
```python
    """Verify that BFS mode respects LRU recency."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 428-447: test case most recent insert selected / 测试用例 most recent insert selected
```python
    def test_most_recent_insert_selected(self):
        corpus = _make_corpus(
            "BFS",
            draft_token_num=2,
            max_bfs_breadth=1,
            min_bfs_breadth=1,
            max_trie_depth=5,
        )
        corpus.batch_put([[1, 2, 3, 10, 11]])
        corpus.synchronize()
        corpus.batch_put([[1, 2, 3, 20, 21]])
        corpus.synchronize()

        ids, _ = _batch_get(corpus, [[1, 2, 3]])
        ids_list = ids.tolist()
        self.assertEqual(
            ids_list[1],
            20,
            f"Token 20 (recent) should be selected over 10 (old), got {ids_list}",
        )
```
**EN:** This test exercises `test_most_recent_insert_selected` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_most_recent_insert_selected`。

### Lines 450-450: class TestOverlappingSuffixes declaration / 类 TestOverlappingSuffixes 声明
```python
class TestOverlappingSuffixes(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 451-451: supporting statements / 辅助语句
```python
    """Verify correct matching when sequences share suffixes."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 453-462: test case shared suffix both match / 测试用例 shared suffix both match
```python
    def test_shared_suffix_both_match(self):
        corpus = _make_corpus("BFS")
        corpus.batch_put([[100, 200, 7, 8, 9, 50, 51]])
        corpus.batch_put([[300, 400, 7, 8, 9, 60, 61]])
        corpus.synchronize()

        ids, _ = _batch_get(corpus, [[7, 8, 9]])
        ids_list = ids.tolist()
        self.assertIn(50, ids_list, "Continuation from first sequence missing")
        self.assertIn(60, ids_list, "Continuation from second sequence missing")
```
**EN:** This test exercises `test_shared_suffix_both_match` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_shared_suffix_both_match`。

### Lines 465-465: class TestSingleTokenContext declaration / 类 TestSingleTokenContext 声明
```python
class TestSingleTokenContext(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 466-466: supporting statements / 辅助语句
```python
    """Verify behavior with minimum-length context."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 468-476: test case single token query / 测试用例 single token query
```python
    def test_single_token_query(self):
        corpus = _make_corpus("BFS")
        corpus.batch_put([[5, 10, 20, 30]])
        corpus.synchronize()

        ids, masks = _batch_get(corpus, [[5]])
        ids_list = ids.tolist()
        self.assertEqual(ids_list[0], 5, "First token should be last context token")
        self.assertIn(10, ids_list, "Should match continuation after single token 5")
```
**EN:** This test exercises `test_single_token_query` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_token_query`。

### Lines 479-479: class TestLongContext declaration / 类 TestLongContext 声明
```python
class TestLongContext(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 480-480: supporting statements / 辅助语句
```python
    """Verify behavior when query context exceeds max_trie_depth."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 482-492: test case context longer than max trie depth / 测试用例 context longer than max trie depth
```python
    def test_context_longer_than_max_trie_depth(self):
        corpus = _make_corpus("BFS", max_trie_depth=6)
        seq = list(range(1, 20))
        corpus.batch_put([seq])
        corpus.synchronize()

        long_query = list(range(1, 16))
        ids, masks = _batch_get(corpus, [long_query])
        ids_list = ids.tolist()
        self.assertEqual(ids_list[0], 15, "First token should be last context token")
        self.assertIn(16, ids_list, "Should match via suffix despite long context")
```
**EN:** This test exercises `test_context_longer_than_max_trie_depth` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_context_longer_than_max_trie_depth`。

### Lines 494-509: test case matches longest stored suffix / 测试用例 matches longest stored suffix
```python
    def test_matches_longest_stored_suffix(self):
        corpus = _make_corpus("BFS", max_trie_depth=6, draft_token_num=4)
        corpus.batch_put([[1, 2, 3, 4, 5, 6, 7]])
        corpus.batch_put([[99, 3, 4, 5, 6, 8]])
        corpus.synchronize()

        ids, _ = _batch_get(corpus, [[2, 3, 4, 5, 6]])
        ids_list = ids.tolist()
        self.assertIn(
            7, ids_list, "Longest stored suffix should contribute a continuation"
        )
        self.assertIn(
            8,
            ids_list,
            "Shorter matching suffixes should still contribute continuations",
        )
```
**EN:** This test exercises `test_matches_longest_stored_suffix` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_matches_longest_stored_suffix`。

### Lines 512-512: class TestDraftBudgetSaturation declaration / 类 TestDraftBudgetSaturation 声明
```python
class TestDraftBudgetSaturation(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 513-513: supporting statements / 辅助语句
```python
    """Verify the draft tree uses exactly draft_token_num slots."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 515-529: test case full budget used / 测试用例 full budget used
```python
    def test_full_budget_used(self):
        corpus = _make_corpus("BFS", draft_token_num=8)
        seq = list(range(1, 30))
        corpus.batch_put([seq])
        corpus.synchronize()

        ids, _ = _batch_get(corpus, [[1, 2, 3]])
        ids_list = ids.tolist()
        self.assertEqual(len(ids_list), 8)
        non_zero = [t for t in ids_list[1:] if t != 0]
        self.assertGreater(
            len(non_zero),
            0,
            "Draft budget should have non-zero tokens when cache has long chains",
        )
```
**EN:** This test exercises `test_full_budget_used` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_full_budget_used`。

### Lines 532-532: class TestTruncate declaration / 类 TestTruncate 声明
```python
class TestTruncate(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 533-533: supporting statements / 辅助语句
```python
    """Verify truncation logic on batch_get output."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 535-547: test case truncate reduces output / 测试用例 truncate reduces output
```python
    def test_truncate_reduces_output(self):
        corpus = _make_corpus("BFS", draft_token_num=8)
        corpus.batch_put(SEED_SEQUENCES)
        corpus.synchronize()

        ids, _ = _batch_get(corpus, [[1, 2, 3]])
        ids = ids.reshape(8)
        self.assertEqual(len(ids), 8)

        # Simulate truncate to 4
        trunc_n = 4
        trunc_ids = ids[:trunc_n]
        self.assertEqual(len(trunc_ids), trunc_n)
```
**EN:** This test exercises `test_truncate_reduces_output` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_truncate_reduces_output`。

### Lines 549-567: test case truncate preserves mask structure / 测试用例 truncate preserves mask structure
```python
    def test_truncate_preserves_mask_structure(self):
        corpus = _make_corpus("BFS", draft_token_num=8)
        corpus.batch_put(SEED_SEQUENCES)
        corpus.synchronize()

        _, masks = _batch_get(corpus, [[1, 2, 3]])
        n = 8
        full_mask = masks.reshape(n, n)

        trunc_n = 4
        trunc_mask = full_mask[:trunc_n, :trunc_n]

        for i in range(trunc_n):
            for j in range(trunc_n):
                self.assertEqual(
                    trunc_mask[i, j],
                    full_mask[i, j],
                    f"Mask mismatch at ({i},{j})",
                )
```
**EN:** This test exercises `test_truncate_preserves_mask_structure` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_truncate_preserves_mask_structure`。

### Lines 570-570: class TestResetAndReinsert declaration / 类 TestResetAndReinsert 声明
```python
class TestResetAndReinsert(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 571-571: supporting statements / 辅助语句
```python
    """Verify that reset followed by new inserts works correctly."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 573-593: test case reset then reinsert / 测试用例 reset then reinsert
```python
    def test_reset_then_reinsert(self):
        corpus = _make_corpus("BFS")
        corpus.batch_put([[1, 2, 3, 4, 5]])
        corpus.synchronize()

        corpus.reset()

        corpus.batch_put([[10, 20, 30, 40, 50]])
        corpus.synchronize()

        ids_old, _ = _batch_get(corpus, [[1, 2, 3]])
        ids_old_list = ids_old.tolist()
        self.assertTrue(
            all(t == 0 for t in ids_old_list[1:]),
            f"Old data should not match after reset+reinsert, got {ids_old_list}",
        )

        ids_new, _ = _batch_get(corpus, [[10, 20, 30]])
        ids_new_list = ids_new.tolist()
        self.assertEqual(ids_new_list[0], 30)
        self.assertIn(40, ids_new_list, "New data should match after reset+reinsert")
```
**EN:** This test exercises `test_reset_then_reinsert` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reset_then_reinsert`。

### Lines 596-596: class TestSqueezeEvictsOld declaration / 类 TestSqueezeEvictsOld 声明
```python
class TestSqueezeEvictsOld(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 597-597: supporting statements / 辅助语句
```python
    """Verify that squeeze actually evicts old data, not just preserves recent."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 599-624: test case old data evicted / 测试用例 old data evicted
```python
    def test_old_data_evicted(self):
        corpus = _make_corpus("BFS", capacity=150, max_trie_depth=6)

        old_seq = list(range(5000, 5030))
        corpus.batch_put([old_seq])
        corpus.synchronize()

        ids_before, _ = _batch_get(corpus, [[5000, 5001, 5002]])
        self.assertIn(
            5003,
            ids_before.tolist(),
            "Old data should match before eviction",
        )

        for i in range(5):
            new_seq = list(range(6000 + i * 30, 6000 + i * 30 + 30))
            corpus.batch_put([new_seq])
            corpus.synchronize()

        ids_after, _ = _batch_get(corpus, [[5000, 5001, 5002]])
        ids_after_list = ids_after.tolist()
        self.assertNotIn(
            5003,
            ids_after_list,
            f"Old data should be evicted after pressure, got {ids_after_list}",
        )
```
**EN:** This test exercises `test_old_data_evicted` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_old_data_evicted`。

### Lines 627-627: class TestNgramCorpusIncremental declaration / 类 TestNgramCorpusIncremental 声明
```python
class TestNgramCorpusIncremental(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 628-628: supporting statements / 辅助语句
```python
    """Verify the incremental matching path matches the stateless path."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 630-652: method assert incremental matches stateless / 方法 assert incremental matches stateless
```python
    def _assert_incremental_matches_stateless(self, match_type: str):
        corpus = _make_corpus(match_type, max_trie_depth=4, draft_token_num=4)
        corpus.batch_put([[1, 2, 3, 4, 5, 6], [9, 3, 4, 7, 8]])
        corpus.synchronize()

        req_id = f"req-{match_type.lower()}"

        steps = [
            [1, 2, 3],
            [1, 2, 3, 4],
            [1, 2, 3, 4, 5, 6],
        ]
        for full_sequence in steps:
            current_tail = full_sequence[-4:]
            inc_ids, inc_masks = _batch_get_with_state(
                corpus,
                req_id,
                current_tail,
                len(full_sequence),
            )
            full_ids, full_masks = _batch_get(corpus, [current_tail])
            np.testing.assert_array_equal(inc_ids, full_ids)
            np.testing.assert_array_equal(inc_masks, full_masks)
```
**EN:** This block implements `_assert_incremental_matches_stateless` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_assert_incremental_matches_stateless`，承担模块行为中的一个聚焦逻辑片段。

### Lines 654-655: test case incremental matches stateless bfs / 测试用例 incremental matches stateless bfs
```python
    def test_incremental_matches_stateless_bfs(self):
        self._assert_incremental_matches_stateless("BFS")
```
**EN:** This test exercises `test_incremental_matches_stateless_bfs` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_incremental_matches_stateless_bfs`。

### Lines 657-658: test case incremental matches stateless prob / 测试用例 incremental matches stateless prob
```python
    def test_incremental_matches_stateless_prob(self):
        self._assert_incremental_matches_stateless("PROB")
```
**EN:** This test exercises `test_incremental_matches_stateless_prob` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_incremental_matches_stateless_prob`。

### Lines 660-683: test case leaf anchor becomes expandable / 测试用例 leaf anchor becomes expandable
```python
    def test_leaf_anchor_becomes_expandable(self):
        corpus = _make_corpus("BFS", max_trie_depth=4, draft_token_num=4)
        corpus.batch_put([[1, 2, 3]])
        corpus.synchronize()

        req_id = "leaf-anchor"
        ids_before, _ = _batch_get_with_state(corpus, req_id, [2, 3], 2)
        self.assertTrue(
            all(t == 0 for t in ids_before.tolist()[1:]),
            f"Expected only the last token before extension, got {ids_before.tolist()}",
        )

        corpus.batch_put([[9, 2, 3, 4]])
        corpus.synchronize()

        inc_ids, inc_masks = _batch_get_with_state(corpus, req_id, [2, 3], 2)
        full_ids, full_masks = _batch_get(corpus, [[2, 3]])
        np.testing.assert_array_equal(inc_ids, full_ids)
        np.testing.assert_array_equal(inc_masks, full_masks)
        self.assertIn(
            4,
            inc_ids.tolist(),
            f"Expected token 4 after extension, got {inc_ids.tolist()}",
        )
```
**EN:** This test exercises `test_leaf_anchor_becomes_expandable` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_leaf_anchor_becomes_expandable`。

### Lines 685-703: test case stale state rebuilds after eviction / 测试用例 stale state rebuilds after eviction
```python
    def test_stale_state_rebuilds_after_eviction(self):
        corpus = _make_corpus("BFS", capacity=150, max_trie_depth=6, draft_token_num=4)
        corpus.batch_put([list(range(5000, 5030))])
        corpus.synchronize()

        req_id = "evicted"
        _batch_get_with_state(corpus, req_id, [5000, 5001, 5002], 3)

        for i in range(5):
            new_seq = list(range(6000 + i * 30, 6000 + i * 30 + 30))
            corpus.batch_put([new_seq])
            corpus.synchronize()

        inc_ids, inc_masks = _batch_get_with_state(
            corpus, req_id, [5000, 5001, 5002], 3
        )
        full_ids, full_masks = _batch_get(corpus, [[5000, 5001, 5002]])
        np.testing.assert_array_equal(inc_ids, full_ids)
        np.testing.assert_array_equal(inc_masks, full_masks)
```
**EN:** This test exercises `test_stale_state_rebuilds_after_eviction` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stale_state_rebuilds_after_eviction`。

### Lines 706-706: class TestNgramCorpusExternalSam declaration / 类 TestNgramCorpusExternalSam 声明
```python
class TestNgramCorpusExternalSam(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 707-707: supporting statements / 辅助语句
```python
    """Verify external SAM loading and fixed-budget composition."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 709-735: test case external corpus iterator streams documents / 测试用例 external corpus iterator streams documents
```python
    def test_external_corpus_iterator_streams_documents(self):
        corpus = _make_corpus(
            "BFS",
            draft_token_num=4,
            external_sam_budget=3,
            external_corpus_max_tokens=8,
        )
        with tempfile.NamedTemporaryFile(mode="w", suffix=".jsonl", delete=False) as f:
            f.write(json.dumps("1 2 3 4 5"))
            f.write("\n")
            f.write(json.dumps("8 9"))
            f.write("\n")
            path = f.name
        self.addCleanup(os.remove, path)

        loaded_token_count = corpus.load_external_corpus_named(
            path,
            iter_external_corpus_chunks(path, _IntTokenizer(), max_tokens=8),
        )
        corpus.commit_external_corpus_load(path, loaded_token_count)
        # 5 doc tokens + 1 separator + 2 doc tokens = 8
        self.assertEqual(loaded_token_count, 8)

        ids, _ = _batch_get(corpus, [[1, 2, 3]])
        ids_list = ids.tolist()
        self.assertEqual(ids_list[0], 3)
        self.assertEqual(ids_list[1:3], [4, 5])
```
**EN:** This test exercises `test_external_corpus_iterator_streams_documents` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_external_corpus_iterator_streams_documents`。

### Lines 737-755: test case external corpus iterator rejects oversized corpus / 测试用例 external corpus iterator rejects oversized corpus
```python
    def test_external_corpus_iterator_rejects_oversized_corpus(self):
        corpus = _make_corpus(
            "BFS",
            external_sam_budget=2,
            external_corpus_max_tokens=4,
        )
        with tempfile.NamedTemporaryFile(mode="w", suffix=".jsonl", delete=False) as f:
            f.write(json.dumps("1 2 3"))
            f.write("\n")
            f.write(json.dumps("4 5"))
            f.write("\n")
            path = f.name
        self.addCleanup(os.remove, path)

        with self.assertRaisesRegex(ValueError, "token limit"):
            corpus.load_external_corpus_named(
                path,
                iter_external_corpus_chunks(path, _IntTokenizer(), max_tokens=4),
            )
```
**EN:** This test exercises `test_external_corpus_iterator_rejects_oversized_corpus` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_external_corpus_iterator_rejects_oversized_corpus`。

### Lines 757-768: test case external sam only chain / 测试用例 external sam only chain
```python
    def test_external_sam_only_chain(self):
        corpus = _make_corpus(
            "BFS",
            draft_token_num=4,
            external_sam_budget=3,
            external_corpus_documents=[[1, 2, 3, 4, 5]],
        )

        ids, masks = _batch_get(corpus, [[1, 2, 3]])
        ids_list = ids.tolist()
        self.assertEqual(ids_list[0], 3)
        self.assertEqual(ids_list[1:3], [4, 5])
```
**EN:** This test exercises `test_external_sam_only_chain` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_external_sam_only_chain`。

### Lines 770-781: test case external sam respects document boundaries / 测试用例 external sam respects document boundaries
```python
    def test_external_sam_respects_document_boundaries(self):
        corpus = _make_corpus(
            "BFS",
            draft_token_num=4,
            external_sam_budget=3,
            external_corpus_documents=[[1, 2, 3], [4, 5, 6]],
        )

        ids, _ = _batch_get(corpus, [[2, 3]])
        ids_list = ids.tolist()
        self.assertEqual(ids_list[0], 3)
        self.assertTrue(all(token == 0 for token in ids_list[1:]), ids_list)
```
**EN:** This test exercises `test_external_sam_respects_document_boundaries` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_external_sam_respects_document_boundaries`。

### Lines 783-798: test case external sam adds distinct root branch / 测试用例 external sam adds distinct root branch
```python
    def test_external_sam_adds_distinct_root_branch(self):
        corpus = _make_corpus(
            "BFS",
            draft_token_num=6,
            external_sam_budget=2,
            external_corpus_documents=[[1, 2, 3, 20, 21]],
        )
        corpus.batch_put([[1, 2, 3, 10, 11]])
        corpus.synchronize()

        ids, masks = _batch_get(corpus, [[1, 2, 3]])
        leaf_paths = corpus.leaf_paths_from_mask(
            ids.tolist(), masks.reshape(6, 6).tolist()
        )
        self.assertIn([3, 10, 11], leaf_paths)
        self.assertIn([3, 20, 21], leaf_paths)
```
**EN:** This test exercises `test_external_sam_adds_distinct_root_branch` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_external_sam_adds_distinct_root_branch`。

### Lines 800-815: test case shared prefix keeps both branches / 测试用例 shared prefix keeps both branches
```python
    def test_shared_prefix_keeps_both_branches(self):
        corpus = _make_corpus(
            "BFS",
            draft_token_num=5,
            external_sam_budget=2,
            external_corpus_documents=[[1, 2, 3, 10, 99]],
        )
        corpus.batch_put([[1, 2, 3, 10, 11]])
        corpus.synchronize()

        ids, masks = _batch_get(corpus, [[1, 2, 3]])
        leaf_paths = corpus.leaf_paths_from_mask(
            ids.tolist(), masks.reshape(5, 5).tolist()
        )
        self.assertIn([3, 10, 11], leaf_paths)
        self.assertIn([3, 10, 99], leaf_paths)
```
**EN:** This test exercises `test_shared_prefix_keeps_both_branches` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_shared_prefix_keeps_both_branches`。

### Lines 817-832: test case shared prefix merge can underfill budget / 测试用例 shared prefix merge can underfill budget
```python
    def test_shared_prefix_merge_can_underfill_budget(self):
        corpus = _make_corpus(
            "BFS",
            draft_token_num=6,
            external_sam_budget=2,
            external_corpus_documents=[[1, 2, 3, 10, 99]],
        )
        corpus.batch_put([[1, 2, 3, 10, 11]])
        corpus.synchronize()

        ids, masks = _batch_get(corpus, [[1, 2, 3]])
        ids_list = ids.tolist()
        leaf_paths = corpus.leaf_paths_from_mask(ids_list, masks.reshape(6, 6).tolist())
        self.assertIn([3, 10, 11], leaf_paths)
        self.assertIn([3, 10, 99], leaf_paths)
        self.assertEqual(ids_list.count(0), 2, ids_list)
```
**EN:** This test exercises `test_shared_prefix_merge_can_underfill_budget` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_shared_prefix_merge_can_underfill_budget`。

### Lines 834-850: test case external sam prob prefers frequent continuation / 测试用例 external sam prob prefers frequent continuation
```python
    def test_external_sam_prob_prefers_frequent_continuation(self):
        corpus = _make_corpus(
            "PROB",
            draft_token_num=2,
            min_bfs_breadth=1,
            max_bfs_breadth=1,
            external_sam_budget=1,
            external_corpus_documents=[
                [1, 2, 3, 10],
                [1, 2, 3, 20],
                [1, 2, 3, 20],
                [1, 2, 3, 20],
            ],
        )

        ids, _ = _batch_get(corpus, [[1, 2, 3]])
        self.assertEqual(ids.tolist(), [3, 20])
```
**EN:** This test exercises `test_external_sam_prob_prefers_frequent_continuation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_external_sam_prob_prefers_frequent_continuation`。

### Lines 853-853: class TestNgramCorpusMatchBenchmark declaration / 类 TestNgramCorpusMatchBenchmark 声明
```python
class TestNgramCorpusMatchBenchmark(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 854-854: supporting statements / 辅助语句
```python
    """Benchmark incremental advance vs full rebuild in match()."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 856-914: test case incremental faster than rebuild / 测试用例 incremental faster than rebuild
```python
    def test_incremental_faster_than_rebuild(self):
        """Incremental advance (O(D) per token) should be faster than rebuild (O(D^2))."""
        import time

        max_trie_depth = 18
        draft_token_num = 8
        corpus = _make_corpus(
            "BFS",
            max_trie_depth=max_trie_depth,
            draft_token_num=draft_token_num,
            capacity=500000,
        )

        # Seed the trie with diverse sequences so suffix matching is non-trivial.
        seed_data = [list(range(i, i + 50)) for i in range(0, 5000, 50)]
        corpus.batch_put(seed_data)
        corpus.synchronize()

        num_steps = 500
        base_seq = list(range(1, max_trie_depth + 1))

        # --- Incremental path: same req_id, total_len grows by 1 each step ---
        req_id = "bench-incremental"
        # Warm up the state with the initial context.
        _batch_get_with_state(corpus, req_id, base_seq, len(base_seq))

        start = time.perf_counter()
        for step in range(num_steps):
            total_len = len(base_seq) + step + 1
            new_token = (step + max_trie_depth + 1) % 5000
            tail = (base_seq + [new_token])[-max_trie_depth:]
            base_seq = tail
            _batch_get_with_state(corpus, req_id, tail, total_len)
        incremental_us = (time.perf_counter() - start) / num_steps * 1e6

        # --- Rebuild path: unique req_id each call forces fresh state ---
        base_seq = list(range(1, max_trie_depth + 1))
        start = time.perf_counter()
        for step in range(num_steps):
            new_token = (step + max_trie_depth + 1) % 5000
            tail = (base_seq + [new_token])[-max_trie_depth:]
            base_seq = tail
            _batch_get(corpus, [tail])
        rebuild_us = (time.perf_counter() - start) / num_steps * 1e6

        print(
            f"\n  Incremental: {incremental_us:.1f} us/step"
            f"\n  Rebuild:     {rebuild_us:.1f} us/step"
            f"\n  Speedup:     {rebuild_us / incremental_us:.2f}x"
        )

        # The incremental path should be at least as fast; allow a small margin
        # for noise. With D=12 the theoretical speedup is ~12x (D^2/D).
        self.assertLess(
            incremental_us,
            rebuild_us * 1.1,
            f"Incremental ({incremental_us:.1f} us) should not be slower than "
            f"rebuild ({rebuild_us:.1f} us)",
        )
```
**EN:** Incremental advance (O(D) per token) should be faster than rebuild (O(D^2)). This test exercises `test_incremental_faster_than_rebuild` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Incremental advance (O(D) per token) should be faster than rebuild (O(D^2)). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_incremental_faster_than_rebuild`。

### Lines 917-917: class TestNgramCorpusMultiSam declaration / 类 TestNgramCorpusMultiSam 声明
```python
class TestNgramCorpusMultiSam(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 918-918: supporting statements / 辅助语句
```python
    """Verify multi-SAM add/remove/list and budget splitting."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 920-931: test case add and list / 测试用例 add and list
```python
    def test_add_and_list(self):
        corpus = _make_corpus("BFS", draft_token_num=4, external_sam_budget=3)
        loaded_token_count = corpus.load_external_corpus_named("a", [[1, 2, 3, 4, 5]])
        corpus.commit_external_corpus_load("a", loaded_token_count)
        loaded_token_count = corpus.load_external_corpus_named(
            "b", [[10, 20, 30, 40, 50]]
        )
        corpus.commit_external_corpus_load("b", loaded_token_count)
        token_counts = corpus.list_external_corpora()
        self.assertEqual(sorted(token_counts.keys()), ["a", "b"])
        self.assertEqual(token_counts["a"], 5)
        self.assertEqual(token_counts["b"], 5)
```
**EN:** This test exercises `test_add_and_list` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_add_and_list`。

### Lines 933-942: test case remove / 测试用例 remove
```python
    def test_remove(self):
        corpus = _make_corpus("BFS", draft_token_num=4, external_sam_budget=3)
        loaded_token_count = corpus.load_external_corpus_named("a", [[1, 2, 3, 4, 5]])
        corpus.commit_external_corpus_load("a", loaded_token_count)
        loaded_token_count = corpus.load_external_corpus_named(
            "b", [[10, 20, 30, 40, 50]]
        )
        corpus.commit_external_corpus_load("b", loaded_token_count)
        corpus.remove_external_corpus("a")
        self.assertEqual(list(corpus.list_external_corpora().keys()), ["b"])
```
**EN:** This test exercises `test_remove` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_remove`。

### Lines 944-947: test case remove nonexistent is noop / 测试用例 remove nonexistent is noop
```python
    def test_remove_nonexistent_is_noop(self):
        corpus = _make_corpus("BFS", draft_token_num=4, external_sam_budget=3)
        corpus.remove_external_corpus("nonexistent")
        self.assertEqual(corpus.list_external_corpora(), {})
```
**EN:** This test exercises `test_remove_nonexistent_is_noop` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_remove_nonexistent_is_noop`。

### Lines 949-962: test case multi sam candidates / 测试用例 multi sam candidates
```python
    def test_multi_sam_candidates(self):
        corpus = _make_corpus("BFS", draft_token_num=6, external_sam_budget=4)
        loaded_token_count = corpus.load_external_corpus_named("a", [[1, 2, 3, 10, 11]])
        corpus.commit_external_corpus_load("a", loaded_token_count)
        loaded_token_count = corpus.load_external_corpus_named("b", [[1, 2, 3, 20, 21]])
        corpus.commit_external_corpus_load("b", loaded_token_count)

        ids, masks = _batch_get(corpus, [[1, 2, 3]])
        leaf_paths = corpus.leaf_paths_from_mask(
            ids.tolist(), masks.reshape(6, 6).tolist()
        )
        # Both SAMs should contribute candidates
        self.assertIn([3, 10, 11], leaf_paths)
        self.assertIn([3, 20, 21], leaf_paths)
```
**EN:** This test exercises `test_multi_sam_candidates` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multi_sam_candidates`。

### Lines 964-978: test case remove reduces candidates / 测试用例 remove reduces candidates
```python
    def test_remove_reduces_candidates(self):
        corpus = _make_corpus("BFS", draft_token_num=6, external_sam_budget=4)
        loaded_token_count = corpus.load_external_corpus_named("a", [[1, 2, 3, 10, 11]])
        corpus.commit_external_corpus_load("a", loaded_token_count)
        loaded_token_count = corpus.load_external_corpus_named("b", [[1, 2, 3, 20, 21]])
        corpus.commit_external_corpus_load("b", loaded_token_count)

        corpus.remove_external_corpus("b")

        ids, masks = _batch_get(corpus, [[1, 2, 3]])
        leaf_paths = corpus.leaf_paths_from_mask(
            ids.tolist(), masks.reshape(6, 6).tolist()
        )
        self.assertIn([3, 10, 11], leaf_paths)
        self.assertNotIn([3, 20, 21], leaf_paths)
```
**EN:** This test exercises `test_remove_reduces_candidates` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_remove_reduces_candidates`。

### Lines 980-989: test case make corpus with documents / 测试用例 make corpus with documents
```python
    def test_make_corpus_with_documents(self):
        """_make_corpus helper loads documents as a named corpus."""
        corpus = _make_corpus(
            "BFS",
            draft_token_num=4,
            external_sam_budget=3,
            external_corpus_documents=[[1, 2, 3, 4, 5]],
        )
        token_counts = corpus.list_external_corpora()
        self.assertIn("test_corpus", token_counts)
```
**EN:** _make_corpus helper loads documents as a named corpus. This test exercises `test_make_corpus_with_documents` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** _make_corpus helper loads documents as a named corpus. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_make_corpus_with_documents`。

### Lines 991-1013: test case remove frees token budget / 测试用例 remove frees token budget
```python
    def test_remove_frees_token_budget(self):
        """Removing a corpus should free its tokens from the total budget."""
        corpus = _make_corpus(
            "BFS",
            draft_token_num=4,
            external_sam_budget=3,
            external_corpus_max_tokens=10,
        )
        loaded_token_count = corpus.load_external_corpus_named("a", [[1, 2, 3, 4, 5]])
        corpus.commit_external_corpus_load("a", loaded_token_count)
        loaded_token_count = corpus.load_external_corpus_named(
            "b", [[10, 20, 30, 40, 50]]
        )
        corpus.commit_external_corpus_load("b", loaded_token_count)
        self.assertEqual(corpus.remaining_token_budget, 0)

        corpus.remove_external_corpus("a")
        self.assertEqual(corpus.remaining_token_budget, 5)

        # Now there's room for a new corpus.
        loaded_token_count = corpus.load_external_corpus_named("c", [[100, 200, 300]])
        corpus.commit_external_corpus_load("c", loaded_token_count)
        self.assertEqual(sorted(corpus.list_external_corpora().keys()), ["b", "c"])
```
**EN:** Removing a corpus should free its tokens from the total budget. This test exercises `test_remove_frees_token_budget` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Removing a corpus should free its tokens from the total budget. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_remove_frees_token_budget`。

### Lines 1015-1039: test case duplicate corpus id is rejected / 测试用例 duplicate corpus id is rejected
```python
    def test_duplicate_corpus_id_is_rejected(self):
        """Adding a duplicate corpus_id should fail without replacing the original corpus."""
        corpus = _make_corpus(
            "BFS",
            draft_token_num=4,
            external_sam_budget=3,
            external_corpus_max_tokens=10,
        )
        loaded_token_count = corpus.load_external_corpus_named("a", [[1, 2, 3, 4, 5]])
        corpus.commit_external_corpus_load("a", loaded_token_count)
        with self.assertRaisesRegex(ValueError, "already exists"):
            corpus.load_external_corpus_named("a", [[10, 20, 30]])

        self.assertEqual(corpus.remaining_token_budget, 5)
        self.assertEqual(list(corpus.list_external_corpora().keys()), ["a"])

        # The original corpus must still be usable for matching.
        ids, masks = _batch_get(corpus, [[1, 2, 3]])
        leaf_paths = corpus.leaf_paths_from_mask(
            ids.tolist(), masks.reshape(4, 4).tolist()
        )
        self.assertTrue(
            any(4 in path or 5 in path for path in leaf_paths),
            f"Expected tokens from corpus 'a' in {leaf_paths}",
        )
```
**EN:** Adding a duplicate corpus_id should fail without replacing the original corpus. This test exercises `test_duplicate_corpus_id_is_rejected` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Adding a duplicate corpus_id should fail without replacing the original corpus. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_duplicate_corpus_id_is_rejected`。

### Lines 1041-1068: test case error on load preserves existing corpora / 测试用例 error on load preserves existing corpora
```python
    def test_error_on_load_preserves_existing_corpora(self):
        """A failed load must not wipe previously loaded corpora (staging-only cleanup)."""
        corpus = _make_corpus(
            "BFS",
            draft_token_num=4,
            external_sam_budget=3,
            external_corpus_max_tokens=10,
        )
        loaded_token_count = corpus.load_external_corpus_named("a", [[1, 2, 3, 4, 5]])
        corpus.commit_external_corpus_load("a", loaded_token_count)

        # Force an error by exceeding the budget.
        with self.assertRaises(ValueError):
            corpus.load_external_corpus_named("b", [[10, 20, 30, 40, 50, 60]])

        self.assertEqual(list(corpus.list_external_corpora().keys()), ["a"])
        self.assertEqual(corpus.remaining_token_budget, 5)

        # "a" must still be usable for matching.
        ids, masks = _batch_get(corpus, [[1, 2, 3]])
        leaf_paths = corpus.leaf_paths_from_mask(
            ids.tolist(), masks.reshape(4, 4).tolist()
        )
        # Should still find continuations from corpus "a".
        self.assertTrue(
            any(4 in path or 5 in path for path in leaf_paths),
            f"Expected tokens from corpus 'a' in {leaf_paths}",
        )
```
**EN:** A failed load must not wipe previously loaded corpora (staging-only cleanup). This test exercises `test_error_on_load_preserves_existing_corpora` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** A failed load must not wipe previously loaded corpora (staging-only cleanup). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_error_on_load_preserves_existing_corpora`。

### Lines 1071-1071: class TestMultiSamHttpMock declaration / 类 TestMultiSamHttpMock 声明
```python
class TestMultiSamHttpMock(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 1072-1072: supporting statements / 辅助语句
```python
    """Test HTTP endpoints for multi-SAM management with a mocked backend."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 1074-1116: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        from unittest.mock import AsyncMock, MagicMock

        try:
            from starlette.testclient import TestClient

            from sglang.srt.entrypoints.http_server import app, set_global_state
        except (ImportError, OSError):
            raise unittest.SkipTest(
                "http_server import requires CUDA libraries not available on CPU"
            )
        from sglang.srt.managers.io_struct import (
            AddExternalCorpusReqOutput,
            ListExternalCorporaReqOutput,
            RemoveExternalCorpusReqOutput,
        )

        mock_state = MagicMock()
        tm = mock_state.tokenizer_manager

        # Wire up async methods that the HTTP handlers call
        tm.add_external_corpus = AsyncMock(
            return_value=AddExternalCorpusReqOutput(
                success=True,
                corpus_id="test-id",
                message="Loaded corpus 'test-id' with 100 tokens.",
                loaded_token_count=100,
            )
        )
        tm.remove_external_corpus = AsyncMock(
            return_value=RemoveExternalCorpusReqOutput(
                success=True, message="Removed corpus 'test-id'."
            )
        )
        tm.list_external_corpora = AsyncMock(
            return_value=ListExternalCorporaReqOutput(
                success=True, corpus_token_counts={"a": 100, "b": 200}
            )
        )
        set_global_state(mock_state)
        cls.client = TestClient(app)
        cls.mock_tm = tm
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 1118-1127: test case add corpus / 测试用例 add corpus
```python
    def test_add_corpus(self):
        resp = self.client.post(
            "/add_external_corpus",
            json={"corpus_id": "my-corpus", "documents": ["hello world"]},
        )
        self.assertEqual(resp.status_code, 200)
        data = resp.json()
        self.assertTrue(data["success"])
        self.assertEqual(data["corpus_id"], "test-id")
        self.assertEqual(data["loaded_token_count"], 100)
```
**EN:** This test exercises `test_add_corpus` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_add_corpus`。

### Lines 1129-1135: test case add corpus auto id / 测试用例 add corpus auto id
```python
    def test_add_corpus_auto_id(self):
        resp = self.client.post(
            "/add_external_corpus",
            json={"documents": ["hello world"]},
        )
        self.assertEqual(resp.status_code, 200)
        self.assertTrue(resp.json()["success"])
```
**EN:** This test exercises `test_add_corpus_auto_id` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_add_corpus_auto_id`。

### Lines 1137-1143: test case remove corpus / 测试用例 remove corpus
```python
    def test_remove_corpus(self):
        resp = self.client.post(
            "/remove_external_corpus",
            json={"corpus_id": "test-id"},
        )
        self.assertEqual(resp.status_code, 200)
        self.assertTrue(resp.json()["success"])
```
**EN:** This test exercises `test_remove_corpus` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_remove_corpus`。

### Lines 1145-1150: test case remove corpus missing id / 测试用例 remove corpus missing id
```python
    def test_remove_corpus_missing_id(self):
        resp = self.client.post(
            "/remove_external_corpus",
            json={},
        )
        self.assertEqual(resp.status_code, 400)
```
**EN:** This test exercises `test_remove_corpus_missing_id` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_remove_corpus_missing_id`。

### Lines 1152-1157: test case list corpora / 测试用例 list corpora
```python
    def test_list_corpora(self):
        resp = self.client.get("/list_external_corpora")
        self.assertEqual(resp.status_code, 200)
        data = resp.json()
        self.assertTrue(data["success"])
        self.assertEqual(data["corpus_token_counts"], {"a": 100, "b": 200})
```
**EN:** This test exercises `test_list_corpora` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_list_corpora`。

### Lines 1160-1161: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=3)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_make_corpus`: This block implements `_make_corpus` and captures one focused piece of the module's behavior. / 该代码块实现 `_make_corpus`，承担模块行为中的一个聚焦逻辑片段。
- `_batch_get`: This block implements `_batch_get` and captures one focused piece of the module's behavior. / 该代码块实现 `_batch_get`，承担模块行为中的一个聚焦逻辑片段。
- `_batch_get_with_state`: This block implements `_batch_get_with_state` and captures one focused piece of the module's behavior. / 该代码块实现 `_batch_get_with_state`，承担模块行为中的一个聚焦逻辑片段。
- `_IntTokenizer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestNgramCorpusBFS`: Golden-output tests for BFS matching mode. / 用于组织相关测试、夹具或辅助方法。
- `TestNgramCorpusProb`: Golden-output tests for Prob matching mode. / 用于组织相关测试、夹具或辅助方法。
- `TestNgramCorpusReset`: Verify reset clears all cached state. / 用于组织相关测试、夹具或辅助方法。
- `TestNgramCorpusNoMatch`: Verify behavior when query has no match in the corpus. / 用于组织相关测试、夹具或辅助方法。
- `_IntTokenizer.encode`: This block implements `encode` and captures one focused piece of the module's behavior. / 该代码块实现 `encode`，承担模块行为中的一个聚焦逻辑片段。
- `TestNgramCorpusBFS.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestNgramCorpusBFS.test_token_ids`: This test exercises `test_token_ids` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_token_ids`。
- `TestNgramCorpusBFS.test_masks`: This test exercises `test_masks` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_masks`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `tempfile`, `unittest`, `uuid`
- **Third-party modules / 第三方模块**: `numpy`
- **Internal modules / 内部模块**: `sglang.srt.speculative.cpp_ngram.external_corpus`, `sglang.srt.speculative.cpp_ngram.ngram_corpus`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 1161
