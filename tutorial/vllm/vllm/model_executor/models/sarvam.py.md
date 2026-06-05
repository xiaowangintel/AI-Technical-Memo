# sarvam.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/sarvam.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Sarvam model components and runtime adapter for vLLM inference. / 实现 Sarvam 在 vLLM 推理中的模型组件与运行时适配器。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-71)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
#
# Copyright 2026 Sarvam AI team. All rights reserved.
#
# This code is based on Llama, Deepseek, and Bailing MoE implementations
# in this library. It has been modified from its original forms to
# accommodate Sarvam's MoE architectures.
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
# ... omitted for brevity ...
)
from vllm.model_executor.model_loader.weight_utils import default_weight_loader
from vllm.sequence import IntermediateTensors

from .bailing_moe import BailingMoeForCausalLM
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
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch supply framework primitives, while internal modules like vllm.config, vllm.distributed, vllm.model_executor.layers.activation, vllm.model_executor.layers.fused_moe connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch 这样的外部依赖提供基础框架能力，而 vllm.config, vllm.distributed, vllm.model_executor.layers.activation, vllm.model_executor.layers.fused_moe 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Function `yarn_get_mscale` (lines 72-75)
```python
def yarn_get_mscale(scale: float = 1, mscale: float = 1) -> float:
    if scale <= 1:
        return 1.0
    return 0.1 * mscale * math.log(scale) + 1.0
```
**EN:** The function `yarn_get_mscale` helps provide a reusable helper for the surrounding model code. Its main inputs are `scale`, `mscale`.
**CN:** 函数 `yarn_get_mscale` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `scale`、`mscale`。

### Function `_is_gate_expert_bias_name` (lines 78-81)
```python
def _is_gate_expert_bias_name(name: str) -> bool:
    return name.endswith(".mlp.gate.e_score_correction_bias") or name.endswith(
        ".gate.e_score_correction_bias"
    )
```
**EN:** The function `_is_gate_expert_bias_name` helps provide a reusable helper for the surrounding model code. Its main inputs are `name`.
**CN:** 函数 `_is_gate_expert_bias_name` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `name`。

