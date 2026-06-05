# flux.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/dits/flux.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `FluxArchConfig`, and `FluxConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `FluxArchConfig` 和 `FluxConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-7: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field
from typing import Tuple

from sglang.multimodal_gen.configs.models.dits.base import DiTArchConfig, DiTConfig
```
**EN:** This block establishes the module context and imports `dataclasses`, `typing`, and `sglang.multimodal_gen.configs.models.dits.base`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`typing` 和 `sglang.multimodal_gen.configs.models.dits.base`。这些依赖为后续实现提供所需符号。

### Lines 11-11: `FluxArchConfig` class overview / `FluxArchConfig` 类概览
```python
class FluxArchConfig(DiTArchConfig):
```
**EN:** This block defines class `FluxArchConfig`. It encapsulates flux arch config behavior. It inherits from `DiTArchConfig`.
**CN:** 该代码块定义了类 `FluxArchConfig`。 它用于封装 flux arch config 相关行为。 它继承自 `DiTArchConfig`。

### Lines 12-102: supporting statements / 辅助语句
```python
    patch_size: int = 1
    in_channels: int = 64
    out_channels: int | None = None
    num_layers: int = 19
    num_single_layers: int = 38
    attention_head_dim: int = 128
    num_attention_heads: int = 24
    joint_attention_dim: int = 4096
    pooled_projection_dim: int = 768
    guidance_embeds: bool = True
    axes_dims_rope: Tuple[int, int, int] = (16, 56, 56)

    stacked_params_mapping: list[tuple[str, str, str]] = field(default_factory=list)

    exclude_lora_layers: list[str] = field(
        default_factory=lambda: [
            "time_guidance_embed.timestep_embedder.linear_1",
            "time_guidance_embed.timestep_embedder.linear_2",
            "time_guidance_embed.guidance_embedder.linear_1",
            "time_guidance_embed.guidance_embedder.linear_2",
        ]
    )

    # nunchaku checkpoint uses different weight names; map to sglang flux layout
    param_names_mapping: dict = field(
        default_factory=lambda: {
            # HF diffusers format: strip leading "transformer." prefix
            r"^transformer\.(\w*)\.(.*)$": r"\1.\2",
            # FLUX2-nvfp4 format: double blocks - image attention QKV (packed, fused)
            r"^double_blocks\.(\d+)\.img_attn\.qkv\.(.*)$": r"transformer_blocks.\1.attn.to_qkv.\2",
            r"^double_blocks\.(\d+)\.img_attn\.proj\.(.*)$": r"transformer_blocks.\1.attn.to_out.0.\2",
            r"^double_blocks\.(\d+)\.img_attn\.norm\.query_norm\.(.*)$": r"transformer_blocks.\1.attn.norm_q.\2",
            r"^double_blocks\.(\d+)\.img_attn\.norm\.key_norm\.(.*)$": r"transformer_blocks.\1.attn.norm_k.\2",
            # FLUX2-nvfp4 format: double blocks - text/context attention QKV (packed, fused)
            r"^double_blocks\.(\d+)\.txt_attn\.qkv\.(.*)$": r"transformer_blocks.\1.attn.to_added_qkv.\2",
            r"^double_blocks\.(\d+)\.txt_attn\.proj\.(.*)$": r"transformer_blocks.\1.attn.to_add_out.\2",
            r"^double_blocks\.(\d+)\.txt_attn\.norm\.query_norm\.(.*)$": r"transformer_blocks.\1.attn.norm_added_q.\2",
            r"^double_blocks\.(\d+)\.txt_attn\.norm\.key_norm\.(.*)$": r"transformer_blocks.\1.attn.norm_added_k.\2",
            # FLUX2-nvfp4  format: double blocks - image MLP
            r"^double_blocks\.(\d+)\.img_mlp\.0\.(.*)$": r"transformer_blocks.\1.ff.linear_in.\2",
            r"^double_blocks\.(\d+)\.img_mlp\.2\.(.*)$": r"transformer_blocks.\1.ff.linear_out.\2",
            # FLUX2-nvfp4  format: double blocks - text/context MLP
            r"^double_blocks\.(\d+)\.txt_mlp\.0\.(.*)$": r"transformer_blocks.\1.ff_context.linear_in.\2",
            r"^double_blocks\.(\d+)\.txt_mlp\.2\.(.*)$": r"transformer_blocks.\1.ff_context.linear_out.\2",
            # FLUX2-nvfp4  format: single blocks
            r"^single_blocks\.(\d+)\.linear1\.(.*)$": r"single_transformer_blocks.\1.attn.to_qkv_mlp_proj.\2",
            r"^single_blocks\.(\d+)\.linear2\.(.*)$": r"single_transformer_blocks.\1.attn.to_out.\2",
            r"^single_blocks\.(\d+)\.norm\.query_norm\.(.*)$": r"single_transformer_blocks.\1.attn.norm_q.\2",
            r"^single_blocks\.(\d+)\.norm\.key_norm\.(.*)$": r"single_transformer_blocks.\1.attn.norm_k.\2",
            # FLUX2-nvfp4  format: non-block input/output projections
            r"^img_in\.(.*)$": r"x_embedder.\1",
            r"^txt_in\.(.*)$": r"context_embedder.\1",
            r"^time_in\.in_layer\.(.*)$": r"time_guidance_embed.timestep_embedder.linear_1.\1",
            r"^time_in\.out_layer\.(.*)$": r"time_guidance_embed.timestep_embedder.linear_2.\1",
            r"^guidance_in\.in_layer\.(.*)$": r"time_guidance_embed.guidance_embedder.linear_1.\1",
            r"^guidance_in\.out_layer\.(.*)$": r"time_guidance_embed.guidance_embedder.linear_2.\1",
            r"^double_stream_modulation_img\.lin\.(.*)$": r"double_stream_modulation_img.linear.\1",
            r"^double_stream_modulation_txt\.lin\.(.*)$": r"double_stream_modulation_txt.linear.\1",
            r"^single_stream_modulation\.lin\.(.*)$": r"single_stream_modulation.linear.\1",
            r"^final_layer\.adaLN_modulation\.1\.(.*)$": r"norm_out.linear.\1",
            r"^final_layer\.linear\.(.*)$": r"proj_out.\1",
            # FLUX2-nvfp4 format: RMSNorm uses "scale" parameter; rename to "weight" (model uses .weight)
            r"^(.*)\.scale$": r"\1.weight",
            # transformer_blocks nunchaku format (raw export - before internal conversion)
            r"^transformer_blocks\.(\d+)\.mlp_fc1\.(.*)$": r"transformer_blocks.\1.ff.net.0.proj.\2",
            r"^transformer_blocks\.(\d+)\.mlp_fc2\.(.*)$": r"transformer_blocks.\1.ff.net.2.\2",
            r"^transformer_blocks\.(\d+)\.mlp_context_fc1\.(.*)$": r"transformer_blocks.\1.ff_context.net.0.proj.\2",
            r"^transformer_blocks\.(\d+)\.mlp_context_fc2\.(.*)$": r"transformer_blocks.\1.ff_context.net.2.\2",
            # nunchaku packed QKV → fused to_qkv / to_added_qkv (matches use_fused_qkv in model)
            r"^transformer_blocks\.(\d+)\.qkv_proj\.(.*)$": r"transformer_blocks.\1.attn.to_qkv.\2",
            r"^transformer_blocks\.(\d+)\.qkv_proj_context\.(.*)$": r"transformer_blocks.\1.attn.to_added_qkv.\2",
            r"^transformer_blocks\.(\d+)\.out_proj\.(.*)$": r"transformer_blocks.\1.attn.to_out.0.\2",
            r"^transformer_blocks\.(\d+)\.out_proj_context\.(.*)$": r"transformer_blocks.\1.attn.to_add_out.\2",
            r"^transformer_blocks\.(\d+)\.norm_q\.(.*)$": r"transformer_blocks.\1.attn.norm_q.\2",
            r"^transformer_blocks\.(\d+)\.norm_k\.(.*)$": r"transformer_blocks.\1.attn.norm_k.\2",
            r"^transformer_blocks\.(\d+)\.norm_added_q\.(.*)$": r"transformer_blocks.\1.attn.norm_added_q.\2",
            r"^transformer_blocks\.(\d+)\.norm_added_k\.(.*)$": r"transformer_blocks.\1.attn.norm_added_k.\2",
            # nunchaku format (already converted): add_qkv_proj → fused to_added_qkv
            r"^transformer_blocks\.(\d+)\.attn\.add_qkv_proj\.(.*)$": r"transformer_blocks.\1.attn.to_added_qkv.\2",
            # single_transformer_blocks nunchaku format (raw export - before internal conversion)
            r"^single_transformer_blocks\.(\d+)\.qkv_proj\.(.*)$": r"single_transformer_blocks.\1.attn.to_qkv_mlp_proj.\2",
            r"^single_transformer_blocks\.(\d+)\.out_proj\.(.*)$": r"single_transformer_blocks.\1.attn.to_out.\2",
            r"^single_transformer_blocks\.(\d+)\.norm_q\.(.*)$": r"single_transformer_blocks.\1.attn.norm_q.\2",
            r"^single_transformer_blocks\.(\d+)\.norm_k\.(.*)$": r"single_transformer_blocks.\1.attn.norm_k.\2",
            # nunchaku quantization parameter name conversions (apply to all blocks)
            r"^(.*)\.smooth_orig$": r"\1.smooth_factor_orig",
            r"^(.*)\.smooth$": r"\1.smooth_factor",
            r"^(.*)\.lora_down$": r"\1.proj_down",
            r"^(.*)\.lora_up$": r"\1.proj_up",
        }
    )
