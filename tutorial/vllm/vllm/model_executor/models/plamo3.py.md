# plamo3.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/plamo3.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Plamo3 model components and runtime adapter for vLLM inference. The module docstring summarizes it as: "Inference-only PLaMo3 model." / 实现 Plamo3 在 vLLM 推理中的模型组件与运行时适配器。 模块文档字符串还将其概括为：“Inference-only PLaMo3 model。”

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-49)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""Inference-only PLaMo3 model."""

from collections.abc import Iterable
from itertools import islice
from typing import TYPE_CHECKING, Any

import torch
from torch import nn
from transformers import PretrainedConfig

from vllm.compilation.decorators import support_torch_compile
from vllm.config import VllmConfig
from vllm.distributed import get_tensor_model_parallel_world_size
from vllm.distributed.parallel_state import get_pp_group
from vllm.model_executor.layers.activation import SiluAndMul
from vllm.model_executor.layers.attention import Attention
# ... omitted for brevity ...
    LoaderFunction,
    composed_weight_loader,
    default_weight_loader,
)
from vllm.model_executor.models.interfaces import SupportsLoRA, SupportsPP
from vllm.model_executor.models.utils import (
    AutoWeightsLoader,
    extract_layer_index,
    make_empty_intermediate_tensors_factory,
    make_layers,
    maybe_prefix,
)
from vllm.model_executor.utils import set_weight_attrs
from vllm.sequence import IntermediateTensors

# Only used for type hinting.
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, transformers supply framework primitives, while internal modules like vllm.compilation.decorators, vllm.config, vllm.distributed, vllm.distributed.parallel_state connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, transformers 这样的外部依赖提供基础框架能力，而 vllm.compilation.decorators, vllm.config, vllm.distributed, vllm.distributed.parallel_state 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### TYPE_CHECKING branch (lines 50-71)
```python
if TYPE_CHECKING:

    class Plamo3Config(PretrainedConfig):  # type: ignore
        model_type: str = "plamo3"

        hidden_size: int
        num_hidden_layers: int
        rms_norm_eps: float
        # Attention
        num_attention_heads: int
        head_dim: int
        num_key_value_heads: int
        # vllm rename `sliding_window` attr to `interleaved_sliding_window`
        # if `sliding_window` is list
        interleaved_sliding_window: list[int | None]
        sliding_window_pattern: int
        rope_parameters: dict[str, Any]
        rope_local_theta: int
        # MLP
        intermediate_size: int
        # Tokenizer
        vocab_size: int
```
**EN:** This conditional block keeps optional imports, type-only definitions, or backend-specific branches isolated from the hot runtime path.
**CN:** 这个条件分支把可选导入、仅类型定义或特定后端分支与主要运行路径隔离开。

### Function `rms_norm_weight_loader` (lines 74-78)
```python
def rms_norm_weight_loader(offset: float) -> LoaderFunction:
    return composed_weight_loader(
        default_weight_loader,
        lambda x: x + offset,
    )
```
**EN:** The function `rms_norm_weight_loader` helps translate or load checkpoint tensors into vLLM modules. Its main inputs are `offset`.
**CN:** 函数 `rms_norm_weight_loader` 用于将检查点张量转换或加载到 vLLM 模块中。 它的主要输入包括 `offset`。

### Class `DenseMLP` (lines 81-112)
```python
class DenseMLP(nn.Module):
    def __init__(
        self,
        config: "Plamo3Config",
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

### Class `Plamo3AttentionMixer` (lines 115-224)
```python
class Plamo3AttentionMixer(nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "", **kwargs) -> None:
        super().__init__()
        config = vllm_config.model_config.hf_config
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
        residual: torch.Tensor | None,
        **kwargs: Any,
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)

        q_shape = q.shape
        q = q.reshape(q_shape[:-1] + (q_shape[-1] // self.head_dim, self.head_dim))
        q = self.q_norm.forward_native(q).reshape(q_shape)
        k_shape = k.shape
        k = k.reshape(k_shape[:-1] + (k_shape[-1] // self.head_dim, self.head_dim))
        k = self.k_norm.forward_native(k).reshape(k_shape)

        q, k = self.rotary_emb(positions, q, k)
        attn_output = self.attn(q, k, v)
```
**EN:** Defines `Plamo3AttentionMixer`, a attention block that projects Q/K/V tensors and dispatches the cache-aware attention path. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Plamo3AttentionMixer`，它是一个负责投影 Q/K/V 张量并分发带缓存注意力路径的注意力模块。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Plamo3DecoderLayer` (lines 227-285)
```python
class Plamo3DecoderLayer(nn.Module):
    def __init__(
        self, vllm_config: VllmConfig, prefix: str = "", **kwargs: Any
    ) -> None:
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        self.mixer = Plamo3AttentionMixer(
            vllm_config=vllm_config,
            prefix=f"{prefix}.mixer",
        )

        self.mlp = DenseMLP(
            config=config, quant_config=quant_config, prefix=f"{prefix}.mlp"
        )
        self.pre_mixer_norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        set_weight_attrs(
            self.pre_mixer_norm.weight,
            {"weight_loader": rms_norm_weight_loader(offset=1.0)},
        )
        self.post_mixer_norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        set_weight_attrs(
            self.post_mixer_norm.weight,
            {"weight_loader": rms_norm_weight_loader(offset=1.0 / 5)},
        )
        self.pre_mlp_norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        set_weight_attrs(
            self.pre_mlp_norm.weight,
            {"weight_loader": rms_norm_weight_loader(offset=1.0)},
        )
        self.post_mlp_norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        set_weight_attrs(
            self.post_mlp_norm.weight,
            {"weight_loader": rms_norm_weight_loader(offset=1.0 / (5**1.5))},
        )

    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        residual: torch.Tensor | None,
        **kwargs: Any,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        if residual is None:
            residual = hidden_states
            hidden_states = self.pre_mixer_norm(hidden_states)
        else:
            hidden_states, residual = self.pre_mixer_norm(hidden_states, residual)

        hidden_states = self.mixer(
            positions=positions, hidden_states=hidden_states, residual=residual
        )
        hidden_states = self.post_mixer_norm(hidden_states)
        # Fully Connected
        hidden_states, residual = self.pre_mlp_norm(hidden_states, residual)
        hidden_states = self.mlp(hidden_states)
        hidden_states = self.post_mlp_norm(hidden_states)
        return hidden_states, residual
```
**EN:** Defines `Plamo3DecoderLayer`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Plamo3DecoderLayer`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Plamo3Decoder` (lines 288-311)
```python
class Plamo3Decoder(torch.nn.Module):
    def __init__(self, vllm_config: VllmConfig, prefix: str = "") -> None:
        super().__init__()
        num_hidden_layers = vllm_config.model_config.hf_config.num_hidden_layers

        self.start_layer, self.end_layer, self.layers = make_layers(
            num_hidden_layers,
            lambda prefix: Plamo3DecoderLayer(vllm_config, prefix=prefix),
            prefix=f"{prefix}.layers",
        )

    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        residual: torch.Tensor | None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        for layer in islice(self.layers, self.start_layer, self.end_layer):
            hidden_states, residual = layer(
                positions=positions,
                hidden_states=hidden_states,
                residual=residual,
            )
        return hidden_states, residual
```
**EN:** Defines `Plamo3Decoder`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from torch.nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Plamo3Decoder`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 torch.nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Plamo3Model` (lines 314-369)
```python
@support_torch_compile
class Plamo3Model(nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config

        self.config = config
        self.vocab_size = config.vocab_size
        self.org_vocab_size = config.vocab_size

        self.embed_tokens = VocabParallelEmbedding(
            self.vocab_size,
            config.hidden_size,
            org_num_embeddings=config.vocab_size,
            prefix=f"{prefix}.embed_tokens",
        )
        self.make_empty_intermediate_tensors = make_empty_intermediate_tensors_factory(
            ["hidden_states", "residual"], config.hidden_size
        )
        self.layers = Plamo3Decoder(vllm_config, prefix=f"{prefix}.layers")
        self.norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        set_weight_attrs(
            self.norm.weight,
            {"weight_loader": rms_norm_weight_loader(offset=1.0)},
        )

    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.embed_tokens(input_ids)

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
            positions=positions, hidden_states=hidden_states, residual=residual
        )
        if not get_pp_group().is_last_rank:
            return IntermediateTensors(
                {"hidden_states": hidden_states, "residual": residual}
            )
        hidden_states, _ = self.norm(hidden_states, residual)
        return hidden_states
```
**EN:** Defines `Plamo3Model`, a backbone model container that orchestrates embeddings, layers, and output heads. It inherits from nn.Module. Key methods such as `__init__`, `embed_input_ids`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Plamo3Model`，它是一个协调嵌入、层堆叠与输出头的主干模型容器。 它继承自 nn.Module。 `__init__`, `embed_input_ids`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Plamo3ForCausalLM` (lines 372-437)
```python
class Plamo3ForCausalLM(nn.Module, SupportsLoRA, SupportsPP):
    packed_modules_mapping = {
        "qkv_proj": ["qkv_proj"],
        "gate_up_proj": ["gate_up_proj"],
    }

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "") -> None:
        super().__init__()
        self.config = vllm_config.model_config.hf_config
        self.vllm_config = vllm_config
        self.model_config = vllm_config.model_config
        self.scheduler_config = vllm_config.scheduler_config

        self.model = Plamo3Model(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "model")
        )

        self.vocab_size = self.config.vocab_size
        self.unpadded_vocab_size = self.config.vocab_size

        num_embeddings = ((self.vocab_size + 15) // 16) * 16
        self.lm_head = ParallelLMHead(
            num_embeddings,
            self.config.hidden_size,
            org_num_embeddings=self.config.vocab_size,
            padding_size=DEFAULT_VOCAB_PADDING_SIZE,
            prefix=f"{prefix}.lm_head",
        )
        if self.config.tie_word_embeddings:
            self.lm_head = self.lm_head.tie_weights(self.model.embed_tokens)
# ... omitted for brevity ...

    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.model.embed_input_ids(input_ids)

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

    def compute_logits(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor | None:
        logits = self.logits_processor(self.lm_head, hidden_states)
        return logits

    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]):
        loader = AutoWeightsLoader(
            self,
            skip_prefixes=(["lm_head."] if self.config.tie_word_embeddings else None),
        )
        return loader.load_weights(weights)
```
**EN:** Defines `Plamo3ForCausalLM`, a top-level language-model wrapper that exposes logits and weight-loading behavior. It inherits from nn.Module, SupportsLoRA, SupportsPP. Key methods such as `__init__`, `embed_input_ids`, `forward`, `compute_logits`, `load_weights` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Plamo3ForCausalLM`，它是一个暴露 logits 计算与权重加载行为的顶层语言模型封装类。 它继承自 nn.Module、SupportsLoRA、SupportsPP。 `__init__`, `embed_input_ids`, `forward`, `compute_logits`, `load_weights` 等关键方法展示了构造、前向执行或权重适配发生的位置。

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
- **vLLM internal modules**: vllm.compilation.decorators, vllm.config, vllm.distributed, vllm.distributed.parallel_state, vllm.model_executor.layers.activation, vllm.model_executor.layers.attention, vllm.model_executor.layers.layernorm, vllm.model_executor.layers.linear
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
