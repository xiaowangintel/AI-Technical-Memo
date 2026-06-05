# wan.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/pipeline_configs/wan.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `WanI2VCommonConfig`, `WanT2V480PConfig`, and `TurboWanT2V480PConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `WanI2VCommonConfig`、`WanT2V480PConfig` 和 `TurboWanT2V480PConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-24: module setup and imports / 模块初始化与导入
```python
from collections.abc import Callable
from dataclasses import dataclass, field

import torch

from sglang.multimodal_gen.configs.models import DiTConfig, EncoderConfig, VAEConfig
from sglang.multimodal_gen.configs.models.dits import WanVideoConfig
from sglang.multimodal_gen.configs.models.encoders import (
    BaseEncoderOutput,
    CLIPVisionConfig,
    T5Config,
)
from sglang.multimodal_gen.configs.models.vaes import WanVAEConfig
from sglang.multimodal_gen.configs.pipeline_configs.base import (
    ModelTaskType,
    PipelineConfig,
)
from sglang.multimodal_gen.configs.pipeline_configs.model_deployment_config import (
    ModelDeploymentConfig,
)
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `collections.abc`, `dataclasses`, `torch`, `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.configs.models.dits`, and `sglang.multimodal_gen.configs.models.encoders`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `collections.abc`、`dataclasses`、`torch`、`sglang.multimodal_gen.configs.models`、`sglang.multimodal_gen.configs.models.dits` 和 `sglang.multimodal_gen.configs.models.encoders`。这些依赖为后续实现提供所需符号。

### Lines 26-26: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 29-42: `t5_postprocess_text` implementation / `t5_postprocess_text` 实现
```python
def t5_postprocess_text(outputs: BaseEncoderOutput, _text_inputs) -> torch.Tensor:
    mask: torch.Tensor = outputs.attention_mask
    hidden_state: torch.Tensor = outputs.last_hidden_state
    seq_lens = mask.gt(0).sum(dim=1).long()
    assert torch.isnan(hidden_state).sum() == 0
    prompt_embeds = [u[:v] for u, v in zip(hidden_state, seq_lens, strict=True)]
    prompt_embeds_tensor: torch.Tensor = torch.stack(
        [
            torch.cat([u, u.new_zeros(512 - u.size(0), u.size(1))])
            for u in prompt_embeds
        ],
        dim=0,
    )
    return prompt_embeds_tensor
```
**EN:** This block defines function `t5_postprocess_text`. It handles t5 postprocess text logic. Key calls include `mask.gt.sum.long`, `torch.stack`, `torch.isnan.sum`, `mask.gt.sum`, and `zip`. Parameters such as `outputs`, and `_text_inputs` drive the behavior in this section.
**CN:** 该代码块定义了函数 `t5_postprocess_text`。 它用于处理 t5 postprocess text 相关逻辑。 关键调用包括 `mask.gt.sum.long`、`torch.stack`、`torch.isnan.sum`、`mask.gt.sum` 和 `zip`。 本段逻辑主要由 `outputs` 和 `_text_inputs` 等参数驱动。

### Lines 46-47: `WanI2VCommonConfig` class overview / `WanI2VCommonConfig` 类概览
```python
class WanI2VCommonConfig(PipelineConfig):
    # for all wan i2v pipelines
```
**EN:** This block defines class `WanI2VCommonConfig`. It encapsulates wan i2 vcommon config behavior. It inherits from `PipelineConfig`.
**CN:** 该代码块定义了类 `WanI2VCommonConfig`。 它用于封装 wan i2 vcommon config 相关行为。 它继承自 `PipelineConfig`。

### Lines 48-58: `adjust_num_frames` implementation / `adjust_num_frames` 实现
```python
    def adjust_num_frames(self, num_frames):
        vae_scale_factor_temporal = self.vae_config.arch_config.scale_factor_temporal
        if num_frames % vae_scale_factor_temporal != 1:
            logger.warning(
                f"`num_frames - 1` has to be divisible by {vae_scale_factor_temporal}. Rounding to the nearest number."
            )
            num_frames = (
                num_frames // vae_scale_factor_temporal * vae_scale_factor_temporal + 1
            )
            return num_frames
        return num_frames
