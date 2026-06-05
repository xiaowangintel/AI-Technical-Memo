# glm4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/glm4.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for glm4, including architecture wrappers and weight loading logic. / 面向推理的 glm4 vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 26-57)
```python
from collections.abc import Iterable

import torch
from torch import nn
from transformers import Glm4Config

from vllm.compilation.decorators import support_torch_compile
from vllm.config import CacheConfig, VllmConfig
from vllm.distributed import get_pp_group, get_tensor_model_parallel_world_size
from vllm.model_executor.layers.attention import Attention
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.model_executor.layers.linear import QKVParallelLinear, RowParallelLinear
from vllm.model_executor.layers.logits_processor import LogitsProcessor
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.rotary_embedding import get_rope
from vllm.model_executor.layers.vocab_parallel_embedding import ParallelLMHead
from vllm.model_executor.model_loader.weight_utils import (
    default_weight_loader,
    maybe_remap_kv_scale_name,
)
from vllm.sequence import IntermediateTensors
from vllm.v1.attention.backend import AttentionType

from .interfaces import SupportsLoRA, SupportsPP
from .llama import LlamaMLP as Glm4MLP
from .llama import LlamaModel
from .utils import (
    AutoWeightsLoader,
    PPMissingLayer,
    is_pp_missing_parameter,
    maybe_prefix,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Constants / assignments (lines 221-223)
```python
ALL_DECODER_LAYER_TYPES = {
    "attention": Glm4DecoderLayer,
}
```
**EN:** This block defines ALL_DECODER_LAYER_TYPES, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 ALL_DECODER_LAYER_TYPES，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Function `get_spec_layer_idx_from_weight_name` (lines 382-392)
```python
def get_spec_layer_idx_from_weight_name(
    config: Glm4Config, weight_name: str
) -> int | None:
    if hasattr(config, "num_nextn_predict_layers") and (
        config.num_nextn_predict_layers > 0
    ):
        layer_idx = config.num_hidden_layers
        for i in range(config.num_nextn_predict_layers):
            if f"layers.{layer_idx + i}." in weight_name:
                return layer_idx + i
    return None
