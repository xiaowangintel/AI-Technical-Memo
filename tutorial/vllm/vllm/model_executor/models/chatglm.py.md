# chatglm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/chatglm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for chatglm, including architecture wrappers and weight loading logic. / 面向推理的 chatglm vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 7-45)
```python
import json
from collections.abc import Iterable
from itertools import islice

import torch
from torch import nn
from torch.nn import LayerNorm

from vllm.compilation.decorators import support_torch_compile
from vllm.config import CacheConfig, VllmConfig
from vllm.distributed import get_pp_group, get_tensor_model_parallel_world_size
from vllm.model_executor.layers.activation import SiluAndMul
from vllm.model_executor.layers.attention import Attention
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.model_executor.layers.linear import (
    MergedColumnParallelLinear,
    QKVParallelLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.logits_processor import LogitsProcessor
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.rotary_embedding import get_rope
from vllm.model_executor.layers.vocab_parallel_embedding import (
    ParallelLMHead,
    VocabParallelEmbedding,
)
from vllm.model_executor.model_loader.weight_utils import default_weight_loader
from vllm.sequence import IntermediateTensors
from vllm.transformers_utils.configs.chatglm import ChatGLMConfig

from .interfaces import SupportsLoRA, SupportsPP, SupportsQuant
from .utils import (
    AutoWeightsLoader,
    WeightsMapper,
    is_pp_missing_parameter,
    make_empty_intermediate_tensors_factory,
    make_layers,
    maybe_prefix,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Class `GLMAttention` (lines 48-136)
```python
class GLMAttention(nn.Module):
    def __init__(
        self,
        config: ChatGLMConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = config.hidden_size
        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = config.num_attention_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.multi_query_attention = config.multi_query_attention
        self.total_num_kv_heads = (
            config.multi_query_group_num
            if config.multi_query_attention
            else config.num_attention_heads
        )
        if self.total_num_kv_heads >= tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % tp_size == 0
        else:
```
**EN:** Class `GLMAttention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `GLMAttention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `GLMAttention.__init__` (lines 49-124)
```python
    def __init__(
        self,
        config: ChatGLMConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = config.hidden_size
        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = config.num_attention_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.multi_query_attention = config.multi_query_attention
        self.total_num_kv_heads = (
            config.multi_query_group_num
            if config.multi_query_attention
            else config.num_attention_heads
        )
        if self.total_num_kv_heads >= tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)
        self.head_dim = config.hidden_size // self.total_num_heads
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5

        self.query_key_value = QKVParallelLinear(
            self.hidden_size,
            self.head_dim,
            self.total_num_heads,
            self.total_num_kv_heads,
            bias=config.add_bias_linear or config.add_qkv_bias,
            quant_config=quant_config,
            prefix=f"{prefix}.query_key_value",
        )
        self.dense = RowParallelLinear(
            self.total_num_heads * self.head_dim,
            config.hidden_size,
            bias=config.add_bias_linear,
            quant_config=quant_config,
            prefix=f"{prefix}.dense",
        )

        # https://huggingface.co/zai-org/chatglm3-6b-32k/blob/e210410255278dd9d74463cf396ba559c0ef801c/modeling_chatglm.py#L141
        rope_ratio = getattr(config, "rope_ratio", 1.0)
        max_positions = getattr(config, "seq_length", 8192)
        rope_parameters = {
            "rope_type": "default",
            "rope_theta": 10000 * rope_ratio,
            "partial_rotary_factor": 0.5,
        }
        # NOTE: zai-org/cogagent-9b-20241220 uses original_rope=False,
        # which is equivalent to is_neox_style=True
# ... truncated for analysis ...
```
**EN:** Method `GLMAttention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GLMAttention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GLMAttention.forward` (lines 126-136)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        position_ids: torch.Tensor,
    ) -> torch.Tensor:
        qkv, _ = self.query_key_value(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        q, k = self.rotary_emb(position_ids, q, k)
        context_layer = self.attn(q, k, v)
        attn_output, _ = self.dense(context_layer)
        return attn_output
```
**EN:** Method `GLMAttention.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `GLMAttention.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `GLMMLP` (lines 139-183)
```python
class GLMMLP(nn.Module):
    """MLP.

    MLP will take the input with h hidden state, project it to 4*h
    hidden dimension, perform nonlinear transformation, and project the
    state back into h hidden dimension.
    """

    def __init__(
        self,
        config: ChatGLMConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()

        self.add_bias = config.add_bias_linear

        # Project to 4h.
        self.dense_h_to_4h = MergedColumnParallelLinear(
            config.hidden_size,
            [config.ffn_hidden_size] * 2,
            bias=config.add_bias_linear,
            quant_config=quant_config,
            prefix=f"{prefix}.dense_h_to_4h",
```
**EN:** Class `GLMMLP` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `GLMMLP` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `GLMMLP.__init__` (lines 147-175)
```python
    def __init__(
        self,
        config: ChatGLMConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()

        self.add_bias = config.add_bias_linear

        # Project to 4h.
        self.dense_h_to_4h = MergedColumnParallelLinear(
            config.hidden_size,
            [config.ffn_hidden_size] * 2,
            bias=config.add_bias_linear,
            quant_config=quant_config,
            prefix=f"{prefix}.dense_h_to_4h",
        )

        self.activation_func = SiluAndMul()

        # Project back to h.
        self.dense_4h_to_h = RowParallelLinear(
            config.ffn_hidden_size,
            config.hidden_size,
            bias=config.add_bias_linear,
            quant_config=quant_config,
            prefix=f"{prefix}.dense_4h_to_h",
        )
```
**EN:** Method `GLMMLP.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GLMMLP.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GLMMLP.forward` (lines 177-183)
```python
    def forward(self, hidden_states):
        # [s, b, 4hp]
        intermediate_parallel, _ = self.dense_h_to_4h(hidden_states)
        intermediate_parallel = self.activation_func(intermediate_parallel)
        # [s, b, h]
        output, _ = self.dense_4h_to_h(intermediate_parallel)
        return output
```
**EN:** Method `GLMMLP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `GLMMLP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `GLMBlock` (lines 186-260)
```python
class GLMBlock(nn.Module):
    """A single transformer layer.

    Transformer layer takes input with size [s, b, h] and returns an
    output of the same size.
    """

    def __init__(
        self,
        config: ChatGLMConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.apply_residual_connection_post_layernorm = (
            config.apply_residual_connection_post_layernorm
        )

        self.fp32_residual_connection = config.fp32_residual_connection

        layer_norm_func = RMSNorm if config.rmsnorm else LayerNorm
        # Layernorm on the input data.
        self.input_layernorm = layer_norm_func(
            config.hidden_size, eps=config.layernorm_epsilon
```
**EN:** Class `GLMBlock` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `GLMBlock` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `GLMBlock.__init__` (lines 193-225)
```python
    def __init__(
        self,
        config: ChatGLMConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.apply_residual_connection_post_layernorm = (
            config.apply_residual_connection_post_layernorm
        )

        self.fp32_residual_connection = config.fp32_residual_connection

        layer_norm_func = RMSNorm if config.rmsnorm else LayerNorm
        # Layernorm on the input data.
        self.input_layernorm = layer_norm_func(
            config.hidden_size, eps=config.layernorm_epsilon
        )

        # Self attention.
        self.self_attention = GLMAttention(
            config, cache_config, quant_config, prefix=f"{prefix}.self_attention"
        )
        self.hidden_dropout = config.hidden_dropout

        # Layernorm on the attention output
        self.post_attention_layernorm = layer_norm_func(
            config.hidden_size, eps=config.layernorm_epsilon
        )

        # MLP
        self.mlp = GLMMLP(config, quant_config, prefix=f"{prefix}.mlp")
```
**EN:** Method `GLMBlock.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GLMBlock.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GLMBlock.forward` (lines 227-260)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        position_ids: torch.Tensor,
    ) -> torch.Tensor:
        # hidden_states: [num_tokens, h]
        # Layer norm at the beginning of the transformer layer.
        layernorm_output = self.input_layernorm(hidden_states)
        # Self attention.
        attention_output = self.self_attention(
            hidden_states=layernorm_output,
            position_ids=position_ids,
        )

        # Residual connection.
        if self.apply_residual_connection_post_layernorm:
            residual = layernorm_output
        else:
            residual = hidden_states

        layernorm_input = residual + attention_output

        # Layer norm post the self attention.
        layernorm_output = self.post_attention_layernorm(layernorm_input)

        # Second residual connection.
        if self.apply_residual_connection_post_layernorm:
            residual = layernorm_output
        else:
            residual = layernorm_input

        output = self.mlp(layernorm_output) + residual

        return output
