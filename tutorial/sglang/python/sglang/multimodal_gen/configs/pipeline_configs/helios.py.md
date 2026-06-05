# helios.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/pipeline_configs/helios.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `HeliosT2VConfig`, `HeliosMidConfig`, and `HeliosDistilledConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `HeliosT2VConfig`、`HeliosMidConfig` 和 `HeliosDistilledConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 2-16: module setup and imports / 模块初始化与导入
```python
from collections.abc import Callable
from dataclasses import dataclass, field

import torch

from sglang.multimodal_gen.configs.models import DiTConfig, EncoderConfig, VAEConfig
from sglang.multimodal_gen.configs.models.dits.helios import HeliosConfig
from sglang.multimodal_gen.configs.models.encoders import BaseEncoderOutput, T5Config
from sglang.multimodal_gen.configs.models.encoders.t5 import T5ArchConfig
from sglang.multimodal_gen.configs.models.vaes import WanVAEConfig
from sglang.multimodal_gen.configs.pipeline_configs.base import (
    ModelTaskType,
    PipelineConfig,
)
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `collections.abc`, `dataclasses`, `torch`, `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.configs.models.dits.helios`, and `sglang.multimodal_gen.configs.models.encoders`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `collections.abc`、`dataclasses`、`torch`、`sglang.multimodal_gen.configs.models`、`sglang.multimodal_gen.configs.models.dits.helios` 和 `sglang.multimodal_gen.configs.models.encoders`。这些依赖为后续实现提供所需符号。

### Lines 18-23: supporting statements / 辅助语句
```python
logger = init_logger(__name__)


# Helios UMT5 max sequence length (used for both tokenizer and post-processing padding)
# Matches diffusers HeliosPipeline.__call__ default max_sequence_length=512
HELIOS_MAX_SEQUENCE_LENGTH = 512
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`, and `HELIOS_MAX_SEQUENCE_LENGTH`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 和 `HELIOS_MAX_SEQUENCE_LENGTH` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 26-41: `umt5_postprocess_text` implementation / `umt5_postprocess_text` 实现
```python
def umt5_postprocess_text(outputs: BaseEncoderOutput, _text_inputs) -> torch.Tensor:
    """Post-process UMT5 text encoder outputs, padding to HELIOS_MAX_SEQUENCE_LENGTH tokens."""
    max_seq_len = HELIOS_MAX_SEQUENCE_LENGTH
    mask: torch.Tensor = outputs.attention_mask
    hidden_state: torch.Tensor = outputs.last_hidden_state
    seq_lens = mask.gt(0).sum(dim=1).long()
    assert torch.isnan(hidden_state).sum() == 0
    prompt_embeds = [u[:v] for u, v in zip(hidden_state, seq_lens, strict=True)]
    prompt_embeds_tensor: torch.Tensor = torch.stack(
        [
            torch.cat([u, u.new_zeros(max_seq_len - u.size(0), u.size(1))])
            for u in prompt_embeds
        ],
        dim=0,
    )
    return prompt_embeds_tensor
```
**EN:** This block defines function `umt5_postprocess_text`. Post-process UMT5 text encoder outputs, padding to HELIOS_MAX_SEQUENCE_LENGTH tokens. Key calls include `mask.gt.sum.long`, `torch.stack`, `torch.isnan.sum`, `mask.gt.sum`, and `zip`. Parameters such as `outputs`, and `_text_inputs` drive the behavior in this section.
**CN:** 该代码块定义了函数 `umt5_postprocess_text`。 它用于处理 umt5 postprocess text 相关逻辑。 关键调用包括 `mask.gt.sum.long`、`torch.stack`、`torch.isnan.sum`、`mask.gt.sum` 和 `zip`。 本段逻辑主要由 `outputs` 和 `_text_inputs` 等参数驱动。

### Lines 45-47: `HeliosT2VConfig` class overview / `HeliosT2VConfig` 类概览
```python
class HeliosT2VConfig(PipelineConfig):
    """Configuration for the Helios T2V pipeline."""
```
**EN:** This block defines class `HeliosT2VConfig`. Configuration for the Helios T2V pipeline. It inherits from `PipelineConfig`.
**CN:** 该代码块定义了类 `HeliosT2VConfig`。 它用于封装 helios t2 vconfig 相关行为。 它继承自 `PipelineConfig`。

### Lines 48-92: supporting statements / 辅助语句
```python
    task_type: ModelTaskType = ModelTaskType.T2V

    # DiT
    dit_config: DiTConfig = field(default_factory=HeliosConfig)

    # VAE (same as Wan)
    vae_config: VAEConfig = field(default_factory=WanVAEConfig)
    vae_tiling: bool = False
    vae_sp: bool = False

    # Denoising stage
    flow_shift: float | None = 1.0

    # Text encoding stage (UMT5 is T5-compatible)
    text_encoder_configs: tuple[EncoderConfig, ...] = field(
        default_factory=lambda: (
            T5Config(arch_config=T5ArchConfig(text_len=HELIOS_MAX_SEQUENCE_LENGTH)),
        )
    )
    postprocess_text_funcs: tuple[Callable[[BaseEncoderOutput], torch.Tensor], ...] = (
        field(default_factory=lambda: (umt5_postprocess_text,))
    )

    # Precision for each component
    precision: str = "bf16"
    vae_precision: str = "fp32"
    text_encoder_precisions: tuple[str, ...] = field(default_factory=lambda: ("fp32",))

    # Helios-specific chunked denoising params
    num_latent_frames_per_chunk: int = 9
    history_sizes: list[int] = field(default_factory=lambda: [16, 2, 1])
    is_cfg_zero_star: bool = False
    zero_steps: int = 1
    keep_first_frame: bool = True

    # Stage 2 (Pyramid SR) & Stage 3 (DMD) params
    is_enable_stage2: bool = False
    pyramid_num_stages: int = 3
    pyramid_num_inference_steps_list: list[int] = field(
        default_factory=lambda: [10, 10, 10]
    )
    is_distilled: bool = False
    is_amplify_first_chunk: bool = False
    scheduler_type: str = "unipc"
    gamma: float = 1 / 3
