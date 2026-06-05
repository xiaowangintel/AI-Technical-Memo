# olmo_hybrid.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/olmo_hybrid.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Olmo Hybrid model components and runtime adapter for vLLM inference. The module docstring summarizes it as: "Inference-only OLMo Hybrid model compatible with HuggingFace weights." / 实现 Olmo Hybrid 在 vLLM 推理中的模型组件与运行时适配器。 模块文档字符串还将其概括为：“Inference-only OLMo Hybrid model compatible with HuggingFace weights。”

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-106)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

# Adapted from:
# https://github.com/huggingface/transformers/blob/main/src/transformers/models/olmo_hybrid/modeling_olmo_hybrid.py
# Copyright 2026 The vLLM team.
#
# This code combines OLMo2/OLMo3 attention with Gated DeltaNet linear attention
# for the OLMo Hybrid architecture.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# ... omitted for brevity ...
from vllm.triton_utils.allocation import set_triton_allocator
from vllm.utils.torch_utils import direct_register_custom_op
from vllm.v1.attention.backend import AttentionMetadata
from vllm.v1.attention.backends.gdn_attn import GDNAttentionMetadata
from vllm.v1.attention.backends.registry import MambaAttentionBackendEnum

from .interfaces import HasInnerState, IsHybrid, SupportsLoRA, SupportsPP
from .utils import (
    AutoWeightsLoader,
    extract_layer_index,
    is_pp_missing_parameter,
    make_empty_intermediate_tensors_factory,
    make_layers,
    maybe_prefix,
)
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, einops, transformers.activations supply framework primitives, while internal modules like vllm.compilation.decorators, vllm.config, vllm.distributed, vllm.distributed.utils connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, einops, transformers.activations 这样的外部依赖提供基础框架能力，而 vllm.compilation.decorators, vllm.config, vllm.distributed, vllm.distributed.utils 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Top-level mapping `logger` (lines 107-107)
```python
logger = init_logger(__name__)
```
**EN:** This assignment initializes the module-level logger used for diagnostics, backend selection messages, and compatibility warnings.
**CN:** 这个赋值语句初始化了模块级日志记录器，用于输出诊断信息、后端选择消息和兼容性警告。

### Function `_make_fused_conv1d_weight_loader` (lines 110-128)
```python
def _make_fused_conv1d_weight_loader(dims, tp_size, tp_rank):
    """Weight loader for loading separate HF conv weights into a fused conv1d.

    dims: list of original (un-sharded) dims per section,
          e.g. [key_dim, key_dim, value_dim]
    """
    sharded_dims = [d // tp_size for d in dims]

    def weight_loader(param, loaded_weight, loaded_shard_id=None):
        if loaded_weight.dim() == 2:
            loaded_weight = loaded_weight.unsqueeze(1)
        dim = dims[loaded_shard_id]
        shard_size = dim // tp_size
        tp_start = tp_rank * shard_size
        sharded_weight = loaded_weight[tp_start : tp_start + shard_size]
        offset = sum(sharded_dims[:loaded_shard_id])
        param.data[offset : offset + shard_size].copy_(sharded_weight)

    return weight_loader
```
**EN:** The function `_make_fused_conv1d_weight_loader` helps translate or load checkpoint tensors into vLLM modules. Its main inputs are `dims`, `tp_size`, `tp_rank`. Docstring hint: "Weight loader for loading separate HF conv weights into a fused conv1d."
**CN:** 函数 `_make_fused_conv1d_weight_loader` 用于将检查点张量转换或加载到 vLLM 模块中。 它的主要输入包括 `dims`、`tp_size`、`tp_rank`。 文档提示：“Weight loader for loading separate HF conv weights into a fused conv1d。”

