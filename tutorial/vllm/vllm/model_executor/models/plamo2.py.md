# plamo2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/plamo2.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Plamo2 model components and runtime adapter for vLLM inference. The module docstring summarizes it as: "Inference-only PLaMo2 model." / 实现 Plamo2 在 vLLM 推理中的模型组件与运行时适配器。 模块文档字符串还将其概括为：“Inference-only PLaMo2 model。”

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-77)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""Inference-only PLaMo2 model."""

from collections.abc import Iterable
from itertools import islice
from typing import TYPE_CHECKING

import torch
from torch import nn
from transformers import PretrainedConfig

from vllm.compilation.decorators import support_torch_compile
from vllm.config import VllmConfig, get_current_vllm_config
from vllm.distributed import divide, get_tensor_model_parallel_world_size
from vllm.distributed.parallel_state import get_pp_group
from vllm.forward_context import ForwardContext, get_forward_context
from vllm.model_executor.custom_op import PluggableLayer
# ... omitted for brevity ...
from vllm.model_executor.models.utils import (
    AutoWeightsLoader,
    is_pp_missing_parameter,
    make_empty_intermediate_tensors_factory,
    make_layers,
    maybe_prefix,
)
from vllm.model_executor.utils import set_weight_attrs
from vllm.platforms import current_platform
from vllm.sequence import IntermediateTensors
from vllm.utils.torch_utils import direct_register_custom_op
from vllm.v1.attention.backend import AttentionMetadata
from vllm.v1.attention.backends.mamba2_attn import Mamba2AttentionMetadata
from vllm.v1.attention.backends.registry import MambaAttentionBackendEnum

# Only used for type hinting.
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, transformers supply framework primitives, while internal modules like vllm.compilation.decorators, vllm.config, vllm.distributed, vllm.distributed.parallel_state connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, transformers 这样的外部依赖提供基础框架能力，而 vllm.compilation.decorators, vllm.config, vllm.distributed, vllm.distributed.parallel_state 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### TYPE_CHECKING branch (lines 78-98)
```python
if TYPE_CHECKING:

    class Plamo2Config(PretrainedConfig):  # type: ignore
        model_type: str = "plamo2"

        hidden_size: int
        num_hidden_layers: int
        rms_norm_eps: float
        # Attention
        num_attention_heads: int
        hidden_size_per_head: int
        num_key_value_heads: int
        # Mamba
        mamba_d_state: int
        mamba_d_conv: int
        mamba_num_heads: int
        mamba_step: int
        # MLP
        intermediate_size: int
        # Tokenizer
        vocab_size: int
```
**EN:** This conditional block keeps optional imports, type-only definitions, or backend-specific branches isolated from the hot runtime path.
**CN:** 这个条件分支把可选导入、仅类型定义或特定后端分支与主要运行路径隔离开。

### Function `is_mamba` (lines 101-107)
```python
def is_mamba(config: "Plamo2Config", i: int) -> bool:
    assert config.mamba_step > 1

    if config.num_hidden_layers <= (config.mamba_step // 2):
        # use attention in last layer
        return i != config.num_hidden_layers - 1
    return (i % config.mamba_step) != (config.mamba_step // 2)
```
**EN:** The function `is_mamba` helps provide a reusable helper for the surrounding model code. Its main inputs are `config`, `i`.
**CN:** 函数 `is_mamba` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `config`、`i`。

### Class `Plamo2MambaMixer` (lines 114-483)
```python
@PluggableLayer.register("plamo2_mamba_mixer")
class Plamo2MambaMixer(MambaBase, PluggableLayer):
    # --8<-- [end:plamo2_mamba_mixer]

    def __init__(self, vllm_config: VllmConfig, *, prefix: str = "", **kwargs) -> None:
        super().__init__()
        self.config = vllm_config.model_config.hf_config
        self.cache_config = vllm_config.cache_config
        self.model_config = vllm_config.model_config
        self.quant_config = vllm_config.quant_config
        self.is_lora_enabled = bool(vllm_config.lora_config)
        self.hidden_size = self.config.hidden_size
        self.ssm_state_size = self.config.mamba_d_state
        self.conv_kernel_size = self.config.mamba_d_conv
        self.intermediate_size = (
            self.config.mamba_num_heads * self.config.hidden_size_per_head
        )
        self.tp_size = get_tensor_model_parallel_world_size()
        self.head_dim = self.config.hidden_size_per_head
        self.num_heads = self.config.mamba_num_heads
        self.time_step_rank = max(64, self.hidden_size // 16)
        self.conv1d = ColumnParallelLinear(
            input_size=self.conv_kernel_size,
# ... omitted for brevity ...
    def _project_ssm_parameters(self, hidden_states):
        if self.is_lora_enabled:
            #  Lora kernel requires contiguous tensor.
            ssm_parameters = self.bcdt_proj(hidden_states.contiguous())
        else:
            ssm_parameters = self.bcdt_proj(hidden_states)
        B, C, time_step = torch.split(
            ssm_parameters,
            [self.ssm_state_size, self.ssm_state_size, self.time_step_rank],
            dim=-1,
        )
# ... omitted for brevity ...
        dt = self.dt_proj(time_step)
        return B, C, dt
# ... omitted for brevity ...
    def forward(
        self,
        hidden_states: torch.Tensor,
        output: torch.Tensor,
        **kwargs,
    ):
        torch.ops.vllm.plamo2_mamba_mixer(
            hidden_states,
            output,
            self.prefix,
        )
# ... omitted for brevity ...
    def forward_impl(
        self,
        hidden_states: torch.Tensor,
        output: torch.Tensor,
        **kwargs,
    ):
        forward_context = get_forward_context()
        # attn_metadata contains metadata necessary for the mamba2 triton
        # kernels to operate in continuous batching and in chunked prefill
        # modes; they are computed at top-level model forward since they
        # stay the same and reused for all mamba layers in the same iteration
        attn_metadata: AttentionMetadata = forward_context.attn_metadata

        if attn_metadata is not None:
            assert isinstance(attn_metadata, dict)
            attn_metadata = attn_metadata[self.prefix]
            assert isinstance(attn_metadata, Mamba2AttentionMetadata)
            self_kv_cache = self.kv_cache
            # conv_state = (..., dim, width-1) yet contiguous along 'dim'
```
**EN:** Defines `Plamo2MambaMixer`, a supporting module used by the surrounding model implementation. It inherits from MambaBase, PluggableLayer. Key methods such as `__init__`, `_project_ssm_parameters`, `forward`, `forward_impl`, `get_state_dtype` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Plamo2MambaMixer`，它是一个被周边模型实现复用的支撑模块。 它继承自 MambaBase、PluggableLayer。 `__init__`, `_project_ssm_parameters`, `forward`, `forward_impl`, `get_state_dtype` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Function `plamo2_mamba_mixer` (lines 486-493)
```python
def plamo2_mamba_mixer(
    hidden_states: torch.Tensor,
    output: torch.Tensor,
    layer_name: str,
) -> None:
    forward_context: ForwardContext = get_forward_context()
    self = forward_context.no_compile_layers[layer_name]
    self.forward_impl(hidden_states=hidden_states, output=output)
```
**EN:** The function `plamo2_mamba_mixer` helps provide a reusable helper for the surrounding model code. Its main inputs are `hidden_states`, `output`, `layer_name`.
**CN:** 函数 `plamo2_mamba_mixer` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `hidden_states`、`output`、`layer_name`。

### Class `DenseMLP` (lines 512-543)
```python
class DenseMLP(nn.Module):
    def __init__(
        self,
        config: "Plamo2Config",
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        self.intermediate_size = config.intermediate_size
        self.gate_up_proj = MergedColumnParallelLinear(
            self.hidden_size,
            [self.intermediate_size] * 2,
            bias=False,
            prefix=f"{prefix}.gate_up_proj",
            quant_config=quant_config,
            return_bias=False,
        )
        self.act = SiluAndMul()
        self.down_proj = RowParallelLinear(
            self.intermediate_size,
            self.hidden_size,
            bias=False,
            prefix=f"{prefix}.down_proj",
            quant_config=quant_config,
            return_bias=False,
        )

    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        h = self.gate_up_proj(hidden_states)
        h = self.act(h)
        return self.down_proj(h)
```
**EN:** Defines `DenseMLP`, a feed-forward block that expands hidden states and projects them back. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `DenseMLP`，它是一个扩展隐藏状态并再投影回主维度的前馈模块。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Plamo2AttentionMixer` (lines 546-647)
```python
class Plamo2AttentionMixer(nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "", **kwargs) -> None:
        super().__init__()
        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config
        self.hidden_size = config.hidden_size
        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = config.num_attention_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = config.num_key_value_heads
        if self.total_num_kv_heads >= tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert tp_size % self.total_num_kv_heads == 0
# ... omitted for brevity ...
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        **kwargs,
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)

        q_shape = q.shape
        q = q.reshape(q_shape[:-1] + self.q_norm.weight.shape)
        q = self.q_norm.forward_native(q).reshape(q_shape)
        k_shape = k.shape
        k = k.reshape(k_shape[:-1] + self.k_norm.weight.shape)
        k = self.k_norm.forward_native(k).reshape(k_shape)

        q, k = self.rotary_emb(positions, q, k)
        attn_output = self.attn(q, k, v)
        output, _ = self.o_proj(attn_output)
```
**EN:** Defines `Plamo2AttentionMixer`, a attention block that projects Q/K/V tensors and dispatches the cache-aware attention path. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Plamo2AttentionMixer`，它是一个负责投影 Q/K/V 张量并分发带缓存注意力路径的注意力模块。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Plamo2DecoderLayer` (lines 650-710)
```python
class Plamo2DecoderLayer(nn.Module):
    def __init__(
        self, vllm_config: VllmConfig, layer_idx: int, prefix: str = "", **kwargs
    ) -> None:
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        self.is_mamba = is_mamba(config, layer_idx)
        if self.is_mamba:
            self.mixer = Plamo2MambaMixer(
                vllm_config=vllm_config, prefix=f"{prefix}.mixer"
            )
        else:
            self.mixer = Plamo2AttentionMixer(
                vllm_config=vllm_config, prefix=f"{prefix}.mixer"
            )

        self.mlp = DenseMLP(
            config=config, quant_config=quant_config, prefix=f"{prefix}.mlp"
        )
        self.pre_mixer_norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.post_mixer_norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.pre_mlp_norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.post_mlp_norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)

    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
# ... omitted for brevity ...
    ):
        if residual is None:
            residual = hidden_states
            hidden_states = self.pre_mixer_norm(hidden_states)
        else:
            hidden_states, residual = self.pre_mixer_norm(hidden_states, residual)

        if self.is_mamba:
            # Plamo2MambaMixer writes output to this tensor
            output = torch.empty_like(hidden_states)
            mixer_kwargs = {
                "output": output,
            }
        else:
            mixer_kwargs = {
                "positions": positions,
            }
        hidden_states = self.mixer(
            hidden_states=hidden_states,
            **mixer_kwargs,
        )
        if self.is_mamba:
            hidden_states = output
        hidden_states = self.post_mixer_norm(hidden_states)
        # Fully Connected
        hidden_states, residual = self.pre_mlp_norm(hidden_states, residual)
        hidden_states = self.mlp(hidden_states)
        hidden_states = self.post_mlp_norm(hidden_states)
        return hidden_states, residual
```
**EN:** Defines `Plamo2DecoderLayer`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Plamo2DecoderLayer`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Plamo2Decoder` (lines 713-744)
```python
class Plamo2Decoder(torch.nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "") -> None:
        super().__init__()
        config = vllm_config.model_config.hf_config
        extra_kwargs = {"is_lora_enabled": bool(vllm_config.lora_config)}

        def get_layer(prefix: str):
            layer_idx = int(prefix.rsplit(".", 1)[1])
            return Plamo2DecoderLayer(
                vllm_config=vllm_config,
                layer_idx=layer_idx,
                prefix=prefix,
                **extra_kwargs,
            )

        self.start_layer, self.end_layer, self.layers = make_layers(
            config.num_hidden_layers, get_layer, prefix=f"{prefix}.layers"
        )

    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        residual: torch.Tensor | None,
    ) -> torch.Tensor:
        for layer in islice(self.layers, self.start_layer, self.end_layer):
            hidden_states, residual = layer(
                positions=positions,
                hidden_states=hidden_states,
                residual=residual,
            )
        return hidden_states, residual
```
**EN:** Defines `Plamo2Decoder`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from torch.nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Plamo2Decoder`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 torch.nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Plamo2Model` (lines 747-876)
```python
@support_torch_compile
class Plamo2Model(torch.nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config = vllm_config.model_config.hf_config

        self.config = config
        self.vocab_size = config.vocab_size

        self.embed_tokens = VocabParallelEmbedding(
            self.vocab_size,
            config.hidden_size,
            prefix=f"{prefix}.embed_tokens",
        )
        self.make_empty_intermediate_tensors = make_empty_intermediate_tensors_factory(
            ["hidden_states", "residual"], config.hidden_size
        )
        self.layers = Plamo2Decoder(vllm_config=vllm_config, prefix=f"{prefix}.layers")
        self.norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
# ... omitted for brevity ...
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.embed_tokens(input_ids)
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor:
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

        hidden_states, residual = self.layers(
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        params_dict = dict(self.named_parameters())
        loaded_params: set[str] = set()
        for name, loaded_weight in weights:
            # Update the weight names to be compatible with the vllm version
            # of the model.
            # Do not change the order of the replacements.
            replacements = {
                # Rename incompatible weight names.
                ".A_log": ".A",
                ".B_norm_weight": ".B_norm.weight",
                ".C_norm_weight": ".C_norm.weight",
                ".dt_norm_weight": ".dt_norm.weight",
                ".q_weight": ".q_norm.weight",
                ".k_weight": ".k_norm.weight",
            }
            # Apply replacements based on the defined mappings
            for old, new in replacements.items():
                if old in name:
```
**EN:** Defines `Plamo2Model`, a backbone model container that orchestrates embeddings, layers, and output heads. It inherits from torch.nn.Module. Key methods such as `__init__`, `embed_input_ids`, `forward`, `load_weights` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Plamo2Model`，它是一个协调嵌入、层堆叠与输出头的主干模型容器。 它继承自 torch.nn.Module。 `__init__`, `embed_input_ids`, `forward`, `load_weights` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Plamo2ForCausalLM` (lines 879-992)
```python
class Plamo2ForCausalLM(
    torch.nn.Module, HasInnerState, SupportsLoRA, SupportsPP, IsHybrid
):
    packed_modules_mapping = {
        "qkv_proj": ["qkv_proj"],
        "gate_up_proj": ["gate_up_proj"],
        "in_proj": ["in_proj"],
    }

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "") -> None:
        super().__init__()
        config = vllm_config.model_config.hf_config
        scheduler_config = vllm_config.scheduler_config

        self.config = config
        self.vllm_config = vllm_config
        self.model_config = vllm_config.model_config
        self.scheduler_config = scheduler_config

        # ModelConfig.get_head_size assumes head_dim is set or calculated as
        # hidden_size // num_attention_heads. However, this is not always
        # the case for PLaMo2, as indicated by the FIXME comment.
        self.config.head_dim = self.config.hidden_size_per_head

        self.model = Plamo2Model(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "model")
        )
        self.vocab_size = self.config.vocab_size
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        **kwargs,
    ):
        hidden_states = self.model(
            input_ids, positions, intermediate_tensors, inputs_embeds
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
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        loader = AutoWeightsLoader(
            self,
            skip_prefixes=(["lm_head."] if self.config.tie_word_embeddings else None),
        )
        return loader.load_weights(weights)
```
**EN:** Defines `Plamo2ForCausalLM`, a top-level language-model wrapper that exposes logits and weight-loading behavior. It inherits from torch.nn.Module, HasInnerState, SupportsLoRA. Key methods such as `__init__`, `embed_input_ids`, `forward`, `get_mamba_state_dtype_from_config`, `get_mamba_state_shape_from_config` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Plamo2ForCausalLM`，它是一个暴露 logits 计算与权重加载行为的顶层语言模型封装类。 它继承自 torch.nn.Module、HasInnerState、SupportsLoRA。 `__init__`, `embed_input_ids`, `forward`, `get_mamba_state_dtype_from_config`, `get_mamba_state_shape_from_config` 等关键方法展示了构造、前向执行或权重适配发生的位置。

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
- **Position encoding**
  - **EN:** Rotary or related positional logic appears in the file to preserve token order information efficiently.
  - **CN:** 文件中出现了 Rotary 等位置编码逻辑，用于高效保留 token 顺序信息。

## Dependencies / 依赖关系
- **Standard library**: collections.abc, itertools, typing
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, transformers
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.compilation.decorators, vllm.config, vllm.distributed, vllm.distributed.parallel_state, vllm.forward_context, vllm.model_executor.custom_op, vllm.model_executor.layers.activation, vllm.model_executor.layers.attention
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