```
**EN:** This block defines method `adjust_num_frames` on `WanI2VCommonConfig`. It handles adjust num frames logic. Key calls include `logger.warning`. The implementation branches on conditions. Parameters such as `num_frames` drive the behavior in this section.
**CN:** 该代码块定义了 `WanI2VCommonConfig` 的方法 `adjust_num_frames`。 它用于处理 adjust num frames 相关逻辑。 关键调用包括 `logger.warning`。 实现中包含条件分支。 本段逻辑主要由 `num_frames` 等参数驱动。

### Lines 62-64: `WanT2V480PConfig` class overview / `WanT2V480PConfig` 类概览
```python
class WanT2V480PConfig(PipelineConfig):
    """Base configuration for Wan T2V 1.3B pipeline architecture."""
```
**EN:** This block defines class `WanT2V480PConfig`. Base configuration for Wan T2V 1.3B pipeline architecture. It inherits from `PipelineConfig`.
**CN:** 该代码块定义了类 `WanT2V480PConfig`。 它用于封装 wan t2 v480 pconfig 相关行为。 它继承自 `PipelineConfig`。

### Lines 65-89: supporting statements / 辅助语句
```python
    task_type: ModelTaskType = ModelTaskType.T2V
    # WanConfig-specific parameters with defaults
    # DiT
    dit_config: DiTConfig = field(default_factory=WanVideoConfig)

    # VAE
    vae_config: VAEConfig = field(default_factory=WanVAEConfig)
    vae_tiling: bool = False
    vae_sp: bool = False

    # Denoising stage
    flow_shift: float | None = 3.0

    # Text encoding stage
    text_encoder_configs: tuple[EncoderConfig, ...] = field(
        default_factory=lambda: (T5Config(),)
    )
    postprocess_text_funcs: tuple[Callable[[BaseEncoderOutput], torch.Tensor], ...] = (
        field(default_factory=lambda: (t5_postprocess_text,))
    )

    # Precision for each component
    precision: str = "bf16"
    vae_precision: str = "fp32"
    text_encoder_precisions: tuple[str, ...] = field(default_factory=lambda: ("fp32",))
```
**EN:** This block gathers supporting statements inside `WanT2V480PConfig`. It updates names such as `task_type`, `dit_config`, `vae_config`, `vae_tiling`, `vae_sp`, and `flow_shift`. The code collaborates with `field`, and `T5Config`.
**CN:** 该代码块汇集了位于 `WanT2V480PConfig` 内部的辅助语句。 它会更新 `task_type`、`dit_config`、`vae_config`、`vae_tiling`、`vae_sp` 和 `flow_shift` 等名称。 代码会与 `field` 和 `T5Config` 协同工作。

### Lines 91-93: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        self.vae_config.load_encoder = False
        self.vae_config.load_decoder = True
```
**EN:** This block defines method `__post_init__` on `WanT2V480PConfig`. It post-processes init.
**CN:** 该代码块定义了 `WanT2V480PConfig` 的方法 `__post_init__`。 它用于后处理init。

### Lines 95-99: `get_model_deployment_config` implementation / `get_model_deployment_config` 实现
```python
    def get_model_deployment_config(self) -> ModelDeploymentConfig:
        return ModelDeploymentConfig(
            auto_dit_layerwise_offload=True,
            auto_dit_layerwise_offload_high_memory_disable_gb=130,
        )
```
**EN:** This block defines method `get_model_deployment_config` on `WanT2V480PConfig`. It retrieves model deployment config. Key calls include `ModelDeploymentConfig`.
**CN:** 该代码块定义了 `WanT2V480PConfig` 的方法 `get_model_deployment_config`。 它用于获取model deployment config。 关键调用包括 `ModelDeploymentConfig`。

