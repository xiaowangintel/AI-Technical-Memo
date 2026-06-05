# hunyuanvideo.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/dits/hunyuanvideo.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `HunyuanVideoArchConfig`, and `HunyuanVideoConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `HunyuanVideoArchConfig` 和 `HunyuanVideoConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-14: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field

import torch

from sglang.multimodal_gen.configs.models.dits.base import DiTArchConfig, DiTConfig
from sglang.multimodal_gen.configs.models.fsdp import (
    is_double_block,
    is_refiner_block,
    is_single_block,
    is_txt_in,
)
```
**EN:** This block establishes the module context and imports `dataclasses`, `torch`, `sglang.multimodal_gen.configs.models.dits.base`, and `sglang.multimodal_gen.configs.models.fsdp`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`torch`、`sglang.multimodal_gen.configs.models.dits.base` 和 `sglang.multimodal_gen.configs.models.fsdp`。这些依赖为后续实现提供所需符号。

### Lines 18-18: `HunyuanVideoArchConfig` class overview / `HunyuanVideoArchConfig` 类概览
```python
class HunyuanVideoArchConfig(DiTArchConfig):
```
**EN:** This block defines class `HunyuanVideoArchConfig`. It encapsulates hunyuan video arch config behavior. It inherits from `DiTArchConfig`.
**CN:** 该代码块定义了类 `HunyuanVideoArchConfig`。 它用于封装 hunyuan video arch config 相关行为。 它继承自 `DiTArchConfig`。

### Lines 19-162: supporting statements / 辅助语句
```python
    _fsdp_shard_conditions: list = field(
        default_factory=lambda: [is_double_block, is_single_block, is_refiner_block]
    )

    _compile_conditions: list = field(
        default_factory=lambda: [is_double_block, is_single_block, is_txt_in]
    )

    param_names_mapping: dict = field(
        default_factory=lambda: {
            # 1. context_embedder.time_text_embed submodules (specific rules, applied first):
            r"^context_embedder\.time_text_embed\.timestep_embedder\.linear_1\.(.*)$": r"txt_in.t_embedder.mlp.fc_in.\1",
            r"^context_embedder\.time_text_embed\.timestep_embedder\.linear_2\.(.*)$": r"txt_in.t_embedder.mlp.fc_out.\1",
            r"^context_embedder\.proj_in\.(.*)$": r"txt_in.input_embedder.\1",
            r"^context_embedder\.time_text_embed\.text_embedder\.linear_1\.(.*)$": r"txt_in.c_embedder.fc_in.\1",
            r"^context_embedder\.time_text_embed\.text_embedder\.linear_2\.(.*)$": r"txt_in.c_embedder.fc_out.\1",
            r"^context_embedder\.token_refiner\.refiner_blocks\.(\d+)\.norm1\.(.*)$": r"txt_in.refiner_blocks.\1.norm1.\2",
            r"^context_embedder\.token_refiner\.refiner_blocks\.(\d+)\.norm2\.(.*)$": r"txt_in.refiner_blocks.\1.norm2.\2",
            r"^context_embedder\.token_refiner\.refiner_blocks\.(\d+)\.attn\.to_q\.(.*)$": (
                r"txt_in.refiner_blocks.\1.self_attn_qkv.\2",
                0,
                3,
            ),
            r"^context_embedder\.token_refiner\.refiner_blocks\.(\d+)\.attn\.to_k\.(.*)$": (
                r"txt_in.refiner_blocks.\1.self_attn_qkv.\2",
                1,
                3,
            ),
            r"^context_embedder\.token_refiner\.refiner_blocks\.(\d+)\.attn\.to_v\.(.*)$": (
                r"txt_in.refiner_blocks.\1.self_attn_qkv.\2",
                2,
                3,
            ),
            r"^context_embedder\.token_refiner\.refiner_blocks\.(\d+)\.attn\.to_out\.0\.(.*)$": r"txt_in.refiner_blocks.\1.self_attn_proj.\2",
            r"^context_embedder\.token_refiner\.refiner_blocks\.(\d+)\.ff\.net\.0(?:\.proj)?\.(.*)$": r"txt_in.refiner_blocks.\1.mlp.fc_in.\2",
            r"^context_embedder\.token_refiner\.refiner_blocks\.(\d+)\.ff\.net\.2(?:\.proj)?\.(.*)$": r"txt_in.refiner_blocks.\1.mlp.fc_out.\2",
            r"^context_embedder\.token_refiner\.refiner_blocks\.(\d+)\.norm_out\.linear\.(.*)$": r"txt_in.refiner_blocks.\1.adaLN_modulation.linear.\2",
            # 3. x_embedder mapping:
            r"^x_embedder\.proj\.(.*)$": r"img_in.proj.\1",
            # 4. Top-level time_text_embed mappings:
            r"^time_text_embed\.timestep_embedder\.linear_1\.(.*)$": r"time_in.mlp.fc_in.\1",
            r"^time_text_embed\.timestep_embedder\.linear_2\.(.*)$": r"time_in.mlp.fc_out.\1",
            r"^time_text_embed\.guidance_embedder\.linear_1\.(.*)$": r"guidance_in.mlp.fc_in.\1",
            r"^time_text_embed\.guidance_embedder\.linear_2\.(.*)$": r"guidance_in.mlp.fc_out.\1",
            r"^time_text_embed\.text_embedder\.linear_1\.(.*)$": r"vector_in.fc_in.\1",
            r"^time_text_embed\.text_embedder\.linear_2\.(.*)$": r"vector_in.fc_out.\1",
            # 5. transformer_blocks mapping:
            r"^transformer_blocks\.(\d+)\.norm1\.linear\.(.*)$": r"double_blocks.\1.img_mod.linear.\2",
            r"^transformer_blocks\.(\d+)\.norm1_context\.linear\.(.*)$": r"double_blocks.\1.txt_mod.linear.\2",
            r"^transformer_blocks\.(\d+)\.attn\.norm_q\.(.*)$": r"double_blocks.\1.img_attn_q_norm.\2",
            r"^transformer_blocks\.(\d+)\.attn\.norm_k\.(.*)$": r"double_blocks.\1.img_attn_k_norm.\2",
            r"^transformer_blocks\.(\d+)\.attn\.to_q\.(.*)$": (
                r"double_blocks.\1.img_attn_qkv.\2",
                0,
                3,
            ),
            r"^transformer_blocks\.(\d+)\.attn\.to_k\.(.*)$": (
                r"double_blocks.\1.img_attn_qkv.\2",
                1,
                3,
            ),
            r"^transformer_blocks\.(\d+)\.attn\.to_v\.(.*)$": (
                r"double_blocks.\1.img_attn_qkv.\2",
                2,
                3,
            ),
            r"^transformer_blocks\.(\d+)\.attn\.add_q_proj\.(.*)$": (
                r"double_blocks.\1.txt_attn_qkv.\2",
                0,
                3,
            ),
            r"^transformer_blocks\.(\d+)\.attn\.add_k_proj\.(.*)$": (
                r"double_blocks.\1.txt_attn_qkv.\2",
                1,
                3,
            ),
            r"^transformer_blocks\.(\d+)\.attn\.add_v_proj\.(.*)$": (
                r"double_blocks.\1.txt_attn_qkv.\2",
                2,
                3,
            ),
            r"^transformer_blocks\.(\d+)\.attn\.to_out\.0\.(.*)$": r"double_blocks.\1.img_attn_proj.\2",
            # Corrected: merge attn.to_add_out into the main projection.
            r"^transformer_blocks\.(\d+)\.attn\.to_add_out\.(.*)$": r"double_blocks.\1.txt_attn_proj.\2",
            r"^transformer_blocks\.(\d+)\.attn\.norm_added_q\.(.*)$": r"double_blocks.\1.txt_attn_q_norm.\2",
            r"^transformer_blocks\.(\d+)\.attn\.norm_added_k\.(.*)$": r"double_blocks.\1.txt_attn_k_norm.\2",
            r"^transformer_blocks\.(\d+)\.ff\.net\.0(?:\.proj)?\.(.*)$": r"double_blocks.\1.img_mlp.fc_in.\2",
            r"^transformer_blocks\.(\d+)\.ff\.net\.2(?:\.proj)?\.(.*)$": r"double_blocks.\1.img_mlp.fc_out.\2",
            r"^transformer_blocks\.(\d+)\.ff_context\.net\.0(?:\.proj)?\.(.*)$": r"double_blocks.\1.txt_mlp.fc_in.\2",
            r"^transformer_blocks\.(\d+)\.ff_context\.net\.2(?:\.proj)?\.(.*)$": r"double_blocks.\1.txt_mlp.fc_out.\2",
            # 6. single_transformer_blocks mapping:
            r"^single_transformer_blocks\.(\d+)\.attn\.norm_q\.(.*)$": r"single_blocks.\1.q_norm.\2",
            r"^single_transformer_blocks\.(\d+)\.attn\.norm_k\.(.*)$": r"single_blocks.\1.k_norm.\2",
            r"^single_transformer_blocks\.(\d+)\.attn\.to_q\.(.*)$": (
                r"single_blocks.\1.linear1.\2",
                0,
                4,
            ),
            r"^single_transformer_blocks\.(\d+)\.attn\.to_k\.(.*)$": (
                r"single_blocks.\1.linear1.\2",
                1,
                4,
            ),
            r"^single_transformer_blocks\.(\d+)\.attn\.to_v\.(.*)$": (
                r"single_blocks.\1.linear1.\2",
                2,
                4,
            ),
            r"^single_transformer_blocks\.(\d+)\.proj_mlp\.(.*)$": (
                r"single_blocks.\1.linear1.\2",
                3,
                4,
            ),
            # Corrected: map proj_out to modulation.linear rather than a separate proj_out branch.
            r"^single_transformer_blocks\.(\d+)\.proj_out\.(.*)$": r"single_blocks.\1.linear2.\2",
            r"^single_transformer_blocks\.(\d+)\.norm\.linear\.(.*)$": r"single_blocks.\1.modulation.linear.\2",
            # 7. Final layers mapping:
            r"^norm_out\.linear\.(.*)$": r"final_layer.adaLN_modulation.linear.\1",
            r"^proj_out\.(.*)$": r"final_layer.linear.\1",
        }
    )

    reverse_param_names_mapping: dict = field(default_factory=lambda: {})

    patch_size: int = 2
    patch_size_t: int = 1
    in_channels: int = 16
    out_channels: int = 16
    num_attention_heads: int = 24
    attention_head_dim: int = 128
    mlp_ratio: float = 4.0
    num_layers: int = 20
    num_single_layers: int = 40
    num_refiner_layers: int = 2
    rope_axes_dim: tuple[int, int, int] = (16, 56, 56)
    guidance_embeds: bool = False
    dtype: torch.dtype | None = None
    text_embed_dim: int = 4096
    pooled_projection_dim: int = 768
    rope_theta: int = 256
    qk_norm: str = "rms_norm"
    exclude_lora_layers: list[str] = field(
        default_factory=lambda: ["img_in", "txt_in", "time_in", "vector_in"]
    )