```
**EN:** This block gathers supporting statements inside `FluxArchConfig`. It updates names such as `patch_size`, `in_channels`, `out_channels`, `num_layers`, `num_single_layers`, and `attention_head_dim`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `FluxArchConfig` 内部的辅助语句。 它会更新 `patch_size`、`in_channels`、`out_channels`、`num_layers`、`num_single_layers` 和 `attention_head_dim` 等名称。 代码会与 `field` 协同工作。

### Lines 104-108: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        super().__post_init__()
        self.out_channels = self.out_channels or self.in_channels
        self.hidden_size = self.num_attention_heads * self.attention_head_dim
        self.num_channels_latents = self.out_channels
```
**EN:** This block defines method `__post_init__` on `FluxArchConfig`. It post-processes init. Key calls include `super.__post_init__`, and `super`.
**CN:** 该代码块定义了 `FluxArchConfig` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `super.__post_init__` 和 `super`。

### Lines 112-113: `FluxConfig` class overview / `FluxConfig` 类概览
```python
class FluxConfig(DiTConfig):
```
**EN:** This block defines class `FluxConfig`. It encapsulates flux config behavior. It inherits from `DiTConfig`.
**CN:** 该代码块定义了类 `FluxConfig`。 它用于封装 flux config 相关行为。 它继承自 `DiTConfig`。

### Lines 114-116: supporting statements / 辅助语句
```python
    arch_config: DiTArchConfig = field(default_factory=FluxArchConfig)

    prefix: str = "Flux"
```
**EN:** This block gathers supporting statements inside `FluxConfig`. It updates names such as `arch_config`, and `prefix`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `FluxConfig` 内部的辅助语句。 它会更新 `arch_config` 和 `prefix` 等名称。 代码会与 `field` 协同工作。

## Key Concepts / 关键概念
- `FluxArchConfig`: Primary class that encapsulates flux arch config behavior. / 核心类，用于封装 flux arch config 相关行为。
- `FluxConfig`: Primary class that encapsulates flux config behavior. / 核心类，用于封装 flux config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.dits.base`

- **Total lines / 总行数**: 116
