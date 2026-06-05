# step3p5.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/step3p5.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Step3p5 model components and runtime adapter for vLLM inference. The module docstring summarizes it as: "Inference-only Jurassic model." / 实现 Step3p5 在 vLLM 推理中的模型组件与运行时适配器。 模块文档字符串还将其概括为：“Inference-only Jurassic model。”

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-60)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""Inference-only Jurassic model."""

import typing
from collections.abc import Callable, Iterable
from typing import Any

import torch
from torch import nn
from torch.nn.parameter import Parameter

from vllm.compilation.decorators import support_torch_compile
from vllm.config import CacheConfig, ModelConfig, VllmConfig
from vllm.distributed import (
    get_dp_group,
    get_ep_group,
    get_pp_group,
# ... omitted for brevity ...
from vllm.model_executor.model_loader.weight_utils import default_weight_loader
from vllm.sequence import IntermediateTensors
from vllm.v1.attention.backend import AttentionType

from .interfaces import MixtureOfExperts, SupportsPP
from .utils import (
    AutoWeightsLoader,
    PPMissingLayer,
    WeightsMapper,
    extract_layer_index,
    is_pp_missing_parameter,
    make_empty_intermediate_tensors_factory,
    make_layers,
    maybe_prefix,
)
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, torch.nn.parameter supply framework primitives, while internal modules like vllm.compilation.decorators, vllm.config, vllm.distributed, vllm.logger connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, torch.nn.parameter 这样的外部依赖提供基础框架能力，而 vllm.compilation.decorators, vllm.config, vllm.distributed, vllm.logger 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Top-level mapping `logger` (lines 61-61)
```python
logger = init_logger(__name__)
```
**EN:** This assignment initializes the module-level logger used for diagnostics, backend selection messages, and compatibility warnings.
**CN:** 这个赋值语句初始化了模块级日志记录器，用于输出诊断信息、后端选择消息和兼容性警告。

### Class `FP32ReplicatedLinear` (lines 64-74)
```python
class FP32ReplicatedLinear(ReplicatedLinear):
    """
    Use FP32 for higher precision.
    """

    def forward(
        self,
        x: torch.Tensor,
    ) -> torch.Tensor | tuple[torch.Tensor, Parameter | None]:
        assert self.params_dtype == torch.float32
        return super().forward(x.to(torch.float32))
```
**EN:** Defines `FP32ReplicatedLinear`, a supporting module used by the surrounding model implementation. It inherits from ReplicatedLinear. Key methods such as `forward` show where construction, forward execution, or weight adaptation happens. Docstring hint: "Use FP32 for higher precision."
**CN:** 定义 `FP32ReplicatedLinear`，它是一个被周边模型实现复用的支撑模块。 它继承自 ReplicatedLinear。 `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“Use FP32 for higher precision。”

### Class `Step3p5MLP` (lines 77-126)
```python
class Step3p5MLP(nn.Module):
    def __init__(
        self,
        config: ModelConfig,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        quant_config: QuantizationConfig | None = None,
        reduce_results: bool = True,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.gate_up_proj = MergedColumnParallelLinear(
            hidden_size,
            [intermediate_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.gate_up_proj",
        )
        self.down_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            reduce_results=reduce_results,
            prefix=f"{prefix}.down_proj",
        )

        if hidden_act != "silu":
            raise ValueError(
                f"Unsupported activation: {hidden_act}. Only silu is supported for now."
            )
        self.act_fn = SiluAndMul()
        self.prefix = prefix
        self.hidden_size = hidden_size
        self.limit = None
        layer_idx = extract_layer_index(prefix)
        if (
            config.swiglu_limits_shared
            and config.swiglu_limits_shared[layer_idx] is not None
            and config.swiglu_limits_shared[layer_idx] != 0
        ):
            self.limit = config.swiglu_limits_shared[layer_idx]
            self.act_fn = SwigluStepAndMul(limit=self.limit)

    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        gate_up, _ = self.gate_up_proj(hidden_states)
        intermediate_act = self.act_fn(gate_up)
        output, _ = self.down_proj(intermediate_act)
        return output
```
**EN:** Defines `Step3p5MLP`, a feed-forward block that expands hidden states and projects them back. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Step3p5MLP`，它是一个扩展隐藏状态并再投影回主维度的前馈模块。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Step3p5Attention` (lines 129-289)
```python
class Step3p5Attention(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        max_position: int = 4096 * 32,
        head_dim: int | None = None,
        rms_norm_eps: float = 1e-06,
        qkv_bias: bool = False,
        rope_theta: float | list[float] | None = 10000,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        rope_scaling: dict[str, Any] | None = None,
        prefix: str = "",
        attn_type: str = AttentionType.DECODER,
        # Step3p5 specific args
        sliding_window: int | None = None,
        use_head_wise_attn_gate: bool = False,
        layer_types: list = None,
# ... omitted for brevity ...
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        # Add qk-norm inline similar to Qwen3 MOE attention
        q_by_head = q.view(*q.shape[:-1], q.shape[-1] // self.head_dim, self.head_dim)
        q_by_head = self.q_norm(q_by_head.contiguous())
        q = q_by_head.view(q.shape)

        k_by_head = k.view(*k.shape[:-1], k.shape[-1] // self.head_dim, self.head_dim)
        k_by_head = self.k_norm(k_by_head.contiguous())
        k = k_by_head.view(k.shape)
        if self.use_rope:
            q, k = self.rotary_emb(positions, q, k)
        attn_output = self.attn(q, k, v)
        if self.use_head_wise_attn_gate:
```
**EN:** Defines `Step3p5Attention`, a attention block that projects Q/K/V tensors and dispatches the cache-aware attention path. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Step3p5Attention`，它是一个负责投影 Q/K/V 张量并分发带缓存注意力路径的注意力模块。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `FusedMoEBlock` (lines 292-411)
```python
class FusedMoEBlock(nn.Module):
    def __init__(
        self,
        vllm_config: VllmConfig,
        prefix: str = "",
    ):
        super().__init__()

        self.tp_size = get_tensor_model_parallel_world_size()
        self.layer_idx = extract_layer_index(prefix)

        self.ep_size = get_ep_group().device_group.size()
        self.ep_rank = get_ep_group().device_group.rank()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        parallel_config = vllm_config.parallel_config

        self.hidden_size = config.hidden_size
        self.enable_eplb = parallel_config.enable_eplb
        self.n_routed_experts = config.moe_num_experts
# ... omitted for brevity ...
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        num_tokens, hidden_dim = hidden_states.shape
        hidden_states = hidden_states.view(-1, hidden_dim)

        if self.experts.is_internal_router:
            final_hidden_states = self.experts(
                hidden_states=hidden_states, router_logits=hidden_states
            )
        else:
            # TODO(bnell): this gate could be moved into the FusedMoE?
            router_logits, _ = self.gate(hidden_states)
            final_hidden_states = self.experts(
                hidden_states=hidden_states, router_logits=router_logits
            )

        return final_hidden_states.view(num_tokens, hidden_dim)
```
**EN:** Defines `FusedMoEBlock`, a supporting module used by the surrounding model implementation. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `FusedMoEBlock`，它是一个被周边模型实现复用的支撑模块。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Step3p5DecoderLayer` (lines 414-543)
```python
class Step3p5DecoderLayer(nn.Module):
    def __init__(
        self,
        vllm_config: VllmConfig,
        prefix: str = "",
    ) -> None:
        super().__init__()
        config = vllm_config.model_config.hf_config
        self.hidden_size = config.hidden_size
        layer_idx = extract_layer_index(prefix)
        self.layer_idx = layer_idx
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config
        if cache_config is not None:
            cache_config.sliding_window = None
        if config.att_impl_type == "GQA":
            num_attention_heads = None
            num_attention_groups = None
            head_dim = None
            if (
# ... omitted for brevity ...
    def add_and_maybe_inplace_all_reduce(
        self, in1: torch.Tensor, in2: torch.Tensor
    ) -> torch.Tensor:
        if not self.use_fused_all_reduce:
            return in1 + in2
        return self.tp_group.all_reduce(in1 + in2)
# ... omitted for brevity ...
    def forward(
        self, positions: torch.Tensor, hidden_states: torch.Tensor
    ) -> torch.Tensor:
        residual = hidden_states
        hidden_states = self.input_layernorm(hidden_states)

        hidden_states = self.self_attn(
            positions=positions,
            hidden_states=hidden_states,
        )
        hidden_states += residual
        residual = hidden_states
        hidden_states = self.post_attention_layernorm(hidden_states)

        if self.use_moe:
            ffn_output = self.moe(hidden_states)
        else:
            ffn_output = self.mlp(hidden_states)
        hidden_states = ffn_output + residual
```
**EN:** Defines `Step3p5DecoderLayer`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from nn.Module. Key methods such as `__init__`, `add_and_maybe_inplace_all_reduce`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Step3p5DecoderLayer`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 nn.Module。 `__init__`, `add_and_maybe_inplace_all_reduce`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Step3p5Model` (lines 546-816)
```python
@support_torch_compile
class Step3p5Model(nn.Module):
    def __init__(self, vllm_config: VllmConfig, prefix: str = "") -> None:
        super().__init__()

        self.vllm_config = vllm_config
        config = vllm_config.model_config.hf_config
        self.vocab_size = config.vocab_size
        self.config = config

        self.moe_num_experts = config.moe_num_experts

        if get_pp_group().is_first_rank or (
            config.tie_word_embeddings and get_pp_group().is_last_rank
        ):
            self.embed_tokens = VocabParallelEmbedding(
                self.vocab_size,
                config.hidden_size,
            )
        else:
            self.embed_tokens = PPMissingLayer()
# ... omitted for brevity ...
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.embed_tokens(input_ids)
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor:
        if get_pp_group().is_first_rank:
            if inputs_embeds is not None:
                hidden_states = inputs_embeds
            else:
                hidden_states = self.embed_input_ids(input_ids)
        else:
            assert intermediate_tensors is not None
            hidden_states = intermediate_tensors["hidden_states"]
        for i in range(self.start_layer, self.end_layer):
            layer = self.layers[i]
            hidden_states = layer(positions, hidden_states)

# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        config = self.config
        assert config.num_attention_groups > 1, "Only support GQA"
        qkv_params_mapping = []
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
        ]

        params_dict = dict(self.named_parameters())
        loaded_params: set[str] = set()
        base_layer = (
            "base_layer." if any(".base_layer." in name for name in params_dict) else ""
        )
```
**EN:** Defines `Step3p5Model`, a backbone model container that orchestrates embeddings, layers, and output heads. It inherits from nn.Module. Key methods such as `__init__`, `embed_input_ids`, `forward`, `load_weights` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Step3p5Model`，它是一个协调嵌入、层堆叠与输出头的主干模型容器。 它继承自 nn.Module。 `__init__`, `embed_input_ids`, `forward`, `load_weights` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Step3p5ForCausalLM` (lines 819-933)
```python
class Step3p5ForCausalLM(nn.Module, SupportsPP, MixtureOfExperts):
    # Required so quantization exclude lists match fused module prefixes.
    packed_modules_mapping = {
        "qkv_proj": ["q_proj", "k_proj", "v_proj"],
        "gate_up_proj": ["gate_proj", "up_proj"],
    }

    hf_to_vllm_mapper = WeightsMapper(
        orig_to_new_substr={".share_expert.": ".moe.share_expert."}
# ... omitted for brevity ...
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
    ):
        super().__init__()
        config = vllm_config.model_config.hf_config
        self.model = Step3p5Model(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "model")
        )
        if get_pp_group().is_last_rank:
            self.lm_head = ParallelLMHead(
                config.vocab_size,
                config.hidden_size,
                quant_config=vllm_config.quant_config,
                prefix=maybe_prefix(prefix, "lm_head"),
            )
            self.logits_processor = LogitsProcessor(config.vocab_size)
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
    ):
        hidden_states = self.model(
            input_ids, positions, intermediate_tensors, inputs_embeds
        )
        return hidden_states
# ... omitted for brevity ...
    def compute_logits(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states = self.model.norm(hidden_states)
        logits = self.logits_processor(self.lm_head, hidden_states)
        return logits
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        loader = AutoWeightsLoader(self)
        return loader.load_weights(weights, mapper=self.hf_to_vllm_mapper)
```
**EN:** Defines `Step3p5ForCausalLM`, a top-level language-model wrapper that exposes logits and weight-loading behavior. It inherits from nn.Module, SupportsPP, MixtureOfExperts. Key methods such as `__init__`, `forward`, `compute_logits`, `embed_input_ids`, `set_eplb_state` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Step3p5ForCausalLM`，它是一个暴露 logits 计算与权重加载行为的顶层语言模型封装类。 它继承自 nn.Module、SupportsPP、MixtureOfExperts。 `__init__`, `forward`, `compute_logits`, `embed_input_ids`, `set_eplb_state` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Function `get_spec_layer_idx_from_weight_name` (lines 936-948)
```python
def get_spec_layer_idx_from_weight_name(
    config: ModelConfig, weight_name: str
) -> int | None:
    if hasattr(config, "num_nextn_predict_layers") and (
        config.num_nextn_predict_layers > 0
    ):
        layer_idx = config.num_hidden_layers
        for i in range(config.num_nextn_predict_layers):
            if weight_name.startswith(
                f"layers.{layer_idx + i}."  # Step3p5Model
            ) or weight_name.startswith(f"model.layers.{layer_idx + i}."):  # Step3p5MTP
                return layer_idx + i
    return None
```
**EN:** The function `get_spec_layer_idx_from_weight_name` helps provide a reusable helper for the surrounding model code. Its main inputs are `config`, `weight_name`.
**CN:** 函数 `get_spec_layer_idx_from_weight_name` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `config`、`weight_name`。

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
- **Standard library**: typing, collections.abc
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, torch.nn.parameter
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.compilation.decorators, vllm.config, vllm.distributed, vllm.logger, vllm.model_executor.layers.activation, vllm.model_executor.layers.attention, vllm.model_executor.layers.fused_moe, vllm.model_executor.layers.layernorm
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .interfaces, .utils
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
