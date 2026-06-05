# ministral3.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/ministral3.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the ministral3 model stack used by SGLang SRT inference, including network blocks, runtime wrappers, and checkpoint-loading utilities. / 该模块实现 SGLang SRT 推理所使用的 ministral3 模型栈，包括网络模块、运行时封装以及检查点加载工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14: Module imports
```python
from typing import Any, Dict, Optional

import torch
from transformers import PretrainedConfig

from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.models.llama import (
    LlamaAttention,
    LlamaDecoderLayer,
    LlamaForCausalLM,
    LlamaModel,
)
from sglang.srt.utils import add_prefix, make_layers
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 17-23: Function `_get_llama_4_attn_scale`
```python
def _get_llama_4_attn_scale(
    positions_ids: torch.Tensor, beta: float, max_position_embeddings: int
) -> torch.Tensor:
    scaling = 1 + beta * torch.log(
        1 + torch.floor(positions_ids / max_position_embeddings)
    )
    return scaling.unsqueeze(-1)
```
**EN:** This function implements `_get_llama_4_attn_scale(positions_ids: ..., beta: ..., max_position_embeddings: ...)` and implements attention-related tensor preparation or execution.
**CN:** 这个函数实现了 `_get_llama_4_attn_scale(positions_ids: ..., beta: ..., max_position_embeddings: ...)`，其作用是实现与注意力相关的张量准备或执行逻辑。

### Lines 26-26: Class `Ministral3Attention` overview
```python
class Ministral3Attention(LlamaAttention):
```
**EN:** Defines `Ministral3Attention` as a reusable runtime type derived from LlamaAttention. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Ministral3Attention`，其继承关系为 LlamaAttention。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 27-64: Method `Ministral3Attention.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        layer_id: int = 0,
        rope_theta: float = 1000000.0,
        rope_scaling: Optional[Dict[str, Any]] = {},
        rope_is_neox_style: bool = True,
        max_position_embeddings: int = 8192,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        bias: bool = False,
    ) -> None:
        super().__init__(
            config,
            hidden_size,
            num_heads,
            num_kv_heads,
            layer_id,
            rope_theta,
            rope_scaling,
            rope_is_neox_style,
            max_position_embeddings,
            quant_config,
            prefix,
            bias,
        )
        # Ministral3 specific: llama 4 style scaling beta
        self.llama_4_scaling_beta = config.rope_parameters.get("llama_4_scaling_beta")

        # sliding window
        self.sliding_window = getattr(config, "sliding_window", None)
        if self.sliding_window is not None:
            # Update RadixAttention with sliding window if needed
            # currently RadixAttention in sglang handles this mostly via logic in forward/flashinfer
            pass
```
**EN:** This method implements `__init__(config: ..., hidden_size: ..., num_heads: ..., num_kv_heads: ..., layer_id: ...=..., rope_theta: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., hidden_size: ..., num_heads: ..., num_kv_heads: ..., layer_id: ...=..., rope_theta: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 66-94: Method `Ministral3Attention.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)

        # Apply RoPE
        q, k = self.rotary_emb(positions, q, k)

        # Ministral3 / Llama 4 scaling
        if self.llama_4_scaling_beta is not None:
            scale = _get_llama_4_attn_scale(
                positions, self.llama_4_scaling_beta, self.max_position_embeddings
            ).to(q.dtype)
            # q shape is [batch_size * seq_len, num_heads * head_dim] or [batch_size * seq_len, num_heads, head_dim]
            # positions is [batch_size * seq_len]
            # scale is [batch_size * seq_len, 1]
            # We need to reshape q to apply scale correctly if it's flattened
            # Assuming q is (total_tokens, num_heads * head_dim)
            q = q.view(-1, self.num_heads, self.head_dim)
            q = q * scale.unsqueeze(1)  # Broadcast over heads
            q = q.view(-1, self.num_heads * self.head_dim)

        attn_output = self.attn(q, k, v, forward_batch)
        output, _ = self.o_proj(attn_output)
        return output
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 97-97: Class `Ministral3DecoderLayer` overview
```python
class Ministral3DecoderLayer(LlamaDecoderLayer):
```
**EN:** Defines `Ministral3DecoderLayer` as a reusable runtime type derived from LlamaDecoderLayer. The class groups 1 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Ministral3DecoderLayer`，其继承关系为 LlamaDecoderLayer。这个类组织了 1 个方法，用于实现模型相关行为。

### Lines 98-115: Method `Ministral3DecoderLayer.__init__`
```python
    def __init__(self, config, layer_id=0, quant_config=None, prefix=""):
        super().__init__(config, layer_id, quant_config, prefix)
        self.self_attn = Ministral3Attention(
            config=config,
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            num_kv_heads=config.num_key_value_heads,
            layer_id=layer_id,
            rope_theta=config.rope_parameters["rope_theta"],
            rope_scaling=config.rope_parameters,  # rope_scaling is rope_parameters in Ministral3Config
            max_position_embeddings=getattr(
                config, "original_max_position_embeddings", 16384
            ),
            quant_config=quant_config,
            prefix=add_prefix("self_attn", prefix),
            bias=getattr(config, "attention_bias", False)
            or getattr(config, "bias", False),
        )
```
**EN:** This method implements `__init__(config, layer_id=..., quant_config=..., prefix=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config, layer_id=..., quant_config=..., prefix=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 118-118: Class `Ministral3Model` overview
```python
class Ministral3Model(LlamaModel):
```
**EN:** Defines `Ministral3Model` as a reusable runtime type derived from LlamaModel. The class groups 1 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Ministral3Model`，其继承关系为 LlamaModel。这个类组织了 1 个方法，用于实现模型相关行为。

### Lines 119-136: Method `Ministral3Model.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        # Override layer creation to use Ministral3Attention
        super().__init__(config, quant_config, prefix)

        self.layers, self.start_layer, self.end_layer = make_layers(
            config.num_hidden_layers,
            lambda idx, prefix: Ministral3DecoderLayer(
                config=config, quant_config=quant_config, layer_id=idx, prefix=prefix
            ),
            pp_rank=self.pp_group.rank_in_group,
            pp_size=self.pp_group.world_size,
            prefix="model.layers",
        )
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 139-139: Class `Ministral3ForCausalLM` overview
```python
class Ministral3ForCausalLM(LlamaForCausalLM):
```
**EN:** Defines `Ministral3ForCausalLM` as a reusable runtime type derived from LlamaForCausalLM. The class groups 1 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Ministral3ForCausalLM`，其继承关系为 LlamaForCausalLM。这个类组织了 1 个方法，用于实现模型相关行为。

### Lines 140-146: Method `Ministral3ForCausalLM._init_model`
```python
    def _init_model(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        return Ministral3Model(config, quant_config, prefix=prefix)
```
**EN:** This method implements `_init_model(config: ..., quant_config: ...=..., prefix: ...=...)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `_init_model(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是执行后续运行时所需的初始化工作。

### Lines 149-149: Top-level assign
```python
EntryClass = [Ministral3ForCausalLM]
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `typing.Any`
- `typing.Dict`
- `typing.Optional`
- `torch`
- `transformers.PretrainedConfig`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.models.llama.LlamaAttention`
- `sglang.srt.models.llama.LlamaDecoderLayer`
- `sglang.srt.models.llama.LlamaForCausalLM`
- `sglang.srt.models.llama.LlamaModel`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.make_layers`
