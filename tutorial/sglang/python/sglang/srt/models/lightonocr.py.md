# lightonocr.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/lightonocr.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Lightonocr architecture into SGLang's serving runtime. Support for lightonai/LightOnOCR-2-1B. / 该模块将 Lightonocr 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Support for lightonai/LightOnOCR-2-1B。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14: Module header and imports / 模块头与导入
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

### Lines 58-295: Class: LightOnOCRForConditionalGeneration / 类：LightOnOCRForConditionalGeneration
```python
class LightOnOCRForConditionalGeneration(nn.Module):
    """
    LightOnOCR model for SGLang inference.

    Architecture:
    - Pixtral-based vision encoder (PixtralHFVisionModel, 24 layers)
    - RMSNorm on vision encoder output
    - Spatial merge via PatchMerger (2x2 = 4x token reduction)
    - VisionLanguageAdapter projection to text hidden size
    - Qwen3-based decoder (28 layers) with QK norms
    """

    merge_by_field_config = True

    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
            return None
        raise ValueError("Only image modality is supported")

    def __init__(self, *, config, prefix: str = "", **kwargs):
        super().__init__()
        self.config = config
        quant_config = kwargs.get("quant_config")

        # Build VisionEncoderArgs from config
        vision_config = config.vision_config
        dataclass_fields = {field.name for field in fields(VisionEncoderArgs)}
        vision_args = {
            key: value
            for key, value in vision_config.to_dict().items()
            if key in dataclass_fields
        }
        # LightOnOCR stores these at the top-level config
        if "image_token_id" not in vision_args:
            vision_args["image_token_id"] = getattr(config, "image_token_id", 151655)
        if "spatial_merge_size" not in vision_args:
            vision_args["spatial_merge_size"] = getattr(config, "spatial_merge_size", 2)
        if "adapter_bias" not in vision_args:
            vision_args["adapter_bias"] = getattr(
# ... truncated for brevity ...
```
**EN:** This class defines Light On O C R For Conditional Generation inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: LightOnOCR model for SGLang inference..
**CN:** 该类定义了 Light On O C R For Conditional Generation，用于封装该模型组件的状态与方法。 文档字符串摘要：LightOnOCR model for SGLang inference.。

### Lines 298-298: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = LightOnOCRForConditionalGeneration
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `dataclasses: fields`
- `typing: Iterable, List, Tuple`
- `torch`
- `torch.nn`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.managers.mm_utils: MultiModalityDataPaddingPatternMultimodalTokens, general_mm_embed_routine`
- `sglang.srt.managers.schedule_batch: MultimodalDataItem, MultimodalInputs`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch`
- `sglang.srt.model_loader.weight_utils: default_weight_loader`
- `sglang.srt.models.pixtral: PATCH_MERGE, PatchMerger, PixtralHFVisionModel, VisionEncoderArgs, VisionLanguageAdapter`
- `sglang.srt.models.qwen3: Qwen3ForCausalLM`
