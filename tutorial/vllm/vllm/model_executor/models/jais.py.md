# jais.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/jais.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for jais, including architecture wrappers and weight loading logic. / 面向推理的 jais vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 24-61)
```python
import math
from collections.abc import Iterable
from itertools import islice

import torch
from torch import nn

from vllm.compilation.decorators import support_torch_compile
from vllm.config import CacheConfig, VllmConfig
from vllm.distributed import (
    get_pp_group,
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
)
from vllm.model_executor.layers.attention import Attention
from vllm.model_executor.layers.linear import (
    ColumnParallelLinear,
    QKVParallelLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.logits_processor import LogitsProcessor
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.vocab_parallel_embedding import (
    ParallelLMHead,
    VocabParallelEmbedding,
)
from vllm.model_executor.model_loader.weight_utils import default_weight_loader
from vllm.sequence import IntermediateTensors
from vllm.transformers_utils.configs.jais import JAISConfig

from .interfaces import SupportsPP
from .utils import (
    AutoWeightsLoader,
    is_pp_missing_parameter,
    make_empty_intermediate_tensors_factory,
    make_layers,
    maybe_prefix,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Function `_get_alibi_slopes` (lines 69-82)
```python
def _get_alibi_slopes(n):
    def get_slopes_power_of_2(n):
        start = 2 ** (-(2 ** -(math.log2(n) - 3)))
        ratio = start
        return [start * ratio**i for i in range(n)]

    if math.log2(n).is_integer():
        return get_slopes_power_of_2(n)
    else:
        closest_power_of_2 = 2 ** math.floor(math.log2(n))
        return (
            get_slopes_power_of_2(closest_power_of_2)
            + _get_alibi_slopes(2 * closest_power_of_2)[0::2][: n - closest_power_of_2]
        )
```
**EN:** Function `_get_alibi_slopes` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `_get_alibi_slopes` 封装了该模块中的一段可复用核心逻辑。

### Class `SwiGLUActivation` (lines 64-66)
```python
class SwiGLUActivation(nn.Module):
    def forward(self, x1: torch.Tensor, x2: torch.Tensor) -> torch.Tensor:
        return x1 * nn.functional.silu(x2)
```
**EN:** Class `SwiGLUActivation` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include forward.
**CN:** 类 `SwiGLUActivation` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 forward。

### Method `SwiGLUActivation.forward` (lines 65-66)
```python
    def forward(self, x1: torch.Tensor, x2: torch.Tensor) -> torch.Tensor:
        return x1 * nn.functional.silu(x2)
