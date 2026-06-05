# param2moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/param2moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Param2moe model components and runtime adapter for vLLM inference. / 实现 Param2moe 在 vLLM 推理中的模型组件与运行时适配器。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-65)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
#
# Copyright 2026 BharatGen AI team. All rights reserved.
#
# This code has been modified to accommodate Param2MoE's GQA-based MoE architecture.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ... omitted for brevity ...
    VocabParallelEmbedding,
)
from vllm.model_executor.model_loader.weight_utils import default_weight_loader
from vllm.sequence import IntermediateTensors

from .interfaces import MixtureOfExperts, SupportsLoRA, SupportsPP
from .utils import (
    AutoWeightsLoader,
    PPMissingLayer,
    is_pp_missing_parameter,
    make_empty_intermediate_tensors_factory,
    make_layers,
    maybe_prefix,
)
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, torch.nn.functional supply framework primitives, while internal modules like vllm.config, vllm.distributed, vllm.model_executor.layers.activation, vllm.model_executor.layers.attention connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, torch.nn.functional 这样的外部依赖提供基础框架能力，而 vllm.config, vllm.distributed, vllm.model_executor.layers.activation, vllm.model_executor.layers.attention 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Function `_is_expert_bias_name` (lines 66-68)
```python
def _is_expert_bias_name(name: str) -> bool:
    """True when the weight is the MoE router's per-expert score bias."""
    return name.endswith(".mlp.gate.expert_bias")
```
**EN:** The function `_is_expert_bias_name` helps provide a reusable helper for the surrounding model code. Its main inputs are `name`. Docstring hint: "True when the weight is the MoE router's per-expert score bias."
**CN:** 函数 `_is_expert_bias_name` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `name`。 文档提示：“True when the weight is the MoE router's per-expert score bias。”

### Function `_zero_mean_tensor` (lines 71-74)
```python
def _zero_mean_tensor(t: torch.Tensor) -> torch.Tensor:
    if t.numel() == 0:
        return t
    return t - t.mean()
```
**EN:** The function `_zero_mean_tensor` helps provide a reusable helper for the surrounding model code. Its main inputs are `t`.
**CN:** 函数 `_zero_mean_tensor` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `t`。

### Function `_rename_and_normalize_weights` (lines 77-116)
```python
def _rename_and_normalize_weights(
    weights: Iterable[tuple[str, torch.Tensor]],
) -> Iterator[tuple[str, torch.Tensor]]:
    """
    Translate HuggingFace Param2MoE weight names to vLLM internal names
    and zero-mean the expert-bias tensor so the router stays balanced.

    Mapping table (HF → vLLM):
      model.word_embeddings.*              → model.embed_tokens.*
      *.attention.query_key_value.*        → *.self_attn.qkv_proj.*
      *.attention.dense.*                  → *.self_attn.o_proj.*
      *.attention.query_layernorm.*        → *.self_attn.q_layernorm.*
      *.attention.key_layernorm.*          → *.self_attn.k_layernorm.*
      *.mlp.gate.expert_bias               → *.mlp.gate.e_score_correction_bias
        (also zero-meant for load balance)
    """
    for name, w in weights:
        # Embedding table
        name = name.replace("model.word_embeddings.", "model.embed_tokens.")
        # Fused QKV projection  (HF: query_key_value → vLLM: qkv_proj)
        name = name.replace(".attention.query_key_value.", ".self_attn.qkv_proj.")
        # Output projection  (HF: dense → vLLM: o_proj)
        name = name.replace(".attention.dense.", ".self_attn.o_proj.")
        # Per-head query norm
        name = name.replace(".attention.query_layernorm.", ".self_attn.q_layernorm.")
        # Per-head key norm
        name = name.replace(".attention.key_layernorm.", ".self_attn.k_layernorm.")
        # Catch any remaining .attention. → .self_attn. prefixes
        # (e.g. future bias params on the projection layers)
        name = name.replace(".attention.", ".self_attn.")

        # Expert-score bias: rename + zero-mean
        if name.endswith(".mlp.gate.expert_bias"):
            name = name.replace(
                ".mlp.gate.expert_bias",
                ".mlp.gate.e_score_correction_bias",
            )
            w = _zero_mean_tensor(w)

        yield name, w
```
**EN:** The function `_rename_and_normalize_weights` helps provide a reusable helper for the surrounding model code. Its main inputs are `weights`. Docstring hint: "Translate HuggingFace Param2MoE weight names to vLLM internal names and zero-mean the expert-bias tensor so the router stays balanced."
**CN:** 函数 `_rename_and_normalize_weights` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `weights`。 文档提示：“Translate HuggingFace Param2MoE weight names to vLLM internal names and zero-mean the expert-bias tensor so the router stays balanced。”

