# glm_ocr.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/glm_ocr.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Glm Ocr architecture into SGLang's serving runtime. Inference-only GLM-OCR model compatible with HuggingFace weights. / 该模块将 Glm Ocr 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only GLM-OCR model compatible with HuggingFace weights。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17: Module header and imports / 模块头与导入
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

# Modeling from:
# ./llama.py and
# https://github.com/huggingface/transformers/blob/main/src/transformers/models/GlmOcr/modular_GlmOcr.py
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

### Lines 62-63: Class: GlmOcrRMSNorm / 类：GlmOcrRMSNorm
```python
class GlmOcrRMSNorm(Glm4vRMSNorm):
    pass
```
**EN:** This class defines Glm Ocr R M S Norm inheriting from Glm4vRMSNorm, grouping state and methods for this model component.
**CN:** 该类定义了 Glm Ocr R M S Norm，用于封装该模型组件的状态与方法。

### Lines 66-67: Class: GlmOcrVisionMLP / 类：GlmOcrVisionMLP
```python
class GlmOcrVisionMLP(Glm4vVisionMLP):
    pass
```
**EN:** This class defines Glm Ocr Vision M L P inheriting from Glm4vVisionMLP, grouping state and methods for this model component.
**CN:** 该类定义了 Glm Ocr Vision M L P，用于封装该模型组件的状态与方法。

### Lines 70-140: Class: GlmOcrVisionBlock / 类：GlmOcrVisionBlock
```python
class GlmOcrVisionBlock(nn.Module):
    def __init__(
        self,
        dim: int,
        intermediate_dim: int,
        num_heads: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        attn_qkv_bias: bool = True,
        num_dummy_heads: int = 0,
        rms_norm_eps: float = 1e-5,
        use_data_parallel: bool = False,
    ) -> None:
        super().__init__()
        self.norm1 = RMSNorm(dim, eps=rms_norm_eps)
        self.norm2 = RMSNorm(dim, eps=rms_norm_eps)
        self.attn = VisionAttention(
            embed_dim=dim,
            num_heads=num_heads,
            projection_size=dim,
            use_qkv_parallel=True,
            qkv_bias=attn_qkv_bias,
            proj_bias=True,
            qk_normalization_by_head_size=True,
            flatten_batch=True,
            quant_config=quant_config,
            prefix=add_prefix("attn", prefix),
            num_dummy_heads=num_dummy_heads,
            use_data_parallel=use_data_parallel,
        )
        self.mlp = GlmOcrVisionMLP(
            dim,
            intermediate_dim,
            bias=True,
            quant_config=quant_config,
            prefix=add_prefix("mlp", prefix),
            use_data_parallel=use_data_parallel,
        )

    def forward(
# ... truncated for brevity ...
```
**EN:** This class defines Glm Ocr Vision Block inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Glm Ocr Vision Block，用于封装该模型组件的状态与方法。

### Lines 143-144: Class: GlmOcrVisionPatchEmbed / 类：GlmOcrVisionPatchEmbed
```python
class GlmOcrVisionPatchEmbed(Glm4vVisionPatchEmbed):
    pass
```
**EN:** This class defines Glm Ocr Vision Patch Embed inheriting from Glm4vVisionPatchEmbed, grouping state and methods for this model component.
**CN:** 该类定义了 Glm Ocr Vision Patch Embed，用于封装该模型组件的状态与方法。

### Lines 147-148: Class: GlmOcrVisionPatchMerger / 类：GlmOcrVisionPatchMerger
```python
class GlmOcrVisionPatchMerger(Glm4vPatchMerger):
    pass
```
**EN:** This class defines Glm Ocr Vision Patch Merger inheriting from Glm4vPatchMerger, grouping state and methods for this model component.
**CN:** 该类定义了 Glm Ocr Vision Patch Merger，用于封装该模型组件的状态与方法。