### Lines 103-105: `TurboWanT2V480PConfig` class overview / `TurboWanT2V480PConfig` 类概览
```python
class TurboWanT2V480PConfig(WanT2V480PConfig):
    """Base configuration for Wan T2V 1.3B pipeline architecture."""
```
**EN:** This block defines class `TurboWanT2V480PConfig`. Base configuration for Wan T2V 1.3B pipeline architecture. It inherits from `WanT2V480PConfig`.
**CN:** 该代码块定义了类 `TurboWanT2V480PConfig`。 它用于封装 turbo wan t2 v480 pconfig 相关行为。 它继承自 `WanT2V480PConfig`。

### Lines 106-109: supporting statements / 辅助语句
```python
    flow_shift: float | None = 8.0
    dmd_denoising_steps: list[int] | None = field(
        default_factory=lambda: [988, 932, 852, 608]
    )
```
**EN:** This block gathers supporting statements inside `TurboWanT2V480PConfig`. It updates names such as `flow_shift`, and `dmd_denoising_steps`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `TurboWanT2V480PConfig` 内部的辅助语句。 它会更新 `flow_shift` 和 `dmd_denoising_steps` 等名称。 代码会与 `field` 协同工作。

### Lines 113-118: `WanT2V720PConfig` class overview / `WanT2V720PConfig` 类概览
```python
class WanT2V720PConfig(WanT2V480PConfig):
    """Base configuration for Wan T2V 14B 720P pipeline architecture."""

    # WanConfig-specific parameters with defaults

    # Denoising stage
```
**EN:** This block defines class `WanT2V720PConfig`. Base configuration for Wan T2V 14B 720P pipeline architecture. It inherits from `WanT2V480PConfig`.
**CN:** 该代码块定义了类 `WanT2V720PConfig`。 它用于封装 wan t2 v720 pconfig 相关行为。 它继承自 `WanT2V480PConfig`。

### Lines 119-119: supporting statements / 辅助语句
```python
    flow_shift: float | None = 5.0
```
**EN:** This block gathers supporting statements inside `WanT2V720PConfig`. It updates names such as `flow_shift`.
**CN:** 该代码块汇集了位于 `WanT2V720PConfig` 内部的辅助语句。 它会更新 `flow_shift` 等名称。

### Lines 123-125: `WanI2V480PConfig` class overview / `WanI2V480PConfig` 类概览
```python
class WanI2V480PConfig(WanT2V480PConfig, WanI2VCommonConfig):
    """Base configuration for Wan I2V 14B 480P pipeline architecture."""
```
**EN:** This block defines class `WanI2V480PConfig`. Base configuration for Wan I2V 14B 480P pipeline architecture. It inherits from `WanT2V480PConfig`, and `WanI2VCommonConfig`.
**CN:** 该代码块定义了类 `WanI2V480PConfig`。 它用于封装 wan i2 v480 pconfig 相关行为。 它继承自 `WanT2V480PConfig` 和 `WanI2VCommonConfig`。

### Lines 126-137: supporting statements / 辅助语句
```python
    max_area: int = 480 * 832
    # WanConfig-specific parameters with defaults
    task_type: ModelTaskType = ModelTaskType.I2V
    # Precision for each component
    image_encoder_config: EncoderConfig = field(default_factory=CLIPVisionConfig)
    image_encoder_precision: str = "fp32"

    image_encoder_extra_args: dict = field(
        default_factory=lambda: dict(
            output_hidden_states=True,
        )
    )
```
**EN:** This block gathers supporting statements inside `WanI2V480PConfig`. It updates names such as `max_area`, `task_type`, `image_encoder_config`, `image_encoder_precision`, and `image_encoder_extra_args`. The code collaborates with `field`, and `dict`.
**CN:** 该代码块汇集了位于 `WanI2V480PConfig` 内部的辅助语句。 它会更新 `max_area`、`task_type`、`image_encoder_config`、`image_encoder_precision` 和 `image_encoder_extra_args` 等名称。 代码会与 `field` 和 `dict` 协同工作。

