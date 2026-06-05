# test_serving_rerank.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/prefill_only/test_serving_rerank.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates serving rerank behavior in SGLang's prefill only area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 prefill only 领域中与 serving rerank 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: module imports and dependencies / 模块导入与依赖
```python
import asyncio
import unittest
from unittest.mock import Mock

from sglang.srt.entrypoints.openai.protocol import V1RerankReqInput
from sglang.srt.managers.tokenizer_manager_score_mixin import ScoreResult
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `asyncio`, `unittest`, `unittest.mock`, `sglang.srt.entrypoints.openai.protocol`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `asyncio`, `unittest`, `unittest.mock`, `sglang.srt.entrypoints.openai.protocol`。

### Lines 8-9: supporting source context / 辅助源码上下文
```python

# Keep consistent with other openai_server/basic unit tests.
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 10-26: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=9, stage="base-b", runner_config="1-gpu-large")
register_amd_ci(est_time=10, suite="stage-b-test-1-gpu-small-amd")

try:
    from sglang.srt.entrypoints.openai.serving_rerank import (
        OpenAIServingRerank,
        _is_qwen3_reranker_template,
        _qwen3_rerank_score,
        _render_jinja_chat_template,
    )
except ModuleNotFoundError as e:
    # Some minimal environments used for unit tests may not have FastAPI/torch installed.
    # Skip this test in that case.
    if e.name in ("fastapi", "torch"):
        OpenAIServingRerank = None  # type: ignore[assignment]
    else:
        raise
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 29-30: class _DummyModelConfig declaration / 类 _DummyModelConfig 声明
```python
class _DummyModelConfig:
    # Keep consistent with TokenizerManager.model_config usage
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 31-31: class-level constants and configuration for `_DummyModelConfig` / 类级常量与配置
```python
    is_generation = False
```
**EN:** This block defines shared names such as `is_generation`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `is_generation` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 34-34: class _DummyTokenizer declaration / 类 _DummyTokenizer 声明
```python
class _DummyTokenizer:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 35-35: class-level constants and configuration for `_DummyTokenizer` / 类级常量与配置
```python
    chat_template = ""
```
**EN:** This block defines shared names such as `chat_template`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `chat_template` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 38-39: class _DummyTokenizerManager declaration / 类 _DummyTokenizerManager 声明
```python
class _DummyTokenizerManager:
    # Minimal surface required by OpenAIServingBase/OpenAIServingRerank
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 40-42: class-level constants and configuration for `_DummyTokenizerManager` / 类级常量与配置
```python
    server_args = object()
    model_config = _DummyModelConfig()
    tokenizer = _DummyTokenizer()
```
**EN:** This block defines shared names such as `server_args`, `model_config`, `tokenizer`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `server_args`, `model_config`, `tokenizer` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 44-45: method generate request / 方法 generate request
```python
    async def generate_request(self, *_args, **_kwargs):
        raise AssertionError("generate_request should not be called in this unit test")
