# hunyuan3d.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/pipeline_configs/hunyuan3d.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `Hunyuan3D2PipelineConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `Hunyuan3D2PipelineConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 2-11: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field
from typing import Optional

from sglang.multimodal_gen.configs.models import DiTConfig, VAEConfig
from sglang.multimodal_gen.configs.models.dits.hunyuan3d import Hunyuan3DDiTConfig
from sglang.multimodal_gen.configs.models.vaes.hunyuan3d import Hunyuan3DVAEConfig
from sglang.multimodal_gen.configs.pipeline_configs.base import (
    ModelTaskType,
    PipelineConfig,
)
```
**EN:** This block establishes the module context and imports `dataclasses`, `typing`, `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.configs.models.dits.hunyuan3d`, `sglang.multimodal_gen.configs.models.vaes.hunyuan3d`, and `sglang.multimodal_gen.configs.pipeline_configs.base`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`typing`、`sglang.multimodal_gen.configs.models`、`sglang.multimodal_gen.configs.models.dits.hunyuan3d`、`sglang.multimodal_gen.configs.models.vaes.hunyuan3d` 和 `sglang.multimodal_gen.configs.pipeline_configs.base`。这些依赖为后续实现提供所需符号。

### Lines 15-17: `Hunyuan3D2PipelineConfig` class overview / `Hunyuan3D2PipelineConfig` 类概览
```python
class Hunyuan3D2PipelineConfig(PipelineConfig):
    """Pipeline configuration for Hunyuan3D image-to-mesh generation."""
```
**EN:** This block defines class `Hunyuan3D2PipelineConfig`. Pipeline configuration for Hunyuan3D image-to-mesh generation. It inherits from `PipelineConfig`.
**CN:** 该代码块定义了类 `Hunyuan3D2PipelineConfig`。 它用于封装 hunyuan3 d2 pipeline config 相关行为。 它继承自 `PipelineConfig`。

### Lines 18-64: supporting statements / 辅助语句
```python
    task_type: ModelTaskType = ModelTaskType.I2M

    # Subfolder paths
    shape_subfolder: str = "hunyuan3d-dit-v2-0"
    paint_subfolder: str = "hunyuan3d-paint-v2-0"
    delight_subfolder: str = "hunyuan3d-delight-v2-0"

    # DiT configuration
    dit_config: DiTConfig = field(default_factory=Hunyuan3DDiTConfig)
    dit_precision: str = "fp16"

    # VAE configuration
    vae_config: VAEConfig = field(default_factory=Hunyuan3DVAEConfig)
    vae_precision: str = "fp32"

    # Shape model configuration
    shape_model_path: Optional[str] = None
    shape_use_safetensors: bool = True
    shape_variant: Optional[str] = "fp16"
    shape_num_inference_steps: int = 50
    guidance_scale: float = 5.0
    shape_box_v: float = 1.01
    shape_octree_resolution: int = 384
    shape_mc_level: float = 0.0
    shape_mc_algo: Optional[str] = "mc"
    shape_num_chunks: int = 32000
    shape_output_type: str = "trimesh"

    # Delight model configuration
    delight_enable: bool = True
    delight_prompt: str = ""
    delight_negative_prompt: str = ""
    delight_strength: float = 1.0
    delight_num_inference_steps: int = 50
    delight_guidance_scale: float = 1.0
    delight_cfg_image: float = 1.5

    # Paint model configuration
    paint_enable: bool = True
    paint_num_inference_steps: int = 30
    paint_guidance_scale: float = 2.0
    paint_resolution: int = 512
    paint_render_size: int = 2048
    paint_texture_size: int = 2048
    paint_use_remesh: bool = True
    paint_save_glb: bool = True
    paint_turbo_mode: bool = False
```
**EN:** This block gathers supporting statements inside `Hunyuan3D2PipelineConfig`. It updates names such as `task_type`, `shape_subfolder`, `paint_subfolder`, `delight_subfolder`, `dit_config`, and `dit_precision`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `Hunyuan3D2PipelineConfig` 内部的辅助语句。 它会更新 `task_type`、`shape_subfolder`、`paint_subfolder`、`delight_subfolder`、`dit_config` 和 `dit_precision` 等名称。 代码会与 `field` 协同工作。

### Lines 66-68: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        self.vae_config.load_encoder = False
        self.vae_config.load_decoder = True
```
**EN:** This block defines method `__post_init__` on `Hunyuan3D2PipelineConfig`. It post-processes init.
**CN:** 该代码块定义了 `Hunyuan3D2PipelineConfig` 的方法 `__post_init__`。 它用于后处理init。

### Lines 70-73: `prepare_latent_shape` implementation / `prepare_latent_shape` 实现
```python
    def prepare_latent_shape(self, batch, batch_size, num_frames):
        latent_shape = self.vae_config.arch_config.latent_shape
        shape = (batch_size, *latent_shape)
        return shape
```
**EN:** This block defines method `prepare_latent_shape` on `Hunyuan3D2PipelineConfig`. It prepares latent shape. Parameters such as `batch`, `batch_size`, and `num_frames` drive the behavior in this section.
**CN:** 该代码块定义了 `Hunyuan3D2PipelineConfig` 的方法 `prepare_latent_shape`。 它用于准备latent shape。 本段逻辑主要由 `batch`、`batch_size` 和 `num_frames` 等参数驱动。

## Key Concepts / 关键概念
- `Hunyuan3D2PipelineConfig`: Pipeline configuration for Hunyuan3D image-to-mesh generation. / 核心类，用于封装 hunyuan3 d2 pipeline config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.configs.models.dits.hunyuan3d`, `sglang.multimodal_gen.configs.models.vaes.hunyuan3d`, `sglang.multimodal_gen.configs.pipeline_configs.base`

- **Total lines / 总行数**: 73