```
**EN:** Function `get_spec_layer_idx_from_weight_name` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Function `get_spec_layer_idx_from_weight_name` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Class `Glm4Attention` (lines 60-147)
```python
class Glm4Attention(nn.Module):
    def __init__(
        self,
        config: Glm4Config,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        max_position: int = 4096 * 32,
        head_dim: int | None = None,
        qkv_bias: bool = False,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
        attn_type: str = AttentionType.DECODER,
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size
        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = num_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = num_kv_heads
        if self.total_num_kv_heads >= tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
```
**EN:** Class `Glm4Attention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Glm4Attention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Glm4Attention.__init__` (lines 61-135)
```python
    def __init__(
        self,
        config: Glm4Config,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        max_position: int = 4096 * 32,
        head_dim: int | None = None,
        qkv_bias: bool = False,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
        attn_type: str = AttentionType.DECODER,
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size
        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = num_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = num_kv_heads
        if self.total_num_kv_heads >= tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert tp_size % self.total_num_kv_heads == 0

        rope_params = getattr(config, "rope_parameters", None)
        if isinstance(rope_params, dict) and "partial_rotary_factor" in rope_params:
            config.rope_parameters.setdefault(
                "partial_rotary_factor", rope_params["partial_rotary_factor"]
            )
        else:
            config.rope_parameters.setdefault("partial_rotary_factor", 0.5)

        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)
        self.head_dim = head_dim or hidden_size // self.total_num_heads
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5
        self.qkv_proj = QKVParallelLinear(
            hidden_size,
            self.head_dim,
            self.total_num_heads,
            self.total_num_kv_heads,
            bias=qkv_bias,
            quant_config=quant_config,
            prefix=f"{prefix}.qkv_proj",
        )
        self.o_proj = RowParallelLinear(
            self.total_num_heads * self.head_dim,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.o_proj",
        )
        self.rotary_emb = get_rope(
# ... truncated for analysis ...
```
**EN:** Method `Glm4Attention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Glm4Attention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Glm4Attention.forward` (lines 137-147)
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
        output, _ = self.o_proj(attn_output)
        return output
```
**EN:** Method `Glm4Attention.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Glm4Attention.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Glm4DecoderLayer` (lines 150-218)
```python
class Glm4DecoderLayer(nn.Module):
    def __init__(
        self,
        vllm_config: VllmConfig,
        prefix: str = "",
        config: Glm4Config | None = None,
    ) -> None:
        super().__init__()

        config = config or vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.hidden_size = config.hidden_size

        self.self_attn = Glm4Attention(
            config=config,
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            max_position=config.max_position_embeddings,
            num_kv_heads=config.num_key_value_heads,
            qkv_bias=getattr(config, "attention_bias", False),
            head_dim=getattr(config, "head_dim", None),
            cache_config=cache_config,
            quant_config=quant_config,
```
**EN:** Class `Glm4DecoderLayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Glm4DecoderLayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Glm4DecoderLayer.__init__` (lines 151-192)
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        prefix: str = "",
        config: Glm4Config | None = None,
    ) -> None:
        super().__init__()

        config = config or vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.hidden_size = config.hidden_size

        self.self_attn = Glm4Attention(
            config=config,
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            max_position=config.max_position_embeddings,
            num_kv_heads=config.num_key_value_heads,
            qkv_bias=getattr(config, "attention_bias", False),
            head_dim=getattr(config, "head_dim", None),
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
            attn_type=AttentionType.DECODER,
        )
        self.mlp = Glm4MLP(
            hidden_size=self.hidden_size,
            intermediate_size=config.intermediate_size,
            hidden_act=config.hidden_act,
            quant_config=quant_config,
            prefix=f"{prefix}.mlp",
        )
        self.input_layernorm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.post_attention_layernorm = RMSNorm(
            config.hidden_size, eps=config.rms_norm_eps
        )
        self.post_self_attn_layernorm = RMSNorm(
            config.hidden_size, eps=config.rms_norm_eps
        )
        self.post_mlp_layernorm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
```
**EN:** Method `Glm4DecoderLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Glm4DecoderLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Glm4DecoderLayer.forward` (lines 194-218)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        residual: torch.Tensor | None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        # Self Attention
        if residual is None:
            residual = hidden_states
            hidden_states = self.input_layernorm(hidden_states)
        else:
            hidden_states, residual = self.input_layernorm(hidden_states, residual)
        hidden_states = self.self_attn(
            positions=positions,
            hidden_states=hidden_states,
        )

        hidden_states = self.post_self_attn_layernorm(hidden_states)

        # Fully Connected
        hidden_states, residual = self.post_attention_layernorm(hidden_states, residual)
        hidden_states = self.mlp(hidden_states)
        hidden_states = self.post_mlp_layernorm(hidden_states)

        return hidden_states, residual
```
**EN:** Method `Glm4DecoderLayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Glm4DecoderLayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Glm4Model` (lines 234-305)
```python
@support_torch_compile(
    dynamic_arg_dims={
        "input_ids": 0,
        "positions": -1,
        "intermediate_tensors": 0,
        "inputs_embeds": 0,
    }
)
class Glm4Model(LlamaModel):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__(
            vllm_config=vllm_config, prefix=prefix, layer_type=Glm4DecoderLayer
        )

    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            (".qkv_proj", ".q_proj", "q"),
            (".qkv_proj", ".k_proj", "k"),
            (".qkv_proj", ".v_proj", "v"),
            (".gate_up_proj", ".gate_proj", 0),
            (".gate_up_proj", ".up_proj", 1),
        ]
        params_dict = dict(self.named_parameters())
        loaded_params: set[str] = set()
```
**EN:** Class `Glm4Model` organizes related behavior for this model family or helper component. It inherits from LlamaModel. Key methods include __init__, load_weights.
**CN:** 类 `Glm4Model` 用于组织该模型族或辅助组件的相关行为。 它继承自 LlamaModel。 关键方法包括 __init__, load_weights。

### Method `Glm4Model.__init__` (lines 235-238)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__(
            vllm_config=vllm_config, prefix=prefix, layer_type=Glm4DecoderLayer
        )
