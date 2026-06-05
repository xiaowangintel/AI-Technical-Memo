# zimage.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/dits/zimage.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `ZImageArchConfig`, and `ZImageDitConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `ZImageArchConfig` 和 `ZImageDitConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-8: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field
from typing import Tuple

from sglang.multimodal_gen.configs.models.dits.base import DiTArchConfig, DiTConfig
from sglang.multimodal_gen.configs.models.fsdp import is_zimage_layer
```
**EN:** This block establishes the module context and imports `dataclasses`, `typing`, `sglang.multimodal_gen.configs.models.dits.base`, and `sglang.multimodal_gen.configs.models.fsdp`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`typing`、`sglang.multimodal_gen.configs.models.dits.base` 和 `sglang.multimodal_gen.configs.models.fsdp`。这些依赖为后续实现提供所需符号。

### Lines 12-12: `ZImageArchConfig` class overview / `ZImageArchConfig` 类概览
```python
class ZImageArchConfig(DiTArchConfig):
```
**EN:** This block defines class `ZImageArchConfig`. It encapsulates zimage arch config behavior. It inherits from `DiTArchConfig`.
**CN:** 该代码块定义了类 `ZImageArchConfig`。 它用于封装 zimage arch config 相关行为。 它继承自 `DiTArchConfig`。

### Lines 13-88: supporting statements / 辅助语句
```python
    all_patch_size: Tuple[int, ...] = (2,)
    all_f_patch_size: Tuple[int, ...] = (1,)
    in_channels: int = 16
    out_channels: int | None = None
    dim: int = 3840
    num_layers: int = 30
    n_refiner_layers: int = 2
    num_attention_heads: int = 30
    n_kv_heads: int = 30
    norm_eps: float = 1e-5
    qk_norm: bool = True
    cap_feat_dim: int = 2560
    rope_theta: float = 256.0
    t_scale: float = 1000.0
    axes_dims: Tuple[int, int, int] = (32, 48, 48)
    axes_lens: Tuple[int, int, int] = (1024, 512, 512)

    _fsdp_shard_conditions: list = field(default_factory=lambda: [is_zimage_layer])

    stacked_params_mapping: list[tuple[str, str, str]] = field(
        default_factory=lambda: [
            # (param_name, shard_name, shard_id)
            (".feed_forward.w13", ".feed_forward.w1", "gate"),
            (".feed_forward.w13", ".feed_forward.w3", "up"),
        ]
    )

    param_names_mapping: dict = field(
        default_factory=lambda: {
            r"(.*)\.attention\.to_q\.weight$": (r"\1.attention.to_qkv.weight", 0, 3),
            r"(.*)\.attention\.to_k\.weight$": (r"\1.attention.to_qkv.weight", 1, 3),
            r"(.*)\.attention\.to_v\.weight$": (r"\1.attention.to_qkv.weight", 2, 3),
            r"(.*)\.attention\.to_q\.weight_scale_inv$": (
                r"\1.attention.to_qkv.weight_scale_inv",
                0,
                3,
            ),
            r"(.*)\.attention\.to_k\.weight_scale_inv$": (
                r"\1.attention.to_qkv.weight_scale_inv",
                1,
                3,
            ),
            r"(.*)\.attention\.to_v\.weight_scale_inv$": (
                r"\1.attention.to_qkv.weight_scale_inv",
                2,
                3,
            ),
            r"(.*)\.attention\.to_q\.(lora_A|lora_B)$": (
                r"\1.attention.to_qkv.\2",
                0,
                3,
            ),
            r"(.*)\.attention\.to_k\.(lora_A|lora_B)$": (
                r"\1.attention.to_qkv.\2",
                1,
                3,
            ),
            r"(.*)\.attention\.to_v\.(lora_A|lora_B)$": (
                r"\1.attention.to_qkv.\2",
                2,
                3,
            ),
            r"(.*)\.feed_forward\.w1\.weight$": (r"\1.feed_forward.w13.weight", 0, 2),
            r"(.*)\.feed_forward\.w3\.weight$": (r"\1.feed_forward.w13.weight", 1, 2),
            r"(.*)\.feed_forward\.w1\.(lora_A|lora_B)$": (
                r"\1.feed_forward.w13.\2",
                0,
                2,
            ),
            r"(.*)\.feed_forward\.w3\.(lora_A|lora_B)$": (
                r"\1.feed_forward.w13.\2",
                1,
                2,
            ),
        }
    )
```
**EN:** This block gathers supporting statements inside `ZImageArchConfig`. It updates names such as `all_patch_size`, `all_f_patch_size`, `in_channels`, `out_channels`, `dim`, and `num_layers`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `ZImageArchConfig` 内部的辅助语句。 它会更新 `all_patch_size`、`all_f_patch_size`、`in_channels`、`out_channels`、`dim` 和 `num_layers` 等名称。 代码会与 `field` 协同工作。

### Lines 90-94: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        super().__post_init__()
        self.out_channels = self.out_channels or self.in_channels
        self.num_channels_latents = self.in_channels
        self.hidden_size = self.dim
```
**EN:** This block defines method `__post_init__` on `ZImageArchConfig`. It post-processes init. Key calls include `super.__post_init__`, and `super`.
**CN:** 该代码块定义了 `ZImageArchConfig` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `super.__post_init__` 和 `super`。

### Lines 98-98: `ZImageDitConfig` class overview / `ZImageDitConfig` 类概览
```python
class ZImageDitConfig(DiTConfig):
```
**EN:** This block defines class `ZImageDitConfig`. It encapsulates zimage dit config behavior. It inherits from `DiTConfig`.
**CN:** 该代码块定义了类 `ZImageDitConfig`。 它用于封装 zimage dit config 相关行为。 它继承自 `DiTConfig`。

### Lines 99-101: supporting statements / 辅助语句
```python
    arch_config: ZImageArchConfig = field(default_factory=ZImageArchConfig)

    prefix: str = "zimage"
```
**EN:** This block gathers supporting statements inside `ZImageDitConfig`. It updates names such as `arch_config`, and `prefix`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `ZImageDitConfig` 内部的辅助语句。 它会更新 `arch_config` 和 `prefix` 等名称。 代码会与 `field` 协同工作。

## Key Concepts / 关键概念
- `ZImageArchConfig`: Primary class that encapsulates zimage arch config behavior. / 核心类，用于封装 zimage arch config 相关行为。
- `ZImageDitConfig`: Primary class that encapsulates zimage dit config behavior. / 核心类，用于封装 zimage dit config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.dits.base`, `sglang.multimodal_gen.configs.models.fsdp`

- **Total lines / 总行数**: 101
