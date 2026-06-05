# base.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/vaes/base.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `VAEArchConfig`, and `VAEConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `VAEArchConfig` 和 `VAEConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-12: module setup and imports / 模块初始化与导入
```python
import argparse
import dataclasses
from dataclasses import dataclass, field
from typing import Any

import torch

from sglang.multimodal_gen.configs.models.base import ArchConfig, ModelConfig
from sglang.multimodal_gen.utils import StoreBoolean
```
**EN:** This block establishes the module context and imports `argparse`, `dataclasses`, `typing`, `torch`, `sglang.multimodal_gen.configs.models.base`, and `sglang.multimodal_gen.utils`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `argparse`、`dataclasses`、`typing`、`torch`、`sglang.multimodal_gen.configs.models.base` 和 `sglang.multimodal_gen.utils`。这些依赖为后续实现提供所需符号。

### Lines 16-16: `VAEArchConfig` class overview / `VAEArchConfig` 类概览
```python
class VAEArchConfig(ArchConfig):
```
**EN:** This block defines class `VAEArchConfig`. It encapsulates vaearch config behavior. It inherits from `ArchConfig`.
**CN:** 该代码块定义了类 `VAEArchConfig`。 它用于封装 vaearch config 相关行为。 它继承自 `ArchConfig`。

### Lines 17-21: supporting statements / 辅助语句
```python
    scaling_factor: float | torch.Tensor = 0

    temporal_compression_ratio: int = 4
    # or vae_scale_factor?
    spatial_compression_ratio: int = 8
```
**EN:** This block gathers supporting statements inside `VAEArchConfig`. It updates names such as `scaling_factor`, `temporal_compression_ratio`, and `spatial_compression_ratio`.
**CN:** 该代码块汇集了位于 `VAEArchConfig` 内部的辅助语句。 它会更新 `scaling_factor`、`temporal_compression_ratio` 和 `spatial_compression_ratio` 等名称。

### Lines 25-25: `VAEConfig` class overview / `VAEConfig` 类概览
```python
class VAEConfig(ModelConfig):
```
**EN:** This block defines class `VAEConfig`. It encapsulates vaeconfig behavior. It inherits from `ModelConfig`.
**CN:** 该代码块定义了类 `VAEConfig`。 它用于封装 vaeconfig 相关行为。 它继承自 `ModelConfig`。

### Lines 26-45: supporting statements / 辅助语句
```python
    arch_config: VAEArchConfig = field(default_factory=VAEArchConfig)

    # sglang-diffusion VAE-specific parameters
    load_encoder: bool = True
    load_decoder: bool = True

    tile_sample_min_height: int = 256
    tile_sample_min_width: int = 256
    tile_sample_min_num_frames: int = 16
    tile_sample_stride_height: int = 192
    tile_sample_stride_width: int = 192
    tile_sample_stride_num_frames: int = 12
    blend_num_frames: int = 0

    use_tiling: bool = True
    use_temporal_tiling: bool = True
    use_parallel_tiling: bool = True
    use_temporal_scaling_frames: bool = True
    use_parallel_decode: bool = False
    parallel_decode_mode: str = "tiled"
```
**EN:** This block gathers supporting statements inside `VAEConfig`. It updates names such as `arch_config`, `load_encoder`, `load_decoder`, `tile_sample_min_height`, `tile_sample_min_width`, and `tile_sample_min_num_frames`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `VAEConfig` 内部的辅助语句。 它会更新 `arch_config`、`load_encoder`、`load_decoder`、`tile_sample_min_height`、`tile_sample_min_width` 和 `tile_sample_min_num_frames` 等名称。 代码会与 `field` 协同工作。

### Lines 47-50: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        self.blend_num_frames = (
            self.tile_sample_min_num_frames - self.tile_sample_stride_num_frames
        )
```
**EN:** This block defines method `__post_init__` on `VAEConfig`. It post-processes init.
**CN:** 该代码块定义了 `VAEConfig` 的方法 `__post_init__`。 它用于后处理init。

### Lines 52-53: `post_init` implementation / `post_init` 实现
```python
    def post_init(self):
        pass
