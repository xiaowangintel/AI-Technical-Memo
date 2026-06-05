# bert.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/bert.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for bert, including architecture wrappers and weight loading logic. / 面向推理的 bert vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-46)
```python
from collections.abc import Iterable, Set

import torch
from torch import nn
from transformers import BertConfig

from vllm.compilation.decorators import support_torch_compile
from vllm.config import CacheConfig, ModelConfig, PoolerConfig, VllmConfig
from vllm.distributed import get_tensor_model_parallel_world_size
from vllm.model_executor.layers.activation import get_act_fn
from vllm.model_executor.layers.attention import (
    EncoderOnlyAttention,
)
from vllm.model_executor.layers.linear import (
    ColumnParallelLinear,
    QKVParallelLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.pooler import (
    DispatchPooler,
    Pooler,
    PoolingParamsUpdate,
)
from vllm.model_executor.layers.pooler.activations import LambdaPoolerActivation
from vllm.model_executor.layers.pooler.seqwise import (
    EmbeddingPoolerHead,
    SequencePooler,
    SequencePoolerOutput,
    get_seq_pooling_method,
)
from vllm.model_executor.layers.pooler.tokwise import (
    pooler_for_token_classify,
    pooler_for_token_embed,
)
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.vocab_parallel_embedding import VocabParallelEmbedding
from vllm.sequence import IntermediateTensors
from vllm.tasks import PoolingTask
from vllm.v1.pool.metadata import PoolingMetadata

from .interfaces import SupportsCrossEncoding, SupportsQuant
from .interfaces_base import attn_type, default_pooling_type
from .utils import AutoWeightsLoader, WeightsMapper, maybe_prefix
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Constants / assignments (lines 551-551)
```python
TOKEN_TYPE_SHIFT = 30
```
**EN:** This block defines TOKEN_TYPE_SHIFT, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 TOKEN_TYPE_SHIFT，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Function `_encode_token_type_ids` (lines 554-558)
```python
def _encode_token_type_ids(
    input_ids: torch.Tensor, token_type_ids: torch.Tensor
) -> None:
    # input_ids can be padded to the right
    input_ids[: token_type_ids.shape[0]].bitwise_or_(token_type_ids << TOKEN_TYPE_SHIFT)
```
**EN:** Function `_encode_token_type_ids` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `_encode_token_type_ids` 封装了该模块中的一段可复用核心逻辑。

### Function `_decode_token_type_ids` (lines 561-572)
```python
def _decode_token_type_ids(input_ids: torch.Tensor) -> torch.Tensor:
    ids_mask = (
        torch.ones_like(input_ids, dtype=torch.int32, device=input_ids.device)
        << TOKEN_TYPE_SHIFT
    )
    tokens_mask = ids_mask.bitwise_not()

    token_type_ids = input_ids.bitwise_and(ids_mask) >> TOKEN_TYPE_SHIFT

    input_ids.bitwise_and_(tokens_mask)

    return token_type_ids
