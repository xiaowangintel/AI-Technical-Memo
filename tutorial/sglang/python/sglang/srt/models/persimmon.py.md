# persimmon.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/persimmon.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the persimmon model stack used by SGLang SRT inference, including network blocks, runtime wrappers, and checkpoint-loading utilities. / 该模块实现 SGLang SRT 推理所使用的 persimmon 模型栈，包括网络模块、运行时封装以及检查点加载工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-26: Module imports
```python
from collections.abc import Iterable
from typing import Optional

import torch
from torch import nn
from transformers import PersimmonConfig

from sglang.srt.distributed import get_pp_group, get_tensor_model_parallel_world_size
from sglang.srt.layers.activation import get_act_fn
from sglang.srt.layers.linear import (
    ColumnParallelLinear,
    QKVParallelLinear,
    RowParallelLinear,
)
from sglang.srt.layers.logits_processor import LogitsProcessor, LogitsProcessorOutput
from sglang.srt.layers.quantization import QuantizationConfig
from sglang.srt.layers.radix_attention import RadixAttention
from sglang.srt.layers.rotary_embedding import get_rope
from sglang.srt.layers.utils import PPMissingLayer
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

### Lines 29-30: Class `PersimmonMLP` overview
```python
class PersimmonMLP(nn.Module):
```
**EN:** Defines `PersimmonMLP` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `PersimmonMLP`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 31-41: Method `PersimmonMLP.__init__`
```python
    def __init__(
        self, config: PersimmonConfig, quant_config: Optional[QuantizationConfig] = None
    ):
        super().__init__()
        self.dense_h_to_4h = ColumnParallelLinear(
            config.hidden_size, config.intermediate_size, quant_config=quant_config
        )
        self.dense_4h_to_h = RowParallelLinear(
            config.intermediate_size, config.hidden_size, quant_config=quant_config
        )
        self.act = get_act_fn(config.hidden_act)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 43-47: Method `PersimmonMLP.forward`
```python
    def forward(self, hidden_states) -> torch.Tensor:
        hidden_states, _ = self.dense_h_to_4h(hidden_states)
        hidden_states = self.act(hidden_states)
        hidden_states, _ = self.dense_4h_to_h(hidden_states)
        return hidden_states
```
**EN:** This method implements `forward(hidden_states)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 50-51: Class `PersimmonAttention` overview
```python
class PersimmonAttention(nn.Module):
```
**EN:** Defines `PersimmonAttention` as a reusable runtime type derived from nn.Module. The class groups 4 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `PersimmonAttention`，其继承关系为 nn.Module。这个类组织了 4 个方法，用于实现模型相关行为。

### Lines 52-110: Method `PersimmonAttention.__init__`
```python
    def __init__(
        self,
        config: PersimmonConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        layer_id: int = 0,
    ):
        super().__init__()
        self.config = config
        tensor_parallel_world_size = get_tensor_model_parallel_world_size()

        self.hidden_size = config.hidden_size
        self.total_num_heads = config.num_attention_heads
        self.num_heads = self.total_num_heads // tensor_parallel_world_size
        self.head_dim = self.hidden_size // self.total_num_heads
        self.max_position_embeddings = config.max_position_embeddings
        self.rope_theta = config.rope_parameters["rope_theta"]
        self.partial_rotary_factor = config.partial_rotary_factor
        self.is_causal = True

        assert (self.head_dim * self.total_num_heads) == self.hidden_size
        assert self.total_num_heads % tensor_parallel_world_size == 0

        self.query_key_value = QKVParallelLinear(
            self.hidden_size,
            self.head_dim,
            self.total_num_heads,
            bias=True,
            quant_config=quant_config,
        )
        self.dense = RowParallelLinear(
            self.total_num_heads * self.head_dim,
            self.hidden_size,
            bias=True,
            quant_config=quant_config,
        )
        self.is_qk_layernorm = config.qk_layernorm

        if self.is_qk_layernorm:
            self.q_layernorm = nn.LayerNorm(self.head_dim)
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=..., layer_id: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=..., layer_id: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 112-114: Method `PersimmonAttention._split_heads`
```python
    def _split_heads(self, x: torch.Tensor) -> torch.Tensor:
        seq_length = x.shape[0]
        return x.view(seq_length, self.num_heads, self.head_dim)
```
**EN:** This method implements `_split_heads(x: ...)` and implements a focused helper that supports the surrounding runtime flow inside `PersimmonAttention`.
**CN:** 这个方法实现了 `_split_heads(x: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `PersimmonAttention` 内部调用。

### Lines 116-118: Method `PersimmonAttention._merge_heads`
```python
    def _merge_heads(self, x: torch.Tensor) -> torch.Tensor:
        seq_length = x.shape[0]
        return x.view(seq_length, self.num_heads * self.head_dim)
