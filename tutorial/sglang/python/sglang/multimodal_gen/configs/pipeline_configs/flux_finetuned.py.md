# flux_finetuned.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/pipeline_configs/flux_finetuned.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `Flux2FinetunedPipelineConfig`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Pipeline configuration for Flux fine-tuned/distilled models. This module provides specialized handling for Flux fine-tuned models from HuggingFace, such as fal/FLUX.2-Tiny-AutoEncoder and other community fine-tuned variants. / 该文件属于配置层。它围绕 `Flux2FinetunedPipelineConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: module setup and imports / 模块初始化与导入
```python
"""
Pipeline configuration for Flux fine-tuned/distilled models.

This module provides specialized handling for Flux fine-tuned models from HuggingFace,
such as fal/FLUX.2-Tiny-AutoEncoder and other community fine-tuned variants.

Key differences from standard Flux2PipelineConfig:
- Handles custom VAE architectures loaded via auto_map
- Supports both patchified (128 channels) and unpatchified (32 channels) latents
- Dynamically adapts scale/shift based on VAE type
- Properly handles 5D latents (batch, channels, frames, height, width) for decoding
"""

from dataclasses import dataclass

import torch

from sglang.multimodal_gen.configs.pipeline_configs.flux import (
    Flux2PipelineConfig,
    _unpatchify_latents,
)
```
**EN:** This block establishes the module context and imports `dataclasses`, `torch`, and `sglang.multimodal_gen.configs.pipeline_configs.flux`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`torch` 和 `sglang.multimodal_gen.configs.pipeline_configs.flux`。这些依赖为后续实现提供所需符号。

### Lines 25-38: `Flux2FinetunedPipelineConfig` class overview / `Flux2FinetunedPipelineConfig` 类概览
```python
class Flux2FinetunedPipelineConfig(Flux2PipelineConfig):
    """
    Pipeline configuration for Flux fine-tuned/distilled models.

    This configuration automatically detects and handles custom VAE architectures
    (e.g., Flux2TinyAutoEncoder) loaded via HuggingFace's auto_map mechanism.

    Features:
    - Automatic VAE type detection (standard vs. distilled)
    - Proper handling of patchified/unpatchified latents
    - Support for custom scaling factors from fine-tuned models
    - 5D latents support for both single-frame and multi-frame generation
    """
```
**EN:** This block defines class `Flux2FinetunedPipelineConfig`. Pipeline configuration for Flux fine-tuned/distilled models. This configuration automatically detects and handles custom VAE architectures (e.g., Flux2TinyAutoEncoder) loaded via HuggingFace's auto_map mechanism. It inherits from `Flux2PipelineConfig`.
**CN:** 该代码块定义了类 `Flux2FinetunedPipelineConfig`。 它用于封装 flux2 finetuned pipeline config 相关行为。 它继承自 `Flux2PipelineConfig`。

