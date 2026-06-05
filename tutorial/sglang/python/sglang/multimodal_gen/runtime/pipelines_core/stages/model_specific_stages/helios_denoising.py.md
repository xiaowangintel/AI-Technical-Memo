# helios_denoising.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/model_specific_stages/helios_denoising.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for helios denoising, connecting stages, tensors, and runtime metadata. Key symbols include `optimized_scale`, `calculate_shift`, `sample_block_noise`. / 该模块负责 helios denoising 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `optimized_scale`, `calculate_shift`, `sample_block_noise`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
"""
Helios-specific chunked denoising stage.

Implements Stage 1 chunked denoising with multi-term memory history
and CFG Zero Star guidance. VAE decoding is handled by the standard
DecodingStage downstream.
"""

import math

import numpy as np
import torch
import torch.nn.functional as F
# ...
from sglang.multimodal_gen.runtime.utils.perf_logger import StageProfiler
from sglang.multimodal_gen.utils import PRECISION_TO_TYPE

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 36-42: Function `optimized_scale` / 函数 `optimized_scale`
```python
def optimized_scale(positive_flat, negative_flat):
    """CFG Zero Star: compute optimal guidance scale."""
    positive_flat = positive_flat.float()
    negative_flat = negative_flat.float()
    dot_product = torch.sum(positive_flat * negative_flat, dim=1, keepdim=True)
    squared_norm = torch.sum(negative_flat**2, dim=1, keepdim=True) + 1e-8
    return dot_product / squared_norm
```
**EN:** This function drives `optimized_scale` with inputs such as `positive_flat`, `negative_flat`. CFG Zero Star: compute optimal guidance scale.
**CN:** 这个函数负责 `optimized_scale`，主要处理 `positive_flat`, `negative_flat` 等输入。 文档字符串说明：CFG Zero Star: compute optimal guidance scale.

### Lines 45-55: Function `calculate_shift` / 函数 `calculate_shift`
```python
def calculate_shift(
    image_seq_len,
    base_seq_len: int = 256,
    max_seq_len: int = 4096,
    base_shift: float = 0.5,
    max_shift: float = 1.15,
):
    m = (max_shift - base_shift) / (max_seq_len - base_seq_len)
    b = base_shift - m * base_seq_len
    mu = image_seq_len * m + b
    return mu
```
**EN:** This function drives `calculate_shift` with inputs such as `image_seq_len`, `base_seq_len`, `max_seq_len`, `base_shift`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `calculate_shift`，主要处理 `image_seq_len`, `base_seq_len`, `max_seq_len`, `base_shift` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 58-91: Function `sample_block_noise` / 函数 `sample_block_noise`
```python
def sample_block_noise(
    batch_size, channel, num_frames, height, width, gamma, patch_size=(1, 2, 2)
):
    """Generate spatially-correlated block noise for pyramid SR."""
    _, ph, pw = patch_size
    block_size = ph * pw

    # Explicitly use CPU to avoid requiring MAGMA on ROCm/CUDA.
    #
    # For the default Helios stage-2 setting gamma=1/3 with a 2x2 block, the
    # covariance has eigenvalues {0, 1+gamma, 1+gamma, 1+gamma} and is therefore
    # only positive semidefinite. `MultivariateNormal(covariance_matrix=...)`
    # requires a strictly positive-definite matrix and fails in the Cholesky
    # factorization path, so sample from the PSD covariance via eigen-decomposition.
# ...
    noise = noise.permute(0, 1, 2, 3, 5, 4, 6).reshape(
        batch_size, channel, num_frames, height, width
    )
    return noise
```
**EN:** This function drives `sample_block_noise` with inputs such as `batch_size`, `channel`, `num_frames`, `height`. Generate spatially-correlated block noise for pyramid SR.
**CN:** 这个函数负责 `sample_block_noise`，主要处理 `batch_size`, `channel`, `num_frames`, `height` 等输入。 文档字符串说明：Generate spatially-correlated block noise for pyramid SR.

### Lines 94-750: Class `HeliosChunkedDenoisingStage` / 类 `HeliosChunkedDenoisingStage`
```python
class HeliosChunkedDenoisingStage(PipelineStage):
    """
    Helios chunked denoising stage implementing Stage 1 loop.

    Iterates over video chunks, manages history buffers (short/mid/long),
    runs transformer per chunk with CFG guidance, scheduler step,
    and accumulates denoised latents. VAE decoding is left to DecodingStage.
    """

    def __init__(self, transformer, scheduler):
        super().__init__()
        self.transformer = transformer
        self.scheduler = scheduler

# ...
        batch.latent_chunks = chunk_latents_list
        batch.latents = history_latents[:, :, -total_generated_latent_frames:]

        return batch
```
**EN:** This class models `HeliosChunkedDenoisingStage` as a specialization of `PipelineStage`. Helios chunked denoising stage implementing Stage 1 loop. Important methods include `__init__`, `parallelism_type`, `component_uses`, `_denoise_one_chunk`.
**CN:** 该类实现 `HeliosChunkedDenoisingStage`，并继承/扩展 `PipelineStage`。 文档字符串指出：Helios chunked denoising stage implementing Stage 1 loop. 其中较重要的方法包括 `__init__`, `parallelism_type`, `component_uses`, `_denoise_one_chunk`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Caching strategy / 缓存策略
- Sampling parameter control / 采样参数控制

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.managers.forward_context`, `sglang.multimodal_gen.runtime.managers.memory_managers.component_manager`, `sglang.multimodal_gen.runtime.pipelines_core.diffusion_scheduler_utils`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages.base`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.runtime.utils.perf_logger`
- **External / 外部**: `numpy`, `torch`, `torch.nn.functional`
- **Stdlib / 标准库**: `math`