### Class `OlmoHybridGatedDeltaNet` (lines 131-603)
```python
class OlmoHybridGatedDeltaNet(nn.Module, MambaBase):
    """
    Gated DeltaNet linear attention layer for OLMo Hybrid.

    This implements the linear attention mechanism that replaces sliding window
    attention in the hybrid architecture.
    """

    @property
# ... omitted for brevity ...
    def __init__(
        self,
        config,
        model_config: ModelConfig | None = None,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        speculative_config: SpeculativeConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.tp_size = get_tensor_model_parallel_world_size()
        self.tp_rank = get_tensor_model_parallel_rank()
        self.hidden_size = config.hidden_size
        self.num_v_heads = config.linear_num_value_heads
        self.num_k_heads = config.linear_num_key_heads
        self.head_k_dim = config.linear_key_head_dim
        self.head_v_dim = config.linear_value_head_dim
        self.key_dim = self.head_k_dim * self.num_k_heads
        self.value_dim = self.head_v_dim * self.num_v_heads
# ... omitted for brevity ...
    def forward(
        self,
        hidden_states: torch.Tensor,
        output: torch.Tensor,
    ):
# ... omitted for brevity ...
        num_tokens = hidden_states.size(0)

        # ============================================================
        # Part 1: Input Projection (2 fused matmuls instead of 6)
        # ============================================================
        projected_qkvg, _ = self.in_proj_qkvg(hidden_states)
        conv_dim_sharded = (self.key_dim * 2 + self.value_dim) // self.tp_size
        mixed_qkv = projected_qkvg[..., :conv_dim_sharded]
        gate = projected_qkvg[..., conv_dim_sharded:]

        b, _ = self.b_proj(hidden_states)
        a, _ = self.a_proj(hidden_states)

        # ============================================================
# ... omitted for brevity ...
    def _forward_core(
        self,
        mixed_qkv: torch.Tensor,
        b: torch.Tensor,
        a: torch.Tensor,
        core_attn_out: torch.Tensor,
    ):
        """
        Core attention computation (called by custom op).
        """
        forward_context = get_forward_context()
        attn_metadata: AttentionMetadata = forward_context.attn_metadata

        if attn_metadata is None:
            # V1 profile run
            return

        assert isinstance(attn_metadata, dict)
        attn_metadata = attn_metadata[self.prefix]
```
**EN:** Defines `OlmoHybridGatedDeltaNet`, a supporting module used by the surrounding model implementation. It inherits from nn.Module, MambaBase. Key methods such as `mamba_type`, `get_state_dtype`, `get_state_shape`, `__init__`, `rearrange_mixed_qkv` show where construction, forward execution, or weight adaptation happens. Docstring hint: "Gated DeltaNet linear attention layer for OLMo Hybrid."
**CN:** 定义 `OlmoHybridGatedDeltaNet`，它是一个被周边模型实现复用的支撑模块。 它继承自 nn.Module、MambaBase。 `mamba_type`, `get_state_dtype`, `get_state_shape`, `__init__`, `rearrange_mixed_qkv` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“Gated DeltaNet linear attention layer for OLMo Hybrid。”

