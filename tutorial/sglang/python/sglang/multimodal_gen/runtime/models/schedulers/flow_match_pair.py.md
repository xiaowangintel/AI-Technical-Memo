# flow_match_pair.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/schedulers/flow_match_pair.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for flow match pair within the multimodal runtime. Key symbols include `FlowMatchScheduler`, `FlowMatchPairScheduler`. / 该模块实现多模态运行时中与 flow match pair 相关的模型构件。 关键符号包括 `FlowMatchScheduler`, `FlowMatchPairScheduler`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/OpenMOSS/MOVA/tree/main/mova/diffusion/schedulers/flow_match.py and flow_match_pair.py
# SPDX-License-Identifier: Apache-2.0

from __future__ import annotations

import math

import torch

from sglang.multimodal_gen.runtime.models.schedulers.base import BaseScheduler
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 13-159: Class `FlowMatchScheduler` / 类 `FlowMatchScheduler`
```python
class FlowMatchScheduler(BaseScheduler):
    def __init__(
        self,
        num_inference_steps=100,
        num_train_timesteps=1000,
        shift=3.0,
        sigma_max=1.0,
        sigma_min=0.003 / 1.002,
        inverse_timesteps=False,
        extra_one_step=False,
        reverse_sigmas=False,
        exponential_shift=False,
        exponential_shift_mu=None,
        shift_terminal=None,
# ...
        m = (max_shift - base_shift) / (max_seq_len - base_seq_len)
        b = base_shift - m * base_seq_len
        mu = image_seq_len * m + b
        return mu
```
**EN:** This class models `FlowMatchScheduler` as a specialization of `BaseScheduler`. Important methods include `__init__`, `set_shift`, `set_timesteps`, `scale_model_input`.
**CN:** 该类实现 `FlowMatchScheduler`，并继承/扩展 `BaseScheduler`。 其中较重要的方法包括 `__init__`, `set_shift`, `set_timesteps`, `scale_model_input`。

### Lines 162-528: Class `FlowMatchPairScheduler` / 类 `FlowMatchPairScheduler`
```python
class FlowMatchPairScheduler(FlowMatchScheduler):
    """Pairing scheduler built on FlowMatchScheduler.

    Provides a convenient pairing interface for timesteps or sigmas.

    Attributes:
        pair_timesteps: Cached timestep pairs of shape [num_timesteps, 2].
        pair_sigmas: Cached sigma pairs of shape [num_timesteps, 2].
    """

    def __init__(
        self,
        num_inference_steps=100,
        num_train_timesteps=1000,
# ...
        base_pairs_sigmas = self._make_pairs_from_vector(self.sigmas)

        self.pair_timesteps = _apply_postprocess(base_pairs_timesteps, "timesteps")
        self.pair_sigmas = _apply_postprocess(base_pairs_sigmas, "sigmas")
```
**EN:** This class models `FlowMatchPairScheduler` as a specialization of `FlowMatchScheduler`. Pairing scheduler built on FlowMatchScheduler. Important methods include `__init__`, `set_pair_postprocess`, `set_pair_postprocess_by_name`, `_make_pairs_from_vector`.
**CN:** 该类实现 `FlowMatchPairScheduler`，并继承/扩展 `FlowMatchScheduler`。 文档字符串指出：Pairing scheduler built on FlowMatchScheduler. 其中较重要的方法包括 `__init__`, `set_pair_postprocess`, `set_pair_postprocess_by_name`, `_make_pairs_from_vector`。

### Lines 529-531: Top-level configuration / 顶层配置
```python


EntryClass = FlowMatchPairScheduler
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Scheduling and batching / 调度与批处理
- Caching strategy / 缓存策略
- Image generation flow / 图像生成流程
- Audio generation flow / 音频生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.models.schedulers.base`
- **External / 外部**: `__future__`, `torch`
- **Stdlib / 标准库**: `math`
