# helios.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/dits/helios.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `HeliosArchConfig`, and `HeliosConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `HeliosArchConfig` 和 `HeliosConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 2-5: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field

from sglang.multimodal_gen.configs.models.dits.base import DiTArchConfig, DiTConfig
from sglang.multimodal_gen.configs.models.fsdp import is_block
```
**EN:** This block establishes the module context and imports `dataclasses`, `sglang.multimodal_gen.configs.models.dits.base`, and `sglang.multimodal_gen.configs.models.fsdp`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`sglang.multimodal_gen.configs.models.dits.base` 和 `sglang.multimodal_gen.configs.models.fsdp`。这些依赖为后续实现提供所需符号。

### Lines 9-9: `HeliosArchConfig` class overview / `HeliosArchConfig` 类概览
```python
class HeliosArchConfig(DiTArchConfig):
```
**EN:** This block defines class `HeliosArchConfig`. It encapsulates helios arch config behavior. It inherits from `DiTArchConfig`.
**CN:** 该代码块定义了类 `HeliosArchConfig`。 它用于封装 helios arch config 相关行为。 它继承自 `DiTArchConfig`。

### Lines 10-64: supporting statements / 辅助语句
```python
    _fsdp_shard_conditions: list = field(default_factory=lambda: [is_block])

    param_names_mapping: dict = field(
        default_factory=lambda: {
            # Patch embeddings
            r"^patch_embedding\.(.*)$": r"patch_embedding.proj.\1",
            # Condition embedder: text
            r"^condition_embedder\.text_embedder\.linear_1\.(.*)$": r"condition_embedder.text_embedder.fc_in.\1",
            r"^condition_embedder\.text_embedder\.linear_2\.(.*)$": r"condition_embedder.text_embedder.fc_out.\1",
            # Condition embedder: time
            r"^condition_embedder\.time_embedder\.linear_1\.(.*)$": r"condition_embedder.time_embedder.mlp.fc_in.\1",
            r"^condition_embedder\.time_embedder\.linear_2\.(.*)$": r"condition_embedder.time_embedder.mlp.fc_out.\1",
            r"^condition_embedder\.time_proj\.(.*)$": r"condition_embedder.time_modulation.linear.\1",
            # Blocks: self-attention (keep attn1. prefix, drop .0. from to_out)
            r"^blocks\.(\d+)\.attn1\.to_out\.0\.(.*)$": r"blocks.\1.attn1.to_out.\2",
            # Blocks: cross-attention output (drop .0. from to_out)
            r"^blocks\.(\d+)\.attn2\.to_out\.0\.(.*)$": r"blocks.\1.attn2.to_out.\2",
            # Blocks: feed-forward
            r"^blocks\.(\d+)\.ffn\.net\.0\.proj\.(.*)$": r"blocks.\1.ffn.fc_in.\2",
            r"^blocks\.(\d+)\.ffn\.net\.2\.(.*)$": r"blocks.\1.ffn.fc_out.\2",
            # Blocks: cross-attn residual norm
            r"^blocks\.(\d+)\.norm2\.(.*)$": r"blocks.\1.self_attn_residual_norm.\2",
        }
    )

    reverse_param_names_mapping: dict = field(default_factory=lambda: {})

    lora_param_names_mapping: dict = field(default_factory=lambda: {})

    patch_size: tuple[int, int, int] = (1, 2, 2)
    text_len: int = 226
    num_attention_heads: int = 40
    attention_head_dim: int = 128
    in_channels: int = 16
    out_channels: int = 16
    text_dim: int = 4096
    freq_dim: int = 256
    ffn_dim: int = 13824
    num_layers: int = 40
    cross_attn_norm: bool = True
    qk_norm: str = "rms_norm_across_heads"
    eps: float = 1e-6
    added_kv_proj_dim: int | None = None
    rope_max_seq_len: int = 1024
    pos_embed_seq_len: int | None = None
    exclude_lora_layers: list[str] = field(default_factory=lambda: ["embedder"])

    # Helios-specific
    rope_dim: tuple[int, int, int] = (44, 42, 42)
    rope_theta: float = 10000.0
    guidance_cross_attn: bool = True
    zero_history_timestep: bool = True
    has_multi_term_memory_patch: bool = True
    is_amplify_history: bool = False
    history_scale_mode: str = "per_head"
```
**EN:** This block gathers supporting statements inside `HeliosArchConfig`. It updates names such as `_fsdp_shard_conditions`, `param_names_mapping`, `reverse_param_names_mapping`, `lora_param_names_mapping`, `patch_size`, and `text_len`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `HeliosArchConfig` 内部的辅助语句。 它会更新 `_fsdp_shard_conditions`、`param_names_mapping`、`reverse_param_names_mapping`、`lora_param_names_mapping`、`patch_size` 和 `text_len` 等名称。 代码会与 `field` 协同工作。

### Lines 66-70: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        super().__post_init__()
        self.out_channels = self.out_channels or self.in_channels
        self.hidden_size = self.num_attention_heads * self.attention_head_dim
        self.num_channels_latents = self.out_channels
```
**EN:** This block defines method `__post_init__` on `HeliosArchConfig`. It post-processes init. Key calls include `super.__post_init__`, and `super`.
**CN:** 该代码块定义了 `HeliosArchConfig` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `super.__post_init__` 和 `super`。

### Lines 74-74: `HeliosConfig` class overview / `HeliosConfig` 类概览
```python
class HeliosConfig(DiTConfig):
```
**EN:** This block defines class `HeliosConfig`. It encapsulates helios config behavior. It inherits from `DiTConfig`.
**CN:** 该代码块定义了类 `HeliosConfig`。 它用于封装 helios config 相关行为。 它继承自 `DiTConfig`。

### Lines 75-77: supporting statements / 辅助语句
```python
    arch_config: DiTArchConfig = field(default_factory=HeliosArchConfig)

    prefix: str = "Helios"
```
**EN:** This block gathers supporting statements inside `HeliosConfig`. It updates names such as `arch_config`, and `prefix`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `HeliosConfig` 内部的辅助语句。 它会更新 `arch_config` 和 `prefix` 等名称。 代码会与 `field` 协同工作。

## Key Concepts / 关键概念
- `HeliosArchConfig`: Primary class that encapsulates helios arch config behavior. / 核心类，用于封装 helios arch config 相关行为。
- `HeliosConfig`: Primary class that encapsulates helios config behavior. / 核心类，用于封装 helios config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.dits.base`, `sglang.multimodal_gen.configs.models.fsdp`

- **Total lines / 总行数**: 77
