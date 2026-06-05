# nemotron_nas.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/nemotron_nas.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Inference-only deci model compatible with HuggingFace weights. This file provides runtime-ready implementation details for SGLang inference. / 该文件围绕上述主题实现了可直接用于 SGLang 推理运行时的具体逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 18-18: Module docstring
```python
"""Inference-only deci model compatible with HuggingFace weights."""
```
**EN:** States the module scope and documents the high-level intention of the implementation.
**CN:** 说明模块范围，并给出实现的高层设计意图。

### Lines 20-44: Module imports
```python
from typing import Iterable, Optional, Tuple, Type, Union

import torch
from torch import nn
from transformers import LlamaConfig

from sglang.srt.distributed import get_pp_group
from sglang.srt.layers.layernorm import RMSNorm
from sglang.srt.layers.logits_processor import LogitsProcessor, LogitsProcessorOutput
from sglang.srt.layers.pooler import Pooler, PoolingType
from sglang.srt.layers.quantization import QuantizationConfig
from sglang.srt.layers.utils import PPMissingLayer
from sglang.srt.layers.vocab_parallel_embedding import (
    DEFAULT_VOCAB_PADDING_SIZE,
    ParallelLMHead,
    VocabParallelEmbedding,
)
from sglang.srt.model_executor.forward_batch_info import ForwardBatch, PPProxyTensors
from sglang.srt.model_loader.weight_utils import (
    default_weight_loader,
    maybe_remap_kv_scale_name,
)
from sglang.srt.models.llama import LlamaAttention, LlamaMLP
from sglang.srt.utils import add_prefix, make_layers
from sglang.utils import logger
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 47-50: Function `_ffn_mult_to_intermediate_size`
```python
def _ffn_mult_to_intermediate_size(ffn_mult: float, n_embd: int) -> int:
    # DeciLM-specific code
    intermediate_size = int(2 * ffn_mult * n_embd / 3)
    return _find_multiple(intermediate_size, 256)