### Lines 139-140: `postprocess_image` implementation / `postprocess_image` 实现
```python
    def postprocess_image(self, image):
        return image.hidden_states[-2]
```
**EN:** This block defines method `postprocess_image` on `WanI2V480PConfig`. It handles postprocess image logic. Parameters such as `image` drive the behavior in this section.
**CN:** 该代码块定义了 `WanI2V480PConfig` 的方法 `postprocess_image`。 它用于处理 postprocess image 相关逻辑。 本段逻辑主要由 `image` 等参数驱动。

### Lines 142-144: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self) -> None:
        self.vae_config.load_encoder = True
        self.vae_config.load_decoder = True
```
**EN:** This block defines method `__post_init__` on `WanI2V480PConfig`. It post-processes init.
**CN:** 该代码块定义了 `WanI2V480PConfig` 的方法 `__post_init__`。 它用于后处理init。

### Lines 146-150: `get_model_deployment_config` implementation / `get_model_deployment_config` 实现
```python
    def get_model_deployment_config(self) -> ModelDeploymentConfig:
        return ModelDeploymentConfig(
            auto_dit_layerwise_offload=True,
            auto_dit_layerwise_offload_high_memory_disable_gb=130,
        )
```
**EN:** This block defines method `get_model_deployment_config` on `WanI2V480PConfig`. It retrieves model deployment config. Key calls include `ModelDeploymentConfig`.
**CN:** 该代码块定义了 `WanI2V480PConfig` 的方法 `get_model_deployment_config`。 它用于获取model deployment config。 关键调用包括 `ModelDeploymentConfig`。

### Lines 154-156: `WanI2V720PConfig` class overview / `WanI2V720PConfig` 类概览
```python
class WanI2V720PConfig(WanI2V480PConfig):
    """Base configuration for Wan I2V 14B 720P pipeline architecture."""
```
**EN:** This block defines class `WanI2V720PConfig`. Base configuration for Wan I2V 14B 720P pipeline architecture. It inherits from `WanI2V480PConfig`.
**CN:** 该代码块定义了类 `WanI2V720PConfig`。 它用于封装 wan i2 v720 pconfig 相关行为。 它继承自 `WanI2V480PConfig`。

### Lines 157-161: supporting statements / 辅助语句
```python
    max_area: int = 720 * 1280
    # WanConfig-specific parameters with defaults

    # Denoising stage
    flow_shift: float | None = 5.0
```
**EN:** This block gathers supporting statements inside `WanI2V720PConfig`. It updates names such as `max_area`, and `flow_shift`.
**CN:** 该代码块汇集了位于 `WanI2V720PConfig` 内部的辅助语句。 它会更新 `max_area` 和 `flow_shift` 等名称。

### Lines 165-165: `TurboWanI2V720Config` class overview / `TurboWanI2V720Config` 类概览
```python
class TurboWanI2V720Config(WanI2V720PConfig):
```
**EN:** This block defines class `TurboWanI2V720Config`. It encapsulates turbo wan i2 v720 config behavior. It inherits from `WanI2V720PConfig`.
**CN:** 该代码块定义了类 `TurboWanI2V720Config`。 它用于封装 turbo wan i2 v720 config 相关行为。 它继承自 `WanI2V720PConfig`。

### Lines 166-170: supporting statements / 辅助语句
```python
    flow_shift: float | None = 8.0
    dmd_denoising_steps: list[int] | None = field(
        default_factory=lambda: [996, 932, 852, 608]
    )
    boundary_ratio: float | None = 0.9
```
**EN:** This block gathers supporting statements inside `TurboWanI2V720Config`. It updates names such as `flow_shift`, `dmd_denoising_steps`, and `boundary_ratio`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `TurboWanI2V720Config` 内部的辅助语句。 它会更新 `flow_shift`、`dmd_denoising_steps` 和 `boundary_ratio` 等名称。 代码会与 `field` 协同工作。

### Lines 172-173: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self) -> None:
        self.dit_config.boundary_ratio = self.boundary_ratio
```
**EN:** This block defines method `__post_init__` on `TurboWanI2V720Config`. It post-processes init.
**CN:** 该代码块定义了 `TurboWanI2V720Config` 的方法 `__post_init__`。 它用于后处理init。

