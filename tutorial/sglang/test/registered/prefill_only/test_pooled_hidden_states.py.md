# test_pooled_hidden_states.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/prefill_only/test_pooled_hidden_states.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates pooled hidden states behavior in SGLang's prefill only area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 prefill only 领域中与 pooled hidden states 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: supporting statements / 辅助语句
```python
"""Tests for the return_pooled_hidden_states feature on the scoring API.

Covers both Engine-level (Python API) and HTTP-level (/v1/score) integration:

  TestPooledHiddenStatesEngine     — SeqCls model, single-item scoring
  TestPooledHiddenStatesMISEngine  — SeqCls model, MIS delimiter mode
  TestPooledHiddenStatesHTTP       — HTTP layer serialization round-trip
  TestPooledHiddenStatesCausalLMRejection — CausalLM must reject the flag

Each test class spins up its own Engine or server so GPU memory is isolated.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 13-28: module imports and dependencies / 模块导入与依赖
```python
import json
import unittest

import requests
import torch

from sglang.srt.entrypoints.engine import Engine
from sglang.srt.utils import is_hip, kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `unittest`, `requests`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `unittest`, `requests`, `torch`。

### Lines 30-42: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=100, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=100, suite="stage-b-test-1-gpu-small-amd")


_SEQCLS_MODEL = "Qwen/Qwen3-0.6B"
_CAUSAL_LM_MODEL = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
_NUM_LABELS = 4

# Local overrides for offline testing (no network).  Set to None to use HF hub.
_LOCAL_SEQCLS_MODEL = (
    "/shared/public/elr-models/Qwen/Qwen3-0.6B/e6de91484c29aa9480d55605af694f39b081c455"
)
_LOCAL_CAUSAL_LM_MODEL = "/shared/public/elr-models/meta-llama/Llama-3.2-1B-Instruct/e9f8effbab1cbdc515c11ee6e098e3d5a9f51e14"
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 44-44: module imports and dependencies / 模块导入与依赖
```python
import os
```
**EN:** This block imports the modules needed by the rest of the file, including `os`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `os`。

