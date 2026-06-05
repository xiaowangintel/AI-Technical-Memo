# voyage.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/voyage.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Voyage model components and runtime adapter for vLLM inference. / 实现 Voyage 在 vLLM 推理中的模型组件与运行时适配器。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-15)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
from __future__ import annotations

from collections import defaultdict
from collections.abc import Iterable

import regex as re
import torch
import torch.nn as nn

from vllm.model_executor.model_loader.weight_utils import default_weight_loader
from vllm.model_executor.models.qwen3 import Qwen3Model
from vllm.model_executor.models.utils import WeightsMapper
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as regex, torch, torch.nn supply framework primitives, while internal modules like vllm.model_executor.model_loader.weight_utils, vllm.model_executor.models.qwen3, vllm.model_executor.models.utils connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 regex, torch, torch.nn 这样的外部依赖提供基础框架能力，而 vllm.model_executor.model_loader.weight_utils, vllm.model_executor.models.qwen3, vllm.model_executor.models.utils 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Top-level mapping `WeightItem, _LAYER_RE` (lines 16-18)
```python
WeightItem = tuple[str, torch.Tensor]

_LAYER_RE = re.compile(r"^layers\.(\d+)\.(.+)$")
```
**EN:** This assignment block centers on `WeightItem, _LAYER_RE` and records module-level configuration, dispatch metadata, or constant defaults used later in the file.
**CN:** 这个赋值代码块以 `WeightItem, _LAYER_RE` 为核心，记录了后续逻辑会使用的模块级配置、分发表元数据或常量默认值。

### Class `VoyageQwen3BidirectionalEmbedModel` (lines 21-130)
```python
class VoyageQwen3BidirectionalEmbedModel(Qwen3Model):
    """
    Qwen3Model + Voyage embedding head + bidirectional attention.

    Checkpoint conventions (HF):
      - MLP: gate_proj + up_proj (unfused)
      - Attn: q_proj + k_proj + v_proj (unfused)
      - Linear head: linear.weight
      - Weights prefixed with "model." (e.g., model.layers.0...)
# ... omitted for brevity ...
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

        # Embedding head (hidden_size -> num_labels, bias=False)
        self.linear = nn.Linear(
            self.config.hidden_size,
            self.config.num_labels,
            bias=False,
        )
# ... omitted for brevity ...
    def forward(self, *args, **kwargs):
        out = super().forward(*args, **kwargs)
        return self.linear(out)
# ... omitted for brevity ...
    def _fuse_qkv_proj(self, weights: Iterable[WeightItem]) -> Iterable[WeightItem]:
        """Fuse q_proj, k_proj, v_proj into qkv_proj."""
        qkv_buf: dict[int, dict[str, torch.Tensor]] = defaultdict(dict)
        qkv_suffixes = {
            "self_attn.q_proj.weight": "q",
            "self_attn.k_proj.weight": "k",
            "self_attn.v_proj.weight": "v",
        }

        for name, tensor in weights:
            m = _LAYER_RE.match(name)
            if m and m.group(2) in qkv_suffixes:
                layer_idx = int(m.group(1))
                qkv_buf[layer_idx][qkv_suffixes[m.group(2)]] = tensor
            else:
                yield name, tensor

        # Yield fused QKV weights
        for layer_idx in sorted(qkv_buf.keys()):
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[WeightItem]) -> set[str]:
        """Remap, fuse, and load weights using generator pipeline."""
        # Chain weight transformations
        weights = self.hf_to_vllm_mapper.apply(weights)
        weights = self._fuse_qkv_proj(weights)
        weights = self._fuse_gate_up_proj(weights)

        # Load weights directly into model parameters
        # (bypass parent's stacked_params_mapping)
        params_dict = dict(self.named_parameters())
        loaded_params: set[str] = set()

        for name, loaded_weight in weights:
            if name not in params_dict:
                continue
            param = params_dict[name]
            weight_loader = getattr(param, "weight_loader", default_weight_loader)
            weight_loader(param, loaded_weight)
            loaded_params.add(name)
```
**EN:** Defines `VoyageQwen3BidirectionalEmbedModel`, a backbone model container that orchestrates embeddings, layers, and output heads. It inherits from Qwen3Model. Key methods such as `__init__`, `forward`, `_fuse_qkv_proj`, `_fuse_gate_up_proj`, `load_weights` show where construction, forward execution, or weight adaptation happens. Docstring hint: "Qwen3Model + Voyage embedding head + bidirectional attention."
**CN:** 定义 `VoyageQwen3BidirectionalEmbedModel`，它是一个协调嵌入、层堆叠与输出头的主干模型容器。 它继承自 Qwen3Model。 `__init__`, `forward`, `_fuse_qkv_proj`, `_fuse_gate_up_proj`, `load_weights` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“Qwen3Model + Voyage embedding head + bidirectional attention。”

## Key Concepts / 关键概念
- **Attention and KV cache**
  - **EN:** Core logic manages query/key/value projection and integrates with vLLM attention backends and cache layout.
  - **CN:** 核心逻辑管理 Q/K/V 投影，并与 vLLM 的注意力后端及 KV Cache 布局集成。
- **Checkpoint remapping**
  - **EN:** Weight-loading helpers translate Hugging Face checkpoints into the parameter names and tensor layout expected by vLLM.
  - **CN:** 权重加载辅助逻辑会把 Hugging Face 检查点转换成 vLLM 期望的参数名和张量布局。

## Dependencies / 依赖关系
- **Standard library**: __future__, collections, collections.abc
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: regex, torch, torch.nn
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.model_executor.model_loader.weight_utils, vllm.model_executor.models.qwen3, vllm.model_executor.models.utils
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
