# phi3_small.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/phi3_small.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the phi3 small model stack used by SGLang SRT inference, including network blocks, runtime wrappers, and checkpoint-loading utilities. / 该模块实现 SGLang SRT 推理所使用的 phi3 small 模型栈，包括网络模块、运行时封装以及检查点加载工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28: Module imports
```python
import math
from typing import Iterable, Optional, Tuple, Union

import torch
from torch import nn
from transformers import Phi3Config
from transformers.configuration_utils import PretrainedConfig

from sglang.srt.distributed import get_pp_group, get_tensor_model_parallel_world_size
from sglang.srt.layers.linear import (
    MergedColumnParallelLinear,
    QKVParallelLinear,
    RowParallelLinear,
)
from sglang.srt.layers.logits_processor import LogitsProcessor, LogitsProcessorOutput
from sglang.srt.layers.pooler import Pooler, PoolingType
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.layers.radix_attention import RadixAttention
from sglang.srt.layers.rotary_embedding import get_rope
from sglang.srt.layers.utils import PPMissingLayer
from sglang.srt.layers.vocab_parallel_embedding import (
    DEFAULT_VOCAB_PADDING_SIZE,
    ParallelLMHead,
    VocabParallelEmbedding,
)
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.model_loader.weight_utils import default_weight_loader
from sglang.srt.utils import add_prefix, make_layers
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 31-33: Function `quick_gelu`
```python
@torch.jit.script
def quick_gelu(x):
    return x * torch.sigmoid(1.702 * x)
```
**EN:** This function implements `quick_gelu(x)` and implements a focused helper that supports the surrounding runtime flow Decorators: torch.jit.script.
**CN:** 这个函数实现了 `quick_gelu(x)`，其作用是实现一个支撑周边运行时流程的辅助逻辑 装饰器：torch.jit.script。

### Lines 36-49: Function `gegelu`
```python
@torch.jit.script
def gegelu(input, limit: Optional[float] = None):
    a_gelu, a_linear = input[..., ::2], input[..., 1::2]
    if limit is not None:
        a_gelu = torch.where(
            torch.isinf(a_gelu), a_gelu, a_gelu.clamp(min=None, max=limit)
        )
        a_linear = torch.where(
            torch.isinf(a_linear),
            a_linear,
            a_linear.clamp(min=-limit, max=limit),
        )
    out_gelu = quick_gelu(a_gelu)
    return out_gelu * (a_linear + 1)
