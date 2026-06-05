# ltx_2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/dits/ltx_2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `LTXModelType`, `LTX2RopeType`, and `LTX2AttentionFunction`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `LTXModelType`、`LTX2RopeType` 和 `LTX2AttentionFunction` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 2-6: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field
from enum import Enum

from sglang.multimodal_gen.configs.models.dits.base import DiTArchConfig, DiTConfig
from sglang.multimodal_gen.configs.models.fsdp import is_blocks_or_transformer_blocks
```
**EN:** This block establishes the module context and imports `dataclasses`, `enum`, `sglang.multimodal_gen.configs.models.dits.base`, and `sglang.multimodal_gen.configs.models.fsdp`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`enum`、`sglang.multimodal_gen.configs.models.dits.base` 和 `sglang.multimodal_gen.configs.models.fsdp`。这些依赖为后续实现提供所需符号。

### Lines 9-16: `LTXModelType` class overview / `LTXModelType` 类概览
```python
class LTXModelType(Enum):
    """
    Model type enum mirroring upstream `LTXModelType`.

    Upstream reference:
      - `LTX-2/packages/ltx-core/src/ltx_core/model/transformer/model.py::LTXModelType`
    """
```
**EN:** This block defines class `LTXModelType`. Model type enum mirroring upstream `LTXModelType`. Upstream reference: - `LTX-2/packages/ltx-core/src/ltx_core/model/transformer/model.py::LTXModelType` It inherits from `Enum`.
**CN:** 该代码块定义了类 `LTXModelType`。 它用于封装 ltxmodel type 相关行为。 它继承自 `Enum`。

### Lines 17-19: supporting statements / 辅助语句
```python
    AudioVideo = "ltx av model"
    VideoOnly = "ltx video only model"
    AudioOnly = "ltx audio only model"
```
**EN:** This block gathers supporting statements inside `LTXModelType`. It updates names such as `AudioVideo`, `VideoOnly`, and `AudioOnly`.
**CN:** 该代码块汇集了位于 `LTXModelType` 内部的辅助语句。 它会更新 `AudioVideo`、`VideoOnly` 和 `AudioOnly` 等名称。

### Lines 21-22: `is_video_enabled` implementation / `is_video_enabled` 实现
```python
    def is_video_enabled(self) -> bool:
        return self in (LTXModelType.AudioVideo, LTXModelType.VideoOnly)
```
**EN:** This block defines method `is_video_enabled` on `LTXModelType`. It handles is video enabled logic.
**CN:** 该代码块定义了 `LTXModelType` 的方法 `is_video_enabled`。 它用于处理 is video enabled 相关逻辑。

### Lines 24-25: `is_audio_enabled` implementation / `is_audio_enabled` 实现
```python
    def is_audio_enabled(self) -> bool:
        return self in (LTXModelType.AudioVideo, LTXModelType.AudioOnly)
```
**EN:** This block defines method `is_audio_enabled` on `LTXModelType`. It handles is audio enabled logic.
**CN:** 该代码块定义了 `LTXModelType` 的方法 `is_audio_enabled`。 它用于处理 is audio enabled 相关逻辑。

### Lines 28-35: `LTX2RopeType` class overview / `LTX2RopeType` 类概览
```python
class LTX2RopeType(str, Enum):
    """
    Minimal RoPE type enum mirroring LTX-2 upstream `LTXRopeType`.

    Upstream reference:
      - `LTX-2/packages/ltx-core/src/ltx_core/model/transformer/rope.py::LTXRopeType`
    """
```
**EN:** This block defines class `LTX2RopeType`. Minimal RoPE type enum mirroring LTX-2 upstream `LTXRopeType`. Upstream reference: - `LTX-2/packages/ltx-core/src/ltx_core/model/transformer/rope.py::LTXRopeType` It inherits from `str`, and `Enum`.
**CN:** 该代码块定义了类 `LTX2RopeType`。 它用于封装 ltx2 rope type 相关行为。 它继承自 `str` 和 `Enum`。

### Lines 36-37: supporting statements / 辅助语句
```python
    INTERLEAVED = "interleaved"
    SPLIT = "split"