### Lines 46-49: module-level constants and configuration / 模块级常量与配置
```python
if _LOCAL_SEQCLS_MODEL and os.path.isdir(_LOCAL_SEQCLS_MODEL):
    _SEQCLS_MODEL = _LOCAL_SEQCLS_MODEL
if _LOCAL_CAUSAL_LM_MODEL and os.path.isdir(_LOCAL_CAUSAL_LM_MODEL):
    _CAUSAL_LM_MODEL = _LOCAL_CAUSAL_LM_MODEL
```
**EN:** This block defines shared names such as `_SEQCLS_MODEL`, `_CAUSAL_LM_MODEL`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `_SEQCLS_MODEL`, `_CAUSAL_LM_MODEL` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 50-56: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# Engine — single-item scoring (no MIS)
# ---------------------------------------------------------------------------


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 57-57: class TestPooledHiddenStatesEngine declaration / 类 TestPooledHiddenStatesEngine 声明
```python
class TestPooledHiddenStatesEngine(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 58-62: supporting statements / 辅助语句
```python
    """Validates return_pooled_hidden_states through the Engine Python API.

    Uses Qwen3ForSequenceClassification with a random head so we only care
    about shape and pipeline plumbing, not numerical accuracy.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 64-76: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.engine = Engine(
            model_path=_SEQCLS_MODEL,
            disable_radix_cache=True,
            json_model_override_args=json.dumps(
                {
                    "architectures": ["Qwen3ForSequenceClassification"],
                    "num_labels": _NUM_LABELS,
                }
            ),
            mem_fraction_static=0.15,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 78-82: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if hasattr(cls, "engine") and cls.engine:
            cls.engine.shutdown()
        torch.cuda.empty_cache()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 84-96: test case phs returned when requested / 测试用例 phs returned when requested
```python
    def test_phs_returned_when_requested(self):
        """Pooled hidden states are present and shaped correctly."""
        result = self.engine.score(
            query="Rate each:",
            items=["Good", "Bad"],
            return_pooled_hidden_states=True,
        )
        self.assertIsNotNone(result.pooled_hidden_states)
        self.assertEqual(len(result.pooled_hidden_states), 2)
        for phs in result.pooled_hidden_states:
            self.assertIsInstance(phs, torch.Tensor)
            self.assertEqual(phs.dim(), 1)
            self.assertGreater(phs.shape[0], 0)
```
**EN:** Pooled hidden states are present and shaped correctly. This test exercises `test_phs_returned_when_requested` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Pooled hidden states are present and shaped correctly. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_phs_returned_when_requested`。

### Lines 98-105: test case phs none when not requested / 测试用例 phs none when not requested
```python
    def test_phs_none_when_not_requested(self):
        """Without the flag, pooled_hidden_states must be None."""
        result = self.engine.score(
            query="Rate each:",
            items=["Good", "Bad"],
            return_pooled_hidden_states=False,
        )
        self.assertIsNone(result.pooled_hidden_states)
```
**EN:** Without the flag, pooled_hidden_states must be None. This test exercises `test_phs_none_when_not_requested` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Without the flag, pooled_hidden_states must be None. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_phs_none_when_not_requested`。

### Lines 107-117: test case phs shape is consistent / 测试用例 phs shape is consistent
```python
    def test_phs_shape_is_consistent(self):
        """PHS tensors for different items share the same hidden dimension."""
        result = self.engine.score(
            query="Evaluate:",
            items=["Alpha", "Beta", "Gamma"],
            return_pooled_hidden_states=True,
        )
        self.assertIsNotNone(result.pooled_hidden_states)
        dims = {phs.shape[0] for phs in result.pooled_hidden_states}
        self.assertEqual(len(dims), 1, "All PHS vectors must share the same hidden dim")
        self.assertGreater(dims.pop(), 0)
```
**EN:** PHS tensors for different items share the same hidden dimension. This test exercises `test_phs_shape_is_consistent` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** PHS tensors for different items share the same hidden dimension. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_phs_shape_is_consistent`。

### Lines 119-129: test case phs count matches items / 测试用例 phs count matches items
```python
    def test_phs_count_matches_items(self):
        """Number of PHS tensors equals number of items for various batch sizes."""
        for n in [1, 3, 5]:
            with self.subTest(n=n):
                result = self.engine.score(
                    query="Classify:",
                    items=[f"Item {i}" for i in range(n)],
                    return_pooled_hidden_states=True,
                )
                self.assertIsNotNone(result.pooled_hidden_states)
                self.assertEqual(len(result.pooled_hidden_states), n)
```
**EN:** Number of PHS tensors equals number of items for various batch sizes. This test exercises `test_phs_count_matches_items` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Number of PHS tensors equals number of items for various batch sizes. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_phs_count_matches_items`。

### Lines 131-139: test case phs on cpu / 测试用例 phs on cpu
```python
    def test_phs_on_cpu(self):
        """Returned tensors live on CPU (no GPU references leak to caller)."""
        result = self.engine.score(
            query="Check device:",
            items=["Test"],
            return_pooled_hidden_states=True,
        )
        for phs in result.pooled_hidden_states:
            self.assertEqual(str(phs.device), "cpu")
```
**EN:** Returned tensors live on CPU (no GPU references leak to caller). This test exercises `test_phs_on_cpu` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Returned tensors live on CPU (no GPU references leak to caller). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_phs_on_cpu`。

### Lines 141-152: test case phs deterministic / 测试用例 phs deterministic
```python
    def test_phs_deterministic(self):
        """Identical requests produce identical PHS tensors."""
        kwargs = dict(
            query="Evaluate:", items=["A", "B"], return_pooled_hidden_states=True
        )
        phs1 = self.engine.score(**kwargs).pooled_hidden_states
        phs2 = self.engine.score(**kwargs).pooled_hidden_states
        for t1, t2 in zip(phs1, phs2):
            self.assertTrue(
                torch.allclose(t1, t2, atol=1e-5),
                "Pooled hidden states differ across identical requests",
            )
```
**EN:** Identical requests produce identical PHS tensors. This test exercises `test_phs_deterministic` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Identical requests produce identical PHS tensors. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_phs_deterministic`。

### Lines 154-166: test case scores unaffected by phs flag / 测试用例 scores unaffected by phs flag
```python
    def test_scores_unaffected_by_phs_flag(self):
        """The phs flag must not change the scores themselves (fp16 tolerance)."""
        kwargs = dict(query="Rate:", items=["X", "Y", "Z"], apply_softmax=True)
        scores_without = self.engine.score(
            **kwargs, return_pooled_hidden_states=False
        ).scores
        scores_with = self.engine.score(
            **kwargs, return_pooled_hidden_states=True
        ).scores
        self.assertEqual(len(scores_without), len(scores_with))
        for row_a, row_b in zip(scores_without, scores_with):
            for a, b in zip(row_a, row_b):
                self.assertAlmostEqual(a, b, places=2)
```
**EN:** The phs flag must not change the scores themselves (fp16 tolerance). This test exercises `test_scores_unaffected_by_phs_flag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** The phs flag must not change the scores themselves (fp16 tolerance). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_scores_unaffected_by_phs_flag`。

### Lines 168-180: test case phs with tokenized inputs / 测试用例 phs with tokenized inputs
```python
    def test_phs_with_tokenized_inputs(self):
        """Pre-tokenized inputs also return PHS correctly."""
        from transformers import AutoTokenizer

        tok = AutoTokenizer.from_pretrained(_SEQCLS_MODEL)
        query, items = "Evaluate:", ["Alpha", "Beta"]
        result = self.engine.score(
            query=tok.encode(query),
            items=[tok.encode(i) for i in items],
            return_pooled_hidden_states=True,
        )
        self.assertIsNotNone(result.pooled_hidden_states)
        self.assertEqual(len(result.pooled_hidden_states), 2)
```
**EN:** Pre-tokenized inputs also return PHS correctly. This test exercises `test_phs_with_tokenized_inputs` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Pre-tokenized inputs also return PHS correctly. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_phs_with_tokenized_inputs`。

### Lines 181-187: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# Engine — MIS delimiter mode
# ---------------------------------------------------------------------------


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 188-193: class TestPooledHiddenStatesMISEngine declaration / 类 TestPooledHiddenStatesMISEngine 声明
```python
@unittest.skipIf(
    is_hip(),
    "Multi-Item Scoring (enable_mis) requires the flashinfer prefill/decode "
    "backend, which is NVIDIA-only.",
)
class TestPooledHiddenStatesMISEngine(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 194-198: supporting statements / 辅助语句
```python
    """Validates return_pooled_hidden_states in MIS (delimiter) scoring mode.

    MIS packs all items into one sequence; the PHS at each delimiter position
    should be returned per-item.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 200-214: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.engine = Engine(
            model_path=_SEQCLS_MODEL,
            disable_radix_cache=True,
            chunked_prefill_size=-1,
            enable_mis=True,
            json_model_override_args=json.dumps(
                {
                    "architectures": ["Qwen3ForSequenceClassification"],
                    "num_labels": _NUM_LABELS,
                }
            ),
            mem_fraction_static=0.15,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 216-220: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if hasattr(cls, "engine") and cls.engine:
            cls.engine.shutdown()
        torch.cuda.empty_cache()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 222-229: test case mis phs count matches items / 测试用例 mis phs count matches items
```python
    def test_mis_phs_count_matches_items(self):
        """MIS must return one PHS tensor per item."""
        items = ["Option A", "Option B", "Option C"]
        result = self.engine.score(
            query="Rate each:", items=items, return_pooled_hidden_states=True
        )
        self.assertIsNotNone(result.pooled_hidden_states)
        self.assertEqual(len(result.pooled_hidden_states), len(items))
```
**EN:** MIS must return one PHS tensor per item. This test exercises `test_mis_phs_count_matches_items` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** MIS must return one PHS tensor per item. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_phs_count_matches_items`。

### Lines 231-237: test case mis phs none when not requested / 测试用例 mis phs none when not requested
```python
    def test_mis_phs_none_when_not_requested(self):
        result = self.engine.score(
            query="Rate each:",
            items=["A", "B"],
            return_pooled_hidden_states=False,
        )
        self.assertIsNone(result.pooled_hidden_states)
```
**EN:** This test exercises `test_mis_phs_none_when_not_requested` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_phs_none_when_not_requested`。

### Lines 239-245: test case mis phs are tensors on cpu / 测试用例 mis phs are tensors on cpu
```python
    def test_mis_phs_are_tensors_on_cpu(self):
        result = self.engine.score(
            query="Classify:", items=["X", "Y"], return_pooled_hidden_states=True
        )
        for phs in result.pooled_hidden_states:
            self.assertIsInstance(phs, torch.Tensor)
            self.assertEqual(str(phs.device), "cpu")
```
**EN:** This test exercises `test_mis_phs_are_tensors_on_cpu` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_phs_are_tensors_on_cpu`。

### Lines 247-261: test case mis phs different items different hidden states / 测试用例 mis phs different items different hidden states
```python
    def test_mis_phs_different_items_different_hidden_states(self):
        """Different items should produce distinct PHS vectors."""
        items = [
            "Option A is about cats",
            "Option B is about dogs",
            "Option C is about fish",
        ]
        result = self.engine.score(
            query="Classify:", items=items, return_pooled_hidden_states=True
        )
        phs = result.pooled_hidden_states
        self.assertFalse(
            all(torch.allclose(phs[0], p, atol=1e-6) for p in phs[1:]),
            "All MIS items returned identical hidden states",
        )
```
**EN:** Different items should produce distinct PHS vectors. This test exercises `test_mis_phs_different_items_different_hidden_states` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Different items should produce distinct PHS vectors. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_phs_different_items_different_hidden_states`。

### Lines 263-269: test case mis single item / 测试用例 mis single item
```python
    def test_mis_single_item(self):
        """Single item through MIS path still returns one PHS tensor."""
        result = self.engine.score(
            query="Evaluate:", items=["Only one"], return_pooled_hidden_states=True
        )
        self.assertIsNotNone(result.pooled_hidden_states)
        self.assertEqual(len(result.pooled_hidden_states), 1)
```
**EN:** Single item through MIS path still returns one PHS tensor. This test exercises `test_mis_single_item` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Single item through MIS path still returns one PHS tensor. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_single_item`。

### Lines 271-280: test case mis many items / 测试用例 mis many items
```python
    def test_mis_many_items(self):
        """10 items all produce PHS tensors of consistent shape."""
        items = [f"Item {i}" for i in range(10)]
        result = self.engine.score(
            query="Classify:", items=items, return_pooled_hidden_states=True
        )
        self.assertIsNotNone(result.pooled_hidden_states)
        self.assertEqual(len(result.pooled_hidden_states), len(items))
        shapes = {phs.shape for phs in result.pooled_hidden_states}
        self.assertEqual(len(shapes), 1, "MIS PHS shapes should be uniform")
```
**EN:** 10 items all produce PHS tensors of consistent shape. This test exercises `test_mis_many_items` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 10 items all produce PHS tensors of consistent shape. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_many_items`。

### Lines 282-295: test case mis scores unaffected by phs flag / 测试用例 mis scores unaffected by phs flag
```python
    def test_mis_scores_unaffected_by_phs_flag(self):
        """Enabling PHS does not alter the returned scores (fp16 tolerance)."""
        kwargs = dict(
            query="Rate:", items=["Alpha", "Beta", "Gamma"], apply_softmax=True
        )
        scores_without = self.engine.score(
            **kwargs, return_pooled_hidden_states=False
        ).scores
        scores_with = self.engine.score(
            **kwargs, return_pooled_hidden_states=True
        ).scores
        for row_a, row_b in zip(scores_without, scores_with):
            for a, b in zip(row_a, row_b):
                self.assertAlmostEqual(a, b, places=2)
```
**EN:** Enabling PHS does not alter the returned scores (fp16 tolerance). This test exercises `test_mis_scores_unaffected_by_phs_flag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Enabling PHS does not alter the returned scores (fp16 tolerance). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_scores_unaffected_by_phs_flag`。

### Lines 296-302: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# CausalLM rejection
# ---------------------------------------------------------------------------


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 303-303: class TestPooledHiddenStatesCausalLMRejection declaration / 类 TestPooledHiddenStatesCausalLMRejection 声明
```python
class TestPooledHiddenStatesCausalLMRejection(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 304-304: supporting statements / 辅助语句
```python
    """CausalLM models must reject return_pooled_hidden_states=True."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 306-308: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.engine = Engine(model_path=_CAUSAL_LM_MODEL)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 310-314: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if hasattr(cls, "engine") and cls.engine:
            cls.engine.shutdown()
        torch.cuda.empty_cache()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 316-325: test case causal lm rejects phs / 测试用例 causal lm rejects phs
```python
    def test_causal_lm_rejects_phs(self):
        """ValueError raised when requesting PHS from a CausalLM."""
        with self.assertRaises(ValueError) as ctx:
            self.engine.score(
                query="Test",
                items=["Item"],
                label_token_ids=[1, 2],
                return_pooled_hidden_states=True,
            )
        self.assertIn("CausalLM", str(ctx.exception))
```
**EN:** ValueError raised when requesting PHS from a CausalLM. This test exercises `test_causal_lm_rejects_phs` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** ValueError raised when requesting PHS from a CausalLM. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_causal_lm_rejects_phs`。

### Lines 327-337: test case causal lm without phs still works / 测试用例 causal lm without phs still works
```python
    def test_causal_lm_without_phs_still_works(self):
        """Baseline: CausalLM scoring without the flag works fine."""
        result = self.engine.score(
            query="Test",
            items=["Item"],
            label_token_ids=[1, 2],
            apply_softmax=True,
            return_pooled_hidden_states=False,
        )
        self.assertEqual(len(result.scores), 1)
        self.assertIsNone(result.pooled_hidden_states)
```
**EN:** Baseline: CausalLM scoring without the flag works fine. This test exercises `test_causal_lm_without_phs_still_works` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Baseline: CausalLM scoring without the flag works fine. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_causal_lm_without_phs_still_works`。

### Lines 338-344: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# HTTP layer
# ---------------------------------------------------------------------------


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 345-345: class TestPooledHiddenStatesHTTP declaration / 类 TestPooledHiddenStatesHTTP 声明
```python
class TestPooledHiddenStatesHTTP(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 346-350: supporting statements / 辅助语句
```python
    """HTTP integration: /v1/score with return_pooled_hidden_states.

    Validates that the Pydantic schema, JSON serialization, and ORJSONResponse
    round-trip preserves the pooled hidden states as nested lists.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 352-372: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = _SEQCLS_MODEL
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--disable-radix-cache",
                "--json-model-override-args",
                json.dumps(
                    {
                        "architectures": ["Qwen3ForSequenceClassification"],
                        "num_labels": _NUM_LABELS,
                    }
                ),
                "--mem-fraction-static",
                "0.15",
            ],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 374-377: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if hasattr(cls, "process") and cls.process:
            kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 379-380: method post / 方法 post
