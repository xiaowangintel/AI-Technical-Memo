# test_npu_bge_reranker_v2_m3.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/rerank_models/test_npu_bge_reranker_v2_m3.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on rerank models npu bge reranker v2 m3 in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 rerank models npu bge reranker v2 m3 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Import dependencies
```python
import multiprocessing as mp
import unittest

import torch

from sglang.test.ascend.test_ascend_utils import BGE_RERANKER_V2_M3_WEIGHTS_PATH
from sglang.test.ci.ci_register import register_npu_ci
from sglang.test.runners import TEST_RERANK_QUERY_DOCS, HFRunner, SRTRunner
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 11-15: Register CI metadata
```python
register_npu_ci(
    est_time=400,
    suite="nightly-1-npu-a3",
    nightly=True,
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 17-21: Define module constants
```python
MODELS = [
    (BGE_RERANKER_V2_M3_WEIGHTS_PATH, 1, 1e-2),
]
ATTENTION_BACKEND = ["ascend"]
TORCH_DTYPES = [torch.bfloat16]
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 24-24: Define class TestBgeReranker
```python
class TestBgeReranker(CustomTestCase):
```
**EN:** This declaration introduces the `TestBgeReranker` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestBgeReranker` 测试类，并说明它通过继承承担的职责。

### Lines 25-30: Document the class `TestBgeReranker`
```python
    """Testcase: This test case validates that the cross-encoder scores from the BAAI/bge-reranker-v2-m3 model in the
    SGLang framework are less than 1e-2 different from the Hugging Face implementation.

    [Test Category] Model
    [Test Target] BAAI/bge-reranker-v2-m3
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestBgeReranker`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestBgeReranker`的设计意图。

### Lines 32-34: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        mp.set_start_method("spawn", force=True)
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 36-68: Define helper: assert close prefill logits
```python
    def assert_close_prefill_logits(
        self,
        prompts,
        model_path,
        tp_size,
        torch_dtype,
        score_tolerance,
        attention_backend,
    ) -> None:
        with HFRunner(
            model_path,
            torch_dtype=torch_dtype,
            model_type="cross_encoder",
        ) as hf_runner:
            hf_scores = hf_runner.forward(prompts).scores

        with SRTRunner(
            model_path,
            tp_size=tp_size,
            torch_dtype=torch_dtype,
            model_type="cross_encoder",
            attention_backend=attention_backend,
            chunked_prefill_size=-1,
            disable_radix_cache=True,
        ) as srt_runner:
            srt_scores = srt_runner.forward(prompts).scores

        for i in range(len(srt_scores)):
            score_difference = abs(hf_scores[i] - srt_scores[i])

            assert (
                score_difference < score_tolerance
            ), "cross encoder scores are not all close"
```
**EN:** This helper function encapsulates reusable logic inside `TestBgeReranker` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestBgeReranker` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 70-77: Define helper: preprocess prompts
```python
    def preprocess_prompts(self, prompt):
        processed_prompts = []
        query = prompt["query"]
        documents = prompt["documents"]
        for document in documents:
            processed_prompts.append([query, document])

        return processed_prompts
```
**EN:** This helper function encapsulates reusable logic inside `TestBgeReranker` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestBgeReranker` 内部调用，从而让场景结构更清晰。

### Lines 79-94: Run test: prefill logits
```python
    def test_prefill_logits(self):
        models_to_test = MODELS

        for model, tp_size, prefill_tolerance in models_to_test:
            for attention_backend in ATTENTION_BACKEND:
                for queryDocs in TEST_RERANK_QUERY_DOCS:
                    prompts = self.preprocess_prompts(queryDocs)
                    for torch_dtype in TORCH_DTYPES:
                        self.assert_close_prefill_logits(
                            prompts,
                            model,
                            tp_size,
                            torch_dtype,
                            prefill_tolerance,
                            attention_backend,
                        )
```
**EN:** This test method exercises prefill logits and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 prefill logits 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 97-98: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。
- Shared base classes centralize common setup, assertions, and diagnostics. / 共享基类集中封装了通用初始化、断言与诊断逻辑。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.test.ascend.test_ascend_utils`, `sglang.test.ci.ci_register`, `sglang.test.runners`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `multiprocessing`, `torch`, `unittest`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