```
**EN:** This block gathers supporting statements inside `HunyuanVideoArchConfig`. It updates names such as `_fsdp_shard_conditions`, `_compile_conditions`, `param_names_mapping`, `reverse_param_names_mapping`, `patch_size`, and `patch_size_t`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `HunyuanVideoArchConfig` 内部的辅助语句。 它会更新 `_fsdp_shard_conditions`、`_compile_conditions`、`param_names_mapping`、`reverse_param_names_mapping`、`patch_size` 和 `patch_size_t` 等名称。 代码会与 `field` 协同工作。

### Lines 164-167: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        super().__post_init__()
        self.hidden_size: int = self.attention_head_dim * self.num_attention_heads
        self.num_channels_latents: int = self.in_channels
```
**EN:** This block defines method `__post_init__` on `HunyuanVideoArchConfig`. It post-processes init. Key calls include `super.__post_init__`, and `super`.
**CN:** 该代码块定义了 `HunyuanVideoArchConfig` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `super.__post_init__` 和 `super`。

### Lines 171-171: `HunyuanVideoConfig` class overview / `HunyuanVideoConfig` 类概览
```python
class HunyuanVideoConfig(DiTConfig):
```
**EN:** This block defines class `HunyuanVideoConfig`. It encapsulates hunyuan video config behavior. It inherits from `DiTConfig`.
**CN:** 该代码块定义了类 `HunyuanVideoConfig`。 它用于封装 hunyuan video config 相关行为。 它继承自 `DiTConfig`。

### Lines 172-174: supporting statements / 辅助语句
```python
    arch_config: DiTArchConfig = field(default_factory=HunyuanVideoArchConfig)

    prefix: str = "Hunyuan"
```
**EN:** This block gathers supporting statements inside `HunyuanVideoConfig`. It updates names such as `arch_config`, and `prefix`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `HunyuanVideoConfig` 内部的辅助语句。 它会更新 `arch_config` 和 `prefix` 等名称。 代码会与 `field` 协同工作。

## Key Concepts / 关键概念
- `HunyuanVideoArchConfig`: Primary class that encapsulates hunyuan video arch config behavior. / 核心类，用于封装 hunyuan video arch config 相关行为。
- `HunyuanVideoConfig`: Primary class that encapsulates hunyuan video config behavior. / 核心类，用于封装 hunyuan video config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.dits.base`, `sglang.multimodal_gen.configs.models.fsdp`

- **Total lines / 总行数**: 174