```
**EN:** Method `GLMBlock.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `GLMBlock.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `GLMTransformer` (lines 263-314)
```python
class GLMTransformer(nn.Module):
    """Transformer class."""

    def __init__(
        self,
        config: ChatGLMConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.post_layer_norm = config.post_layer_norm

        # Number of layers.
        self.num_layers = config.num_layers

        # Transformer layers.
        self.start_layer, self.end_layer, self.layers = make_layers(
            self.num_layers,
            lambda prefix: GLMBlock(config, cache_config, quant_config, prefix=prefix),
            prefix=f"{prefix}.layers",
        )

        if self.post_layer_norm:
            layer_norm_func = RMSNorm if config.rmsnorm else LayerNorm
```
**EN:** Class `GLMTransformer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `GLMTransformer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `GLMTransformer.__init__` (lines 266-295)
```python
    def __init__(
        self,
        config: ChatGLMConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.post_layer_norm = config.post_layer_norm

        # Number of layers.
        self.num_layers = config.num_layers

        # Transformer layers.
        self.start_layer, self.end_layer, self.layers = make_layers(
            self.num_layers,
            lambda prefix: GLMBlock(config, cache_config, quant_config, prefix=prefix),
            prefix=f"{prefix}.layers",
        )

        if self.post_layer_norm:
            layer_norm_func = RMSNorm if config.rmsnorm else LayerNorm
            # Final layer norm before output.
            self.final_layernorm = layer_norm_func(
                config.hidden_size, eps=config.layernorm_epsilon
            )

        self.make_empty_intermediate_tensors = make_empty_intermediate_tensors_factory(
            ["hidden_states"], config.hidden_size
        )
```
**EN:** Method `GLMTransformer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GLMTransformer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GLMTransformer.forward` (lines 297-314)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        position_ids: torch.Tensor,
    ) -> torch.Tensor | IntermediateTensors:
        for layer in islice(self.layers, self.start_layer, self.end_layer):
            hidden_states = layer(
                hidden_states=hidden_states, position_ids=position_ids
            )

        if not get_pp_group().is_last_rank:
            return IntermediateTensors({"hidden_states": hidden_states})

        # Final layer norm.
        if self.post_layer_norm:
            hidden_states = self.final_layernorm(hidden_states)

        return hidden_states
```
**EN:** Method `GLMTransformer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `GLMTransformer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `ChatGLMModel` (lines 318-422)
```python
@support_torch_compile
class ChatGLMModel(nn.Module, SupportsQuant):
    packed_modules_mapping = {
        "linear_proj.merged_proj": [
            "linear_proj.gate_proj",
            "linear_proj.dense_h_to_4h",
        ]
    }

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.config = config

        self.embedding = VocabParallelEmbedding(
            config.padded_vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=f"{prefix}.embedding",
        )
```
**EN:** Class `ChatGLMModel` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsQuant. Key methods include __init__, embed_input_ids, forward, load_weights.
**CN:** 类 `ChatGLMModel` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsQuant。 关键方法包括 __init__, embed_input_ids, forward, load_weights。

### Method `ChatGLMModel.__init__` (lines 326-358)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.config = config

        self.embedding = VocabParallelEmbedding(
            config.padded_vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=f"{prefix}.embedding",
        )

        self.num_layers = config.num_layers
        self.multi_query_group_num = config.multi_query_group_num
        self.kv_channels = config.kv_channels
        self.encoder = GLMTransformer(
            config, cache_config, quant_config, prefix=f"{prefix}.encoder"
        )

        self.output_layer = ParallelLMHead(
            config.padded_vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=f"{prefix}.output_layer",
        )

        self.make_empty_intermediate_tensors = (
            self.encoder.make_empty_intermediate_tensors
        )
```
**EN:** Method `ChatGLMModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `ChatGLMModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `ChatGLMModel.embed_input_ids` (lines 360-361)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.embedding(input_ids)
```
**EN:** Method `ChatGLMModel.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `ChatGLMModel.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `ChatGLMModel.forward` (lines 363-386)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        **kwargs: object,
    ) -> torch.Tensor | IntermediateTensors:
        if get_pp_group().is_first_rank:
            if inputs_embeds is not None:
                hidden_states = inputs_embeds
            else:
                hidden_states = self.embed_input_ids(input_ids)
        else:
            assert intermediate_tensors is not None
            hidden_states = intermediate_tensors["hidden_states"]

        # Run encoder.
        hidden_states = self.encoder(
            hidden_states=hidden_states,
            position_ids=positions,
        )

        return hidden_states
```
**EN:** Method `ChatGLMModel.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `ChatGLMModel.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Method `ChatGLMModel.load_weights` (lines 388-422)
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("linear_proj.merged_proj", "linear_proj.gate_proj", 0),
            ("linear_proj.merged_proj", "linear_proj.dense_h_to_4h", 1),
        ]
        params_dict = dict(self.named_parameters())
        loaded_params: set[str] = set()

        for name, loaded_weight in weights:
            for param_name, weight_name, shard_id in stacked_params_mapping:
                if weight_name not in name:
                    continue
                name = name.replace(weight_name, param_name)
                # Skip loading extra bias for GPTQ models.
                if name.endswith(".bias") and name not in params_dict:
                    continue
                if is_pp_missing_parameter(name, self):
                    continue
                param = params_dict[name]
                weight_loader = param.weight_loader
                weight_loader(param, loaded_weight, shard_id)
                break
            else:
                if "rotary_pos_emb.inv_freq" in name:
                    continue
                if name.endswith(".bias") and name not in params_dict:
                    continue
                if is_pp_missing_parameter(name, self):
                    continue
                param = params_dict[name]
                weight_loader = getattr(param, "weight_loader", default_weight_loader)
                weight_loader(param, loaded_weight)
            loaded_params.add(name)
        return loaded_params
```
**EN:** Method `ChatGLMModel.load_weights` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `ChatGLMModel.load_weights` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Class `ChatGLMBaseModel` (lines 425-470)
```python
class ChatGLMBaseModel(nn.Module):
    hf_to_vllm_mapper = WeightsMapper(
        orig_to_new_substr={".word_embeddings": ""},
    )

    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        transformer_type: type[ChatGLMModel] = ChatGLMModel,
    ) -> None:
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        multimodal_config = vllm_config.model_config.multimodal_config
        self.config = config
        self.multimodal_config = multimodal_config

        self.quant_config = quant_config
        self.max_position_embeddings = getattr(config, "max_sequence_length", 8192)
        self.transformer = transformer_type(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "transformer")
        )
```
**EN:** Class `ChatGLMBaseModel` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, embed_input_ids, compute_logits, load_weights.
**CN:** 类 `ChatGLMBaseModel` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, embed_input_ids, compute_logits, load_weights。

### Method `ChatGLMBaseModel.__init__` (lines 430-456)
```python
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        transformer_type: type[ChatGLMModel] = ChatGLMModel,
    ) -> None:
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        multimodal_config = vllm_config.model_config.multimodal_config
        self.config = config
        self.multimodal_config = multimodal_config

        self.quant_config = quant_config
        self.max_position_embeddings = getattr(config, "max_sequence_length", 8192)
        self.transformer = transformer_type(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "transformer")
        )
        if self.config.tie_word_embeddings:
            self.transformer.output_layer.weight = self.transformer.embedding.weight
        self.lm_head = self.transformer.output_layer
        self.logits_processor = LogitsProcessor(config.padded_vocab_size)
        self.make_empty_intermediate_tensors = (
            self.transformer.make_empty_intermediate_tensors
        )
```
**EN:** Method `ChatGLMBaseModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `ChatGLMBaseModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `ChatGLMBaseModel.embed_input_ids` (lines 458-459)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.transformer.embed_input_ids(input_ids)
```
**EN:** Method `ChatGLMBaseModel.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `ChatGLMBaseModel.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `ChatGLMBaseModel.compute_logits` (lines 461-466)
```python
    def compute_logits(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor | None:
        logits = self.logits_processor(self.lm_head, hidden_states)
        return logits
```
**EN:** Method `ChatGLMBaseModel.compute_logits` converts hidden states into prediction-ready outputs, often by applying a head, post-processing, or sampling logic.
**CN:** Method `ChatGLMBaseModel.compute_logits` 将隐藏状态转成可预测输出，通常会经过输出头、后处理或采样逻辑。

### Method `ChatGLMBaseModel.load_weights` (lines 468-470)
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]):
        loader = AutoWeightsLoader(self)
        return loader.load_weights(weights, mapper=self.hf_to_vllm_mapper)
```
**EN:** Method `ChatGLMBaseModel.load_weights` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `ChatGLMBaseModel.load_weights` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Class `ChatGLMForCausalLM` (lines 473-502)
```python
class ChatGLMForCausalLM(ChatGLMBaseModel, SupportsLoRA, SupportsPP, SupportsQuant):
    packed_modules_mapping = {
        "query_key_value": ["query_key_value"],
        "dense_h_to_4h": ["dense_h_to_4h"],
    }

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        config = vllm_config.model_config.hf_config
        if hasattr(config, "vision_config"):
            hf_overrides = {"architectures": ["GLM4VForCausalLM"]}
            raise RuntimeError(
                "The configuration of this model indicates that it supports "
                "vision inputs, but you instantiated the text-only version "
                "of this model. Please use the vision model by setting "
                f"`--hf-overrides '{json.dumps(hf_overrides)}'`"
            )

        super().__init__(vllm_config=vllm_config, prefix=prefix)

    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
```
**EN:** Class `ChatGLMForCausalLM` organizes related behavior for this model family or helper component. It inherits from ChatGLMBaseModel, SupportsLoRA, SupportsPP, SupportsQuant. Key methods include __init__, forward.
**CN:** 类 `ChatGLMForCausalLM` 用于组织该模型族或辅助组件的相关行为。 它继承自 ChatGLMBaseModel、SupportsLoRA、SupportsPP、SupportsQuant。 关键方法包括 __init__, forward。

### Method `ChatGLMForCausalLM.__init__` (lines 479-490)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        config = vllm_config.model_config.hf_config
        if hasattr(config, "vision_config"):
            hf_overrides = {"architectures": ["GLM4VForCausalLM"]}
            raise RuntimeError(
                "The configuration of this model indicates that it supports "
                "vision inputs, but you instantiated the text-only version "
                "of this model. Please use the vision model by setting "
                f"`--hf-overrides '{json.dumps(hf_overrides)}'`"
            )

        super().__init__(vllm_config=vllm_config, prefix=prefix)
```
**EN:** Method `ChatGLMForCausalLM.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `ChatGLMForCausalLM.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `ChatGLMForCausalLM.forward` (lines 492-502)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor | IntermediateTensors:
        hidden_states = self.transformer(
            input_ids, positions, intermediate_tensors, inputs_embeds
        )
        return hidden_states
```
**EN:** Method `ChatGLMForCausalLM.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `ChatGLMForCausalLM.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

## Key Concepts / 关键概念
- **Attention / 注意力**
  - **EN:** Attention layers model token-to-token or token-to-feature interactions inside the architecture.
  - **CN:** 注意力层负责建模 token 与 token，或 token 与特征之间的交互。
- **LoRA support / LoRA 支持**
  - **EN:** The file is aware of low-rank adaptation interfaces or compatible parameter layouts.
  - **CN:** 该文件考虑了 LoRA 接口或兼容的低秩参数布局。
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **Quantization / 量化**
  - **EN:** Some execution paths are adapted to quantized weights, kernels, or configuration branches.
  - **CN:** 部分执行路径会适配量化权重、算子或配置分支。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import json`, `from collections.abc import Iterable`, `from itertools import islice`
- **Third-party / 第三方**: `import torch`, `from torch import nn`, `from torch.nn import LayerNorm`
- **vLLM internal / vLLM 内部依赖**: `from vllm.compilation.decorators import support_torch_compile`, `from vllm.config import CacheConfig, VllmConfig`, `from vllm.distributed import get_pp_group, get_tensor_model_parallel_world_size`, `from vllm.model_executor.layers.activation import SiluAndMul`, `from vllm.model_executor.layers.attention import Attention`, `from vllm.model_executor.layers.layernorm import RMSNorm`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.logits_processor import LogitsProcessor`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.layers.rotary_embedding import get_rope`, `from vllm.model_executor.layers.vocab_parallel_embedding import (`, `from vllm.model_executor.model_loader.weight_utils import default_weight_loader`
- **Module note / 模块说明**: **EN:** Inference-only ChatGLM model compatible with THUDM weights. **CN:** 模块文档字符串给出的原始说明是：Inference-only ChatGLM model compatible with THUDM weights.。
