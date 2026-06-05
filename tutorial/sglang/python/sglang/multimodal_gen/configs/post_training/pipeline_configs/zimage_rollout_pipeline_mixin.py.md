# zimage_rollout_pipeline_mixin.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/post_training/pipeline_configs/zimage_rollout_pipeline_mixin.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `ZImageRolloutPipelineMixin`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Rollout / RL hooks for Z-Image pipeline configs. / 该文件属于配置层。它围绕 `ZImageRolloutPipelineMixin` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 2-11: module setup and imports / 模块初始化与导入
```python
"""Rollout / RL hooks for Z-Image pipeline configs."""

from __future__ import annotations

import torch

from sglang.multimodal_gen.runtime.post_training.sp_utils import (
    all_gather_if_sp_sharded,
    maybe_trim_sp_rope_seq_for_batch,
)
```
**EN:** This block establishes the module context and imports `__future__`, `torch`, and `sglang.multimodal_gen.runtime.post_training.sp_utils`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `__future__`、`torch` 和 `sglang.multimodal_gen.runtime.post_training.sp_utils`。这些依赖为后续实现提供所需符号。

### Lines 14-15: `ZImageRolloutPipelineMixin` class overview / `ZImageRolloutPipelineMixin` 类概览
```python
class ZImageRolloutPipelineMixin:
```
**EN:** This block defines class `ZImageRolloutPipelineMixin`. It encapsulates zimage rollout pipeline mixin behavior.
**CN:** 该代码块定义了类 `ZImageRolloutPipelineMixin`。 它用于封装 zimage rollout pipeline mixin 相关行为。

### Lines 16-27: `gather_denoising_env_static_for_sp` implementation / `gather_denoising_env_static_for_sp` 实现
```python
    def gather_denoising_env_static_for_sp(self, batch, cond_kwargs: dict | None):
        if cond_kwargs is None:
            return None
        out = dict(cond_kwargs)
        freqs = out.get("freqs_cis")
        if freqs is not None:
            cap_freqs, x_freqs = freqs[0], freqs[1]
            if isinstance(x_freqs, torch.Tensor) and x_freqs.dim() >= 2:
                x_g = all_gather_if_sp_sharded(batch, x_freqs, dim=0)
                x_g = maybe_trim_sp_rope_seq_for_batch(batch, x_g)
                out["freqs_cis"] = (cap_freqs, x_g)
        return out
```
**EN:** This block defines method `gather_denoising_env_static_for_sp` on `ZImageRolloutPipelineMixin`. It handles gather denoising env static for sp logic. Key calls include `dict`, `out.get`, `isinstance`, `all_gather_if_sp_sharded`, and `maybe_trim_sp_rope_seq_for_batch`. The implementation branches on conditions. Parameters such as `batch`, and `cond_kwargs` drive the behavior in this section.
**CN:** 该代码块定义了 `ZImageRolloutPipelineMixin` 的方法 `gather_denoising_env_static_for_sp`。 它用于处理 gather denoising env static for sp 相关逻辑。 关键调用包括 `dict`、`out.get`、`isinstance`、`all_gather_if_sp_sharded` 和 `maybe_trim_sp_rope_seq_for_batch`。 实现中包含条件分支。 本段逻辑主要由 `batch` 和 `cond_kwargs` 等参数驱动。

## Key Concepts / 关键概念
- `ZImageRolloutPipelineMixin`: Primary class that encapsulates zimage rollout pipeline mixin behavior. / 核心类，用于封装 zimage rollout pipeline mixin 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.post_training.sp_utils`

- **Total lines / 总行数**: 27
