# sana.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/dits/sana.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `SanaArchConfig`, and `SanaConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `SanaArchConfig` 和 `SanaConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 15-17: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field

from sglang.multimodal_gen.configs.models.dits.base import DiTArchConfig, DiTConfig
```
**EN:** This block establishes the module context and imports `dataclasses`, and `sglang.multimodal_gen.configs.models.dits.base`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses` 和 `sglang.multimodal_gen.configs.models.dits.base`。这些依赖为后续实现提供所需符号。

### Lines 21-21: `SanaArchConfig` class overview / `SanaArchConfig` 类概览
```python
class SanaArchConfig(DiTArchConfig):
```
**EN:** This block defines class `SanaArchConfig`. It encapsulates sana arch config behavior. It inherits from `DiTArchConfig`.
**CN:** 该代码块定义了类 `SanaArchConfig`。 它用于封装 sana arch config 相关行为。 它继承自 `DiTArchConfig`。

### Lines 22-46: supporting statements / 辅助语句
```python
    patch_size: int = 1
    in_channels: int = 32
    out_channels: int = 32
    num_layers: int = 20
    attention_head_dim: int = 32
    num_attention_heads: int = 70
    num_cross_attention_heads: int = 20
    cross_attention_head_dim: int = 112
    cross_attention_dim: int = 2240
    caption_channels: int = 2304

    mlp_ratio: float = 2.5
    # "rms_norm_across_heads" applies RMSNorm over the full (num_heads * head_dim)

    qk_norm: str = "rms_norm_across_heads"
    norm_elementwise_affine: bool = False
    norm_eps: float = 1e-6
    sample_size: int = 32
    guidance_embeds: bool = False

    param_names_mapping: dict = field(
        default_factory=lambda: {
            r"^transformer\.(.*)$": r"\1",
        }
    )
```
**EN:** This block gathers supporting statements inside `SanaArchConfig`. It updates names such as `patch_size`, `in_channels`, `out_channels`, `num_layers`, `attention_head_dim`, and `num_attention_heads`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `SanaArchConfig` 内部的辅助语句。 它会更新 `patch_size`、`in_channels`、`out_channels`、`num_layers`、`attention_head_dim` 和 `num_attention_heads` 等名称。 代码会与 `field` 协同工作。

### Lines 48-51: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        super().__post_init__()
        self.hidden_size = self.num_attention_heads * self.attention_head_dim
        self.num_channels_latents = self.out_channels
```
**EN:** This block defines method `__post_init__` on `SanaArchConfig`. It post-processes init. Key calls include `super.__post_init__`, and `super`.
**CN:** 该代码块定义了 `SanaArchConfig` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `super.__post_init__` 和 `super`。

### Lines 55-55: `SanaConfig` class overview / `SanaConfig` 类概览
```python
class SanaConfig(DiTConfig):
```
**EN:** This block defines class `SanaConfig`. It encapsulates sana config behavior. It inherits from `DiTConfig`.
**CN:** 该代码块定义了类 `SanaConfig`。 它用于封装 sana config 相关行为。 它继承自 `DiTConfig`。

### Lines 56-57: supporting statements / 辅助语句
```python
    arch_config: DiTArchConfig = field(default_factory=SanaArchConfig)
    prefix: str = "Sana"
```
**EN:** This block gathers supporting statements inside `SanaConfig`. It updates names such as `arch_config`, and `prefix`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `SanaConfig` 内部的辅助语句。 它会更新 `arch_config` 和 `prefix` 等名称。 代码会与 `field` 协同工作。

## Key Concepts / 关键概念
- `SanaArchConfig`: Primary class that encapsulates sana arch config behavior. / 核心类，用于封装 sana arch config 相关行为。
- `SanaConfig`: Primary class that encapsulates sana config behavior. / 核心类，用于封装 sana config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.dits.base`

- **Total lines / 总行数**: 57