```
**EN:** This function implements `_ffn_mult_to_intermediate_size(ffn_mult: ..., n_embd: ...)` and implements a focused helper that supports the surrounding runtime flow.
**CN:** 这个函数实现了 `_ffn_mult_to_intermediate_size(ffn_mult: ..., n_embd: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑。

### Lines 53-57: Function `_find_multiple`
```python
def _find_multiple(n: int, k: int) -> int:
    # DeciLM-specific code
    if n % k == 0:
        return n
    return n + k - (n % k)
```
**EN:** This function implements `_find_multiple(n: ..., k: ...)` and implements a focused helper that supports the surrounding runtime flow.
**CN:** 这个函数实现了 `_find_multiple(n: ..., k: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑。

### Lines 60-61: Class `DeciLMDecoderLayer` overview
```python
class DeciLMDecoderLayer(nn.Module):
```
**EN:** Defines `DeciLMDecoderLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `DeciLMDecoderLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 62-128: Method `DeciLMDecoderLayer.__init__`
```python
    def __init__(
        self,
        config: LlamaConfig,
        layer_idx: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        block_config = config.block_configs[layer_idx]
        self._is_no_op_attention = block_config.attention.no_op
        self._is_no_op_ffn = block_config.ffn.no_op

        self.hidden_size = config.hidden_size
        rope_theta = config.rope_parameters["rope_theta"]
        rope_scaling = config.rope_parameters
        if rope_scaling is not None and getattr(
            config, "original_max_position_embeddings", None
        ):
            rope_scaling["original_max_position_embeddings"] = (
                config.original_max_position_embeddings
            )
        max_position_embeddings = getattr(config, "max_position_embeddings", 8192)
        # Support abacusai/Smaug-72B-v0.1 with attention_bias
        # Support internlm/internlm-7b with bias
        rope_is_neox_style = getattr(config, "rope_is_neox_style", True)
        attention_bias = getattr(config, "attention_bias", False) or getattr(
            config, "bias", False
        )
        # support internlm/internlm3-8b with qkv_bias
        if hasattr(config, "qkv_bias"):
            attention_bias = config.qkv_bias

        if not self._is_no_op_attention:
            num_kv_heads = (
                config.num_attention_heads // block_config.attention.n_heads_in_group
            )
            self.self_attn = LlamaAttention(
                config=config,
                hidden_size=self.hidden_size,
                num_heads=config.num_attention_heads,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_idx: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_idx: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 130-159: Method `DeciLMDecoderLayer.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
        residual: Optional[torch.Tensor],
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        # Self Attention

        if self._is_no_op_attention:
            pass
        else:
            if residual is None:
                residual = hidden_states
                hidden_states = self.input_layernorm(hidden_states)
            else:
                hidden_states, residual = self.input_layernorm(hidden_states, residual)
            hidden_states = self.self_attn(
                positions=positions,
                hidden_states=hidden_states,
                forward_batch=forward_batch,
            )

        # Fully Connected
        if not self._is_no_op_ffn:
            hidden_states, residual = self.post_attention_layernorm(
                hidden_states, residual
            )
            hidden_states = self.mlp(hidden_states)
        return hidden_states, residual
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 162-162: Class `DeciModel` overview
```python
class DeciModel(nn.Module):
```
**EN:** Defines `DeciModel` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `DeciModel`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 163-211: Method `DeciModel.__init__`
```python
    def __init__(
        self,
        *,
        config: LlamaConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        layer_type: Type[DeciLMDecoderLayer] = DeciLMDecoderLayer,
    ):
        super().__init__()

        lora_config = None
        self.config = config
        self.quant_config = quant_config
        self.padding_idx = config.pad_token_id
        lora_vocab = (
            (lora_config.lora_extra_vocab_size * (lora_config.max_loras or 1))
            if lora_config
            else 0
        )
        vocab_size = config.vocab_size + lora_vocab
        if get_pp_group().is_first_rank:
            self.embed_tokens = VocabParallelEmbedding(
                vocab_size,
                config.hidden_size,
                org_num_embeddings=config.vocab_size,
                quant_config=quant_config,
            )
        else:
            self.embed_tokens = PPMissingLayer()

        def get_layer(idx: int, prefix: str):
            return layer_type(
                config,
                layer_idx=idx,
                quant_config=quant_config,
                prefix=prefix,
            )

        self.layers, self.start_layer, self.end_layer = make_layers(
            config.num_hidden_layers,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(*, config: ..., quant_config: ...=..., prefix: ...=..., layer_type: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(*, config: ..., quant_config: ...=..., prefix: ...=..., layer_type: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 213-214: Method `DeciModel.get_input_embeddings`
```python
    def get_input_embeddings(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.embed_tokens(input_ids)
```
**EN:** This method implements `get_input_embeddings(input_ids: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings(input_ids: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 216-254: Method `DeciModel.forward`
```python
    def forward(
        self,
        input_ids: Optional[torch.Tensor],
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        inputs_embeds: Optional[torch.Tensor] = None,
        pp_proxy_tensors: Optional[PPProxyTensors] = None,
    ) -> Union[torch.Tensor, PPProxyTensors]:
        if get_pp_group().is_first_rank:
            if inputs_embeds is not None:
                hidden_states = inputs_embeds
            else:
                hidden_states = self.get_input_embeddings(input_ids)
            residual = None
        else:
            assert pp_proxy_tensors is not None
            hidden_states = pp_proxy_tensors["hidden_states"]
            residual = pp_proxy_tensors["residual"]

        kv_cache_index = 0
        for i in range(self.start_layer, self.end_layer):
            layer = self.layers[i]
            if not layer._is_no_op_attention:
                hidden_states, residual = layer(
                    positions, hidden_states, forward_batch, residual
                )
                kv_cache_index += 1
            else:
                hidden_states, residual = layer(
                    positions, hidden_states, forward_batch, residual
                )

        if not get_pp_group().is_last_rank:
            return PPProxyTensors(
                {"hidden_states": hidden_states, "residual": residual}
            )

        hidden_states, _ = self.norm(hidden_states, residual)
        return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., inputs_embeds: ...=..., pp_proxy_tensors: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., inputs_embeds: ...=..., pp_proxy_tensors: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 257-257: Class `DeciLMForCausalLM` overview
```python
class DeciLMForCausalLM(nn.Module):
```
**EN:** Defines `DeciLMForCausalLM` as a reusable runtime type derived from nn.Module. The class groups 5 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `DeciLMForCausalLM`，其继承关系为 nn.Module。这个类组织了 5 个方法，用于实现模型相关行为。

### Lines 258-296: Class `DeciLMForCausalLM` attributes
```python
    packed_modules_mapping = {
        "qkv_proj": ["q_proj", "k_proj", "v_proj"],
        "gate_up_proj": ["gate_proj", "up_proj"],
    }

    # LoRA specific attributes
    supported_lora_modules = [
        "qkv_proj",
        "o_proj",
        "gate_up_proj",
        "down_proj",
        "embed_tokens",
        "lm_head",
    ]
    embedding_modules = {
        "embed_tokens": "input_embeddings",
        "lm_head": "output_embeddings",
    }
    embedding_padding_modules = ["lm_head"]

    # Mistral/Llama models can also be loaded with --load-format mistral
    # from consolidated.safetensors checkpoints
    mistral_mapping = {
        "layers": "model.layers",
        "attention": "self_attn",
        "wq": "q_proj",
        "wk": "k_proj",
        "wv": "v_proj",
        "wo": "o_proj",
        "attention_norm": "input_layernorm",
        "feed_forward": "mlp",
        "w1": "gate_proj",
        "w2": "down_proj",
        "w3": "up_proj",
        "ffn_norm": "post_attention_layernorm",
        "tok_embeddings": "model.embed_tokens",
        "output": "lm_head",
        "norm": "model.norm",
    }
```
**EN:** Defines class-level attributes and metadata that shape how `DeciLMForCausalLM` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `DeciLMForCausalLM` 在运行时的行为。

### Lines 298-334: Method `DeciLMForCausalLM.__init__`
```python
    def __init__(
        self,
        *,
        config: LlamaConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        lora_config = None
        self.config = config
        self.lora_config = lora_config

        self.model = self._init_model(
            config=config, quant_config=quant_config, prefix=add_prefix("model", prefix)
        )
        if self.config.tie_word_embeddings:
            self.lm_head = self.model.embed_tokens
        else:
            self.unpadded_vocab_size = config.vocab_size
            if lora_config:
                self.unpadded_vocab_size += lora_config.lora_extra_vocab_size
            self.lm_head = ParallelLMHead(
                self.unpadded_vocab_size,
                config.hidden_size,
                org_num_embeddings=config.vocab_size,
                padding_size=(
                    DEFAULT_VOCAB_PADDING_SIZE
                    # We need bigger padding if using lora for kernel
                    # compatibility
                    if not lora_config
                    else lora_config.lora_vocab_padding_size
                ),
                quant_config=quant_config,
                prefix=add_prefix("lm_head", prefix),
            )
        self.logits_processor = LogitsProcessor(config)
        self.pooler = Pooler(pooling_type=PoolingType.LAST, normalize=True)
```
**EN:** This method implements `__init__(*, config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(*, config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 336-342: Method `DeciLMForCausalLM._init_model`
```python
    def _init_model(
        self,
        config: LlamaConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        return DeciModel(config=config, quant_config=quant_config, prefix=prefix)
```
**EN:** This method implements `_init_model(config: ..., quant_config: ...=..., prefix: ...=...)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `_init_model(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是执行后续运行时所需的初始化工作。

### Lines 344-345: Method `DeciLMForCausalLM.get_input_embeddings`
```python
    def get_input_embeddings(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.model.get_input_embeddings(input_ids)
```
**EN:** This method implements `get_input_embeddings(input_ids: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings(input_ids: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 347-372: Method `DeciLMForCausalLM.forward`
```python
    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        inputs_embeds: Optional[torch.Tensor] = None,
        get_embedding: bool = False,
        pp_proxy_tensors: Optional[PPProxyTensors] = None,
    ) -> LogitsProcessorOutput:
        hidden_states = self.model(
            input_ids,
            positions,
            forward_batch,
            inputs_embeds,
            pp_proxy_tensors=pp_proxy_tensors,
        )
        if get_pp_group().is_last_rank:
            if not get_embedding:
                return self.logits_processor(
                    input_ids, hidden_states, self.lm_head, forward_batch
                )
            else:
                return self.pooler(hidden_states, forward_batch)
        else:
            return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., inputs_embeds: ...=..., get_embedding: ...=..., pp_proxy_tensors: ...=...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., inputs_embeds: ...=..., get_embedding: ...=..., pp_proxy_tensors: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 374-435: Method `DeciLMForCausalLM.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]) -> None:
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            (".qkv_proj", ".q_proj", "q"),
            (".qkv_proj", ".k_proj", "k"),
            (".qkv_proj", ".v_proj", "v"),
            (".gate_up_proj", ".gate_proj", 0),
            (".gate_up_proj", ".up_proj", 1),
        ]

        params_dict = dict(self.named_parameters())

        for name, loaded_weight in weights:
            if "rotary_emb.inv_freq" in name:
                continue
            if "rotary_emb.cos_cached" in name or "rotary_emb.sin_cached" in name:
                # Models trained using ColossalAI may include these tensors in
                # the checkpoint. Skip them.
                continue
            if self.config.tie_word_embeddings and "lm_head.weight" in name:
                continue
            if self.model.quant_config is not None and (
                scale_name := self.model.quant_config.get_cache_scale(name)
            ):
                # Loading kv cache quantization scales
                param = params_dict[scale_name]
                weight_loader = getattr(param, "weight_loader", default_weight_loader)
                loaded_weight = (
                    loaded_weight if loaded_weight.dim() == 0 else loaded_weight[0]
                )
                weight_loader(param, loaded_weight)
                continue
            if "scale" in name:
                name = maybe_remap_kv_scale_name(name, params_dict)
                if name is None:
                    continue

            for param_name, weight_name, shard_id in stacked_params_mapping:
                if weight_name not in name:
                    continue
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 438-438: Top-level assign
```python
EntryClass = [DeciLMForCausalLM]
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
- `typing.Tuple`
- `typing.Type`
- `typing.Union`
- `torch`
- `torch.nn`
- `transformers.LlamaConfig`
- `sglang.srt.distributed.get_pp_group`
- `sglang.srt.layers.layernorm.RMSNorm`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.logits_processor.LogitsProcessorOutput`
- `sglang.srt.layers.pooler.Pooler`
- `sglang.srt.layers.pooler.PoolingType`
- `sglang.srt.layers.quantization.QuantizationConfig`
- `sglang.srt.layers.utils.PPMissingLayer`
- `sglang.srt.layers.vocab_parallel_embedding.DEFAULT_VOCAB_PADDING_SIZE`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_executor.forward_batch_info.PPProxyTensors`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.model_loader.weight_utils.maybe_remap_kv_scale_name`
- `sglang.srt.models.llama.LlamaAttention`
- `sglang.srt.models.llama.LlamaMLP`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.make_layers`
- `sglang.utils.logger`