### Lines 177-182: `FastWan2_1_T2V_480P_Config` class overview / `FastWan2_1_T2V_480P_Config` 类概览
```python
class FastWan2_1_T2V_480P_Config(WanT2V480PConfig):
    """Base configuration for FastWan T2V 1.3B 480P pipeline architecture with DMD"""

    # WanConfig-specific parameters with defaults

    # Denoising stage
```
**EN:** This block defines class `FastWan2_1_T2V_480P_Config`. Base configuration for FastWan T2V 1.3B 480P pipeline architecture with DMD It inherits from `WanT2V480PConfig`.
**CN:** 该代码块定义了类 `FastWan2_1_T2V_480P_Config`。 它用于封装 fast wan2 1 t2 v 480 p config 相关行为。 它继承自 `WanT2V480PConfig`。

### Lines 183-186: supporting statements / 辅助语句
```python
    flow_shift: float | None = 8.0
    dmd_denoising_steps: list[int] | None = field(
        default_factory=lambda: [1000, 757, 522]
    )
```
**EN:** This block gathers supporting statements inside `FastWan2_1_T2V_480P_Config`. It updates names such as `flow_shift`, and `dmd_denoising_steps`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `FastWan2_1_T2V_480P_Config` 内部的辅助语句。 它会更新 `flow_shift` 和 `dmd_denoising_steps` 等名称。 代码会与 `field` 协同工作。

### Lines 190-190: `Wan2_2_TI2V_5B_Config` class overview / `Wan2_2_TI2V_5B_Config` 类概览
```python
class Wan2_2_TI2V_5B_Config(WanT2V480PConfig, WanI2VCommonConfig):
```
**EN:** This block defines class `Wan2_2_TI2V_5B_Config`. It encapsulates wan2 2 ti2 v 5 b config behavior. It inherits from `WanT2V480PConfig`, and `WanI2VCommonConfig`.
**CN:** 该代码块定义了类 `Wan2_2_TI2V_5B_Config`。 它用于封装 wan2 2 ti2 v 5 b config 相关行为。 它继承自 `WanT2V480PConfig` 和 `WanI2VCommonConfig`。

### Lines 191-195: supporting statements / 辅助语句
```python
    flow_shift: float | None = 5.0
    task_type: ModelTaskType = ModelTaskType.TI2V
    expand_timesteps: bool = True
    # ti2v, 5B
    vae_stride = (4, 16, 16)
```
**EN:** This block gathers supporting statements inside `Wan2_2_TI2V_5B_Config`. It updates names such as `flow_shift`, `task_type`, `expand_timesteps`, and `vae_stride`.
**CN:** 该代码块汇集了位于 `Wan2_2_TI2V_5B_Config` 内部的辅助语句。 它会更新 `flow_shift`、`task_type`、`expand_timesteps` 和 `vae_stride` 等名称。

### Lines 197-204: `prepare_latent_shape` implementation / `prepare_latent_shape` 实现
```python
    def prepare_latent_shape(self, batch, batch_size, num_frames):
        F = num_frames
        z_dim = self.vae_config.arch_config.z_dim
        vae_stride = self.vae_stride
        oh = batch.height
        ow = batch.width
        shape = (batch_size, z_dim, F, oh // vae_stride[1], ow // vae_stride[2])
        return shape
```
**EN:** This block defines method `prepare_latent_shape` on `Wan2_2_TI2V_5B_Config`. It prepares latent shape. Parameters such as `batch`, `batch_size`, and `num_frames` drive the behavior in this section.
**CN:** 该代码块定义了 `Wan2_2_TI2V_5B_Config` 的方法 `prepare_latent_shape`。 它用于准备latent shape。 本段逻辑主要由 `batch`、`batch_size` 和 `num_frames` 等参数驱动。

