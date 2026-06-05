# test_vision_openai_server_a.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/vlm/test_vision_openai_server_a.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates vision openai server a behavior in SGLang's vlm area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 vlm 领域中与 vision openai server a 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: supporting statements / 辅助语句
```python
"""
Usage:
python3 -m unittest test_vision_openai_server.TestOpenAIVisionServer.test_mixed_batch
python3 -m unittest test_vision_openai_server.TestOpenAIVisionServer.test_multi_images_chat_completion
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 7-20: module imports and dependencies / 模块导入与依赖
```python
import unittest

import openai

from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.vlm_utils import *
from sglang.test.vlm_utils import (
    AudioOpenAITestMixin,
    CustomTestCase,
    ImageOpenAITestMixin,
    OmniOpenAITestMixin,
    TestOpenAIMLLMServerBase,
    VideoOpenAITestMixin,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `openai`, `sglang.test.ci.ci_register`, `sglang.test.vlm_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `openai`, `sglang.test.ci.ci_register`, `sglang.test.vlm_utils`。

### Lines 22-22: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=780, stage="base-b", runner_config="1-gpu-large")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 25-25: class TestLlavaServer declaration / 类 TestLlavaServer 声明
```python
class TestLlavaServer(ImageOpenAITestMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `ImageOpenAITestMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `ImageOpenAITestMixin`。

### Lines 26-26: class-level constants and configuration for `TestLlavaServer` / 类级常量与配置
```python
    model = "lmms-lab/llava-onevision-qwen2-0.5b-ov"
```
**EN:** This block defines shared names such as `model`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 29-29: class TestLfm2VlServer declaration / 类 TestLfm2VlServer 声明
```python
class TestLfm2VlServer(ImageOpenAITestMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `ImageOpenAITestMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `ImageOpenAITestMixin`。

### Lines 30-30: class-level constants and configuration for `TestLfm2VlServer` / 类级常量与配置
```python
    model = "LiquidAI/LFM2.5-VL-1.6B"
```
**EN:** This block defines shared names such as `model`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 33-33: class TestQwen25VLServer declaration / 类 TestQwen25VLServer 声明
```python
class TestQwen25VLServer(ImageOpenAITestMixin, VideoOpenAITestMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `ImageOpenAITestMixin`, `VideoOpenAITestMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `ImageOpenAITestMixin`, `VideoOpenAITestMixin`。

### Lines 34-37: class-level constants and configuration for `TestQwen25VLServer` / 类级常量与配置
```python
    model = "Qwen/Qwen2.5-VL-7B-Instruct"
    extra_args = [
        "--cuda-graph-max-bs=4",
    ]
```
**EN:** This block defines shared names such as `model`, `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 40-40: class TestQwen3VLServer declaration / 类 TestQwen3VLServer 声明
```python
class TestQwen3VLServer(ImageOpenAITestMixin, VideoOpenAITestMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `ImageOpenAITestMixin`, `VideoOpenAITestMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `ImageOpenAITestMixin`, `VideoOpenAITestMixin`。

### Lines 41-42: class-level constants and configuration for `TestQwen3VLServer` / 类级常量与配置
```python
    model = "Qwen/Qwen3-VL-30B-A3B-Instruct"
    extra_args = ["--cuda-graph-max-bs=4"]
```
**EN:** This block defines shared names such as `model`, `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 45-45: class TestQwen3OmniServer declaration / 类 TestQwen3OmniServer 声明
```python
class TestQwen3OmniServer(OmniOpenAITestMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `OmniOpenAITestMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `OmniOpenAITestMixin`。

### Lines 46-52: class-level constants and configuration for `TestQwen3OmniServer` / 类级常量与配置
```python
    model = "Qwen/Qwen3-Omni-30B-A3B-Instruct"
    extra_args = [  # workaround to fit into H100
        "--mem-fraction-static=0.90",
        "--disable-cuda-graph",
        "--disable-fast-image-processor",
        "--grammar-backend=none",
    ]
```
**EN:** This block defines shared names such as `model`, `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 55-55: class TestQwen2VLContextLengthServer declaration / 类 TestQwen2VLContextLengthServer 声明
```python
class TestQwen2VLContextLengthServer(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 56-73: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = "Qwen/Qwen2-VL-7B-Instruct"
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            api_key=cls.api_key,
            other_args=[
                "--context-length",
                "300",
                "--cuda-graph-max-bs",
                "4",
            ],
        )
        cls.base_url += "/v1"
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 75-77: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 79-108: test case single image chat completion / 测试用例 single image chat completion
```python
    def test_single_image_chat_completion(self):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        with self.assertRaises(openai.BadRequestError) as cm:
            client.chat.completions.create(
                model="default",
                messages=[
                    {
                        "role": "user",
                        "content": [
                            {
                                "type": "image_url",
                                "image_url": {"url": IMAGE_MAN_IRONING_URL},
                            },
                            {
                                "type": "text",
                                "text": "Give a lengthy description of this picture",
                            },
                        ],
                    },
                ],
                temperature=0,
            )

        # context length is checked first, then max_req_input_len, which is calculated from the former
        assert (
            "Multimodal prompt is too long after expanding multimodal tokens."
            in str(cm.exception)
            or "is longer than the model's context length" in str(cm.exception)
        )
```
**EN:** This test exercises `test_single_image_chat_completion` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_image_chat_completion`。

### Lines 109-115: supporting source context / 辅助源码上下文
```python


# flaky
# class TestMllamaServer(ImageOpenAITestMixin):
#     model = "meta-llama/Llama-3.2-11B-Vision-Instruct"


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 116-116: class TestInternVL25Server declaration / 类 TestInternVL25Server 声明
```python
class TestInternVL25Server(ImageOpenAITestMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `ImageOpenAITestMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `ImageOpenAITestMixin`。

### Lines 117-120: class-level constants and configuration for `TestInternVL25Server` / 类级常量与配置
```python
    model = "OpenGVLab/InternVL2_5-2B"
    extra_args = [
        "--cuda-graph-max-bs=4",
    ]
```
**EN:** This block defines shared names such as `model`, `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 123-123: class TestMiniCPMV4Server declaration / 类 TestMiniCPMV4Server 声明
```python
class TestMiniCPMV4Server(ImageOpenAITestMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `ImageOpenAITestMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `ImageOpenAITestMixin`。

### Lines 124-127: class-level constants and configuration for `TestMiniCPMV4Server` / 类级常量与配置
```python
    model = "openbmb/MiniCPM-V-4"
    extra_args = [
        "--cuda-graph-max-bs=4",
    ]
```
**EN:** This block defines shared names such as `model`, `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 130-130: class TestMiniCPMo26Server declaration / 类 TestMiniCPMo26Server 声明
```python
class TestMiniCPMo26Server(ImageOpenAITestMixin, AudioOpenAITestMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `ImageOpenAITestMixin`, `AudioOpenAITestMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `ImageOpenAITestMixin`, `AudioOpenAITestMixin`。

### Lines 131-134: class-level constants and configuration for `TestMiniCPMo26Server` / 类级常量与配置
```python
    model = "openbmb/MiniCPM-o-2_6"
    extra_args = [
        "--cuda-graph-max-bs=4",
    ]
```
**EN:** This block defines shared names such as `model`, `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 137-137: class TestGemma3itServer declaration / 类 TestGemma3itServer 声明
```python
class TestGemma3itServer(ImageOpenAITestMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `ImageOpenAITestMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `ImageOpenAITestMixin`。

### Lines 138-141: class-level constants and configuration for `TestGemma3itServer` / 类级常量与配置
```python
    model = "google/gemma-3-4b-it"
    extra_args = [
        "--cuda-graph-max-bs=4",
    ]
```
**EN:** This block defines shared names such as `model`, `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 144-144: class TestKimiVLServer declaration / 类 TestKimiVLServer 声明
```python
class TestKimiVLServer(ImageOpenAITestMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `ImageOpenAITestMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `ImageOpenAITestMixin`。

### Lines 145-149: class-level constants and configuration for `TestKimiVLServer` / 类级常量与配置
```python
    model = "moonshotai/Kimi-VL-A3B-Instruct"
    extra_args = [
        "--context-length=8192",
        "--dtype=bfloat16",
    ]
```
**EN:** This block defines shared names such as `model`, `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 151-153: test case video images chat completion / 测试用例 video images chat completion
```python
    def test_video_images_chat_completion(self):
        # model context length exceeded
        pass
```
**EN:** This test exercises `test_video_images_chat_completion` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_video_images_chat_completion`。

### Lines 156-159: class TestGLM41VServer declaration / 类 TestGLM41VServer 声明
```python
@unittest.skip(
    "Disabling this test to speed up CI. Prefer to test it within nightly test."
)
class TestGLM41VServer(ImageOpenAITestMixin, VideoOpenAITestMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `ImageOpenAITestMixin`, `VideoOpenAITestMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `ImageOpenAITestMixin`, `VideoOpenAITestMixin`。

### Lines 160-163: class-level constants and configuration for `TestGLM41VServer` / 类级常量与配置
```python
    model = "zai-org/GLM-4.1V-9B-Thinking"
    extra_args = [
        "--reasoning-parser=glm45",
    ]
```
**EN:** This block defines shared names such as `model`, `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 166-166: class TestQwen2AudioServer declaration / 类 TestQwen2AudioServer 声明
```python
class TestQwen2AudioServer(AudioOpenAITestMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `AudioOpenAITestMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `AudioOpenAITestMixin`。

### Lines 167-167: class-level constants and configuration for `TestQwen2AudioServer` / 类级常量与配置
```python
    model = "Qwen/Qwen2-Audio-7B-Instruct"
```
**EN:** This block defines shared names such as `model`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 170-170: class TestDeepseekOCRServer declaration / 类 TestDeepseekOCRServer 声明
```python
class TestDeepseekOCRServer(TestOpenAIMLLMServerBase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestOpenAIMLLMServerBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestOpenAIMLLMServerBase`。

### Lines 171-176: class-level constants and configuration for `TestDeepseekOCRServer` / 类级常量与配置
```python
    model = "deepseek-ai/DeepSeek-OCR"
    trust_remote_code = False
    extra_args = [
        "--mem-fraction-static=0.70",
        "--cuda-graph-max-bs=4",
    ]
```
**EN:** This block defines shared names such as `model`, `trust_remote_code`, `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `trust_remote_code`, `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 178-200: method verify single image response for ocr / 方法 verify single image response for ocr
```python
    def verify_single_image_response_for_ocr(self, response):
        """Verify DeepSeek-OCR grounding output with coordinates"""
        assert response.choices[0].message.role == "assistant"
        text = response.choices[0].message.content
        assert isinstance(text, str)

        # DeepSeek-OCR uses grounding format, outputs coordinates
        assert "text" in text.lower(), f"OCR text: {text}, should contain 'text'"

        # Verify coordinate format [[x1, y1, x2, y2]]
        import re

        coord_pattern = r"\[\[[\d\s,]+\]\]"
        assert re.search(
            coord_pattern, text
        ), f"OCR text: {text}, should contain coordinate format [[x1, y1, x2, y2]]"

        # Verify basic response fields
        assert response.id
        assert response.created
        assert response.usage.prompt_tokens > 0
        assert response.usage.completion_tokens > 0
        assert response.usage.total_tokens > 0
```
**EN:** Verify DeepSeek-OCR grounding output with coordinates This block implements `verify_single_image_response_for_ocr` and captures one focused piece of the module's behavior.
**CN:** Verify DeepSeek-OCR grounding output with coordinates 该代码块实现 `verify_single_image_response_for_ocr`，承担模块行为中的一个聚焦逻辑片段。

### Lines 202-227: test case single image chat completion / 测试用例 single image chat completion
```python
    def test_single_image_chat_completion(self):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        image_url = "https://raw.githubusercontent.com/sgl-project/sgl-test-files/refs/heads/main/images/ocr-text.png"

        response = client.chat.completions.create(
            model="default",
            messages=[
                {
                    "role": "user",
                    "content": [
                        {
                            "type": "image_url",
                            "image_url": {"url": image_url},
                        },
                        {
                            "type": "text",
                            "text": "<|grounding|>Convert the document to markdown.",
                        },
                    ],
                },
            ],
            temperature=0,
            **(self.get_vision_request_kwargs()),
        )

        self.verify_single_image_response_for_ocr(response)
```
**EN:** This test exercises `test_single_image_chat_completion` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_image_chat_completion`。

### Lines 228-230: supporting source context / 辅助源码上下文
```python


# Delete the mixin classes so that they are not collected by pytest
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 231-241: supporting statements / 辅助语句
```python
del (
    TestOpenAIMLLMServerBase,
    ImageOpenAITestMixin,
    VideoOpenAITestMixin,
    AudioOpenAITestMixin,
    OmniOpenAITestMixin,
)


if __name__ == "__main__":
    unittest.main()
```
**EN:** This block performs supporting work through calls such as `main`, preparing state for nearby definitions.
**CN:** 该代码块通过 `main` 等调用完成辅助性工作，为相邻定义准备状态。

## Key Concepts / 关键概念
- `TestLlavaServer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestLfm2VlServer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestQwen25VLServer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestQwen3VLServer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestQwen3OmniServer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestQwen2VLContextLengthServer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestInternVL25Server`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestMiniCPMV4Server`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestQwen2VLContextLengthServer.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestQwen2VLContextLengthServer.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestQwen2VLContextLengthServer.test_single_image_chat_completion`: This test exercises `test_single_image_chat_completion` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_image_chat_completion`。
- `TestKimiVLServer.test_video_images_chat_completion`: This test exercises `test_video_images_chat_completion` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_video_images_chat_completion`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `openai`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.vlm_utils`

- **Total lines / 总行数**: 241
