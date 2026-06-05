# test_vlm_input_format.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/vlm/test_vlm_input_format.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates vlm input format behavior in SGLang's vlm area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 vlm 领域中与 vlm input format 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: module imports and dependencies / 模块导入与依赖
```python
import json
import unittest
from io import BytesIO
from typing import Optional

import requests
import torch

# Compatibility shim: Kimi-VL dynamic module expects PytorchGELUTanh which may
# be missing in transformers==4.57.1. Inject a lightweight implementation so
# the model can import successfully without downgrading transformers.
import transformers.activations as _hf_activations
from PIL import Image
from transformers import (
    AutoModel,
    AutoProcessor,
    Gemma3ForConditionalGeneration,
    Qwen2_5_VLForConditionalGeneration,
)

from sglang.test.ci.ci_register import register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `unittest`, `io`, `typing`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `unittest`, `io`, `typing`。

### Lines 23-33: supporting statements / 辅助语句
```python
if not hasattr(_hf_activations, "PytorchGELUTanh"):

    class PytorchGELUTanh(torch.nn.Module):
        def forward(self, x):
            return torch.nn.functional.gelu(x, approximate="tanh")

    _hf_activations.PytorchGELUTanh = PytorchGELUTanh
    _hf_activations.ACT2FN.setdefault(
        "pytorch_gelu_tanh",
        lambda x: torch.nn.functional.gelu(x, approximate="tanh"),
    )
```
**EN:** This block performs supporting work through calls such as `hasattr`, `setdefault`, `gelu`, preparing state for nearby definitions.
**CN:** 该代码块通过 `hasattr`, `setdefault`, `gelu` 等调用完成辅助性工作，为相邻定义准备状态。

### Lines 35-39: module imports and dependencies / 模块导入与依赖
```python
from sglang import Engine
from sglang.srt.entrypoints.openai.protocol import ChatCompletionRequest
from sglang.srt.parser.conversation import generate_chat_conv
from sglang.srt.utils.common import is_cuda, is_xpu
from sglang.srt.utils.hf_transformers_utils import _fix_added_tokens_encoding
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.parser.conversation`, `sglang.srt.utils.common`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.parser.conversation`, `sglang.srt.utils.common`。

### Lines 41-47: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=747, stage="base-b", runner_config="1-gpu-large")

IMAGE_MAN_IRONING_URL = "https://raw.githubusercontent.com/sgl-project/sgl-test-files/refs/heads/main/images/man_ironing_on_back_of_suv.png"
IMAGE_SGL_LOGO_URL = "https://raw.githubusercontent.com/sgl-project/sgl-test-files/refs/heads/main/images/sgl_logo.png"

_is_cuda = is_cuda()
_is_xpu = is_xpu()
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, is_cuda, is_xpu.
**CN:** 该代码块通过 register_cuda_ci, is_cuda, is_xpu 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 50-50: class VLMInputTestBase declaration / 类 VLMInputTestBase 声明
```python
class VLMInputTestBase:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 51-54: class-level constants and configuration for `VLMInputTestBase` / 类级常量与配置
```python
    model_path = None
    chat_template = None
    processor = None
    visual = None  # Should be a callable for precomputed embeddings
```
**EN:** This block defines shared names such as `model_path`, `chat_template`, `processor`, `visual`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model_path`, `chat_template`, `processor`, `visual` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 56-78: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        assert cls.model_path is not None, "Set model_path in subclass"
        assert cls.chat_template is not None, "Set chat_template in subclass"

        cls.image_urls = [IMAGE_MAN_IRONING_URL, IMAGE_SGL_LOGO_URL]
        if _is_cuda:
            cls.device = torch.device("cuda")
        elif _is_xpu:
            cls.device = torch.device("xpu")
        else:
            cls.device = torch.device("cpu")

        cls.main_image = []
        for image_url in cls.image_urls:
            response = requests.get(image_url)
            cls.main_image.append(Image.open(BytesIO(response.content)))

        cls.processor = AutoProcessor.from_pretrained(
            cls.model_path, trust_remote_code=True, use_fast=True
        )
        _fix_added_tokens_encoding(cls.processor.tokenizer)
        cls._init_visual()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 80-83: method init visual / 方法 init visual
```python
    @classmethod
    def _init_visual(cls):
        """Override in subclass to set up cls.visual as a callable for precomputed embeddings."""
        raise NotImplementedError
```
**EN:** Override in subclass to set up cls.visual as a callable for precomputed embeddings. This block implements `_init_visual` and captures one focused piece of the module's behavior.
**CN:** Override in subclass to set up cls.visual as a callable for precomputed embeddings. 该代码块实现 `_init_visual`，承担模块行为中的一个聚焦逻辑片段。