```
**EN:** This block gathers supporting statements inside `LTX2RopeType`. It updates names such as `INTERLEAVED`, and `SPLIT`.
**CN:** 该代码块汇集了位于 `LTX2RopeType` 内部的辅助语句。 它会更新 `INTERLEAVED` 和 `SPLIT` 等名称。

### Lines 40-47: `LTX2AttentionFunction` class overview / `LTX2AttentionFunction` 类概览
```python
class LTX2AttentionFunction(str, Enum):
    """
    Placeholder enum for upstream `AttentionFunction.DEFAULT`.

    Upstream reference:
      - `LTX-2/packages/ltx-core/src/ltx_core/model/transformer/attention.py`
    """
```
**EN:** This block defines class `LTX2AttentionFunction`. Placeholder enum for upstream `AttentionFunction.DEFAULT`. Upstream reference: - `LTX-2/packages/ltx-core/src/ltx_core/model/transformer/attention.py` It inherits from `str`, and `Enum`.
**CN:** 该代码块定义了类 `LTX2AttentionFunction`。 它用于封装 ltx2 attention function 相关行为。 它继承自 `str` 和 `Enum`。

### Lines 48-48: supporting statements / 辅助语句
```python
    DEFAULT = "default"
```
**EN:** This block gathers supporting statements inside `LTX2AttentionFunction`. It updates names such as `DEFAULT`.
**CN:** 该代码块汇集了位于 `LTX2AttentionFunction` 内部的辅助语句。 它会更新 `DEFAULT` 等名称。

### Lines 52-54: `LTX2ArchConfig` class overview / `LTX2ArchConfig` 类概览
```python
class LTX2ArchConfig(DiTArchConfig):
    """Architecture configuration for LTX-2 Video Transformer."""