### Class `OlmoHybridAttention` (lines 606-714)
```python
class OlmoHybridAttention(nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        self.config = vllm_config.model_config.hf_config

        hidden_size = self.config.hidden_size
        self.tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = self.config.num_attention_heads

        assert hidden_size % self.total_num_heads == 0
        assert self.total_num_heads % self.tp_size == 0

        self.num_heads = self.total_num_heads // self.tp_size
        self.total_num_kv_heads = (
            self.config.num_key_value_heads or self.total_num_heads
        )
        if self.total_num_kv_heads >= self.tp_size:
            assert self.total_num_kv_heads % self.tp_size == 0
        else:
            assert self.tp_size % self.total_num_kv_heads == 0
# ... omitted for brevity ...
    def _apply_qk_norm(
        self, q: torch.Tensor, k: torch.Tensor
    ) -> tuple[torch.Tensor, torch.Tensor]:
        if self.tp_size > 1:
            q = tensor_model_parallel_all_gather(q.contiguous())
            k = tensor_model_parallel_all_gather(k.contiguous())
        q = self.q_norm(q)
        k = self.k_norm(k)
        if self.tp_size > 1:
            splitter = partial(split_tensor_along_last_dim, num_partitions=self.tp_size)
            q = splitter(q)[self.tp_rank]
            k = splitter(k)[self.tp_rank]
        return q, k
# ... omitted for brevity ...
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        q, k = self._apply_qk_norm(q, k)
        if self._use_rope:
            q, k = self.rotary_emb(positions, q, k)
        attn_output = self.attn(q, k, v)
        output, _ = self.o_proj(attn_output)
        return output
```
**EN:** Defines `OlmoHybridAttention`, a attention block that projects Q/K/V tensors and dispatches the cache-aware attention path. It inherits from nn.Module. Key methods such as `__init__`, `_apply_qk_norm`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `OlmoHybridAttention`，它是一个负责投影 Q/K/V 张量并分发带缓存注意力路径的注意力模块。 它继承自 nn.Module。 `__init__`, `_apply_qk_norm`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `OlmoHybridMLP` (lines 717-746)
```python
class OlmoHybridMLP(nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        hidden_size = config.hidden_size
        intermediate_size = config.intermediate_size

        self.gate_up_proj = MergedColumnParallelLinear(
            hidden_size,
            [intermediate_size] * 2,
            bias=False,
            quant_config=vllm_config.quant_config,
            prefix=f"{prefix}.gate_up_proj",
        )

        self.act_fn = SiluAndMul()

        self.down_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=False,
            quant_config=vllm_config.quant_config,
            prefix=f"{prefix}.down_proj",
        )

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        gate_up, _ = self.gate_up_proj(x)
        x = self.act_fn(gate_up)
        x, _ = self.down_proj(x)
        return x
```
**EN:** Defines `OlmoHybridMLP`, a feed-forward block that expands hidden states and projects them back. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `OlmoHybridMLP`，它是一个扩展隐藏状态并再投影回主维度的前馈模块。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `OlmoHybridDecoderLayer` (lines 749-829)
```python
class OlmoHybridDecoderLayer(nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "") -> None:
        super().__init__()
        config = vllm_config.model_config.hf_config
        model_config = vllm_config.model_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config
        speculative_config = vllm_config.speculative_config

        layer_idx = extract_layer_index(prefix)
        self.layer_type = config.layer_types[layer_idx]
        self.layer_idx = layer_idx

        if self.layer_type == "linear_attention":
            self.linear_attn = OlmoHybridGatedDeltaNet(
                config,
                model_config=model_config,
                cache_config=cache_config,
                quant_config=quant_config,
                speculative_config=speculative_config,
# ... omitted for brevity ...
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        if self.layer_type == "linear_attention":
            residual = hidden_states
            hidden_states = self.input_layernorm(hidden_states)

            attn_output = torch.empty_like(hidden_states)
            self.linear_attn(
                hidden_states=hidden_states,
                output=attn_output,
            )
            hidden_states = residual + attn_output

            residual = hidden_states
            hidden_states = self.post_attention_layernorm(hidden_states)
            hidden_states = self.mlp(hidden_states)
```
**EN:** Defines `OlmoHybridDecoderLayer`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `OlmoHybridDecoderLayer`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `OlmoHybridModel` (lines 832-963)
```python
@support_torch_compile
class OlmoHybridModel(nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        self.config = vllm_config.model_config.hf_config

        self.embed_tokens = VocabParallelEmbedding(
            self.config.vocab_size,
            self.config.hidden_size,
            prefix=f"{prefix}.embed_tokens",
        )

        self.start_layer, self.end_layer, self.layers = make_layers(
            self.config.num_hidden_layers,
            lambda prefix: OlmoHybridDecoderLayer(
                vllm_config=vllm_config, prefix=prefix
            ),
            prefix=f"{prefix}.layers",
        )

        self.norm = RMSNorm(
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
                hidden_states = self.embed_tokens(input_ids)
        else:
            assert intermediate_tensors is not None
            hidden_states = intermediate_tensors["hidden_states"]
            assert isinstance(hidden_states, torch.Tensor)

        for layer in islice(self.layers, self.start_layer, self.end_layer):
            hidden_states = layer(positions, hidden_states)
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        stacked_params_mapping = [
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
        ]

        linear_attn_stacked_params_mapping = [
            ("in_proj_qkvg", "q_proj", 0),
            ("in_proj_qkvg", "k_proj", 1),
            ("in_proj_qkvg", "v_proj", 2),
            ("in_proj_qkvg", "g_proj", 3),
            ("conv1d", "q_conv1d", 0),
            ("conv1d", "k_conv1d", 1),
            ("conv1d", "v_conv1d", 2),
        ]
```
**EN:** Defines `OlmoHybridModel`, a backbone model container that orchestrates embeddings, layers, and output heads. It inherits from nn.Module. Key methods such as `__init__`, `embed_input_ids`, `forward`, `load_weights` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `OlmoHybridModel`，它是一个协调嵌入、层堆叠与输出头的主干模型容器。 它继承自 nn.Module。 `__init__`, `embed_input_ids`, `forward`, `load_weights` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `OlmoHybridForCausalLM` (lines 966-1070)
```python
class OlmoHybridForCausalLM(
    nn.Module, HasInnerState, SupportsPP, SupportsLoRA, IsHybrid
):
    packed_modules_mapping = {
        "qkv_proj": ["q_proj", "k_proj", "v_proj"],
        "gate_up_proj": ["gate_proj", "up_proj"],
        "in_proj_qkvg": ["q_proj", "k_proj", "v_proj", "g_proj"],
    }

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        self.config = config
        self.vllm_config = vllm_config
        self.model_config = vllm_config.model_config

        self.model = OlmoHybridModel(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "model")
        )

        if config.tie_word_embeddings:
            self.lm_head = self.model.embed_tokens
        else:
            self.lm_head = ParallelLMHead(
                config.vocab_size,
                config.hidden_size,
                quant_config=vllm_config.quant_config,
                prefix=maybe_prefix(prefix, "lm_head"),
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor | IntermediateTensors:
        hidden_states = self.model(
            input_ids=input_ids,
            positions=positions,
            intermediate_tensors=intermediate_tensors,
            inputs_embeds=inputs_embeds,
        )
        return hidden_states
# ... omitted for brevity ...
    def compute_logits(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor | None:
        logits = self.logits_processor(self.lm_head, hidden_states)
        return logits
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]):
        loader = AutoWeightsLoader(
            self,
            skip_prefixes=(
                ["lm_head.weight"] if self.config.tie_word_embeddings else None
            ),
        )
        return loader.load_weights(weights)
```
**EN:** Defines `OlmoHybridForCausalLM`, a top-level language-model wrapper that exposes logits and weight-loading behavior. It inherits from nn.Module, HasInnerState, SupportsPP. Key methods such as `__init__`, `embed_input_ids`, `forward`, `compute_logits`, `get_mamba_state_dtype_from_config` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `OlmoHybridForCausalLM`，它是一个暴露 logits 计算与权重加载行为的顶层语言模型封装类。 它继承自 nn.Module、HasInnerState、SupportsPP。 `__init__`, `embed_input_ids`, `forward`, `compute_logits`, `get_mamba_state_dtype_from_config` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Function `olmo_hybrid_gdn_full_forward` (lines 1073-1089)
```python
def olmo_hybrid_gdn_full_forward(
    hidden_states: torch.Tensor,
    output: torch.Tensor,
    layer_name: str,
) -> None:
    """Full linear attention forward wrapped as a custom op.

    Prevents inductor from compiling the projections around the GDN core,
    which would introduce numerical divergence that compounds through
    the recurrent state.
    """
    forward_context: ForwardContext = get_forward_context()
    self = forward_context.no_compile_layers[layer_name]
    self._full_forward(
        hidden_states=hidden_states,
        output=output,
    )
```
**EN:** The function `olmo_hybrid_gdn_full_forward` helps run a forward computation step for this backend. Its main inputs are `hidden_states`, `output`, `layer_name`. Docstring hint: "Full linear attention forward wrapped as a custom op."
**CN:** 函数 `olmo_hybrid_gdn_full_forward` 用于为该后端执行一次前向计算。 它的主要输入包括 `hidden_states`、`output`、`layer_name`。 文档提示：“Full linear attention forward wrapped as a custom op。”

