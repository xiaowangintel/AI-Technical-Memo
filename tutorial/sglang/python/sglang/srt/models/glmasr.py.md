# glmasr.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/glmasr.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Glmasr architecture into SGLang's serving runtime. Inference-only GLM-ASR-HF model compatible with HuggingFace weights. / 该模块将 Glmasr 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only GLM-ASR-HF model compatible with HuggingFace weights。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17: Module header and imports / 模块头与导入
```python
# Copyright 2023-2025 SGLang Team
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

# Modeling from:
# ./llama.py and
# https://github.com/huggingface/transformers/blob/main/src/transformers/models/glmasr/modular_glmasr.py
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 46-46: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 49-168: Class: GlmAsrForConditionalGeneration / 类：GlmAsrForConditionalGeneration
```python
class GlmAsrForConditionalGeneration(nn.Module):
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
        # shard_name, weight_name, index
        "q_proj": ("qkv_proj", 0),
        "k_proj": ("qkv_proj", 1),
        "v_proj": ("qkv_proj", 2),
        "gate_proj": ("gate_up_proj", 0),
        "up_proj": ("gate_up_proj", 1),
    }

    def __init__(
        self,
        config: GlmAsrConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.config = config

        if getattr(self.config, "audio_config", None) is None:
            self.config.audio_config = GlmAsrEncoderConfig(self.config._name_or_path)

        self.audio_tower = GlmAsrEncoder(
            config.audio_config,
        )
        self.multi_modal_projector = GlmAsrMultiModalProjector(config)
        self.language_model = LlamaForCausalLM(
            config.text_config, quant_config, prefix=add_prefix("model", prefix)
        )
# ... truncated for brevity ...
```
**EN:** This class defines Glm Asr For Conditional Generation inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Glm Asr For Conditional Generation，用于封装该模型组件的状态与方法。

### Lines 171-171: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = GlmAsrForConditionalGeneration
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `logging`
- `typing: Any, Iterable, List, Optional, Tuple`
- `torch`
- `torch.nn`
- `transformers: GlmAsrConfig, GlmAsrEncoderConfig`
- `transformers.models.glmasr.modeling_glmasr: GlmAsrEncoder, GlmAsrMultiModalProjector`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.managers.mm_utils: MultiModalityDataPaddingPatternMultimodalTokens, general_mm_embed_routine`
- `sglang.srt.managers.schedule_batch: Modality, MultimodalDataItem, MultimodalInputs`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch`
- `sglang.srt.model_loader.weight_utils: default_weight_loader`
- `sglang.srt.models.llama: LlamaForCausalLM`
- `sglang.srt.utils: add_prefix`