```
**EN:** This block defines class `LTX2ArchConfig`. Architecture configuration for LTX-2 Video Transformer. It inherits from `DiTArchConfig`.
**CN:** 该代码块定义了类 `LTX2ArchConfig`。 它用于封装 ltx2 arch config 相关行为。 它继承自 `DiTArchConfig`。

### Lines 55-164: supporting statements / 辅助语句
```python
    _fsdp_shard_conditions: list = field(
        default_factory=lambda: [is_blocks_or_transformer_blocks]
    )

    param_names_mapping: dict = field(
        default_factory=lambda: {
            # Parameter name mappings from HuggingFace checkpoint keys to SGLang module names.
            # We use upstream variable names (patchify_proj, adaln_single) but HF uses different keys.
            #
            # HF key -> SGLang key (upstream naming)
            r"^model\.diffusion_model\.(.*)$": r"\1",
            r"^proj_in\.(.*)$": r"patchify_proj.\1",
            r"^time_embed\.(.*)$": r"adaln_single.\1",
            r"^audio_proj_in\.(.*)$": r"audio_patchify_proj.\1",
            r"^audio_time_embed\.(.*)$": r"audio_adaln_single.\1",
            # FeedForward
            r"(.*)ff\.net\.0\.proj\.(.*)$": r"\1ff.proj_in.\2",
            r"(.*)ff\.net\.2\.(.*)$": r"\1ff.proj_out.\2",
            # Attention Norms
            r"(.*)\.norm_q\.(.*)$": r"\1.q_norm.\2",
            r"(.*)\.norm_k\.(.*)$": r"\1.k_norm.\2",
            # Scale Shift Tables (Global)
            r"^av_cross_attn_video_scale_shift\.(.*)$": r"av_ca_video_scale_shift_adaln_single.\1",
            r"^av_cross_attn_audio_scale_shift\.(.*)$": r"av_ca_audio_scale_shift_adaln_single.\1",
            r"^av_cross_attn_video_a2v_gate\.(.*)$": r"av_ca_a2v_gate_adaln_single.\1",
            r"^av_cross_attn_audio_v2a_gate\.(.*)$": r"av_ca_v2a_gate_adaln_single.\1",
            # Scale Shift Tables (Block Level)
            # HF: scale_shift_table_a2v_ca_video -> SGLang: video_a2v_cross_attn_scale_shift_table
            r"(.*)scale_shift_table_a2v_ca_video": r"\1video_a2v_cross_attn_scale_shift_table",
            r"(.*)scale_shift_table_a2v_ca_audio": r"\1audio_a2v_cross_attn_scale_shift_table",
        }
    )

    reverse_param_names_mapping: dict = field(
        default_factory=lambda: {
            # Reverse mapping: SGLang module names -> HF checkpoint keys (for saving).
            r"^patchify_proj\.(.*)$": r"proj_in.\1",
            r"^adaln_single\.(.*)$": r"time_embed.\1",
            r"^audio_patchify_proj\.(.*)$": r"audio_proj_in.\1",
            r"^audio_adaln_single\.(.*)$": r"audio_time_embed.\1",
            # FeedForward
            r"(.*)ff\.proj_in\.(.*)$": r"\1ff.net.0.proj.\2",
            r"(.*)ff\.proj_out\.(.*)$": r"\1ff.net.2.\2",
            # Attention Norms
            r"(.*)\.q_norm\.(.*)$": r"\1.norm_q.\2",
            r"(.*)\.k_norm\.(.*)$": r"\1.norm_k.\2",
            # Scale Shift Tables (Global)
            r"^av_ca_video_scale_shift_adaln_single\.(.*)$": r"av_cross_attn_video_scale_shift.\1",
            r"^av_ca_audio_scale_shift_adaln_single\.(.*)$": r"av_cross_attn_audio_scale_shift.\1",
            r"^av_ca_a2v_gate_adaln_single\.(.*)$": r"av_cross_attn_video_a2v_gate.\1",
            r"^av_ca_v2a_gate_adaln_single\.(.*)$": r"av_cross_attn_audio_v2a_gate.\1",
            # Scale Shift Tables (Block Level)
            # SGLang: video_a2v_cross_attn_scale_shift_table -> HF: scale_shift_table_a2v_ca_video
            r"(.*)video_a2v_cross_attn_scale_shift_table": r"\1scale_shift_table_a2v_ca_video",
            r"(.*)audio_a2v_cross_attn_scale_shift_table": r"\1scale_shift_table_a2v_ca_audio",
        }
    )

    lora_param_names_mapping: dict = field(
        default_factory=lambda: {
            # LoRA parameter name mappings from official repo format to HF format.
            # This is applied before param_names_mapping when loading LoRA adapters.
            # Will be populated if LoRA adapters use different naming conventions.
        }
    )

    # Model type and attention configuration
    model_type: LTXModelType = LTXModelType.AudioVideo
    attention_type: LTX2AttentionFunction = LTX2AttentionFunction.DEFAULT
    rope_type: LTX2RopeType = LTX2RopeType.INTERLEAVED
    double_precision_rope: bool = False
    quantize_video_rope_coords_to_hidden_dtype: bool = False
    apply_gated_attention: bool = False
    cross_attention_adaln: bool = False
    caption_proj_before_connector: bool = False

    # Video parameters
    num_attention_heads: int = 32
    attention_head_dim: int = 128
    in_channels: int = 128
    out_channels: int = 128
    num_layers: int = 48
    cross_attention_dim: int = 4096
    norm_eps: float = 1e-6
    caption_channels: int = 3840
    positional_embedding_theta: float = 10000.0
    positional_embedding_max_pos: list[int] | None = None
    timestep_scale_multiplier: int = 1000
    use_middle_indices_grid: bool = True

    # Audio parameters
    audio_num_attention_heads: int = 32
    audio_attention_head_dim: int = 64
    audio_in_channels: int = 128
    audio_out_channels: int = 128
    audio_cross_attention_dim: int = 2048
    audio_positional_embedding_max_pos: list[int] | None = None
    av_ca_timestep_scale_multiplier: int = 1

    # 2.3 connector-related fields may show up in transformer/config.json.
    connector_attention_head_dim: int = 128
    connector_num_attention_heads: int = 30
    connector_num_layers: int = 2
    audio_connector_attention_head_dim: int = 128
    audio_connector_num_attention_heads: int = 30
    audio_connector_num_layers: int = 2

    # SGLang-specific parameters
    patch_size: tuple[int, int, int] = (1, 2, 2)
    text_len: int = 512
