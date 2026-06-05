# aria.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/aria.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for aria, including architecture wrappers and weight loading logic. / 面向推理的 aria vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-55)
```python
from collections.abc import Iterable, Mapping, Sequence
from typing import Annotated, Literal

import torch
import torch.nn as nn
from transformers import AriaConfig, AriaTextConfig, BatchFeature
from transformers.models.aria.modeling_aria import AriaCrossAttention
from transformers.models.aria.processing_aria import AriaProcessor

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.distributed import get_tensor_model_parallel_rank
from vllm.inputs import MultiModalDataDict
from vllm.model_executor.layers.activation import get_act_fn
from vllm.model_executor.layers.fused_moe import (
    FusedMoE,
)
from vllm.model_executor.layers.linear import ColumnParallelLinear, RowParallelLinear
from vllm.model_executor.layers.logits_processor import LogitsProcessor
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.vocab_parallel_embedding import ParallelLMHead
from vllm.model_executor.model_loader.weight_utils import (
    default_weight_loader,
    maybe_remap_kv_scale_name,
)
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFieldConfig,
    MultiModalKwargsItems,
)
from vllm.multimodal.parse import MultiModalDataItems
from vllm.multimodal.processing import (
    BaseDummyInputsBuilder,
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    PromptReplacement,
    PromptUpdate,
)
from vllm.sequence import IntermediateTensors
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .idefics2_vision_model import Idefics2VisionConfig
from .idefics2_vision_model import (
    Idefics2VisionTransformer as Idefics3VisionTransformer,
)
from .interfaces import MultiModalEmbeddings, SupportsMultiModal, SupportsQuant
from .llama import LlamaDecoderLayer, LlamaMLP, LlamaModel
from .utils import (
    AutoWeightsLoader,
    WeightsMapper,
    is_pp_missing_parameter,
    maybe_prefix,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Class `AriaImagePixelInputs` (lines 58-78)
```python
class AriaImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - b: Batch size
        - n: Number of images
        - c: Number of channels
        - h: Height of each image
        - w: Width of each image
    """

    type: Literal["pixel_values"]

    pixel_values: Annotated[
        torch.Tensor,
        TensorShape("bn", 3, "h", "w"),
    ]

    pixel_mask: Annotated[
        torch.Tensor | None,
        TensorShape("bn", "h", "w"),
    ]
```
**EN:** Class `AriaImagePixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `AriaImagePixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `AriaVisionTransformer` (lines 81-123)
```python
class AriaVisionTransformer(Idefics3VisionTransformer, SupportsQuant):
    packed_modules_mapping = {"qkv_proj": ["q_proj", "k_proj", "v_proj"]}

    def __init__(
        self,
        config: Idefics2VisionConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__(config, quant_config=quant_config, prefix=prefix)
        # Unlike Idefics3VisionTransformer which uses LayerNorm after the
        # final layer, Aria omits this normalization, so we replace it with an
        # Identity layer
        self.post_layernorm = nn.Identity()

    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
        ]
        params_dict = dict(self.named_parameters())
        loaded_params: set[str] = set()
        for name, loaded_weight in weights:
```
**EN:** Class `AriaVisionTransformer` organizes related behavior for this model family or helper component. It inherits from Idefics3VisionTransformer, SupportsQuant. Key methods include __init__, load_weights.
**CN:** 类 `AriaVisionTransformer` 用于组织该模型族或辅助组件的相关行为。 它继承自 Idefics3VisionTransformer、SupportsQuant。 关键方法包括 __init__, load_weights。

### Method `AriaVisionTransformer.__init__` (lines 84-94)
```python
    def __init__(
        self,
        config: Idefics2VisionConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__(config, quant_config=quant_config, prefix=prefix)
        # Unlike Idefics3VisionTransformer which uses LayerNorm after the
        # final layer, Aria omits this normalization, so we replace it with an
        # Identity layer
        self.post_layernorm = nn.Identity()
```
**EN:** Method `AriaVisionTransformer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `AriaVisionTransformer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `AriaVisionTransformer.load_weights` (lines 96-123)
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
        ]
        params_dict = dict(self.named_parameters())
        loaded_params: set[str] = set()
        for name, loaded_weight in weights:
            # NOTE: post_layernorm is not used in Aria
            if "post_layernorm" in name:
                continue

            for param_name, weight_name, shard_id in stacked_params_mapping:
                if weight_name not in name:
                    continue
                name = name.replace(weight_name, param_name)
                param = params_dict[name]
                weight_loader = param.weight_loader
                weight_loader(param, loaded_weight, shard_id)
                break
            else:
                param = params_dict[name]
                weight_loader = getattr(param, "weight_loader", default_weight_loader)
                weight_loader(param, loaded_weight)
            loaded_params.add(name)
        return loaded_params
```
**EN:** Method `AriaVisionTransformer.load_weights` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `AriaVisionTransformer.load_weights` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Class `AriaProjectorMLP` (lines 126-148)
```python
class AriaProjectorMLP(nn.Module):
    def __init__(
        self,
        in_features: int,
        hidden_features: int,
        output_dim: int,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.linear_in = ColumnParallelLinear(
            in_features, hidden_features, bias=False, prefix=f"{prefix}.linear_in"
        )
        self.linear_out = RowParallelLinear(
            hidden_features, output_dim, bias=False, prefix=f"{prefix}.linear_out"
        )
        self.act = get_act_fn("gelu_new")

    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states, _ = self.linear_in(hidden_states)
        hidden_states = self.act(hidden_states)
        hidden_states, _ = self.linear_out(hidden_states)
        return hidden_states
```
**EN:** Class `AriaProjectorMLP` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `AriaProjectorMLP` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `AriaProjectorMLP.__init__` (lines 127-142)
```python
    def __init__(
        self,
        in_features: int,
        hidden_features: int,
        output_dim: int,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.linear_in = ColumnParallelLinear(
            in_features, hidden_features, bias=False, prefix=f"{prefix}.linear_in"
        )
        self.linear_out = RowParallelLinear(
            hidden_features, output_dim, bias=False, prefix=f"{prefix}.linear_out"
        )
        self.act = get_act_fn("gelu_new")
```
**EN:** Method `AriaProjectorMLP.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `AriaProjectorMLP.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `AriaProjectorMLP.forward` (lines 144-148)
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states, _ = self.linear_in(hidden_states)
        hidden_states = self.act(hidden_states)
        hidden_states, _ = self.linear_out(hidden_states)
        return hidden_states
```
**EN:** Method `AriaProjectorMLP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `AriaProjectorMLP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `AriaProjector` (lines 151-216)
```python
class AriaProjector(nn.Module):
    """
    A projection module with one cross attention layer and one FFN layer, which
    projects ViT's outputs into MoE's inputs.

    Args:
        config: [AriaConfig](https://huggingface.co/docs/transformers/main/model_doc/aria#transformers.AriaConfig)
            containing projector configuration parameters.

    Outputs:
        A tensor with the shape of (batch_size, query_number, output_dim)
    """

    def __init__(self, config: AriaConfig, prefix: str = "") -> None:
        super().__init__()

        self.patch_to_query_dict = config.projector_patch_to_query_dict
        self.in_features = config.vision_config.hidden_size
        self.num_heads = config.vision_config.num_attention_heads
        self.kv_dim = config.vision_config.hidden_size
        self.hidden_features = config.text_config.hidden_size
        self.output_dim = config.text_config.hidden_size

        self.query = nn.Parameter(
            torch.empty(
```
**EN:** Class `AriaProjector` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `AriaProjector` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `AriaProjector.__init__` (lines 164-188)
```python
    def __init__(self, config: AriaConfig, prefix: str = "") -> None:
        super().__init__()

        self.patch_to_query_dict = config.projector_patch_to_query_dict
        self.in_features = config.vision_config.hidden_size
        self.num_heads = config.vision_config.num_attention_heads
        self.kv_dim = config.vision_config.hidden_size
        self.hidden_features = config.text_config.hidden_size
        self.output_dim = config.text_config.hidden_size

        self.query = nn.Parameter(
            torch.empty(
                config.max_value_projector_patch_to_query_dict, self.in_features
            )
        )

        self.cross_attn = AriaCrossAttention(config)

        self.layer_norm = nn.LayerNorm(self.in_features)
        self.feed_forward = AriaProjectorMLP(
            self.in_features,
            self.hidden_features,
            self.output_dim,
            prefix=f"{prefix}.feed_forward",
        )
```
**EN:** Method `AriaProjector.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `AriaProjector.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `AriaProjector.forward` (lines 190-216)
```python
    def forward(
        self,
        x: torch.Tensor,
        attn_mask: torch.Tensor | None = None,
    ) -> torch.Tensor:
        batch_size, num_patches = x.shape[0], x.shape[1]

        if num_patches not in self.patch_to_query_dict:
            raise KeyError(
                f"Number of patches {num_patches} not found in "
                "patch_to_query_dict amongst possible values "
                f"{self.patch_to_query_dict.keys()}."
            )

        query_num = self.patch_to_query_dict[num_patches]

        queries = self.query[:query_num].unsqueeze(0).repeat(batch_size, 1, 1)

        if attn_mask is not None:
            attn_mask = attn_mask.repeat_interleave(self.num_heads, 0)
            attn_mask = attn_mask.unsqueeze(1).expand(-1, queries.size(1), -1)

        attention_out = self.cross_attn(x, queries, attn_mask=attn_mask)

        out = self.feed_forward(self.layer_norm(attention_out))

        return out
```
**EN:** Method `AriaProjector.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `AriaProjector.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `AriaFusedMoE` (lines 219-248)
```python
class AriaFusedMoE(FusedMoE):
    def weight_loader(
        self, param: nn.Parameter, loaded_weight: torch.Tensor, shard_id: str
    ) -> None:
        # Override the weight_loader to handle the expert weights in the Aria
        # model, which are already packed with experts, and merge the gate and
        # up weights for each expert.
        # Note: Loading expert weights with quantization is not supported
        tp_rank = get_tensor_model_parallel_rank()
        if shard_id == "w13":
            # the shape of loaded_weight is
            # (num_experts, hidden_size, 2 * moe_intermediate_size)
            if self.tp_size > 1:
                up, gate = loaded_weight.chunk(2, dim=-1)
                up_current_rank = up.chunk(self.tp_size, dim=-1)[tp_rank]
                gate_current_rank = gate.chunk(self.tp_size, dim=-1)[tp_rank]
                up_and_gate = torch.cat(
                    [up_current_rank, gate_current_rank], dim=-1
                ).transpose(1, 2)
                param.data.copy_(up_and_gate)
            else:
                param.data.copy_(loaded_weight.transpose(1, 2))
        elif shard_id == "w2":
            # the shape of loaded_weight is
            # (num_experts, moe_intermediate_size, hidden_size)
```
**EN:** Class `AriaFusedMoE` organizes related behavior for this model family or helper component. It inherits from FusedMoE. Key methods include weight_loader.
**CN:** 类 `AriaFusedMoE` 用于组织该模型族或辅助组件的相关行为。 它继承自 FusedMoE。 关键方法包括 weight_loader。

### Method `AriaFusedMoE.weight_loader` (lines 220-248)
```python
    def weight_loader(
        self, param: nn.Parameter, loaded_weight: torch.Tensor, shard_id: str
    ) -> None:
        # Override the weight_loader to handle the expert weights in the Aria
        # model, which are already packed with experts, and merge the gate and
        # up weights for each expert.
        # Note: Loading expert weights with quantization is not supported
        tp_rank = get_tensor_model_parallel_rank()
        if shard_id == "w13":
            # the shape of loaded_weight is
            # (num_experts, hidden_size, 2 * moe_intermediate_size)
            if self.tp_size > 1:
                up, gate = loaded_weight.chunk(2, dim=-1)
                up_current_rank = up.chunk(self.tp_size, dim=-1)[tp_rank]
                gate_current_rank = gate.chunk(self.tp_size, dim=-1)[tp_rank]
                up_and_gate = torch.cat(
                    [up_current_rank, gate_current_rank], dim=-1
                ).transpose(1, 2)
                param.data.copy_(up_and_gate)
            else:
                param.data.copy_(loaded_weight.transpose(1, 2))
        elif shard_id == "w2":
            # the shape of loaded_weight is
            # (num_experts, moe_intermediate_size, hidden_size)
            if self.tp_size > 1:
                down_current_rank = loaded_weight.chunk(self.tp_size, dim=1)[tp_rank]
                param.data.copy_(down_current_rank.transpose(1, 2))
            else:
                param.data.copy_(loaded_weight.transpose(1, 2))
```
**EN:** Method `AriaFusedMoE.weight_loader` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `AriaFusedMoE.weight_loader` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Class `AriaTextMoELayer` (lines 251-305)
```python
class AriaTextMoELayer(nn.Module):
    """
    Mixture of Experts (MoE) Layer for the AriaMoE model.

    This layer implements the MoE mechanism, which routes input tokens to
    different experts based on a routing algorithm, processes them through the
    experts, and then combines the outputs.
    """

    def __init__(
        self,
        config: AriaTextConfig,
        quant_config: QuantizationConfig | None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config

        self.router_weight = nn.Parameter(
            torch.empty((self.config.moe_num_experts, self.config.hidden_size))
        )

        self.shared_experts = LlamaMLP(
            config.hidden_size,
            config.intermediate_size * config.moe_num_shared_experts,
```
**EN:** Class `AriaTextMoELayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `AriaTextMoELayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `AriaTextMoELayer.__init__` (lines 260-289)
```python
    def __init__(
        self,
        config: AriaTextConfig,
        quant_config: QuantizationConfig | None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config

        self.router_weight = nn.Parameter(
            torch.empty((self.config.moe_num_experts, self.config.hidden_size))
        )

        self.shared_experts = LlamaMLP(
            config.hidden_size,
            config.intermediate_size * config.moe_num_shared_experts,
            "silu",
            quant_config=quant_config,
            bias=config.mlp_bias,
        )

        self.experts = AriaFusedMoE(
            shared_experts=self.shared_experts,
            num_experts=config.moe_num_experts,
            top_k=config.moe_topk,
            hidden_size=config.hidden_size,
            intermediate_size=config.intermediate_size,
            quant_config=quant_config,
            prefix=f"{prefix}.experts",
        )
```
**EN:** Method `AriaTextMoELayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `AriaTextMoELayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `AriaTextMoELayer.forward` (lines 291-305)
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        """
        Forward pass of the MoE Layer.

        Args:
            hidden_states: Input tensor of shape
                (batch_size, sequence_length, hidden_size).

        Returns:
            torch.Tensor: Output tensor after passing through the MoE layer.
        """

        router_output = torch.nn.functional.linear(hidden_states, self.router_weight)

        return self.experts(hidden_states, router_output)
```
**EN:** Method `AriaTextMoELayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream. The docstring says: Forward pass of the MoE Layer.
**CN:** Method `AriaTextMoELayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。 文档字符串表达的核心意思是：Forward pass of the MoE Layer。

### Class `AriaTextDecoderLayer` (lines 308-323)
```python
class AriaTextDecoderLayer(LlamaDecoderLayer):
    """
    Custom Decoder Layer for the AriaMoE model which modifies the standard
    `LlamaDecoderLayer` by replacing the traditional MLP with a Mixture of
    Experts (MoE) Layer.
    """

    def __init__(self, vllm_config: VllmConfig, prefix: str = "") -> None:
        super().__init__(vllm_config, prefix)

        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        self.mlp = AriaTextMoELayer(
            config, quant_config=quant_config, prefix=f"{prefix}.mlp"
        )
```
**EN:** Class `AriaTextDecoderLayer` organizes related behavior for this model family or helper component. It inherits from LlamaDecoderLayer. Key methods include __init__.
**CN:** 类 `AriaTextDecoderLayer` 用于组织该模型族或辅助组件的相关行为。 它继承自 LlamaDecoderLayer。 关键方法包括 __init__。

### Method `AriaTextDecoderLayer.__init__` (lines 315-323)
```python
    def __init__(self, vllm_config: VllmConfig, prefix: str = "") -> None:
        super().__init__(vllm_config, prefix)

        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        self.mlp = AriaTextMoELayer(
            config, quant_config=quant_config, prefix=f"{prefix}.mlp"
        )
```
**EN:** Method `AriaTextDecoderLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `AriaTextDecoderLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Class `AriaTextModel` (lines 326-409)
```python
class AriaTextModel(LlamaModel, SupportsQuant):
    """
    Custom LlamaModel for the AriaMoE model which modifies the standard
    LlamaModel by replacing the `LlamaDecoderLayer` with `MoEDecoderLayer`.
    """

    packed_modules_mapping = {
        "qkv_proj": ["q_proj", "k_proj", "v_proj"],
        "gate_up_proj": ["gate_proj", "up_proj"],
        "experts.w13_weight": ["experts.fc1.weight"],
        "experts.w2_weight": ["experts.fc2.weight"],
    }

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__(
            vllm_config=vllm_config, prefix=prefix, layer_type=AriaTextDecoderLayer
        )

    # Adapted from LlamaModel.load_weights with the modification of adding
    # the expert weights mapping to `stacked_params_mapping`
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            (".qkv_proj", ".q_proj", "q"),
            (".qkv_proj", ".k_proj", "k"),
```
**EN:** Class `AriaTextModel` organizes related behavior for this model family or helper component. It inherits from LlamaModel, SupportsQuant. Key methods include __init__, load_weights.
**CN:** 类 `AriaTextModel` 用于组织该模型族或辅助组件的相关行为。 它继承自 LlamaModel、SupportsQuant。 关键方法包括 __init__, load_weights。

### Method `AriaTextModel.__init__` (lines 339-342)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__(
            vllm_config=vllm_config, prefix=prefix, layer_type=AriaTextDecoderLayer
        )
```
**EN:** Method `AriaTextModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `AriaTextModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `AriaTextModel.load_weights` (lines 346-409)
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            (".qkv_proj", ".q_proj", "q"),
            (".qkv_proj", ".k_proj", "k"),
            (".qkv_proj", ".v_proj", "v"),
            (".gate_up_proj", ".gate_proj", 0),
            (".gate_up_proj", ".up_proj", 1),
            ("experts.w13_weight", "experts.fc1.weight", "w13"),
            ("experts.w2_weight", "experts.fc2.weight", "w2"),
        ]
        params_dict = dict(self.named_parameters())
        loaded_params: set[str] = set()
        for name, loaded_weight in weights:
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
                # Remapping the name of FP8 kv-scale.
                name = maybe_remap_kv_scale_name(name, params_dict)
                if name is None:
                    continue

                if is_pp_missing_parameter(name, self):
                    continue

                param = params_dict[name]
# ... truncated for analysis ...
```
**EN:** Method `AriaTextModel.load_weights` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `AriaTextModel.load_weights` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Class `AriaProcessingInfo` (lines 412-427)
```python
class AriaProcessingInfo(BaseProcessingInfo):
    def get_hf_config(self):
        return self.ctx.get_hf_config(AriaConfig)

    def get_vision_config(self):
        return self.get_hf_config().vision_config

    def get_hf_processor(self, **kwargs: object):
        return self.ctx.get_hf_processor(AriaProcessor, **kwargs)

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}

    def get_num_image_tokens(self) -> int:
        hf_config = self.get_hf_config()
        return max(hf_config.projector_patch_to_query_dict.values())
```
**EN:** Class `AriaProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include get_hf_config, get_vision_config, get_hf_processor, get_supported_mm_limits, get_num_image_tokens.
**CN:** 类 `AriaProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 get_hf_config, get_vision_config, get_hf_processor, get_supported_mm_limits, get_num_image_tokens。

### Method `AriaProcessingInfo.get_hf_config` (lines 413-414)
```python
    def get_hf_config(self):
        return self.ctx.get_hf_config(AriaConfig)
```
**EN:** Method `AriaProcessingInfo.get_hf_config` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `AriaProcessingInfo.get_hf_config` 封装了该模块中的一段可复用核心逻辑。

### Method `AriaProcessingInfo.get_vision_config` (lines 416-417)
```python
    def get_vision_config(self):
        return self.get_hf_config().vision_config
```
**EN:** Method `AriaProcessingInfo.get_vision_config` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `AriaProcessingInfo.get_vision_config` 封装了该模块中的一段可复用核心逻辑。

### Method `AriaProcessingInfo.get_hf_processor` (lines 419-420)
```python
    def get_hf_processor(self, **kwargs: object):
        return self.ctx.get_hf_processor(AriaProcessor, **kwargs)
```
**EN:** Method `AriaProcessingInfo.get_hf_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `AriaProcessingInfo.get_hf_processor` 封装了该模块中的一段可复用核心逻辑。

### Method `AriaProcessingInfo.get_supported_mm_limits` (lines 422-423)
```python
    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}
```
**EN:** Method `AriaProcessingInfo.get_supported_mm_limits` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `AriaProcessingInfo.get_supported_mm_limits` 封装了该模块中的一段可复用核心逻辑。

### Class `AriaDummyInputsBuilder` (lines 430-459)
```python
class AriaDummyInputsBuilder(BaseDummyInputsBuilder[AriaProcessingInfo]):
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)

        processor = self.info.get_hf_processor()
        image_token: str = processor.tokenizer.image_token  # type: ignore

        return image_token * num_images

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        vision_config = self.info.get_vision_config()

        max_image_size = vision_config.image_size
        num_images = mm_counts.get("image", 0)

        image_overrides = mm_options.get("image")

        return {
            "image": self._get_dummy_images(
                width=max_image_size,
```
**EN:** Class `AriaDummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseDummyInputsBuilder[AriaProcessingInfo]. Key methods include get_dummy_text, get_dummy_mm_data.
**CN:** 类 `AriaDummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseDummyInputsBuilder[AriaProcessingInfo]。 关键方法包括 get_dummy_text, get_dummy_mm_data。

### Method `AriaDummyInputsBuilder.get_dummy_text` (lines 431-437)
```python
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)

        processor = self.info.get_hf_processor()
        image_token: str = processor.tokenizer.image_token  # type: ignore

        return image_token * num_images
```
**EN:** Method `AriaDummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `AriaDummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Method `AriaDummyInputsBuilder.get_dummy_mm_data` (lines 439-459)
```python
    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        vision_config = self.info.get_vision_config()

        max_image_size = vision_config.image_size
        num_images = mm_counts.get("image", 0)

        image_overrides = mm_options.get("image")

        return {
            "image": self._get_dummy_images(
                width=max_image_size,
                height=max_image_size,
                num_images=num_images,
                overrides=image_overrides,
            )
        }
```
**EN:** Method `AriaDummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `AriaDummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Class `AriaMultiModalProcessor` (lines 462-490)
```python
class AriaMultiModalProcessor(BaseMultiModalProcessor[AriaProcessingInfo]):
    def _get_mm_fields_config(
        self,
        hf_inputs: BatchFeature,
        hf_processor_mm_kwargs: Mapping[str, object],
    ) -> Mapping[str, MultiModalFieldConfig]:
        return dict(
            pixel_values=MultiModalFieldConfig.batched("image"),
            pixel_mask=MultiModalFieldConfig.batched("image"),
        )

    def _get_prompt_updates(
        self,
        mm_items: MultiModalDataItems,
        hf_processor_mm_kwargs: Mapping[str, object],
        out_mm_kwargs: MultiModalKwargsItems,
    ) -> Sequence[PromptUpdate]:
        hf_config = self.info.get_hf_config()
        image_token_id = hf_config.image_token_index

        num_image_tokens = self.info.get_num_image_tokens()

        return [
            PromptReplacement(
                modality="image",
```
**EN:** Class `AriaMultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor[AriaProcessingInfo]. Key methods include _get_mm_fields_config, _get_prompt_updates.
**CN:** 类 `AriaMultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor[AriaProcessingInfo]。 关键方法包括 _get_mm_fields_config, _get_prompt_updates。

### Class `AriaForConditionalGeneration` (lines 498-655)
```python
@MULTIMODAL_REGISTRY.register_processor(
    AriaMultiModalProcessor,
    info=AriaProcessingInfo,
    dummy_inputs=AriaDummyInputsBuilder,
)
class AriaForConditionalGeneration(nn.Module, SupportsMultiModal):
    """
    Aria model for conditional generation tasks.

    This model combines a vision tower, a multi-modal projector, and a language
    model to perform tasks that involve both image and text inputs.
    """

    hf_to_vllm_mapper = WeightsMapper(
        orig_to_new_prefix={
            # mapping for new names in checkpoint saved after transformers v4.52
            "model.language_model.": "language_model.model.",
            "model.vision_tower.": "vision_tower.",
            "model.multi_modal_projector.": "multi_modal_projector.",
            # mapping for original checkpoint
            "language_model.model": "language_model",
            "language_model.lm_head": "lm_head",
        },
        orig_to_new_suffix={
            "router.weight": "router_weight",
```
**EN:** Class `AriaForConditionalGeneration` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsMultiModal. Key methods include get_placeholder_str, __init__, _parse_and_validate_image_input, _create_patch_attention_mask, _process_image_input, embed_multimodal.
**CN:** 类 `AriaForConditionalGeneration` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsMultiModal。 关键方法包括 get_placeholder_str, __init__, _parse_and_validate_image_input, _create_patch_attention_mask, _process_image_input, embed_multimodal。

### Method `AriaForConditionalGeneration.get_placeholder_str` (lines 522-526)
```python
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
            return "<|fim_prefix|><|img|><|fim_suffix|>"

        raise ValueError("Only image modality is supported")
```
**EN:** Method `AriaForConditionalGeneration.get_placeholder_str` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `AriaForConditionalGeneration.get_placeholder_str` 封装了该模块中的一段可复用核心逻辑。

### Method `AriaForConditionalGeneration.__init__` (lines 528-565)
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        prefix: str = "",
    ):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        self.config = config

        with self._mark_tower_model(vllm_config, "image"):
            self.vision_tower = AriaVisionTransformer(
                config.vision_config,
                quant_config=quant_config,
                prefix=maybe_prefix(prefix, "vision_tower"),
            )
            self.multi_modal_projector = AriaProjector(
                config, prefix=maybe_prefix(prefix, "multi_modal_projector")
            )

        with self._mark_language_model(vllm_config):
            self.language_model = AriaTextModel(
                vllm_config=vllm_config.with_hf_config(config.text_config),
                prefix=maybe_prefix(prefix, "language_model.model"),
            )

            self.lm_head = ParallelLMHead(
                config.text_config.vocab_size,
                config.text_config.hidden_size,
                quant_config=quant_config,
                prefix=maybe_prefix(prefix, "lm_head"),
            )

            logit_scale = getattr(config, "logit_scale", 1.0)
            self.logits_processor = LogitsProcessor(
                config.text_config.vocab_size, scale=logit_scale
            )
```
**EN:** Method `AriaForConditionalGeneration.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `AriaForConditionalGeneration.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `AriaForConditionalGeneration.embed_multimodal` (lines 619-624)
```python
    def embed_multimodal(self, **kwargs: object) -> MultiModalEmbeddings:
        image_input = self._parse_and_validate_image_input(**kwargs)
        if image_input is None:
            return []
        multimodal_embeddings = self._process_image_input(image_input)
        return multimodal_embeddings
```
**EN:** Method `AriaForConditionalGeneration.embed_multimodal` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `AriaForConditionalGeneration.embed_multimodal` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `AriaForConditionalGeneration.forward` (lines 626-644)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        **kwargs: object,
    ) -> torch.Tensor | IntermediateTensors:
        if intermediate_tensors is not None:
            inputs_embeds = None

        hidden_states = self.language_model(
            input_ids,
            positions,
            intermediate_tensors,
            inputs_embeds=inputs_embeds,
        )

        return hidden_states
```
**EN:** Method `AriaForConditionalGeneration.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `AriaForConditionalGeneration.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

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
- **Standard library / 标准库**: `from collections.abc import Iterable, Mapping, Sequence`, `from typing import Annotated, Literal`
- **Third-party / 第三方**: `import torch`, `import torch.nn as nn`, `from transformers import AriaConfig, AriaTextConfig, BatchFeature`, `from transformers.models.aria.modeling_aria import AriaCrossAttention`, `from transformers.models.aria.processing_aria import AriaProcessor`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.distributed import get_tensor_model_parallel_rank`, `from vllm.inputs import MultiModalDataDict`, `from vllm.model_executor.layers.activation import get_act_fn`, `from vllm.model_executor.layers.fused_moe import (`, `from vllm.model_executor.layers.linear import ColumnParallelLinear, RowParallelLinear`, `from vllm.model_executor.layers.logits_processor import LogitsProcessor`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.layers.vocab_parallel_embedding import ParallelLMHead`, `from vllm.model_executor.model_loader.weight_utils import (`, `from vllm.multimodal import MULTIMODAL_REGISTRY`
