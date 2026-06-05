# test_gme_qwen_models.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/models/test_gme_qwen_models.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `gme qwen models` scenario in `test/manual/models`. It uses SGLang's shared test infrastructure to configure models or services and verify expected accuracy and functional behavior. / 该手动测试模块覆盖 `test/manual/models` 中的 `gme qwen models` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的精度与功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: Constants and scenario settings / 常量与场景配置
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


import multiprocessing as mp
import unittest

import torch

from sglang.test.runners import HFRunner, SRTRunner
from sglang.test.test_utils import CustomTestCase, get_similarities

TEXTS = "two Subway Series sandwiches with meats, cheese, lettuce, tomatoes, and onions on a black background, accompanied by the Subway Series logo, highlighting a new sandwich series."
IMAGES = "https://huggingface.co/datasets/liuhaotian/llava-bench-in-the-wild/resolve/main/images/023.jpg"


MODELS = [
    ("Alibaba-NLP/gme-Qwen2-VL-2B-Instruct", 1e-3),
]
TORCH_DTYPES = [torch.float16]
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。

### Lines 34-35: Class definition for TestQmeQwenModels / 类定义
```python
class TestQmeQwenModels(CustomTestCase):
    @classmethod
```
**EN:** This range declares `TestQmeQwenModels`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 36-37: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        mp.set_start_method("spawn", force=True)
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `set_start_method`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 39-58: Helper routines around assert_close_embeddings / 辅助例程
```python
    def assert_close_embeddings(self, model, prefill_tolerance, torch_dtype):

        prompts_no_image = f"<|im_start|>system\nYou are a helpful assistant<|im_end|>\n<|im_start|>user\n{TEXTS}<|im_end|>\n<|im_start|>assistant\n<|endoftext|>"
        prompts_with_image = f"<|im_start|>system\nYou are a helpful assistant<|im_end|>\n<|im_start|>user\n<|vision_start|><|image_pad|><|vision_end|><|im_end|>\n<|im_start|>assistant\n<|endoftext|>"
        with HFRunner(
            model,
            torch_dtype=torch_dtype,
            model_type="embedding",
        ) as hf_runner:
            hf_text_embeddings = hf_runner.forward(prompts=[prompts_no_image])
            hf_image_embeddings = hf_runner.forward(
                prompts=[prompts_with_image], image_data=[IMAGES]
            )
        with SRTRunner(
            model,
            tp_size=1,
            torch_dtype=torch_dtype,
            model_type="embedding",
        ) as srt_runner:
            srt_text_embeddings = srt_runner.forward(prompts=prompts_no_image)
```
**EN:** This range implements helper routine(s) `assert_close_embeddings` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `HFRunner`, `forward` and `SRTRunner`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 59-63: Scenario logic / 场景逻辑
```python
            srt_image_embeddings = srt_runner.forward(
                prompts=prompts_with_image, image_data=IMAGES
            )

        similarity = get_similarities(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `forward` and `get_similarities`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 64-76: Assertions and result checks / 断言与结果检查
```python
            hf_text_embeddings.embed_logits[0], srt_text_embeddings.embed_logits[0]
        )
        print("texts similarity diff", abs(similarity - 1))
        assert torch.all(
            abs(similarity - 1) < prefill_tolerance
        ), "embeddings are not all close"
        similarity = get_similarities(
            hf_image_embeddings.embed_logits[0], srt_image_embeddings.embed_logits[0]
        )
        print("images similarity diff", abs(similarity - 1))
        assert torch.all(
            abs(similarity - 1) < prefill_tolerance
        ), "embeddings are not all close"
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `abs` and `get_similarities`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 78-81: Test routines around test_accuracy / 测试例程
```python
    def test_accuracy(self):
        for model, prefill_tolerance in MODELS:
            for torch_dtype in TORCH_DTYPES:
                self.assert_close_embeddings(model, prefill_tolerance, torch_dtype)
```
**EN:** This range defines concrete test routine(s) `test_accuracy`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `assert_close_embeddings`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 82-85: Script entry point / 脚本入口
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
- HTTP/API interaction / HTTP/API 交互
- Multimodal inputs / 多模态输入
- Disaggregated serving / 解耦式服务

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `multiprocessing`, `unittest`
- **Third-party / 第三方库**: `torch`
- **Project Modules / 项目模块**: `sglang.test.runners`, `sglang.test.test_utils`