### Class `Param2MoEAttention` (lines 119-242)
```python
class Param2MoEAttention(nn.Module):
    """
    Grouped-Query Attention (GQA) for Param2MoE.

    Notable differences from a vanilla GQA layer:
      * The checkpoint fuses Q, K, V into a single ``query_key_value`` weight.
        vLLM receives it already renamed to ``qkv_proj`` by the weight-name
        translator and splits it during ``load_weights``.
      * Optional per-head RMS norms on Q and K (``use_qk_norm=True``).
# ... omitted for brevity ...
    def __init__(
        self,
        config,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.hidden_size = config.hidden_size
        self.num_heads = config.num_attention_heads
        self.num_kv_heads = config.num_key_value_heads
        self.head_dim = config.head_dim or (self.hidden_size // self.num_heads)
        self.use_qk_norm: bool = getattr(config, "use_qk_norm", False)

        tp_size = get_tensor_model_parallel_world_size()
        assert self.num_heads % tp_size == 0, (
            f"num_attention_heads ({self.num_heads}) must be divisible "
            f"by tensor-parallel world size ({tp_size})."
# ... omitted for brevity ...
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split(
            [self.q_size_local, self.kv_size_local, self.kv_size_local],
            dim=-1,
        )
        q = q.contiguous()
        k = k.contiguous()
        v = v.contiguous()

        if self.use_qk_norm:
            T = q.shape[0]
            q = self.q_layernorm(q.view(T, self.num_local_heads, self.head_dim)).view(
                T, self.q_size_local
            )
```
**EN:** Defines `Param2MoEAttention`, a attention block that projects Q/K/V tensors and dispatches the cache-aware attention path. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens. Docstring hint: "Grouped-Query Attention (GQA) for Param2MoE."
**CN:** 定义 `Param2MoEAttention`，它是一个负责投影 Q/K/V 张量并分发带缓存注意力路径的注意力模块。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“Grouped-Query Attention (GQA) for Param2MoE。”

### Class `Param2MoEMLP` (lines 245-279)
```python
class Param2MoEMLP(nn.Module):
    """SwiGLU feed-forward block used for dense layers."""

    def __init__(
        self,
        intermediate_size: int,
        config,
        quant_config: QuantizationConfig | None = None,
        reduce_results: bool = True,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.gate_up_proj = MergedColumnParallelLinear(
            input_size=config.hidden_size,
            output_sizes=[intermediate_size, intermediate_size],
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.gate_up_proj",
        )
        self.down_proj = RowParallelLinear(
            input_size=intermediate_size,
            output_size=config.hidden_size,
            bias=False,
            quant_config=quant_config,
            reduce_results=reduce_results,
            prefix=f"{prefix}.down_proj",
        )
        self.act_fn = SiluAndMul()

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        gate_up, _ = self.gate_up_proj(x)
        x = self.act_fn(gate_up)
        x, _ = self.down_proj(x)
        return x
```
**EN:** Defines `Param2MoEMLP`, a feed-forward block that expands hidden states and projects them back. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens. Docstring hint: "SwiGLU feed-forward block used for dense layers."
**CN:** 定义 `Param2MoEMLP`，它是一个扩展隐藏状态并再投影回主维度的前馈模块。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“SwiGLU feed-forward block used for dense layers。”

