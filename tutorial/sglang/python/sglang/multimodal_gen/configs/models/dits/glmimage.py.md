# glmimage.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/dits/glmimage.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `GlmImageArchConfig`, and `GlmImageDitConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `GlmImageArchConfig` 和 `GlmImageDitConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field

from sglang.multimodal_gen.configs.models.dits.base import DiTArchConfig, DiTConfig
```
**EN:** This block establishes the module context and imports `dataclasses`, and `sglang.multimodal_gen.configs.models.dits.base`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses` 和 `sglang.multimodal_gen.configs.models.dits.base`。这些依赖为后续实现提供所需符号。

### Lines 7-7: `GlmImageArchConfig` class overview / `GlmImageArchConfig` 类概览
```python
class GlmImageArchConfig(DiTArchConfig):
```
**EN:** This block defines class `GlmImageArchConfig`. It encapsulates glm image arch config behavior. It inherits from `DiTArchConfig`.
**CN:** 该代码块定义了类 `GlmImageArchConfig`。 它用于封装 glm image arch config 相关行为。 它继承自 `DiTArchConfig`。

### Lines 8-26: supporting statements / 辅助语句
```python
    patch_size: int = 2
    in_channels: int = 16
    out_channels: int | None = 16
    num_layers: int = 30
    attention_head_dim: int = 128
    num_attention_heads: int = 32
    condition_dim: int = 256
    prior_vq_quantizer_codebook_size: int = 16384
    text_embed_dim: int = 1472
    time_embed_dim: int = 512

    stacked_params_mapping: list[tuple[str, str, str]] = field(default_factory=list)

    param_names_mapping: dict = field(
        default_factory=lambda: {
            # LoRA mappings
            r"^(transformer_blocks\.\d+\.attn\..*\.lora_[AB])\.default$": r"\1",
        }
    )
```
**EN:** This block gathers supporting statements inside `GlmImageArchConfig`. It updates names such as `patch_size`, `in_channels`, `out_channels`, `num_layers`, `attention_head_dim`, and `num_attention_heads`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `GlmImageArchConfig` 内部的辅助语句。 它会更新 `patch_size`、`in_channels`、`out_channels`、`num_layers`、`attention_head_dim` 和 `num_attention_heads` 等名称。 代码会与 `field` 协同工作。

### Lines 28-32: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        super().__post_init__()
        self.out_channels = self.out_channels or self.in_channels
        self.hidden_size = self.num_attention_heads * self.attention_head_dim
        self.num_channels_latents = self.out_channels
```
**EN:** This block defines method `__post_init__` on `GlmImageArchConfig`. It post-processes init. Key calls include `super.__post_init__`, and `super`.
**CN:** 该代码块定义了 `GlmImageArchConfig` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `super.__post_init__` 和 `super`。

### Lines 36-36: `GlmImageDitConfig` class overview / `GlmImageDitConfig` 类概览
```python
class GlmImageDitConfig(DiTConfig):
```
**EN:** This block defines class `GlmImageDitConfig`. It encapsulates glm image dit config behavior. It inherits from `DiTConfig`.
**CN:** 该代码块定义了类 `GlmImageDitConfig`。 它用于封装 glm image dit config 相关行为。 它继承自 `DiTConfig`。

### Lines 37-39: supporting statements / 辅助语句
```python
    arch_config: DiTArchConfig = field(default_factory=GlmImageArchConfig)

    prefix: str = "glmimage"
```
**EN:** This block gathers supporting statements inside `GlmImageDitConfig`. It updates names such as `arch_config`, and `prefix`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `GlmImageDitConfig` 内部的辅助语句。 它会更新 `arch_config` 和 `prefix` 等名称。 代码会与 `field` 协同工作。

## Key Concepts / 关键概念
- `GlmImageArchConfig`: Primary class that encapsulates glm image arch config behavior. / 核心类，用于封装 glm image arch config 相关行为。
- `GlmImageDitConfig`: Primary class that encapsulates glm image dit config behavior. / 核心类，用于封装 glm image dit config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.dits.base`

- **Total lines / 总行数**: 39
