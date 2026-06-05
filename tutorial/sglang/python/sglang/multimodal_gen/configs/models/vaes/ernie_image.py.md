# ernie_image.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/vaes/ernie_image.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `ErnieImageVAEArchConfig`, and `ErnieImageVAEConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `ErnieImageVAEArchConfig` 和 `ErnieImageVAEConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 2-4: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field

from sglang.multimodal_gen.configs.models.vaes.base import VAEArchConfig, VAEConfig
```
**EN:** This block establishes the module context and imports `dataclasses`, and `sglang.multimodal_gen.configs.models.vaes.base`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses` 和 `sglang.multimodal_gen.configs.models.vaes.base`。这些依赖为后续实现提供所需符号。

### Lines 8-8: `ErnieImageVAEArchConfig` class overview / `ErnieImageVAEArchConfig` 类概览
```python
class ErnieImageVAEArchConfig(VAEArchConfig):
```
**EN:** This block defines class `ErnieImageVAEArchConfig`. It encapsulates ernie image vaearch config behavior. It inherits from `VAEArchConfig`.
**CN:** 该代码块定义了类 `ErnieImageVAEArchConfig`。 它用于封装 ernie image vaearch config 相关行为。 它继承自 `VAEArchConfig`。

### Lines 9-26: supporting statements / 辅助语句
```python
    spatial_compression_ratio: int = 8

    base_dim: int = 96
    decoder_base_dim: int | None = None
    z_dim: int = 32
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
**EN:** This block gathers supporting statements inside `ErnieImageVAEArchConfig`. It updates names such as `spatial_compression_ratio`, `base_dim`, `decoder_base_dim`, `z_dim`, `dim_mult`, and `num_res_blocks`.
**CN:** 该代码块汇集了位于 `ErnieImageVAEArchConfig` 内部的辅助语句。 它会更新 `spatial_compression_ratio`、`base_dim`、`decoder_base_dim`、`z_dim`、`dim_mult` 和 `num_res_blocks` 等名称。

### Lines 30-30: `ErnieImageVAEConfig` class overview / `ErnieImageVAEConfig` 类概览
```python
class ErnieImageVAEConfig(VAEConfig):
```
**EN:** This block defines class `ErnieImageVAEConfig`. It encapsulates ernie image vaeconfig behavior. It inherits from `VAEConfig`.
**CN:** 该代码块定义了类 `ErnieImageVAEConfig`。 它用于封装 ernie image vaeconfig 相关行为。 它继承自 `VAEConfig`。

### Lines 31-39: supporting statements / 辅助语句
```python
    arch_config: ErnieImageVAEArchConfig = field(
        default_factory=ErnieImageVAEArchConfig
    )

    use_feature_cache: bool = True

    use_tiling: bool = False
    use_temporal_tiling: bool = False
    use_parallel_tiling: bool = False
```
**EN:** This block gathers supporting statements inside `ErnieImageVAEConfig`. It updates names such as `arch_config`, `use_feature_cache`, `use_tiling`, `use_temporal_tiling`, and `use_parallel_tiling`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `ErnieImageVAEConfig` 内部的辅助语句。 它会更新 `arch_config`、`use_feature_cache`、`use_tiling`、`use_temporal_tiling` 和 `use_parallel_tiling` 等名称。 代码会与 `field` 协同工作。

### Lines 41-43: `get_vae_scale_factor` implementation / `get_vae_scale_factor` 实现
```python
    def get_vae_scale_factor(self):
        # 8 spatial compression (VAE) * 2 patch = 16 total, consistent with pipeline config
        return self.arch_config.scale_factor_spatial
```
**EN:** This block defines method `get_vae_scale_factor` on `ErnieImageVAEConfig`. It retrieves vae scale factor.
**CN:** 该代码块定义了 `ErnieImageVAEConfig` 的方法 `get_vae_scale_factor`。 它用于获取vae scale factor。

### Lines 45-48: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        self.blend_num_frames = (
            self.tile_sample_min_num_frames - self.tile_sample_stride_num_frames
        ) * 2
```
**EN:** This block defines method `__post_init__` on `ErnieImageVAEConfig`. It post-processes init.
**CN:** 该代码块定义了 `ErnieImageVAEConfig` 的方法 `__post_init__`。 它用于后处理init。

### Lines 50-57: `post_init` implementation / `post_init` 实现
```python
    def post_init(self):
        if self.arch_config.dim_mult:
            self.arch_config.vae_scale_factor = 2 ** (
                len(self.arch_config.dim_mult) - 1
            )
        else:
            self.arch_config.vae_scale_factor = self.arch_config.scale_factor_spatial
        self.arch_config.spatial_compression_ratio = self.arch_config.vae_scale_factor
```
**EN:** This block defines method `post_init` on `ErnieImageVAEConfig`. It post-processes init. Key calls include `len`. The implementation branches on conditions.
**CN:** 该代码块定义了 `ErnieImageVAEConfig` 的方法 `post_init`。 它用于后处理init。 关键调用包括 `len`。 实现中包含条件分支。

## Key Concepts / 关键概念
- `ErnieImageVAEArchConfig`: Primary class that encapsulates ernie image vaearch config behavior. / 核心类，用于封装 ernie image vaearch config 相关行为。
- `ErnieImageVAEConfig`: Primary class that encapsulates ernie image vaeconfig behavior. / 核心类，用于封装 ernie image vaeconfig 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.vaes.base`

- **Total lines / 总行数**: 57
