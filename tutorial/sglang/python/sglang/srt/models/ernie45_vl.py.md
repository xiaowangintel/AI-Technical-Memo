# ernie45_vl.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/ernie45_vl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the ernie45 Vl architecture into SGLang's serving runtime. Inference-only Ernie45-VL model compatible with HuggingFace weights. / 该模块将 ernie45 Vl 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only Ernie45-VL model compatible with HuggingFace weights。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12: Module header and imports / 模块头与导入
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
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 46-46: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 52-81: Class: Ernie4_5_VisionMLP / 类：Ernie4_5_VisionMLP
```python
class Ernie4_5_VisionMLP(nn.Module):

    def __init__(
        self,
        in_features: int,
        hidden_features: int = None,
        act_layer: Type[nn.Module] = QuickGELU,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.fc1 = ColumnParallelLinear(
            in_features,
            hidden_features,
            quant_config=quant_config,
            prefix=add_prefix("fc1", prefix),
        )
        self.act = act_layer()
        self.fc2 = RowParallelLinear(
            hidden_features,
            in_features,
            quant_config=quant_config,
            prefix=add_prefix("fc2", prefix),
        )

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x_parallel, _ = self.fc1(x)
        x_parallel = self.act(x_parallel)
        x, _ = self.fc2(x_parallel)
        return x
```
**EN:** This class defines Ernie4 5 Vision M L P inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Ernie4 5 Vision M L P，用于封装该模型组件的状态与方法。

### Lines 84-138: Class: Ernie4_5_VisionBlock / 类：Ernie4_5_VisionBlock
```python
class Ernie4_5_VisionBlock(nn.Module):

    def __init__(
        self,
        dim: int,
        num_heads: int,
        mlp_ratio: float,
        act_layer: Type[nn.Module] = QuickGELU,
        norm_layer: Type[nn.Module] = None,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        if norm_layer is None:
            norm_layer = partial(nn.LayerNorm, eps=1e-6)
        self.norm1 = norm_layer(dim)
        self.norm2 = norm_layer(dim)
        mlp_hidden_dim = int(dim * mlp_ratio)

        self.attn = VisionAttention(
            embed_dim=dim,
            num_heads=num_heads,
            projection_size=dim,
            use_qkv_parallel=True,
            flatten_batch=True,
            quant_config=quant_config,
            prefix=add_prefix("attn", prefix),
        )
        self.mlp = Ernie4_5_VisionMLP(
            dim,
            mlp_hidden_dim,
            act_layer=act_layer,
            quant_config=quant_config,
            prefix=add_prefix("mlp", prefix),
        )

    def forward(
        self,
        x: torch.Tensor,
        cu_seqlens: torch.Tensor,
# ... truncated for brevity ...
```
**EN:** This class defines Ernie4 5 Vision Block inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Ernie4 5 Vision Block，用于封装该模型组件的状态与方法。

### Lines 141-161: Class: Ernie4_5_VisionPatchEmbed / 类：Ernie4_5_VisionPatchEmbed
```python
class Ernie4_5_VisionPatchEmbed(nn.Module):

    def __init__(
        self,
        patch_size: int = 14,
        in_chans: int = 3,
        embed_dim: int = 1280,
    ) -> None:
        super().__init__()
        self.patch_size = patch_size
        self.in_channels = in_chans
        self.embed_dim = embed_dim

        self.proj = nn.Linear(in_chans * patch_size * patch_size, embed_dim, bias=False)

    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        target_dtype = self.proj.weight.dtype
        hidden_states = hidden_states.to(target_dtype)
        hidden_states = self.proj(hidden_states)

        return hidden_states
```
**EN:** This class defines Ernie4 5 Vision Patch Embed inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Ernie4 5 Vision Patch Embed，用于封装该模型组件的状态与方法。

### Lines 164-345: Class: VariableResolutionResamplerModel / 类：VariableResolutionResamplerModel
```python
class VariableResolutionResamplerModel(nn.Module):
    def __init__(
        self,
        in_dim,
        out_dim,
        spatial_conv_size,
        temporal_conv_size,
        config,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.in_dim = in_dim
        self.out_dim = out_dim
        self.config = config
        self.spatial_conv_size = spatial_conv_size
        self.temporal_conv_size = temporal_conv_size
        self.use_temporal_conv = config.use_temporal_conv

        # compress 2d conv(picture) to 1d
        self.spatial_dim = self.in_dim * self.spatial_conv_size * self.spatial_conv_size
        # compress 3d conv(video) to 1d
        self.temporal_dim = (
            self.in_dim
            * self.spatial_conv_size
            * self.spatial_conv_size
            * self.temporal_conv_size
        )

        self.spatial_linear1 = ColumnParallelLinear(
            self.spatial_dim,
            self.spatial_dim,
            bias=True,
            gather_output=True,
            quant_config=getattr(config, "quant_config", None),
            prefix=f"{prefix}.spatial_linear1",
        )

        self.spatial_gelu = nn.GELU()

        self.spatial_linear2 = ColumnParallelLinear(
# ... truncated for brevity ...
```
**EN:** This class defines Variable Resolution Resampler Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Variable Resolution Resampler Model，用于封装该模型组件的状态与方法。

