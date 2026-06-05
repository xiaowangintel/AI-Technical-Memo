# test_cross_encoder_models.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/prefill_only/test_cross_encoder_models.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `cross encoder models` scenario in `test/manual/prefill_only`. It uses SGLang's shared test infrastructure to configure models or services and verify expected accuracy and functional behavior. / 该手动测试模块覆盖 `test/manual/prefill_only` 中的 `cross encoder models` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的精度与功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Imports and shared helpers / 导入与共享辅助项
```python
import multiprocessing as mp
import random
import unittest

import torch

from sglang.test.runners import TEST_RERANK_QUERY_DOCS, HFRunner, SRTRunner
from sglang.test.test_utils import CustomTestCase, is_in_ci

# Cross encoder model tests


MODELS = [
    ("cross-encoder/ms-marco-MiniLM-L6-v2", 1, 1e-2),
    ("BAAI/bge-reranker-v2-m3", 1, 1e-2),
]
ATTENTION_BACKEND = ["torch_native", "triton"]

TORCH_DTYPES = [torch.float32]
```
**EN:** This range imports `multiprocessing`, `random`, `unittest` and `torch`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。

### Lines 22-24: Class definition for TestCrossEncoderModels / 类定义
```python
class TestCrossEncoderModels(CustomTestCase):

    @classmethod
```
**EN:** This range declares `TestCrossEncoderModels`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 25-26: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        mp.set_start_method("spawn", force=True)
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `set_start_method`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 28-46: Helper routines around assert_close_prefill_logits / 辅助例程
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
```
**EN:** This range implements helper routine(s) `assert_close_prefill_logits` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `HFRunner`, `forward` and `SRTRunner`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 47-52: Scenario logic / 场景逻辑
```python
            torch_dtype=torch_dtype,
            model_type="cross_encoder",
            attention_backend=attention_backend,
            chunked_prefill_size=-1,
            disable_radix_cache=True,
        ) as srt_runner:
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 53-60: Assertions and result checks / 断言与结果检查
```python
            srt_scores = srt_runner.forward(prompts).scores

        for i in range(len(srt_scores)):
            score_difference = abs(hf_scores[i] - srt_scores[i])

            assert (
                score_difference < score_tolerance
            ), "cross encoder scores are not all close"
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `forward`, `abs` and `assert`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 62-69: Helper routines around preprocess_prompts / 辅助例程
```python
    def preprocess_prompts(self, prompt):
        processed_prompts = []
        query = prompt["query"]
        documents = prompt["documents"]
        for document in documents:
            processed_prompts.append([query, document])

        return processed_prompts
```
**EN:** This range implements helper routine(s) `preprocess_prompts` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `append`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 71-71: Test routines around test_prefill_logits / 测试例程
```python
    def test_prefill_logits(self):
```
**EN:** This range defines concrete test routine(s) `test_prefill_logits`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 72-89: Assertions and result checks / 断言与结果检查
```python
        models_to_test = MODELS

        if is_in_ci():
            models_to_test = [random.choice(MODELS)]

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
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `is_in_ci`, `choice`, `preprocess_prompts` and `assert_close_prefill_logits`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 90-93: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Accuracy evaluation / 精度评测
- Multi-GPU orchestration / 多 GPU 编排
- Disaggregated serving / 解耦式服务
- Prefix caching / 前缀缓存

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `multiprocessing`, `random`, `unittest`
- **Third-party / 第三方库**: `torch`
- **Project Modules / 项目模块**: `sglang.test.runners`, `sglang.test.test_utils`
