# test_vlm_accuracy.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/test_vlm_accuracy.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `vlm accuracy` scenario in `test/manual`. It uses SGLang's shared test infrastructure to configure models or services and verify expected accuracy and functional behavior. / 该手动测试模块覆盖 `test/manual` 中的 `vlm accuracy` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的精度与功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Imports and shared helpers / 导入与共享辅助项
```python
""" """

import unittest
from typing import List, Optional

import numpy as np
import torch
import torch.nn.functional as F
from transformers import AutoModel, AutoProcessor, AutoTokenizer

from sglang.srt.configs.model_config import ModelConfig
from sglang.srt.entrypoints.openai.protocol import ChatCompletionRequest
from sglang.srt.managers.mm_utils import embed_mm_inputs, init_mm_embedding_cache
from sglang.srt.managers.schedule_batch import (
    Modality,
    MultimodalDataItem,
    MultimodalInputs,
)
from sglang.srt.model_executor.model_runner import ModelRunner
from sglang.srt.multimodal.processors.base_processor import BaseMultimodalProcessor
from sglang.srt.parser.conversation import generate_chat_conv
from sglang.srt.server_args import ServerArgs
from sglang.test.test_utils import download_image_with_retry


# Test the logits output between HF and SGLang
```
**EN:** This range imports `unittest`, `typing`, `numpy` and `torch`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. It interacts with request/response APIs to observe live model behavior.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 它会与请求/响应 API 交互，以观察模型的实时行为。

### Lines 27-28: Class definition for VisionLLMLogitsBase / 类定义
```python
class VisionLLMLogitsBase(unittest.IsolatedAsyncioTestCase):
    @classmethod
```
**EN:** This range declares `VisionLLMLogitsBase`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 29-35: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.image_url = "https://github.com/sgl-project/sglang/blob/main/examples/assets/example_image.png?raw=true"
        cls.device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
        cls.model_path = ""
        cls.chat_template = ""
        cls.processor = ""
        cls.main_image = download_image_with_retry(cls.image_url)
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `device`, `is_available` and `download_image_with_retry`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 37-51: Helper routines around compare_outputs / 辅助例程
```python
    def compare_outputs(self, sglang_output: torch.Tensor, hf_output: torch.Tensor):
        # Convert to float32 for numerical stability if needed
        hf = hf_output.float()
        sg = sglang_output.float()

        # Basic shape and dtype comparison
        print("\n=== Basic Properties ===")
        print(f"Shapes match: {hf.shape == sg.shape}")
        print(f"HF shape: {hf.shape}, SGLang shape: {sg.shape}")
        print(f"HF dtype: {hf.dtype}, SGLang dtype: {sg.dtype}")

        # Move tensors to CPU for numpy operations
        hf_np = hf.cpu().numpy()
        sg_np = sg.cpu().numpy()
```
**EN:** This range implements helper routine(s) `compare_outputs` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `cpu` and `numpy`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 52-61: Scenario logic / 场景逻辑
```python
        # Statistical metrics
        print("\n=== Statistical Metrics ===")
        print(f"Mean absolute difference: {torch.mean(torch.abs(hf - sg)).item():.6f}")
        print(f"Max absolute difference: {torch.max(torch.abs(hf - sg)).item():.6f}")
        print(f"Mean squared error: {torch.mean((hf - sg) ** 2).item():.6f}")
        print(
            f"Root mean squared error: {torch.sqrt(torch.mean((hf - sg) ** 2)).item():.6f}"
        )

        # Cosine similarity (across feature dimension)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `mean`, `abs`, `item` and `sqrt`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 62-76: Scenario logic / 场景逻辑
```python
        cos_sim = F.cosine_similarity(hf, sg)
        print(f"Mean cosine similarity: {torch.mean(cos_sim).item():.6f}")
        print(f"Min cosine similarity: {torch.min(cos_sim).item():.6f}")

        # Find largest absolute differences
        print("\n=== Largest Absolute Differences ===")
        diffs = torch.abs(hf - sg)
        flat_diffs = diffs.flatten()

        # Get indices of top 10 differences
        top_k = 10
        top_values, top_flat_indices = torch.topk(flat_diffs, top_k)

        # Convert flat indices to multidimensional indices
        top_indices = np.unravel_index(top_flat_indices.cpu().numpy(), diffs.shape)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `cosine_similarity`, `mean`, `item` and `abs`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 77-86: Scenario logic / 场景逻辑
```python

        print(f"\nTop {top_k} largest absolute differences:")
        print(
            "Index".ljust(30)
            + "Difference".ljust(15)
            + "HF Value".ljust(15)
            + "SGLang Value"
        )
        print("-" * 75)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `ljust`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 87-98: Assertions and result checks / 断言与结果检查
```python
        for i in range(top_k):
            # Get the index tuple for this difference
            idx = tuple(dim[i] for dim in top_indices)
        diff_val = top_values[i].item()
        hf_val = hf[idx].item()
        sg_val = sg[idx].item()

        # Format the index tuple and values
        idx_str = str(idx)
        print(f"{idx_str:<30}{diff_val:<15.6f}{hf_val:<15.6f}{sg_val:.6f}")

        np.testing.assert_allclose(hf_np, sg_np)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `item` and `assert_allclose`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 100-101: Helper routines around get_completion_request / 辅助例程
