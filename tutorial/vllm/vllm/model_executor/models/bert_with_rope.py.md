# bert_with_rope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/bert_with_rope.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for bert_with_rope, including architecture wrappers and weight loading logic. / 面向推理的 bert_with_rope vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-45)
```python
from collections.abc import Iterable

import torch
from torch import nn
from transformers import PretrainedConfig

from vllm.compilation.decorators import support_torch_compile
from vllm.config import CacheConfig, VllmConfig
from vllm.distributed import (
    divide,
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
    tensor_model_parallel_all_reduce,
)
from vllm.model_executor.layers.activation import get_act_and_mul_fn, get_act_fn
from vllm.model_executor.layers.attention import (
    EncoderOnlyAttention,
)
from vllm.model_executor.layers.fused_moe import activation_without_mul, fused_topk
from vllm.model_executor.layers.linear import (
    ColumnParallelLinear,
    MergedColumnParallelLinear,
    QKVParallelLinear,
    ReplicatedLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.pooler import DispatchPooler
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.rotary_embedding import get_rope
from vllm.model_executor.layers.vocab_parallel_embedding import VocabParallelEmbedding
from vllm.model_executor.model_loader.weight_utils import default_weight_loader
from vllm.model_executor.models.utils import (
    AutoWeightsLoader,
    WeightsMapper,
    maybe_prefix,
)
from vllm.model_executor.utils import set_weight_attrs
from vllm.platforms import current_platform
from vllm.sequence import IntermediateTensors

from .bert import BertPooler
from .interfaces import SupportsCrossEncoding, SupportsQuant
from .interfaces_base import default_pooling_type
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Class `BertWithRopeEmbedding` (lines 48-87)
```python
class BertWithRopeEmbedding(nn.Module):
    def __init__(self, config: PretrainedConfig):
        super().__init__()
        if config.position_embedding_type not in ["rope", "rotary"]:
            raise ValueError(
                "Only 'rotary'('rope') position_embedding_type" + " is supported"
            )

        self.word_embeddings = VocabParallelEmbedding(
            config.vocab_size, config.hidden_size
        )
        if config.type_vocab_size > 0:
            self.token_type_embeddings = VocabParallelEmbedding(
                config.type_vocab_size, config.hidden_size
            )
        else:
            self.token_type_embeddings = None

        self.LayerNorm = nn.LayerNorm(config.hidden_size, eps=config.layer_norm_eps)

    def forward(
        self,
        input_ids: torch.Tensor,
        token_type_ids: torch.Tensor | None = None,
    ) -> torch.Tensor:
```
**EN:** Class `BertWithRopeEmbedding` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `BertWithRopeEmbedding` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `BertWithRopeEmbedding.__init__` (lines 49-66)
```python
    def __init__(self, config: PretrainedConfig):
        super().__init__()
        if config.position_embedding_type not in ["rope", "rotary"]:
            raise ValueError(
                "Only 'rotary'('rope') position_embedding_type" + " is supported"
            )

        self.word_embeddings = VocabParallelEmbedding(
            config.vocab_size, config.hidden_size
        )
        if config.type_vocab_size > 0:
            self.token_type_embeddings = VocabParallelEmbedding(
                config.type_vocab_size, config.hidden_size
            )
        else:
            self.token_type_embeddings = None

        self.LayerNorm = nn.LayerNorm(config.hidden_size, eps=config.layer_norm_eps)