```
**EN:** This block gathers supporting statements inside `LTX2ArchConfig`. It updates names such as `_fsdp_shard_conditions`, `param_names_mapping`, `reverse_param_names_mapping`, `lora_param_names_mapping`, `model_type`, and `attention_type`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `LTX2ArchConfig` 内部的辅助语句。 它会更新 `_fsdp_shard_conditions`、`param_names_mapping`、`reverse_param_names_mapping`、`lora_param_names_mapping`、`model_type` 和 `attention_type` 等名称。 代码会与 `field` 协同工作。

### Lines 166-179: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        super().__post_init__()
        # Video derived values
        self.hidden_size = self.num_attention_heads * self.attention_head_dim
        self.num_channels_latents = self.out_channels
        if self.positional_embedding_max_pos is None:
            self.positional_embedding_max_pos = [20, 2048, 2048]

        # Audio derived values
        self.audio_hidden_size = (
            self.audio_num_attention_heads * self.audio_attention_head_dim
        )
        if self.audio_positional_embedding_max_pos is None:
            self.audio_positional_embedding_max_pos = [20]
```
**EN:** This block defines method `__post_init__` on `LTX2ArchConfig`. It post-processes init. Key calls include `super.__post_init__`, and `super`. The implementation branches on conditions.
**CN:** 该代码块定义了 `LTX2ArchConfig` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `super.__post_init__` 和 `super`。 实现中包含条件分支。

### Lines 183-185: `LTX2Config` class overview / `LTX2Config` 类概览
```python
class LTX2Config(DiTConfig):
    """Configuration for LTX-2 Video Transformer."""
```
**EN:** This block defines class `LTX2Config`. Configuration for LTX-2 Video Transformer. It inherits from `DiTConfig`.
**CN:** 该代码块定义了类 `LTX2Config`。 它用于封装 ltx2 config 相关行为。 它继承自 `DiTConfig`。

### Lines 186-188: supporting statements / 辅助语句
```python
    arch_config: LTX2ArchConfig = field(default_factory=LTX2ArchConfig)

    prefix: str = "ltx2"
```
**EN:** This block gathers supporting statements inside `LTX2Config`. It updates names such as `arch_config`, and `prefix`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `LTX2Config` 内部的辅助语句。 它会更新 `arch_config` 和 `prefix` 等名称。 代码会与 `field` 协同工作。

## Key Concepts / 关键概念
- `LTXModelType`: Model type enum mirroring upstream `LTXModelType`. / 核心类，用于封装 ltxmodel type 相关行为。
- `LTX2RopeType`: Minimal RoPE type enum mirroring LTX-2 upstream `LTXRopeType`. / 核心类，用于封装 ltx2 rope type 相关行为。
- `LTX2AttentionFunction`: Placeholder enum for upstream `AttentionFunction.DEFAULT`. / 核心类，用于封装 ltx2 attention function 相关行为。
- `LTX2ArchConfig`: Architecture configuration for LTX-2 Video Transformer. / 核心类，用于封装 ltx2 arch config 相关行为。
- `LTX2Config`: Configuration for LTX-2 Video Transformer. / 核心类，用于封装 ltx2 config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `enum`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.dits.base`, `sglang.multimodal_gen.configs.models.fsdp`

- **Total lines / 总行数**: 188
