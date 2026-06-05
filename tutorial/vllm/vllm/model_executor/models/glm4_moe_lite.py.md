# glm4_moe_lite.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/glm4_moe_lite.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Mixture-of-experts model implementation for glm4_moe_lite, covering expert routing, transformer blocks, and weight loading. / 面向 glm4_moe_lite 的 MoE 模型实现，涵盖专家路由、Transformer 模块与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 26-77)
```python
import typing
from collections.abc import Callable, Iterable
from itertools import islice
from typing import TYPE_CHECKING

import torch
from torch import nn

if TYPE_CHECKING:
    from transformers.models.glm4_moe_lite import Glm4MoeLiteConfig

from vllm._aiter_ops import rocm_aiter_ops
from vllm.compilation.decorators import support_torch_compile
from vllm.config import VllmConfig
from vllm.distributed import (
    get_pp_group,
)
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe import (
    fused_moe_make_expert_params_mapping,
)
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.model_executor.layers.logits_processor import LogitsProcessor
from vllm.model_executor.layers.vocab_parallel_embedding import (
    ParallelLMHead,
    VocabParallelEmbedding,
)
from vllm.model_executor.model_loader.weight_utils import (
    default_weight_loader,
    maybe_remap_kv_scale_name,
)
from vllm.model_executor.models.deepseek_v2 import (
    DeepseekV2Attention,
    DeepseekV2MLAAttention,
)
from vllm.model_executor.models.glm4_moe import (
    Glm4MixtureOfExperts,
    Glm4MoE,
    Glm4MoeMLP,
)
from vllm.platforms import current_platform
from vllm.sequence import IntermediateTensors

from .interfaces import SupportsLoRA, SupportsPP
from .utils import (
    AutoWeightsLoader,
    PPMissingLayer,
    is_pp_missing_parameter,
    make_empty_intermediate_tensors_factory,
    make_layers,
    maybe_prefix,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Function `get_spec_layer_idx_from_weight_name` (lines 635-645)
```python
def get_spec_layer_idx_from_weight_name(
    config: "Glm4MoeLiteConfig", weight_name: str
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

### Class `Glm4MoeLiteMLP` (lines 82-83)
```python
class Glm4MoeLiteMLP(Glm4MoeMLP):
    pass
```
**EN:** Class `Glm4MoeLiteMLP` organizes related behavior for this model family or helper component. It inherits from Glm4MoeMLP.
**CN:** 类 `Glm4MoeLiteMLP` 用于组织该模型族或辅助组件的相关行为。 它继承自 Glm4MoeMLP。

### Class `Glm4MoeLite` (lines 86-87)
```python
class Glm4MoeLite(Glm4MoE):
    pass
```
**EN:** Class `Glm4MoeLite` organizes related behavior for this model family or helper component. It inherits from Glm4MoE.
**CN:** 类 `Glm4MoeLite` 用于组织该模型族或辅助组件的相关行为。 它继承自 Glm4MoE。

### Class `Glm4LiteMixtureOfExperts` (lines 90-91)
```python
class Glm4LiteMixtureOfExperts(Glm4MixtureOfExperts):
    pass
```
**EN:** Class `Glm4LiteMixtureOfExperts` organizes related behavior for this model family or helper component. It inherits from Glm4MixtureOfExperts.
**CN:** 类 `Glm4LiteMixtureOfExperts` 用于组织该模型族或辅助组件的相关行为。 它继承自 Glm4MixtureOfExperts。

### Class `Glm4MoeLiteAttention` (lines 94-95)
```python
class Glm4MoeLiteAttention(DeepseekV2Attention):
    pass
```
**EN:** Class `Glm4MoeLiteAttention` organizes related behavior for this model family or helper component. It inherits from DeepseekV2Attention.
**CN:** 类 `Glm4MoeLiteAttention` 用于组织该模型族或辅助组件的相关行为。 它继承自 DeepseekV2Attention。

### Class `Glm4MoeLiteMLAAttention` (lines 98-99)
```python
class Glm4MoeLiteMLAAttention(DeepseekV2MLAAttention):
    pass
```
**EN:** Class `Glm4MoeLiteMLAAttention` organizes related behavior for this model family or helper component. It inherits from DeepseekV2MLAAttention.
**CN:** 类 `Glm4MoeLiteMLAAttention` 用于组织该模型族或辅助组件的相关行为。 它继承自 DeepseekV2MLAAttention。

### Class `Glm4MoeLiteDecoderLayer` (lines 102-202)
```python
class Glm4MoeLiteDecoderLayer(nn.Module):
    def __init__(
        self,
        vllm_config: VllmConfig,
        prefix: str,
        config: "Glm4MoeLiteConfig | None" = None,
        topk_indices_buffer: torch.Tensor | None = None,
    ) -> None:
        super().__init__()

        if config is None:
            config = vllm_config.model_config.hf_config
        model_config = vllm_config.model_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.hidden_size = config.hidden_size
        max_position_embeddings = getattr(config, "max_position_embeddings", 8192)
        moe_layer_freq = getattr(config, "moe_layer_freq", 1)
        # DecoderLayers are created with `make_layers` which passes the prefix
        # with the layer's index.
        layer_idx = int(prefix.split(sep=".")[-1])
        self.layer_idx = layer_idx

        # verify MLA attention specific fields
```
**EN:** Class `Glm4MoeLiteDecoderLayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Glm4MoeLiteDecoderLayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Glm4MoeLiteDecoderLayer.__init__` (lines 103-176)
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        prefix: str,
        config: "Glm4MoeLiteConfig | None" = None,
        topk_indices_buffer: torch.Tensor | None = None,
    ) -> None:
        super().__init__()

        if config is None:
            config = vllm_config.model_config.hf_config
        model_config = vllm_config.model_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.hidden_size = config.hidden_size
        max_position_embeddings = getattr(config, "max_position_embeddings", 8192)
        moe_layer_freq = getattr(config, "moe_layer_freq", 1)
        # DecoderLayers are created with `make_layers` which passes the prefix
        # with the layer's index.
        layer_idx = int(prefix.split(sep=".")[-1])
        self.layer_idx = layer_idx

        # verify MLA attention specific fields
        qk_nope_head_dim = getattr(config, "qk_nope_head_dim", 0)
        qk_rope_head_dim = getattr(config, "qk_rope_head_dim", 0)
        v_head_dim = getattr(config, "v_head_dim", 0)
        kv_lora_rank = getattr(config, "kv_lora_rank", 0)

        if model_config.use_mla:
            attn_cls = Glm4MoeLiteMLAAttention
        else:
            attn_cls = Glm4MoeLiteAttention

        self.self_attn = attn_cls(
            vllm_config=vllm_config,
            config=config,
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            qk_nope_head_dim=qk_nope_head_dim,
            qk_rope_head_dim=qk_rope_head_dim,
            v_head_dim=v_head_dim,
            q_lora_rank=config.q_lora_rank if hasattr(config, "q_lora_rank") else None,
            kv_lora_rank=kv_lora_rank,
            max_position_embeddings=max_position_embeddings,
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
            topk_indices_buffer=topk_indices_buffer,
        )

        if (
            config.n_routed_experts is not None
            and layer_idx >= config.first_k_dense_replace
            and layer_idx % moe_layer_freq == 0
        ):
            self.mlp = Glm4MoeLite(
                config=config,
                quant_config=quant_config,
                prefix=f"{prefix}.mlp",
# ... truncated for analysis ...
```
**EN:** Method `Glm4MoeLiteDecoderLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Glm4MoeLiteDecoderLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Glm4MoeLiteDecoderLayer.forward` (lines 178-202)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        residual: torch.Tensor | None,
        llama_4_scaling: torch.Tensor | None = None,
    ) -> torch.Tensor:
        # Self Attention
        if residual is None:
            residual = hidden_states.clone()
            hidden_states = self.input_layernorm(hidden_states)
        else:
            hidden_states, residual = self.input_layernorm(hidden_states, residual)

        attn_kwargs = {
            "positions": positions,
            "hidden_states": hidden_states,
        }
        attn_kwargs["llama_4_scaling"] = llama_4_scaling
        hidden_states = self.self_attn(**attn_kwargs)

        hidden_states, residual = self.post_attention_layernorm(hidden_states, residual)
        hidden_states = self.mlp(hidden_states)

        return hidden_states, residual
```
**EN:** Method `Glm4MoeLiteDecoderLayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Glm4MoeLiteDecoderLayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Glm4MoeLiteModel` (lines 213-514)
```python
@support_torch_compile(
    dynamic_arg_dims={
        "input_ids": 0,
        "positions": -1,
        "intermediate_tensors": 0,
        "inputs_embeds": 0,
    }
)
class Glm4MoeLiteModel(nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        self.config = config
        self.device = current_platform.device_type

        self.vocab_size = config.vocab_size
        self.is_v32 = hasattr(config, "index_topk")
        if self.is_v32:
            topk_tokens = config.index_topk
            topk_indices_buffer = torch.empty(
                vllm_config.scheduler_config.max_num_batched_tokens,
                topk_tokens,
                dtype=torch.int32,
```
**EN:** Class `Glm4MoeLiteModel` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, embed_input_ids, forward, make_empty_intermediate_tensors, get_expert_mapping, load_weights.
**CN:** 类 `Glm4MoeLiteModel` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, embed_input_ids, forward, make_empty_intermediate_tensors, get_expert_mapping, load_weights。

### Method `Glm4MoeLiteModel.__init__` (lines 214-262)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        self.config = config
        self.device = current_platform.device_type

        self.vocab_size = config.vocab_size
        self.is_v32 = hasattr(config, "index_topk")
        if self.is_v32:
            topk_tokens = config.index_topk
            topk_indices_buffer = torch.empty(
                vllm_config.scheduler_config.max_num_batched_tokens,
                topk_tokens,
                dtype=torch.int32,
                device=self.device,
            )
        else:
            topk_indices_buffer = None

        if get_pp_group().is_first_rank:
            self.embed_tokens = VocabParallelEmbedding(
                config.vocab_size,
                config.hidden_size,
                quant_config=quant_config,
                prefix=f"{prefix}.embed_tokens",
            )
        else:
            self.embed_tokens = PPMissingLayer()

        self.start_layer, self.end_layer, self.layers = make_layers(
            config.num_hidden_layers,
            lambda prefix: Glm4MoeLiteDecoderLayer(
                vllm_config=vllm_config,
                config=config,
                prefix=prefix,
                topk_indices_buffer=topk_indices_buffer,
            ),
            prefix=f"{prefix}.layers",
        )

        if get_pp_group().is_last_rank:
            self.norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        else:
            self.norm = PPMissingLayer()
        self.make_empty_intermediate_tensors = make_empty_intermediate_tensors_factory(
            ["hidden_states", "residual"], config.hidden_size
        )
```
**EN:** Method `Glm4MoeLiteModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Glm4MoeLiteModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Glm4MoeLiteModel.embed_input_ids` (lines 264-265)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.embed_tokens(input_ids)
```
**EN:** Method `Glm4MoeLiteModel.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `Glm4MoeLiteModel.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `Glm4MoeLiteModel.forward` (lines 267-294)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor | IntermediateTensors:
        if get_pp_group().is_first_rank:
            if inputs_embeds is not None:
                hidden_states = inputs_embeds
            else:
                hidden_states = self.embed_input_ids(input_ids)
            residual = None
        else:
            assert intermediate_tensors is not None
            hidden_states = intermediate_tensors["hidden_states"]
            residual = intermediate_tensors["residual"]

        for layer in islice(self.layers, self.start_layer, self.end_layer):
            hidden_states, residual = layer(positions, hidden_states, residual)

        if not get_pp_group().is_last_rank:
            return IntermediateTensors(
                {"hidden_states": hidden_states, "residual": residual}
            )

        hidden_states, _ = self.norm(hidden_states, residual)
        return hidden_states
```
**EN:** Method `Glm4MoeLiteModel.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Glm4MoeLiteModel.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Method `Glm4MoeLiteModel.make_empty_intermediate_tensors` (lines 296-308)
```python
    def make_empty_intermediate_tensors(
        self, batch_size: int, dtype: torch.dtype, device: torch.device
    ) -> IntermediateTensors:
        return IntermediateTensors(
            {
                "hidden_states": torch.zeros(
                    (batch_size, self.config.hidden_size), dtype=dtype, device=device
                ),
                "residual": torch.zeros(
                    (batch_size, self.config.hidden_size), dtype=dtype, device=device
                ),
            }
        )
```
**EN:** Method `Glm4MoeLiteModel.make_empty_intermediate_tensors` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Glm4MoeLiteModel.make_empty_intermediate_tensors` 封装了该模块中的一段可复用核心逻辑。

### Class `Glm4MoeLiteForCausalLM` (lines 517-632)
```python
class Glm4MoeLiteForCausalLM(
    nn.Module, SupportsPP, SupportsLoRA, Glm4LiteMixtureOfExperts
):
    packed_modules_mapping = {
        "gate_up_proj": ["gate_proj", "up_proj"],
    }

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        self.config = config
        self.quant_config = quant_config

        qk_nope_head_dim = getattr(config, "qk_nope_head_dim", 0)
        qk_rope_head_dim = getattr(config, "qk_rope_head_dim", 0)
        self.use_mha = config.model_type == "deepseek" or all(
            dim == 0 for dim in (qk_nope_head_dim, qk_rope_head_dim)
        )

        if self.use_mha:
            self.packed_modules_mapping["qkv_proj"] = ["q_proj", "k_proj", "v_proj"]

        # `packed_modules_mapping` needs to be modified before
        # initializing DeepseekV2Model, as it is passed inplace to
```
**EN:** Class `Glm4MoeLiteForCausalLM` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsPP, SupportsLoRA, Glm4LiteMixtureOfExperts. Key methods include __init__, set_moe_parameters, embed_input_ids, forward, compute_logits, get_expert_mapping.
**CN:** 类 `Glm4MoeLiteForCausalLM` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsPP、SupportsLoRA、Glm4LiteMixtureOfExperts。 关键方法包括 __init__, set_moe_parameters, embed_input_ids, forward, compute_logits, get_expert_mapping。

### Method `Glm4MoeLiteForCausalLM.__init__` (lines 524-573)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        self.config = config
        self.quant_config = quant_config

        qk_nope_head_dim = getattr(config, "qk_nope_head_dim", 0)
        qk_rope_head_dim = getattr(config, "qk_rope_head_dim", 0)
        self.use_mha = config.model_type == "deepseek" or all(
            dim == 0 for dim in (qk_nope_head_dim, qk_rope_head_dim)
        )

        if self.use_mha:
            self.packed_modules_mapping["qkv_proj"] = ["q_proj", "k_proj", "v_proj"]

        # `packed_modules_mapping` needs to be modified before
        # initializing DeepseekV2Model, as it is passed inplace to
        # quantization config init and may be used to select the
        # quant_method for relevant layers during initialization.
        self.fuse_qkv_a_proj = (
            hasattr(config, "q_lora_rank") and config.q_lora_rank is not None
        )
        if self.fuse_qkv_a_proj:
            self.packed_modules_mapping["fused_qkv_a_proj"] = [
                "q_a_proj",
                "kv_a_proj_with_mqa",
            ]

        self.model = Glm4MoeLiteModel(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "model")
        )
        if get_pp_group().is_last_rank:
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
        # Set MoE hyperparameters
        self.num_moe_layers = (
            self.config.num_hidden_layers - self.config.first_k_dense_replace
        )
        self.set_moe_parameters()
```
**EN:** Method `Glm4MoeLiteForCausalLM.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Glm4MoeLiteForCausalLM.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Glm4MoeLiteForCausalLM.set_moe_parameters` (lines 575-594)
```python
    def set_moe_parameters(self):
        self.expert_weights = []

        self.num_expert_groups = getattr(self.config, "n_group", 1)

        self.moe_layers = []
        self.moe_mlp_layers = []
        example_moe = None
        for layer in self.model.layers:
            if isinstance(layer, PPMissingLayer):
                continue

            assert isinstance(layer, Glm4MoeLiteDecoderLayer)
            if isinstance(layer.mlp, Glm4MoeLite):
                # Pick last one layer since the first ones may be dense layers.
                example_moe = layer.mlp
                self.moe_mlp_layers.append(layer.mlp)
                self.moe_layers.append(layer.mlp.experts)

        self.extract_moe_parameters(example_moe)
```
**EN:** Method `Glm4MoeLiteForCausalLM.set_moe_parameters` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Glm4MoeLiteForCausalLM.set_moe_parameters` 封装了该模块中的一段可复用核心逻辑。

### Method `Glm4MoeLiteForCausalLM.embed_input_ids` (lines 596-597)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.model.embed_input_ids(input_ids)
```
**EN:** Method `Glm4MoeLiteForCausalLM.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `Glm4MoeLiteForCausalLM.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `Glm4MoeLiteForCausalLM.forward` (lines 599-609)
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
**EN:** Method `Glm4MoeLiteForCausalLM.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Glm4MoeLiteForCausalLM.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

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
- **Standard library / 标准库**: `import typing`, `from collections.abc import Callable, Iterable`, `from itertools import islice`, `from typing import TYPE_CHECKING`
- **Third-party / 第三方**: `import torch`, `from torch import nn`, `from transformers.models.glm4_moe_lite import Glm4MoeLiteConfig`
- **vLLM internal / vLLM 内部依赖**: `from vllm._aiter_ops import rocm_aiter_ops`, `from vllm.compilation.decorators import support_torch_compile`, `from vllm.config import VllmConfig`, `from vllm.distributed import (`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.fused_moe import (`, `from vllm.model_executor.layers.layernorm import RMSNorm`, `from vllm.model_executor.layers.logits_processor import LogitsProcessor`, `from vllm.model_executor.layers.vocab_parallel_embedding import (`, `from vllm.model_executor.model_loader.weight_utils import (`, `from vllm.model_executor.models.deepseek_v2 import (`, `from vllm.model_executor.models.glm4_moe import (`
- **Module note / 模块说明**: **EN:** Inference-only GLM-4.7-Flash model compatible with HuggingFace weights. **CN:** 模块文档字符串给出的原始说明是：Inference-only GLM-4.7-Flash model compatible with HuggingFace weights.。
