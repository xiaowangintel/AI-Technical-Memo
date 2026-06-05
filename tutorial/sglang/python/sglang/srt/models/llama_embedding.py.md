# llama_embedding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/llama_embedding.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the llama embedding task-specific model components used by SGLang inference, together with runtime wrappers and weight-loading paths. / 该模块实现 SGLang 推理中 llama embedding 的任务特定模型组件，以及运行时封装与权重加载路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11: Module imports
```python
from typing import Iterable, Tuple

import torch
from torch import nn
from transformers import LlamaConfig

from sglang.srt.layers.pooler import EmbeddingPoolerOutput, Pooler, PoolingType
from sglang.srt.model_executor.model_runner import ForwardBatch
from sglang.srt.model_loader.weight_utils import default_weight_loader
from sglang.srt.models.llama import LlamaModel
from sglang.srt.utils import add_prefix
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 14-14: Class `LlamaEmbeddingModel` overview
```python
class LlamaEmbeddingModel(nn.Module):
```
**EN:** Defines `LlamaEmbeddingModel` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `LlamaEmbeddingModel`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 15-25: Method `LlamaEmbeddingModel.__init__`
```python
    def __init__(
        self,
        config: LlamaConfig,
        quant_config=None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.model = LlamaModel(
            config, quant_config=quant_config, prefix=add_prefix("model", prefix)
        )
        self.pooler = Pooler(pooling_type=PoolingType.LAST, normalize=True)
```
**EN:** This method implements `__init__(config: ..., quant_config=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 27-40: Method `LlamaEmbeddingModel.forward`
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
        assert (
            get_embedding
        ), "LlamaEmbeddingModel / MistralModel is only used for embedding"
        hidden_states = self.model(input_ids, positions, forward_batch, input_embeds)
        return self.pooler(hidden_states, forward_batch)
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., get_embedding: ...=...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., get_embedding: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 42-80: Method `LlamaEmbeddingModel.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
        ]
        params_dict = dict(self.model.named_parameters())

        for name, loaded_weight in weights:
            if "rotary_emb.inv_freq" in name or "projector" in name:
                return
            if "rotary_emb.cos_cached" in name or "rotary_emb.sin_cached" in name:
                # Models trained using ColossalAI may include these tensors in
                # the checkpoint. Skip them.
                return
            if name.startswith("model.vision_tower") and name not in params_dict:
                return

            for param_name, weight_name, shard_id in stacked_params_mapping:
                if weight_name not in name:
                    continue
                name = name.replace(weight_name, param_name)
                # Skip loading extra bias for GPTQ models.
                if name.endswith(".bias") and name not in params_dict:
                    continue
                param = params_dict[name]
                weight_loader = param.weight_loader
                weight_loader(param, loaded_weight, shard_id)
                break
            else:
                # Skip loading extra bias for GPTQ models.
                if name.endswith(".bias") and name not in params_dict:
                    return
                param = params_dict[name]
                weight_loader = getattr(param, "weight_loader", default_weight_loader)
                weight_loader(param, loaded_weight)
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 83-83: Class `MistralModel` overview
```python
class MistralModel(LlamaEmbeddingModel):
```
**EN:** Defines `MistralModel` as a reusable runtime type derived from LlamaEmbeddingModel. The class groups 0 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MistralModel`，其继承关系为 LlamaEmbeddingModel。这个类组织了 0 个方法，用于实现模型相关行为。

### Lines 84-84: Class `MistralModel` support logic
```python
    pass
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 87-87: Top-level assign
```python
EntryClass = [LlamaEmbeddingModel, MistralModel]
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `typing.Iterable`
- `typing.Tuple`
- `torch`
- `torch.nn`
- `transformers.LlamaConfig`
- `sglang.srt.layers.pooler.EmbeddingPoolerOutput`
- `sglang.srt.layers.pooler.Pooler`
- `sglang.srt.layers.pooler.PoolingType`
- `sglang.srt.model_executor.model_runner.ForwardBatch`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.models.llama.LlamaModel`
- `sglang.srt.utils.add_prefix`
