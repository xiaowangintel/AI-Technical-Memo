# llama4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/llama4.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Llama4 model components and runtime adapter for vLLM inference. The module docstring summarizes it as: "Inference-only LLaMA model compatible with HuggingFace weights." / 实现 Llama4 在 vLLM 推理中的模型组件与运行时适配器。 模块文档字符串还将其概括为：“Inference-only LLaMA model compatible with HuggingFace weights。”

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-68)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
#
# Copyright 2025 the LLAMA4, Meta Inc., vLLM, and HuggingFace Inc. team.
# All rights reserved.
#
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
    maybe_remap_kv_scale_name,
)
from vllm.model_executor.models.interfaces import MixtureOfExperts
from vllm.model_executor.models.utils import sequence_parallel_chunk
from vllm.platforms import current_platform
from vllm.utils.torch_utils import is_torch_equal_or_newer

from .llama import LlamaForCausalLM, LlamaMLP, LlamaModel
from .utils import (
    AutoWeightsLoader,
    PPMissingLayer,
    extract_layer_index,
    fast_topk,
    is_pp_missing_parameter,
)
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, transformers supply framework primitives, while internal modules like vllm.compilation.decorators, vllm.config, vllm.distributed, vllm.logger connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, transformers 这样的外部依赖提供基础框架能力，而 vllm.compilation.decorators, vllm.config, vllm.distributed, vllm.logger 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Top-level mapping `logger` (lines 69-69)
```python
logger = init_logger(__name__)
```
**EN:** This assignment initializes the module-level logger used for diagnostics, backend selection messages, and compatibility warnings.
**CN:** 这个赋值语句初始化了模块级日志记录器，用于输出诊断信息、后端选择消息和兼容性警告。