### Class `SarvamMLAAttention` (lines 100-235)
```python
class SarvamMLAAttention(nn.Module):
    def __init__(
        self,
        vllm_config: VllmConfig,
        config,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.config = config
        self.hidden_size = config.hidden_size
        self.qk_nope_head_dim = config.qk_nope_head_dim
        self.qk_rope_head_dim = config.qk_rope_head_dim
        self.qk_head_dim = self.qk_nope_head_dim + self.qk_rope_head_dim
        self.v_head_dim = config.v_head_dim

        self.q_lora_rank = getattr(config, "q_lora_rank", None)
        self.kv_lora_rank = config.kv_lora_rank
# ... omitted for brevity ...
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        return self.mla_attn(positions, hidden_states, llama_4_scaling=None)
```
**EN:** Defines `SarvamMLAAttention`, a attention block that projects Q/K/V tensors and dispatches the cache-aware attention path. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `SarvamMLAAttention`，它是一个负责投影 Q/K/V 张量并分发带缓存注意力路径的注意力模块。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `SarvamMLAMLP` (lines 238-270)
```python
class SarvamMLAMLP(nn.Module):
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
            config.hidden_size,
            [intermediate_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.gate_up_proj",
        )
        self.down_proj = RowParallelLinear(
            intermediate_size,
            config.hidden_size,
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
**EN:** Defines `SarvamMLAMLP`, a feed-forward block that expands hidden states and projects them back. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `SarvamMLAMLP`，它是一个扩展隐藏状态并再投影回主维度的前馈模块。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `SarvamMLAMoE` (lines 273-375)
```python
class SarvamMLAMoE(nn.Module):
    def __init__(
        self,
        config,
        parallel_config: ParallelConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.config = config
        self.tp_size = get_tensor_model_parallel_world_size()
        self.tp_rank = get_tensor_model_parallel_rank()
        self.hidden_size = config.hidden_size

        self.num_experts = config.num_experts
        self.top_k = config.num_experts_per_tok
        self.routed_scaling_factor = getattr(config, "routed_scaling_factor", 2.5)

        self.n_group = getattr(config, "n_group", None)
# ... omitted for brevity ...
    def maybe_get_fused_moe(self) -> FusedMoE:
        return self.experts
# ... omitted for brevity ...
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        num_tokens, hidden_dim = hidden_states.shape
        hidden_states = hidden_states.view(-1, hidden_dim)
        router_logits = self.gate(
            hidden_states.to(self.router_dtype)
            if self.router_dtype is not None
            else hidden_states
        )
        router_logits = router_logits.to(hidden_states.dtype)
        final_hidden = self.experts(
            hidden_states=hidden_states,
            router_logits=router_logits,
        )

        return final_hidden.view(num_tokens, hidden_dim)
```
**EN:** Defines `SarvamMLAMoE`, a supporting module used by the surrounding model implementation. It inherits from nn.Module. Key methods such as `__init__`, `maybe_get_fused_moe`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `SarvamMLAMoE`，它是一个被周边模型实现复用的支撑模块。 它继承自 nn.Module。 `__init__`, `maybe_get_fused_moe`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `SarvamMLABlock` (lines 378-446)
```python
class SarvamMLABlock(nn.Module):
    def __init__(
        self,
        vllm_config: VllmConfig,
        prefix: str = "",
    ) -> None:
        super().__init__()
        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config
        parallel_config = vllm_config.parallel_config
        layer_idx = int(prefix.split(".")[-1])
        hidden_size = config.hidden_size
        dense_intermediate = getattr(config, "intermediate_size", 16384)

        self.input_layernorm = RMSNorm(hidden_size, eps=config.rms_norm_eps)
        self.self_attn = SarvamMLAAttention(
            vllm_config=vllm_config,
            config=config,
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
        )
        self.post_attention_layernorm = RMSNorm(hidden_size, eps=config.rms_norm_eps)
        use_moe = hasattr(config, "num_experts") and config.num_experts is not None
        first_k_dense = getattr(config, "first_k_dense_replace", 1)
        moe_layer_freq = getattr(config, "moe_layer_freq", 1)
        if use_moe:
            is_moe_layer = layer_idx >= first_k_dense and (
                (layer_idx - first_k_dense) % moe_layer_freq == 0
# ... omitted for brevity ...
            )
        else:
            self.mlp = SarvamMLAMLP(
                intermediate_size=dense_intermediate,
                config=config,
                quant_config=quant_config,
                reduce_results=True,
                prefix=f"{prefix}.mlp",
            )

    def forward(
        self,
        hidden_states: torch.Tensor,
        positions: torch.Tensor,
        residual: torch.Tensor | None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        if residual is None:
            residual = hidden_states
            hidden_states = self.input_layernorm(hidden_states)
        else:
            hidden_states, residual = self.input_layernorm(hidden_states, residual)

        hidden_states = self.self_attn(
            positions=positions,
            hidden_states=hidden_states,
        )
        hidden_states, residual = self.post_attention_layernorm(hidden_states, residual)
        hidden_states = self.mlp(hidden_states)
        return hidden_states, residual
```
**EN:** Defines `SarvamMLABlock`, a supporting module used by the surrounding model implementation. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `SarvamMLABlock`，它是一个被周边模型实现复用的支撑模块。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `SarvamMLAModel` (lines 449-624)
```python
class SarvamMLAModel(nn.Module):
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
        self.tie_word_embeddings = getattr(config, "tie_word_embeddings", False)
        if get_pp_group().is_first_rank or (
            self.tie_word_embeddings and get_pp_group().is_last_rank
        ):
            self.embed_tokens = VocabParallelEmbedding(
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
            hidden_states = self.embedding_dropout(hidden_states)
            residual = None
        else:
            assert intermediate_tensors is not None
            hidden_states = intermediate_tensors["hidden_states"]
            residual = intermediate_tensors["residual"]

# ... omitted for brevity ...
    def load_weights(
        self,
        weights: Iterable[tuple[str, torch.Tensor]],
    ) -> set[str]:
        """Load weights with stacked gate+up and MoE expert remapping."""
        weights = _normalized_weights(weights)
        stacked_params_mapping = [
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
        ]

        params_dict = dict(self.named_parameters(remove_duplicate=False))
        loaded_params: set[str] = set()
        expert_params_mapping = self.get_expert_mapping()

        for name, loaded_weight in weights:
            for param_name, weight_name, shard_id in stacked_params_mapping:
                if weight_name not in name:
                    continue
```
**EN:** Defines `SarvamMLAModel`, a backbone model container that orchestrates embeddings, layers, and output heads. It inherits from nn.Module. Key methods such as `__init__`, `embed_input_ids`, `forward`, `get_expert_mapping`, `load_weights` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `SarvamMLAModel`，它是一个协调嵌入、层堆叠与输出头的主干模型容器。 它继承自 nn.Module。 `__init__`, `embed_input_ids`, `forward`, `get_expert_mapping`, `load_weights` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `SarvamMixtureOfExperts` (lines 627-668)
```python
class SarvamMixtureOfExperts(MixtureOfExperts):
    def extract_moe_parameters(self, example_moe: SarvamMLAMoE | None) -> None:
        if example_moe is None:
            raise RuntimeError("No SarvamMLAMoE layer found in model.layers.")

        self.num_logical_experts = example_moe.num_experts
        self.num_routed_experts = example_moe.num_experts  # routed pool size
        self.num_shared_experts = getattr(example_moe.config, "num_shared_experts", 1)

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

    def set_eplb_state(self, eplb_state) -> None:
        self.eplb_state = eplb_state
        for moe in self.moe_layers:
            if hasattr(moe, "set_eplb_state"):
                moe.set_eplb_state(eplb_state)
```
**EN:** Defines `SarvamMixtureOfExperts`, a supporting module used by the surrounding model implementation. It inherits from MixtureOfExperts. Key methods such as `extract_moe_parameters`, `update_physical_experts_metadata`, `set_eplb_state` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `SarvamMixtureOfExperts`，它是一个被周边模型实现复用的支撑模块。 它继承自 MixtureOfExperts。 `extract_moe_parameters`, `update_physical_experts_metadata`, `set_eplb_state` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `SarvamMLAForCausalLM` (lines 671-768)
```python
class SarvamMLAForCausalLM(nn.Module, SupportsPP, SupportsLoRA, SarvamMixtureOfExperts):
    packed_modules_mapping = {
        "q_proj": ["q_proj"],
        "q_a_proj": ["q_a_proj"],
        "q_b_proj": ["q_b_proj"],
        "kv_a_proj_with_mqa": ["kv_a_proj_with_mqa"],
        "kv_b_proj": ["kv_b_proj"],
        "gate_up_proj": ["gate_proj", "up_proj"],
    }
# ... omitted for brevity ...
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "") -> None:
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        self.config = config
        self.quant_config = quant_config

        self.model = SarvamMLAModel(
            vllm_config=vllm_config,
            prefix=maybe_prefix(prefix, "model"),
        )

        self.tie_word_embeddings = getattr(config, "tie_word_embeddings", False)
        if get_pp_group().is_last_rank:
            if self.tie_word_embeddings:
                self.lm_head = self.model.embed_tokens
            else:
                self.lm_head = ParallelLMHead(
                    config.vocab_size,
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
        logits = self.logits_processor(self.lm_head, hidden_states)
        return logits
# ... omitted for brevity ...
    def load_weights(
        self,
        weights: Iterable[tuple[str, torch.Tensor]],
    ) -> set[str]:
        loader = AutoWeightsLoader(
            self,
            skip_prefixes=(["lm_head."] if self.tie_word_embeddings else None),
        )
        return loader.load_weights(weights)
```
**EN:** Defines `SarvamMLAForCausalLM`, a top-level language-model wrapper that exposes logits and weight-loading behavior. It inherits from nn.Module, SupportsPP, SupportsLoRA. Key methods such as `__init__`, `embed_input_ids`, `forward`, `compute_logits`, `load_weights` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `SarvamMLAForCausalLM`，它是一个暴露 logits 计算与权重加载行为的顶层语言模型封装类。 它继承自 nn.Module、SupportsPP、SupportsLoRA。 `__init__`, `embed_input_ids`, `forward`, `compute_logits`, `load_weights` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `SarvamMoEForCausalLM` (lines 771-775)
```python
class SarvamMoEForCausalLM(BailingMoeForCausalLM):
    """Same as BailingMoeForCausalLM, but normalizes gate expert_bias pre-load."""

    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        return super().load_weights(_normalized_weights(weights))
```
**EN:** Defines `SarvamMoEForCausalLM`, a top-level language-model wrapper that exposes logits and weight-loading behavior. It inherits from BailingMoeForCausalLM. Key methods such as `load_weights` show where construction, forward execution, or weight adaptation happens. Docstring hint: "Same as BailingMoeForCausalLM, but normalizes gate expert_bias pre-load."
**CN:** 定义 `SarvamMoEForCausalLM`，它是一个暴露 logits 计算与权重加载行为的顶层语言模型封装类。 它继承自 BailingMoeForCausalLM。 `load_weights` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“Same as BailingMoeForCausalLM, but normalizes gate expert_bias pre-load。”

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
- **Standard library**: __future__, math, collections.abc, itertools
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.config, vllm.distributed, vllm.model_executor.layers.activation, vllm.model_executor.layers.fused_moe, vllm.model_executor.layers.layernorm, vllm.model_executor.layers.linear, vllm.model_executor.layers.logits_processor, vllm.model_executor.layers.mla
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .bailing_moe, .interfaces, .utils
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