### Lines 206-209: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self) -> None:
        self.vae_config.load_encoder = True
        self.vae_config.load_decoder = True
        self.dit_config.expand_timesteps = self.expand_timesteps
```
**EN:** This block defines method `__post_init__` on `Wan2_2_TI2V_5B_Config`. It post-processes init.
**CN:** 该代码块定义了 `Wan2_2_TI2V_5B_Config` 的方法 `__post_init__`。 它用于后处理init。

### Lines 213-213: `FastWan2_2_TI2V_5B_Config` class overview / `FastWan2_2_TI2V_5B_Config` 类概览
```python
class FastWan2_2_TI2V_5B_Config(Wan2_2_TI2V_5B_Config):
```
**EN:** This block defines class `FastWan2_2_TI2V_5B_Config`. It encapsulates fast wan2 2 ti2 v 5 b config behavior. It inherits from `Wan2_2_TI2V_5B_Config`.
**CN:** 该代码块定义了类 `FastWan2_2_TI2V_5B_Config`。 它用于封装 fast wan2 2 ti2 v 5 b config 相关行为。 它继承自 `Wan2_2_TI2V_5B_Config`。

### Lines 214-217: supporting statements / 辅助语句
```python
    flow_shift: float | None = 5.0
    dmd_denoising_steps: list[int] | None = field(
        default_factory=lambda: [1000, 757, 522]
    )
```
**EN:** This block gathers supporting statements inside `FastWan2_2_TI2V_5B_Config`. It updates names such as `flow_shift`, and `dmd_denoising_steps`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `FastWan2_2_TI2V_5B_Config` 内部的辅助语句。 它会更新 `flow_shift` 和 `dmd_denoising_steps` 等名称。 代码会与 `field` 协同工作。

### Lines 221-221: `Wan2_2_T2V_A14B_Config` class overview / `Wan2_2_T2V_A14B_Config` 类概览
```python
class Wan2_2_T2V_A14B_Config(WanT2V480PConfig):
```
**EN:** This block defines class `Wan2_2_T2V_A14B_Config`. It encapsulates wan2 2 t2 v a14 b config behavior. It inherits from `WanT2V480PConfig`.
**CN:** 该代码块定义了类 `Wan2_2_T2V_A14B_Config`。 它用于封装 wan2 2 t2 v a14 b config 相关行为。 它继承自 `WanT2V480PConfig`。

### Lines 222-223: supporting statements / 辅助语句
```python
    flow_shift: float | None = 12.0
    boundary_ratio: float | None = 0.875
```
**EN:** This block gathers supporting statements inside `Wan2_2_T2V_A14B_Config`. It updates names such as `flow_shift`, and `boundary_ratio`.
**CN:** 该代码块汇集了位于 `Wan2_2_T2V_A14B_Config` 内部的辅助语句。 它会更新 `flow_shift` 和 `boundary_ratio` 等名称。

### Lines 225-226: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self) -> None:
        self.dit_config.boundary_ratio = self.boundary_ratio
```
**EN:** This block defines method `__post_init__` on `Wan2_2_T2V_A14B_Config`. It post-processes init.
**CN:** 该代码块定义了 `Wan2_2_T2V_A14B_Config` 的方法 `__post_init__`。 它用于后处理init。

### Lines 230-230: `Wan2_2_I2V_A14B_Config` class overview / `Wan2_2_I2V_A14B_Config` 类概览
```python
class Wan2_2_I2V_A14B_Config(WanI2V720PConfig):
```
**EN:** This block defines class `Wan2_2_I2V_A14B_Config`. It encapsulates wan2 2 i2 v a14 b config behavior. It inherits from `WanI2V720PConfig`.
**CN:** 该代码块定义了类 `Wan2_2_I2V_A14B_Config`。 它用于封装 wan2 2 i2 v a14 b config 相关行为。 它继承自 `WanI2V720PConfig`。

