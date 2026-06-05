# glm4v.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/glm4v.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the glm4v architecture into SGLang's serving runtime. Inference-only GLM-4.1V model compatible with HuggingFace weights. / 该模块将 glm4v 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only GLM-4.1V model compatible with HuggingFace weights。

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
# https://github.com/huggingface/transformers/blob/main/src/transformers/models/glm4v/modular_glm4v.py
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 64-64: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 66-66: Assignment: cached_get_processor / 赋值：cached_get_processor
```python
cached_get_processor = lru_cache(get_processor)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 69-75: Class: Glm4vRMSNorm / 类：Glm4vRMSNorm
```python
class Glm4vRMSNorm(RMSNorm):
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        original_shape = x.shape
        x_2d = x.contiguous().reshape(-1, original_shape[-1])
        x_2d = super().forward(x_2d)
        x = x_2d.reshape(original_shape)
        return x
```
**EN:** This class defines Glm4v R M S Norm inheriting from RMSNorm, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4v R M S Norm，用于封装该模型组件的状态与方法。

### Lines 78-117: Class: Glm4vVisionMLP / 类：Glm4vVisionMLP
```python
class Glm4vVisionMLP(nn.Module):
    def __init__(
        self,
        in_features: int,
        hidden_features: int,
        bias: bool = False,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        use_data_parallel: bool = False,
    ):
        super().__init__()
        self.tp_size = (
            1 if use_data_parallel else get_tensor_model_parallel_world_size()
        )
        self.tp_rank = 0 if use_data_parallel else get_tensor_model_parallel_rank()
        self.gate_up_proj = MergedColumnParallelLinear(
            input_size=in_features,
            output_sizes=[hidden_features] * 2,  # [gate_proj, up_proj]
            bias=bias,
            quant_config=quant_config,
            prefix=add_prefix("gate_up_proj", prefix),
            tp_size=self.tp_size,
            tp_rank=self.tp_rank,
        )
        self.down_proj = RowParallelLinear(
            hidden_features,
            in_features,
            bias=bias,
            quant_config=quant_config,
            prefix=add_prefix("down_proj", prefix),
            tp_size=self.tp_size,
            tp_rank=self.tp_rank,
        )
        self.act_fn = SiluAndMul()

    def forward(self, x: torch.Tensor):
        gate_up, _ = self.gate_up_proj(x)
        x = self.act_fn(gate_up)
        x, _ = self.down_proj(x)
        return x