```
**EN:** This method implements `_merge_heads(x: ...)` and implements a focused helper that supports the surrounding runtime flow inside `PersimmonAttention`.
**CN:** 这个方法实现了 `_merge_heads(x: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `PersimmonAttention` 内部调用。

### Lines 120-142: Method `PersimmonAttention.forward`
```python
    def forward(
        self,
        position_ids: torch.Tensor,
        forward_batch: ForwardBatch,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        qkv, _ = self.query_key_value(hidden_states)
        q, k, v = qkv.chunk(chunks=3, dim=-1)

        if self.is_qk_layernorm:
            q = self._split_heads(q)
            k = self._split_heads(k)

            q = self.q_layernorm(q)
            k = self.k_layernorm(k)

            q = self._merge_heads(q)
            k = self._merge_heads(k)

        q, k = self.rotary_emb(position_ids, q, k)
        attn_output = self.attn(q, k, v, forward_batch=forward_batch)
        output, _ = self.dense(attn_output)
        return output
```
**EN:** This method implements `forward(position_ids: ..., forward_batch: ..., hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(position_ids: ..., forward_batch: ..., hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 145-146: Class `PersimmonDecoderLayer` overview
```python
class PersimmonDecoderLayer(nn.Module):
```
**EN:** Defines `PersimmonDecoderLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `PersimmonDecoderLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 147-168: Method `PersimmonDecoderLayer.__init__`
```python
    def __init__(
        self,
        config: PersimmonConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        idx: int = 0,
    ):
        super().__init__()
        self.hidden_size = config.hidden_size
        self.self_attn = PersimmonAttention(
            config=config,
            quant_config=quant_config,
            prefix=add_prefix("self_attn", prefix),
            layer_id=idx,
        )
        self.mlp = PersimmonMLP(config, quant_config=quant_config)
        self.input_layernorm = nn.LayerNorm(
            config.hidden_size, eps=config.layer_norm_eps
        )
        self.post_attention_layernorm = nn.LayerNorm(
            config.hidden_size, eps=config.layer_norm_eps
        )
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=..., idx: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=..., idx: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 170-194: Method `PersimmonDecoderLayer.forward`
```python
    def forward(
        self,
        position_ids: torch.Tensor,
        forward_batch: ForwardBatch,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        residual = hidden_states

        hidden_states = self.input_layernorm(hidden_states)

        hidden_states = self.self_attn(
            position_ids=position_ids,
            hidden_states=hidden_states,
            forward_batch=forward_batch,
        )
        hidden_states = residual + hidden_states

        residual = hidden_states
        hidden_states = self.post_attention_layernorm(hidden_states)
        hidden_states = self.mlp(hidden_states)

        hidden_states = hidden_states + residual

        outputs = hidden_states
        return outputs
```
**EN:** This method implements `forward(position_ids: ..., forward_batch: ..., hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(position_ids: ..., forward_batch: ..., hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 197-198: Class `PersimmonModel` overview
```python
class PersimmonModel(nn.Module):
```
**EN:** Defines `PersimmonModel` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `PersimmonModel`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 199-231: Method `PersimmonModel.__init__`
```python
    def __init__(
        self,
        config: PersimmonConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.pp_group = get_pp_group()

        if self.pp_group.is_first_rank:
            self.embed_tokens = VocabParallelEmbedding(
                config.vocab_size, config.hidden_size
            )
        else:
            self.embed_tokens = PPMissingLayer()

        self.layers, self.start_layer, self.end_layer = make_layers(
            config.num_hidden_layers,
            lambda idx, prefix: PersimmonDecoderLayer(
                config, quant_config=quant_config, prefix=prefix, idx=idx
            ),
            prefix="model.layers",
            pp_rank=self.pp_group.rank_in_group,
            pp_size=self.pp_group.world_size,
        )

        if self.pp_group.is_last_rank:
            self.final_layernorm = nn.LayerNorm(
                config.hidden_size, eps=config.layer_norm_eps
            )
        else:
            self.final_layernorm = PPMissingLayer()
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 233-234: Method `PersimmonModel.get_input_embeddings`
```python
    def get_input_embeddings(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.embed_tokens(input_ids)
```
**EN:** This method implements `get_input_embeddings(input_ids: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings(input_ids: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 236-257: Method `PersimmonModel.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        forward_batch: ForwardBatch,
        positions: torch.Tensor,
        inputs_embeds: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        if self.pp_group.is_first_rank:
            if inputs_embeds is not None:
                hidden_states = inputs_embeds
            else:
                hidden_states = self.get_input_embeddings(input_ids)
        else:
            hidden_states = forward_batch.pp_input_hidden
        for i in range(self.start_layer, self.end_layer):
            layer = self.layers[i]
            hidden_states = layer(
                position_ids=positions,
                forward_batch=forward_batch,
                hidden_states=hidden_states,
            )
        return self.final_layernorm(hidden_states)
```
**EN:** This method implements `forward(input_ids: ..., forward_batch: ..., positions: ..., inputs_embeds: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., forward_batch: ..., positions: ..., inputs_embeds: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 260-261: Class `PersimmonForCausalLM` overview
```python
class PersimmonForCausalLM(nn.Module):
```
**EN:** Defines `PersimmonForCausalLM` as a reusable runtime type derived from nn.Module. The class groups 4 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `PersimmonForCausalLM`，其继承关系为 nn.Module。这个类组织了 4 个方法，用于实现模型相关行为。

### Lines 262-280: Method `PersimmonForCausalLM.__init__`
```python
    def __init__(
        self,
        config: PersimmonConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.quant_config = quant_config
        self.model = PersimmonModel(
            config=config, quant_config=quant_config, prefix=add_prefix("model", prefix)
        )
        self.lm_head = ParallelLMHead(
            config.vocab_size,
            config.hidden_size,
            bias=False,
            quant_config=quant_config,
        )
        self.logits_processor = LogitsProcessor(config)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 282-283: Method `PersimmonForCausalLM.get_input_embeddings`
```python
    def get_input_embeddings(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.model.get_input_embeddings(input_ids)
```
**EN:** This method implements `get_input_embeddings(input_ids: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings(input_ids: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 285-301: Method `PersimmonForCausalLM.forward`
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

### Lines 303-327: Method `PersimmonForCausalLM.load_weights`
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]):
        params_dict = dict(self.named_parameters())
        for name, loaded_weight in weights:
            if "rotary_emb.inv_freq" in name:
                continue
            if name not in params_dict:
                if name == "lm_head.weight":
                    continue
                print(f"Warning: weight {name} not found in model.")
                continue
            param = params_dict[name]
            if "query_key_value" in name:
                output_dim = getattr(param, "output_dim", None)
                if output_dim is not None:
                    loaded_weight_shape = loaded_weight.shape
                    num_heads = self.config.num_attention_heads
                    loaded_weight = loaded_weight.view(
                        loaded_weight_shape[:output_dim]
                        + (num_heads, 3, -1)
                        + loaded_weight_shape[output_dim + 1 :]
                    )
                    loaded_weight = loaded_weight.transpose(output_dim, output_dim + 1)
                    loaded_weight = loaded_weight.reshape(loaded_weight_shape)
            weight_loader = getattr(param, "weight_loader", default_weight_loader)
            weight_loader(param, loaded_weight)
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 330-330: Top-level assign
```python
EntryClass = PersimmonForCausalLM
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `collections.abc.Iterable`
- `typing.Optional`
- `torch`
- `torch.nn`
- `transformers.PersimmonConfig`
- `sglang.srt.distributed.get_pp_group`
- `sglang.srt.distributed.get_tensor_model_parallel_world_size`
- `sglang.srt.layers.activation.get_act_fn`
- `sglang.srt.layers.linear.ColumnParallelLinear`
- `sglang.srt.layers.linear.QKVParallelLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.logits_processor.LogitsProcessorOutput`
- `sglang.srt.layers.quantization.QuantizationConfig`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.layers.rotary_embedding.get_rope`
- `sglang.srt.layers.utils.PPMissingLayer`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.make_layers`