```python
    def _post(self, payload):
        return requests.post(self.base_url + "/v1/score", json=payload)
```
**EN:** This block implements `_post` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_post`，承担模块行为中的一个聚焦逻辑片段。

### Lines 382-401: test case phs in response json / 测试用例 phs in response json
```python
    def test_phs_in_response_json(self):
        """Response includes pooled_hidden_states as nested float lists."""
        resp = self._post(
            {
                "query": "Rate each:",
                "items": ["Good", "Bad"],
                "return_pooled_hidden_states": True,
                "model": self.model,
            }
        )
        self.assertEqual(resp.status_code, 200)
        body = resp.json()
        phs = body.get("pooled_hidden_states")
        self.assertIsNotNone(phs)
        self.assertEqual(len(phs), 2)
        for item_phs in phs:
            self.assertIsInstance(item_phs, list)
            self.assertGreater(len(item_phs), 0)
            for v in item_phs:
                self.assertIsInstance(v, float)
```
**EN:** Response includes pooled_hidden_states as nested float lists. This test exercises `test_phs_in_response_json` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Response includes pooled_hidden_states as nested float lists. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_phs_in_response_json`。

### Lines 403-414: test case phs absent when not requested / 测试用例 phs absent when not requested
```python
    def test_phs_absent_when_not_requested(self):
        """Without the flag, pooled_hidden_states is null in JSON."""
        resp = self._post(
            {
                "query": "Rate each:",
                "items": ["Good"],
                "model": self.model,
            }
        )
        self.assertEqual(resp.status_code, 200)
        body = resp.json()
        self.assertIsNone(body.get("pooled_hidden_states"))
```
**EN:** Without the flag, pooled_hidden_states is null in JSON. This test exercises `test_phs_absent_when_not_requested` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Without the flag, pooled_hidden_states is null in JSON. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_phs_absent_when_not_requested`。

### Lines 416-429: test case phs matches item count / 测试用例 phs matches item count
```python
    def test_phs_matches_item_count(self):
        """Number of PHS vectors equals number of items."""
        items = ["A", "B", "C", "D"]
        resp = self._post(
            {
                "query": "Classify:",
                "items": items,
                "return_pooled_hidden_states": True,
                "model": self.model,
            }
        )
        self.assertEqual(resp.status_code, 200)
        phs = resp.json()["pooled_hidden_states"]
        self.assertEqual(len(phs), len(items))
