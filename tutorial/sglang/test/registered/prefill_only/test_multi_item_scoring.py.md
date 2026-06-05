# test_multi_item_scoring.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/prefill_only/test_multi_item_scoring.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates multi item scoring behavior in SGLang's prefill only area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 prefill only 领域中与 multi item scoring 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: supporting statements / 辅助语句
```python
"""Tests for the Multi-Item Scoring (MIS) optimization.

MIS is a server-side optimization enabled via --enable-mis that batches
multiple items into a single forward pass using delimiter tokens (token ID 9999).
This is different from batch scoring (multiple items in one API call) which
processes items as separate requests.

The key difference:
- Batch scoring: N items -> N separate forward passes
- MIS optimization: N items -> 1 forward pass with delimiter-separated items

These tests ensure the MIS optimization produces correct results and catches
bugs in tensor shape handling (e.g., 2D tensors [num_delimiters, num_label_tokens]).
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 16-28: module imports and dependencies / 模块导入与依赖
```python
import asyncio
import os
import unittest

import torch
from transformers import AutoConfig, AutoTokenizer

from sglang.srt.entrypoints.engine import Engine
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    CustomTestCase,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `asyncio`, `os`, `unittest`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `asyncio`, `os`, `unittest`, `torch`。

### Lines 30-37: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=211, stage="base-b", runner_config="1-gpu-small")

