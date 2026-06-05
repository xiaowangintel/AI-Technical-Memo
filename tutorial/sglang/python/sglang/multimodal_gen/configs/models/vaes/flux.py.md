# flux.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/vaes/flux.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `FluxVAEArchConfig`, `Flux2VAEArchConfig`, and `FluxVAEConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `FluxVAEArchConfig`、`Flux2VAEArchConfig` 和 `FluxVAEConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 2-4: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field

from sglang.multimodal_gen.configs.models.vaes.base import VAEArchConfig, VAEConfig
```
**EN:** This block establishes the module context and imports `dataclasses`, and `sglang.multimodal_gen.configs.models.vaes.base`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses` 和 `sglang.multimodal_gen.configs.models.vaes.base`。这些依赖为后续实现提供所需符号。

### Lines 8-8: `FluxVAEArchConfig` class overview / `FluxVAEArchConfig` 类概览
```python
class FluxVAEArchConfig(VAEArchConfig):
```
**EN:** This block defines class `FluxVAEArchConfig`. It encapsulates flux vaearch config behavior. It inherits from `VAEArchConfig`.
**CN:** 该代码块定义了类 `FluxVAEArchConfig`。 它用于封装 flux vaearch config 相关行为。 它继承自 `VAEArchConfig`。

### Lines 9-26: supporting statements / 辅助语句
```python
    spatial_compression_ratio: int = 1

    base_dim: int = 96
    decoder_base_dim: int | None = None
    z_dim: int = 16
    dim_mult: tuple[int, ...] = (1, 2, 4, 4)
    num_res_blocks: int = 2
    attn_scales: tuple[float, ...] = ()
    temperal_downsample: tuple[bool, ...] = (False, True, True)
    dropout: float = 0.0

    is_residual: bool = False
    in_channels: int = 3
    out_channels: int = 3
    patch_size: int | None = None
    scale_factor_temporal: int = 4
    scale_factor_spatial: int = 8
    clip_output: bool = True
```
**EN:** This block gathers supporting statements inside `FluxVAEArchConfig`. It updates names such as `spatial_compression_ratio`, `base_dim`, `decoder_base_dim`, `z_dim`, `dim_mult`, and `num_res_blocks`.
**CN:** 该代码块汇集了位于 `FluxVAEArchConfig` 内部的辅助语句。 它会更新 `spatial_compression_ratio`、`base_dim`、`decoder_base_dim`、`z_dim`、`dim_mult` 和 `num_res_blocks` 等名称。

### Lines 30-30: `Flux2VAEArchConfig` class overview / `Flux2VAEArchConfig` 类概览
```python
class Flux2VAEArchConfig(FluxVAEArchConfig):
```
**EN:** This block defines class `Flux2VAEArchConfig`. It encapsulates flux2 vaearch config behavior. It inherits from `FluxVAEArchConfig`.
**CN:** 该代码块定义了类 `Flux2VAEArchConfig`。 它用于封装 flux2 vaearch config 相关行为。 它继承自 `FluxVAEArchConfig`。

### Lines 31-31: supporting statements / 辅助语句
```python
    decoder_block_out_channels: tuple[int, ...] | None = None
```
**EN:** This block gathers supporting statements inside `Flux2VAEArchConfig`. It updates names such as `decoder_block_out_channels`.
**CN:** 该代码块汇集了位于 `Flux2VAEArchConfig` 内部的辅助语句。 它会更新 `decoder_block_out_channels` 等名称。

### Lines 35-35: `FluxVAEConfig` class overview / `FluxVAEConfig` 类概览
```python
class FluxVAEConfig(VAEConfig):
```
**EN:** This block defines class `FluxVAEConfig`. It encapsulates flux vaeconfig behavior. It inherits from `VAEConfig`.
**CN:** 该代码块定义了类 `FluxVAEConfig`。 它用于封装 flux vaeconfig 相关行为。 它继承自 `VAEConfig`。