### Function `fused_olmo_hybrid_gdn_gating` (lines 1151-1180)
```python
def fused_olmo_hybrid_gdn_gating(
    A_log: torch.Tensor,
    a: torch.Tensor,
    b: torch.Tensor,
    dt_bias: torch.Tensor,
    allow_neg_eigval: bool = False,
    beta: float = 1.0,
    threshold: float = 20.0,
) -> tuple[torch.Tensor, torch.Tensor]:
    batch, num_heads = a.shape
    seq_len = 1
    grid = (batch, seq_len, triton.cdiv(num_heads, 8))
    g = torch.empty(1, batch, num_heads, dtype=torch.float32, device=a.device)
    beta_output = torch.empty(1, batch, num_heads, dtype=torch.float32, device=b.device)
    fused_olmo_hybrid_gdn_gating_kernel[grid](
        g,
        beta_output,
        A_log,
        a,
        b,
        dt_bias,
        seq_len,
        allow_neg_eigval,
        num_heads,
        beta,
        threshold,
        8,
        num_warps=1,
    )
    return g, beta_output
```
**EN:** The function `fused_olmo_hybrid_gdn_gating` helps provide a reusable helper for the surrounding model code. Its main inputs are `A_log`, `a`, `b`, `dt_bias`, `allow_neg_eigval`, `beta`.
**CN:** 函数 `fused_olmo_hybrid_gdn_gating` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `A_log`、`a`、`b`、`dt_bias`、`allow_neg_eigval`、`beta`。

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
- **Standard library**: collections.abc, functools, itertools
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, einops, transformers.activations
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.compilation.decorators, vllm.config, vllm.distributed, vllm.distributed.utils, vllm.forward_context, vllm.logger, vllm.model_executor.layers.activation, vllm.model_executor.layers.attention
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .interfaces, .utils
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
