# gemma4_mm.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/gemma4_mm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the gemma4 Mm architecture into SGLang's serving runtime, typically defining layers, model wrappers, and weight-loading behavior for inference. / 该模块将 gemma4 Mm 架构接入 SGLang 服务运行时，通常定义推理所需的层、模型封装以及权重加载逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15: Module header and imports / 模块头与导入
```python
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
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 57-57: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 59-59: Assignment: cached_get_processor / 赋值：cached_get_processor
```python
cached_get_processor = lru_cache(get_processor)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 62-64: Class: Gemma4ImagePixelInputs / 类：Gemma4ImagePixelInputs
```python
class Gemma4ImagePixelInputs(TypedDict):
    pixel_values: torch.Tensor
    """Shape: `(batch_size * num_images, num_channels, height, width)`"""
```
**EN:** This class defines Gemma4 Image Pixel Inputs inheriting from TypedDict, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma4 Image Pixel Inputs，用于封装该模型组件的状态与方法。

### Lines 67-71: Class: Gemma4AudioInputs / 类：Gemma4AudioInputs
```python
class Gemma4AudioInputs(TypedDict):
    input_features_padded: torch.Tensor
    """Shape: `(batch_size * num_audio, seq_length, num_features)`"""
    input_features_mask: torch.Tensor
    """Shape: `(batch_size * num_audio, seq_length)`"""
```
**EN:** This class defines Gemma4 Audio Inputs inheriting from TypedDict, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma4 Audio Inputs，用于封装该模型组件的状态与方法。

### Lines 74-117: Class: Gemma4MultimodalEmbedder / 类：Gemma4MultimodalEmbedder
```python
class Gemma4MultimodalEmbedder(nn.Module):
    """Projects vision/audio soft tokens into LM embedding space."""

    def __init__(
        self,
        multimodal_config: Union[Gemma4AudioConfig, Gemma4VisionConfig],
        text_config: Gemma4TextConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()

        self.eps = multimodal_config.rms_norm_eps
        self.text_hidden_size = text_config.hidden_size

        # Audio tower uses output_proj_dims (1536) rather than hidden_size
        # (1024); vision uses hidden_size (768) directly.
        embedding_dim = (
            getattr(multimodal_config, "output_proj_dims", None)
            or multimodal_config.hidden_size
        )

        self.embedding_projection = ReplicatedLinear(
            embedding_dim,
            self.text_hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("embedding_projection", prefix),
        )

        self.embedding_pre_projection_norm = Gemma4RMSNorm(
            embedding_dim,
            eps=self.eps,
            with_scale=False,
        )

    def forward(
        self,
        inputs_embeds: torch.Tensor,
    ) -> torch.Tensor:
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 Multimodal Embedder inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: Projects vision/audio soft tokens into LM embedding space..
**CN:** 该类定义了 Gemma4 Multimodal Embedder，用于封装该模型组件的状态与方法。 文档字符串摘要：Projects vision/audio soft tokens into LM embedding space.。

### Lines 120-968: Class: Gemma4ForConditionalGeneration / 类：Gemma4ForConditionalGeneration
```python
class Gemma4ForConditionalGeneration(PreTrainedModel):
    config_class = Gemma4Config
    """Gemma4 multimodal model for conditional generation."""

    # BitandBytes specific attributes
    default_bitsandbytes_target_modules = [
        ".gate_proj.",
        ".down_proj.",
        ".up_proj.",
        ".q_proj.",
        ".k_proj.",
        ".v_proj.",
        ".o_proj.",
    ]
    bitsandbytes_stacked_params_mapping = {
        "q_proj": ("qkv_proj", 0),
        "k_proj": ("qkv_proj", 1),
        "v_proj": ("qkv_proj", 2),
        "gate_proj": ("gate_up_proj", 0),
        "up_proj": ("gate_up_proj", 1),
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
        "o_proj",
        "gate_up_proj",
        "down_proj",
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 For Conditional Generation inheriting from PreTrainedModel, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma4 For Conditional Generation，用于封装该模型组件的状态与方法。

### Lines 971-971: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = Gemma4ForConditionalGeneration
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
- `typing: Iterable, List, Optional, Set, Tuple, TypedDict, Union`
- `torch`
- `torch: nn`
- `transformers: Gemma4AudioConfig, Gemma4Config, Gemma4TextConfig, Gemma4VisionConfig, PreTrainedModel`
- `sglang.srt.layers.attention.triton_backend: TritonAttnBackend`
- `sglang.srt.layers.layernorm: Gemma4RMSNorm`
- `sglang.srt.layers.linear: ReplicatedLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.managers.mm_utils: MultiModalityDataPaddingPatternMultimodalTokens, general_mm_embed_routine`
- `sglang.srt.managers.schedule_batch: Modality, MultimodalDataItem, MultimodalInputs, flatten_nested_list`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch, ForwardMode`
- `sglang.srt.model_loader.weight_utils: default_weight_loader, maybe_remap_kv_scale_name`
- `sglang.srt.models.gemma4_audio: Gemma4AudioEncoder`
- `sglang.srt.models.gemma4_causal: Gemma4TextModel`
- `sglang.srt.models.gemma4_vision: Gemma4VisionEncoder`
- `sglang.srt.utils: add_prefix`