```
**EN:** This class defines Glm4v Vision M L P inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4v Vision M L P，用于封装该模型组件的状态与方法。

### Lines 120-189: Class: Glm4vVisionBlock / 类：Glm4vVisionBlock
```python
class Glm4vVisionBlock(nn.Module):
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
            proj_bias=False,
            qkv_bias=attn_qkv_bias,
            flatten_batch=True,
            quant_config=quant_config,
            prefix=add_prefix("attn", prefix),
            num_dummy_heads=num_dummy_heads,
            use_data_parallel=use_data_parallel,
        )
        self.mlp = Glm4vVisionMLP(
            dim,
            intermediate_dim,
            quant_config=quant_config,
            prefix=add_prefix("mlp", prefix),
            use_data_parallel=use_data_parallel,
        )

    def forward(
        self,
# ... truncated for brevity ...
```
**EN:** This class defines Glm4v Vision Block inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4v Vision Block，用于封装该模型组件的状态与方法。

### Lines 192-225: Class: Glm4vVisionPatchEmbed / 类：Glm4vVisionPatchEmbed
```python
class Glm4vVisionPatchEmbed(nn.Module):
    def __init__(
        self,
        patch_size: int = 14,
        temporal_patch_size: int = 2,
        in_channels: int = 3,
        hidden_size: int = 1536,
    ) -> None:
        super().__init__()
        self.patch_size = patch_size
        self.temporal_patch_size = temporal_patch_size
        self.hidden_size = hidden_size
        self.in_channels = in_channels

        kernel_size = (temporal_patch_size, patch_size, patch_size)
        self.proj = Conv3dLayer(
            in_channels,
            hidden_size,
            kernel_size=kernel_size,
            stride=kernel_size,
            bias=True,
        )

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        # Input x is 2-D: (num_patches, C * T * P * P)
        # Reshape to 5-D for Conv3dLayer, then flatten back.
        x = x.view(
            -1,
            self.in_channels,
            self.temporal_patch_size,
            self.patch_size,
            self.patch_size,
        )
        return self.proj(x).view(-1, self.hidden_size)
```
**EN:** This class defines Glm4v Vision Patch Embed inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4v Vision Patch Embed，用于封装该模型组件的状态与方法。

### Lines 228-277: Class: Glm4vPatchMerger / 类：Glm4vPatchMerger
```python
class Glm4vPatchMerger(nn.Module):
    def __init__(
        self,
        d_model: int,
        context_dim: int,
        quant_config: Optional[QuantizationConfig] = None,
        bias: bool = False,
        prefix: str = "",
        use_data_parallel: bool = False,
    ) -> None:
        super().__init__()
        self.hidden_size = d_model
        tp_size = 1 if use_data_parallel else get_tensor_model_parallel_world_size()
        tp_rank = 0 if use_data_parallel else get_tensor_model_parallel_rank()
        self.proj = ReplicatedLinear(
            self.hidden_size,
            self.hidden_size,
            bias=bias,
            quant_config=quant_config,
            prefix=add_prefix("proj", prefix),
        )
        self.post_projection_norm = LayerNorm(self.hidden_size)
        self.gate_up_proj = MergedColumnParallelLinear(
            input_size=self.hidden_size,
            output_sizes=[context_dim] * 2,
            bias=bias,
            quant_config=quant_config,
            prefix=add_prefix("gate_up_proj", prefix),
            tp_size=tp_size,
            tp_rank=tp_rank,
        )
        self.down_proj = RowParallelLinear(
            context_dim,
            self.hidden_size,
            bias=bias,
            quant_config=quant_config,
            prefix=add_prefix("down_proj", prefix),
            tp_size=tp_size,
            tp_rank=tp_rank,
        )
# ... truncated for brevity ...
```
**EN:** This class defines Glm4v Patch Merger inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4v Patch Merger，用于封装该模型组件的状态与方法。

### Lines 280-368: Class: Glm4vVisionEmbeddings / 类：Glm4vVisionEmbeddings
```python
class Glm4vVisionEmbeddings(nn.Module):
    def __init__(self, config: Glm4vVisionConfig):
        super().__init__()
        self.config = config
        self.embed_dim = config.hidden_size
        self.image_size = config.image_size
        self.patch_size = config.patch_size

        self.num_patches = (self.image_size // self.patch_size) ** 2
        self.num_positions = self.num_patches
        self.position_embedding = nn.Embedding(self.num_positions, self.embed_dim)
        self.register_buffer(
            "position_ids",
            torch.arange(self.num_positions).expand((1, -1)),
            persistent=False,
        )

    def forward(
        self, embeddings, lengths, image_shapes, h_coords, w_coords
    ) -> torch.Tensor:
        pos_embed_weight = self.position_embedding.weight
        hidden_size = pos_embed_weight.shape[1]
        total_seq = h_coords.shape[0]
        device = pos_embed_weight.device

        # Move coordinates to correct device
        h_coords, w_coords = h_coords.to(device), w_coords.to(device)

        # Handle empty sequence case
        if total_seq == 0:
            adapted_pos_embed = torch.empty(
                0, hidden_size, device=device, dtype=pos_embed_weight.dtype
            )
        else:
            # Convert inputs to tensors if needed
            if isinstance(lengths, list):
                lengths = torch.tensor(lengths, device=device, dtype=torch.long)
            if not isinstance(image_shapes, torch.Tensor):
                image_shapes = torch.tensor(
                    image_shapes, device=device, dtype=torch.long
# ... truncated for brevity ...
```
**EN:** This class defines Glm4v Vision Embeddings inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4v Vision Embeddings，用于封装该模型组件的状态与方法。

### Lines 371-542: Class: Glm4vVisionModel / 类：Glm4vVisionModel
```python
class Glm4vVisionModel(nn.Module):
    def __init__(
        self,
        vision_config: Glm4vVisionConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        use_data_parallel: bool = False,
    ) -> None:
        super().__init__()

        patch_size = vision_config.patch_size
        temporal_patch_size = vision_config.temporal_patch_size
        in_channels = vision_config.in_channels
        depth = vision_config.depth
        self.hidden_size = vision_config.hidden_size
        self.num_heads = vision_config.num_heads

        self.patch_size = vision_config.patch_size
        self.spatial_merge_size = vision_config.spatial_merge_size
        self.out_hidden_size = vision_config.out_hidden_size
        self.use_data_parallel = use_data_parallel

        self.patch_embed = Glm4vVisionPatchEmbed(
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

        self.blocks = nn.ModuleList(
            [
# ... truncated for brevity ...
```
**EN:** This class defines Glm4v Vision Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4v Vision Model，用于封装该模型组件的状态与方法。

### Lines 545-817: Class: Glm4vForConditionalGeneration / 类：Glm4vForConditionalGeneration
```python
class Glm4vForConditionalGeneration(nn.Module):
    def __init__(
        self,
        config: Glm4vConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.pp_group = get_pp_group()
        self.config = config
        self.use_data_parallel = get_global_server_args().mm_enable_dp_encoder
        vision_utils.update_vit_attn_dummy_heads_config(self.config)
        self.visual = Glm4vVisionModel(
            config.vision_config,
            quant_config=quant_config,
            prefix=add_prefix("visual", prefix),
            use_data_parallel=self.use_data_parallel,
        )

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
            self.lm_head = PPMissingLayer()

# ... truncated for brevity ...
```
**EN:** This class defines Glm4v For Conditional Generation inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4v For Conditional Generation，用于封装该模型组件的状态与方法。

### Lines 820-820: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [Glm4vForConditionalGeneration]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `logging`
- `functools: lru_cache`
- `typing: Iterable, List, Optional, Tuple`
- `torch`
- `torch.nn`
- `torch.nn.functional`
- `einops: rearrange`
- `transformers.models.glm4v.configuration_glm4v: Glm4vConfig, Glm4vVisionConfig`
- `sglang.srt.distributed: get_tensor_model_parallel_rank, get_tensor_model_parallel_world_size`
- `sglang.srt.distributed.parallel_state: get_pp_group`
- `sglang.srt.layers.activation: SiluAndMul`
- `sglang.srt.layers.attention: vision_utils`
- `sglang.srt.layers.attention.vision: VisionAttention`
- `sglang.srt.layers.conv: Conv3dLayer`
- `sglang.srt.layers.layernorm: LayerNorm, RMSNorm`
- `sglang.srt.layers.linear: MergedColumnParallelLinear, ReplicatedLinear, RowParallelLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.pooler: Pooler, PoolingType`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.rotary_embedding: get_rope`
