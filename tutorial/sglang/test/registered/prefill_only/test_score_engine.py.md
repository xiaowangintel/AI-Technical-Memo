# test_score_engine.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/prefill_only/test_score_engine.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates score engine behavior in SGLang's prefill only area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 prefill only 领域中与 score engine 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: supporting statements / 辅助语句
```python
"""Engine API tests for the /v1/score scoring pipeline.

Two model types, two scoring modes:

  TestCausalLMScoring        — CausalLM, single-item and batched multi-item
  TestSeqClsScoring          — SequenceClassification, single-item mode
  TestSeqClsMISScoring       — SequenceClassification, MIS mode (--enable-mis)
  TestSeqClsMISAdvancedScoring — SeqCls MIS with 12 labels (tensor shape stress)

The Engine (Python API) is the right layer for correctness testing: it
exercises tokenization, forward pass, pooling, and score extraction without
the HTTP serialization overhead.  HTTP-layer tests live in test_score_api.py.
Thorough MIS tests (parity, concurrency, generation models) live in
test_multi_item_scoring.py.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 17-27: module imports and dependencies / 模块导入与依赖
```python
import json
import os
import unittest
from unittest.mock import patch

import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

from sglang.srt.entrypoints.engine import Engine
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import DEFAULT_SMALL_MODEL_NAME_FOR_TEST, CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `os`, `unittest`, `unittest.mock`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `os`, `unittest`, `unittest.mock`。

### Lines 29-32: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=85, stage="base-b", runner_config="1-gpu-small")