### Lines 151-261: Class: GlmOcrVisionModel / 类：GlmOcrVisionModel
```python
class GlmOcrVisionModel(Glm4vVisionModel):
    def __init__(
        self,
        vision_config: GlmOcrVisionConfig,
        text_config: GlmOcrTextConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        use_data_parallel: bool = False,
    ) -> None:
        super().__init__(vision_config, quant_config, prefix, use_data_parallel)

        patch_size = vision_config.patch_size
        temporal_patch_size = vision_config.temporal_patch_size
        in_channels = vision_config.in_channels
        depth = vision_config.depth
        self.hidden_size = vision_config.hidden_size
        self.num_heads = vision_config.num_heads

        self.patch_size = vision_config.patch_size
        self.spatial_merge_size = vision_config.spatial_merge_size
        self.out_hidden_size = vision_config.out_hidden_size
        self.intermediate_size = vision_config.intermediate_size
        self.use_data_parallel = use_data_parallel

        self.patch_embed = GlmOcrVisionPatchEmbed(
            patch_size=patch_size,
            temporal_patch_size=temporal_patch_size,
            in_channels=in_channels,
            hidden_size=self.hidden_size,
        )

        head_dim = self.hidden_size // self.num_heads
        self.rotary_pos_emb = get_rope(
            head_size=head_dim,
            rotary_dim=head_dim // 2,
            max_position=8192,
            base=10000.0,
            is_neox_style=True,
        )

# ... truncated for brevity ...
```
**EN:** This class defines Glm Ocr Vision Model inheriting from Glm4vVisionModel, grouping state and methods for this model component.
**CN:** 该类定义了 Glm Ocr Vision Model，用于封装该模型组件的状态与方法。

### Lines 264-435: Class: GlmOcrForConditionalGeneration / 类：GlmOcrForConditionalGeneration
```python
class GlmOcrForConditionalGeneration(Glm4vForConditionalGeneration):
    def __init__(
        self,
        config: GlmOcrConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__(config, quant_config, prefix)

        self.pp_group = get_pp_group()
        self.config = config
        self.use_data_parallel = get_global_server_args().mm_enable_dp_encoder
        self.visual = GlmOcrVisionModel(
            vision_config=config.vision_config,
            text_config=config.text_config,
            quant_config=quant_config,
            prefix=add_prefix("visual", prefix),
            use_data_parallel=self.use_data_parallel,
        )

        vision_utils.update_vit_attn_dummy_heads_config(self.config)

        self.model = Glm4Model(
            config,
            quant_config=quant_config,
            prefix=add_prefix("model", prefix),
        )

        if self.pp_group.is_last_rank:
            if self.pp_group.world_size == 1 and self.config.tie_word_embeddings:
                self.lm_head = self.model.embed_tokens
            else:
                self.lm_head = ParallelLMHead(
                    self.config.vocab_size,
                    self.config.hidden_size,
                    quant_config=quant_config,
                    prefix=add_prefix("lm_head", prefix),
                )
        else:
            # ranks other than the last rank will have a placeholder layer
# ... truncated for brevity ...
```
**EN:** This class defines Glm Ocr For Conditional Generation inheriting from Glm4vForConditionalGeneration, grouping state and methods for this model component.
**CN:** 该类定义了 Glm Ocr For Conditional Generation，用于封装该模型组件的状态与方法。

### Lines 438-438: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [GlmOcrForConditionalGeneration]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `logging`
- `functools: lru_cache`
- `typing: Iterable, Optional, Tuple`
- `torch`
- `torch.nn`
- `einops: rearrange`
- `transformers.models.glm_ocr.configuration_glm_ocr: GlmOcrConfig, GlmOcrTextConfig, GlmOcrVisionConfig`
- `sglang.srt.distributed.parallel_state: get_pp_group`
- `sglang.srt.layers.attention: vision_utils`
- `sglang.srt.layers.attention.vision: VisionAttention`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.pooler: Pooler, PoolingType`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.rotary_embedding: get_rope`
- `sglang.srt.layers.utils: PPMissingLayer`
- `sglang.srt.layers.vocab_parallel_embedding: ParallelLMHead`
- `sglang.srt.model_loader.weight_utils: default_weight_loader`
- `sglang.srt.models.glm4: Glm4Model`
- `sglang.srt.models.glm4v: Glm4vForConditionalGeneration, Glm4vPatchMerger, Glm4vRMSNorm, Glm4vVisionMLP, Glm4vVisionModel, Glm4vVisionPatchEmbed`