### Class `Param2MoEMoEBlock` (lines 282-398)
```python
class Param2MoEMoEBlock(nn.Module):
    """
    Mixture-of-Experts block for Param2MoE.

    Routing:
      * Sigmoid scoring  (config.score_function = "sigmoid")
      * Grouped top-k   (n_group, topk_group)
      * Per-expert bias  (gate.expert_bias → e_score_correction_bias)
      * routed_scaling_factor normalisation
# ... omitted for brevity ...
    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.config = config
        self.tp_size = get_tensor_model_parallel_world_size()
        self.hidden_size = config.hidden_size

        self.num_experts: int = config.num_experts
        self.top_k: int = config.num_experts_per_tok
        self.routed_scaling_factor: float = getattr(
            config, "routed_scaling_factor", 1.0
        )

        self.n_group: int | None = getattr(config, "n_group", None)
# ... omitted for brevity ...
    def maybe_get_fused_moe(self) -> FusedMoE:
        return self.experts
# ... omitted for brevity ...
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        num_tokens, hidden_dim = hidden_states.shape
        hidden_states = hidden_states.view(-1, hidden_dim)

        # Router: both input and weight must be float32 for numerical
        # stability (mirrors the original Param2MoEGate behaviour).
        # The gate nn.Linear weight lives in the model dtype (bfloat16),
        # so we must cast both explicitly via F.linear instead of calling
        # self.gate() which would hit a dtype mismatch.
        router_logits = F.linear(
            hidden_states.float(),
            self.gate.weight.float(),
        ).to(hidden_states.dtype)

        expert_output = self.experts(
            hidden_states=hidden_states,
            router_logits=router_logits,
        )
```
**EN:** Defines `Param2MoEMoEBlock`, a supporting module used by the surrounding model implementation. It inherits from nn.Module. Key methods such as `__init__`, `maybe_get_fused_moe`, `forward` show where construction, forward execution, or weight adaptation happens. Docstring hint: "Mixture-of-Experts block for Param2MoE."
**CN:** 定义 `Param2MoEMoEBlock`，它是一个被周边模型实现复用的支撑模块。 它继承自 nn.Module。 `__init__`, `maybe_get_fused_moe`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“Mixture-of-Experts block for Param2MoE。”

### Class `Param2MoEDecoderLayer` (lines 401-471)
```python
class Param2MoEDecoderLayer(nn.Module):
    """
    Single transformer decoder block.

    Dense for the first ``first_k_dense_replace`` layers; MoE thereafter.
    """

    def __init__(
        self,
        vllm_config: VllmConfig,
        prefix: str = "",
    ) -> None:
        super().__init__()

        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        hidden_size = config.hidden_size
        # Derive the layer index from the prefix (e.g. "model.layers.3")
        layer_idx = int(prefix.split(".")[-1])

        self.input_layernorm = RMSNorm(hidden_size, eps=config.rms_norm_eps)
        self.self_attn = Param2MoEAttention(
            config=config,
            cache_config=cache_config,
# ... omitted for brevity ...
    def forward(
        self,
        hidden_states: torch.Tensor,
        positions: torch.Tensor,
        residual: torch.Tensor | None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        # Pre-norm + attention
        if residual is None:
            residual = hidden_states
            hidden_states = self.input_layernorm(hidden_states)
        else:
            hidden_states, residual = self.input_layernorm(hidden_states, residual)

        hidden_states = self.self_attn(
            positions=positions,
            hidden_states=hidden_states,
        )

        # Pre-norm + MLP
```
**EN:** Defines `Param2MoEDecoderLayer`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens. Docstring hint: "Single transformer decoder block."
**CN:** 定义 `Param2MoEDecoderLayer`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“Single transformer decoder block。”

