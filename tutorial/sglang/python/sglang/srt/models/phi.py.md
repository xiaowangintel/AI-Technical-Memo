# phi.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/phi.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the phi model stack used by SGLang SRT inference, including network blocks, runtime wrappers, and checkpoint-loading utilities. / 该模块实现 SGLang SRT 推理所使用的 phi 模型栈，包括网络模块、运行时封装以及检查点加载工具。

## Line-by-Line Analysis / 逐行分析

### Lines 4-27: Module imports
```python
from typing import Iterable, Optional

import torch
from torch import nn
from transformers import PhiConfig

from sglang.srt.distributed import get_pp_group, get_tensor_model_parallel_world_size
from sglang.srt.layers.activation import get_act_fn
from sglang.srt.layers.linear import (
    ColumnParallelLinear,
    QKVParallelLinear,
    RowParallelLinear,
)
from sglang.srt.layers.logits_processor import LogitsProcessor, LogitsProcessorOutput
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.layers.radix_attention import RadixAttention
from sglang.srt.layers.rotary_embedding import get_rope
from sglang.srt.layers.vocab_parallel_embedding import (
    ParallelLMHead,
    VocabParallelEmbedding,
)
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.model_loader.weight_utils import default_weight_loader
from sglang.srt.utils import add_prefix, make_layers
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 30-31: Class `PhiAttention` overview
```python
class PhiAttention(nn.Module):
```
**EN:** Defines `PhiAttention` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `PhiAttention`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 32-84: Method `PhiAttention.__init__`
```python
    def __init__(
        self,
        config: PhiConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        layer_id: int = 0,
    ):
        super().__init__()
        self.total_num_heads = config.num_attention_heads
        self.hidden_size = config.hidden_size
        self.head_size = self.hidden_size // self.total_num_heads

        tensor_model_parallel_world_size = get_tensor_model_parallel_world_size()
        assert self.total_num_heads % tensor_model_parallel_world_size == 0
        self.num_heads = self.total_num_heads // tensor_model_parallel_world_size

        self.qkv_proj = QKVParallelLinear(
            self.hidden_size,
            self.head_size,
            self.total_num_heads,
            bias=True,
            quant_config=quant_config,
        )
        self.dense = RowParallelLinear(
            self.hidden_size,
            self.hidden_size,
            quant_config=quant_config,
        )

        scaling = self.head_size**-0.5
        rotary_dim = int(
            config.partial_rotary_factor
            * (config.hidden_size // config.num_attention_heads)
        )
        assert rotary_dim % 2 == 0

        rope_theta = config.rope_parameters["rope_theta"]
        max_position_embeddings = getattr(config, "max_position_embeddings", 2048)
        self.rotary_emb = get_rope(
            self.head_size,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=..., layer_id: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=..., layer_id: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 86-97: Method `PhiAttention.forward`
```python
    def forward(
        self,
        position_ids: torch.Tensor,
        forward_batch: ForwardBatch,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.chunk(chunks=3, dim=-1)
        q, k = self.rotary_emb(position_ids, q, k)
        attn_output = self.attn(q, k, v, forward_batch=forward_batch)
        output, _ = self.dense(attn_output)
        return output
```
**EN:** This method implements `forward(position_ids: ..., forward_batch: ..., hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(position_ids: ..., forward_batch: ..., hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 100-101: Class `PhiMLP` overview
```python
class PhiMLP(nn.Module):
```
**EN:** Defines `PhiMLP` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `PhiMLP`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 102-120: Method `PhiMLP.__init__`
```python
    def __init__(
        self, config: PhiConfig, quant_config: Optional[QuantizationConfig] = None
    ):
        super().__init__()

        n_inner = getattr(config, "n_inner", None)
        n_inner = n_inner if n_inner is not None else 4 * config.hidden_size

        self.fc1 = ColumnParallelLinear(
            config.hidden_size,
            n_inner,
            quant_config=quant_config,
        )
        self.fc2 = RowParallelLinear(
            n_inner,
            config.hidden_size,
            quant_config=quant_config,
        )
        self.act = get_act_fn(config.hidden_act)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 122-126: Method `PhiMLP.forward`
```python
    def forward(self, hidden_states):
        hidden_states, _ = self.fc1(hidden_states)
        hidden_states = self.act(hidden_states)
        hidden_states, _ = self.fc2(hidden_states)
        return hidden_states
```
**EN:** This method implements `forward(hidden_states)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 129-130: Class `PhiLayer` overview
```python
class PhiLayer(nn.Module):
```
**EN:** Defines `PhiLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `PhiLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 131-148: Method `PhiLayer.__init__`
```python
    def __init__(
        self,
        config: PhiConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        idx: int = 0,
    ):
        super().__init__()
        self.input_layernorm = nn.LayerNorm(
            config.hidden_size, eps=config.layer_norm_eps
        )
        self.self_attn = PhiAttention(
            config,
            quant_config,
            prefix=add_prefix("self_attn", prefix),
            layer_id=idx,
        )
        self.mlp = PhiMLP(config, quant_config)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=..., idx: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=..., idx: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 150-165: Method `PhiLayer.forward`
```python
    def forward(
        self,
        position_ids: torch.Tensor,
        forward_batch: ForwardBatch,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        residual = hidden_states
        hidden_states = self.input_layernorm(hidden_states)
        attn_outputs = self.self_attn(
            position_ids=position_ids,
            hidden_states=hidden_states,
            forward_batch=forward_batch,
        )
        feed_forward_hidden_states = self.mlp(hidden_states)
        hidden_states = attn_outputs + feed_forward_hidden_states + residual
        return hidden_states
```
**EN:** This method implements `forward(position_ids: ..., forward_batch: ..., hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(position_ids: ..., forward_batch: ..., hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 168-169: Class `PhiModel` overview
```python
class PhiModel(nn.Module):
```
**EN:** Defines `PhiModel` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `PhiModel`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 170-199: Method `PhiModel.__init__`
```python
    def __init__(
        self,
        config: PhiConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size, config.hidden_size
        )

        pp_group = get_pp_group()
        pp_size = pp_group.world_size
        pp_rank = pp_group.rank

        self.start_layer = pp_rank * config.num_hidden_layers // pp_size
        self.end_layer = (pp_rank + 1) * config.num_hidden_layers // pp_size

        self.layers = make_layers(
            config.num_hidden_layers,
            lambda idx, prefix: PhiLayer(
                config, quant_config=quant_config, prefix=prefix, idx=idx
            ),
            prefix=add_prefix("layers", prefix),
        )

        self.final_layernorm = nn.LayerNorm(
            config.hidden_size, eps=config.layer_norm_eps
        )
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 201-202: Method `PhiModel.get_input_embeddings`
```python
    def get_input_embeddings(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.embed_tokens(input_ids)
```
**EN:** This method implements `get_input_embeddings(input_ids: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings(input_ids: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 204-224: Method `PhiModel.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        forward_batch: ForwardBatch,
        positions: torch.Tensor,
        inputs_embeds: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        if inputs_embeds is not None:
            hidden_states = inputs_embeds
        else:
            hidden_states = self.get_input_embeddings(input_ids)
        for i in range(self.start_layer, self.end_layer):
            layer = self.layers[i]

            hidden_states = layer(
                position_ids=positions,
                forward_batch=forward_batch,
                hidden_states=hidden_states,
            )
        hidden_states = self.final_layernorm(hidden_states)
        return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., forward_batch: ..., positions: ..., inputs_embeds: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., forward_batch: ..., positions: ..., inputs_embeds: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 227-227: Class `PhiForCausalLM` overview
```python
class PhiForCausalLM(nn.Module):
```
**EN:** Defines `PhiForCausalLM` as a reusable runtime type derived from nn.Module. The class groups 4 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `PhiForCausalLM`，其继承关系为 nn.Module。这个类组织了 4 个方法，用于实现模型相关行为。

### Lines 228-234: Class `PhiForCausalLM` attributes
```python
    packed_modules_mapping = {
        "qkv_proj": [
            "q_proj",
            "k_proj",
            "v_proj",
        ]
    }
```
**EN:** Defines class-level attributes and metadata that shape how `PhiForCausalLM` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `PhiForCausalLM` 在运行时的行为。

### Lines 236-257: Method `PhiForCausalLM.__init__`
```python
    def __init__(
        self,
        config: PhiConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.quant_config = quant_config
        self.model = PhiModel(
            config=config,
            quant_config=quant_config,
            prefix=add_prefix("model", prefix),
        )

        self.lm_head = ParallelLMHead(
            config.vocab_size,
            config.hidden_size,
            bias=True,
            quant_config=quant_config,
        )
        self.logits_processor = LogitsProcessor(config)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 259-260: Method `PhiForCausalLM.get_input_embeddings`
```python
    def get_input_embeddings(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.model.get_input_embeddings(input_ids)
```
**EN:** This method implements `get_input_embeddings(input_ids: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings(input_ids: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 262-279: Method `PhiForCausalLM.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        inputs_embeds: Optional[torch.Tensor] = None,
    ) -> LogitsProcessorOutput:

        hidden_states = self.model(
            input_ids=input_ids,
            forward_batch=forward_batch,
            positions=positions,
            inputs_embeds=inputs_embeds,
        )

        return self.logits_processor(
            input_ids, hidden_states, self.lm_head, forward_batch
        )
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., inputs_embeds: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., inputs_embeds: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 281-320: Method `PhiForCausalLM.load_weights`
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]):
        params_dict = dict(self.named_parameters())
        weights = dict(weights)
        loaded_keys = set()

        for name, param in params_dict.items():
            if name in loaded_keys:
                continue

            # Handle packed weights
            is_packed = False
            for packed_name, src_names in self.packed_modules_mapping.items():
                if packed_name not in name:
                    continue

                weight_loader = getattr(param, "weight_loader", default_weight_loader)
                for src_name in src_names:
                    full_src_name = name.replace(packed_name, src_name)
                    if full_src_name in weights:
                        loaded_weight = weights[full_src_name]
                        # The shard_id for QKVParallelLinear is 'q', 'k', 'v'.
                        shard_id = src_name.split("_")[0]
                        weight_loader(param, loaded_weight, shard_id)
                        loaded_keys.add(full_src_name)

                loaded_keys.add(name)
                is_packed = True
                break
            if is_packed:
                continue

            # Handle non-packed weights
            if name not in weights:
                # Redundant with the check in the loop, but good for safety
                continue

            loaded_weight = weights[name]
            weight_loader = getattr(param, "weight_loader", default_weight_loader)
            weight_loader(param, loaded_weight)
            loaded_keys.add(name)
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 323-323: Top-level assign
```python
EntryClass = PhiForCausalLM
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
- `typing.Optional`
- `torch`
- `torch.nn`
- `transformers.PhiConfig`
- `sglang.srt.distributed.get_pp_group`
- `sglang.srt.distributed.get_tensor_model_parallel_world_size`
- `sglang.srt.layers.activation.get_act_fn`
- `sglang.srt.layers.linear.ColumnParallelLinear`
- `sglang.srt.layers.linear.QKVParallelLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.logits_processor.LogitsProcessorOutput`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.layers.rotary_embedding.get_rope`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.make_layers`