```
**EN:** Method `BertWithRopeEmbedding.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `BertWithRopeEmbedding.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `BertWithRopeEmbedding.forward` (lines 68-87)
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        token_type_ids: torch.Tensor | None = None,
    ) -> torch.Tensor:
        input_shape = input_ids.size()
        inputs_embeds = self.word_embeddings(input_ids)

        embeddings = inputs_embeds
        if self.token_type_embeddings is not None:
            if token_type_ids is None:
                token_type_ids = torch.zeros(
                    input_shape, dtype=torch.long, device=inputs_embeds.device
                )

            token_type_embeddings = self.token_type_embeddings(token_type_ids)
            embeddings += token_type_embeddings

        embeddings = self.LayerNorm(embeddings)
        return embeddings
```
**EN:** Method `BertWithRopeEmbedding.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `BertWithRopeEmbedding.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `BertWithRopeAttention` (lines 90-160)
```python
class BertWithRopeAttention(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        num_attention_heads: int,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        bias: bool = True,
        rotary_kwargs: dict | None = None,
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
```
**EN:** Class `BertWithRopeAttention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `BertWithRopeAttention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `BertWithRopeAttention.__init__` (lines 91-148)
```python
    def __init__(
        self,
        hidden_size: int,
        num_attention_heads: int,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        bias: bool = True,
        rotary_kwargs: dict | None = None,
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
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.qkv_proj",
        )

        self.rotary_emb = get_rope(**rotary_kwargs)

        self.attn = EncoderOnlyAttention(
            num_heads=self.num_heads,
            head_size=self.head_dim,
            scale=self.scaling,
            num_kv_heads=self.num_kv_heads,
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.attn",
        )

        self.out_proj = RowParallelLinear(
            input_size=hidden_size,
            output_size=hidden_size,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.dense",
        )
```
**EN:** Method `BertWithRopeAttention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `BertWithRopeAttention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `BertWithRopeAttention.forward` (lines 150-160)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        q, k = self.rotary_emb(positions, q, k)
        attn_output = self.attn(q, k, v)
        output, _ = self.out_proj(attn_output)
        return output
