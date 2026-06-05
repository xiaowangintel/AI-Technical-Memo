# gemma3_mm.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/gemma3_mm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the gemma3 Mm architecture into SGLang's serving runtime, typically defining layers, model wrappers, and weight-loading behavior for inference. / 该模块将 gemma3 Mm 架构接入 SGLang 服务运行时，通常定义推理所需的层、模型封装以及权重加载逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19: Module header and imports / 模块头与导入
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Copyright 2025 SGLang Team
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

# Adapted from:
# https://github.com/vllm-project/vllm/blob/main/vllm/model_executor/models/gemma3_mm.py
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 52-52: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 54-54: Assignment: cached_get_processor / 赋值：cached_get_processor
```python
cached_get_processor = lru_cache(get_processor)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 57-59: Class: Gemma3ImagePixelInputs / 类：Gemma3ImagePixelInputs
```python
class Gemma3ImagePixelInputs(TypedDict):
    pixel_values: torch.Tensor
    """Shape: `(batch_size * num_images, num_channels, height, width)`"""
```
**EN:** This class defines Gemma3 Image Pixel Inputs inheriting from TypedDict, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma3 Image Pixel Inputs，用于封装该模型组件的状态与方法。

### Lines 62-110: Class: Gemma3MultiModalProjector / 类：Gemma3MultiModalProjector
```python
class Gemma3MultiModalProjector(nn.Module):
    """Projector for Gemma3 multimodal."""

    def __init__(self, config: Gemma3Config):
        super().__init__()

        self.mm_input_projection_weight = nn.Parameter(
            torch.zeros(
                config.vision_config.hidden_size, config.text_config.hidden_size
            )
        )

        self.mm_soft_emb_norm = Gemma3RMSNorm(
            config.vision_config.hidden_size, eps=config.vision_config.layer_norm_eps
        )

        self.patches_per_image = int(
            config.vision_config.image_size // config.vision_config.patch_size
        )
        self.tokens_per_side = int(config.mm_tokens_per_image**0.5)
        self.kernel_size = self.patches_per_image // self.tokens_per_side
        self.avg_pool = nn.AvgPool2d(
            kernel_size=self.kernel_size, stride=self.kernel_size
        )

    def forward(self, vision_outputs: torch.Tensor) -> torch.Tensor:
        batch_size, seq_length, hidden_size = vision_outputs.shape

        # Reshape for pooling
        reshaped_vision_outputs = vision_outputs.transpose(1, 2)
        reshaped_vision_outputs = reshaped_vision_outputs.reshape(
            batch_size, hidden_size, self.patches_per_image, self.patches_per_image
        )
        reshaped_vision_outputs = reshaped_vision_outputs.contiguous()

        # Apply pooling
        pooled_vision_outputs = self.avg_pool(reshaped_vision_outputs)
        pooled_vision_outputs = pooled_vision_outputs.flatten(2)
        pooled_vision_outputs = pooled_vision_outputs.transpose(1, 2)

# ... truncated for brevity ...
```
**EN:** This class defines Gemma3 Multi Modal Projector inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: Projector for Gemma3 multimodal..
**CN:** 该类定义了 Gemma3 Multi Modal Projector，用于封装该模型组件的状态与方法。 文档字符串摘要：Projector for Gemma3 multimodal.。

### Lines 113-499: Class: Gemma3ForConditionalGeneration / 类：Gemma3ForConditionalGeneration
```python
class Gemma3ForConditionalGeneration(PreTrainedModel):
    config_class = Gemma3Config
    """Gemma3 multimodal model for conditional generation."""

    # BitandBytes specific attributes
    default_bitsandbytes_target_modules = [
        ".gate_proj.",
        ".down_proj.",
        ".up_proj.",
        ".q_proj.",
        ".k_proj.",
        ".v_proj.",
        ".o_proj.",
        ".out_proj.",
    ]
    bitsandbytes_stacked_params_mapping = {
        # shard_name, weight_name, index
        "q_proj": ("qkv_proj", 0),
        "k_proj": ("qkv_proj", 1),
        "v_proj": ("qkv_proj", 2),
        "gate_proj": ("gate_up_proj", 0),
        "up_proj": ("gate_up_proj", 1),
        "out_proj": ("proj", 0),
    }

    packed_modules_mapping = {
        "qkv_proj": [
            "q_proj",
            "k_proj",
            "v_proj",
        ],
        "gate_up_proj": [
            "gate_proj",
            "up_proj",
        ],
    }

    # LoRA specific attributes
    supported_lora_modules = [
        "qkv_proj",
# ... truncated for brevity ...
```
**EN:** This class defines Gemma3 For Conditional Generation inheriting from PreTrainedModel, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma3 For Conditional Generation，用于封装该模型组件的状态与方法。

### Lines 502-502: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = Gemma3ForConditionalGeneration
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `logging`
- `re`
- `functools: lru_cache`
- `typing: Iterable, List, Optional, Set, Tuple, TypedDict`
- `torch`
- `torch: nn`
- `transformers: Gemma3Config, PreTrainedModel`
- `sglang.srt.layers.attention.triton_backend: TritonAttnBackend`
- `sglang.srt.layers.layernorm: Gemma3RMSNorm`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.managers.mm_utils: MultiModalityDataPaddingPatternTokenPairs, general_mm_embed_routine`
- `sglang.srt.managers.schedule_batch: MultimodalDataItem, MultimodalInputs, flatten_nested_list`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch, ForwardMode`
- `sglang.srt.model_loader.weight_utils: default_weight_loader, maybe_remap_kv_scale_name`
- `sglang.srt.models.gemma3_causal: Gemma3ForCausalLM`
- `sglang.srt.models.siglip: SiglipVisionModel`
- `sglang.srt.utils: add_prefix`
- `sglang.srt.utils.hf_transformers_utils: get_processor`