```
**EN:** This block implements `generate_request` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `generate_request`，承担模块行为中的一个聚焦逻辑片段。

### Lines 48-49: class TestOpenAIServingRerankUnit declaration / 类 TestOpenAIServingRerankUnit 声明
```python
@unittest.skipIf(OpenAIServingRerank is None, "fastapi/torch is not installed")
class TestOpenAIServingRerankUnit(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 50-51: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.handler = OpenAIServingRerank(_DummyTokenizerManager())
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 53-66: test case convert to internal request cross encoder pairs / 测试用例 convert to internal request cross encoder pairs
```python
    def test_convert_to_internal_request_cross_encoder_pairs(self):
        req = V1RerankReqInput(
            query="q",
            documents=["doc-a", "doc-b"],
            instruct="Retrieve semantically similar text.",
        )

        adapted, processed = self.handler._convert_to_internal_request(req)

        # Avoid importing EmbeddingReqInput (requires torch). Use duck-typing checks instead.
        self.assertTrue(hasattr(adapted, "is_cross_encoder_request"))
        self.assertTrue(adapted.is_cross_encoder_request)
        self.assertEqual(getattr(adapted, "text"), [["q", "doc-a"], ["q", "doc-b"]])
        self.assertEqual(processed, req)
```
**EN:** This test exercises `test_convert_to_internal_request_cross_encoder_pairs` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_convert_to_internal_request_cross_encoder_pairs`。

### Lines 68-77: test case convert to internal request qwen3 template returns request / 测试用例 convert to internal request qwen3 template returns request
```python
    def test_convert_to_internal_request_qwen3_template_returns_request(self):
        tm = _DummyTokenizerManager()
        tm.tokenizer.chat_template = (
            '... Note that the answer can only be "yes" or "no". ...'
        )
        handler = OpenAIServingRerank(tm)
        req = V1RerankReqInput(query="q", documents=["d1"])
        adapted, processed = handler._convert_to_internal_request(req)
        self.assertIs(adapted, req)
        self.assertIs(processed, req)
```
**EN:** This test exercises `test_convert_to_internal_request_qwen3_template_returns_request` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_convert_to_internal_request_qwen3_template_returns_request`。

### Lines 79-105: test case build rerank response embedding list uses first scalar / 测试用例 build rerank response embedding list uses first scalar
```python
    def test_build_rerank_response_embedding_list_uses_first_scalar(self):
        req = V1RerankReqInput(
            query="q",
            documents=["doc-a", "doc-b"],
            return_documents=True,
        )
        # Two results with embedding as list, should coerce embedding[0] to float.
        # Also verifies sorting (doc-b > doc-a).
        ret = [
            {"embedding": [0.1, 0.2], "meta_info": {"id": "a"}},
            {"embedding": [0.9, -1.0], "meta_info": {"id": "b"}},
        ]

        res = self.handler._build_rerank_response(ret, req)

        self.assertEqual(len(res), 2)

        # Sorted descending by score, so doc-b first.
        self.assertEqual(res[0].document, "doc-b")
        self.assertEqual(res[0].index, 1)
        self.assertAlmostEqual(res[0].score, 0.9)
        self.assertEqual(res[0].meta_info, {"id": "b"})

        self.assertEqual(res[1].document, "doc-a")
        self.assertEqual(res[1].index, 0)
        self.assertAlmostEqual(res[1].score, 0.1)
        self.assertEqual(res[1].meta_info, {"id": "a"})
```
**EN:** This test exercises `test_build_rerank_response_embedding_list_uses_first_scalar` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_build_rerank_response_embedding_list_uses_first_scalar`。

### Lines 107-117: test case build rerank response float list / 测试用例 build rerank response float list
```python
    def test_build_rerank_response_float_list(self):
        req = V1RerankReqInput(
            query="q", documents=["a", "b", "c"], return_documents=True
        )
        scores = [0.2, 0.9, 0.1]
        res = self.handler._build_rerank_response(scores, req)
        self.assertEqual([r.document for r in res], ["b", "a", "c"])
        self.assertEqual([r.index for r in res], [1, 0, 2])
        self.assertAlmostEqual(res[0].score, 0.9)
        self.assertAlmostEqual(res[1].score, 0.2)
        self.assertAlmostEqual(res[2].score, 0.1)
```
**EN:** This test exercises `test_build_rerank_response_float_list` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_build_rerank_response_float_list`。

### Lines 119-125: test case helper is qwen3 reranker template / 测试用例 helper is qwen3 reranker template
```python
    def test_helper_is_qwen3_reranker_template(self):
        self.assertTrue(
            _is_qwen3_reranker_template(
                'Note that the answer can only be "yes" or "no".'
            )
        )
        self.assertFalse(_is_qwen3_reranker_template("plain template"))
```
**EN:** This test exercises `test_helper_is_qwen3_reranker_template` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_helper_is_qwen3_reranker_template`。

### Lines 127-129: test case helper qwen3 rerank score / 测试用例 helper qwen3 rerank score
```python
    def test_helper_qwen3_rerank_score(self):
        self.assertAlmostEqual(_qwen3_rerank_score(0.9, 0.1), 0.9)
        self.assertAlmostEqual(_qwen3_rerank_score(0.0, 0.0), 0.0)
```
**EN:** This test exercises `test_helper_qwen3_rerank_score` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_helper_qwen3_rerank_score`。

### Lines 131-146: test case helper render jinja chat template / 测试用例 helper render jinja chat template
```python
    def test_helper_render_jinja_chat_template(self):
        # Skip if jinja2 isn't installed in this environment.
        try:
            import jinja2  # noqa: F401
        except ModuleNotFoundError:
            self.skipTest("jinja2 is not installed")

        tpl = "{{ instruct | default('DEF') }}|{{ messages[0]['content'] }}|{{ messages[1]['content'] }}"
        self.assertEqual(
            _render_jinja_chat_template(tpl, query="Q", document="D", instruct=None),
            "DEF|Q|D",
        )
        self.assertEqual(
            _render_jinja_chat_template(tpl, query="Q", document="D", instruct="I"),
            "I|Q|D",
        )
```
**EN:** This test exercises `test_helper_render_jinja_chat_template` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_helper_render_jinja_chat_template`。

### Lines 148-179: test case handle non streaming request qwen3 path uses score prompts / 测试用例 handle non streaming request qwen3 path uses score prompts
```python
    def test_handle_non_streaming_request_qwen3_path_uses_score_prompts(self):
        class _TM(_DummyTokenizerManager):
            def __init__(self):
                self.server_args = object()
                self.model_config = Mock()
                self.model_config.is_generation = True
                self.model_config.model_path = "qwen/qwen3"
                self.tokenizer = Mock()
                self.tokenizer.chat_template = (
                    'Note that the answer can only be "yes" or "no". '
                    "{{ messages[0]['content'] }} {{ messages[1]['content'] }}"
                )

            async def score_prompts(
                self, prompts, label_token_ids, apply_softmax, request
            ):
                # Return [p_yes, p_no] for each prompt
                assert len(prompts) == 2
                assert label_token_ids and len(label_token_ids) == 2
                return ScoreResult(scores=[[0.9, 0.1], [0.2, 0.8]], prompt_tokens=42)

        handler = OpenAIServingRerank(_TM())
        req = V1RerankReqInput(query="q", documents=["d1", "d2"], return_documents=True)
        adapted, _ = handler._convert_to_internal_request(req)
        raw_request = Mock()

        res = asyncio.run(
            handler._handle_non_streaming_request(adapted, req, raw_request)
        )
        self.assertEqual([r.document for r in res], ["d1", "d2"])
        self.assertAlmostEqual(res[0].score, 0.9 / (0.9 + 0.1))
        self.assertAlmostEqual(res[1].score, 0.2 / (0.2 + 0.8))
```
**EN:** This test exercises `test_handle_non_streaming_request_qwen3_path_uses_score_prompts` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_handle_non_streaming_request_qwen3_path_uses_score_prompts`。

### Lines 181-192: test case build rerank response return documents false / 测试用例 build rerank response return documents false
```python
    def test_build_rerank_response_return_documents_false(self):
        """Test that document field is None when return_documents=False"""
        req = V1RerankReqInput(
            query="q", documents=["a", "b", "c"], return_documents=False
        )
        scores = [0.2, 0.9, 0.1]
        res = self.handler._build_rerank_response(scores, req)
        # All documents should be None
        self.assertEqual([r.document for r in res], [None, None, None])
        # But scores and indices should still be correct
        self.assertEqual([r.index for r in res], [1, 0, 2])
        self.assertAlmostEqual(res[0].score, 0.9)
```
**EN:** Test that document field is None when return_documents=False This test exercises `test_build_rerank_response_return_documents_false` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that document field is None when return_documents=False 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_build_rerank_response_return_documents_false`。

### Lines 194-206: test case build rerank response top n / 测试用例 build rerank response top n
```python
    def test_build_rerank_response_top_n(self):
        """Test that top_n limits the number of returned results"""
        req = V1RerankReqInput(
            query="q", documents=["a", "b", "c"], return_documents=True, top_n=2
        )
        scores = [0.2, 0.9, 0.1]
        res = self.handler._build_rerank_response(scores, req)
        # Should only return top 2 results
        self.assertEqual(len(res), 2)
        self.assertEqual([r.document for r in res], ["b", "a"])
        self.assertEqual([r.index for r in res], [1, 0])
        self.assertAlmostEqual(res[0].score, 0.9)
        self.assertAlmostEqual(res[1].score, 0.2)
```
**EN:** Test that top_n limits the number of returned results This test exercises `test_build_rerank_response_top_n` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that top_n limits the number of returned results 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_build_rerank_response_top_n`。

### Lines 208-217: test case build rerank response top n greater than total / 测试用例 build rerank response top n greater than total
```python
    def test_build_rerank_response_top_n_greater_than_total(self):
        """Test that top_n greater than total documents returns all documents"""
        req = V1RerankReqInput(
            query="q", documents=["a", "b"], return_documents=True, top_n=10
        )
        scores = [0.2, 0.9]
        res = self.handler._build_rerank_response(scores, req)
        # Should return all 2 documents even though top_n=10
        self.assertEqual(len(res), 2)
        self.assertEqual([r.document for r in res], ["b", "a"])
```
**EN:** Test that top_n greater than total documents returns all documents This test exercises `test_build_rerank_response_top_n_greater_than_total` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that top_n greater than total documents returns all documents 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_build_rerank_response_top_n_greater_than_total`。

### Lines 219-230: test case build rerank response top n with return documents false / 测试用例 build rerank response top n with return documents false
```python
    def test_build_rerank_response_top_n_with_return_documents_false(self):
        """Test top_n works correctly with return_documents=False"""
        req = V1RerankReqInput(
            query="q", documents=["a", "b", "c"], return_documents=False, top_n=1
        )
        scores = [0.2, 0.9, 0.1]
        res = self.handler._build_rerank_response(scores, req)
        # Should only return top 1 result, and document should be None
        self.assertEqual(len(res), 1)
        self.assertIsNone(res[0].document)
        self.assertEqual(res[0].index, 1)
        self.assertAlmostEqual(res[0].score, 0.9)
```
**EN:** Test top_n works correctly with return_documents=False This test exercises `test_build_rerank_response_top_n_with_return_documents_false` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test top_n works correctly with return_documents=False 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_build_rerank_response_top_n_with_return_documents_false`。

### Lines 232-306: test case handle vl reranker request / 测试用例 handle vl reranker request
```python
    def test_handle_vl_reranker_request(self):
        """Test the Qwen3-VL reranker path with mocked logprobs."""
        import math

        # Mock tokenizer manager that supports generate_request
        class _AsyncGen:
            def __init__(self, val):
                self.val = val

            def __aiter__(self):
                return self

            async def __anext__(self):
                return self.val

        class _TM(_DummyTokenizerManager):
            def __init__(self):
                self.server_args = object()
                self.model_config = Mock()
                self.model_config.is_generation = True
                self.model_config.model_path = "qwen/qwen3-vl"
                self.tokenizer = Mock()
                # Mock VL template detection
                self.tokenizer.chat_template = (
                    "{% for x in query %}{{ x.text }}{% endfor %}"
                    "{% for x in document %}{{ x.text }}{% endfor %}"
                    'answer can only be "yes" or "no" <|vision_start|>'
                )

            async def generate_request(self, req, _raw):
                # Return logprobs for yes/no
                # Mock logprobs: P(yes) > P(no) for first doc, P(no) > P(yes) for second

                if not hasattr(self, "call_count"):
                    self.call_count = 0

                if self.call_count == 0:
                    # First doc: yes is likely
                    yes_logprob = math.log(0.8)
                    no_logprob = math.log(0.2)
                else:
                    # Second doc: no is likely
                    yes_logprob = math.log(0.3)
                    no_logprob = math.log(0.7)

                self.call_count += 1

                # Qwen3 token IDs: YES=9693, NO=2152
                top_logprobs = [
                    (yes_logprob, 9693, "yes"),
                    (no_logprob, 2152, "no"),
                ]

                # The rerank handler checks output_top_logprobs[0] for the first generated token
                meta_info = {"output_top_logprobs": [top_logprobs]}

                yield {"meta_info": meta_info, "embedding": None}

        handler = OpenAIServingRerank(_TM())
        req = V1RerankReqInput(
            query="query", documents=["doc1", "doc2"], return_documents=True
        )
        # Force VL path is handled by detection logic inside handler
        # We mocked chat_template to satisfy _is_qwen3_vl_reranker_template

        raw_request = Mock()
        res = asyncio.run(handler._handle_non_streaming_request(req, req, raw_request))

        self.assertEqual(len(res), 2)
        # First doc should have higher score
        self.assertEqual(res[0].document, "doc1")
        self.assertAlmostEqual(res[0].score, 0.8)  # 0.8 / (0.8+0.2) = 0.8

        self.assertEqual(res[1].document, "doc2")
        self.assertAlmostEqual(res[1].score, 0.3)  # 0.3 / (0.3+0.7) = 0.3
```
**EN:** Test the Qwen3-VL reranker path with mocked logprobs. This test exercises `test_handle_vl_reranker_request` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test the Qwen3-VL reranker path with mocked logprobs. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_handle_vl_reranker_request`。

### Lines 309-310: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=2)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_DummyModelConfig`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_DummyTokenizer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_DummyTokenizerManager`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestOpenAIServingRerankUnit`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_DummyTokenizerManager.generate_request`: This block implements `generate_request` and captures one focused piece of the module's behavior. / 该代码块实现 `generate_request`，承担模块行为中的一个聚焦逻辑片段。
- `TestOpenAIServingRerankUnit.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestOpenAIServingRerankUnit.test_convert_to_internal_request_cross_encoder_pairs`: This test exercises `test_convert_to_internal_request_cross_encoder_pairs` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_convert_to_internal_request_cross_encoder_pairs`。
- `TestOpenAIServingRerankUnit.test_convert_to_internal_request_qwen3_template_returns_request`: This test exercises `test_convert_to_internal_request_qwen3_template_returns_request` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_convert_to_internal_request_qwen3_template_returns_request`。
- `TestOpenAIServingRerankUnit.test_build_rerank_response_embedding_list_uses_first_scalar`: This test exercises `test_build_rerank_response_embedding_list_uses_first_scalar` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_build_rerank_response_embedding_list_uses_first_scalar`。
- `TestOpenAIServingRerankUnit.test_build_rerank_response_float_list`: This test exercises `test_build_rerank_response_float_list` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_build_rerank_response_float_list`。
- `TestOpenAIServingRerankUnit.test_helper_is_qwen3_reranker_template`: This test exercises `test_helper_is_qwen3_reranker_template` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_helper_is_qwen3_reranker_template`。
- `TestOpenAIServingRerankUnit.test_helper_qwen3_rerank_score`: This test exercises `test_helper_qwen3_rerank_score` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_helper_qwen3_rerank_score`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `unittest`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.managers.tokenizer_manager_score_mixin`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 310