### Lines 85-94: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.engine = Engine(
            model_path=self.model_path,
            chat_template=self.chat_template,
            device=self.device.type,
            mem_fraction_static=0.8,
            enable_multimodal=True,
            disable_cuda_graph=True,
            trust_remote_code=True,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 96-97: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        self.engine.shutdown()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 99-123: method verify response / 方法 verify response
```python
    def verify_response(self, output):
        # The goal is to check that the model roughly understands:
        #   - image 1: taxi / car scene
        #   - image 2: SGL logo / company
        # We intentionally keep the check keyword-based and loose to avoid
        # overfitting to a specific phrasing.
        out_text = output["text"].lower()

        assert any(w in out_text for w in ("taxi", "cab", "car")), out_text

        has_sg_or_logo_side = any(
            kw in out_text
            for kw in (
                "sg ",
                "sgl",
                " sgl",
                "logo",
                "software guidance",
                "labs",
                "laborator",
                "company",
                " text",
            )
        )
        assert has_sg_or_logo_side, out_text
```
**EN:** This block implements `verify_response` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `verify_response`，承担模块行为中的一个聚焦逻辑片段。

### Lines 125-143: helper routine get completion request / 辅助流程 get completion request
```python
    def get_completion_request(self) -> ChatCompletionRequest:
        json_structure = {
            "model": self.model_path,
            "messages": [
                {
                    "role": "user",
                    "content": [
                        {"type": "image_url", "image_url": {"url": self.image_urls[0]}},
                        {"type": "image_url", "image_url": {"url": self.image_urls[1]}},
                        {
                            "type": "text",
                            "text": "Describe both the first image and the second image in detail separately.",  # update prompt, ensure kimi-vl understands the images separately.
                        },
                    ],
                }
            ],
        }
        json_str = json.dumps(json_structure)
        return ChatCompletionRequest.model_validate_json(json_str)
```
**EN:** This helper encapsulates `get_completion_request` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_completion_request`，以便周围测试复用准备、执行或校验逻辑。

### Lines 145-158: helper routine get processor output / 辅助流程 get processor output
```python
    def get_processor_output(self, req: Optional[ChatCompletionRequest] = None):
        if req is None:
            req = self.get_completion_request()
        conv = generate_chat_conv(req, template_name=self.chat_template)
        text = conv.get_prompt()

        # Process inputs using processor
        inputs = self.processor(
            text=[text],
            images=self.main_image,
            return_tensors="pt",
        ).to(self.device)

        return inputs, text
```
**EN:** This helper encapsulates `get_processor_output` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_processor_output`，以便周围测试复用准备、执行或校验逻辑。

### Lines 160-169: test case accepts image / 测试用例 accepts image
```python
    async def test_accepts_image(self):
        req = self.get_completion_request()
        conv = generate_chat_conv(req, template_name=self.chat_template)
        text = conv.get_prompt()
        output = await self.engine.async_generate(
            prompt=text,
            image_data=self.main_image,
            sampling_params=dict(temperature=0.0, max_new_tokens=512),
        )
        self.verify_response(output)
```
**EN:** This test exercises `test_accepts_image` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_accepts_image`。

### Lines 171-185: test case accepts precomputed embeddings / 测试用例 accepts precomputed embeddings
```python
    async def test_accepts_precomputed_embeddings(self):
        req = self.get_completion_request()
        processor_output, _ = self.get_processor_output(req=req)

        with torch.inference_mode():
            precomputed_embeddings = self.__class__.visual(processor_output)

        output = await self.engine.async_generate(
            input_ids=processor_output["input_ids"][0].detach().cpu().tolist(),
            image_data=[
                self._precomputed_image_data(processor_output, precomputed_embeddings)
            ],
            sampling_params=dict(temperature=0.0, max_new_tokens=512),
        )
        self.verify_response(output)
```
**EN:** This test exercises `test_accepts_precomputed_embeddings` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_accepts_precomputed_embeddings`。

### Lines 187-195: test case accepts processor output / 测试用例 accepts processor output
```python
    async def test_accepts_processor_output(self):
        req = self.get_completion_request()
        processor_output, prompt = self.get_processor_output(req=req)
        output = await self.engine.async_generate(
            input_ids=processor_output["input_ids"][0].detach().cpu().tolist(),
            image_data=[self._processor_output_image_data(processor_output)],
            sampling_params=dict(temperature=0.0, max_new_tokens=512),
        )
        self.verify_response(output)
```
**EN:** This test exercises `test_accepts_processor_output` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_accepts_processor_output`。

### Lines 197-203: method precomputed image data / 方法 precomputed image data
```python
    def _precomputed_image_data(self, processor_output, precomputed_embeddings):
        """This should not be overridden."""
        return dict(
            processor_output,
            format="precomputed_embedding",
            feature=precomputed_embeddings,
        )