### Lines 36-42: supporting statements / 辅助语句
```python
    arch_config: FluxVAEArchConfig = field(default_factory=FluxVAEArchConfig)

    use_feature_cache: bool = True

    use_tiling: bool = False
    use_temporal_tiling: bool = False
    use_parallel_tiling: bool = False
```
**EN:** This block gathers supporting statements inside `FluxVAEConfig`. It updates names such as `arch_config`, `use_feature_cache`, `use_tiling`, `use_temporal_tiling`, and `use_parallel_tiling`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `FluxVAEConfig` 内部的辅助语句。 它会更新 `arch_config`、`use_feature_cache`、`use_tiling`、`use_temporal_tiling` 和 `use_parallel_tiling` 等名称。 代码会与 `field` 协同工作。

### Lines 44-47: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        self.blend_num_frames = (
            self.tile_sample_min_num_frames - self.tile_sample_stride_num_frames
        ) * 2
```
**EN:** This block defines method `__post_init__` on `FluxVAEConfig`. It post-processes init.
**CN:** 该代码块定义了 `FluxVAEConfig` 的方法 `__post_init__`。 它用于后处理init。

### Lines 49-65: `post_init` implementation / `post_init` 实现
```python
    def post_init(self):
        # Calculate vae_scale_factor: prefer block_out_channels, fallback to dim_mult or scale_factor_spatial
        if (
            hasattr(self.arch_config, "block_out_channels")
            and self.arch_config.block_out_channels
        ):
            self.arch_config.vae_scale_factor = 2 ** (
                len(self.arch_config.block_out_channels) - 1
            )
        elif self.arch_config.dim_mult:
            self.arch_config.vae_scale_factor = 2 ** (
                len(self.arch_config.dim_mult) - 1
            )
        else:
            self.arch_config.vae_scale_factor = self.arch_config.scale_factor_spatial

        self.arch_config.spatial_compression_ratio = self.arch_config.vae_scale_factor
```
**EN:** This block defines method `post_init` on `FluxVAEConfig`. It post-processes init. Key calls include `hasattr`, and `len`. The implementation branches on conditions.
**CN:** 该代码块定义了 `FluxVAEConfig` 的方法 `post_init`。 它用于后处理init。 关键调用包括 `hasattr` 和 `len`。 实现中包含条件分支。

### Lines 69-69: `Flux2VAEConfig` class overview / `Flux2VAEConfig` 类概览
```python
class Flux2VAEConfig(FluxVAEConfig):
```
**EN:** This block defines class `Flux2VAEConfig`. It encapsulates flux2 vaeconfig behavior. It inherits from `FluxVAEConfig`.
**CN:** 该代码块定义了类 `Flux2VAEConfig`。 它用于封装 flux2 vaeconfig 相关行为。 它继承自 `FluxVAEConfig`。

### Lines 70-70: supporting statements / 辅助语句
```python
    arch_config: Flux2VAEArchConfig = field(default_factory=Flux2VAEArchConfig)
```
**EN:** This block gathers supporting statements inside `Flux2VAEConfig`. It updates names such as `arch_config`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `Flux2VAEConfig` 内部的辅助语句。 它会更新 `arch_config` 等名称。 代码会与 `field` 协同工作。

## Key Concepts / 关键概念
- `FluxVAEArchConfig`: Primary class that encapsulates flux vaearch config behavior. / 核心类，用于封装 flux vaearch config 相关行为。
- `Flux2VAEArchConfig`: Primary class that encapsulates flux2 vaearch config behavior. / 核心类，用于封装 flux2 vaearch config 相关行为。
- `FluxVAEConfig`: Primary class that encapsulates flux vaeconfig behavior. / 核心类，用于封装 flux vaeconfig 相关行为。
- `Flux2VAEConfig`: Primary class that encapsulates flux2 vaeconfig behavior. / 核心类，用于封装 flux2 vaeconfig 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.vaes.base`

- **Total lines / 总行数**: 70