```
**EN:** Method `SwiGLUActivation.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `SwiGLUActivation.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `JAISAttention` (lines 85-147)
```python
class JAISAttention(nn.Module):
    def __init__(
        self,
        config: JAISConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = config.hidden_size
        total_num_heads = config.num_attention_heads
        tensor_model_parallel_world_size = get_tensor_model_parallel_world_size()
        assert total_num_heads % tensor_model_parallel_world_size == 0
        self.num_heads = total_num_heads // tensor_model_parallel_world_size
        self.head_dim = self.hidden_size // total_num_heads
        if hasattr(config, "scale_qk_dot_by_d"):
            config.mup_scale_qk_dot_by_d = config.scale_qk_dot_by_d
        self.attn_scale_power = 1.0 if config.mup_scale_qk_dot_by_d else 0.5
        self.scale = self.head_dim**-self.attn_scale_power

        self.c_attn = QKVParallelLinear(
            self.hidden_size,
            self.head_dim,
            total_num_heads,
            bias=True,
```
**EN:** Class `JAISAttention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `JAISAttention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `JAISAttention.__init__` (lines 86-137)
```python
    def __init__(
        self,
        config: JAISConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = config.hidden_size
        total_num_heads = config.num_attention_heads
        tensor_model_parallel_world_size = get_tensor_model_parallel_world_size()
        assert total_num_heads % tensor_model_parallel_world_size == 0
        self.num_heads = total_num_heads // tensor_model_parallel_world_size
        self.head_dim = self.hidden_size // total_num_heads
        if hasattr(config, "scale_qk_dot_by_d"):
            config.mup_scale_qk_dot_by_d = config.scale_qk_dot_by_d
        self.attn_scale_power = 1.0 if config.mup_scale_qk_dot_by_d else 0.5
        self.scale = self.head_dim**-self.attn_scale_power

        self.c_attn = QKVParallelLinear(
            self.hidden_size,
            self.head_dim,
            total_num_heads,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.c_attn",
        )
        self.c_proj = RowParallelLinear(
            self.hidden_size,
            self.hidden_size,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.c_proj",
        )

        self.use_alibi = config.position_embedding_type == "alibi"
        alibi_slopes = None
        if self.use_alibi:
            tp_rank = get_tensor_model_parallel_rank()
            head_start = tp_rank * self.num_heads
            head_end = (tp_rank + 1) * self.num_heads
            alibi_slopes = _get_alibi_slopes(total_num_heads)
            alibi_slopes = alibi_slopes[head_start:head_end]
        self.attn = Attention(
            self.num_heads,
            self.head_dim,
            scale=self.scale,
            alibi_slopes=alibi_slopes,
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.attn",
        )
```
**EN:** Method `JAISAttention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `JAISAttention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `JAISAttention.forward` (lines 139-147)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        qkv, _ = self.c_attn(hidden_states)
        q, k, v = qkv.chunk(chunks=3, dim=-1)
        attn_output = self.attn(q, k, v)
        attn_output, _ = self.c_proj(attn_output)
        return attn_output
```
**EN:** Method `JAISAttention.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `JAISAttention.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `JAISMLP` (lines 150-199)
```python
class JAISMLP(nn.Module):
    def __init__(
        self,
        intermediate_size: int,
        config: JAISConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        hidden_size = config.hidden_size
        self.swiglu = config.activation_function == "swiglu"
        self.c_fc = ColumnParallelLinear(
            hidden_size,
            intermediate_size,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.c_fc",
        )
        self.c_fc2 = (
            ColumnParallelLinear(
                hidden_size,
                intermediate_size,
                bias=True,
                quant_config=quant_config,
                prefix=f"{prefix}.c_fc2",
```
**EN:** Class `JAISMLP` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `JAISMLP` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `JAISMLP.__init__` (lines 151-187)
```python
    def __init__(
        self,
        intermediate_size: int,
        config: JAISConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        hidden_size = config.hidden_size
        self.swiglu = config.activation_function == "swiglu"
        self.c_fc = ColumnParallelLinear(
            hidden_size,
            intermediate_size,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.c_fc",
        )
        self.c_fc2 = (
            ColumnParallelLinear(
                hidden_size,
                intermediate_size,
                bias=True,
                quant_config=quant_config,
                prefix=f"{prefix}.c_fc2",
            )
            if self.swiglu
            else None
        )
        self.c_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.c_proj",
        )

        self.act = SwiGLUActivation()
```
**EN:** Method `JAISMLP.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `JAISMLP.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `JAISMLP.forward` (lines 189-199)
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        if self.swiglu:
            hidden_states2, _ = self.c_fc2(hidden_states)
        hidden_states, _ = self.c_fc(hidden_states)
        hidden_states = (
            self.act(hidden_states, hidden_states2)
            if self.swiglu
            else self.act(hidden_states)
        )
        hidden_states, _ = self.c_proj(hidden_states)
        return hidden_states
```
**EN:** Method `JAISMLP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `JAISMLP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `JAISBlock` (lines 202-238)
```python
class JAISBlock(nn.Module):
    def __init__(
        self,
        config: JAISConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        hidden_size = config.hidden_size
        inner_dim = config.n_inner if config.n_inner is not None else 4 * hidden_size

        self.ln_1 = nn.LayerNorm(hidden_size, eps=config.layer_norm_epsilon)
        self.attn = JAISAttention(
            config, cache_config, quant_config, prefix=f"{prefix}.attn"
        )
        self.ln_2 = nn.LayerNorm(hidden_size, eps=config.layer_norm_epsilon)
        self.mlp = JAISMLP(inner_dim, config, quant_config, prefix=f"{prefix}.mlp")

    def forward(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        residual = hidden_states
        hidden_states = self.ln_1(hidden_states)
```
**EN:** Class `JAISBlock` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `JAISBlock` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `JAISBlock.__init__` (lines 203-219)
```python
    def __init__(
        self,
        config: JAISConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        hidden_size = config.hidden_size
        inner_dim = config.n_inner if config.n_inner is not None else 4 * hidden_size

        self.ln_1 = nn.LayerNorm(hidden_size, eps=config.layer_norm_epsilon)
        self.attn = JAISAttention(
            config, cache_config, quant_config, prefix=f"{prefix}.attn"
        )
        self.ln_2 = nn.LayerNorm(hidden_size, eps=config.layer_norm_epsilon)
        self.mlp = JAISMLP(inner_dim, config, quant_config, prefix=f"{prefix}.mlp")
```
**EN:** Method `JAISBlock.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `JAISBlock.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `JAISBlock.forward` (lines 221-238)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        residual = hidden_states
        hidden_states = self.ln_1(hidden_states)
        attn_output = self.attn(
            hidden_states=hidden_states,
        )
        # residual connection
        hidden_states = attn_output + residual

        residual = hidden_states
        hidden_states = self.ln_2(hidden_states)
        feed_forward_hidden_states = self.mlp(hidden_states)
        # residual connection
        hidden_states = residual + feed_forward_hidden_states
        return hidden_states
```
**EN:** Method `JAISBlock.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `JAISBlock.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `JAISModel` (lines 242-342)
```python
@support_torch_compile
class JAISModel(nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.config = config
        assert not config.scale_attn_by_inverse_layer_idx
        assert not config.reorder_and_upcast_attn
        self.embed_dim = config.hidden_size
        self.wte = VocabParallelEmbedding(config.vocab_size, self.embed_dim)
        self.wpe = (
            nn.Embedding(config.max_position_embeddings, self.embed_dim)
            if config.position_embedding_type != "alibi"
            else None
        )
        if hasattr(config, "embeddings_scale"):
            self.embeddings_scale = config.embeddings_scale
        else:
            self.embeddings_scale = config.mup_embeddings_scale

        self.start_layer, self.end_layer, self.h = make_layers(
```
**EN:** Class `JAISModel` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, embed_input_ids, forward, load_weights.
**CN:** 类 `JAISModel` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, embed_input_ids, forward, load_weights。

### Method `JAISModel.__init__` (lines 243-279)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.config = config
        assert not config.scale_attn_by_inverse_layer_idx
        assert not config.reorder_and_upcast_attn
        self.embed_dim = config.hidden_size
        self.wte = VocabParallelEmbedding(config.vocab_size, self.embed_dim)
        self.wpe = (
            nn.Embedding(config.max_position_embeddings, self.embed_dim)
            if config.position_embedding_type != "alibi"
            else None
        )
        if hasattr(config, "embeddings_scale"):
            self.embeddings_scale = config.embeddings_scale
        else:
            self.embeddings_scale = config.mup_embeddings_scale

        self.start_layer, self.end_layer, self.h = make_layers(
            config.num_hidden_layers,
            lambda prefix: JAISBlock(
                config=config,
                cache_config=cache_config,
                quant_config=quant_config,
                prefix=prefix,
            ),
            prefix=f"{prefix}.h",
        )

        self.ln_f = nn.LayerNorm(self.embed_dim, eps=config.layer_norm_epsilon)
        self.make_empty_intermediate_tensors = make_empty_intermediate_tensors_factory(
            ["hidden_states"], config.n_embd
        )
```
**EN:** Method `JAISModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `JAISModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `JAISModel.embed_input_ids` (lines 281-282)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.wte(input_ids)
```
**EN:** Method `JAISModel.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `JAISModel.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `JAISModel.forward` (lines 284-313)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        position_ids: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> IntermediateTensors | torch.Tensor:
        if get_pp_group().is_first_rank:
            if inputs_embeds is None:
                inputs_embeds = self.embed_input_ids(input_ids)
            if self.wpe is not None:
                position_embeds = self.wpe(position_ids)
                hidden_states = inputs_embeds + position_embeds
            else:
                hidden_states = inputs_embeds
            hidden_states *= torch.tensor(
                float(self.embeddings_scale), dtype=hidden_states.dtype
            )
        else:
            assert intermediate_tensors is not None
            hidden_states = intermediate_tensors["hidden_states"]

        for layer in islice(self.h, self.start_layer, self.end_layer):
            hidden_states = layer(hidden_states)

        if not get_pp_group().is_last_rank:
            return IntermediateTensors({"hidden_states": hidden_states})

        hidden_states = self.ln_f(hidden_states)
        return hidden_states
```
**EN:** Method `JAISModel.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `JAISModel.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Method `JAISModel.load_weights` (lines 315-342)
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        params_dict = dict(self.named_parameters(remove_duplicate=False))
        loaded_params: set[str] = set()
        for name, loaded_weight in weights:
            if ".attn.bias" in name or ".attn.masked_bias" in name:
                # Skip attention mask.
                # NOTE: "c_attn.bias" should not be skipped.
                continue
            if "relative_pe" in name:
                continue

            if is_pp_missing_parameter(name, self):
                continue

            param = params_dict[name]
            # The HF's GPT-2 implementation uses Conv1D instead of Linear.
            # Because of this, we need to transpose the weights.
            # Note(zhuohan): the logic below might break quantized models.
            for conv1d_weight_name in ["c_attn", "c_proj", "c_fc"]:
                if conv1d_weight_name not in name:
                    continue
                if not name.endswith(".weight"):
                    continue
                loaded_weight = loaded_weight.t()
            weight_loader = getattr(param, "weight_loader", default_weight_loader)
            weight_loader(param, loaded_weight)
            loaded_params.add(name)
        return loaded_params
```
**EN:** Method `JAISModel.load_weights` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `JAISModel.load_weights` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Class `JAISLMHeadModel` (lines 345-401)
```python
class JAISLMHeadModel(nn.Module, SupportsPP):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        self.config = config
        self.quant_config = quant_config
        self.transformer = JAISModel(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "transformer")
        )
        if self.config.tie_word_embeddings:
            self.lm_head = self.transformer.wte
        else:
            self.lm_head = ParallelLMHead(
                self.config.vocab_size,
                self.config.hidden_size,
                prefix=maybe_prefix(prefix, "lm_head"),
            )
        if hasattr(config, "width_scale"):
            self.output_logits_scale = config.width_scale
        else:
            self.output_logits_scale = config.mup_output_alpha * config.mup_width_scale
        self.logits_processor = LogitsProcessor(
            vocab_size=config.vocab_size, scale=self.output_logits_scale
        )
```
**EN:** Class `JAISLMHeadModel` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsPP. Key methods include __init__, embed_input_ids, forward, compute_logits, load_weights.
**CN:** 类 `JAISLMHeadModel` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsPP。 关键方法包括 __init__, embed_input_ids, forward, compute_logits, load_weights。

### Method `JAISLMHeadModel.__init__` (lines 346-372)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        self.config = config
        self.quant_config = quant_config
        self.transformer = JAISModel(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "transformer")
        )
        if self.config.tie_word_embeddings:
            self.lm_head = self.transformer.wte
        else:
            self.lm_head = ParallelLMHead(
                self.config.vocab_size,
                self.config.hidden_size,
                prefix=maybe_prefix(prefix, "lm_head"),
            )
        if hasattr(config, "width_scale"):
            self.output_logits_scale = config.width_scale
        else:
            self.output_logits_scale = config.mup_output_alpha * config.mup_width_scale
        self.logits_processor = LogitsProcessor(
            vocab_size=config.vocab_size, scale=self.output_logits_scale
        )
        self.make_empty_intermediate_tensors = (
            self.transformer.make_empty_intermediate_tensors
        )
```
**EN:** Method `JAISLMHeadModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `JAISLMHeadModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `JAISLMHeadModel.embed_input_ids` (lines 374-375)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.transformer.embed_input_ids(input_ids)
```
**EN:** Method `JAISLMHeadModel.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `JAISLMHeadModel.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `JAISLMHeadModel.forward` (lines 377-387)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> IntermediateTensors | torch.Tensor:
        hidden_states = self.transformer(
            input_ids, positions, intermediate_tensors, inputs_embeds
        )
        return hidden_states
```
**EN:** Method `JAISLMHeadModel.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `JAISLMHeadModel.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Method `JAISLMHeadModel.compute_logits` (lines 389-394)
```python
    def compute_logits(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor | None:
        logits = self.logits_processor(self.lm_head, hidden_states)
        return logits
```
**EN:** Method `JAISLMHeadModel.compute_logits` converts hidden states into prediction-ready outputs, often by applying a head, post-processing, or sampling logic.
**CN:** Method `JAISLMHeadModel.compute_logits` 将隐藏状态转成可预测输出，通常会经过输出头、后处理或采样逻辑。

## Key Concepts / 关键概念
- **Attention / 注意力**
  - **EN:** Attention layers model token-to-token or token-to-feature interactions inside the architecture.
  - **CN:** 注意力层负责建模 token 与 token，或 token 与特征之间的交互。
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **Quantization / 量化**
  - **EN:** Some execution paths are adapted to quantized weights, kernels, or configuration branches.
  - **CN:** 部分执行路径会适配量化权重、算子或配置分支。
- **KV cache / KV 缓存**
  - **EN:** The runtime stores key/value state so decoding can reuse prior computation efficiently.
  - **CN:** 运行时会保存 key/value 状态，以便解码阶段高效复用历史计算。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import math`, `from collections.abc import Iterable`, `from itertools import islice`
- **Third-party / 第三方**: `import torch`, `from torch import nn`
- **vLLM internal / vLLM 内部依赖**: `from vllm.compilation.decorators import support_torch_compile`, `from vllm.config import CacheConfig, VllmConfig`, `from vllm.distributed import (`, `from vllm.model_executor.layers.attention import Attention`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.logits_processor import LogitsProcessor`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.layers.vocab_parallel_embedding import (`, `from vllm.model_executor.model_loader.weight_utils import default_weight_loader`, `from vllm.sequence import IntermediateTensors`, `from vllm.transformers_utils.configs.jais import JAISConfig`, `from .interfaces import SupportsPP`
- **Module note / 模块说明**: **EN:** Inference-only Jais model compatible with HuggingFace weights. **CN:** 模块文档字符串给出的原始说明是：Inference-only Jais model compatible with HuggingFace weights.。