TEST_MODEL_NAME = os.environ.get("TEST_MODEL_NAME", DEFAULT_SMALL_MODEL_NAME_FOR_TEST)
TEST_CLASSIFICATION_BASE_MODEL = os.environ.get(
    "TEST_CLASSIFICATION_BASE_MODEL",
    "tomaarsen/Qwen3-Reranker-0.6B-seq-cls",
)
_CLS_NUM_LABELS = AutoConfig.from_pretrained(TEST_CLASSIFICATION_BASE_MODEL).num_labels
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, get, from_pretrained.
**CN:** 该代码块通过 register_cuda_ci, get, from_pretrained 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 40-40: class TestMISServerArgsValidation declaration / 类 TestMISServerArgsValidation 声明
```python
class TestMISServerArgsValidation(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 41-41: supporting statements / 辅助语句
```python
    """Test ServerArgs defaults for MIS mode."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 43-47: test case enable mis default / 测试用例 enable mis default
```python
    def test_enable_mis_default(self):
        """Test that enable_mis defaults to False."""
        from sglang.srt.server_args import ServerArgs

        self.assertEqual(ServerArgs.enable_mis, False)
```
**EN:** Test that enable_mis defaults to False. This test exercises `test_enable_mis_default` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that enable_mis defaults to False. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_enable_mis_default`。

### Lines 50-50: class TestMultiItemScoringOptimization declaration / 类 TestMultiItemScoringOptimization 声明
```python
class TestMultiItemScoringOptimization(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 51-51: supporting statements / 辅助语句
```python
    """Test the Multi-Item Scoring (MIS) optimization with generation models."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 53-68: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.engine = Engine(
            model_path=TEST_MODEL_NAME,
            disable_radix_cache=True,
            chunked_prefill_size=-1,
            enable_mis=True,
            attention_backend="flashinfer",
            mem_fraction_static=0.15,
        )
        cls.non_mis_engine = Engine(
            model_path=TEST_MODEL_NAME,
            disable_radix_cache=True,
            chunked_prefill_size=-1,
            mem_fraction_static=0.15,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 70-76: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if cls.engine is not None:
            cls.engine.shutdown()
        if cls.non_mis_engine is not None:
            cls.non_mis_engine.shutdown()
        torch.cuda.empty_cache()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 78-97: test case mis basic / 测试用例 mis basic
```python
    def test_mis_basic(self):
        """Test basic MIS: correct shapes, valid probabilities."""
        query = "Rate each option:"
        items = ["Option A", "Option B", "Option C"]
        label_token_ids = [9454, 2753]  # "Yes" and "No" tokens

        scores = self.engine.score(
            query=query,
            items=items,
            label_token_ids=label_token_ids,
            apply_softmax=True,
        ).scores

        self.assertEqual(len(scores), len(items))
        for i, score_list in enumerate(scores):
            self.assertEqual(len(score_list), len(label_token_ids))
            self.assertAlmostEqual(sum(score_list), 1.0, places=5)
            for score in score_list:
                self.assertGreaterEqual(score, 0)
                self.assertLessEqual(score, 1)
```
**EN:** Test basic MIS: correct shapes, valid probabilities. This test exercises `test_mis_basic` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test basic MIS: correct shapes, valid probabilities. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_basic`。

### Lines 99-127: test case mis consistency with single item / 测试用例 mis consistency with single item
```python
    def test_mis_consistency_with_single_item(self):
        """MIS with one item should match non-MIS scoring closely."""
        query = "Is this a fact?\n"
        items = [" The sun rises in the east"]
        label_token_ids = [9454, 2753]

        mis_scores = self.engine.score(
            query=query,
            items=items,
            label_token_ids=label_token_ids,
            apply_softmax=True,
        ).scores

        non_mis_scores = self.non_mis_engine.score(
            query=query,
            items=items,
            label_token_ids=label_token_ids,
            apply_softmax=True,
        ).scores

        self.assertEqual(len(mis_scores), 1)
        self.assertEqual(len(non_mis_scores), 1)
        for j, (m, n) in enumerate(zip(mis_scores[0], non_mis_scores[0])):
            relative_diff = abs(m - n) / max(abs(n), 1e-6)
            self.assertLess(
                relative_diff,
                0.08,
                msg=f"label {j}: MIS={m} vs non-MIS={n} (diff: {relative_diff:.3f})",
            )
```
**EN:** MIS with one item should match non-MIS scoring closely. This test exercises `test_mis_consistency_with_single_item` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** MIS with one item should match non-MIS scoring closely. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_consistency_with_single_item`。

### Lines 129-144: test case mis empty query / 测试用例 mis empty query
```python
    def test_mis_empty_query(self):
        """MIS with empty query — delimiter indices start at position 0."""
        items = ["alpha", "beta"]
        label_token_ids = [9454, 2753]

        scores = self.engine.score(
            query="",
            items=items,
            label_token_ids=label_token_ids,
            apply_softmax=True,
        ).scores

        self.assertEqual(len(scores), len(items))
        for score_list in scores:
            self.assertEqual(len(score_list), len(label_token_ids))
            self.assertAlmostEqual(sum(score_list), 1.0, places=5)
```
**EN:** MIS with empty query — delimiter indices start at position 0. This test exercises `test_mis_empty_query` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** MIS with empty query — delimiter indices start at position 0. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_empty_query`。

### Lines 147-147: class TestMultiItemScoringClassification declaration / 类 TestMultiItemScoringClassification 声明
```python
class TestMultiItemScoringClassification(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 148-154: class-level constants and configuration for `TestMultiItemScoringClassification` / 类级常量与配置
```python
    """Test MIS with classification models.

    Uses a pre-trained Qwen3ForSequenceClassification model so that the
    classification head weights are deterministic across Engine instances.
    """

    NUM_LABELS = _CLS_NUM_LABELS
```
**EN:** This block defines shared names such as `NUM_LABELS`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `NUM_LABELS` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 156-164: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.engine = Engine(
            model_path=TEST_CLASSIFICATION_BASE_MODEL,
            disable_radix_cache=True,
            chunked_prefill_size=-1,
            enable_mis=True,
            attention_backend="flashinfer",
            mem_fraction_static=0.15,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 166-169: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        if self.engine is not None:
            self.engine.shutdown()
            torch.cuda.empty_cache()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 171-184: test case classification mis basic / 测试用例 classification mis basic
```python
    def test_classification_mis_basic(self):
        """Classification MIS: correct shapes, valid softmax probabilities."""
        query = "Rate each option:"
        items = ["Option A", "Option B", "Option C"]

        scores = self.engine.score(query=query, items=items, apply_softmax=True).scores

        self.assertEqual(len(scores), len(items))
        for i, score_list in enumerate(scores):
            self.assertEqual(len(score_list), self.NUM_LABELS)
            self.assertAlmostEqual(sum(score_list), 1.0, places=5)
            for score in score_list:
                self.assertGreaterEqual(score, 0)
                self.assertLessEqual(score, 1)
```
**EN:** Classification MIS: correct shapes, valid softmax probabilities. This test exercises `test_classification_mis_basic` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Classification MIS: correct shapes, valid softmax probabilities. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_classification_mis_basic`。

### Lines 186-202: test case classification mis tokenized input / 测试用例 classification mis tokenized input
```python
    def test_classification_mis_tokenized_input(self):
        """Classification MIS with pre-tokenized query and items."""
        tokenizer = AutoTokenizer.from_pretrained(TEST_CLASSIFICATION_BASE_MODEL)
        query_ids = tokenizer.encode("Rate each option:", add_special_tokens=False)
        items_ids = [
            tokenizer.encode(item, add_special_tokens=False)
            for item in ["Option A", "Option B"]
        ]

        scores = self.engine.score(
            query=query_ids, items=items_ids, apply_softmax=True
        ).scores

        self.assertEqual(len(scores), len(items_ids))
        for score_list in scores:
            self.assertEqual(len(score_list), self.NUM_LABELS)
            self.assertAlmostEqual(sum(score_list), 1.0, places=5)
```
**EN:** Classification MIS with pre-tokenized query and items. This test exercises `test_classification_mis_tokenized_input` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Classification MIS with pre-tokenized query and items. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_classification_mis_tokenized_input`。

### Lines 204-222: test case classification non mis fallback / 测试用例 classification non mis fallback
```python
    def test_classification_non_mis_fallback(self):
        """Classification model works correctly without --enable-mis."""
        non_mis_engine = Engine(
            model_path=TEST_CLASSIFICATION_BASE_MODEL,
            disable_radix_cache=True,
            mem_fraction_static=0.15,
        )
        try:
            scores = non_mis_engine.score(
                query="Test:", items=["A", "B"], apply_softmax=True
            ).scores

            self.assertEqual(len(scores), 2)
            for score_list in scores:
                self.assertEqual(len(score_list), self.NUM_LABELS)
                self.assertAlmostEqual(sum(score_list), 1.0, places=5)
        finally:
            non_mis_engine.shutdown()
            torch.cuda.empty_cache()
```
**EN:** Classification model works correctly without --enable-mis. This test exercises `test_classification_non_mis_fallback` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Classification model works correctly without --enable-mis. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_classification_non_mis_fallback`。

### Lines 225-225: class TestMultiItemScoringParity declaration / 类 TestMultiItemScoringParity 声明
```python
class TestMultiItemScoringParity(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 226-226: supporting statements / 辅助语句
```python
    """Test that MIS produces the same results as single-item scoring."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 228-244: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.engine_single = Engine(
            model_path=TEST_MODEL_NAME,
            disable_radix_cache=True,
            log_level="error",
            mem_fraction_static=0.15,
        )
        cls.engine_mis = Engine(
            model_path=TEST_MODEL_NAME,
            disable_radix_cache=True,
            chunked_prefill_size=-1,
            log_level="error",
            enable_mis=True,
            attention_backend="flashinfer",
            mem_fraction_static=0.15,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 246-252: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if cls.engine_single is not None:
            cls.engine_single.shutdown()
        if cls.engine_mis is not None:
            cls.engine_mis.shutdown()
        torch.cuda.empty_cache()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 254-283: method compare scores / 方法 compare scores
```python
    def _compare_scores(
        self, query, items, label_token_ids=None, apply_softmax=True, test_name=""
    ):
        """Compare MIS vs single-item scoring results."""
        single_scores = self.engine_single.score(
            query=query,
            items=items,
            label_token_ids=label_token_ids,
            apply_softmax=apply_softmax,
        ).scores

        mis_scores = self.engine_mis.score(
            query=query,
            items=items,
            label_token_ids=label_token_ids,
            apply_softmax=apply_softmax,
        ).scores

        self.assertEqual(
            len(mis_scores), len(single_scores), f"{test_name}: count mismatch"
        )
        for i, (ms, ss) in enumerate(zip(mis_scores, single_scores)):
            self.assertEqual(len(ms), len(ss), f"{test_name}: item {i} length mismatch")
            for j, (m, s) in enumerate(zip(ms, ss)):
                self.assertAlmostEqual(
                    m,
                    s,
                    places=1,
                    msg=f"{test_name}: item {i} label {j}: MIS={m} vs single={s}",
                )
```
**EN:** Compare MIS vs single-item scoring results. This block implements `_compare_scores` and captures one focused piece of the module's behavior.
**CN:** Compare MIS vs single-item scoring results. 该代码块实现 `_compare_scores`，承担模块行为中的一个聚焦逻辑片段。

### Lines 285-291: test case parity basic / 测试用例 parity basic
```python
    def test_parity_basic(self):
        tokenizer = AutoTokenizer.from_pretrained(TEST_MODEL_NAME)
        query = "Rate this option:"
        items = [" Option A", " Option B", " Option C"]
        labels = [" good", " bad"]
        label_ids = [tokenizer.encode(lb, add_special_tokens=False)[0] for lb in labels]
        self._compare_scores(query, items, label_ids, test_name="basic")
```
**EN:** This test exercises `test_parity_basic` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parity_basic`。

### Lines 293-301: test case parity tokenized inputs / 测试用例 parity tokenized inputs
```python
    def test_parity_tokenized_inputs(self):
        tokenizer = AutoTokenizer.from_pretrained(TEST_MODEL_NAME)
        query = "Rate this option:"
        items = [" Option X", " Option Y"]
        labels = [" good", " bad"]
        query_ids = tokenizer.encode(query, add_special_tokens=False)
        items_ids = [tokenizer.encode(i, add_special_tokens=False) for i in items]
        label_ids = [tokenizer.encode(lb, add_special_tokens=False)[0] for lb in labels]
        self._compare_scores(query_ids, items_ids, label_ids, test_name="tokenized")
```
**EN:** This test exercises `test_parity_tokenized_inputs` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parity_tokenized_inputs`。

### Lines 303-311: test case parity without softmax / 测试用例 parity without softmax
```python
    def test_parity_without_softmax(self):
        tokenizer = AutoTokenizer.from_pretrained(TEST_MODEL_NAME)
        query = "The weather today is"
        items = [" sunny", " cloudy", " rainy"]
        labels = [" nice", " bad"]
        label_ids = [tokenizer.encode(lb, add_special_tokens=False)[0] for lb in labels]
        self._compare_scores(
            query, items, label_ids, apply_softmax=False, test_name="no_softmax"
        )
```
**EN:** This test exercises `test_parity_without_softmax` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parity_without_softmax`。

### Lines 313-319: test case parity many items / 测试用例 parity many items
```python
    def test_parity_many_items(self):
        tokenizer = AutoTokenizer.from_pretrained(TEST_MODEL_NAME)
        query = "Rate this option from 1 to 5:"
        items = [f" Option {i}" for i in range(10)]
        labels = [" 1", " 2", " 3", " 4", " 5"]
        label_ids = [tokenizer.encode(lb, add_special_tokens=False)[0] for lb in labels]
        self._compare_scores(query, items, label_ids, test_name="many_items")
```
**EN:** This test exercises `test_parity_many_items` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parity_many_items`。

### Lines 322-322: class TestMultiItemScoringClassificationParity declaration / 类 TestMultiItemScoringClassificationParity 声明
```python
class TestMultiItemScoringClassificationParity(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 323-331: class-level constants and configuration for `TestMultiItemScoringClassificationParity` / 类级常量与配置
```python
    """Test that MIS multi-item batching matches single-item MIS scoring.

    Both paths use the MIS engine (with delimiter tokens in the attention
    context).  The reference scores each item individually so each gets its
    own forward pass; the batched path packs all items into one pass.
    This isolates the MIS batching logic from the delimiter-presence effect.
    """

    NUM_LABELS = _CLS_NUM_LABELS
```
**EN:** This block defines shared names such as `NUM_LABELS`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `NUM_LABELS` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 333-342: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.engine = Engine(
            model_path=TEST_CLASSIFICATION_BASE_MODEL,
            disable_radix_cache=True,
            chunked_prefill_size=-1,
            enable_mis=True,
            attention_backend="flashinfer",
            mem_fraction_static=0.15,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 344-348: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if cls.engine is not None:
            cls.engine.shutdown()
        torch.cuda.empty_cache()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 350-378: method compare scores / 方法 compare scores
```python
    def _compare_scores(self, query, items, apply_softmax=True, test_name=""):
        """Compare MIS batched vs MIS single-item scoring results."""
        single_scores = []
        for item in items:
            result = self.engine.score(
                query=query,
                items=[item],
                apply_softmax=apply_softmax,
            ).scores
            single_scores.append(result[0])

        batched_scores = self.engine.score(
            query=query,
            items=items,
            apply_softmax=apply_softmax,
        ).scores

        self.assertEqual(
            len(batched_scores), len(single_scores), f"{test_name}: count mismatch"
        )
        for i, (bs, ss) in enumerate(zip(batched_scores, single_scores)):
            self.assertEqual(len(bs), len(ss), f"{test_name}: item {i} length mismatch")
            for j, (b, s) in enumerate(zip(bs, ss)):
                self.assertAlmostEqual(
                    b,
                    s,
                    places=1,
                    msg=f"{test_name}: item {i} label {j}: batched={b} vs single={s}",
                )
```
**EN:** Compare MIS batched vs MIS single-item scoring results. This block implements `_compare_scores` and captures one focused piece of the module's behavior.
**CN:** Compare MIS batched vs MIS single-item scoring results. 该代码块实现 `_compare_scores`，承担模块行为中的一个聚焦逻辑片段。

### Lines 380-383: test case parity basic / 测试用例 parity basic
```python
    def test_parity_basic(self):
        query = "Rate this option:"
        items = [" Option A", " Option B", " Option C"]
        self._compare_scores(query, items, test_name="cls_basic")
```
**EN:** This test exercises `test_parity_basic` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parity_basic`。

### Lines 385-392: test case parity tokenized inputs / 测试用例 parity tokenized inputs
```python
    def test_parity_tokenized_inputs(self):
        tokenizer = AutoTokenizer.from_pretrained(TEST_CLASSIFICATION_BASE_MODEL)
        query_ids = tokenizer.encode("Rate this option:", add_special_tokens=False)
        items_ids = [
            tokenizer.encode(item, add_special_tokens=False)
            for item in [" Option X", " Option Y"]
        ]
        self._compare_scores(query_ids, items_ids, test_name="cls_tokenized")
```
**EN:** This test exercises `test_parity_tokenized_inputs` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parity_tokenized_inputs`。

### Lines 394-399: test case parity without softmax / 测试用例 parity without softmax
```python
    def test_parity_without_softmax(self):
        query = "The weather today is"
        items = [" sunny", " cloudy", " rainy"]
        self._compare_scores(
            query, items, apply_softmax=False, test_name="cls_no_softmax"
        )
```
**EN:** This test exercises `test_parity_without_softmax` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parity_without_softmax`。

### Lines 401-404: test case parity many items / 测试用例 parity many items
```python
    def test_parity_many_items(self):
        query = "Classify this option:"
        items = [f" Option {i}" for i in range(10)]
        self._compare_scores(query, items, test_name="cls_many_items")
```
**EN:** This test exercises `test_parity_many_items` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parity_many_items`。

### Lines 407-407: class TestMultiItemScoringClassificationMISvsNonMIS declaration / 类 TestMultiItemScoringClassificationMISvsNonMIS 声明
```python
class TestMultiItemScoringClassificationMISvsNonMIS(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 408-416: supporting statements / 辅助语句
```python
    """Test that MIS single-item approximates non-MIS single-item.

    The MIS path inserts delimiter tokens into the attention context,
    which slightly perturbs hidden states.  After softmax the scores
    should still be close.  Uses places=1 (±0.05) tolerance.

    Runs as a separate class so each engine is created and destroyed
    independently to avoid GPU OOM.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 418-463: test case mis single vs non mis / 测试用例 mis single vs non mis
```python
    def test_mis_single_vs_non_mis(self):
        non_mis_engine = Engine(
            model_path=TEST_CLASSIFICATION_BASE_MODEL,
            disable_radix_cache=True,
            mem_fraction_static=0.15,
        )
        try:
            query = "Rate this option:"
            items = [" Option A", " Option B", " Option C"]
            non_mis_scores = non_mis_engine.score(
                query=query,
                items=items,
                apply_softmax=True,
            ).scores
        finally:
            non_mis_engine.shutdown()
            torch.cuda.empty_cache()

        mis_engine = Engine(
            model_path=TEST_CLASSIFICATION_BASE_MODEL,
            disable_radix_cache=True,
            chunked_prefill_size=-1,
            enable_mis=True,
            attention_backend="flashinfer",
            mem_fraction_static=0.15,
        )
        try:
            mis_scores = mis_engine.score(
                query=query,
                items=items,
                apply_softmax=True,
            ).scores
        finally:
            mis_engine.shutdown()
            torch.cuda.empty_cache()

        self.assertEqual(len(mis_scores), len(non_mis_scores))
        for i, (ms, ns) in enumerate(zip(mis_scores, non_mis_scores)):
            self.assertEqual(len(ms), len(ns))
            for j, (m, n) in enumerate(zip(ms, ns)):
                self.assertAlmostEqual(
                    m,
                    n,
                    places=1,
                    msg=f"item {i} label {j}: MIS={m} vs non-MIS={n}",
                )
```
**EN:** This test exercises `test_mis_single_vs_non_mis` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_single_vs_non_mis`。

### Lines 466-466: class TestMultiItemScoringClassificationAdvanced declaration / 类 TestMultiItemScoringClassificationAdvanced 声明
```python
class TestMultiItemScoringClassificationAdvanced(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 467-470: class-level constants and configuration for `TestMultiItemScoringClassificationAdvanced` / 类级常量与配置
```python
    """Advanced MIS tests for classification models: score distinctness,
    determinism, and concurrent request handling."""

    NUM_LABELS = _CLS_NUM_LABELS
```
**EN:** This block defines shared names such as `NUM_LABELS`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `NUM_LABELS` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 472-481: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.engine = Engine(
            model_path=TEST_CLASSIFICATION_BASE_MODEL,
            disable_radix_cache=True,
            chunked_prefill_size=-1,
            enable_mis=True,
            attention_backend="flashinfer",
            mem_fraction_static=0.15,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 483-487: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if cls.engine is not None:
            cls.engine.shutdown()
        torch.cuda.empty_cache()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 489-511: test case items produce distinct scores / 测试用例 items produce distinct scores
```python
    def test_items_produce_distinct_scores(self):
        """Different items must produce different score vectors.

        Core regression test: before the delimiter-index fix, all items got
        identical scores because the MIS attention mask only let delimiter
        tokens attend to the query prefix.
        """
        query = "Rate each option:"
        items = [
            "Option A is about cats",
            "Option B is about dogs",
            "Option C is about fish",
        ]

        scores = self.engine.score(query=query, items=items).scores

        self.assertEqual(len(scores), len(items))
        all_identical = all(scores[0] == s for s in scores[1:])
        self.assertFalse(
            all_identical,
            f"All {len(items)} items returned identical scores — "
            f"MIS delimiter indexing is broken. Scores: {scores[0]}",
        )
```
**EN:** Different items must produce different score vectors. This test exercises `test_items_produce_distinct_scores` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Different items must produce different score vectors. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_items_produce_distinct_scores`。

### Lines 513-525: test case many items distinct / 测试用例 many items distinct
```python
    def test_many_items_distinct(self):
        """Stress test: 15 items should not all produce identical scores."""
        query = "Classify each city:"
        items = [f"City {i}" for i in range(15)]

        scores = self.engine.score(query=query, items=items).scores

        self.assertEqual(len(scores), len(items))
        for score_list in scores:
            self.assertEqual(len(score_list), self.NUM_LABELS)

        unique_count = len({tuple(s) for s in scores})
        self.assertGreater(unique_count, 1, "All 15 items returned identical scores")
```
**EN:** Stress test: 15 items should not all produce identical scores. This test exercises `test_many_items_distinct` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Stress test: 15 items should not all produce identical scores. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_many_items_distinct`。

### Lines 527-537: test case deterministic / 测试用例 deterministic
```python
    def test_deterministic(self):
        """Identical requests should return identical scores."""
        query = "Evaluate:"
        items = ["alpha", "beta", "gamma"]

        scores1 = self.engine.score(query=query, items=items).scores
        scores2 = self.engine.score(query=query, items=items).scores

        self.assertEqual(
            scores1, scores2, "Identical inputs must produce identical scores"
        )
```
**EN:** Identical requests should return identical scores. This test exercises `test_deterministic` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Identical requests should return identical scores. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_deterministic`。

### Lines 539-595: test case concurrent requests / 测试用例 concurrent requests
```python
    def test_concurrent_requests(self):
        """Concurrent MIS requests must produce the same scores as sequential.

        Runs each request sequentially to get baseline scores, then runs all
        concurrently and asserts the results match. This catches cross-request
        contamination when multiple MIS requests share a GPU batch.
        """
        test_cases = [
            {"query": "Is this a fruit?", "items": ["apple", "car", "banana"]},
            {"query": "Is this an animal?", "items": ["dog", "table"]},
            {
                "query": "Is this a country?",
                "items": ["France", "pizza", "Japan", "chair"],
            },
            {"query": "Is this a color?", "items": ["red"]},
        ]

        # Sequential baseline
        sequential_scores = []
        for tc in test_cases:
            result = self.engine.score(query=tc["query"], items=tc["items"])
            sequential_scores.append(result.scores)

        # Concurrent execution
        async def _gather():
            return await asyncio.gather(
                *(
                    self.engine.async_score(query=tc["query"], items=tc["items"])
                    for tc in test_cases
                )
            )

        concurrent_results = self.engine.loop.run_until_complete(_gather())

        for idx, (tc, seq_scores, conc_result) in enumerate(
            zip(test_cases, sequential_scores, concurrent_results)
        ):
            conc_scores = conc_result.scores
            self.assertEqual(
                len(conc_scores),
                len(seq_scores),
                f"Case {idx}: count mismatch",
            )
            for i, (cs, ss) in enumerate(zip(conc_scores, seq_scores)):
                self.assertEqual(
                    len(cs),
                    len(ss),
                    f"Case {idx} item {i}: label count mismatch",
                )
                for j, (c, s) in enumerate(zip(cs, ss)):
                    self.assertAlmostEqual(
                        c,
                        s,
                        places=1,
                        msg=f"Case {idx} item {i} label {j}: "
                        f"concurrent={c} vs sequential={s}",
                    )
```
**EN:** Concurrent MIS requests must produce the same scores as sequential. This test exercises `test_concurrent_requests` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Concurrent MIS requests must produce the same scores as sequential. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_concurrent_requests`。

### Lines 598-599: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestMISServerArgsValidation`: Test ServerArgs defaults for MIS mode. / 用于组织相关测试、夹具或辅助方法。
- `TestMultiItemScoringOptimization`: Test the Multi-Item Scoring (MIS) optimization with generation models. / 用于组织相关测试、夹具或辅助方法。
- `TestMultiItemScoringClassification`: Test MIS with classification models. / 用于组织相关测试、夹具或辅助方法。
- `TestMultiItemScoringParity`: Test that MIS produces the same results as single-item scoring. / 用于组织相关测试、夹具或辅助方法。
- `TestMultiItemScoringClassificationParity`: Test that MIS multi-item batching matches single-item MIS scoring. / 用于组织相关测试、夹具或辅助方法。
- `TestMultiItemScoringClassificationMISvsNonMIS`: Test that MIS single-item approximates non-MIS single-item. / 用于组织相关测试、夹具或辅助方法。
- `TestMultiItemScoringClassificationAdvanced`: Advanced MIS tests for classification models: score distinctness, determinism, and concurrent request handling. / 用于组织相关测试、夹具或辅助方法。
- `TestMISServerArgsValidation.test_enable_mis_default`: Test that enable_mis defaults to False. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_enable_mis_default`。
- `TestMultiItemScoringOptimization.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestMultiItemScoringOptimization.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestMultiItemScoringOptimization.test_mis_basic`: Test basic MIS: correct shapes, valid probabilities. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_basic`。
- `TestMultiItemScoringOptimization.test_mis_consistency_with_single_item`: MIS with one item should match non-MIS scoring closely. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_consistency_with_single_item`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `os`, `unittest`
- **Third-party modules / 第三方模块**: `torch`, `transformers`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.engine`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 599