```
**EN:** Number of PHS vectors equals number of items. This test exercises `test_phs_matches_item_count` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Number of PHS vectors equals number of items. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_phs_matches_item_count`。

### Lines 432-433: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=3)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestPooledHiddenStatesEngine`: Validates return_pooled_hidden_states through the Engine Python API. / 用于组织相关测试、夹具或辅助方法。
- `TestPooledHiddenStatesMISEngine`: Validates return_pooled_hidden_states in MIS (delimiter) scoring mode. / 用于组织相关测试、夹具或辅助方法。
- `TestPooledHiddenStatesCausalLMRejection`: CausalLM models must reject return_pooled_hidden_states=True. / 用于组织相关测试、夹具或辅助方法。
- `TestPooledHiddenStatesHTTP`: HTTP integration: /v1/score with return_pooled_hidden_states. / 用于组织相关测试、夹具或辅助方法。
- `TestPooledHiddenStatesEngine.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestPooledHiddenStatesEngine.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestPooledHiddenStatesEngine.test_phs_returned_when_requested`: Pooled hidden states are present and shaped correctly. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_phs_returned_when_requested`。
- `TestPooledHiddenStatesEngine.test_phs_none_when_not_requested`: Without the flag, pooled_hidden_states must be None. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_phs_none_when_not_requested`。
- `TestPooledHiddenStatesEngine.test_phs_shape_is_consistent`: PHS tensors for different items share the same hidden dimension. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_phs_shape_is_consistent`。
- `TestPooledHiddenStatesEngine.test_phs_count_matches_items`: Number of PHS tensors equals number of items for various batch sizes. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_phs_count_matches_items`。
- `TestPooledHiddenStatesEngine.test_phs_on_cpu`: Returned tensors live on CPU (no GPU references leak to caller). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_phs_on_cpu`。
- `TestPooledHiddenStatesEngine.test_phs_deterministic`: Identical requests produce identical PHS tensors. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_phs_deterministic`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `unittest`, `os`
- **Third-party modules / 第三方模块**: `requests`, `torch`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.engine`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 433
