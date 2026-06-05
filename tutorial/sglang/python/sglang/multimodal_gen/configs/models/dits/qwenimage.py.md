# qwenimage.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/dits/qwenimage.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `QwenImageArchConfig`, `QwenImageEditPlus_2511_ArchConfig`, and `QwenImageDitConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `QwenImageArchConfig`、`QwenImageEditPlus_2511_ArchConfig` 和 `QwenImageDitConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-8: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field
from typing import Tuple

from sglang.multimodal_gen.configs.models.dits.base import DiTArchConfig, DiTConfig
from sglang.multimodal_gen.configs.models.fsdp import is_transformer_block
```
**EN:** This block establishes the module context and imports `dataclasses`, `typing`, `sglang.multimodal_gen.configs.models.dits.base`, and `sglang.multimodal_gen.configs.models.fsdp`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`typing`、`sglang.multimodal_gen.configs.models.dits.base` 和 `sglang.multimodal_gen.configs.models.fsdp`。这些依赖为后续实现提供所需符号。

### Lines 12-12: `QwenImageArchConfig` class overview / `QwenImageArchConfig` 类概览
```python
class QwenImageArchConfig(DiTArchConfig):
```
**EN:** This block defines class `QwenImageArchConfig`. It encapsulates qwen image arch config behavior. It inherits from `DiTArchConfig`.
**CN:** 该代码块定义了类 `QwenImageArchConfig`。 它用于封装 qwen image arch config 相关行为。 它继承自 `DiTArchConfig`。

### Lines 13-39: supporting statements / 辅助语句
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
    guidance_embeds: bool = False
    axes_dims_rope: Tuple[int, int, int] = (16, 56, 56)
    zero_cond_t: bool = False

    _fsdp_shard_conditions: list = field(default_factory=lambda: [is_transformer_block])

    stacked_params_mapping: list[tuple[str, str, str]] = field(default_factory=list)

    param_names_mapping: dict = field(
        default_factory=lambda: {
            # LoRA mappings
            r"^(transformer_blocks\.\d+\.attn\..*\.lora_[AB])\.default$": r"\1",
            # SVDquant mappings
            r"(.*)\.add_qkv_proj\.(.+)$": r"\1.to_added_qkv.\2",
            r"(transformer_blocks\.\d+\.(img_mlp|txt_mlp)\..*\.(smooth_factor_orig|wcscales))$": r"\1",
            r".*\.wtscale$": r"",
        }
    )
```
**EN:** This block gathers supporting statements inside `QwenImageArchConfig`. It updates names such as `patch_size`, `in_channels`, `out_channels`, `num_layers`, `num_single_layers`, and `attention_head_dim`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `QwenImageArchConfig` 内部的辅助语句。 它会更新 `patch_size`、`in_channels`、`out_channels`、`num_layers`、`num_single_layers` 和 `attention_head_dim` 等名称。 代码会与 `field` 协同工作。

### Lines 41-45: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        super().__post_init__()
        self.out_channels = self.out_channels or self.in_channels
        self.hidden_size = self.num_attention_heads * self.attention_head_dim
        self.num_channels_latents = self.out_channels
```
**EN:** This block defines method `__post_init__` on `QwenImageArchConfig`. It post-processes init. Key calls include `super.__post_init__`, and `super`.
**CN:** 该代码块定义了 `QwenImageArchConfig` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `super.__post_init__` 和 `super`。

### Lines 49-49: `QwenImageEditPlus_2511_ArchConfig` class overview / `QwenImageEditPlus_2511_ArchConfig` 类概览
```python
class QwenImageEditPlus_2511_ArchConfig(QwenImageArchConfig):
```
**EN:** This block defines class `QwenImageEditPlus_2511_ArchConfig`. It encapsulates qwen image edit plus 2511 arch config behavior. It inherits from `QwenImageArchConfig`.
**CN:** 该代码块定义了类 `QwenImageEditPlus_2511_ArchConfig`。 它用于封装 qwen image edit plus 2511 arch config 相关行为。 它继承自 `QwenImageArchConfig`。

### Lines 50-50: supporting statements / 辅助语句
```python
    zero_cond_t: bool = True
```
**EN:** This block gathers supporting statements inside `QwenImageEditPlus_2511_ArchConfig`. It updates names such as `zero_cond_t`.
**CN:** 该代码块汇集了位于 `QwenImageEditPlus_2511_ArchConfig` 内部的辅助语句。 它会更新 `zero_cond_t` 等名称。

### Lines 54-54: `QwenImageDitConfig` class overview / `QwenImageDitConfig` 类概览
```python
class QwenImageDitConfig(DiTConfig):
```
**EN:** This block defines class `QwenImageDitConfig`. It encapsulates qwen image dit config behavior. It inherits from `DiTConfig`.
**CN:** 该代码块定义了类 `QwenImageDitConfig`。 它用于封装 qwen image dit config 相关行为。 它继承自 `DiTConfig`。

### Lines 55-57: supporting statements / 辅助语句
```python
    arch_config: DiTArchConfig = field(default_factory=QwenImageArchConfig)

    prefix: str = "qwenimage"
```
**EN:** This block gathers supporting statements inside `QwenImageDitConfig`. It updates names such as `arch_config`, and `prefix`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `QwenImageDitConfig` 内部的辅助语句。 它会更新 `arch_config` 和 `prefix` 等名称。 代码会与 `field` 协同工作。

### Lines 61-61: `QwenImageEditPlus_2511_DitConfig` class overview / `QwenImageEditPlus_2511_DitConfig` 类概览
```python
class QwenImageEditPlus_2511_DitConfig(DiTConfig):
```
**EN:** This block defines class `QwenImageEditPlus_2511_DitConfig`. It encapsulates qwen image edit plus 2511 dit config behavior. It inherits from `DiTConfig`.
**CN:** 该代码块定义了类 `QwenImageEditPlus_2511_DitConfig`。 它用于封装 qwen image edit plus 2511 dit config 相关行为。 它继承自 `DiTConfig`。

### Lines 62-66: supporting statements / 辅助语句
```python
    arch_config: DiTArchConfig = field(
        default_factory=QwenImageEditPlus_2511_ArchConfig
    )

    prefix: str = "qwenimageedit"
```
**EN:** This block gathers supporting statements inside `QwenImageEditPlus_2511_DitConfig`. It updates names such as `arch_config`, and `prefix`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `QwenImageEditPlus_2511_DitConfig` 内部的辅助语句。 它会更新 `arch_config` 和 `prefix` 等名称。 代码会与 `field` 协同工作。

## Key Concepts / 关键概念
- `QwenImageArchConfig`: Primary class that encapsulates qwen image arch config behavior. / 核心类，用于封装 qwen image arch config 相关行为。
- `QwenImageEditPlus_2511_ArchConfig`: Primary class that encapsulates qwen image edit plus 2511 arch config behavior. / 核心类，用于封装 qwen image edit plus 2511 arch config 相关行为。
- `QwenImageDitConfig`: Primary class that encapsulates qwen image dit config behavior. / 核心类，用于封装 qwen image dit config 相关行为。
- `QwenImageEditPlus_2511_DitConfig`: Primary class that encapsulates qwen image edit plus 2511 dit config behavior. / 核心类，用于封装 qwen image edit plus 2511 dit config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.dits.base`, `sglang.multimodal_gen.configs.models.fsdp`

- **Total lines / 总行数**: 66