```
**EN:** This block defines method `post_init` on `VAEConfig`. It post-processes init.
**CN:** 该代码块定义了 `VAEConfig` 的方法 `post_init`。 它用于后处理init。

### Lines 55-157: `add_cli_args` implementation / `add_cli_args` 实现
```python
    @staticmethod
    def add_cli_args(parser: Any, prefix: str = "vae-config") -> Any:
        """Add CLI arguments for VAEConfig fields"""
        parser.add_argument(
            f"--{prefix}.load-encoder",
            action=StoreBoolean,
            dest=f"{prefix.replace('-', '_')}.load_encoder",
            default=VAEConfig.load_encoder,
            help="Whether to load the VAE encoder",
        )
        parser.add_argument(
            f"--{prefix}.load-decoder",
            action=StoreBoolean,
            dest=f"{prefix.replace('-', '_')}.load_decoder",
            default=VAEConfig.load_decoder,
            help="Whether to load the VAE decoder",
        )
        parser.add_argument(
            f"--{prefix}.tile-sample-min-height",
            type=int,
            dest=f"{prefix.replace('-', '_')}.tile_sample_min_height",
            default=VAEConfig.tile_sample_min_height,
            help="Minimum height for VAE tile sampling",
        )
        parser.add_argument(
            f"--{prefix}.tile-sample-min-width",
            type=int,
            dest=f"{prefix.replace('-', '_')}.tile_sample_min_width",
            default=VAEConfig.tile_sample_min_width,
            help="Minimum width for VAE tile sampling",
        )
        parser.add_argument(
            f"--{prefix}.tile-sample-min-num-frames",
            type=int,
            dest=f"{prefix.replace('-', '_')}.tile_sample_min_num_frames",
            default=VAEConfig.tile_sample_min_num_frames,
            help="Minimum number of frames for VAE tile sampling",
        )
        parser.add_argument(
            f"--{prefix}.tile-sample-stride-height",
            type=int,
            dest=f"{prefix.replace('-', '_')}.tile_sample_stride_height",
            default=VAEConfig.tile_sample_stride_height,
            help="Stride height for VAE tile sampling",
        )
        parser.add_argument(
            f"--{prefix}.tile-sample-stride-width",
            type=int,
            dest=f"{prefix.replace('-', '_')}.tile_sample_stride_width",
            default=VAEConfig.tile_sample_stride_width,
            help="Stride width for VAE tile sampling",
        )
        parser.add_argument(
            f"--{prefix}.tile-sample-stride-num-frames",
            type=int,
            dest=f"{prefix.replace('-', '_')}.tile_sample_stride_num_frames",
            default=VAEConfig.tile_sample_stride_num_frames,
            help="Stride number of frames for VAE tile sampling",
        )
        parser.add_argument(
            f"--{prefix}.blend-num-frames",
            type=int,
            dest=f"{prefix.replace('-', '_')}.blend_num_frames",
            default=VAEConfig.blend_num_frames,
            help="Number of frames to blend for VAE tile sampling",
        )
        parser.add_argument(
            f"--{prefix}.use-tiling",
            action=StoreBoolean,
            dest=f"{prefix.replace('-', '_')}.use_tiling",
            default=VAEConfig.use_tiling,
            help="Whether to use tiling for VAE",
        )
        parser.add_argument(
            f"--{prefix}.use-temporal-tiling",
            action=StoreBoolean,
            dest=f"{prefix.replace('-', '_')}.use_temporal_tiling",
            default=VAEConfig.use_temporal_tiling,
            help="Whether to use temporal tiling for VAE",
        )
        parser.add_argument(
            f"--{prefix}.use-parallel-tiling",
            action=StoreBoolean,
            dest=f"{prefix.replace('-', '_')}.use_parallel_tiling",
            default=VAEConfig.use_parallel_tiling,
            help="Whether to use parallel tiling for VAE",
        )
        parser.add_argument(
            f"--{prefix}.use-parallel-decode",
            action=StoreBoolean,
            dest=f"{prefix.replace('-', '_')}.use_parallel_decode",
            default=VAEConfig.use_parallel_decode,
            help="Whether to use parallel decode for VAE",
        )
        parser.add_argument(
            f"--{prefix}.parallel-decode-mode",
            choices=("tiled", "patch", "auto"),
            dest=f"{prefix.replace('-', '_')}.parallel_decode_mode",
            default=VAEConfig.parallel_decode_mode,
            help="Parallel decode mode for VAE",
        )

        return parser
```
**EN:** This block defines method `add_cli_args` on `VAEConfig`. Add CLI arguments for VAEConfig fields Key calls include `parser.add_argument`, and `prefix.replace`. Parameters such as `parser`, and `prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `VAEConfig` 的方法 `add_cli_args`。 它用于处理 add cli args 相关逻辑。 关键调用包括 `parser.add_argument` 和 `prefix.replace`。 本段逻辑主要由 `parser` 和 `prefix` 等参数驱动。

### Lines 159-160: `get_vae_scale_factor` implementation / `get_vae_scale_factor` 实现
```python
    def get_vae_scale_factor(self):
        return 2 ** (len(self.arch_config.block_out_channels) - 1)
```
**EN:** This block defines method `get_vae_scale_factor` on `VAEConfig`. It retrieves vae scale factor. Key calls include `len`.
**CN:** 该代码块定义了 `VAEConfig` 的方法 `get_vae_scale_factor`。 它用于获取vae scale factor。 关键调用包括 `len`。

### Lines 162-163: `encode_sample_mode` implementation / `encode_sample_mode` 实现
```python
    def encode_sample_mode(self):
        return "argmax"
```
**EN:** This block defines method `encode_sample_mode` on `VAEConfig`. It encodes sample mode.
**CN:** 该代码块定义了 `VAEConfig` 的方法 `encode_sample_mode`。 它用于编码sample mode。

### Lines 165-172: `from_cli_args` implementation / `from_cli_args` 实现
```python
    @classmethod
    def from_cli_args(cls, args: argparse.Namespace) -> "VAEConfig":
        kwargs = {}
        for attr in dataclasses.fields(cls):
            value = getattr(args, attr.name, None)
            if value is not None:
                kwargs[attr.name] = value
        return cls(**kwargs)
```
**EN:** This block defines method `from_cli_args` on `VAEConfig`. It constructs from cli args. Key calls include `dataclasses.fields`, `cls`, and `getattr`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `args` drive the behavior in this section.
**CN:** 该代码块定义了 `VAEConfig` 的方法 `from_cli_args`。 它用于从…构造cli args。 关键调用包括 `dataclasses.fields`、`cls` 和 `getattr`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `args` 等参数驱动。

## Key Concepts / 关键概念
- `VAEArchConfig`: Primary class that encapsulates vaearch config behavior. / 核心类，用于封装 vaearch config 相关行为。
- `VAEConfig`: Primary class that encapsulates vaeconfig behavior. / 核心类，用于封装 vaeconfig 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.base`, `sglang.multimodal_gen.utils`

- **Total lines / 总行数**: 172
