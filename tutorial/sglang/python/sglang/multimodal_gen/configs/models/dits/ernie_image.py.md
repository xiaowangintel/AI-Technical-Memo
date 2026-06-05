# ernie_image.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/dits/ernie_image.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `ErnieImageArchConfig`, and `ErnieImageDitConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `ErnieImageArchConfig` 和 `ErnieImageDitConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 3-7: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field
from typing import Tuple

from sglang.multimodal_gen.configs.models.dits.base import DiTArchConfig, DiTConfig
from sglang.multimodal_gen.configs.models.fsdp import is_layer
```
**EN:** This block establishes the module context and imports `dataclasses`, `typing`, `sglang.multimodal_gen.configs.models.dits.base`, and `sglang.multimodal_gen.configs.models.fsdp`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`typing`、`sglang.multimodal_gen.configs.models.dits.base` 和 `sglang.multimodal_gen.configs.models.fsdp`。这些依赖为后续实现提供所需符号。

### Lines 11-11: `ErnieImageArchConfig` class overview / `ErnieImageArchConfig` 类概览
```python
class ErnieImageArchConfig(DiTArchConfig):
```
**EN:** This block defines class `ErnieImageArchConfig`. It encapsulates ernie image arch config behavior. It inherits from `DiTArchConfig`.
**CN:** 该代码块定义了类 `ErnieImageArchConfig`。 它用于封装 ernie image arch config 相关行为。 它继承自 `DiTArchConfig`。

### Lines 12-34: supporting statements / 辅助语句
```python
    patch_size: int = 1
    in_channels: int = 128
    out_channels: int = 128
    num_layers: int = 36
    attention_head_dim: int = 128
    num_attention_heads: int = 32
    ffn_hidden_size: int = 12288
    text_in_dim: int = 3072
    rope_theta: int = 256
    rope_axes_dim: Tuple[int, int, int] = (32, 48, 48)
    eps: float = 1e-6
    qk_layernorm: bool = True

    stacked_params_mapping: list[tuple[str, str, str]] = field(default_factory=list)

    param_names_mapping: dict = field(
        default_factory=lambda: {
            r"(.*)\.mlp\.gate_proj\.(.*)": (r"\1.mlp.gate_up_proj.\2", 0, 2),
            r"(.*)\.mlp\.up_proj\.(.*)": (r"\1.mlp.gate_up_proj.\2", 1, 2),
        }
    )

    _fsdp_shard_conditions: list = field(default_factory=lambda: [is_layer])
```
**EN:** This block gathers supporting statements inside `ErnieImageArchConfig`. It updates names such as `patch_size`, `in_channels`, `out_channels`, `num_layers`, `attention_head_dim`, and `num_attention_heads`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `ErnieImageArchConfig` 内部的辅助语句。 它会更新 `patch_size`、`in_channels`、`out_channels`、`num_layers`、`attention_head_dim` 和 `num_attention_heads` 等名称。 代码会与 `field` 协同工作。

### Lines 36-39: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        super().__post_init__()
        self.hidden_size = self.num_attention_heads * self.attention_head_dim
        self.num_channels_latents = self.out_channels
```
**EN:** This block defines method `__post_init__` on `ErnieImageArchConfig`. It post-processes init. Key calls include `super.__post_init__`, and `super`.
**CN:** 该代码块定义了 `ErnieImageArchConfig` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `super.__post_init__` 和 `super`。

### Lines 43-43: `ErnieImageDitConfig` class overview / `ErnieImageDitConfig` 类概览
```python
class ErnieImageDitConfig(DiTConfig):
```
**EN:** This block defines class `ErnieImageDitConfig`. It encapsulates ernie image dit config behavior. It inherits from `DiTConfig`.
**CN:** 该代码块定义了类 `ErnieImageDitConfig`。 它用于封装 ernie image dit config 相关行为。 它继承自 `DiTConfig`。

### Lines 44-45: supporting statements / 辅助语句
```python
    arch_config: DiTArchConfig = field(default_factory=ErnieImageArchConfig)
    prefix: str = "ernieimage"
```
**EN:** This block gathers supporting statements inside `ErnieImageDitConfig`. It updates names such as `arch_config`, and `prefix`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `ErnieImageDitConfig` 内部的辅助语句。 它会更新 `arch_config` 和 `prefix` 等名称。 代码会与 `field` 协同工作。

## Key Concepts / 关键概念
- `ErnieImageArchConfig`: Primary class that encapsulates ernie image arch config behavior. / 核心类，用于封装 ernie image arch config 相关行为。
- `ErnieImageDitConfig`: Primary class that encapsulates ernie image dit config behavior. / 核心类，用于封装 ernie image dit config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.dits.base`, `sglang.multimodal_gen.configs.models.fsdp`

- **Total lines / 总行数**: 45
