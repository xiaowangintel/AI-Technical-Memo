# wanvae.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/vaes/wanvae.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `WanVAEArchConfig`, and `WanVAEConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `WanVAEArchConfig` 和 `WanVAEConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-8: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field

import torch

from sglang.multimodal_gen.configs.models.vaes.base import VAEArchConfig, VAEConfig
```
**EN:** This block establishes the module context and imports `dataclasses`, `torch`, and `sglang.multimodal_gen.configs.models.vaes.base`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`torch` 和 `sglang.multimodal_gen.configs.models.vaes.base`。这些依赖为后续实现提供所需符号。

### Lines 12-12: `WanVAEArchConfig` class overview / `WanVAEArchConfig` 类概览
```python
class WanVAEArchConfig(VAEArchConfig):
```
**EN:** This block defines class `WanVAEArchConfig`. It encapsulates wan vaearch config behavior. It inherits from `VAEArchConfig`.
**CN:** 该代码块定义了类 `WanVAEArchConfig`。 它用于封装 wan vaearch config 相关行为。 它继承自 `VAEArchConfig`。

### Lines 13-63: supporting statements / 辅助语句
```python
    base_dim: int = 96
    decoder_base_dim: int | None = None
    z_dim: int = 16
    dim_mult: tuple[int, ...] = (1, 2, 4, 4)
    num_res_blocks: int = 2
    attn_scales: tuple[float, ...] = ()
    temperal_downsample: tuple[bool, ...] = (False, True, True)
    dropout: float = 0.0
    latents_mean: tuple[float, ...] = (
        -0.7571,
        -0.7089,
        -0.9113,
        0.1075,
        -0.1745,
        0.9653,
        -0.1517,
        1.5508,
        0.4134,
        -0.0715,
        0.5517,
        -0.3632,
        -0.1922,
        -0.9497,
        0.2503,
        -0.2921,
    )
    latents_std: tuple[float, ...] = (
        2.8184,
        1.4541,
        2.3275,
        2.6558,
        1.2196,
        1.7708,
        2.6052,
        2.0743,
        3.2687,
        2.1526,
        2.8652,
        1.5579,
        1.6382,
        1.1253,
        2.8251,
        1.9160,
    )
    is_residual: bool = False
    in_channels: int = 3
    out_channels: int = 3
    patch_size: int | None = None
    scale_factor_temporal: int = 4
    scale_factor_spatial: int = 8
    clip_output: bool = True
```
**EN:** This block gathers supporting statements inside `WanVAEArchConfig`. It updates names such as `base_dim`, `decoder_base_dim`, `z_dim`, `dim_mult`, `num_res_blocks`, and `attn_scales`.
**CN:** 该代码块汇集了位于 `WanVAEArchConfig` 内部的辅助语句。 它会更新 `base_dim`、`decoder_base_dim`、`z_dim`、`dim_mult`、`num_res_blocks` 和 `attn_scales` 等名称。

### Lines 65-73: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        self.scaling_factor: torch.tensor = 1.0 / torch.tensor(self.latents_std).view(
            1, self.z_dim, 1, 1, 1
        )
        self.shift_factor: torch.tensor = torch.tensor(self.latents_mean).view(
            1, self.z_dim, 1, 1, 1
        )
        self.temporal_compression_ratio = self.scale_factor_temporal
        self.spatial_compression_ratio = self.scale_factor_spatial
```
**EN:** This block defines method `__post_init__` on `WanVAEArchConfig`. It post-processes init. Key calls include `torch.tensor.view`, and `torch.tensor`.
**CN:** 该代码块定义了 `WanVAEArchConfig` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `torch.tensor.view` 和 `torch.tensor`。

### Lines 77-77: `WanVAEConfig` class overview / `WanVAEConfig` 类概览
```python
class WanVAEConfig(VAEConfig):
```
**EN:** This block defines class `WanVAEConfig`. It encapsulates wan vaeconfig behavior. It inherits from `VAEConfig`.
**CN:** 该代码块定义了类 `WanVAEConfig`。 它用于封装 wan vaeconfig 相关行为。 它继承自 `VAEConfig`。

### Lines 78-86: supporting statements / 辅助语句
```python
    arch_config: WanVAEArchConfig = field(default_factory=WanVAEArchConfig)
    use_feature_cache: bool = True

    use_tiling: bool = False
    use_temporal_tiling: bool = False
    use_parallel_tiling: bool = False

    use_parallel_encode: bool = True
    use_parallel_decode: bool = True
```
**EN:** This block gathers supporting statements inside `WanVAEConfig`. It updates names such as `arch_config`, `use_feature_cache`, `use_tiling`, `use_temporal_tiling`, `use_parallel_tiling`, and `use_parallel_encode`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `WanVAEConfig` 内部的辅助语句。 它会更新 `arch_config`、`use_feature_cache`、`use_tiling`、`use_temporal_tiling`、`use_parallel_tiling` 和 `use_parallel_encode` 等名称。 代码会与 `field` 协同工作。

### Lines 88-91: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        self.blend_num_frames = (
            self.tile_sample_min_num_frames - self.tile_sample_stride_num_frames
        ) * 2
```
**EN:** This block defines method `__post_init__` on `WanVAEConfig`. It post-processes init.
**CN:** 该代码块定义了 `WanVAEConfig` 的方法 `__post_init__`。 它用于后处理init。

### Lines 93-96: `get_vae_scale_factor` implementation / `get_vae_scale_factor` 实现
```python
    def get_vae_scale_factor(self):
        # Wan VAE does not expose block_out_channels like SD-style VAEs.
        # Its spatial downsample factor is explicitly defined by scale_factor_spatial.
        return self.arch_config.scale_factor_spatial
```
**EN:** This block defines method `get_vae_scale_factor` on `WanVAEConfig`. It retrieves vae scale factor.
**CN:** 该代码块定义了 `WanVAEConfig` 的方法 `get_vae_scale_factor`。 它用于获取vae scale factor。

## Key Concepts / 关键概念
- `WanVAEArchConfig`: Primary class that encapsulates wan vaearch config behavior. / 核心类，用于封装 wan vaearch config 相关行为。
- `WanVAEConfig`: Primary class that encapsulates wan vaeconfig behavior. / 核心类，用于封装 wan vaeconfig 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.vaes.base`

- **Total lines / 总行数**: 96