_CAUSAL_LM_MODEL = os.environ.get("TEST_MODEL_NAME", DEFAULT_SMALL_MODEL_NAME_FOR_TEST)
_SEQCLS_MODEL = os.environ.get("TEST_CLASSIFICATION_BASE_MODEL", "Qwen/Qwen3-0.6B")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, get.
**CN:** 该代码块通过 register_cuda_ci, get 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 33-39: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# CausalLM
# ---------------------------------------------------------------------------


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 40-40: class TestCausalLMScoring declaration / 类 TestCausalLMScoring 声明
```python
class TestCausalLMScoring(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 41-45: supporting statements / 辅助语句
```python
    """CausalLM scoring via Engine — correctness, batching, and edge cases.

    A single Engine instance is shared across all test methods (class-level
    setup) so model loading happens once, not once per test.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 47-49: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.engine = Engine(model_path=_CAUSAL_LM_MODEL)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 51-55: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if hasattr(cls, "engine") and cls.engine:
            cls.engine.shutdown()
        torch.cuda.empty_cache()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 56-60: supporting source context / 辅助源码上下文
```python

    # ------------------------------------------------------------------
    # Helpers
    # ------------------------------------------------------------------

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 61-82: method hf scores / 方法 hf scores
```python
    def _hf_scores(self, query, items, label_token_ids, item_first=False):
        """Reference scores computed directly with HuggingFace (CPU inference)."""
        tokenizer = AutoTokenizer.from_pretrained(
            _CAUSAL_LM_MODEL, trust_remote_code=True
        )
        model = AutoModelForCausalLM.from_pretrained(
            _CAUSAL_LM_MODEL, trust_remote_code=True
        )
        try:
            scores = []
            for item in items:
                text = f"{item}{query}" if item_first else f"{query}{item}"
                inputs = tokenizer(text, return_tensors="pt").to(model.device)
                with torch.no_grad():
                    last_logits = model(**inputs).logits[0, -1]
                target_probs = torch.softmax(last_logits[label_token_ids], dim=-1)
                scores.append([p.item() for p in target_probs])
            return scores
        finally:
            model.cpu()
            del model, tokenizer
            torch.cuda.empty_cache()
```
**EN:** Reference scores computed directly with HuggingFace (CPU inference). This block implements `_hf_scores` and captures one focused piece of the module's behavior.
**CN:** Reference scores computed directly with HuggingFace (CPU inference). 该代码块实现 `_hf_scores`，承担模块行为中的一个聚焦逻辑片段。

### Lines 84-90: method assert scores close / 方法 assert scores close
```python
    def _assert_scores_close(self, hf, sgl, tol=0.01):
        self.assertEqual(len(hf), len(sgl))
        for hf_row, sgl_row in zip(hf, sgl):
            self.assertEqual(len(hf_row), len(sgl_row))
            for h, s in zip(hf_row, sgl_row):
                self.assertLessEqual(abs(h - s), tol, f"HF={h:.6f} SGLang={s:.6f}")
            self.assertAlmostEqual(sum(sgl_row), 1.0, places=6)
```
**EN:** This block implements `_assert_scores_close` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_assert_scores_close`，承担模块行为中的一个聚焦逻辑片段。

### Lines 91-95: supporting source context / 辅助源码上下文
```python

    # ------------------------------------------------------------------
    # Correctness
    # ------------------------------------------------------------------

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 96-121: test case scores match hf reference / 测试用例 scores match hf reference
```python
    def test_scores_match_hf_reference(self):
        """SGLang scores agree with HuggingFace within 1% tolerance."""
        label_token_ids = []
        tokenizer = AutoTokenizer.from_pretrained(
            _CAUSAL_LM_MODEL, trust_remote_code=True
        )
        for token in [" to", " the"]:
            label_token_ids.append(
                tokenizer(token, add_special_tokens=False)["input_ids"][0]
            )
        del tokenizer

        for query, items, item_first in [
            ("I pledge allegiance", ["", " to"], False),
            (" is a city", ["Tokyo", "Japan"], True),
        ]:
            with self.subTest(query=query):
                sgl = self.engine.score(
                    query=query,
                    items=items,
                    label_token_ids=label_token_ids,
                    apply_softmax=True,
                    item_first=item_first,
                ).scores
                hf = self._hf_scores(query, items, label_token_ids, item_first)
                self._assert_scores_close(hf, sgl)
```
**EN:** SGLang scores agree with HuggingFace within 1% tolerance. This test exercises `test_scores_match_hf_reference` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** SGLang scores agree with HuggingFace within 1% tolerance. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_scores_match_hf_reference`。

### Lines 123-157: test case request avoids decode phase / 测试用例 request avoids decode phase
```python
    def test_request_avoids_decode_phase(self):
        """Internal request must have max_new_tokens=0, logprob=True, stream=False."""
        captured = []
        original = self.engine.tokenizer_manager.generate_request

        async def capturing_gen(req, request=None):
            captured.append(req)
            async for result in original(req, request):
                yield result

        with patch.object(
            self.engine.tokenizer_manager,
            "generate_request",
            side_effect=capturing_gen,
        ):
            self.engine.score(
                query="What is the capital of",
                items=["France", "Germany"],
                label_token_ids=[1, 2, 3],
                apply_softmax=True,
            )

        self.assertEqual(len(captured), 1)
        req = captured[0]

        if isinstance(req.sampling_params, list):
            max_new_tokens = req.sampling_params[0].get("max_new_tokens", 0)
        elif isinstance(req.sampling_params, dict):
            max_new_tokens = req.sampling_params.get("max_new_tokens", 0)
        else:
            max_new_tokens = getattr(req.sampling_params, "max_new_tokens", 0)

        self.assertEqual(max_new_tokens, 0)
        self.assertTrue(req.return_logprob)
        self.assertFalse(req.stream)
```
**EN:** Internal request must have max_new_tokens=0, logprob=True, stream=False. This test exercises `test_request_avoids_decode_phase` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Internal request must have max_new_tokens=0, logprob=True, stream=False. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_request_avoids_decode_phase`。

### Lines 158-162: supporting source context / 辅助源码上下文
```python

    # ------------------------------------------------------------------
    # Multi-item / batching
    # ------------------------------------------------------------------

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 163-178: test case score batch sizes / 测试用例 score batch sizes
```python
    def test_score_batch_sizes(self):
        """Correct output count and shape for batch sizes 1, 2, 4, 8."""
        label_token_ids = [1, 2, 3]
        for n in [1, 2, 4, 8]:
            with self.subTest(n=n):
                scores = self.engine.score(
                    query="The test was",
                    items=[f"test {i}" for i in range(n)],
                    label_token_ids=label_token_ids,
                    apply_softmax=True,
                ).scores
                self.assertEqual(len(scores), n)
                for row in scores:
                    self.assertEqual(len(row), len(label_token_ids))
                    self.assertTrue(all(isinstance(v, float) for v in row))
                    self.assertAlmostEqual(sum(row), 1.0, places=6)
```
**EN:** Correct output count and shape for batch sizes 1, 2, 4, 8. This test exercises `test_score_batch_sizes` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Correct output count and shape for batch sizes 1, 2, 4, 8. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_score_batch_sizes`。

### Lines 180-186: test case score empty items / 测试用例 score empty items
```python
    def test_score_empty_items(self):
        """Empty items list → empty scores and zero prompt_tokens."""
        result = self.engine.score(
            query="Test query", items=[], label_token_ids=[1, 2], apply_softmax=True
        )
        self.assertEqual(len(result.scores), 0)
        self.assertEqual(result.prompt_tokens, 0)
```
**EN:** Empty items list → empty scores and zero prompt_tokens. This test exercises `test_score_empty_items` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Empty items list → empty scores and zero prompt_tokens. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_score_empty_items`。

### Lines 188-200: test case score without softmax / 测试用例 score without softmax
```python
    def test_score_without_softmax(self):
        """apply_softmax=False returns raw logits (not probability-constrained)."""
        scores = self.engine.score(
            query="Rate each:",
            items=["Good", "Bad", "Neutral"],
            label_token_ids=[1, 2, 3],
            apply_softmax=False,
        ).scores
        self.assertEqual(len(scores), 3)
        for row in scores:
            self.assertEqual(len(row), 3)
            for v in row:
                self.assertIsInstance(v, (int, float))
```
**EN:** apply_softmax=False returns raw logits (not probability-constrained). This test exercises `test_score_without_softmax` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** apply_softmax=False returns raw logits (not probability-constrained). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_score_without_softmax`。

### Lines 202-215: test case score varying label token sets / 测试用例 score varying label token sets
```python
    def test_score_varying_label_token_sets(self):
        """Different label_token_ids lengths all produce correct-shaped output."""
        for n_labels in [1, 2, 4, 8]:
            with self.subTest(n_labels=n_labels):
                scores = self.engine.score(
                    query="Choose:",
                    items=["Option A", "Option B"],
                    label_token_ids=list(range(1, n_labels + 1)),
                    apply_softmax=True,
                ).scores
                self.assertEqual(len(scores), 2)
                for row in scores:
                    self.assertEqual(len(row), n_labels)
                    self.assertAlmostEqual(sum(row), 1.0, places=6)
```
**EN:** Different label_token_ids lengths all produce correct-shaped output. This test exercises `test_score_varying_label_token_sets` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Different label_token_ids lengths all produce correct-shaped output. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_score_varying_label_token_sets`。

### Lines 217-227: test case score unicode / 测试用例 score unicode
```python
    def test_score_unicode(self):
        """Unicode query and items do not crash and produce valid scores."""
        scores = self.engine.score(
            query="选择最佳选项：",
            items=["选项A", "选项B", "选项C"],
            label_token_ids=[1, 2, 3],
            apply_softmax=True,
        ).scores
        self.assertEqual(len(scores), 3)
        for row in scores:
            self.assertAlmostEqual(sum(row), 1.0, places=6)
```
**EN:** Unicode query and items do not crash and produce valid scores. This test exercises `test_score_unicode` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Unicode query and items do not crash and produce valid scores. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_score_unicode`。

### Lines 229-238: test case score deterministic / 测试用例 score deterministic
```python
    def test_score_deterministic(self):
        """Identical calls return numerically equivalent scores (within GPU float tolerance)."""
        kwargs = dict(query="Choose:", items=["A", "B", "C"], label_token_ids=[1, 2, 3])
        scores_a = self.engine.score(**kwargs).scores
        scores_b = self.engine.score(**kwargs).scores
        self.assertEqual(len(scores_a), len(scores_b))
        for row_a, row_b in zip(scores_a, scores_b):
            self.assertEqual(len(row_a), len(row_b))
            for a, b in zip(row_a, row_b):
                self.assertAlmostEqual(a, b, places=5)
```
**EN:** Identical calls return numerically equivalent scores (within GPU float tolerance). This test exercises `test_score_deterministic` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Identical calls return numerically equivalent scores (within GPU float tolerance). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_score_deterministic`。

### Lines 240-249: test case score error handling / 测试用例 score error handling
```python
    def test_score_error_handling(self):
        """Invalid argument types raise ValueError or TypeError."""
        with self.assertRaises((ValueError, TypeError)):
            self.engine.score(
                query="Q", items=["X"], label_token_ids="bad", apply_softmax=True
            )
        with self.assertRaises((ValueError, TypeError)):
            self.engine.score(
                query="Q", items=None, label_token_ids=[1, 2], apply_softmax=True
            )
```
**EN:** Invalid argument types raise ValueError or TypeError. This test exercises `test_score_error_handling` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Invalid argument types raise ValueError or TypeError. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_score_error_handling`。

### Lines 250-256: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# SequenceClassification — single-item mode
# ---------------------------------------------------------------------------


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 257-257: class TestSeqClsScoring declaration / 类 TestSeqClsScoring 声明
```python
class TestSeqClsScoring(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 258-265: class-level constants and configuration for `TestSeqClsScoring` / 类级常量与配置
```python
    """SequenceClassification scoring via Engine — no MIS delimiter.

    Uses json_model_override_args to load Qwen3-0.6B backbone weights into
    Qwen3ForSequenceClassification.  The classification head is randomly
    initialised; shape/pipeline correctness is what matters here.
    """

    NUM_LABELS = 2
```
**EN:** This block defines shared names such as `NUM_LABELS`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `NUM_LABELS` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 267-279: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.engine = Engine(
            model_path=_SEQCLS_MODEL,
            disable_radix_cache=True,
            json_model_override_args=json.dumps(
                {
                    "architectures": ["Qwen3ForSequenceClassification"],
                    "num_labels": cls.NUM_LABELS,
                }
            ),
            mem_fraction_static=0.15,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 281-285: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if hasattr(cls, "engine") and cls.engine:
            cls.engine.shutdown()
        torch.cuda.empty_cache()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 287-300: test case score shape / 测试用例 score shape
```python
    def test_score_shape(self):
        """Each item gets a score vector of length num_labels."""
        scores = self.engine.score(
            query="Rate each option:",
            items=["Option A", "Option B"],
            apply_softmax=True,
        ).scores
        self.assertEqual(len(scores), 2)
        for i, row in enumerate(scores):
            self.assertEqual(len(row), self.NUM_LABELS)
            self.assertAlmostEqual(sum(row), 1.0, places=5)
            for v in row:
                self.assertGreaterEqual(v, 0.0)
                self.assertLessEqual(v, 1.0)
```
**EN:** Each item gets a score vector of length num_labels. This test exercises `test_score_shape` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Each item gets a score vector of length num_labels. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_score_shape`。

### Lines 302-309: test case score single item edge case / 测试用例 score single item edge case
```python
    def test_score_single_item_edge_case(self):
        """Single item in the list."""
        scores = self.engine.score(
            query="Evaluate:", items=["Only item"], apply_softmax=True
        ).scores
        self.assertEqual(len(scores), 1)
        self.assertEqual(len(scores[0]), self.NUM_LABELS)
        self.assertAlmostEqual(sum(scores[0]), 1.0, places=5)
```
**EN:** Single item in the list. This test exercises `test_score_single_item_edge_case` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Single item in the list. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_score_single_item_edge_case`。

### Lines 311-320: test case score without softmax / 测试用例 score without softmax
```python
    def test_score_without_softmax(self):
        """Without softmax, returns raw logits (no probability constraints)."""
        scores = self.engine.score(
            query="Evaluate:", items=["Alpha", "Beta"], apply_softmax=False
        ).scores
        self.assertEqual(len(scores), 2)
        for row in scores:
            self.assertEqual(len(row), self.NUM_LABELS)
            for v in row:
                self.assertIsInstance(v, (int, float))
```
**EN:** Without softmax, returns raw logits (no probability constraints). This test exercises `test_score_without_softmax` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Without softmax, returns raw logits (no probability constraints). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_score_without_softmax`。

### Lines 322-330: test case score deterministic / 测试用例 score deterministic
```python
    def test_score_deterministic(self):
        """Identical inputs yield near-identical scores (fp16 tolerance)."""
        kwargs = dict(query="Evaluate:", items=["alpha", "beta", "gamma"])
        scores1 = self.engine.score(**kwargs).scores
        scores2 = self.engine.score(**kwargs).scores
        self.assertEqual(len(scores1), len(scores2))
        for s1, s2 in zip(scores1, scores2):
            for v1, v2 in zip(s1, s2):
                self.assertAlmostEqual(v1, v2, places=1)
```
**EN:** Identical inputs yield near-identical scores (fp16 tolerance). This test exercises `test_score_deterministic` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Identical inputs yield near-identical scores (fp16 tolerance). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_score_deterministic`。

### Lines 332-351: test case score tokenized inputs / 测试用例 score tokenized inputs
```python
    def test_score_tokenized_inputs(self):
        """Pre-tokenized query/items match text input scores."""
        from transformers import AutoTokenizer

        tok = AutoTokenizer.from_pretrained(_SEQCLS_MODEL)
        query, items = "Rate this:", ["Good", "Bad"]

        text_scores = self.engine.score(
            query=query, items=items, apply_softmax=True
        ).scores
        token_scores = self.engine.score(
            query=tok.encode(query),
            items=[tok.encode(i) for i in items],
            apply_softmax=True,
        ).scores

        self.assertEqual(len(text_scores), len(token_scores))
        for ts, ks in zip(text_scores, token_scores):
            for t, k in zip(ts, ks):
                self.assertAlmostEqual(t, k, places=4)
```
**EN:** Pre-tokenized query/items match text input scores. This test exercises `test_score_tokenized_inputs` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Pre-tokenized query/items match text input scores. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_score_tokenized_inputs`。

### Lines 353-362: test case label token ids ignored / 测试用例 label token ids ignored
```python
    def test_label_token_ids_ignored(self):
        """SeqCls models ignore label_token_ids — output width is always num_labels."""
        scores = self.engine.score(
            query="Evaluate:",
            items=["Test item"],
            label_token_ids=[1, 2, 3],
            apply_softmax=True,
        ).scores
        self.assertEqual(len(scores), 1)
        self.assertEqual(len(scores[0]), self.NUM_LABELS)
```
**EN:** SeqCls models ignore label_token_ids — output width is always num_labels. This test exercises `test_label_token_ids_ignored` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** SeqCls models ignore label_token_ids — output width is always num_labels. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_label_token_ids_ignored`。

### Lines 363-369: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# SequenceClassification — MIS (delimiter) mode
# ---------------------------------------------------------------------------


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 370-370: class TestSeqClsMISScoring declaration / 类 TestSeqClsMISScoring 声明
```python
class TestSeqClsMISScoring(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 371-378: class-level constants and configuration for `TestSeqClsMISScoring` / 类级常量与配置
```python
    """SeqCls MIS: all items packed into one sequence separated by delimiter token.

    Uses --enable-mis which hardcodes delimiter token ID 9999.
    Basic pipeline correctness only — thorough MIS tests (parity,
    concurrency, advanced) live in test_multi_item_scoring.py.
    """

    NUM_LABELS = 2
```
**EN:** This block defines shared names such as `NUM_LABELS`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `NUM_LABELS` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 380-395: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.engine = Engine(
            model_path=_SEQCLS_MODEL,
            disable_radix_cache=True,
            chunked_prefill_size=-1,
            enable_mis=True,
            attention_backend="flashinfer",
            json_model_override_args=json.dumps(
                {
                    "architectures": ["Qwen3ForSequenceClassification"],
                    "num_labels": cls.NUM_LABELS,
                }
            ),
            mem_fraction_static=0.15,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 397-401: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if hasattr(cls, "engine") and cls.engine:
            cls.engine.shutdown()
        torch.cuda.empty_cache()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 403-415: test case mis one vector per item / 测试用例 mis one vector per item
```python
    def test_mis_one_vector_per_item(self):
        """MIS produces exactly one score vector per item."""
        items = ["Option A", "Option B", "Option C"]
        scores = self.engine.score(
            query="Rate each option:", items=items, apply_softmax=True
        ).scores
        self.assertEqual(len(scores), len(items))
        for i, row in enumerate(scores):
            self.assertEqual(len(row), self.NUM_LABELS)
            self.assertAlmostEqual(sum(row), 1.0, places=5)
            for v in row:
                self.assertGreaterEqual(v, 0.0)
                self.assertLessEqual(v, 1.0)
```
**EN:** MIS produces exactly one score vector per item. This test exercises `test_mis_one_vector_per_item` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** MIS produces exactly one score vector per item. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_one_vector_per_item`。

### Lines 417-424: test case mis single item edge case / 测试用例 mis single item edge case
```python
    def test_mis_single_item_edge_case(self):
        """Single item through MIS path."""
        scores = self.engine.score(
            query="Evaluate:", items=["Single item"], apply_softmax=True
        ).scores
        self.assertEqual(len(scores), 1)
        self.assertEqual(len(scores[0]), self.NUM_LABELS)
        self.assertAlmostEqual(sum(scores[0]), 1.0, places=5)
```
**EN:** Single item through MIS path. This test exercises `test_mis_single_item_edge_case` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Single item through MIS path. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_single_item_edge_case`。

### Lines 426-435: test case mis many items / 测试用例 mis many items
```python
    def test_mis_many_items(self):
        """10 items all return valid probability vectors."""
        items = [f"Item {i}" for i in range(10)]
        scores = self.engine.score(
            query="Classify each:", items=items, apply_softmax=True
        ).scores
        self.assertEqual(len(scores), len(items))
        for row in scores:
            self.assertEqual(len(row), self.NUM_LABELS)
            self.assertAlmostEqual(sum(row), 1.0, places=5)
```
**EN:** 10 items all return valid probability vectors. This test exercises `test_mis_many_items` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 10 items all return valid probability vectors. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_many_items`。

### Lines 436-442: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# SequenceClassification — MIS with many labels (tensor shape stress test)
# ---------------------------------------------------------------------------


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 443-443: class TestSeqClsMISAdvancedScoring declaration / 类 TestSeqClsMISAdvancedScoring 声明
```python
class TestSeqClsMISAdvancedScoring(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 444-450: class-level constants and configuration for `TestSeqClsMISAdvancedScoring` / 类级常量与配置
```python
    """SeqCls MIS with 12 labels — stresses the 2-D tensor path in score_and_pool.

    Kept in a separate class (own Engine instance) so it doesn't fight the
    2-label class-level engine for GPU memory.
    """

    NUM_LABELS = 12
```
**EN:** This block defines shared names such as `NUM_LABELS`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `NUM_LABELS` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 452-467: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.engine = Engine(
            model_path=_SEQCLS_MODEL,
            disable_radix_cache=True,
            chunked_prefill_size=-1,
            enable_mis=True,
            attention_backend="flashinfer",
            json_model_override_args=json.dumps(
                {
                    "architectures": ["Qwen3ForSequenceClassification"],
                    "num_labels": cls.NUM_LABELS,
                }
            ),
            mem_fraction_static=0.15,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 469-473: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if hasattr(cls, "engine") and cls.engine:
            cls.engine.shutdown()
        torch.cuda.empty_cache()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 475-484: test case many labels correct shape / 测试用例 many labels correct shape
```python
    def test_many_labels_correct_shape(self):
        """5 items × 12 labels — each score vector has the right length."""
        items = [f"Item {i}" for i in range(5)]
        scores = self.engine.score(
            query="Classify:", items=items, apply_softmax=True
        ).scores
        self.assertEqual(len(scores), len(items))
        for row in scores:
            self.assertEqual(len(row), self.NUM_LABELS)
            self.assertAlmostEqual(sum(row), 1.0, places=5)
```
**EN:** 5 items × 12 labels — each score vector has the right length. This test exercises `test_many_labels_correct_shape` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 5 items × 12 labels — each score vector has the right length. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_many_labels_correct_shape`。

### Lines 487-488: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=3)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestCausalLMScoring`: CausalLM scoring via Engine — correctness, batching, and edge cases. / 用于组织相关测试、夹具或辅助方法。
- `TestSeqClsScoring`: SequenceClassification scoring via Engine — no MIS delimiter. / 用于组织相关测试、夹具或辅助方法。
- `TestSeqClsMISScoring`: SeqCls MIS: all items packed into one sequence separated by delimiter token. / 用于组织相关测试、夹具或辅助方法。
- `TestSeqClsMISAdvancedScoring`: SeqCls MIS with 12 labels — stresses the 2-D tensor path in score_and_pool. / 用于组织相关测试、夹具或辅助方法。
- `TestCausalLMScoring.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestCausalLMScoring.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestCausalLMScoring._hf_scores`: Reference scores computed directly with HuggingFace (CPU inference). / 该代码块实现 `_hf_scores`，承担模块行为中的一个聚焦逻辑片段。
- `TestCausalLMScoring._assert_scores_close`: This block implements `_assert_scores_close` and captures one focused piece of the module's behavior. / 该代码块实现 `_assert_scores_close`，承担模块行为中的一个聚焦逻辑片段。
- `TestCausalLMScoring.test_scores_match_hf_reference`: SGLang scores agree with HuggingFace within 1% tolerance. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_scores_match_hf_reference`。
- `TestCausalLMScoring.test_request_avoids_decode_phase`: Internal request must have max_new_tokens=0, logprob=True, stream=False. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_request_avoids_decode_phase`。
- `TestCausalLMScoring.test_score_batch_sizes`: Correct output count and shape for batch sizes 1, 2, 4, 8. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_score_batch_sizes`。
- `TestCausalLMScoring.test_score_empty_items`: Empty items list → empty scores and zero prompt_tokens. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_score_empty_items`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `unittest`, `unittest.mock`
- **Third-party modules / 第三方模块**: `torch`, `transformers`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.engine`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 488
