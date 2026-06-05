# wanvideo.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/dits/wanvideo.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `WanVideoArchConfig`, and `WanVideoConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `WanVideoArchConfig` 和 `WanVideoConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-7: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field

from sglang.multimodal_gen.configs.models.dits.base import DiTArchConfig, DiTConfig
from sglang.multimodal_gen.configs.models.fsdp import is_block
```
**EN:** This block establishes the module context and imports `dataclasses`, `sglang.multimodal_gen.configs.models.dits.base`, and `sglang.multimodal_gen.configs.models.fsdp`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`sglang.multimodal_gen.configs.models.dits.base` 和 `sglang.multimodal_gen.configs.models.fsdp`。这些依赖为后续实现提供所需符号。

### Lines 11-11: `WanVideoArchConfig` class overview / `WanVideoArchConfig` 类概览
```python
class WanVideoArchConfig(DiTArchConfig):
```
**EN:** This block defines class `WanVideoArchConfig`. It encapsulates wan video arch config behavior. It inherits from `DiTArchConfig`.
**CN:** 该代码块定义了类 `WanVideoArchConfig`。 它用于封装 wan video arch config 相关行为。 它继承自 `DiTArchConfig`。

### Lines 12-112: supporting statements / 辅助语句
```python
    _fsdp_shard_conditions: list = field(default_factory=lambda: [is_block])

    param_names_mapping: dict = field(
        default_factory=lambda: {
            r"^patch_embedding\.(.*)$": r"patch_embedding.proj.\1",
            r"^condition_embedder\.text_embedder\.linear_1\.(.*)$": r"condition_embedder.text_embedder.fc_in.\1",
            r"^condition_embedder\.text_embedder\.linear_2\.(.*)$": r"condition_embedder.text_embedder.fc_out.\1",
            r"^condition_embedder\.time_embedder\.linear_1\.(.*)$": r"condition_embedder.time_embedder.mlp.fc_in.\1",
            r"^condition_embedder\.time_embedder\.linear_2\.(.*)$": r"condition_embedder.time_embedder.mlp.fc_out.\1",
            r"^condition_embedder\.time_proj\.(.*)$": r"condition_embedder.time_modulation.linear.\1",
            r"^condition_embedder\.image_embedder\.ff\.net\.0\.proj\.(.*)$": r"condition_embedder.image_embedder.ff.fc_in.\1",
            r"^condition_embedder\.image_embedder\.ff\.net\.2\.(.*)$": r"condition_embedder.image_embedder.ff.fc_out.\1",
            r"^blocks\.(\d+)\.attn1\.to_q\.(.*)$": r"blocks.\1.to_q.\2",
            r"^blocks\.(\d+)\.attn1\.to_k\.(.*)$": r"blocks.\1.to_k.\2",
            r"^blocks\.(\d+)\.attn1\.to_v\.(.*)$": r"blocks.\1.to_v.\2",
            r"^blocks\.(\d+)\.attn1\.to_out\.0\.(.*)$": r"blocks.\1.to_out.\2",
            r"^blocks\.(\d+)\.attn1\.norm_q\.(.*)$": r"blocks.\1.norm_q.\2",
            r"^blocks\.(\d+)\.attn1\.norm_k\.(.*)$": r"blocks.\1.norm_k.\2",
            r"^blocks\.(\d+)\.attn1\.attn_op\.local_attn\.proj_l\.(.*)$": r"blocks.\1.attn1.local_attn.proj_l.\2",
            r"^blocks\.(\d+)\.attn2\.norm_added_q\.(.*)$": "",
            r"^blocks\.(\d+)\.attn2\.to_out\.0\.(.*)$": r"blocks.\1.attn2.to_out.\2",
            r"^blocks\.(\d+)\.ffn\.net\.0\.proj\.(.*)$": r"blocks.\1.ffn.fc_in.\2",
            r"^blocks\.(\d+)\.ffn\.net\.2\.(.*)$": r"blocks.\1.ffn.fc_out.\2",
            r"^blocks\.(\d+)\.norm2\.(.*)$": r"blocks.\1.self_attn_residual_norm.norm.\2",
        }
    )

    reverse_param_names_mapping: dict = field(
        default_factory=lambda: {
            r"^patch_embedding\.proj\.(.*)$": r"patch_embedding.\1",
            r"^condition_embedder\.text_embedder\.fc_in\.(.*)$": r"condition_embedder.text_embedder.linear_1.\1",
            r"^condition_embedder\.text_embedder\.fc_out\.(.*)$": r"condition_embedder.text_embedder.linear_2.\1",
            r"^condition_embedder\.time_embedder\.mlp\.fc_in\.(.*)$": r"condition_embedder.time_embedder.linear_1.\1",
            r"^condition_embedder\.time_embedder\.mlp\.fc_out\.(.*)$": r"condition_embedder.time_embedder.linear_2.\1",
            r"^condition_embedder\.time_modulation\.linear\.(.*)$": r"condition_embedder.time_proj.\1",
            r"^condition_embedder\.image_embedder\.ff\.fc_in\.(.*)$": r"condition_embedder.image_embedder.ff.net.0.proj.\1",
            r"^condition_embedder\.image_embedder\.ff\.fc_out\.(.*)$": r"condition_embedder.image_embedder.ff.net.2.\1",
            r"^blocks\.(\d+)\.to_q\.(.*)$": r"blocks.\1.attn1.to_q.\2",
            r"^blocks\.(\d+)\.to_k\.(.*)$": r"blocks.\1.attn1.to_k.\2",
            r"^blocks\.(\d+)\.to_v\.(.*)$": r"blocks.\1.attn1.to_v.\2",
            r"^blocks\.(\d+)\.to_out\.(.*)$": r"blocks.\1.attn1.to_out.0.\2",
            r"^blocks\.(\d+)\.norm_q\.(.*)$": r"blocks.\1.attn1.norm_q.\2",
            r"^blocks\.(\d+)\.norm_k\.(.*)$": r"blocks.\1.attn1.norm_k.\2",
            r"^blocks\.(\d+)\.attn1\.local_attn\.proj_l\.(.*)$": r"blocks.\1.attn1.attn_op.local_attn.proj_l.\2",
            r"^blocks\.(\d+)\.attn2\.to_out\.(.*)$": r"blocks.\1.attn2.to_out.0.\2",
            r"^blocks\.(\d+)\.ffn\.fc_in\.(.*)$": r"blocks.\1.ffn.net.0.proj.\2",
            r"^blocks\.(\d+)\.ffn\.fc_out\.(.*)$": r"blocks.\1.ffn.net.2.\2",
            r"^blocks\.(\d+)\.self_attn_residual_norm\.norm\.(.*)$": r"blocks.\1.norm2.\2",
        }
    )

    # Some LoRA adapters use the original official layer names instead of hf layer names,
    # so apply this before the param_names_mapping
    lora_param_names_mapping: dict = field(
        default_factory=lambda: {
            r"^blocks\.(\d+)\.self_attn\.q\.(.*)$": r"blocks.\1.attn1.to_q.\2",
            r"^blocks\.(\d+)\.self_attn\.k\.(.*)$": r"blocks.\1.attn1.to_k.\2",
            r"^blocks\.(\d+)\.self_attn\.v\.(.*)$": r"blocks.\1.attn1.to_v.\2",
            r"^blocks\.(\d+)\.self_attn\.o\.(.*)$": r"blocks.\1.attn1.to_out.0.\2",
            r"^blocks\.(\d+)\.cross_attn\.q\.(.*)$": r"blocks.\1.attn2.to_q.\2",
            r"^blocks\.(\d+)\.cross_attn\.k\.(.*)$": r"blocks.\1.attn2.to_k.\2",
            r"^blocks\.(\d+)\.cross_attn\.v\.(.*)$": r"blocks.\1.attn2.to_v.\2",
            r"^blocks\.(\d+)\.cross_attn\.o\.(.*)$": r"blocks.\1.attn2.to_out.0.\2",
            r"^blocks\.(\d+)\.ffn\.0\.(.*)$": r"blocks.\1.ffn.fc_in.\2",
            r"^blocks\.(\d+)\.ffn\.2\.(.*)$": r"blocks.\1.ffn.fc_out.\2",
        }
    )

    patch_size: tuple[int, int, int] = (1, 2, 2)
    text_len = 512
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
    image_dim: int | None = None
    added_kv_proj_dim: int | None = None
    rope_max_seq_len: int = 1024
    pos_embed_seq_len: int | None = None
    exclude_lora_layers: list[str] = field(default_factory=lambda: ["embedder"])

    # Wan MoE
    boundary_ratio: float | None = None

    # Causal Wan
    local_attn_size: int = (
        -1
    )  # Window size for temporal local attention (-1 indicates global attention)
    sink_size: int = (
        0  # Size of the attention sink, we keep the first `sink_size` frames unchanged when rolling the KV cache
    )
    num_frames_per_block: int = 3
    sliding_window_num_frames: int = 21
    attention_type: str = "original"
    sla_topk: float = 0.1