```
**EN:** Function `_decode_token_type_ids` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `_decode_token_type_ids` 封装了该模块中的一段可复用核心逻辑。

### Class `BertEmbedding` (lines 49-93)
```python
class BertEmbedding(nn.Module):
    def __init__(self, config: BertConfig):
        super().__init__()
        self.size = config.hidden_size
        self.word_embeddings = VocabParallelEmbedding(
            config.vocab_size, config.hidden_size
        )
        self.position_embeddings = VocabParallelEmbedding(
            config.max_position_embeddings, config.hidden_size
        )
        self.token_type_embeddings = VocabParallelEmbedding(
            config.type_vocab_size, config.hidden_size
        )
        self.LayerNorm = nn.LayerNorm(config.hidden_size, eps=config.layer_norm_eps)

        self.register_buffer(
            "position_ids",
            torch.arange(config.max_position_embeddings).unsqueeze(0),
        )
        self.position_embedding_type = getattr(
            config, "position_embedding_type", "absolute"
        )
        if self.position_embedding_type != "absolute":
            raise ValueError(
                "Only 'absolute' position_embedding_type" + " is supported"
```
**EN:** Class `BertEmbedding` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `BertEmbedding` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `BertEmbedding.__init__` (lines 50-74)
```python
    def __init__(self, config: BertConfig):
        super().__init__()
        self.size = config.hidden_size
        self.word_embeddings = VocabParallelEmbedding(
            config.vocab_size, config.hidden_size
        )
        self.position_embeddings = VocabParallelEmbedding(
            config.max_position_embeddings, config.hidden_size
        )
        self.token_type_embeddings = VocabParallelEmbedding(
            config.type_vocab_size, config.hidden_size
        )
        self.LayerNorm = nn.LayerNorm(config.hidden_size, eps=config.layer_norm_eps)

        self.register_buffer(
            "position_ids",
            torch.arange(config.max_position_embeddings).unsqueeze(0),
        )
        self.position_embedding_type = getattr(
            config, "position_embedding_type", "absolute"
        )
        if self.position_embedding_type != "absolute":
            raise ValueError(
                "Only 'absolute' position_embedding_type" + " is supported"
            )
```
**EN:** Method `BertEmbedding.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `BertEmbedding.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `BertEmbedding.forward` (lines 76-93)
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        position_ids: torch.Tensor,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor:
        token_type_ids = _decode_token_type_ids(input_ids)

        if inputs_embeds is None:
            inputs_embeds = self.word_embeddings(input_ids)

        position_embeddings = self.position_embeddings(position_ids)

        token_type_embeddings = self.token_type_embeddings(token_type_ids)

        embeddings = inputs_embeds + token_type_embeddings + position_embeddings
        embeddings = self.LayerNorm(embeddings)
        return embeddings
```
**EN:** Method `BertEmbedding.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `BertEmbedding.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `BertPooler` (lines 96-122)
```python
class BertPooler(SequencePooler):
    def __init__(self, model_config: ModelConfig):
        pooler_config = model_config.pooler_config
        assert pooler_config is not None

        config: BertConfig = model_config.hf_config

        super().__init__(
            pooling=get_seq_pooling_method(pooler_config.seq_pooling_type),
            # We set this dummy to avoid adding parameters to nn.Module too early
            head=nn.Identity(),
        )

        head_dtype = model_config.head_dtype
        self.dense = nn.Linear(
            config.hidden_size,
            config.hidden_size,
            dtype=head_dtype,
        )
        self.act_fn = nn.Tanh()

        # Use lambdas so that weights are not registered under `self.head`
        self.head = EmbeddingPoolerHead(
            head_dtype=head_dtype,
            projector=lambda x: self.dense(x),
```
**EN:** Class `BertPooler` is a structural model block in the vLLM execution graph. It inherits from SequencePooler. Key methods include __init__.
**CN:** 类 `BertPooler` 是 vLLM 执行图中的结构化模型模块。 它继承自 SequencePooler。 关键方法包括 __init__。

### Method `BertPooler.__init__` (lines 97-122)
```python
    def __init__(self, model_config: ModelConfig):
        pooler_config = model_config.pooler_config
        assert pooler_config is not None

        config: BertConfig = model_config.hf_config

        super().__init__(
            pooling=get_seq_pooling_method(pooler_config.seq_pooling_type),
            # We set this dummy to avoid adding parameters to nn.Module too early
            head=nn.Identity(),
        )

        head_dtype = model_config.head_dtype
        self.dense = nn.Linear(
            config.hidden_size,
            config.hidden_size,
            dtype=head_dtype,
        )
        self.act_fn = nn.Tanh()

        # Use lambdas so that weights are not registered under `self.head`
        self.head = EmbeddingPoolerHead(
            head_dtype=head_dtype,
            projector=lambda x: self.dense(x),
            activation=LambdaPoolerActivation(self.act_fn),
        )
```
**EN:** Method `BertPooler.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `BertPooler.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Class `BertEncoder` (lines 125-149)
```python
class BertEncoder(nn.Module):
    def __init__(self, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config
        self.layer = nn.ModuleList(
            [
                BertLayer(
                    config=config,
                    cache_config=cache_config,
                    quant_config=quant_config,
                    prefix=f"{prefix}.layer.{layer_idx}",
                )
                for layer_idx in range(config.num_hidden_layers)
            ]
        )

    def forward(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        for layer in self.layer:
            hidden_states = layer(hidden_states)
        return hidden_states
```
**EN:** Class `BertEncoder` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `BertEncoder` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `BertEncoder.__init__` (lines 126-141)
```python
    def __init__(self, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config
        self.layer = nn.ModuleList(
            [
                BertLayer(
                    config=config,
                    cache_config=cache_config,
                    quant_config=quant_config,
                    prefix=f"{prefix}.layer.{layer_idx}",
                )
                for layer_idx in range(config.num_hidden_layers)
            ]
        )
```
**EN:** Method `BertEncoder.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `BertEncoder.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `BertEncoder.forward` (lines 143-149)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        for layer in self.layer:
            hidden_states = layer(hidden_states)
        return hidden_states
```
**EN:** Method `BertEncoder.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `BertEncoder.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `BertLayer` (lines 152-191)
```python
class BertLayer(nn.Module):
    def __init__(
        self,
        config: BertConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()

        self.attention = BertAttention(
            hidden_size=config.hidden_size,
            num_attention_heads=config.num_attention_heads,
            layer_norm_eps=config.layer_norm_eps,
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.attention",
        )

        self.intermediate = BertIntermediate(
            hidden_size=config.hidden_size,
            intermediate_size=config.intermediate_size,
            hidden_act=config.hidden_act,
            quant_config=quant_config,
            prefix=f"{prefix}.intermediate",
```
**EN:** Class `BertLayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `BertLayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `BertLayer.__init__` (lines 153-185)
```python
    def __init__(
        self,
        config: BertConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()

        self.attention = BertAttention(
            hidden_size=config.hidden_size,
            num_attention_heads=config.num_attention_heads,
            layer_norm_eps=config.layer_norm_eps,
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.attention",
        )

        self.intermediate = BertIntermediate(
            hidden_size=config.hidden_size,
            intermediate_size=config.intermediate_size,
            hidden_act=config.hidden_act,
            quant_config=quant_config,
            prefix=f"{prefix}.intermediate",
        )

        self.output = BertOutput(
            hidden_size=config.hidden_size,
            intermediate_size=config.intermediate_size,
            layer_norm_eps=config.layer_norm_eps,
            quant_config=quant_config,
            prefix=f"{prefix}.output",
        )
```
**EN:** Method `BertLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `BertLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `BertLayer.forward` (lines 187-191)
```python
    def forward(self, hidden_states: torch.Tensor):
        attn_output = self.attention(hidden_states)
        intermediate_output = self.intermediate(attn_output)
        output = self.output(intermediate_output, attn_output)
        return output
```
**EN:** Method `BertLayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `BertLayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `BertAttention` (lines 194-226)
```python
class BertAttention(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        num_attention_heads: int,
        layer_norm_eps: float,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()

        self.self = BertSelfAttention(
            hidden_size=hidden_size,
            num_attention_heads=num_attention_heads,
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.output",
        )

        self.output = BertSelfOutput(
            hidden_size=hidden_size,
            layer_norm_eps=layer_norm_eps,
            quant_config=quant_config,
            prefix=f"{prefix}.output",
```
**EN:** Class `BertAttention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `BertAttention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `BertAttention.__init__` (lines 195-219)
```python
    def __init__(
        self,
        hidden_size: int,
        num_attention_heads: int,
        layer_norm_eps: float,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()

        self.self = BertSelfAttention(
            hidden_size=hidden_size,
            num_attention_heads=num_attention_heads,
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.output",
        )

        self.output = BertSelfOutput(
            hidden_size=hidden_size,
            layer_norm_eps=layer_norm_eps,
            quant_config=quant_config,
            prefix=f"{prefix}.output",
        )
```
**EN:** Method `BertAttention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `BertAttention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `BertAttention.forward` (lines 221-226)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        self_output = self.self(hidden_states)
        return self.output(self_output, hidden_states)
```
**EN:** Method `BertAttention.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `BertAttention.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `BertSelfAttention` (lines 229-282)
```python
class BertSelfAttention(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        num_attention_heads: int,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = hidden_size
        tp_size = get_tensor_model_parallel_world_size()

        self.total_num_heads = num_attention_heads
        assert self.total_num_heads % tp_size == 0

        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = self.total_num_heads
        self.head_dim = self.hidden_size // self.total_num_heads
        assert self.head_dim * self.total_num_heads == self.hidden_size

        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)

        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
```
**EN:** Class `BertSelfAttention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `BertSelfAttention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `BertSelfAttention.__init__` (lines 230-273)
```python
    def __init__(
        self,
        hidden_size: int,
        num_attention_heads: int,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = hidden_size
        tp_size = get_tensor_model_parallel_world_size()

        self.total_num_heads = num_attention_heads
        assert self.total_num_heads % tp_size == 0

        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = self.total_num_heads
        self.head_dim = self.hidden_size // self.total_num_heads
        assert self.head_dim * self.total_num_heads == self.hidden_size

        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)

        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5
        self.qkv_proj = QKVParallelLinear(
            hidden_size=self.hidden_size,
            head_size=self.head_dim,
            total_num_heads=self.total_num_heads,
            total_num_kv_heads=self.total_num_kv_heads,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.qkv_proj",
        )

        self.attn = EncoderOnlyAttention(
            num_heads=self.num_heads,
            head_size=self.head_dim,
            scale=self.scaling,
            num_kv_heads=self.num_kv_heads,
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.attn",
        )
```
**EN:** Method `BertSelfAttention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `BertSelfAttention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `BertSelfAttention.forward` (lines 275-282)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        output = self.attn(q, k, v)
        return output
```
**EN:** Method `BertSelfAttention.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `BertSelfAttention.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `BertSelfOutput` (lines 285-308)
```python
class BertSelfOutput(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        layer_norm_eps: float,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.dense = RowParallelLinear(
            input_size=hidden_size,
            output_size=hidden_size,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.dense",
        )
        self.LayerNorm = nn.LayerNorm(hidden_size, eps=layer_norm_eps)

    def forward(
        self, hidden_states: torch.Tensor, input_tensor: torch.Tensor
    ) -> torch.Tensor:
        hidden_states, _ = self.dense(hidden_states)
        hidden_states = self.LayerNorm(hidden_states + input_tensor)
        return hidden_states
```
**EN:** Class `BertSelfOutput` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `BertSelfOutput` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `BertSelfOutput.__init__` (lines 286-301)
```python
    def __init__(
        self,
        hidden_size: int,
        layer_norm_eps: float,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.dense = RowParallelLinear(
            input_size=hidden_size,
            output_size=hidden_size,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.dense",
        )
        self.LayerNorm = nn.LayerNorm(hidden_size, eps=layer_norm_eps)
```
**EN:** Method `BertSelfOutput.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `BertSelfOutput.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `BertSelfOutput.forward` (lines 303-308)
```python
    def forward(
        self, hidden_states: torch.Tensor, input_tensor: torch.Tensor
    ) -> torch.Tensor:
        hidden_states, _ = self.dense(hidden_states)
        hidden_states = self.LayerNorm(hidden_states + input_tensor)
        return hidden_states
```
**EN:** Method `BertSelfOutput.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `BertSelfOutput.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `BertIntermediate` (lines 311-333)
```python
class BertIntermediate(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.dense = ColumnParallelLinear(
            input_size=hidden_size,
            output_size=intermediate_size,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.dense",
        )
        self.intermediate_act_fn = get_act_fn(hidden_act)

    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states, _ = self.dense(hidden_states)
        hidden_states = self.intermediate_act_fn(hidden_states)
        return hidden_states
```
**EN:** Class `BertIntermediate` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `BertIntermediate` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `BertIntermediate.__init__` (lines 312-328)
```python
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.dense = ColumnParallelLinear(
            input_size=hidden_size,
            output_size=intermediate_size,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.dense",
        )
        self.intermediate_act_fn = get_act_fn(hidden_act)
```
**EN:** Method `BertIntermediate.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `BertIntermediate.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `BertIntermediate.forward` (lines 330-333)
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states, _ = self.dense(hidden_states)
        hidden_states = self.intermediate_act_fn(hidden_states)
        return hidden_states
```
**EN:** Method `BertIntermediate.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `BertIntermediate.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `BertOutput` (lines 336-362)
```python
class BertOutput(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        layer_norm_eps: float,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()

        self.dense = RowParallelLinear(
            input_size=intermediate_size,
            output_size=hidden_size,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.dense",
        )

        self.LayerNorm = nn.LayerNorm(hidden_size, eps=layer_norm_eps)

    def forward(
        self, hidden_states: torch.Tensor, input_tensor: torch.Tensor
    ) -> torch.Tensor:
        hidden_states, _ = self.dense(hidden_states)
```
**EN:** Class `BertOutput` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `BertOutput` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `BertOutput.__init__` (lines 337-355)
```python
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        layer_norm_eps: float,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()

        self.dense = RowParallelLinear(
            input_size=intermediate_size,
            output_size=hidden_size,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.dense",
        )

        self.LayerNorm = nn.LayerNorm(hidden_size, eps=layer_norm_eps)
```
**EN:** Method `BertOutput.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `BertOutput.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `BertOutput.forward` (lines 357-362)
```python
    def forward(
        self, hidden_states: torch.Tensor, input_tensor: torch.Tensor
    ) -> torch.Tensor:
        hidden_states, _ = self.dense(hidden_states)
        hidden_states = self.LayerNorm(hidden_states + input_tensor)
        return hidden_states
```
**EN:** Method `BertOutput.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `BertOutput.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `BertModel` (lines 367-439)
```python
@support_torch_compile
@default_pooling_type(seq_pooling_type="CLS")
class BertModel(nn.Module, SupportsQuant):
    is_pooling_model = True

    packed_modules_mapping = {"qkv_proj": ["query", "key", "value"]}

    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        embedding_class: type[nn.Module] = BertEmbedding,
    ) -> None:
        super().__init__()

        self.config = vllm_config.model_config.hf_config
        self.embeddings = embedding_class(self.config)
        self.encoder = BertEncoder(vllm_config=vllm_config, prefix=f"{prefix}.encoder")

    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.embeddings.word_embeddings(input_ids)

    def forward(
        self,
```
**EN:** Class `BertModel` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsQuant. Key methods include __init__, embed_input_ids, forward, _load_weights, load_weights.
**CN:** 类 `BertModel` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsQuant。 关键方法包括 __init__, embed_input_ids, forward, _load_weights, load_weights。

### Method `BertModel.__init__` (lines 372-383)
```python
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        embedding_class: type[nn.Module] = BertEmbedding,
    ) -> None:
        super().__init__()

        self.config = vllm_config.model_config.hf_config
        self.embeddings = embedding_class(self.config)
        self.encoder = BertEncoder(vllm_config=vllm_config, prefix=f"{prefix}.encoder")
```
**EN:** Method `BertModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `BertModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `BertModel.embed_input_ids` (lines 385-386)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.embeddings.word_embeddings(input_ids)
```
**EN:** Method `BertModel.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `BertModel.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

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
- **Standard library / 标准库**: `from collections.abc import Iterable, Set`
- **Third-party / 第三方**: `import torch`, `from torch import nn`, `from transformers import BertConfig`
- **vLLM internal / vLLM 内部依赖**: `from vllm.compilation.decorators import support_torch_compile`, `from vllm.config import CacheConfig, ModelConfig, PoolerConfig, VllmConfig`, `from vllm.distributed import get_tensor_model_parallel_world_size`, `from vllm.model_executor.layers.activation import get_act_fn`, `from vllm.model_executor.layers.attention import (`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.pooler import (`, `from vllm.model_executor.layers.pooler.activations import LambdaPoolerActivation`, `from vllm.model_executor.layers.pooler.seqwise import (`, `from vllm.model_executor.layers.pooler.tokwise import (`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.layers.vocab_parallel_embedding import VocabParallelEmbedding`