```
**EN:** This function implements `gegelu(input, limit: ...=...)` and implements a focused helper that supports the surrounding runtime flow Decorators: torch.jit.script.
**CN:** 这个函数实现了 `gegelu(input, limit: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑 装饰器：torch.jit.script。

### Lines 52-53: Class `Phi3SmallMLP` overview
```python
class Phi3SmallMLP(nn.Module):
```
**EN:** Defines `Phi3SmallMLP` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Phi3SmallMLP`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 54-82: Method `Phi3SmallMLP.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        assert (
            self.config.hidden_act == "gegelu"
        ), "Only `gegelu` is supported for the 4.7 series of models .."
        self.hidden_size = config.hidden_size
        self.gegelu_limit = config.gegelu_limit
        self.intermediate_size = config.intermediate_size

        self.up_proj = MergedColumnParallelLinear(
            self.hidden_size,
            2 * [self.intermediate_size],
            bias=True,
            quant_config=quant_config,
            prefix=add_prefix("up_proj", prefix),
        )
        self.down_proj = RowParallelLinear(
            self.intermediate_size,
            self.hidden_size,
            bias=True,
            quant_config=quant_config,
            prefix=add_prefix("down_proj", prefix),
        )
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 84-88: Method `Phi3SmallMLP.forward`
```python
    def forward(self, x):
        gate_up, _ = self.up_proj(x)
        x = gegelu(gate_up)
        x, _ = self.down_proj(x)
        return x
```
**EN:** This method implements `forward(x)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 91-92: Class `Phi3SmallSelfAttention` overview
```python
class Phi3SmallSelfAttention(nn.Module):
```
**EN:** Defines `Phi3SmallSelfAttention` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Phi3SmallSelfAttention`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 93-208: Method `Phi3SmallSelfAttention.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int = 0,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.layer_id = layer_id
        self.config = config
        self.sparse_block_size = config.blocksparse_block_size
        self.homo_heads = config.blocksparse_homo_head_pattern
        self.local_blocks = config.blocksparse_num_local_blocks
        self.vert_stride = config.blocksparse_vert_stride

        assert (
            config.blocksparse_block_size == config.blocksparse_triton_kernel_block_size
        )

        self.hidden_size = config.hidden_size
        # Number of Query Heads
        self.num_heads = config.num_attention_heads

        self.head_dim = self.hidden_size // self.num_heads
        self.tp_size = get_tensor_model_parallel_world_size()
        # Number of total Key Value Heads before tensor parallel
        self.num_key_value_heads = config.num_key_value_heads
        self.num_q_per_kv = self.num_heads // self.num_key_value_heads
        if self.tp_size > 1:
            assert self.num_key_value_heads % self.tp_size == 0
        self.num_kv_heads_per_partion = max(1, self.num_key_value_heads // self.tp_size)
        self.num_heads_per_partition = self.num_heads // self.tp_size

        self.max_position_embeddings = config.max_position_embeddings
        self.rope_embedding_base = config.rope_embedding_base
        self.rope_position_scale = config.rope_position_scale
        self.is_causal = True

        norm_factor = None
        if config.mup_use_scaling:
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_id: ...=..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ...=..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 210-231: Method `Phi3SmallSelfAttention.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> Tuple[torch.Tensor, Optional[torch.Tensor], Optional[Tuple[torch.Tensor]]]:
        qkv, _ = self.query_key_value(hidden_states)

        qkv = qkv.view(qkv.shape[:-1] + (-1, (self.num_q_per_kv + 2), self.head_dim))
        q, k, v = qkv.split([self.num_q_per_kv, 1, 1], dim=-2)

        # NOTE: this is required by RotaryEmbed, which indeed does not have to
        # TODO: allow 3D QK for rotary forward
        q = q.reshape(-1, self.head_dim * self.num_heads_per_partition)
        k = k.reshape(-1, self.head_dim * self.num_kv_heads_per_partion)
        v = v.reshape(-1, self.head_dim * self.num_kv_heads_per_partion)

        q, k = self.rotary_emb(positions, q, k)
        attn_output = self.attn(q, k, v, forward_batch=forward_batch)
        output, _ = self.dense(attn_output)

        return output
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 234-235: Class `Phi3SmallDecoderLayer` overview
```python
class Phi3SmallDecoderLayer(nn.Module):
```
**EN:** Defines `Phi3SmallDecoderLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Phi3SmallDecoderLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 236-262: Method `Phi3SmallDecoderLayer.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = config.hidden_size
        self.self_attn = Phi3SmallSelfAttention(
            config,
            layer_id,
            quant_config=quant_config,
            prefix=add_prefix("self_attn", prefix),
        )
        self.mlp = Phi3SmallMLP(
            config,
            quant_config,
            prefix=add_prefix("mlp", prefix),
        )

        self.input_layernorm = nn.LayerNorm(
            config.hidden_size, eps=config.layer_norm_epsilon
        )
        self.post_attention_layernorm = nn.LayerNorm(
            config.hidden_size, eps=config.layer_norm_epsilon
        )
```
**EN:** This method implements `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 264-284: Method `Phi3SmallDecoderLayer.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        residual = hidden_states
        hidden_states = self.input_layernorm(hidden_states)

        hidden_states = self.self_attn(
            positions=positions,
            hidden_states=hidden_states,
            forward_batch=forward_batch,
        )
        hidden_states = residual + hidden_states

        residual = hidden_states
        hidden_states = self.post_attention_layernorm(hidden_states)
        hidden_states = self.mlp(hidden_states)
        hidden_states = residual + hidden_states
        return hidden_states
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 287-288: Class `Phi3SmallModel` overview
```python
class Phi3SmallModel(nn.Module):
```
**EN:** Defines `Phi3SmallModel` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Phi3SmallModel`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 289-330: Method `Phi3SmallModel.__init__`
```python
    def __init__(
        self,
        config: Phi3Config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()

        self.config = config

        self.pp_group = get_pp_group()
        if self.pp_group.is_first_rank:
            self.embed_tokens = VocabParallelEmbedding(
                config.vocab_size,
                config.hidden_size,
                prefix=add_prefix("embed_tokens", prefix),
            )
        else:
            self.embed_tokens = PPMissingLayer()

        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size,
            config.hidden_size,
            prefix=add_prefix("embed_tokens", prefix),
        )
        self.mup_embedding_multiplier = config.mup_embedding_multiplier
        self.layers, self.start_layer, self.end_layer = make_layers(
            config.num_hidden_layers,
            lambda idx, prefix: Phi3SmallDecoderLayer(
                config,
                int(prefix.split(".")[-1]),
                quant_config,
                prefix=prefix,
            ),
            pp_rank=self.pp_group.rank_in_group,
            pp_size=self.pp_group.world_size,
            prefix=add_prefix("layers", prefix),
        )

        self.final_layernorm = nn.LayerNorm(
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 332-333: Method `Phi3SmallModel.get_input_embeddings`
```python
    def get_input_embeddings(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.embed_tokens(input_ids)
```
**EN:** This method implements `get_input_embeddings(input_ids: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings(input_ids: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 335-358: Method `Phi3SmallModel.forward`
```python
    def forward(
        self,
        input_ids: torch.LongTensor,
        positions: Optional[torch.LongTensor],
        forward_batch: ForwardBatch,
        inputs_embeds: Optional[torch.Tensor],
    ) -> Union[torch.Tensor]:

        if inputs_embeds is not None:
            hidden_states = inputs_embeds
        else:
            hidden_states = self.get_input_embeddings(input_ids)
        if (
            self.mup_embedding_multiplier is not None
            and self.mup_embedding_multiplier > 0.0
        ):
            hidden_states = hidden_states * self.mup_embedding_multiplier

        for i in range(self.start_layer, self.end_layer):
            layer = self.layers[i]
            hidden_states = layer(positions, hidden_states, forward_batch=forward_batch)

        hidden_states = self.final_layernorm(hidden_states)
        return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., inputs_embeds: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., inputs_embeds: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 361-361: Class `Phi3SmallForCausalLM` overview
```python
class Phi3SmallForCausalLM(nn.Module):
```
**EN:** Defines `Phi3SmallForCausalLM` as a reusable runtime type derived from nn.Module. The class groups 10 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Phi3SmallForCausalLM`，其继承关系为 nn.Module。这个类组织了 10 个方法，用于实现模型相关行为。

### Lines 362-362: Class `Phi3SmallForCausalLM` attributes
```python
    _tied_weights_keys = ["lm_head.weight"]
```
**EN:** Defines class-level attributes and metadata that shape how `Phi3SmallForCausalLM` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `Phi3SmallForCausalLM` 在运行时的行为。

### Lines 364-404: Method `Phi3SmallForCausalLM.__init__`
```python
    def __init__(
        self,
        config: Phi3Config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):

        super().__init__()

        self.config = config
        self.quant_config = quant_config
        self.model = Phi3SmallModel(
            config=config,
            quant_config=quant_config,
            prefix=add_prefix("model", prefix),
        )
        self.vocab_size = config.vocab_size
        self.mup_width_multiplier = config.mup_width_multiplier
        self.lm_head = ParallelLMHead(
            self.vocab_size,
            config.hidden_size,
            org_num_embeddings=config.vocab_size,
            padding_size=DEFAULT_VOCAB_PADDING_SIZE,
            quant_config=quant_config,
            prefix=add_prefix("lm_head", prefix),
        )
        if self.config.tie_word_embeddings:
            self.lm_head.weight = self.model.embed_tokens.weight
        self.logits_processor = LogitsProcessor(config)
        self.pooler = Pooler(pooling_type=PoolingType.LAST, normalize=True)

        # tokens in tiktoken but not used
        if hasattr(config, "dummy_token_indices"):
            device = self.lm_head.weight.device
            self.register_buffer(
                "dummy_token_indices",
                torch.LongTensor(config.dummy_token_indices).to(device),
                persistent=False,
            )
        else:
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 406-407: Method `Phi3SmallForCausalLM.get_input_embeddings`
```python
    def get_input_embeddings(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.model.get_input_embeddings(input_ids)
```
**EN:** This method implements `get_input_embeddings(input_ids: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings(input_ids: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 409-410: Method `Phi3SmallForCausalLM.set_input_embeddings`
```python
    def set_input_embeddings(self, value):
        self.model.embed_tokens = value
```
**EN:** This method implements `set_input_embeddings(value)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `set_input_embeddings(value)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 412-413: Method `Phi3SmallForCausalLM.get_output_embeddings`
```python
    def get_output_embeddings(self):
        return self.lm_head
```
**EN:** This method implements `get_output_embeddings()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_output_embeddings()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 415-416: Method `Phi3SmallForCausalLM.set_output_embeddings`
```python
    def set_output_embeddings(self, value):
        self.lm_head = value
```
**EN:** This method implements `set_output_embeddings(value)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `set_output_embeddings(value)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 418-419: Method `Phi3SmallForCausalLM.set_decoder`
```python
    def set_decoder(self, decoder):
        self.model = decoder
```
**EN:** This method implements `set_decoder(decoder)` and implements a focused helper that supports the surrounding runtime flow inside `Phi3SmallForCausalLM`.
**CN:** 这个方法实现了 `set_decoder(decoder)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Phi3SmallForCausalLM` 内部调用。

### Lines 421-422: Method `Phi3SmallForCausalLM.get_decoder`
```python
    def get_decoder(self):
        return self.model
```
**EN:** This method implements `get_decoder()` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_decoder()`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 424-435: Method `Phi3SmallForCausalLM.compute_logits`
```python
    def compute_logits(
        self,
        input_ids: torch.LongTensor,
        hidden_states: torch.Tensor,
        sampling_metadata,
    ) -> Optional[torch.Tensor]:
        logits = self.logits_processor(
            input_ids, self.lm_head, hidden_states, sampling_metadata
        )
        if self.dummy_token_indices is not None and logits is not None:
            logits.index_fill_(-1, self.dummy_token_indices, -torch.inf)
        return logits
```
**EN:** This method implements `compute_logits(input_ids: ..., hidden_states: ..., sampling_metadata)` and computes or post-processes logits before sampling or scoring.
**CN:** 这个方法实现了 `compute_logits(input_ids: ..., hidden_states: ..., sampling_metadata)`，其作用是在采样或打分前计算或后处理 logits。

### Lines 437-458: Method `Phi3SmallForCausalLM.forward`
```python
    def forward(
        self,
        input_ids: torch.LongTensor,
        positions: Optional[torch.LongTensor],
        forward_batch: ForwardBatch,
        inputs_embeds: Optional[torch.Tensor] = None,
        get_embedding: bool = False,
    ) -> LogitsProcessorOutput:
        hidden_states = self.model(
            input_ids=input_ids,
            positions=positions,
            forward_batch=forward_batch,
            inputs_embeds=inputs_embeds,
        )

        if not get_embedding:
            return self.logits_processor(
                input_ids, hidden_states, self.lm_head, forward_batch
            )

        else:
            return self.pooler(hidden_states, forward_batch)
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., inputs_embeds: ...=..., get_embedding: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., inputs_embeds: ...=..., get_embedding: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 460-473: Method `Phi3SmallForCausalLM.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):

        params_dict = dict(self.named_parameters())
        for name, loaded_weight in weights:
            if "rotary_emb.inv_freq" in name:
                continue
            if name.endswith(".bias") and name not in params_dict:
                continue
            if self.config.tie_word_embeddings and "lm_head.weight" in name:
                continue

            param = params_dict[name]
            weight_loader = getattr(param, "weight_loader", default_weight_loader)
            weight_loader(param, loaded_weight)
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 476-476: Top-level assign
```python
EntryClass = Phi3SmallForCausalLM
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `math`
- `typing.Iterable`
- `typing.Optional`
- `typing.Tuple`
- `typing.Union`
- `torch`
- `torch.nn`
- `transformers.Phi3Config`
- `transformers.configuration_utils.PretrainedConfig`
- `sglang.srt.distributed.get_pp_group`
- `sglang.srt.distributed.get_tensor_model_parallel_world_size`
- `sglang.srt.layers.linear.MergedColumnParallelLinear`
- `sglang.srt.layers.linear.QKVParallelLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.logits_processor.LogitsProcessorOutput`
- `sglang.srt.layers.pooler.Pooler`
- `sglang.srt.layers.pooler.PoolingType`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.layers.rotary_embedding.get_rope`
- `sglang.srt.layers.utils.PPMissingLayer`
- `sglang.srt.layers.vocab_parallel_embedding.DEFAULT_VOCAB_PADDING_SIZE`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.make_layers`