### Lines 231-232: supporting statements / 辅助语句
```python
    flow_shift: float | None = 5.0
    boundary_ratio: float | None = 0.900
```
**EN:** This block gathers supporting statements inside `Wan2_2_I2V_A14B_Config`. It updates names such as `flow_shift`, and `boundary_ratio`.
**CN:** 该代码块汇集了位于 `Wan2_2_I2V_A14B_Config` 内部的辅助语句。 它会更新 `flow_shift` 和 `boundary_ratio` 等名称。

### Lines 234-236: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self) -> None:
        super().__post_init__()
        self.dit_config.boundary_ratio = self.boundary_ratio
```
**EN:** This block defines method `__post_init__` on `Wan2_2_I2V_A14B_Config`. It post-processes init. Key calls include `super.__post_init__`, and `super`.
**CN:** 该代码块定义了 `Wan2_2_I2V_A14B_Config` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `super.__post_init__` 和 `super`。

### Lines 243-243: `SelfForcingWanT2V480PConfig` class overview / `SelfForcingWanT2V480PConfig` 类概览
```python
class SelfForcingWanT2V480PConfig(WanT2V480PConfig):
```
**EN:** This block defines class `SelfForcingWanT2V480PConfig`. It encapsulates self forcing wan t2 v480 pconfig behavior. It inherits from `WanT2V480PConfig`.
**CN:** 该代码块定义了类 `SelfForcingWanT2V480PConfig`。 它用于封装 self forcing wan t2 v480 pconfig 相关行为。 它继承自 `WanT2V480PConfig`。

### Lines 244-249: supporting statements / 辅助语句
```python
    is_causal: bool = True
    flow_shift: float | None = 5.0
    dmd_denoising_steps: list[int] | None = field(
        default_factory=lambda: [1000, 750, 500, 250]
    )
    warp_denoising_step: bool = True
```
**EN:** This block gathers supporting statements inside `SelfForcingWanT2V480PConfig`. It updates names such as `is_causal`, `flow_shift`, `dmd_denoising_steps`, and `warp_denoising_step`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `SelfForcingWanT2V480PConfig` 内部的辅助语句。 它会更新 `is_causal`、`flow_shift`、`dmd_denoising_steps` 和 `warp_denoising_step` 等名称。 代码会与 `field` 协同工作。

## Key Concepts / 关键概念
- `t5_postprocess_text`: Top-level function that handles t5 postprocess text logic. / 顶层函数，用于处理 t5 postprocess text 相关逻辑。
- `WanI2VCommonConfig`: Primary class that encapsulates wan i2 vcommon config behavior. / 核心类，用于封装 wan i2 vcommon config 相关行为。
- `WanT2V480PConfig`: Base configuration for Wan T2V 1.3B pipeline architecture. / 核心类，用于封装 wan t2 v480 pconfig 相关行为。
- `TurboWanT2V480PConfig`: Base configuration for Wan T2V 1.3B pipeline architecture. / 核心类，用于封装 turbo wan t2 v480 pconfig 相关行为。
- `WanT2V720PConfig`: Base configuration for Wan T2V 14B 720P pipeline architecture. / 核心类，用于封装 wan t2 v720 pconfig 相关行为。
- `WanI2V480PConfig`: Base configuration for Wan I2V 14B 480P pipeline architecture. / 核心类，用于封装 wan i2 v480 pconfig 相关行为。
- `WanI2V720PConfig`: Base configuration for Wan I2V 14B 720P pipeline architecture. / 核心类，用于封装 wan i2 v720 pconfig 相关行为。
- `TurboWanI2V720Config`: Primary class that encapsulates turbo wan i2 v720 config behavior. / 核心类，用于封装 turbo wan i2 v720 config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `collections.abc`, `dataclasses`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.configs.models.dits`, `sglang.multimodal_gen.configs.models.encoders`, `sglang.multimodal_gen.configs.models.vaes`, `sglang.multimodal_gen.configs.pipeline_configs.base`, `sglang.multimodal_gen.configs.pipeline_configs.model_deployment_config`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 249
