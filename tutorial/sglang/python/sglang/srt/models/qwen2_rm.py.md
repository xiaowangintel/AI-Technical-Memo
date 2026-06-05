# qwen2_rm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/qwen2_rm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the qwen2 rm model stack used by SGLang SRT inference, including network blocks, runtime wrappers, and checkpoint-loading utilities. / 该模块实现 SGLang SRT 推理所使用的 qwen2 rm 模型栈，包括网络模块、运行时封装以及检查点加载工具。

## Line-by-Line Analysis / 逐行分析

### Lines 15-30: Module imports
```python
from typing import Iterable, Optional, Tuple

import torch
from torch import nn
from transformers import Qwen2Config

from sglang.srt.layers.pooler import (
    EmbeddingPoolerOutput,
    Pooler,
    PoolingType,
    pool_hidden_states,
)
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.models.qwen2 import Qwen2ForCausalLM, Qwen2Model
from sglang.srt.utils import add_prefix
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 33-33: Class `Qwen2ForRewardModel` overview
```python
class Qwen2ForRewardModel(nn.Module):
```
**EN:** Defines `Qwen2ForRewardModel` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen2ForRewardModel`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 34-54: Method `Qwen2ForRewardModel.__init__`
```python
    def __init__(
        self,
        config: Qwen2Config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.quant_config = quant_config
        self.num_labels = 1
        self.model = Qwen2Model(
            config, quant_config=quant_config, prefix=add_prefix("model", prefix)
        )
        self.score = nn.Sequential(
            nn.Linear(config.hidden_size, config.hidden_size),
            nn.ReLU(),
            nn.Linear(config.hidden_size, self.num_labels),
        )
        self.pooler = Pooler(pooling_type=PoolingType.LAST, normalize=False)

        self.eos_token_id = config.eos_token_id
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 56-80: Method `Qwen2ForRewardModel.forward`
```python
    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
        get_embedding: bool = True,
    ) -> EmbeddingPoolerOutput:
        assert get_embedding, "Qwen2ForRewardModel is only used for embedding"

        hidden_states = self.model(input_ids, positions, forward_batch, input_embeds)
        logits = self.score(hidden_states)
        pooled_logits = self.pooler(logits, forward_batch).embeddings

        pooled_hidden = None
        if forward_batch.return_pooled_hidden_states:
            pooled_hidden = pool_hidden_states(
                self.pooler.pooling_type, hidden_states, forward_batch
            )

        return EmbeddingPoolerOutput(
            embeddings=pooled_logits,
            pooled_hidden_states=pooled_hidden,
        )
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., get_embedding: ...=...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., get_embedding: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 82-87: Method `Qwen2ForRewardModel.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        # Filter out lm_head weights of Qwen2ForCausalLM
        filtered_weights = [
            (name, w) for name, w in weights if not name.startswith("lm_head")
        ]
        return Qwen2ForCausalLM.load_weights(self, filtered_weights)
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 90-92: Top-level assign
```python
EntryClass = [
    Qwen2ForRewardModel,
]
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影
- **EN:** Checkpoint remapping and weight loading / **CN:** 检查点重映射与权重加载
- **EN:** Dynamic model registration / **CN:** 动态模型注册

## Dependencies / 依赖关系
- `typing.Iterable`
- `typing.Optional`
- `typing.Tuple`
- `torch`
- `torch.nn`
- `transformers.Qwen2Config`
- `sglang.srt.layers.pooler.EmbeddingPoolerOutput`
- `sglang.srt.layers.pooler.Pooler`
- `sglang.srt.layers.pooler.PoolingType`
- `sglang.srt.layers.pooler.pool_hidden_states`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.models.qwen2.Qwen2ForCausalLM`
- `sglang.srt.models.qwen2.Qwen2Model`
- `sglang.srt.utils.add_prefix`