### Class `Param2MoEModel` (lines 474-702)
```python
class Param2MoEModel(nn.Module):
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
    ) -> None:
        super().__init__()

        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        self.config = config
        self.vocab_size = config.vocab_size
        self.embed_dim = config.hidden_size
        self.tie_word_embeddings: bool = getattr(config, "tie_word_embeddings", False)

        # Embedding  (HF name: word_embeddings → vLLM name: embed_tokens)
        if get_pp_group().is_first_rank or (
            self.tie_word_embeddings and get_pp_group().is_last_rank
# ... omitted for brevity ...
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.embed_tokens(input_ids)
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None,
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
# ... omitted for brevity ...
    def load_weights(
        self,
        weights: Iterable[tuple[str, torch.Tensor]],
    ) -> set[str]:
        """
        Custom weight loader for the inner Param2MoEModel.

        Receives weights that have already been renamed/normalised by the
        outer model and whose ``model.`` prefix has been stripped by
        ``AutoWeightsLoader``.  Handles:
          1. Fused QKV split (query_key_value → qkv_proj q/k/v shards).
          2. gate_proj + up_proj → gate_up_proj stacking (dense + shared-exp).
          3. Routed-expert weights via the fused-MoE mapping.
          4. All remaining weights via their default loader.
        """
        config = self.config
        num_heads: int = config.num_attention_heads
        num_kv_heads: int = config.num_key_value_heads
        head_dim: int = config.head_dim or (config.hidden_size // num_heads)
```
**EN:** Defines `Param2MoEModel`, a backbone model container that orchestrates embeddings, layers, and output heads. It inherits from nn.Module. Key methods such as `__init__`, `embed_input_ids`, `forward`, `load_weights`, `get_expert_mapping` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Param2MoEModel`，它是一个协调嵌入、层堆叠与输出头的主干模型容器。 它继承自 nn.Module。 `__init__`, `embed_input_ids`, `forward`, `load_weights`, `get_expert_mapping` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Param2MoEMixtureOfExperts` (lines 705-764)
```python
class Param2MoEMixtureOfExperts(MixtureOfExperts):
    """Implements the vLLM MixtureOfExperts protocol for Param2MoE."""

    expert_weights: list[torch.Tensor]

    def extract_moe_parameters(self, example_moe: Param2MoEMoEBlock | None) -> None:
        if example_moe is None:
            raise RuntimeError(
                "No Param2MoEMoEBlock found in model.layers. "
                "Check first_k_dense_replace and num_experts in config."
            )
        self.num_logical_experts = example_moe.num_experts
        self.num_routed_experts = example_moe.num_experts
        self.num_shared_experts = example_moe.num_shared_experts

        self.num_physical_experts = self.num_logical_experts
        self.num_local_physical_experts = self.num_logical_experts
        self.num_redundant_experts = 0

    def update_physical_experts_metadata(
        self,
        num_physical_experts: int,
        num_local_physical_experts: int,
    ) -> None:
        self.num_physical_experts = num_physical_experts
        self.num_local_physical_experts = num_local_physical_experts
        self.num_redundant_experts = num_physical_experts - self.num_logical_experts

        for moe in self.moe_mlp_layers:
            moe.n_physical_experts = num_physical_experts
            moe.n_local_physical_experts = num_local_physical_experts
            moe.n_redundant_experts = self.num_redundant_experts

            fused = moe.experts
            if hasattr(fused, "n_local_physical_experts"):
                fused.n_local_physical_experts = num_local_physical_experts
            if hasattr(fused, "n_physical_experts"):
                fused.n_physical_experts = num_physical_experts
            if hasattr(fused, "n_redundant_experts"):
                fused.n_redundant_experts = self.num_redundant_experts
            if hasattr(fused, "update_expert_map"):
                fused.update_expert_map()

    def set_eplb_state(
        self,
        expert_load_view: torch.Tensor,
        logical_to_physical_map: torch.Tensor,
        logical_replica_count: torch.Tensor,
    ) -> None:
        self.expert_weights.clear()
        for layer_idx, layer in enumerate(self.moe_layers):
            if hasattr(layer, "get_expert_weights"):
                self.expert_weights.append(layer.get_expert_weights())
            if hasattr(layer, "set_eplb_state"):
                layer.set_eplb_state(
                    moe_layer_idx=layer_idx,
                    expert_load_view=expert_load_view,
                    logical_to_physical_map=logical_to_physical_map,
                    logical_replica_count=logical_replica_count,
                )
```
**EN:** Defines `Param2MoEMixtureOfExperts`, a supporting module used by the surrounding model implementation. It inherits from MixtureOfExperts. Key methods such as `extract_moe_parameters`, `update_physical_experts_metadata`, `set_eplb_state` show where construction, forward execution, or weight adaptation happens. Docstring hint: "Implements the vLLM MixtureOfExperts protocol for Param2MoE."
**CN:** 定义 `Param2MoEMixtureOfExperts`，它是一个被周边模型实现复用的支撑模块。 它继承自 MixtureOfExperts。 `extract_moe_parameters`, `update_physical_experts_metadata`, `set_eplb_state` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“Implements the vLLM MixtureOfExperts protocol for Param2MoE。”