### Lines 348-361: Class: Ernie4_5_VisionRotaryEmbedding / 类：Ernie4_5_VisionRotaryEmbedding
```python
class Ernie4_5_VisionRotaryEmbedding(nn.Module):

    def __init__(self, dim: int, theta: float = 10000.0) -> None:
        super().__init__()
        self.inv_freq = 1.0 / theta ** (
            torch.arange(start=0, end=dim, step=2, dtype=torch.float32) / dim
        )

    def forward(self, seqlen: int) -> torch.Tensor:
        seq = torch.arange(
            seqlen, device=self.inv_freq.device, dtype=self.inv_freq.dtype
        )
        freqs = torch.outer(input=seq, vec2=self.inv_freq)
        return freqs
```
**EN:** This class defines Ernie4 5 Vision Rotary Embedding inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Ernie4 5 Vision Rotary Embedding，用于封装该模型组件的状态与方法。

### Lines 364-500: Class: Ernie4_5_VisionTransformer / 类：Ernie4_5_VisionTransformer
```python
class Ernie4_5_VisionTransformer(nn.Module):

    def __init__(
        self,
        vision_config: PretrainedConfig,
        norm_eps: float = 1e-6,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        patch_size: int = vision_config.patch_size
        spatial_merge_size: int = vision_config.spatial_merge_size
        in_chans: int = vision_config.in_chans
        hidden_size: int = vision_config.hidden_size
        embed_dim: int = vision_config.embed_dim
        depth: int = vision_config.depth
        num_heads: int = vision_config.num_heads
        mlp_ratio: float = vision_config.mlp_ratio

        self.spatial_merge_size = spatial_merge_size

        self.patch_embed = Ernie4_5_VisionPatchEmbed(
            patch_size=patch_size,
            in_chans=in_chans,
            embed_dim=embed_dim,
        )

        norm_layer = partial(nn.LayerNorm, eps=norm_eps)
        head_dim = embed_dim // num_heads
        self.rotary_pos_emb = get_rope(
            head_size=head_dim,
            rotary_dim=head_dim // 2,
            max_position=8192,
            base=10000.0,
            is_neox_style=True,
        )
        self.blocks = nn.ModuleList(
            [
                Ernie4_5_VisionBlock(
# ... truncated for brevity ...
```
**EN:** This class defines Ernie4 5 Vision Transformer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Ernie4 5 Vision Transformer，用于封装该模型组件的状态与方法。

### Lines 503-503: Assignment: cached_get_processor / 赋值：cached_get_processor
```python
cached_get_processor = lru_cache(get_processor)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 506-865: Class: Ernie4_5_VLMoeForConditionalGeneration / 类：Ernie4_5_VLMoeForConditionalGeneration
```python
class Ernie4_5_VLMoeForConditionalGeneration(nn.Module):
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
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.config = config
        self.vision_model = Ernie4_5_VisionTransformer(
            config.vision_config,
            norm_eps=getattr(config, "rms_norm_eps", 1e-6),
            quant_config=quant_config,
            prefix=add_prefix("vision_model", prefix),
        )

        self.model = Ernie4_5_VLMoeModel(
            config, quant_config, prefix=add_prefix("model", prefix)
        )

# ... truncated for brevity ...
```
**EN:** This class defines Ernie4 5 V L Moe For Conditional Generation inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Ernie4 5 V L Moe For Conditional Generation，用于封装该模型组件的状态与方法。

### Lines 868-868: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [Ernie4_5_VLMoeForConditionalGeneration]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `logging`
- `functools: lru_cache, partial`
- `typing: Iterable, List, Optional, Tuple, Type`
- `numpy`
- `torch`
- `torch.nn`
- `torch.nn.functional`
- `einops: rearrange`
- `transformers: PretrainedConfig`
- `sglang.srt.layers.activation: QuickGELU`
- `sglang.srt.layers.attention.vision: VisionAttention`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.linear: ColumnParallelLinear, RowParallelLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.moe.fused_moe_triton.layer: FusedMoE`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.rotary_embedding: get_rope`
- `sglang.srt.layers.vocab_parallel_embedding: ParallelLMHead`
- `sglang.srt.managers.mm_utils: MultiModalityDataPaddingPatternMultimodalTokens, general_mm_embed_routine`
- `sglang.srt.managers.schedule_batch: MultimodalDataItem, MultimodalInputs`