```
**EN:** This block gathers supporting statements inside `WanVideoArchConfig`. It updates names such as `_fsdp_shard_conditions`, `param_names_mapping`, `reverse_param_names_mapping`, `lora_param_names_mapping`, `patch_size`, and `text_len`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `WanVideoArchConfig` 内部的辅助语句。 它会更新 `_fsdp_shard_conditions`、`param_names_mapping`、`reverse_param_names_mapping`、`lora_param_names_mapping`、`patch_size` 和 `text_len` 等名称。 代码会与 `field` 协同工作。

### Lines 114-118: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        super().__post_init__()
        self.out_channels = self.out_channels or self.in_channels
        self.hidden_size = self.num_attention_heads * self.attention_head_dim
        self.num_channels_latents = self.out_channels
```
**EN:** This block defines method `__post_init__` on `WanVideoArchConfig`. It post-processes init. Key calls include `super.__post_init__`, and `super`.
**CN:** 该代码块定义了 `WanVideoArchConfig` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `super.__post_init__` 和 `super`。

### Lines 122-122: `WanVideoConfig` class overview / `WanVideoConfig` 类概览
```python
class WanVideoConfig(DiTConfig):
```
**EN:** This block defines class `WanVideoConfig`. It encapsulates wan video config behavior. It inherits from `DiTConfig`.
**CN:** 该代码块定义了类 `WanVideoConfig`。 它用于封装 wan video config 相关行为。 它继承自 `DiTConfig`。

### Lines 123-125: supporting statements / 辅助语句
```python
    arch_config: DiTArchConfig = field(default_factory=WanVideoArchConfig)

    prefix: str = "Wan"
```
**EN:** This block gathers supporting statements inside `WanVideoConfig`. It updates names such as `arch_config`, and `prefix`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `WanVideoConfig` 内部的辅助语句。 它会更新 `arch_config` 和 `prefix` 等名称。 代码会与 `field` 协同工作。

## Key Concepts / 关键概念
- `WanVideoArchConfig`: Primary class that encapsulates wan video arch config behavior. / 核心类，用于封装 wan video arch config 相关行为。
- `WanVideoConfig`: Primary class that encapsulates wan video config behavior. / 核心类，用于封装 wan video config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.dits.base`, `sglang.multimodal_gen.configs.models.fsdp`

- **Total lines / 总行数**: 125
