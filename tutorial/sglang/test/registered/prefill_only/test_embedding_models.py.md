# test_embedding_models.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/prefill_only/test_embedding_models.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates embedding models behavior in SGLang's prefill only area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 prefill only 领域中与 embedding models 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: supporting source context / 辅助源码上下文
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 15-30: module imports and dependencies / 模块导入与依赖
```python
import multiprocessing as mp
import random
import unittest
from typing import Optional

import torch
from transformers import AutoConfig, AutoTokenizer

from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.runners import DEFAULT_PROMPTS, HFRunner, SRTRunner
from sglang.test.test_utils import (
    CustomTestCase,
    get_similarities,
    is_in_amd_ci,
    is_in_ci,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `multiprocessing`, `random`, `unittest`, `typing`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `multiprocessing`, `random`, `unittest`, `typing`。

### Lines 31-32: supporting source context / 辅助源码上下文
```python

# Embedding model tests
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 33-56: CI registration and metadata / CI 注册与元数据
```python
register_amd_ci(
    est_time=73,
    suite="stage-b-test-1-gpu-small-amd",
    disabled="see https://github.com/sgl-project/sglang/issues/11127",
)
register_cuda_ci(est_time=136, stage="base-b", runner_config="1-gpu-small")

MODEL_TO_CONFIG = {
    "Alibaba-NLP/gte-Qwen2-1.5B-instruct": (1, 1e-5),
    "intfloat/e5-mistral-7b-instruct": (1, 1e-5),
    "Qwen/Qwen3-Embedding-8B": (1, 1e-5),
    # Temporarily disable: HF reference path in runners.py runs this Qwen2-VL
    # fine-tune with bidirectional attention (the non-sentence-transformers
    # branch in _get_sentence_transformer_embedding_model does not pass
    # is_causal=True), while SGLang's Qwen2-VL embedding is always causal —
    # producing ~0.30 cosine diffs vs HF on short prompts.
    # See https://github.com/sgl-project/sglang/actions/runs/25224929325/job/73966043206
    # "marco/mcdse-2b-v1": (1, 1e-5),
    # Temporarily disable before this model is fixed
    # "jason9693/Qwen2.5-1.5B-apeach": (1, 1e-5),
}
MODELS = [(key, *MODEL_TO_CONFIG[key]) for key in MODEL_TO_CONFIG]

TORCH_DTYPES = [torch.float16]
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_amd_ci, register_cuda_ci.
**CN:** 该代码块通过 register_amd_ci, register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 59-60: class TestEmbeddingModels declaration / 类 TestEmbeddingModels 声明
```python
class TestEmbeddingModels(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 61-63: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        mp.set_start_method("spawn", force=True)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 65-81: method truncate prompts / 方法 truncate prompts
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
**EN:** This block implements `_truncate_prompts` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_truncate_prompts`，承担模块行为中的一个聚焦逻辑片段。

### Lines 83-127: helper routine assert close prefill logits / 辅助流程 assert close prefill logits
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

        attention_backend = "triton" if is_in_amd_ci() else None
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
**EN:** This helper encapsulates `assert_close_prefill_logits` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `assert_close_prefill_logits`，以便周围测试复用准备、执行或校验逻辑。

### Lines 129-139: test case prefill logits / 测试用例 prefill logits
```python
    def test_prefill_logits(self):
        models_to_test = MODELS

        if is_in_ci():
            models_to_test = [random.choice(MODELS)]

        for model, tp_size, prefill_tolerance in models_to_test:
            for torch_dtype in TORCH_DTYPES:
                self.assert_close_prefill_logits(
                    DEFAULT_PROMPTS, model, tp_size, torch_dtype, prefill_tolerance
                )
```
**EN:** This test exercises `test_prefill_logits` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prefill_logits`。

### Lines 141-158: test case matryoshka embedding / 测试用例 matryoshka embedding
```python
    def test_matryoshka_embedding(self):
        models_to_test = [
            (
                "Alibaba-NLP/gte-Qwen2-1.5B-instruct",
                *MODEL_TO_CONFIG["Alibaba-NLP/gte-Qwen2-1.5B-instruct"],
            )
        ]

        for model, tp_size, prefill_tolerance in models_to_test:
            for torch_dtype in TORCH_DTYPES:
                self.assert_close_prefill_logits(
                    DEFAULT_PROMPTS,
                    model,
                    tp_size,
                    torch_dtype,
                    prefill_tolerance,
                    matryoshka_dim=128,
                )
```
**EN:** This test exercises `test_matryoshka_embedding` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_matryoshka_embedding`。

### Lines 161-162: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestEmbeddingModels`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestEmbeddingModels.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestEmbeddingModels._truncate_prompts`: This block implements `_truncate_prompts` and captures one focused piece of the module's behavior. / 该代码块实现 `_truncate_prompts`，承担模块行为中的一个聚焦逻辑片段。
- `TestEmbeddingModels.assert_close_prefill_logits`: This helper encapsulates `assert_close_prefill_logits` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `assert_close_prefill_logits`，以便周围测试复用准备、执行或校验逻辑。
- `TestEmbeddingModels.test_prefill_logits`: This test exercises `test_prefill_logits` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prefill_logits`。
- `TestEmbeddingModels.test_matryoshka_embedding`: This test exercises `test_matryoshka_embedding` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_matryoshka_embedding`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `multiprocessing`, `random`, `unittest`, `typing`
- **Third-party modules / 第三方模块**: `torch`, `transformers`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.runners`, `sglang.test.test_utils`

- **Total lines / 总行数**: 162