### Lines 39-71: `preprocess_decoding` implementation / `preprocess_decoding` 实现
```python
    def preprocess_decoding(
        self, latents: torch.Tensor, server_args=None, vae=None
    ) -> torch.Tensor:
        """
        Preprocess latents before decoding.

        Handles both standard Flux2 VAE and fine-tuned/distilled VAEs:
        - Standard Flux2 VAE (has bn): needs unpatchify (128 channels -> 32 channels)
        - Distilled/Finetuned VAE (no bn): keeps patchified latents (128 channels)

        Also handles 5D latents (batch, channels, frames, height, width) by converting
        to 4D (batch, channels, height, width) for single-frame cases.

        Args:
            latents: Input latents tensor, can be 4D or 5D
            server_args: Server arguments (optional, for compatibility)
            vae: VAE model instance for dynamic type detection

        Returns:
            Preprocessed latents ready for VAE decoding
        """
        # Handle 5D latents (batch, channels, frames, height, width)
        if latents.ndim == 5:
            batch_size, channels, frames, height, width = latents.shape
            if frames == 1:
                latents = latents.squeeze(2)
            else:
                latents = latents.permute(0, 2, 1, 3, 4).contiguous()
                latents = latents.view(batch_size * frames, channels, height, width)

        if vae is not None and self._check_vae_has_bn(vae):
            latents = _unpatchify_latents(latents)
        return latents
```
**EN:** This block defines method `preprocess_decoding` on `Flux2FinetunedPipelineConfig`. Preprocess latents before decoding. Handles both standard Flux2 VAE and fine-tuned/distilled VAEs: - Standard Flux2 VAE (has bn): needs unpatchify (128 channels -> 32 channels) - Distilled/Finetuned VAE (no bn): keeps patchified latents (128 channels) Also handles 5D latents (batch, channels, frames, height, width) by converting to 4D (batch, channels, height, width) for single-frame cases. Key calls include `self._check_vae_has_bn`, `_unpatchify_latents`, `latents.squeeze`, `latents.permute.contiguous`, and `latents.view`. The implementation branches on conditions. Parameters such as `latents`, `server_args`, and `vae` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2FinetunedPipelineConfig` 的方法 `preprocess_decoding`。 它用于处理 preprocess decoding 相关逻辑。 关键调用包括 `self._check_vae_has_bn`、`_unpatchify_latents`、`latents.squeeze`、`latents.permute.contiguous` 和 `latents.view`。 实现中包含条件分支。 本段逻辑主要由 `latents`、`server_args` 和 `vae` 等参数驱动。

### Lines 73-103: `get_decode_scale_and_shift` implementation / `get_decode_scale_and_shift` 实现
```python
    def get_decode_scale_and_shift(self, device, dtype, vae):
        """
        Get scale and shift for decoding.

        Dynamically adapts based on VAE type:
        - Standard Flux2 VAE (has bn): uses BatchNorm statistics
        - Distilled/Finetuned VAE (no bn): uses scaling_factor from config

        Args:
            device: Target device for tensors
            dtype: Target dtype for tensors
            vae: VAE model instance

        Returns:
            Tuple of (scaling_factor, shift_factor)
            - scaling_factor: Tensor or scalar to divide latents by
            - shift_factor: Tensor or scalar to add to latents (None for distilled VAEs)
        """
        vae_arch_config = self.vae_config.arch_config

        if self._check_vae_has_bn(vae):
            # Standard Flux2 VAE: use BatchNorm statistics
            latents_bn_mean = vae.bn.running_mean.view(1, -1, 1, 1).to(device, dtype)
            latents_bn_std = torch.sqrt(
                vae.bn.running_var.view(1, -1, 1, 1) + vae_arch_config.batch_norm_eps
            ).to(device, dtype)
            return 1 / latents_bn_std, latents_bn_mean

        # Distilled/Finetuned VAE: Flux2TinyAutoEncoder doesn't need external scaling
        scale = torch.tensor(1.0, device=device, dtype=dtype).view(1, 1, 1, 1)
        return scale, None
```
**EN:** This block defines method `get_decode_scale_and_shift` on `Flux2FinetunedPipelineConfig`. Get scale and shift for decoding. Dynamically adapts based on VAE type: - Standard Flux2 VAE (has bn): uses BatchNorm statistics - Distilled/Finetuned VAE (no bn): uses scaling_factor from config Args: device: Target device for tensors dtype: Target dtype for tensors vae: VAE model instance Returns: Tuple of (scaling_factor, shift_factor) - scaling_factor: Tensor or scalar to divide latents by - shift_factor: Tensor or scalar to add to latents (None for distilled VAEs) Key calls include `self._check_vae_has_bn`, `torch.tensor.view`, `vae.bn.running_mean.view.to`, `torch.sqrt.to`, and `torch.tensor`. The implementation branches on conditions. Parameters such as `device`, `dtype`, and `vae` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2FinetunedPipelineConfig` 的方法 `get_decode_scale_and_shift`。 它用于获取decode scale and shift。 关键调用包括 `self._check_vae_has_bn`、`torch.tensor.view`、`vae.bn.running_mean.view.to`、`torch.sqrt.to` 和 `torch.tensor`。 实现中包含条件分支。 本段逻辑主要由 `device`、`dtype` 和 `vae` 等参数驱动。

## Key Concepts / 关键概念
- `Flux2FinetunedPipelineConfig`: Pipeline configuration for Flux fine-tuned/distilled models. / 核心类，用于封装 flux2 finetuned pipeline config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.pipeline_configs.flux`

- **Total lines / 总行数**: 103
