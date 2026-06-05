# mova_video.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/dits/mova_video.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `MOVAVideoArchConfig`, and `MOVAVideoConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `MOVAVideoArchConfig` 和 `MOVAVideoConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-7: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field

from sglang.multimodal_gen.configs.models.dits.base import DiTArchConfig, DiTConfig
from sglang.multimodal_gen.configs.models.fsdp import is_block
```
**EN:** This block establishes the module context and imports `dataclasses`, `sglang.multimodal_gen.configs.models.dits.base`, and `sglang.multimodal_gen.configs.models.fsdp`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`sglang.multimodal_gen.configs.models.dits.base` 和 `sglang.multimodal_gen.configs.models.fsdp`。这些依赖为后续实现提供所需符号。

### Lines 11-11: `MOVAVideoArchConfig` class overview / `MOVAVideoArchConfig` 类概览
```python
class MOVAVideoArchConfig(DiTArchConfig):
```
**EN:** This block defines class `MOVAVideoArchConfig`. It encapsulates movavideo arch config behavior. It inherits from `DiTArchConfig`.
**CN:** 该代码块定义了类 `MOVAVideoArchConfig`。 它用于封装 movavideo arch config 相关行为。 它继承自 `DiTArchConfig`。

### Lines 12-48: supporting statements / 辅助语句
```python
    _fsdp_shard_conditions: list = field(default_factory=lambda: [is_block])

    param_names_mapping: dict = field(
        default_factory=lambda: {
            r"^blocks\.(\d+)\.ffn\.0\.(.*)$": r"blocks.\1.ffn.fc_in.\2",
            r"^blocks\.(\d+)\.ffn\.2\.(.*)$": r"blocks.\1.ffn.fc_out.\2",
            r"^blocks\.(\d+)\.norm3\.(.*)$": r"blocks.\1.self_attn_norm.\2",
            r"^text_embedding\.0\.(.*)$": r"text_embedding.fc_in.\1",
            r"^text_embedding\.2\.(.*)$": r"text_embedding.fc_out.\1",
            r"^time_embedding\.0\.(.*)$": r"time_embedding.fc_in.\1",
            r"^time_embedding\.2\.(.*)$": r"time_embedding.fc_out.\1",
            r"^img_emb\.proj\.1\.(.*)$": r"img_emb.fc_in.\1",
            r"^img_emb\.proj\.3\.(.*)$": r"img_emb.fc_out.\1",
        }
    )
    reverse_param_names_mapping: dict = field(default_factory=dict)
    lora_param_names_mapping: dict = field(default_factory=dict)

    dim: int = 5120
    in_dim: int = 16
    ffn_dim: int = 13824
    out_dim: int = 16
    text_dim: int = 4096
    freq_dim: int = 256
    eps: float = 1e-6
    patch_size: tuple[int, int, int] = (1, 2, 2)
    num_heads: int = 40
    num_layers: int = 40
    has_image_input: bool = False
    has_image_pos_emb: bool = False
    has_ref_conv: bool = False
    add_control_adapter: bool = False
    in_dim_control_adapter: int = 24
    separated_timestep: bool = False
    require_vae_embedding: bool = True
    require_clip_embedding: bool = True
    fuse_vae_embedding_in_latents: bool = False
```
**EN:** This block gathers supporting statements inside `MOVAVideoArchConfig`. It updates names such as `_fsdp_shard_conditions`, `param_names_mapping`, `reverse_param_names_mapping`, `lora_param_names_mapping`, `dim`, and `in_dim`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `MOVAVideoArchConfig` 内部的辅助语句。 它会更新 `_fsdp_shard_conditions`、`param_names_mapping`、`reverse_param_names_mapping`、`lora_param_names_mapping`、`dim` 和 `in_dim` 等名称。 代码会与 `field` 协同工作。

### Lines 50-57: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        super().__post_init__()
        self.hidden_size = self.dim
        self.num_attention_heads = self.num_heads
        self.num_channels_latents = self.out_dim
        assert (
            not self.has_image_input
        ), "has_image_input must be False; it's a config from Diffsynth Studio, which means the model uses CLIP for image encoding (we don't)."
```
**EN:** This block defines method `__post_init__` on `MOVAVideoArchConfig`. It post-processes init. Key calls include `super.__post_init__`, and `super`.
**CN:** 该代码块定义了 `MOVAVideoArchConfig` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `super.__post_init__` 和 `super`。

### Lines 61-61: `MOVAVideoConfig` class overview / `MOVAVideoConfig` 类概览
```python
class MOVAVideoConfig(DiTConfig):
```
**EN:** This block defines class `MOVAVideoConfig`. It encapsulates movavideo config behavior. It inherits from `DiTConfig`.
**CN:** 该代码块定义了类 `MOVAVideoConfig`。 它用于封装 movavideo config 相关行为。 它继承自 `DiTConfig`。

### Lines 62-63: supporting statements / 辅助语句
```python
    arch_config: DiTArchConfig = field(default_factory=MOVAVideoArchConfig)
    prefix: str = "mova_video"
```
**EN:** This block gathers supporting statements inside `MOVAVideoConfig`. It updates names such as `arch_config`, and `prefix`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `MOVAVideoConfig` 内部的辅助语句。 它会更新 `arch_config` 和 `prefix` 等名称。 代码会与 `field` 协同工作。

## Key Concepts / 关键概念
- `MOVAVideoArchConfig`: Primary class that encapsulates movavideo arch config behavior. / 核心类，用于封装 movavideo arch config 相关行为。
- `MOVAVideoConfig`: Primary class that encapsulates movavideo config behavior. / 核心类，用于封装 movavideo config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.dits.base`, `sglang.multimodal_gen.configs.models.fsdp`

- **Total lines / 总行数**: 63