```
**EN:** Method `Glm4Model.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Glm4Model.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Glm4Model.load_weights` (lines 240-305)
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            (".qkv_proj", ".q_proj", "q"),
            (".qkv_proj", ".k_proj", "k"),
            (".qkv_proj", ".v_proj", "v"),
            (".gate_up_proj", ".gate_proj", 0),
            (".gate_up_proj", ".up_proj", 1),
        ]
        params_dict = dict(self.named_parameters())
        loaded_params: set[str] = set()
        for name, loaded_weight in weights:
            spec_layer = get_spec_layer_idx_from_weight_name(self.config, name)
            if spec_layer is not None:
                continue
            if "rotary_emb.inv_freq" in name:
                continue
            if "rotary_emb.cos_cached" in name or "rotary_emb.sin_cached" in name:
                # Models trained using ColossalAI may include these tensors in
                # the checkpoint. Skip them.
                continue
            if self.quant_config is not None and (
                scale_name := self.quant_config.get_cache_scale(name)
            ):
                # Loading kv cache quantization scales
                param = params_dict[scale_name]
                weight_loader = getattr(param, "weight_loader", default_weight_loader)
                loaded_weight = (
                    loaded_weight if loaded_weight.dim() == 0 else loaded_weight[0]
                )
                weight_loader(param, loaded_weight)
                loaded_params.add(scale_name)
                continue
            if "scale" in name or "zero_point" in name:
                # Remapping the name of FP8 kv-scale or zero point.
                name = maybe_remap_kv_scale_name(name, params_dict)
                if name is None:
                    continue
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
                # Skip loading extra bias for GPTQ models.
                if name.endswith(".bias") and name not in params_dict:
                    continue

                if is_pp_missing_parameter(name, self):
                    continue
# ... truncated for analysis ...
```
**EN:** Method `Glm4Model.load_weights` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `Glm4Model.load_weights` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Class `Glm4ForCausalLM` (lines 308-379)
```python
class Glm4ForCausalLM(nn.Module, SupportsLoRA, SupportsPP):
    packed_modules_mapping = {
        "qkv_proj": [
            "q_proj",
            "k_proj",
            "v_proj",
        ],
        "gate_up_proj": [
            "gate_proj",
            "up_proj",
        ],
    }

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        self.config = config

        self.quant_config = quant_config
        self.model = Glm4Model(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "model")
        )
```
**EN:** Class `Glm4ForCausalLM` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsLoRA, SupportsPP. Key methods include __init__, embed_input_ids, forward, compute_logits, load_weights.
**CN:** 类 `Glm4ForCausalLM` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsLoRA、SupportsPP。 关键方法包括 __init__, embed_input_ids, forward, compute_logits, load_weights。

### Method `Glm4ForCausalLM.__init__` (lines 321-350)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        self.config = config

        self.quant_config = quant_config
        self.model = Glm4Model(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "model")
        )

        if get_pp_group().is_last_rank:
            if config.tie_word_embeddings:
                self.lm_head = self.model.embed_tokens
            else:
                self.lm_head = ParallelLMHead(
                    config.vocab_size,
                    config.hidden_size,
                    quant_config=quant_config,
                    prefix=maybe_prefix(prefix, "lm_head"),
                )
        else:
            self.lm_head = PPMissingLayer()

        self.logits_processor = LogitsProcessor(config.vocab_size)

        self.make_empty_intermediate_tensors = (
            self.model.make_empty_intermediate_tensors
        )
```
**EN:** Method `Glm4ForCausalLM.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Glm4ForCausalLM.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Glm4ForCausalLM.embed_input_ids` (lines 352-353)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.model.embed_input_ids(input_ids)
```
**EN:** Method `Glm4ForCausalLM.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `Glm4ForCausalLM.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `Glm4ForCausalLM.forward` (lines 355-365)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor | IntermediateTensors:
        hidden_states = self.model(
            input_ids, positions, intermediate_tensors, inputs_embeds
        )
        return hidden_states
```
**EN:** Method `Glm4ForCausalLM.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Glm4ForCausalLM.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Method `Glm4ForCausalLM.compute_logits` (lines 367-372)
```python
    def compute_logits(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor | None:
        logits = self.logits_processor(self.lm_head, hidden_states)
        return logits
```
**EN:** Method `Glm4ForCausalLM.compute_logits` converts hidden states into prediction-ready outputs, often by applying a head, post-processing, or sampling logic.
**CN:** Method `Glm4ForCausalLM.compute_logits` 将隐藏状态转成可预测输出，通常会经过输出头、后处理或采样逻辑。

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
- **Third-party / 第三方**: `import torch`, `from torch import nn`, `from transformers import Glm4Config`
- **vLLM internal / vLLM 内部依赖**: `from vllm.compilation.decorators import support_torch_compile`, `from vllm.config import CacheConfig, VllmConfig`, `from vllm.distributed import get_pp_group, get_tensor_model_parallel_world_size`, `from vllm.model_executor.layers.attention import Attention`, `from vllm.model_executor.layers.layernorm import RMSNorm`, `from vllm.model_executor.layers.linear import QKVParallelLinear, RowParallelLinear`, `from vllm.model_executor.layers.logits_processor import LogitsProcessor`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.layers.rotary_embedding import get_rope`, `from vllm.model_executor.layers.vocab_parallel_embedding import ParallelLMHead`, `from vllm.model_executor.model_loader.weight_utils import (`, `from vllm.sequence import IntermediateTensors`
- **Module note / 模块说明**: **EN:** Inference-only GLM-4-0414 model compatible with HuggingFace weights. **CN:** 模块文档字符串给出的原始说明是：Inference-only GLM-4-0414 model compatible with HuggingFace weights.。