```python
    def get_completion_request(self) -> ChatCompletionRequest:
        json_str = f"""
```
**EN:** This range implements helper routine(s) `get_completion_request` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 102-124: Scenario logic / 场景逻辑
```python
        {{
  "model": "{self.model_path}",
  "messages": [
    {{
      "role": "user",
      "content": [
        {{
          "type": "image_url",
          "image_url": {{
            "url": "{self.image_url}"
          }}
        }},
        {{
          "type": "text",
          "text": "What's in this picture?"
        }}
      ]
    }}
  ]
}}
        """

        return ChatCompletionRequest.model_validate_json(json_str)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `model_validate_json`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 126-126: Helper routines around get_processor_output / 辅助例程
```python
    def get_processor_output(self, req: Optional[ChatCompletionRequest] = None):
```
**EN:** This range implements helper routine(s) `get_processor_output` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 127-140: Scenario logic / 场景逻辑
```python
        if req is None:
            req = self.get_completion_request()
        conv = generate_chat_conv(req, template_name=self.chat_template)
        text = conv.get_prompt()

        # Process inputs using processor
        # FIXME: the formal arguments may differ
        inputs = self.processor(
            text=[text],
            images=[self.main_image],
            return_tensors="pt",
        ).to(self.device)

        return inputs
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `get_completion_request`, `generate_chat_conv`, `get_prompt` and `processor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 142-151: Helper routines around get_sglang_model / 辅助例程
```python
    def get_sglang_model(self):
        self.model_runner = ModelRunner(
            model_config=ModelConfig(self.model_path, model_override_args="{}"),
            mem_fraction_static=0.8,
            gpu_id=0,
            tp_rank=0,
            tp_size=1,
            pp_rank=0,
            pp_size=1,
            nccl_port=12435,
```
**EN:** This range implements helper routine(s) `get_sglang_model` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `ModelRunner` and `ModelConfig`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 152-157: Scenario logic / 场景逻辑
```python
            server_args=ServerArgs(
                model_path=self.model_path,
                disable_cuda_graph=True,
            ),
        )
        return self.model_runner.model
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `ServerArgs`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 160-161: Class definition for TestMiniCPMV2_6Logits / 类定义
```python
class TestMiniCPMV2_6Logits(VisionLLMLogitsBase):
    @classmethod
```
**EN:** This range declares `TestMiniCPMV2_6Logits`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 162-181: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        super().setUpClass()
        cls.model_path = "openbmb/MiniCPM-V-2_6"
        cls.tokenizer = AutoTokenizer.from_pretrained(
            cls.model_path, trust_remote_code=True
        )
        cls.processor = AutoProcessor.from_pretrained(
            cls.model_path, trust_remote_code=True
        )
        cls.chat_template = "minicpmv"

        cls.device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
        cls.hf_model = (
            AutoModel.from_pretrained(
                cls.model_path, torch_dtype=torch.bfloat16, trust_remote_code=True
            )
            .eval()
            .to(cls.device)
        )
        init_mm_embedding_cache()
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `from_pretrained`, `device`, `is_available` and `eval`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 183-184: Test routines around test_vlm_embedding_output / 测试例程
```python
    async def test_vlm_embedding_output(self):
        """
```
**EN:** This range defines concrete test routine(s) `test_vlm_embedding_output`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 185-207: Scenario logic / 场景逻辑
```python
        Compares the embedding output of vlm
        """
        inputs = self.get_processor_output()

        with torch.no_grad():
            # hf
            model_inputs = {
                "input_ids": inputs.input_ids,
                "image_bound": inputs.image_bound,
                "pixel_values": inputs.pixel_values,
                "tgt_sizes": inputs.tgt_sizes,
            }
            hf_output, _ = self.hf_model.get_vllm_embedding(
                model_inputs,
            )
            hf_output = hf_output.squeeze(0)

            # sglang
            model = self.get_sglang_model()
            input_ids = inputs["input_ids"].to(self.device).flatten()

            pixel_values = inputs["pixel_values"]
            tgt_sizes = inputs["tgt_sizes"]
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `get_processor_output`, `no_grad`, `get_vllm_embedding` and `squeeze`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 208-209: Scenario logic / 场景逻辑
```python
            pixel_values_flat: List[torch.Tensor] = []
            tgt_sizes_flat: List[torch.Tensor] = []
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 210-232: Scenario logic / 场景逻辑
```python
            for pixel_b, tgt_b in zip(pixel_values, tgt_sizes):
                # per image
                if len(pixel_b) != len(tgt_b):
                    raise ValueError(
                        "Inconsistent N lengths, found: "
                        f"{len(pixel_b)} vs {len(tgt_b)}"
                    )
                for pixel_n, tgt_n in zip(pixel_b, tgt_b):
                    pixel_values_flat += [pixel_n]
                    tgt_sizes_flat += [tgt_n]

            im_start_id, im_end_id = (
                self.tokenizer.im_start_id,
                self.tokenizer.im_end_id,
            )
            slice_start_id, slice_end_id = (
                self.tokenizer.slice_start_id,
                self.tokenizer.slice_end_id,
            )

            image_offsets = BaseMultimodalProcessor.get_mm_items_offset_by_pair(
                input_ids=input_ids, mm_start_id=im_start_id, mm_end_id=im_end_id
            )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `ValueError` and `get_mm_items_offset_by_pair`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 233-234: Scenario logic / 场景逻辑
```python
            slice_offsets = BaseMultimodalProcessor.get_mm_items_offset_by_pair(
                input_ids=input_ids, mm_start_id=slice_start_id, mm_end_id=slice_end_id
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `get_mm_items_offset_by_pair`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 235-257: Scenario logic / 场景逻辑
```python
            )
            image_offsets.extend(slice_offsets)
            image_offsets = sorted(image_offsets)

            sglang_output = embed_mm_inputs(
                mm_inputs_list=[
                    MultimodalInputs(
                        mm_items=[
                            MultimodalDataItem(
                                feature=pixel_values_flat,
                                offsets=image_offsets,
                                tgt_size=tgt_sizes_flat,
                                modality=Modality.IMAGE,
                                pad_value=self.processor.tokenizer.unk_token_id,
                            )
                        ]
                    ),
                ],
                extend_prefix_lens=[0],
                extend_seq_lens=[input_ids.shape[0]],
                input_ids=input_ids,
                input_embedding=model.get_input_embeddings(),
                multimodal_model=model,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `extend`, `sorted`, `embed_mm_inputs` and `MultimodalInputs`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 258-259: Scenario logic / 场景逻辑
```python
                placeholder_tokens={
                    Modality.IMAGE: self.processor.tokenizer.unk_token_id,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 260-263: Scenario logic / 场景逻辑
```python
                },
            )

        self.compare_outputs(sglang_output, hf_output)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `compare_outputs`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 266-267: Class definition for TestMiniCPMV4Logits / 类定义
```python
class TestMiniCPMV4Logits(VisionLLMLogitsBase):
    @classmethod
```
**EN:** This range declares `TestMiniCPMV4Logits`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 268-287: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        super().setUpClass()
        cls.model_path = "openbmb/MiniCPM-V-4"
        cls.tokenizer = AutoTokenizer.from_pretrained(
            cls.model_path, trust_remote_code=True
        )
        cls.processor = AutoProcessor.from_pretrained(
            cls.model_path, trust_remote_code=True
        )
        cls.chat_template = "minicpmv"

        cls.device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
        cls.hf_model = (
            AutoModel.from_pretrained(
                cls.model_path, torch_dtype=torch.bfloat16, trust_remote_code=True
            )
            .eval()
            .to(cls.device)
        )
        init_mm_embedding_cache()
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `from_pretrained`, `device`, `is_available` and `eval`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 289-290: Test routines around test_vlm_embedding_output / 测试例程
```python
    async def test_vlm_embedding_output(self):
        """
```
**EN:** This range defines concrete test routine(s) `test_vlm_embedding_output`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 291-313: Scenario logic / 场景逻辑
```python
        Compares the embedding output of vlm
        """
        inputs = self.get_processor_output()

        with torch.no_grad():
            # hf
            model_inputs = {
                "input_ids": inputs.input_ids,
                "image_bound": inputs.image_bound,
                "pixel_values": inputs.pixel_values,
                "tgt_sizes": inputs.tgt_sizes,
            }
            hf_output = self.hf_model.get_input_embeddings()(inputs.input_ids)

            # sglang
            model = self.get_model()
            sglang_output = self.vlm_func(
                model,
                input_ids=inputs.input_ids.to(self.device),
                pixel_values=inputs.pixel_values,
                image_bound=inputs.image_bound.to(self.device),
                tgt_sizes=inputs.tgt_sizes.to(self.device),
                input_embedding=model.get_input_embeddings(),
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `get_processor_output`, `no_grad`, `get_input_embeddings` and `get_model`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 314-315: Scenario logic / 场景逻辑
```python
                multimodal_model=model,
                placeholder_tokens={
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 316-320: Scenario logic / 场景逻辑
```python
                    Modality.IMAGE: self.processor.tokenizer.unk_token_id,
                },
            )

        self.compare_outputs(sglang_output, hf_output)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `compare_outputs`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Accuracy evaluation / 精度评测
- Multi-GPU orchestration / 多 GPU 编排
- HTTP/API interaction / HTTP/API 交互
- Token-level inspection / Token 级分析
- Multimodal inputs / 多模态输入

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `typing`, `unittest`
- **Third-party / 第三方库**: `numpy`, `torch`, `torch.nn.functional`, `transformers`
- **Project Modules / 项目模块**: `sglang.srt.configs.model_config`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.managers.mm_utils`, `sglang.srt.managers.schedule_batch`, `sglang.srt.model_executor.model_runner`, `sglang.srt.multimodal.processors.base_processor`, `sglang.srt.parser.conversation`, `sglang.srt.server_args`, `sglang.test.test_utils`