```
**EN:** This block gathers supporting statements inside `HeliosT2VConfig`. It updates names such as `task_type`, `dit_config`, `vae_config`, `vae_tiling`, `vae_sp`, and `flow_shift`. The code collaborates with `field`, `T5Config`, and `T5ArchConfig`.
**CN:** 该代码块汇集了位于 `HeliosT2VConfig` 内部的辅助语句。 它会更新 `task_type`、`dit_config`、`vae_config`、`vae_tiling`、`vae_sp` 和 `flow_shift` 等名称。 代码会与 `field`、`T5Config` 和 `T5ArchConfig` 协同工作。

### Lines 94-96: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        self.vae_config.load_encoder = False
        self.vae_config.load_decoder = True
```
**EN:** This block defines method `__post_init__` on `HeliosT2VConfig`. It post-processes init.
**CN:** 该代码块定义了 `HeliosT2VConfig` 的方法 `__post_init__`。 它用于后处理init。

### Lines 100-102: `HeliosMidConfig` class overview / `HeliosMidConfig` 类概览
```python
class HeliosMidConfig(HeliosT2VConfig):
    """Configuration for Helios-Mid (Stage 1 + Stage 2 pyramid SR)."""
```
**EN:** This block defines class `HeliosMidConfig`. Configuration for Helios-Mid (Stage 1 + Stage 2 pyramid SR). It inherits from `HeliosT2VConfig`.
**CN:** 该代码块定义了类 `HeliosMidConfig`。 它用于封装 helios mid config 相关行为。 它继承自 `HeliosT2VConfig`。

### Lines 103-107: supporting statements / 辅助语句
```python
    is_enable_stage2: bool = True
    is_cfg_zero_star: bool = True
    pyramid_num_inference_steps_list: list[int] = field(
        default_factory=lambda: [20, 20, 20]
    )
```
**EN:** This block gathers supporting statements inside `HeliosMidConfig`. It updates names such as `is_enable_stage2`, `is_cfg_zero_star`, and `pyramid_num_inference_steps_list`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `HeliosMidConfig` 内部的辅助语句。 它会更新 `is_enable_stage2`、`is_cfg_zero_star` 和 `pyramid_num_inference_steps_list` 等名称。 代码会与 `field` 协同工作。

### Lines 111-113: `HeliosDistilledConfig` class overview / `HeliosDistilledConfig` 类概览
```python
class HeliosDistilledConfig(HeliosT2VConfig):
    """Configuration for Helios-Distilled (Stage 1 + Stage 2 + Stage 3 DMD)."""
```
**EN:** This block defines class `HeliosDistilledConfig`. Configuration for Helios-Distilled (Stage 1 + Stage 2 + Stage 3 DMD). It inherits from `HeliosT2VConfig`.
**CN:** 该代码块定义了类 `HeliosDistilledConfig`。 它用于封装 helios distilled config 相关行为。 它继承自 `HeliosT2VConfig`。

### Lines 114-120: supporting statements / 辅助语句
```python
    is_enable_stage2: bool = True
    is_distilled: bool = True
    is_amplify_first_chunk: bool = True
    scheduler_type: str = "dmd"
    pyramid_num_inference_steps_list: list[int] = field(
        default_factory=lambda: [10, 10, 10]
    )
```
**EN:** This block gathers supporting statements inside `HeliosDistilledConfig`. It updates names such as `is_enable_stage2`, `is_distilled`, `is_amplify_first_chunk`, `scheduler_type`, and `pyramid_num_inference_steps_list`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `HeliosDistilledConfig` 内部的辅助语句。 它会更新 `is_enable_stage2`、`is_distilled`、`is_amplify_first_chunk`、`scheduler_type` 和 `pyramid_num_inference_steps_list` 等名称。 代码会与 `field` 协同工作。

## Key Concepts / 关键概念
- `umt5_postprocess_text`: Post-process UMT5 text encoder outputs, padding to HELIOS_MAX_SEQUENCE_LENGTH tokens. / 顶层函数，用于处理 umt5 postprocess text 相关逻辑。
- `HeliosT2VConfig`: Configuration for the Helios T2V pipeline. / 核心类，用于封装 helios t2 vconfig 相关行为。
- `HeliosMidConfig`: Configuration for Helios-Mid (Stage 1 + Stage 2 pyramid SR). / 核心类，用于封装 helios mid config 相关行为。
- `HeliosDistilledConfig`: Configuration for Helios-Distilled (Stage 1 + Stage 2 + Stage 3 DMD). / 核心类，用于封装 helios distilled config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `collections.abc`, `dataclasses`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.configs.models.dits.helios`, `sglang.multimodal_gen.configs.models.encoders`, `sglang.multimodal_gen.configs.models.encoders.t5`, `sglang.multimodal_gen.configs.models.vaes`, `sglang.multimodal_gen.configs.pipeline_configs.base`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 120
