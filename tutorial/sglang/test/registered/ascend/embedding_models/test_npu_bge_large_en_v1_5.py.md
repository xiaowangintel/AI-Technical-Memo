# test_npu_bge_large_en_v1_5.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/embedding_models/test_npu_bge_large_en_v1_5.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on embedding models npu bge large en v1 5 in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 embedding models npu bge large en v1 5 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Import dependencies
```python
import multiprocessing as mp
import unittest
from typing import Optional

import torch
from transformers import AutoConfig, AutoTokenizer

from sglang.test.ci.ci_register import register_npu_ci
from sglang.test.runners import HFRunner, SRTRunner
from sglang.test.test_utils import CustomTestCase, get_similarities
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 12-17: Register CI metadata
```python
register_npu_ci(
    est_time=400,
    suite="full-1-npu-a3",
    nightly=True,
    disabled="embeddings are not all close",
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 19-28: Define module constants
```python
DEFAULT_PROMPTS = [
    "The capital of the United Kingdom is",
    "Today is a sunny day and I like",
    "AI is a field of computer science focused on",
]

MODELS = [
    ("/root/.cache/modelscope/hub/models/bge-large-en-v1.5", 1, 1e-5),
]
TORCH_DTYPES = [torch.float16]
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 31-31: Define class TestEmbeddingModels
```python
class TestEmbeddingModels(CustomTestCase):
```
**EN:** This declaration introduces the `TestEmbeddingModels` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestEmbeddingModels` 测试类，并说明它通过继承承担的职责。

### Lines 33-35: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        mp.set_start_method("spawn", force=True)
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 37-53: Define helper: truncate prompts
```python
    def _truncate_prompts(self, prompts, model_path):
        config = AutoConfig.from_pretrained(model_path)
        max_length = getattr(config, "max_position_embeddings", 2048)

        tokenizer = AutoTokenizer.from_pretrained(model_path)

        truncated_prompts = []
        for prompt in prompts:
            tokens = tokenizer(prompt, return_tensors="pt", truncation=False)
            if len(tokens.input_ids[0]) > max_length:
                truncated_text = tokenizer.decode(
                    tokens.input_ids[0][: max_length - 1], skip_special_tokens=True
                )
                truncated_prompts.append(truncated_text)
            else:
                truncated_prompts.append(prompt)
        return truncated_prompts
```
**EN:** This helper function encapsulates reusable logic inside `TestEmbeddingModels` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestEmbeddingModels` 内部调用，从而让场景结构更清晰。

### Lines 55-99: Define helper: assert close prefill logits
```python
    def assert_close_prefill_logits(
        self,
        prompts,
        model_path,
        tp_size,
        torch_dtype,
        prefill_tolerance,
        matryoshka_dim: Optional[int] = None,
    ) -> None:
        truncated_prompts = self._truncate_prompts(prompts, model_path)

        with HFRunner(
            model_path,
            torch_dtype=torch_dtype,
            model_type="embedding",
            matryoshka_dim=matryoshka_dim,
        ) as hf_runner:
            hf_outputs = hf_runner.forward(truncated_prompts)

        attention_backend = "ascend"
        with SRTRunner(
            model_path,
            tp_size=tp_size,
            torch_dtype=torch_dtype,
            model_type="embedding",
            attention_backend=attention_backend,
            json_model_override_args=(
                {"matryoshka_dimensions": [matryoshka_dim]} if matryoshka_dim else None
            ),
        ) as srt_runner:
            srt_outputs = srt_runner.forward(
                truncated_prompts, dimensions=matryoshka_dim
            )

        for i in range(len(prompts)):
            hf_logits = torch.Tensor(hf_outputs.embed_logits[i])
            srt_logits = torch.Tensor(srt_outputs.embed_logits[i])

            similarity = torch.tensor(get_similarities(hf_logits, srt_logits))
            print("similarity diff", abs(similarity - 1))

            if len(prompts[i]) <= 1000:
                assert torch.all(
                    abs(similarity - 1) < prefill_tolerance
                ), "embeddings are not all close"
```
**EN:** This helper function encapsulates reusable logic inside `TestEmbeddingModels` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestEmbeddingModels` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 101-108: Run test: prefill logits
```python
    def test_prefill_logits(self):
        models_to_test = MODELS

        for model, tp_size, prefill_tolerance in models_to_test:
            for torch_dtype in TORCH_DTYPES:
                self.assert_close_prefill_logits(
                    DEFAULT_PROMPTS, model, tp_size, torch_dtype, prefill_tolerance
                )
```
**EN:** This test method exercises prefill logits and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 prefill logits 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 111-112: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.test.ci.ci_register`, `sglang.test.runners`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `multiprocessing`, `torch`, `transformers`, `typing`, `unittest`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