### Class `Llama4MoE` (lines 72-165)
```python
class Llama4MoE(nn.Module):
    @staticmethod
    def custom_routing_function(
        hidden_states: torch.Tensor,
        gating_output: torch.Tensor,
        topk: int,
        renormalize: bool,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        router_scores, router_indices = fast_topk(gating_output, topk, dim=-1)
        # pseudo-standard is that the router scores are floats
        router_scores = torch.sigmoid(router_scores.float())
        return (router_scores, router_indices.to(torch.int32))
# ... omitted for brevity ...
    def __init__(self, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config = vllm_config.model_config.hf_config
        parallel_config = vllm_config.parallel_config
        quant_config = vllm_config.quant_config

        self.tp_size = get_tensor_model_parallel_world_size()
        self.top_k = config.num_experts_per_tok
        self.is_sequence_parallel = parallel_config.use_sequence_parallel_moe
        self.ep_group = get_ep_group().device_group
        self.ep_rank = get_ep_group().rank_in_group
        self.ep_size = self.ep_group.size()

        intermediate_size_moe = config.intermediate_size
        self.router = ReplicatedLinear(
            config.hidden_size,
            config.num_local_experts,
            bias=False,
# ... omitted for brevity ...
    def forward(self, hidden_states):
        num_tokens = hidden_states.shape[0]
        if self.is_sequence_parallel:
            hidden_states = sequence_parallel_chunk(hidden_states)

        router_logits, _ = self.router(hidden_states)

        experts_out = self.experts(
            hidden_states=hidden_states,
            router_logits=router_logits,
        )

        if self.is_sequence_parallel:
            experts_out = tensor_model_parallel_all_gather(experts_out, 0)
            experts_out = experts_out[:num_tokens]

        return experts_out
```
**EN:** Defines `Llama4MoE`, a supporting module used by the surrounding model implementation. It inherits from nn.Module. Key methods such as `custom_routing_function`, `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Llama4MoE`，它是一个被周边模型实现复用的支撑模块。 它继承自 nn.Module。 `custom_routing_function`, `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Llama4Attention` (lines 168-311)
```python
class Llama4Attention(nn.Module):
    def __init__(
        self,
        config: Llama4TextConfig,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        max_position_embeddings: int = 8192,
        quant_config: QuantizationConfig | None = None,
        bias: bool = False,
        bias_o_proj: bool = False,
        cache_config: CacheConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.layer_idx = extract_layer_index(prefix)
        self.hidden_size = hidden_size
        self.no_rope_layers = config.no_rope_layers
        self.nope = self.no_rope_layers[self.layer_idx] == 0
        self.use_qk_norm = config.use_qk_norm and not self.nope
# ... omitted for brevity ...
    def _get_attn_scale(self, positions: torch.Tensor) -> torch.Tensor:
        floor = torch.floor((positions + 1.0) / self.floor_scale)
        attn_scale = torch.log(floor + 1.0) * self.attn_scale + 1.0

        return attn_scale.unsqueeze(-1)
# ... omitted for brevity ...
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)

        if self.rotary_emb is not None:
            q, k = self.rotary_emb(positions, q, k)

        if self.qk_norm is not None:
            # Normalization is applied on the head_dim dimension. The rest of
            # the dimensions are collapsed into a single dimension to support
            # custom rms_norm cuda kernel.
            q = q.reshape(-1, self.head_dim)
            q = self.qk_norm(q.float()).reshape(-1, self.q_size).to(q.dtype)
            k = k.reshape(-1, self.head_dim)
            k = self.qk_norm(k.float()).reshape(-1, self.kv_size).to(k.dtype)
```
**EN:** Defines `Llama4Attention`, a attention block that projects Q/K/V tensors and dispatches the cache-aware attention path. It inherits from nn.Module. Key methods such as `__init__`, `_get_attn_scale`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Llama4Attention`，它是一个负责投影 Q/K/V 张量并分发带缓存注意力路径的注意力模块。 它继承自 nn.Module。 `__init__`, `_get_attn_scale`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Llama4DecoderLayer` (lines 314-384)
```python
class Llama4DecoderLayer(nn.Module):
    def __init__(
        self,
        vllm_config: VllmConfig,
        prefix: str = "",
        config: Llama4TextConfig | None = None,
    ) -> None:
        super().__init__()

        config = config or vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.layer_idx = extract_layer_index(prefix)
        self.global_layer = config.no_rope_layers[self.layer_idx] == 0
        self.hidden_size = config.hidden_size
        max_position_embeddings = config.max_position_embeddings

        self.self_attn = Llama4Attention(
            config=config,
# ... omitted for brevity ...
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
        hidden_states = self.self_attn(positions=positions, hidden_states=hidden_states)

        # Fully Connected
        hidden_states, residual = self.post_attention_layernorm(hidden_states, residual)
        hidden_states = self.feed_forward(hidden_states)
        return hidden_states, residual
```
**EN:** Defines `Llama4DecoderLayer`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Llama4DecoderLayer`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Llama4Model` (lines 387-722)
```python
@support_torch_compile
class Llama4Model(LlamaModel):
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        layer_type: type[Llama4DecoderLayer] = Llama4DecoderLayer,
    ):
        self.num_experts = vllm_config.model_config.hf_config.num_local_experts
        self.n_redundant_experts = (
            vllm_config.parallel_config.eplb_config.num_redundant_experts
        )
        super().__init__(vllm_config=vllm_config, prefix=prefix, layer_type=layer_type)
# ... omitted for brevity ...
    def load_moe_expert_weights(
        self,
        name: str,
        loaded_weight: torch.Tensor,
        params_dict: dict[str, nn.Parameter],
        loaded_params: set[str],
        expert_params_mapping: list[tuple[str, str, int, str]],
        fused: bool = True,
    ) -> bool:
        """
        Load MoE expert weights.

        Args:
            name: The name of the weight to load.
            loaded_weight: The weight to load.
            params_dict: The dictionary of module parameters.
            loaded_params: The set of already loaded parameters.
            expert_params_mapping: The mapping of expert parameters. Must be
                generated by fused_moe_make_expert_params_mapping().
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        # Name mapping from the parameter name to the shard name and
        # corresponding shard id.
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            (".qkv_proj", ".q_proj", "q"),
            (".qkv_proj", ".k_proj", "k"),
            (".qkv_proj", ".v_proj", "v"),
            (".gate_up_proj", ".gate_proj", 0),
            (".gate_up_proj", ".up_proj", 1),
        ]
        # Indicate whether the expert weights are fused into a single weight
        # tensor.
        fused_experts_params = False
        # Expert parameter mapping for the case where the expert weights are
        # not fused into a single weight tensor.
        expert_params_mapping = fused_moe_make_expert_params_mapping(
            self,
            ckpt_gate_proj_name="gate_proj",
```
**EN:** Defines `Llama4Model`, a backbone model container that orchestrates embeddings, layers, and output heads. It inherits from LlamaModel. Key methods such as `__init__`, `load_moe_expert_weights`, `load_weights` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Llama4Model`，它是一个协调嵌入、层堆叠与输出头的主干模型容器。 它继承自 LlamaModel。 `__init__`, `load_moe_expert_weights`, `load_weights` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Llama4ForCausalLM` (lines 725-862)
```python
class Llama4ForCausalLM(LlamaForCausalLM, MixtureOfExperts):
    packed_modules_mapping = {
        "qkv_proj": ["q_proj", "k_proj", "v_proj"],
        "gate_up_proj": ["gate_proj", "up_proj"],
    }

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        # update temperature tuning config from generation config
        gen_config = vllm_config.model_config.try_get_generation_config()
        gen_config.update(vllm_config.model_config.override_generation_config)
        # enable temperature tuning by default when max_model_len > 32K
        default_attn_temperature_tuning = vllm_config.model_config.max_model_len > 32768
        vllm_config.model_config.hf_config.attn_temperature_tuning = gen_config.get(
            "attn_temperature_tuning", default_attn_temperature_tuning
        )

        super().__init__(
            vllm_config=vllm_config, prefix=prefix, layer_type=Llama4DecoderLayer
        )
        # Set MoE hyperparameters
        self.set_moe_parameters()
# ... omitted for brevity ...
    def set_moe_parameters(self):
        self.expert_weights = []

        self.moe_layers = []
        example_moe = None
        for layer in self.model.layers:
            if isinstance(layer, PPMissingLayer):
                continue

            assert isinstance(layer, Llama4DecoderLayer)
            if isinstance(layer.feed_forward, Llama4MoE):
                # Pick last one layer since the first ones may be dense layers.
                example_moe = layer.feed_forward
# ... omitted for brevity ...
            self.num_moe_layers = 0
            self.num_expert_groups = 0
            self.num_logical_experts = 0
# ... omitted for brevity ...
    def update_physical_experts_metadata(
        self,
        num_physical_experts: int,
        num_local_physical_experts: int,
    ) -> None:
        assert self.num_local_physical_experts == num_local_physical_experts
        self.num_physical_experts = num_physical_experts
        self.num_local_physical_experts = num_local_physical_experts
        self.num_redundant_experts = num_physical_experts - self.num_logical_experts
        for layer in self.model.layers:
            if isinstance(layer, PPMissingLayer):
                continue

            if isinstance(layer.feed_forward, Llama4MoE):
                moe = layer.feed_forward
                moe.n_local_physical_experts = num_local_physical_experts
                moe.n_physical_experts = num_physical_experts
                moe.n_redundant_experts = self.num_redundant_experts
                moe.experts.update_expert_map()
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        loader = AutoWeightsLoader(
            self,
            skip_prefixes=(["lm_head."] if self.config.tie_word_embeddings else None),
        )
        weights = [
            self.permute_qk_weight_for_rotary(name, loaded_weight)
            for name, loaded_weight in weights
        ]
        return loader.load_weights(weights)
```
**EN:** Defines `Llama4ForCausalLM`, a top-level language-model wrapper that exposes logits and weight-loading behavior. It inherits from LlamaForCausalLM, MixtureOfExperts. Key methods such as `__init__`, `set_moe_parameters`, `update_physical_experts_metadata`, `_init_model`, `load_weights` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Llama4ForCausalLM`，它是一个暴露 logits 计算与权重加载行为的顶层语言模型封装类。 它继承自 LlamaForCausalLM、MixtureOfExperts。 `__init__`, `set_moe_parameters`, `update_physical_experts_metadata`, `_init_model`, `load_weights` 等关键方法展示了构造、前向执行或权重适配发生的位置。

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
- **Standard library**: collections.abc
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, transformers
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.compilation.decorators, vllm.config, vllm.distributed, vllm.logger, vllm.model_executor.layers.attention, vllm.model_executor.layers.fused_moe, vllm.model_executor.layers.layernorm, vllm.model_executor.layers.linear
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .llama, .utils
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