```
**EN:** This should not be overridden. This block implements `_precomputed_image_data` and captures one focused piece of the module's behavior.
**CN:** This should not be overridden. 该代码块实现 `_precomputed_image_data`，承担模块行为中的一个聚焦逻辑片段。

### Lines 205-207: method processor output image data / 方法 processor output image data
```python
    def _processor_output_image_data(self, processor_output):
        """Override in subclass to pass the correct set of arguments."""
        raise NotImplementedError
```
**EN:** Override in subclass to pass the correct set of arguments. This block implements `_processor_output_image_data` and captures one focused piece of the module's behavior.
**CN:** Override in subclass to pass the correct set of arguments. 该代码块实现 `_processor_output_image_data`，承担模块行为中的一个聚焦逻辑片段。

### Lines 210-210: class TestQwenVLUnderstandsImage declaration / 类 TestQwenVLUnderstandsImage 声明
```python
class TestQwenVLUnderstandsImage(VLMInputTestBase, unittest.IsolatedAsyncioTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `VLMInputTestBase`, `unittest.IsolatedAsyncioTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `VLMInputTestBase`, `unittest.IsolatedAsyncioTestCase`。

### Lines 211-212: class-level constants and configuration for `TestQwenVLUnderstandsImage` / 类级常量与配置
```python
    model_path = "Qwen/Qwen2.5-VL-3B-Instruct"
    chat_template = "qwen2-vl"
```
**EN:** This block defines shared names such as `model_path`, `chat_template`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model_path`, `chat_template` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 214-231: method init visual / 方法 init visual
```python
    @classmethod
    def _init_visual(cls):
        model = Qwen2_5_VLForConditionalGeneration.from_pretrained(
            cls.model_path, torch_dtype=torch.bfloat16
        ).eval()
        # In transformers v5, .visual moved under .model
        visual = model.model.visual
        cls.visual_model = visual.to(cls.device)

        # In transformers v5, the visual encoder returns BaseModelOutputWithPooling;
        # pooler_output has the spatially-merged embeddings we need.
        def visual(processor_output):
            out = cls.visual_model(
                processor_output["pixel_values"], processor_output["image_grid_thw"]
            )
            return out.pooler_output if hasattr(out, "pooler_output") else out

        cls.visual = visual
```
**EN:** This block implements `_init_visual` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_init_visual`，承担模块行为中的一个聚焦逻辑片段。

### Lines 233-234: method processor output image data / 方法 processor output image data
```python
    def _processor_output_image_data(self, processor_output):
        return dict(processor_output, format="processor_output")
```
**EN:** This block implements `_processor_output_image_data` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_processor_output_image_data`，承担模块行为中的一个聚焦逻辑片段。

### Lines 237-237: class TestGemmaUnderstandsImage declaration / 类 TestGemmaUnderstandsImage 声明
```python
class TestGemmaUnderstandsImage(VLMInputTestBase, unittest.IsolatedAsyncioTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `VLMInputTestBase`, `unittest.IsolatedAsyncioTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `VLMInputTestBase`, `unittest.IsolatedAsyncioTestCase`。

### Lines 238-239: class-level constants and configuration for `TestGemmaUnderstandsImage` / 类级常量与配置
```python
    model_path = "google/gemma-3-4b-it"
    chat_template = "gemma-it"
```
**EN:** This block defines shared names such as `model_path`, `chat_template`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model_path`, `chat_template` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 241-259: method init visual / 方法 init visual
```python
    @classmethod
    def _init_visual(cls):
        model = Gemma3ForConditionalGeneration.from_pretrained(
            cls.model_path, torch_dtype=torch.bfloat16
        )
        base_model = model.model

        cls.vision_tower = base_model.vision_tower.eval().to(cls.device)

        if hasattr(base_model, "multi_modal_projector"):
            cls.mm_projector = base_model.multi_modal_projector.eval().to(cls.device)
        else:
            cls.mm_projector = model.multi_modal_projector.eval().to(cls.device)

        cls.visual = lambda processor_output: cls.mm_projector(
            cls.vision_tower(
                pixel_values=processor_output["pixel_values"]
            ).last_hidden_state
        )
```
**EN:** This block implements `_init_visual` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_init_visual`，承担模块行为中的一个聚焦逻辑片段。

### Lines 261-262: method processor output image data / 方法 processor output image data
```python
    def _processor_output_image_data(self, processor_output):
        return dict(processor_output, format="processor_output")
```
**EN:** This block implements `_processor_output_image_data` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_processor_output_image_data`，承担模块行为中的一个聚焦逻辑片段。

### Lines 263-265: supporting source context / 辅助源码上下文
```python


# Updated Kimi-VL test to use the new input format.
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 266-268: class TestKimiVLImageUnderstandsImage declaration / 类 TestKimiVLImageUnderstandsImage 声明
```python
class TestKimiVLImageUnderstandsImage(
    VLMInputTestBase, unittest.IsolatedAsyncioTestCase
):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `VLMInputTestBase`, `unittest.IsolatedAsyncioTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `VLMInputTestBase`, `unittest.IsolatedAsyncioTestCase`。

### Lines 269-270: class-level constants and configuration for `TestKimiVLImageUnderstandsImage` / 类级常量与配置
```python
    model_path = "moonshotai/Kimi-VL-A3B-Instruct"
    chat_template = "kimi-vl"
```
**EN:** This block defines shared names such as `model_path`, `chat_template`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model_path`, `chat_template` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 272-317: method init visual / 方法 init visual
```python
    @classmethod
    def _init_visual(cls):
        import inspect

        from transformers import AutoConfig
        from transformers.dynamic_module_utils import get_class_from_dynamic_module

        config = AutoConfig.from_pretrained(cls.model_path, trust_remote_code=True)

        # Transformers v5 auto-populates rope_scaling with
        # {"rope_theta": ..., "rope_type": "default"} even when the original
        # config had rope_scaling: null. The remote KimiVL code branches on
        # `if self.config.rope_scaling is None` so we must reset it.
        tc = getattr(config, "text_config", None)
        if tc is not None:
            rs = getattr(tc, "rope_scaling", None)
            if isinstance(rs, dict) and rs.get("rope_type") == "default":
                tc.rope_scaling = None

        # Transformers v5 calls tie_weights(recompute_mapping=False) in
        # post_init, but KimiVL's tie_weights doesn't accept that kwarg.
        auto_map = getattr(config, "auto_map", {})
        model_ref = auto_map.get("AutoModel")
        if model_ref:
            model_cls = get_class_from_dynamic_module(model_ref, cls.model_path)
            orig_tie = model_cls.tie_weights
            if "recompute_mapping" not in inspect.signature(orig_tie).parameters:

                def _patched_tie(self, **kwargs):
                    return orig_tie(self)

                model_cls.tie_weights = _patched_tie

        model = AutoModel.from_pretrained(
            cls.model_path, config=config, trust_remote_code=True
        )
        cls.vision_tower = model.vision_tower.eval().to(cls.device)
        cls.mm_projector = model.multi_modal_projector.eval().to(cls.device)
        _vt_dtype = next(cls.vision_tower.parameters()).dtype

        cls.visual = lambda tokenizer_output: cls.mm_projector(
            cls.vision_tower(
                pixel_values=tokenizer_output["pixel_values"].to(_vt_dtype),
                grid_hws=tokenizer_output["image_grid_hws"],
            )
        )
```
**EN:** This block implements `_init_visual` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_init_visual`，承担模块行为中的一个聚焦逻辑片段。

### Lines 319-320: method processor output image data / 方法 processor output image data
```python
    def _processor_output_image_data(self, processor_output):
        return dict(processor_output, format="processor_output")
```
**EN:** This block implements `_processor_output_image_data` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_processor_output_image_data`，承担模块行为中的一个聚焦逻辑片段。

### Lines 321-405: supporting source context / 辅助源码上下文
```python


# not for CI: too large
# class TestLlama4ImageUnderstandsImage(
#     VLMInputTestBase, unittest.IsolatedAsyncioTestCase
# ):
#     # Allow overriding via env for local/offline runs.
#     model_path = "meta-llama/Llama-4-Scout-17B-16E-Instruct"
#     chat_template = "llama-4"

#     def setUp(self):
#         if torch.cuda.device_count() < 4:
#             self.skipTest("Skipping Llama-4 test: requires 4 GPUs for TP=4")
#         self.engine = Engine(
#             model_path=self.model_path,
#             trust_remote_code=True,
#             chat_template=self.chat_template,
#             enable_multimodal=True,
#             mem_fraction_static=0.8,
#             tp_size=4,
#             attention_backend="fa3",
#             context_length=65536,
#         )

#     @classmethod
#     def _init_visual(cls):
#         model = AutoModel.from_pretrained(
#             cls.model_path,
#             trust_remote_code=True,
#             torch_dtype="auto",
#             force_download=True,
#         )
#         cls.vision_tower = model.vision_model.eval().to(cls.device)
#         cls.mm_projector = model.multi_modal_projector.eval().to(cls.device)

#         cls.visual = lambda tokenizer_output: cls.mm_projector(
#             cls.vision_tower(
#                 pixel_values=tokenizer_output["pixel_values"],
#             ).last_hidden_state.flatten(0, -2)
#         )

#     def _processor_output_image_data(self, processor_output):
#         # Llama-4 vision expects processor_output format with pixel_values
#         return dict(processor_output, format="processor_output")


# class TestLlavaUnderstandsImage(VLMInputTestBase, unittest.IsolatedAsyncioTestCase):
#     model_path = "llava-hf/llava-1.5-7b-hf"
#     chat_template = "vicuna_v1.1"

#     @classmethod
#     def _init_visual(cls):
#         from transformers import LlavaForConditionalGeneration

#         model = LlavaForConditionalGeneration.from_pretrained(
#             cls.model_path,
#             torch_dtype=torch.float16,
#             low_cpu_mem_usage=True,
#         )
#         cls.vision_tower = model.vision_tower.eval().to(cls.device)
#         cls.multi_modal_projector = model.multi_modal_projector.eval().to(cls.device)
#         cls.config = model.config

#         def visual_func(processor_output):
#             pixel_values = processor_output["pixel_values"].to(
#                 cls.device, dtype=torch.float16
#             )

#             vision_outputs = cls.vision_tower(pixel_values, output_hidden_states=True)
#             image_features = vision_outputs.hidden_states[-2]

#             if cls.config.vision_feature_select_strategy == "default":
#                 image_features = image_features[:, 1:]
#             elif cls.config.vision_feature_select_strategy == "full":
#                 image_features = image_features

#             image_features = cls.multi_modal_projector(image_features)
#             return image_features

#         cls.visual = visual_func

#     def _processor_output_image_data(self, processor_output):
#         return dict(processor_output, format="processor_output")


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 406-406: class TestInternVLUnderstandsImage declaration / 类 TestInternVLUnderstandsImage 声明
```python
class TestInternVLUnderstandsImage(VLMInputTestBase, unittest.IsolatedAsyncioTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `VLMInputTestBase`, `unittest.IsolatedAsyncioTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `VLMInputTestBase`, `unittest.IsolatedAsyncioTestCase`。

### Lines 407-408: class-level constants and configuration for `TestInternVLUnderstandsImage` / 类级常量与配置
```python
    model_path = "OpenGVLab/InternVL2-2B"
    chat_template = "internvl-2-5"
```
**EN:** This block defines shared names such as `model_path`, `chat_template`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model_path`, `chat_template` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 410-429: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        assert cls.model_path is not None, "Set model_path in subclass"
        assert cls.chat_template is not None, "Set chat_template in subclass"
        cls.image_urls = [IMAGE_MAN_IRONING_URL, IMAGE_SGL_LOGO_URL]
        cls.device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
        cls.main_image = []
        for image_url in cls.image_urls:
            response = requests.get(image_url)
            cls.main_image.append(Image.open(BytesIO(response.content)))

        # InternVL models (2, 3, 3.5) do not ship a standard HuggingFace
        # Processor; AutoProcessor.from_pretrained returns a bare tokenizer.
        # Use AutoTokenizer explicitly so the intent is clear.
        from transformers import AutoTokenizer

        cls.processor = AutoTokenizer.from_pretrained(
            cls.model_path, trust_remote_code=True
        )
        cls._init_visual()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 431-510: method init visual (part 1/2) / 方法 init visual（第 1/2 部分）
```python
    @classmethod
    def _init_visual(cls):
        try:
            model = AutoModel.from_pretrained(
                cls.model_path,
                trust_remote_code=True,
                torch_dtype=torch.bfloat16,
                low_cpu_mem_usage=False,
            )
        except (RuntimeError, AttributeError) as e:
            if isinstance(e, RuntimeError) and "meta" not in str(e):
                raise
            # Transformers v5 always uses meta tensors for init, which breaks
            # models calling .item() in __init__ (e.g. InternVL's drop_path_rate).
            # Transformers v5.5.3 may also raise AttributeError for remote-code
            # models missing new internal attributes (e.g. all_tied_weights_keys).
            # Fall back to from_config + manual weight loading.
            import gc
            import glob
            import os

            from huggingface_hub import snapshot_download
            from safetensors.torch import load_file
            from transformers import AutoConfig

            config = AutoConfig.from_pretrained(cls.model_path, trust_remote_code=True)
            with torch.device("cpu"):
                model = AutoModel.from_config(
                    config,
                    trust_remote_code=True,
                    torch_dtype=torch.bfloat16,
                )
            model_dir = snapshot_download(cls.model_path)
            for f in sorted(glob.glob(os.path.join(model_dir, "*.safetensors"))):
                shard = load_file(f)
                model.load_state_dict(shard, strict=False)
                del shard
            gc.collect()

        cls.vision_model = model.vision_model.eval().to(cls.device)
        cls.mlp1 = model.mlp1.eval().to(cls.device)

        config = model.config
        cls.internvl_config = config
        image_size = getattr(config, "force_image_size", None) or (
            config.vision_config.image_size
        )
        patch_size = config.vision_config.patch_size
        cls.num_image_token = int(
            (image_size // patch_size) ** 2 * (config.downsample_ratio**2)
        )
        cls.internvl_image_size = image_size
        cls.internvl_downsample_ratio = config.downsample_ratio
        cls.internvl_ps_version = config.ps_version
        cls.internvl_select_layer = config.select_layer

        del model

        def pixel_shuffle(x, scale_factor):
            n, w, h, c = x.size()
            x = x.view(n, w, int(h * scale_factor), int(c / scale_factor))
            x = x.permute(0, 2, 1, 3).contiguous()
            x = x.view(
                n,
                int(h * scale_factor),
                int(w * scale_factor),
                int(c / (scale_factor * scale_factor)),
            )
            if cls.internvl_ps_version != "v1":
                x = x.permute(0, 2, 1, 3).contiguous()
            return x

        def visual_func(processor_output):
            pixel_values = processor_output["pixel_values"].to(
                cls.device, dtype=torch.bfloat16
            )
            if cls.internvl_select_layer == -1:
                vit_embeds = cls.vision_model(
                    pixel_values=pixel_values,
                    output_hidden_states=False,
```
**EN:** This block implements `_init_visual` and captures one focused piece of the module's behavior. This chunk continues part 1 of the same logical block.
**CN:** 该代码块实现 `_init_visual`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 1 部分。

### Lines 511-532: method init visual (part 2/2) / 方法 init visual（第 2/2 部分）
```python
                    return_dict=True,
                ).last_hidden_state
            else:
                vit_embeds = cls.vision_model(
                    pixel_values=pixel_values,
                    output_hidden_states=True,
                    return_dict=True,
                ).hidden_states[cls.internvl_select_layer]
            vit_embeds = vit_embeds[:, 1:, :]

            h = w = int(vit_embeds.shape[1] ** 0.5)
            vit_embeds = vit_embeds.reshape(vit_embeds.shape[0], h, w, -1)
            vit_embeds = pixel_shuffle(
                vit_embeds, scale_factor=cls.internvl_downsample_ratio
            )
            vit_embeds = vit_embeds.reshape(
                vit_embeds.shape[0], -1, vit_embeds.shape[-1]
            )
            vit_embeds = cls.mlp1(vit_embeds)
            return vit_embeds

        cls.visual = visual_func
```
**EN:** This block implements `_init_visual` and captures one focused piece of the module's behavior. This chunk continues part 2 of the same logical block.
**CN:** 该代码块实现 `_init_visual`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 2 部分。

### Lines 534-587: helper routine get processor output / 辅助流程 get processor output
```python
    def get_processor_output(self, req=None):
        """Override to handle InternVL's custom preprocessing.

        Uses shared ``image_to_pixel_values`` from ``internvl_utils`` for
        image preprocessing (dynamic tiling + normalize) and expands
        ``<IMG_CONTEXT>`` placeholders into ``<img>`` + context tokens +
        ``</img>`` — mirroring the logic in
        ``InternVLProcessor.process_internlm2_mm_data_async``.
        """
        from sglang.srt.multimodal.internvl_utils import image_to_pixel_values
        from sglang.srt.multimodal.processors.internvl import InternVLProcessor

        if req is None:
            req = self.get_completion_request()
        conv = generate_chat_conv(req, template_name=self.chat_template)
        text = conv.get_prompt()

        # Preprocess images using the shared utility (dynamic tiling +
        # bicubic resize + ImageNet normalize), same pipeline as the engine.
        all_pixel_values = []
        num_patches_list = []
        for img in self.main_image:
            pv = image_to_pixel_values(
                img,
                input_size=self.internvl_image_size,
                max_num_tiles=InternVLProcessor.IMAGE_MAX_NUM,
                use_thumbnail=True,
            )
            all_pixel_values.append(pv)
            num_patches_list.append(pv.shape[0])

        pixel_values = torch.cat(all_pixel_values, dim=0).to(self.device)

        # Expand each <IMG_CONTEXT> placeholder into <img> + <IMG_CONTEXT>*N + </img>.
        # This mirrors InternVLProcessor.process_internlm2_mm_data_async.
        ph = "<<<__IMG_PH__>>>"
        expanded_text = text.replace(InternVLProcessor.IMG_CONTEXT, ph)
        for num_patches in num_patches_list:
            image_tokens = (
                InternVLProcessor.IMG_START
                + InternVLProcessor.IMG_CONTEXT * (self.num_image_token * num_patches)
                + InternVLProcessor.IMG_END
            )
            expanded_text = expanded_text.replace(ph, image_tokens, 1)
        # Remove any remaining placeholders (more placeholders than images)
        expanded_text = expanded_text.replace(ph, "")

        # Tokenize the expanded text
        input_ids = self.processor(expanded_text, return_tensors="pt")["input_ids"]

        return {
            "input_ids": input_ids,
            "pixel_values": pixel_values,
        }, text
```
**EN:** Override to handle InternVL's custom preprocessing. This helper encapsulates `get_processor_output` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Override to handle InternVL's custom preprocessing. 该辅助函数封装了 `get_processor_output`，以便周围测试复用准备、执行或校验逻辑。

### Lines 589-590: method processor output image data / 方法 processor output image data
```python
    def _processor_output_image_data(self, processor_output):
        return dict(processor_output, format="processor_output")
```
**EN:** This block implements `_processor_output_image_data` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_processor_output_image_data`，承担模块行为中的一个聚焦逻辑片段。

### Lines 593-593: class TestMiniCPMVUnderstandsImage declaration / 类 TestMiniCPMVUnderstandsImage 声明
```python
class TestMiniCPMVUnderstandsImage(VLMInputTestBase, unittest.IsolatedAsyncioTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `VLMInputTestBase`, `unittest.IsolatedAsyncioTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `VLMInputTestBase`, `unittest.IsolatedAsyncioTestCase`。

### Lines 594-595: class-level constants and configuration for `TestMiniCPMVUnderstandsImage` / 类级常量与配置
```python
    model_path = "openbmb/MiniCPM-V-4"
    chat_template = "minicpmv"
```
**EN:** This block defines shared names such as `model_path`, `chat_template`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model_path`, `chat_template` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 597-619: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        assert cls.model_path is not None, "Set model_path in subclass"
        assert cls.chat_template is not None, "Set chat_template in subclass"
        cls.image_urls = [IMAGE_MAN_IRONING_URL, IMAGE_SGL_LOGO_URL]
        cls.device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
        cls.main_image = []
        for image_url in cls.image_urls:
            response = requests.get(image_url)
            cls.main_image.append(Image.open(BytesIO(response.content)))

        cls.processor = AutoProcessor.from_pretrained(
            cls.model_path, trust_remote_code=True
        )
        # In transformers v5.5.3, AutoTokenizer may return TokenizersBackend
        # which lacks model-specific attributes (e.g. im_start_id for MiniCPM-V).
        # Replace with sglang's tokenizer which handles this via declared-class
        # fallback, then fix added tokens encoding.
        from sglang.srt.utils.hf_transformers import get_tokenizer

        cls.processor.tokenizer = get_tokenizer(cls.model_path, trust_remote_code=True)
        _fix_added_tokens_encoding(cls.processor.tokenizer)
        cls._init_visual()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 621-700: method init visual (part 1/2) / 方法 init visual（第 1/2 部分）
```python
    @classmethod
    def _init_visual(cls):
        try:
            model = AutoModel.from_pretrained(
                cls.model_path, trust_remote_code=True, torch_dtype=torch.bfloat16
            )
        except (AttributeError, RuntimeError) as e:
            err = str(e)
            if "all_tied_weights_keys" not in err and "meta" not in err:
                raise
            # Transformers v5: remote model code may lack all_tied_weights_keys
            # or meta-tensor init may break .item() calls.  Fall back to
            # from_config + manual weight loading.
            import gc
            import glob
            import os

            from huggingface_hub import snapshot_download
            from safetensors.torch import load_file
            from transformers import AutoConfig

            config = AutoConfig.from_pretrained(cls.model_path, trust_remote_code=True)
            with torch.device("cpu"):
                model = AutoModel.from_config(
                    config,
                    trust_remote_code=True,
                    torch_dtype=torch.bfloat16,
                )
            model_dir = snapshot_download(cls.model_path)
            for f in sorted(glob.glob(os.path.join(model_dir, "*.safetensors"))):
                shard = load_file(f)
                model.load_state_dict(shard, strict=False)
                del shard
            gc.collect()

        cls.vpm_model = model.vpm.eval().to(cls.device)
        cls.resampler_model = model.resampler.eval().to(cls.device)
        del model

        def visual_func(processor_output):
            pixel_values = processor_output["pixel_values"]
            tgt_sizes = processor_output["tgt_sizes"]

            pixel_values_flat = []
            tgt_sizes_flat = []
            for pixel_b, tgt_b in zip(pixel_values, tgt_sizes):
                if isinstance(pixel_b, (list, tuple)):
                    for pixel_n, tgt_n in zip(pixel_b, tgt_b):
                        pixel_values_flat.append(pixel_n)
                        tgt_sizes_flat.append(tgt_n)
                else:
                    pixel_values_flat.append(pixel_b)
                    tgt_sizes_flat.append(tgt_b)

            tgt_sizes_tensor = torch.stack(tgt_sizes_flat, dim=0)
            device = cls.vpm_model.embeddings.position_embedding.weight.device
            dtype = cls.vpm_model.embeddings.position_embedding.weight.dtype

            all_pixel_values_lst = [
                i.flatten(end_dim=1).permute(1, 0) for i in pixel_values_flat
            ]
            max_patches = int(
                (tgt_sizes_tensor[:, 0] * tgt_sizes_tensor[:, 1]).max().item()
            )
            all_pixel_values = torch.nn.utils.rnn.pad_sequence(
                all_pixel_values_lst, batch_first=True, padding_value=0.0
            )
            B, L, _ = all_pixel_values.shape
            all_pixel_values = all_pixel_values.permute(0, 2, 1).reshape(B, 3, -1, L)
            patch_attn_mask = torch.zeros(
                (B, 1, max_patches), dtype=torch.bool, device=device
            )
            tgt_sizes_dev = tgt_sizes_tensor.to(device)
            mask_shapes = tgt_sizes_dev[:, 0] * tgt_sizes_dev[:, 1]
            patch_attn_mask[:, 0, :] = torch.arange(
                max_patches, device=device
            ).unsqueeze(0) < mask_shapes.unsqueeze(1)

            vision_output = cls.vpm_model(
                all_pixel_values.type(dtype),
```
**EN:** This block implements `_init_visual` and captures one focused piece of the module's behavior. This chunk continues part 1 of the same logical block.
**CN:** 该代码块实现 `_init_visual`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 1 部分。

### Lines 701-707: method init visual (part 2/2) / 方法 init visual（第 2/2 部分）
```python
                patch_attention_mask=patch_attn_mask,
                tgt_sizes=tgt_sizes_tensor,
            )
            vision_embedding = vision_output.last_hidden_state
            return cls.resampler_model(vision_embedding, tgt_sizes_tensor)

        cls.visual = visual_func
```
**EN:** This block implements `_init_visual` and captures one focused piece of the module's behavior. This chunk continues part 2 of the same logical block.
**CN:** 该代码块实现 `_init_visual`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 2 部分。

### Lines 709-710: method processor output image data / 方法 processor output image data
```python
    def _processor_output_image_data(self, processor_output):
        return dict(processor_output, format="processor_output")
```
**EN:** This block implements `_processor_output_image_data` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_processor_output_image_data`，承担模块行为中的一个聚焦逻辑片段。

### Lines 713-714: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `VLMInputTestBase`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestQwenVLUnderstandsImage`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestGemmaUnderstandsImage`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestKimiVLImageUnderstandsImage`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestInternVLUnderstandsImage`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestMiniCPMVUnderstandsImage`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `VLMInputTestBase.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `VLMInputTestBase._init_visual`: Override in subclass to set up cls.visual as a callable for precomputed embeddings. / 该代码块实现 `_init_visual`，承担模块行为中的一个聚焦逻辑片段。
- `VLMInputTestBase.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `VLMInputTestBase.tearDown`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `VLMInputTestBase.verify_response`: This block implements `verify_response` and captures one focused piece of the module's behavior. / 该代码块实现 `verify_response`，承担模块行为中的一个聚焦逻辑片段。
- `VLMInputTestBase.get_completion_request`: This helper encapsulates `get_completion_request` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `get_completion_request`，以便周围测试复用准备、执行或校验逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `unittest`, `io`, `typing`
- **Third-party modules / 第三方模块**: `requests`, `torch`, `transformers.activations`, `PIL`, `transformers`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.parser.conversation`, `sglang.srt.utils.common`, `sglang.srt.utils.hf_transformers_utils`

- **Total lines / 总行数**: 714
