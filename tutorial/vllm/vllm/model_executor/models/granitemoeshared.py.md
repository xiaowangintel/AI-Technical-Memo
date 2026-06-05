# granitemoeshared.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/granitemoeshared.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Mixture-of-experts model implementation for granitemoeshared, covering expert routing, transformer blocks, and weight loading. / 面向 granitemoeshared 的 MoE 模型实现，涵盖专家路由、Transformer 模块与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 9-35)
```python
from collections.abc import Iterable
from itertools import islice

import torch
from torch import nn
from transformers.models.granitemoeshared import GraniteMoeSharedConfig

from vllm.compilation.decorators import support_torch_compile
from vllm.config import CacheConfig, VllmConfig
from vllm.distributed import get_pp_group
from vllm.model_executor.layers.activation import SiluAndMul
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.model_executor.layers.linear import (
    MergedColumnParallelLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.logits_processor import LogitsProcessor
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.vocab_parallel_embedding import (
    ParallelLMHead,
    VocabParallelEmbedding,
)
from vllm.sequence import IntermediateTensors

from .granitemoe import GraniteMoeAttention, GraniteMoeModel, GraniteMoeMoE
from .interfaces import SupportsLoRA, SupportsPP
from .utils import AutoWeightsLoader, make_layers, maybe_prefix
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Class `GraniteMoeSharedMLP` (lines 38-74)
```python
class GraniteMoeSharedMLP(nn.Module):
    def __init__(
        self,
        config: GraniteMoeSharedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()

        self.input_size = config.hidden_size
        self.hidden_size = config.shared_intermediate_size
        self.input_linear = MergedColumnParallelLinear(
            input_size=self.input_size,
            output_sizes=[self.hidden_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.input_linear",
        )
        self.output_linear = RowParallelLinear(
            self.hidden_size,
            self.input_size,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.output_linear",
        )
```
**EN:** Class `GraniteMoeSharedMLP` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `GraniteMoeSharedMLP` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `GraniteMoeSharedMLP.__init__` (lines 39-68)
```python
    def __init__(
        self,
        config: GraniteMoeSharedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()

        self.input_size = config.hidden_size
        self.hidden_size = config.shared_intermediate_size
        self.input_linear = MergedColumnParallelLinear(
            input_size=self.input_size,
            output_sizes=[self.hidden_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.input_linear",
        )
        self.output_linear = RowParallelLinear(
            self.hidden_size,
            self.input_size,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.output_linear",
        )
        if config.hidden_act != "silu":
            raise ValueError(
                f"Unsupported activation: {config.hidden_act}. "
                "Only silu is supported for now."
            )
        self.act_fn = SiluAndMul()
```
**EN:** Method `GraniteMoeSharedMLP.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GraniteMoeSharedMLP.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GraniteMoeSharedMLP.forward` (lines 70-74)
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states, _ = self.input_linear(hidden_states)
        hidden_states = self.act_fn(hidden_states)
        hidden_states, _ = self.output_linear(hidden_states)
        return hidden_states
```
**EN:** Method `GraniteMoeSharedMLP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `GraniteMoeSharedMLP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `GraniteMoeSharedDecoderLayer` (lines 77-146)
```python
class GraniteMoeSharedDecoderLayer(nn.Module):
    def __init__(
        self,
        config: GraniteMoeSharedConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        self.self_attn = GraniteMoeAttention(
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            max_position=config.max_position_embeddings,
            num_kv_heads=config.num_key_value_heads,
            rope_parameters=config.rope_parameters,
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
            attention_multiplier=config.attention_multiplier,
        )
        self.block_sparse_moe = GraniteMoeMoE(
            num_experts=config.num_local_experts,
            top_k=config.num_experts_per_tok,
            hidden_size=config.hidden_size,
```
**EN:** Class `GraniteMoeSharedDecoderLayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `GraniteMoeSharedDecoderLayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `GraniteMoeSharedDecoderLayer.__init__` (lines 78-119)
```python
    def __init__(
        self,
        config: GraniteMoeSharedConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        self.self_attn = GraniteMoeAttention(
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            max_position=config.max_position_embeddings,
            num_kv_heads=config.num_key_value_heads,
            rope_parameters=config.rope_parameters,
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
            attention_multiplier=config.attention_multiplier,
        )
        self.block_sparse_moe = GraniteMoeMoE(
            num_experts=config.num_local_experts,
            top_k=config.num_experts_per_tok,
            hidden_size=config.hidden_size,
            intermediate_size=config.intermediate_size,
            quant_config=quant_config,
            prefix=f"{prefix}.block_sparse_moe",
        )
        self.shared_mlp = (
            None
            if getattr(config, "shared_intermediate_size", 0) == 0
            else GraniteMoeSharedMLP(
                config, quant_config=quant_config, prefix=f"{prefix}.shared_mlp"
            )
        )

        self.input_layernorm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.post_attention_layernorm = RMSNorm(
            config.hidden_size, eps=config.rms_norm_eps
        )

        self.residual_multiplier = config.residual_multiplier
```
**EN:** Method `GraniteMoeSharedDecoderLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GraniteMoeSharedDecoderLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GraniteMoeSharedDecoderLayer.forward` (lines 121-146)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        # Self Attention
        residual = hidden_states
        hidden_states = self.input_layernorm(hidden_states)
        hidden_states = self.self_attn(
            positions=positions,
            hidden_states=hidden_states,
        )
        hidden_states = residual + hidden_states * self.residual_multiplier
        residual = hidden_states
        hidden_states = self.post_attention_layernorm(hidden_states)
        if self.shared_mlp is None:
            hidden_states = self.block_sparse_moe(hidden_states)
        else:
            # create a copy since block_sparse_moe modifies in-place
            moe_hidden_states = hidden_states.clone()
            moe_hidden_states = self.block_sparse_moe(moe_hidden_states)
            hidden_states = moe_hidden_states + self.shared_mlp(hidden_states)
            del moe_hidden_states
        hidden_states = residual + hidden_states * self.residual_multiplier

        return hidden_states
```
**EN:** Method `GraniteMoeSharedDecoderLayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `GraniteMoeSharedDecoderLayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `GraniteMoeSharedModel` (lines 150-246)
```python
@support_torch_compile
class GraniteMoeSharedModel(nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.config = config
        self.quant_config = quant_config  # Required by MixtralModel

        self.vocab_size = config.vocab_size

        self.embed_tokens = VocabParallelEmbedding(
            self.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
        )
        self.embedding_multiplier = config.embedding_multiplier

        self.start_layer, self.end_layer, self.layers = make_layers(
            config.num_hidden_layers,
            lambda prefix: GraniteMoeSharedDecoderLayer(
                config, cache_config, quant_config=quant_config, prefix=prefix
```
**EN:** Class `GraniteMoeSharedModel` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, embed_input_ids, forward, load_weights.
**CN:** 类 `GraniteMoeSharedModel` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, embed_input_ids, forward, load_weights。

### Method `GraniteMoeSharedModel.__init__` (lines 151-178)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.config = config
        self.quant_config = quant_config  # Required by MixtralModel

        self.vocab_size = config.vocab_size

        self.embed_tokens = VocabParallelEmbedding(
            self.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
        )
        self.embedding_multiplier = config.embedding_multiplier

        self.start_layer, self.end_layer, self.layers = make_layers(
            config.num_hidden_layers,
            lambda prefix: GraniteMoeSharedDecoderLayer(
                config, cache_config, quant_config=quant_config, prefix=prefix
            ),
            prefix=f"{prefix}.layers",
        )

        self.norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
```
**EN:** Method `GraniteMoeSharedModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GraniteMoeSharedModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GraniteMoeSharedModel.embed_input_ids` (lines 180-181)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.embed_tokens(input_ids)
```
**EN:** Method `GraniteMoeSharedModel.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `GraniteMoeSharedModel.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `GraniteMoeSharedModel.forward` (lines 183-208)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor:
        if get_pp_group().is_first_rank:
            if inputs_embeds is not None:
                hidden_states = inputs_embeds
            else:
                hidden_states = self.embed_input_ids(input_ids)
            hidden_states *= self.embedding_multiplier
        else:
            assert intermediate_tensors is not None
            hidden_states = intermediate_tensors["hidden_states"]
        for layer in islice(self.layers, self.start_layer, self.end_layer):
            hidden_states = layer(positions, hidden_states)
        if not get_pp_group().is_last_rank:
            return IntermediateTensors(
                {
                    "hidden_states": hidden_states,
                }
            )
        hidden_states = self.norm(hidden_states)
        return hidden_states
```
**EN:** Method `GraniteMoeSharedModel.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `GraniteMoeSharedModel.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Method `GraniteMoeSharedModel.load_weights` (lines 210-246)
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        new_weights = {}
        for n, p in weights:
            if n.endswith(".block_sparse_moe.input_linear.weight"):
                for e in range(p.size(0)):
                    w1_name = n.replace(
                        ".block_sparse_moe.input_linear.weight",
                        f".block_sparse_moe.experts.{e}.w1.weight",
                    )
                    w3_name = n.replace(
                        ".block_sparse_moe.input_linear.weight",
                        f".block_sparse_moe.experts.{e}.w3.weight",
                    )
                    w1_param, w3_param = p[e].chunk(2, dim=0)
                    assert w1_name not in new_weights
                    assert w3_name not in new_weights
                    new_weights[w1_name] = w1_param
                    new_weights[w3_name] = w3_param
            elif n.endswith(".block_sparse_moe.output_linear.weight"):
                for e in range(p.size(0)):
                    w2_name = n.replace(
                        ".block_sparse_moe.output_linear.weight",
                        f".block_sparse_moe.experts.{e}.w2.weight",
                    )
                    w2_param = p[e]
                    assert w2_name not in new_weights
                    new_weights[w2_name] = w2_param
            elif n.endswith(".block_sparse_moe.router.layer.weight"):
                gate_name = n.replace(
                    ".block_sparse_moe.router.layer.weight",
                    ".block_sparse_moe.gate.weight",
                )
                assert gate_name not in new_weights
                new_weights[gate_name] = p
            else:
                new_weights[n] = p
        return GraniteMoeModel._load_weights(self, new_weights.items())
```
**EN:** Method `GraniteMoeSharedModel.load_weights` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `GraniteMoeSharedModel.load_weights` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Class `GraniteMoeSharedForCausalLM` (lines 249-327)
```python
class GraniteMoeSharedForCausalLM(nn.Module, SupportsLoRA, SupportsPP):
    fall_back_to_pt_during_load = False

    packed_modules_mapping = {
        "qkv_proj": [
            "q_proj",
            "k_proj",
            "v_proj",
        ],
    }

    # LoRA specific attributes
    embedding_modules = {
        "embed_tokens": "input_embeddings",
        "lm_head": "output_embeddings",
    }

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        self.config = config

        self.model = GraniteMoeSharedModel(
```
**EN:** Class `GraniteMoeSharedForCausalLM` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsLoRA, SupportsPP. Key methods include __init__, embed_input_ids, forward, compute_logits, make_empty_intermediate_tensors, load_weights.
**CN:** 类 `GraniteMoeSharedForCausalLM` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsLoRA、SupportsPP。 关键方法包括 __init__, embed_input_ids, forward, compute_logits, make_empty_intermediate_tensors, load_weights。

### Method `GraniteMoeSharedForCausalLM.__init__` (lines 266-290)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        self.config = config

        self.model = GraniteMoeSharedModel(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "model")
        )

        self.lm_head = ParallelLMHead(
            config.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=maybe_prefix(prefix, "lm_head"),
        )
        if config.tie_word_embeddings:
            self.lm_head.weight = self.model.embed_tokens.weight

        self.logits_processor = LogitsProcessor(
            config.vocab_size,
            config.vocab_size,
            scale=1 / self.config.logits_scaling,
        )
```
**EN:** Method `GraniteMoeSharedForCausalLM.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GraniteMoeSharedForCausalLM.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GraniteMoeSharedForCausalLM.embed_input_ids` (lines 292-293)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.model.embed_input_ids(input_ids)
```
**EN:** Method `GraniteMoeSharedForCausalLM.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `GraniteMoeSharedForCausalLM.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `GraniteMoeSharedForCausalLM.forward` (lines 295-305)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor:
        hidden_states = self.model(
            input_ids, positions, intermediate_tensors, inputs_embeds
        )
        return hidden_states
```
**EN:** Method `GraniteMoeSharedForCausalLM.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `GraniteMoeSharedForCausalLM.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Method `GraniteMoeSharedForCausalLM.compute_logits` (lines 307-309)
```python
    def compute_logits(self, hidden_states: torch.Tensor) -> torch.Tensor | None:
        logits = self.logits_processor(self.lm_head, hidden_states)
        return logits
```
**EN:** Method `GraniteMoeSharedForCausalLM.compute_logits` converts hidden states into prediction-ready outputs, often by applying a head, post-processing, or sampling logic.
**CN:** Method `GraniteMoeSharedForCausalLM.compute_logits` 将隐藏状态转成可预测输出，通常会经过输出头、后处理或采样逻辑。

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
- **Standard library / 标准库**: `from collections.abc import Iterable`, `from itertools import islice`
- **Third-party / 第三方**: `import torch`, `from torch import nn`, `from transformers.models.granitemoeshared import GraniteMoeSharedConfig`
- **vLLM internal / vLLM 内部依赖**: `from vllm.compilation.decorators import support_torch_compile`, `from vllm.config import CacheConfig, VllmConfig`, `from vllm.distributed import get_pp_group`, `from vllm.model_executor.layers.activation import SiluAndMul`, `from vllm.model_executor.layers.layernorm import RMSNorm`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.logits_processor import LogitsProcessor`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.layers.vocab_parallel_embedding import (`, `from vllm.sequence import IntermediateTensors`, `from .granitemoe import GraniteMoeAttention, GraniteMoeModel, GraniteMoeMoE`, `from .interfaces import SupportsLoRA, SupportsPP`
- **Module note / 模块说明**: **EN:** Inference-only GraniteMoeShared model.  The architecture is the same as granitemoe but with the addition of shared experts. **CN:** 模块文档字符串给出的原始说明是：Inference-only GraniteMoeShared model.  The architecture is the same as granitemoe but with the addition of shared experts.。