```
**EN:** Method `BertWithRopeAttention.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `BertWithRopeAttention.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `BertWithRopeGatedMLP` (lines 163-194)
```python
class BertWithRopeGatedMLP(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        bias: bool = True,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.act_fn = get_act_and_mul_fn(hidden_act)
        self.gate_up_proj = MergedColumnParallelLinear(
            hidden_size,
            [intermediate_size] * 2,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.gate_up_proj",
        )
        self.down_proj = RowParallelLinear(
            input_size=intermediate_size,
            output_size=hidden_size,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.down_proj",
```
**EN:** Class `BertWithRopeGatedMLP` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `BertWithRopeGatedMLP` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `BertWithRopeGatedMLP.__init__` (lines 164-188)
```python
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        bias: bool = True,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.act_fn = get_act_and_mul_fn(hidden_act)
        self.gate_up_proj = MergedColumnParallelLinear(
            hidden_size,
            [intermediate_size] * 2,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.gate_up_proj",
        )
        self.down_proj = RowParallelLinear(
            input_size=intermediate_size,
            output_size=hidden_size,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.down_proj",
        )
```
**EN:** Method `BertWithRopeGatedMLP.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `BertWithRopeGatedMLP.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `BertWithRopeGatedMLP.forward` (lines 190-194)
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        gate_up, _ = self.gate_up_proj(hidden_states)
        hidden_states = self.act_fn(gate_up)
        hidden_states, _ = self.down_proj(hidden_states)
        return hidden_states
```
**EN:** Method `BertWithRopeGatedMLP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `BertWithRopeGatedMLP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `BertWithRopeMLP` (lines 197-228)
```python
class BertWithRopeMLP(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        bias: bool = True,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.act_fn = get_act_fn(hidden_act)
        self.up_proj = ColumnParallelLinear(
            input_size=hidden_size,
            output_size=intermediate_size,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.up_proj",
        )
        self.down_proj = RowParallelLinear(
            input_size=intermediate_size,
            output_size=hidden_size,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.down_proj",
```
**EN:** Class `BertWithRopeMLP` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `BertWithRopeMLP` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `BertWithRopeMLP.__init__` (lines 198-222)
```python
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        bias: bool = True,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.act_fn = get_act_fn(hidden_act)
        self.up_proj = ColumnParallelLinear(
            input_size=hidden_size,
            output_size=intermediate_size,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.up_proj",
        )
        self.down_proj = RowParallelLinear(
            input_size=intermediate_size,
            output_size=hidden_size,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.down_proj",
        )
```
**EN:** Method `BertWithRopeMLP.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `BertWithRopeMLP.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `BertWithRopeMLP.forward` (lines 224-228)
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states, _ = self.up_proj(hidden_states)
        hidden_states = self.act_fn(hidden_states)
        hidden_states, _ = self.down_proj(hidden_states)
        return hidden_states
```
**EN:** Method `BertWithRopeMLP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `BertWithRopeMLP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `NomicMoE` (lines 231-341)
```python
class NomicMoE(nn.Module):
    def __init__(
        self,
        num_experts: int,
        top_k: int,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        params_dtype: torch.dtype | None = None,
        tp_size: int | None = None,
    ):
        super().__init__()

        self.tp_size = tp_size or get_tensor_model_parallel_world_size()
        self.num_total_experts = num_experts
        self.top_k = top_k
        self.hidden_size = hidden_size
        self.total_intermediate_size = intermediate_size
        self.intermediate_size = divide(intermediate_size, self.tp_size)
        self.hidden_act = activation_without_mul(hidden_act)

        if params_dtype is None:
            params_dtype = torch.get_default_dtype()
        self.params_dtype = params_dtype
```
**EN:** Class `NomicMoE` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, weight_loader, forward.
**CN:** 类 `NomicMoE` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, weight_loader, forward。

### Method `NomicMoE.__init__` (lines 232-289)
```python
    def __init__(
        self,
        num_experts: int,
        top_k: int,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        params_dtype: torch.dtype | None = None,
        tp_size: int | None = None,
    ):
        super().__init__()

        self.tp_size = tp_size or get_tensor_model_parallel_world_size()
        self.num_total_experts = num_experts
        self.top_k = top_k
        self.hidden_size = hidden_size
        self.total_intermediate_size = intermediate_size
        self.intermediate_size = divide(intermediate_size, self.tp_size)
        self.hidden_act = activation_without_mul(hidden_act)

        if params_dtype is None:
            params_dtype = torch.get_default_dtype()
        self.params_dtype = params_dtype

        self.router = ReplicatedLinear(
            self.hidden_size, self.num_total_experts, bias=False
        )
        self.w1 = nn.Parameter(
            torch.empty(
                self.num_total_experts,
                self.intermediate_size,
                self.hidden_size,
                device=current_platform.device_type,
                dtype=self.params_dtype,
            )
        )
        self.w2 = nn.Parameter(
            torch.empty(
                self.num_total_experts,
                self.hidden_size,
                self.intermediate_size,
                device=current_platform.device_type,
                dtype=self.params_dtype,
            )
        )
        self.bias = nn.Parameter(torch.zeros(self.hidden_size))
        set_weight_attrs(
            self.w1,
            {
                "weight_loader": self.weight_loader,
            },
        )
        set_weight_attrs(
            self.w2,
            {
                "weight_loader": self.weight_loader,
            },
        )
```
**EN:** Method `NomicMoE.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `NomicMoE.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `NomicMoE.weight_loader` (lines 291-315)
```python
    def weight_loader(
        self,
        param: nn.Parameter,
        loaded_weight: torch.Tensor,
        weight_name: str,
    ):
        # NOTE: Nomic-MoE has fused experts weights with shape
        # (num_experts * intermediate_size, hidden_size)
        tp_rank = get_tensor_model_parallel_rank()
        param_data = param.data
        shard_size = self.intermediate_size
        shard = slice(tp_rank * shard_size, (tp_rank + 1) * shard_size)
        if weight_name.endswith("w1"):
            loaded_weight = loaded_weight.reshape(
                self.num_total_experts,
                self.total_intermediate_size,
                self.hidden_size,
            )[:, shard]
        if weight_name.endswith("w2"):
            loaded_weight = loaded_weight.reshape(
                self.num_total_experts,
                self.total_intermediate_size,
                self.hidden_size,
            )[:, shard].transpose(1, 2)
        param_data.copy_(loaded_weight)
```
**EN:** Method `NomicMoE.weight_loader` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `NomicMoE.weight_loader` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Class `BertWithRopeBlock` (lines 344-402)
```python
class BertWithRopeBlock(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        moe: bool = False,
        bias: bool = True,
        rotary_kwargs: dict | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.attn = BertWithRopeAttention(
            hidden_size=config.hidden_size,
            num_attention_heads=config.num_attention_heads,
            cache_config=cache_config,
            quant_config=quant_config,
            bias=bias,
            rotary_kwargs=rotary_kwargs,
            prefix=f"{prefix}.attention",
        )

        if moe:
            self.mlp = NomicMoE(
                num_experts=config.num_experts,
```
**EN:** Class `BertWithRopeBlock` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `BertWithRopeBlock` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `BertWithRopeBlock.__init__` (lines 345-395)
```python
    def __init__(
        self,
        config: PretrainedConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        moe: bool = False,
        bias: bool = True,
        rotary_kwargs: dict | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.attn = BertWithRopeAttention(
            hidden_size=config.hidden_size,
            num_attention_heads=config.num_attention_heads,
            cache_config=cache_config,
            quant_config=quant_config,
            bias=bias,
            rotary_kwargs=rotary_kwargs,
            prefix=f"{prefix}.attention",
        )

        if moe:
            self.mlp = NomicMoE(
                num_experts=config.num_experts,
                top_k=config.moe_top_k,
                hidden_size=config.hidden_size,
                intermediate_size=config.intermediate_size,
                hidden_act=config.hidden_act,
            )
        else:
            if config.hidden_act in ["silu", "geglu"]:
                self.mlp = BertWithRopeGatedMLP(
                    hidden_size=config.hidden_size,
                    intermediate_size=config.intermediate_size,
                    hidden_act=config.hidden_act,
                    bias=bias,
                    quant_config=quant_config,
                    prefix=f"{prefix}.mlp",
                )
            else:
                self.mlp = BertWithRopeMLP(
                    hidden_size=config.hidden_size,
                    intermediate_size=config.intermediate_size,
                    hidden_act=config.hidden_act,
                    bias=bias,
                    quant_config=quant_config,
                    prefix=f"{prefix}.mlp",
                )

        self.attn_ln = nn.LayerNorm(config.hidden_size, eps=config.layer_norm_eps)
        self.mlp_ln = nn.LayerNorm(config.hidden_size, eps=config.layer_norm_eps)
```
**EN:** Method `BertWithRopeBlock.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `BertWithRopeBlock.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `BertWithRopeBlock.forward` (lines 397-402)
```python
    def forward(self, positions: torch.Tensor, hidden_states: torch.Tensor):
        attn_output = self.attn(positions, hidden_states)
        hidden_states = self.attn_ln(hidden_states + attn_output)
        mlp_out = self.mlp(hidden_states)
        hidden_states = self.mlp_ln(hidden_states + mlp_out)
        return hidden_states
```
**EN:** Method `BertWithRopeBlock.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `BertWithRopeBlock.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `BertWithRopeEncoder` (lines 405-440)
```python
class BertWithRopeEncoder(nn.Module):
    def __init__(
        self,
        vllm_config: VllmConfig,
        bias: bool = True,
        rotary_kwargs: dict | None = None,
        prefix: str = "",
    ):
        super().__init__()
        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config
        every_n = getattr(config, "moe_every_n_layers", 0)
        self.layers = nn.ModuleList(
            [
                BertWithRopeBlock(
                    config=config,
                    cache_config=cache_config,
                    quant_config=quant_config,
                    bias=bias,
                    moe=every_n > 0 and (layer_idx % every_n == 1),
                    rotary_kwargs=rotary_kwargs,
                    prefix=f"{prefix}.layer.{layer_idx}",
                )
                for layer_idx in range(config.num_hidden_layers)
```
**EN:** Class `BertWithRopeEncoder` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `BertWithRopeEncoder` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `BertWithRopeEncoder.__init__` (lines 406-431)
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        bias: bool = True,
        rotary_kwargs: dict | None = None,
        prefix: str = "",
    ):
        super().__init__()
        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config
        every_n = getattr(config, "moe_every_n_layers", 0)
        self.layers = nn.ModuleList(
            [
                BertWithRopeBlock(
                    config=config,
                    cache_config=cache_config,
                    quant_config=quant_config,
                    bias=bias,
                    moe=every_n > 0 and (layer_idx % every_n == 1),
                    rotary_kwargs=rotary_kwargs,
                    prefix=f"{prefix}.layer.{layer_idx}",
                )
                for layer_idx in range(config.num_hidden_layers)
            ]
        )
```
**EN:** Method `BertWithRopeEncoder.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `BertWithRopeEncoder.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `BertWithRopeEncoder.forward` (lines 433-440)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        for layer in self.layers:
            hidden_states = layer(positions, hidden_states)
        return hidden_states
```
**EN:** Method `BertWithRopeEncoder.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `BertWithRopeEncoder.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `BertWithRope` (lines 445-532)
```python
@support_torch_compile
@default_pooling_type(seq_pooling_type="CLS")
class BertWithRope(nn.Module, SupportsQuant):
    hf_to_vllm_mapper = WeightsMapper(orig_to_new_prefix={"model.": ""})

    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        add_pooling_layer: bool = False,
    ):
        super().__init__()

        self.vllm_config = vllm_config
        self.add_pooling_layer = add_pooling_layer
        self.config = vllm_config.model_config.hf_config
        self.embeddings = BertWithRopeEmbedding(self.config)
        self.encoder = BertWithRopeEncoder(
            vllm_config=vllm_config,
            bias=getattr(self.config, "bias", True),
            rotary_kwargs=self.config.rotary_kwargs,
            prefix=f"{prefix}.encoder",
        )
```
**EN:** Class `BertWithRope` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsQuant. Key methods include __init__, embed_input_ids, forward, load_weights.
**CN:** 类 `BertWithRope` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsQuant。 关键方法包括 __init__, embed_input_ids, forward, load_weights。

### Method `BertWithRope.__init__` (lines 448-471)
```python
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        add_pooling_layer: bool = False,
    ):
        super().__init__()

        self.vllm_config = vllm_config
        self.add_pooling_layer = add_pooling_layer
        self.config = vllm_config.model_config.hf_config
        self.embeddings = BertWithRopeEmbedding(self.config)
        self.encoder = BertWithRopeEncoder(
            vllm_config=vllm_config,
            bias=getattr(self.config, "bias", True),
            rotary_kwargs=self.config.rotary_kwargs,
            prefix=f"{prefix}.encoder",
        )

        if add_pooling_layer:
            self.pooler = BertPooler(vllm_config.model_config)
        else:
            self.pooler = None
```
**EN:** Method `BertWithRope.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `BertWithRope.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `BertWithRope.embed_input_ids` (lines 473-474)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.embeddings(input_ids)
```
**EN:** Method `BertWithRope.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `BertWithRope.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Class `NomicBertModel` (lines 535-553)
```python
class NomicBertModel(BertWithRope):
    # for https://huggingface.co/nomic-ai/nomic-bert-2048

    hf_to_vllm_mapper = WeightsMapper(
        orig_to_new_substr={
            "emb_ln": "embeddings.LayerNorm",
            "attn.Wqkv": "attn.qkv_proj",
            "norm1": "attn_ln",
            "mlp.fc1.": "mlp.up_proj.",
            "mlp.fc11": "mlp.up_proj",
            "mlp.fc12": "mlp.gate_proj",
            "mlp.fc2": "mlp.down_proj",
            "norm2": "mlp_ln",
            # MoE mapping
            "experts.mlp.": "",
            "experts.": "",
            "router.layer": "router",
        }
    )
```
**EN:** Class `NomicBertModel` organizes related behavior for this model family or helper component. It inherits from BertWithRope.
**CN:** 类 `NomicBertModel` 用于组织该模型族或辅助组件的相关行为。 它继承自 BertWithRope。

### Class `GteNewModel` (lines 556-596)
```python
class GteNewModel(BertWithRope):
    # for https://huggingface.co/Alibaba-NLP/new-impl

    hf_to_vllm_mapper = WeightsMapper(
        orig_to_new_substr={
            "new.": "",
            "layer": "layers",
            "attention.qkv_proj": "attn.qkv_proj",
            "attention.o_proj": "attn.out_proj",
        }
    )

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "", **kwargs):
        super().__init__(vllm_config=vllm_config, prefix=prefix, **kwargs)

        # GteNewModel only gate_up_proj does not have bias.
        # Hack method learned from vllm/model_executor/models/glm.py
        for layer in self.encoder.layers:
            layer.mlp.gate_up_proj.bias = None
            layer.mlp.gate_up_proj.skip_bias_add = True

    def split_up_gate_proj(self, weights: Iterable[tuple[str, torch.Tensor]]):
        n = "mlp.up_gate_proj"
        for name, weight in weights:
            if n in name:
```
**EN:** Class `GteNewModel` organizes related behavior for this model family or helper component. It inherits from BertWithRope. Key methods include __init__, split_up_gate_proj, ignore_unnecessary_layers, load_weights.
**CN:** 类 `GteNewModel` 用于组织该模型族或辅助组件的相关行为。 它继承自 BertWithRope。 关键方法包括 __init__, split_up_gate_proj, ignore_unnecessary_layers, load_weights。

### Method `GteNewModel.__init__` (lines 568-575)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "", **kwargs):
        super().__init__(vllm_config=vllm_config, prefix=prefix, **kwargs)

        # GteNewModel only gate_up_proj does not have bias.
        # Hack method learned from vllm/model_executor/models/glm.py
        for layer in self.encoder.layers:
            layer.mlp.gate_up_proj.bias = None
            layer.mlp.gate_up_proj.skip_bias_add = True
```
**EN:** Method `GteNewModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GteNewModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GteNewModel.split_up_gate_proj` (lines 577-585)
```python
    def split_up_gate_proj(self, weights: Iterable[tuple[str, torch.Tensor]]):
        n = "mlp.up_gate_proj"
        for name, weight in weights:
            if n in name:
                up, gate = weight.chunk(2, dim=0)
                yield name.replace(n, "mlp.up_proj"), up
                yield name.replace(n, "mlp.gate_proj"), gate
            else:
                yield name, weight
```
**EN:** Method `GteNewModel.split_up_gate_proj` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `GteNewModel.split_up_gate_proj` 封装了该模块中的一段可复用核心逻辑。

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
- **Standard library / 标准库**: `from collections.abc import Iterable`
- **Third-party / 第三方**: `import torch`, `from torch import nn`, `from transformers import PretrainedConfig`
- **vLLM internal / vLLM 内部依赖**: `from vllm.compilation.decorators import support_torch_compile`, `from vllm.config import CacheConfig, VllmConfig`, `from vllm.distributed import (`, `from vllm.model_executor.layers.activation import get_act_and_mul_fn, get_act_fn`, `from vllm.model_executor.layers.attention import (`, `from vllm.model_executor.layers.fused_moe import activation_without_mul, fused_topk`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.pooler import DispatchPooler`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.layers.rotary_embedding import get_rope`, `from vllm.model_executor.layers.vocab_parallel_embedding import VocabParallelEmbedding`, `from vllm.model_executor.model_loader.weight_utils import default_weight_loader`
