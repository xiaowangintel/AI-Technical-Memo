# test_encoder_embedding_models.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/prefill_only/test_encoder_embedding_models.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `encoder embedding models` scenario in `test/manual/prefill_only`. It uses SGLang's shared test infrastructure to configure models or services and verify expected functional behavior. / 该手动测试模块覆盖 `test/manual/prefill_only` 中的 `encoder embedding models` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-38: Constants and scenario settings / 常量与场景配置
```python
import multiprocessing as mp
import random
import time
import unittest

import torch
from transformers import AutoConfig, AutoTokenizer

from sglang.test.runners import DEFAULT_PROMPTS, HFRunner, SRTRunner
from sglang.test.test_utils import CustomTestCase, get_similarities, is_in_ci

# Encoder embedding model tests (CUDA only)

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

# python -m unittest test_encoder_embedding_models.TestEncoderEmbeddingModels.test_prefill_logits


MODELS = [("BAAI/bge-small-en", 1, 1e-5), ("BAAI/bge-m3", 1, 1e-5)]

ATTENTION_BACKEND = ["torch_native", "triton", "flashinfer"]
BATCH_SIZE = [1, 2]
TORCH_DTYPES = [torch.float32, torch.float16]
sgl_to_st_ratio = []
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. Representative call sites include `tests`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 39-41: Class definition for TestEncoderEmbeddingModels / 类定义
```python
class TestEncoderEmbeddingModels(CustomTestCase):

    @classmethod
```
**EN:** This range declares `TestEncoderEmbeddingModels`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 42-43: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        mp.set_start_method("spawn", force=True)
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `set_start_method`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 45-62: Helper routines around _truncate_prompts / 辅助例程
```python
    def _truncate_prompts(self, prompts, model_path):
        config = AutoConfig.from_pretrained(model_path)
        max_length = getattr(config, "max_position_embeddings", 512) - 20

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
**EN:** This range implements helper routine(s) `_truncate_prompts` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `from_pretrained`, `getattr`, `tokenizer` and `decode`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 64-88: Helper routines around assert_close_prefill_logits / 辅助例程
```python
    def assert_close_prefill_logits(
        self,
        prompts,
        model_path,
        tp_size,
        torch_dtype,
        prefill_tolerance,
        attention_backend,
        batch_size,
    ) -> None:
        truncated_prompts = self._truncate_prompts(prompts, model_path)
        truncated_prompts = truncated_prompts * batch_size

        with HFRunner(
            model_path,
            torch_dtype=torch_dtype,
            model_type="embedding",
        ) as hf_runner:
            # warm up
            hf_outputs = hf_runner.forward(truncated_prompts)

            st_start_time = time.perf_counter()
            hf_outputs = hf_runner.forward(truncated_prompts)
            st_end_time = time.perf_counter()
```
**EN:** This range implements helper routine(s) `assert_close_prefill_logits` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `_truncate_prompts`, `HFRunner`, `forward` and `perf_counter`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 89-113: Scenario logic / 场景逻辑
```python
        with SRTRunner(
            model_path,
            tp_size=tp_size,
            torch_dtype=torch_dtype,
            model_type="embedding",
            attention_backend=attention_backend,
            chunked_prefill_size=-1,
            disable_radix_cache=True,
        ) as srt_runner:
            # warm up
            srt_outputs = srt_runner.forward(truncated_prompts)

            sgl_start_time = time.perf_counter()
            srt_outputs = srt_runner.forward(truncated_prompts)
            sgl_end_time = time.perf_counter()

        transformer_time = st_end_time - st_start_time
        sgl_time = sgl_end_time - sgl_start_time
        sgl_to_st_ratio.append(sgl_time / transformer_time)

        for i in range(len(truncated_prompts)):
            hf_logits = torch.Tensor(hf_outputs.embed_logits[i])
            srt_logits = torch.Tensor(srt_outputs.embed_logits[i])

            similarity = torch.tensor(get_similarities(hf_logits, srt_logits))
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `SRTRunner`, `forward`, `perf_counter` and `append`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 114-120: Assertions and result checks / 断言与结果检查
```python
            # If something is wrong, uncomment this to observe similarity.
            # print("similarity diff", abs(similarity - 1))

            if len(truncated_prompts[i]) <= 1000:
                assert torch.all(
                    abs(similarity - 1) < prefill_tolerance
                ), "embeddings are not all close"
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `abs`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 122-138: Test routines around test_prefill_logits / 测试例程
```python
    def test_prefill_logits(self):
        models_to_test = MODELS

        if is_in_ci():
            models_to_test = [random.choice(MODELS)]

        for model, tp_size, prefill_tolerance in models_to_test:
            for attention_backend in ATTENTION_BACKEND:
                for batch_size in BATCH_SIZE:
                    for torch_dtype in TORCH_DTYPES:
                        # NOTE: FlashInfer currently has limitations with head_dim = 32 or
                        # other dimensions.
                        # The FlashInfer head_dim limitation itself is tracked here:
                        # https://github.com/flashinfer-ai/flashinfer/issues/1048
                        #
                        # Flashinfer does not support torch.float32 for dtype_q, so skip it
                        if attention_backend == "flashinfer":
```
**EN:** This range defines concrete test routine(s) `test_prefill_logits`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `is_in_ci` and `choice`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 139-146: Assertions and result checks / 断言与结果检查
```python
                            if (
                                model == "BAAI/bge-small-en"
                                or torch_dtype == torch.float32
                            ):
                                continue

                        self.assert_close_prefill_logits(
                            DEFAULT_PROMPTS,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assert_close_prefill_logits`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 147-161: Scenario logic / 场景逻辑
```python
                            model,
                            tp_size,
                            torch_dtype,
                            prefill_tolerance,
                            attention_backend,
                            batch_size,
                        )

        for i in range(len(BATCH_SIZE)):
            print(
                "bacth size: ",
                BATCH_SIZE[i] * 5,
                "sgl_time/st_time",
                round(sgl_to_st_ratio[i], 3),
            )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `round`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 162-165: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Multi-GPU orchestration / 多 GPU 编排
- HTTP/API interaction / HTTP/API 交互
- Token-level inspection / Token 级分析
- Disaggregated serving / 解耦式服务
- Prefix caching / 前缀缓存

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `multiprocessing`, `random`, `time`, `unittest`
- **Third-party / 第三方库**: `torch`, `transformers`
- **Project Modules / 项目模块**: `sglang.test.runners`, `sglang.test.test_utils`