### Class `Param2MoEForCausalLM` (lines 767-883)
```python
class Param2MoEForCausalLM(
    nn.Module, SupportsPP, SupportsLoRA, Param2MoEMixtureOfExperts
):
    """
    vLLM-native Param2MoE CausalLM.

    Uses Grouped-Query Attention (GQA) with a Sigmoid-scored,
    grouped-topk Mixture-of-Experts MLP.
    """
# ... omitted for brevity ...
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "") -> None:
        super().__init__()

        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        self.config = config
        self.quant_config = quant_config

        self.model = Param2MoEModel(
            vllm_config=vllm_config,
            prefix=maybe_prefix(prefix, "model"),
        )

        self.tie_word_embeddings: bool = getattr(config, "tie_word_embeddings", False)
        if get_pp_group().is_last_rank:
            if self.tie_word_embeddings:
                self.lm_head = self.model.embed_tokens
            else:
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor | IntermediateTensors:
        return self.model(
            input_ids=input_ids,
            positions=positions,
            intermediate_tensors=intermediate_tensors,
            inputs_embeds=inputs_embeds,
        )
# ... omitted for brevity ...
    def compute_logits(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor | None:
        if not get_pp_group().is_last_rank:
            return None
        return self.logits_processor(self.lm_head, hidden_states)
# ... omitted for brevity ...
    def load_weights(
        self,
        weights: Iterable[tuple[str, torch.Tensor]],
    ) -> set[str]:
        loader = AutoWeightsLoader(self)
        return loader.load_weights(_rename_and_normalize_weights(weights))
```
**EN:** Defines `Param2MoEForCausalLM`, a top-level language-model wrapper that exposes logits and weight-loading behavior. It inherits from nn.Module, SupportsPP, SupportsLoRA. Key methods such as `__init__`, `embed_input_ids`, `forward`, `compute_logits`, `load_weights` show where construction, forward execution, or weight adaptation happens. Docstring hint: "vLLM-native Param2MoE CausalLM."
**CN:** 定义 `Param2MoEForCausalLM`，它是一个暴露 logits 计算与权重加载行为的顶层语言模型封装类。 它继承自 nn.Module、SupportsPP、SupportsLoRA。 `__init__`, `embed_input_ids`, `forward`, `compute_logits`, `load_weights` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“vLLM-native Param2MoE CausalLM。”

## Key Concepts / 关键概念
- **Attention and KV cache**
  - **EN:** Core logic manages query/key/value projection and integrates with vLLM attention backends and cache layout.
  - **CN:** 核心逻辑管理 Q/K/V 投影，并与 vLLM 的注意力后端及 KV Cache 布局集成。
- **Distributed execution**
  - **EN:** The implementation is aware of tensor parallelism or pipeline parallelism, so layers and weights can be sharded across devices.
  - **CN:** 实现考虑了张量并行或流水线并行，因此层和权重可以在多设备间切分。
- **Quantized weights**
  - **EN:** Quantization hooks allow the model to load compressed checkpoints or specialized linear layers.
  - **CN:** 量化钩子使模型能够加载压缩检查点或专用线性层。
- **Checkpoint remapping**
  - **EN:** Weight-loading helpers translate Hugging Face checkpoints into the parameter names and tensor layout expected by vLLM.
  - **CN:** 权重加载辅助逻辑会把 Hugging Face 检查点转换成 vLLM 期望的参数名和张量布局。
- **Mixture-of-Experts routing**
  - **EN:** The code contains expert selection or grouped feed-forward logic typical of MoE architectures.
  - **CN:** 代码包含专家选择或分组前馈逻辑，这是 MoE 架构的典型特征。

## Dependencies / 依赖关系
- **Standard library**: __future__, collections.abc, itertools
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, torch.nn.functional
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.config, vllm.distributed, vllm.model_executor.layers.activation, vllm.model_executor.layers.attention, vllm.model_executor.layers.fused_moe, vllm.model_executor.layers.layernorm, vllm.model_executor.layers.linear, vllm.model_executor.layers.logits_processor
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .interfaces, .utils
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
