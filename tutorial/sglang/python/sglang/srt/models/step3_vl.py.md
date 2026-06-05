# step3_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/step3_vl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the step3 vl multimodal runtime stack in SGLang, including model blocks, adapters, and inference helpers. / 该模块实现 SGLang 中 step3 vl 的多模态运行时堆栈，包括模型模块、适配器与推理辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-64: Module imports
```python
import logging
import math
from math import sqrt
from typing import Any, Dict, Iterable, List, Optional, Tuple

import torch
from torch import nn
from torch.nn import LayerNorm
from torch.nn import functional as F
from transformers import PretrainedConfig
from transformers.activations import ACT2FN

from sglang.srt.configs.step3_vl import (
    Step3TextConfig,
    Step3VisionEncoderConfig,
    Step3VLConfig,
)
from sglang.srt.distributed import (
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
    tensor_model_parallel_all_reduce,
)
from sglang.srt.eplb.expert_location import ModelConfigForExpertLocation
from sglang.srt.layers.activation import SiluAndMul
from sglang.srt.layers.attention.vision import VisionAttention
from sglang.srt.layers.communicator import LayerCommunicator, LayerScatterModes
from sglang.srt.layers.conv import Conv2dLayer
from sglang.srt.layers.dp_attention import (
    get_attention_tp_rank,
    get_attention_tp_size,
    is_dp_attention_enabled,
)
from sglang.srt.layers.layernorm import RMSNorm
from sglang.srt.layers.linear import (
    ColumnParallelLinear,
    MergedColumnParallelLinear,
    ReplicatedLinear,
    RowParallelLinear,
)
from sglang.srt.layers.logits_processor import LogitsProcessor
# ... truncated for brevity ...
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 66-66: Top-level assign
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines or updates logger, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 logger，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 69-71: Top-level expr
```python
"""
Text Model
"""
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 74-74: Class `Step3TextMLP` overview
```python
class Step3TextMLP(nn.Module):
```
**EN:** Defines `Step3TextMLP` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Step3TextMLP`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 75-103: Method `Step3TextMLP.__init__`
```python
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.gate_up_proj = MergedColumnParallelLinear(
            hidden_size,
            [intermediate_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("gate_up_proj", prefix),
        )
        self.down_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("down_proj", prefix),
        )
        if hidden_act != "silu":
            raise ValueError(
                f"Unsupported activation: {hidden_act}. "
                "Only silu is supported for now."
            )
        self.act_fn = SiluAndMul()
```
**EN:** This method implements `__init__(hidden_size: ..., intermediate_size: ..., hidden_act: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(hidden_size: ..., intermediate_size: ..., hidden_act: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 105-109: Method `Step3TextMLP.forward`
```python
    def forward(self, x):
        gate_up, _ = self.gate_up_proj(x)
        x = self.act_fn(gate_up)
        x, _ = self.down_proj(x)
        return x
```
**EN:** This method implements `forward(x)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 112-113: Class `Step3TextMoEMLP` overview
```python
class Step3TextMoEMLP(nn.Module):
    # Native
```
**EN:** Defines `Step3TextMoEMLP` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Step3TextMoEMLP`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 114-156: Method `Step3TextMoEMLP.__init__`
```python
    def __init__(
        self,
        layer_id: int,
        config: Step3TextConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.tp_size = get_tensor_model_parallel_world_size()
        self.layer_id = layer_id
        if self.tp_size > config.moe_num_experts:
            raise ValueError(
                f"Tensor parallel size {self.tp_size} is greater than "
                f"the number of experts {config.moe_num_experts}."
            )

        self.topk = TopK(
            top_k=config.moe_top_k,
            renormalize=config.norm_expert_weight,
            use_grouped_topk=False,
            layer_id=layer_id,
        )

        self.experts = get_moe_impl_class(quant_config)(
            num_experts=config.moe_num_experts,
            top_k=config.moe_top_k,
            hidden_size=config.hidden_size,
            intermediate_size=config.moe_intermediate_size,
            layer_id=layer_id,
            quant_config=quant_config,
            prefix=add_prefix("experts", prefix),
        )

        self.gate = ReplicatedLinear(
            config.hidden_size,
            output_size=config.moe_num_experts,
            bias=False,
            quant_config=None,
            prefix=add_prefix("gate", prefix),
        )
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(layer_id: ..., config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(layer_id: ..., config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 158-170: Method `Step3TextMoEMLP.forward`
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        num_tokens, hidden_dim = hidden_states.shape
        hidden_states = hidden_states.view(-1, hidden_dim)

        router_logits, _ = self.gate(hidden_states)
        topk_output = self.topk(hidden_states, router_logits)
        final_hidden_states = self.experts(
            hidden_states=hidden_states, topk_output=topk_output
        )

        if self.tp_size > 1:
            final_hidden_states = tensor_model_parallel_all_reduce(final_hidden_states)
        return final_hidden_states.view(num_tokens, hidden_dim)
```
**EN:** This method implements `forward(hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 173-173: Class `Step3TextAttention` overview
```python
class Step3TextAttention(nn.Module):
```
**EN:** Defines `Step3TextAttention` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Step3TextAttention`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 174-266: Method `Step3TextAttention.__init__`
```python
    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        head_dim: int,
        share_q_dim: int,
        layer_id: int = 0,
        rope_theta: float = 10000,
        rope_scaling: Optional[Dict[str, Any]] = None,
        max_position_embeddings: int = 8192,
        quant_config: Optional[QuantizationConfig] = None,
        rms_norm_eps=None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size

        attn_tp_rank = get_attention_tp_rank()
        attn_tp_size = get_attention_tp_size()

        self.all_tp_rank = get_tensor_model_parallel_rank()
        self.total_num_heads = num_heads
        self.attn_tp_rank = attn_tp_rank
        self.layer_id = layer_id
        assert self.total_num_heads % attn_tp_size == 0
        self.num_heads = self.total_num_heads // attn_tp_size
        self.total_num_kv_heads = num_kv_heads
        if self.total_num_kv_heads >= attn_tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % attn_tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert attn_tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // attn_tp_size)
        self.head_dim = head_dim
        self.q_size = share_q_dim if share_q_dim else head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(hidden_size: ..., num_heads: ..., num_kv_heads: ..., head_dim: ..., share_q_dim: ..., layer_id: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(hidden_size: ..., num_heads: ..., num_kv_heads: ..., head_dim: ..., share_q_dim: ..., layer_id: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 268-281: Method `Step3TextAttention.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        q = self.inter_norm(q.contiguous())
        q, _ = self.wq(q)
        q, k = self.rotary_emb(positions, q, k)
        attn_output = self.attn(q, k, v, forward_batch)
        output, _ = self.o_proj(attn_output)
        return output
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 284-284: Class `Step3TextDecoderLayer` overview
```python
class Step3TextDecoderLayer(nn.Module):
```
**EN:** Defines `Step3TextDecoderLayer` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Step3TextDecoderLayer`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 285-388: Method `Step3TextDecoderLayer.__init__`
```python
    def __init__(
        self,
        config: Step3TextConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        rope_theta, rope_scaling = get_rope_config(config)
        max_position_embeddings = getattr(config, "max_position_embeddings", 8192)
        head_dim = getattr(
            config, "head_dim", config.hidden_size // config.num_attention_heads
        )
        # TODO: support shared experts fusion
        # self.n_shared_experts = 1
        # self.num_fused_shared_experts = (
        #     0
        #     if global_server_args.disable_shared_experts_fusion
        #     else self.n_shared_experts
        # )
        self.num_fused_shared_experts = 0
        rms_norm_eps = config.rms_norm_eps
        self.self_attn = Step3TextAttention(
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            num_kv_heads=1,
            head_dim=head_dim,
            share_q_dim=config.share_q_dim,
            layer_id=layer_id,
            rope_theta=rope_theta,
            rope_scaling=rope_scaling,
            max_position_embeddings=max_position_embeddings,
            rms_norm_eps=rms_norm_eps,
            quant_config=quant_config,
            prefix=add_prefix("self_attn", prefix),
        )

        moe_layers_enum = getattr(config, "moe_layers_enum", None)
        if moe_layers_enum is not None:
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 390-397: Method `Step3TextDecoderLayer.moe_mlp_forward`
```python
    def moe_mlp_forward(self, hidden_states):
        if not self.num_fused_shared_experts:
            h = hidden_states.clone()
            hidden_states = self.moe(hidden_states)
            hidden_states += self.share_expert(h)
        else:
            hidden_states = self.moe(hidden_states)
        return hidden_states
```
**EN:** This method implements `moe_mlp_forward(hidden_states)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `moe_mlp_forward(hidden_states)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 399-430: Method `Step3TextDecoderLayer.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
        residual: Optional[torch.Tensor],
    ) -> Tuple[torch.Tensor, torch.Tensor]:

        hidden_states, residual = self.layer_communicator.prepare_attn(
            hidden_states, residual, forward_batch
        )

        if hidden_states.shape[0] != 0:
            hidden_states = self.self_attn(
                positions=positions,
                hidden_states=hidden_states,
                forward_batch=forward_batch,
            )

        hidden_states, residual = self.layer_communicator.prepare_mlp(
            hidden_states, residual, forward_batch
        )
        if self.use_moe:
            hidden_states = self.moe_mlp_forward(hidden_states)
        else:
            hidden_states = self.mlp(hidden_states)

        hidden_states, residual = self.layer_communicator.postprocess_layer(
            hidden_states, residual, forward_batch
        )

        return hidden_states, residual
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 433-433: Class `Step3TextModel` overview
```python
class Step3TextModel(nn.Module):
```
**EN:** Defines `Step3TextModel` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Step3TextModel`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 434-461: Method `Step3TextModel.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.padding_idx = config.pad_token_id
        self.vocab_size = config.vocab_size

        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size,
            config.hidden_size,
            use_attn_tp_group=is_dp_attention_enabled(),
            prefix=add_prefix("embed_tokens", prefix),
        )

        self.layers = make_layers(
            config.num_hidden_layers,
            lambda idx, prefix: Step3TextDecoderLayer(
                layer_id=idx,
                config=config,
                quant_config=quant_config,
                prefix=prefix,
            ),
            prefix=add_prefix("layers", prefix),
        )
        self.norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 463-464: Method `Step3TextModel.get_input_embeddings`
```python
    def get_input_embeddings(self):
        return self.embed_tokens
```
**EN:** This method implements `get_input_embeddings()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 466-490: Method `Step3TextModel.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
    ) -> torch.Tensor:
        if input_embeds is None:
            hidden_states = self.embed_tokens(input_ids)
        else:
            hidden_states = input_embeds

        residual = None
        for i in range(len(self.layers)):
            layer = self.layers[i]
            hidden_states, residual = layer(
                positions, hidden_states, forward_batch, residual
            )

        if hidden_states.shape[0] != 0:
            if residual is None:
                hidden_states = self.norm(hidden_states)
            else:
                hidden_states, _ = self.norm(hidden_states, residual)
        return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 493-495: Top-level expr
```python
"""
Vision Model
"""
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 498-527: Function `get_abs_pos`
```python
def get_abs_pos(abs_pos, tgt_size):
    dim = abs_pos.size(-1)
    abs_pos_new = abs_pos.squeeze(0)
    cls_token, old_pos_embed = abs_pos_new[:1], abs_pos_new[1:]

    src_size = int(math.sqrt(abs_pos_new.shape[0] - 1))
    tgt_size = int(math.sqrt(tgt_size))
    dtype = abs_pos.dtype

    if src_size != tgt_size:
        old_pos_embed = (
            old_pos_embed.view(1, src_size, src_size, dim)
            .permute(0, 3, 1, 2)
            .contiguous()
        )
        old_pos_embed = old_pos_embed.to(torch.float32)
        new_pos_embed = F.interpolate(
            old_pos_embed,
            size=(tgt_size, tgt_size),
            mode="bicubic",
            antialias=True,
            align_corners=False,
        ).to(dtype)
        new_pos_embed = new_pos_embed.permute(0, 2, 3, 1)
        new_pos_embed = new_pos_embed.view(tgt_size * tgt_size, dim)
        vision_pos_embed = torch.cat([cls_token, new_pos_embed], dim=0)
        vision_pos_embed = vision_pos_embed.view(1, tgt_size * tgt_size + 1, dim)
        return vision_pos_embed
    else:
        return abs_pos
```
**EN:** This function implements `get_abs_pos(abs_pos, tgt_size)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个函数实现了 `get_abs_pos(abs_pos, tgt_size)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 530-530: Class `Step3VisionMLP` overview
```python
class Step3VisionMLP(nn.Module):
```
**EN:** Defines `Step3VisionMLP` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Step3VisionMLP`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 531-564: Method `Step3VisionMLP.__init__`
```python
    def __init__(
        self,
        dim: int,
        intermediate_size: int,
        bias: bool = True,
        hidden_act="quick_gelu",
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        # Since this is a dense model,
        # the MLP component likewise adopts a DP-MLP approach modeled after DP Attention.
        # This choice may not represent the optimal solution and remains open to further deliberation.
        attn_tp_rank = get_attention_tp_rank()
        attn_tp_size = get_attention_tp_size()
        self.fc1 = ColumnParallelLinear(
            dim,
            intermediate_size,
            bias=bias,
            quant_config=quant_config,
            tp_rank=attn_tp_rank,
            tp_size=attn_tp_size,
            prefix=add_prefix("gate_proj", prefix),
        )
        self.act = ACT2FN[hidden_act]  # quick_gelu
        self.fc2 = RowParallelLinear(
            intermediate_size,
            dim,
            bias=bias,
            quant_config=quant_config,
            tp_rank=attn_tp_rank,
            tp_size=attn_tp_size,
            prefix=add_prefix("down_proj", prefix),
        )
```
**EN:** This method implements `__init__(dim: ..., intermediate_size: ..., bias: ...=..., hidden_act=..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(dim: ..., intermediate_size: ..., bias: ...=..., hidden_act=..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 566-570: Method `Step3VisionMLP.forward`
```python
    def forward(self, hidden_states) -> torch.Tensor:
        hidden_states, _ = self.fc1(hidden_states)
        hidden_states = self.act(hidden_states)
        hidden_states, _ = self.fc2(hidden_states)
        return hidden_states
```
**EN:** This method implements `forward(hidden_states)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 573-573: Class `Step3VisionAttention` overview
```python
class Step3VisionAttention(nn.Module):
```
**EN:** Defines `Step3VisionAttention` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Step3VisionAttention`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 574-602: Method `Step3VisionAttention.__init__`
```python
    def __init__(
        self,
        dim: int,
        num_heads: int = 16,
        quant_config=None,
        prefix: str = "",
    ) -> None:

        super().__init__()
        self.num_heads = num_heads
        self.head_dim = dim // num_heads
        self.out_proj = RowParallelLinear(
            dim,
            dim,
            bias=True,
            quant_config=quant_config,
            prefix=add_prefix("out_proj", prefix),
        )
        self.scale = self.head_dim**-0.5

        self.attn = VisionAttention(
            embed_dim=dim,
            num_heads=num_heads,
            projection_size=dim,
            use_qkv_parallel=True,
            proj_bias=True,
            quant_config=quant_config,
            prefix=add_prefix("attn", prefix),
        )
```
**EN:** This method implements `__init__(dim: ..., num_heads: ...=..., quant_config=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(dim: ..., num_heads: ...=..., quant_config=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 604-606: Method `Step3VisionAttention.forward`
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        attn_output = self.attn(hidden_states)
        return attn_output
```
**EN:** This method implements `forward(hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 609-610: Class `Step3VisionEmbeddings` overview
```python
class Step3VisionEmbeddings(nn.Module):
```
**EN:** Defines `Step3VisionEmbeddings` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Step3VisionEmbeddings`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 611-638: Method `Step3VisionEmbeddings.__init__`
```python
    def __init__(self, config: Step3VisionEncoderConfig):
        super().__init__()
        self.config = config
        self.embed_dim = config.hidden_size
        self.image_size = config.image_size
        self.patch_size = config.patch_size

        self.class_embedding = nn.Parameter(torch.randn(1, self.embed_dim))

        self.patch_embedding = Conv2dLayer(
            in_channels=config.num_channels,
            out_channels=self.embed_dim,
            kernel_size=self.patch_size,
            stride=self.patch_size,
            bias=True,
        )

        self.num_patches = (self.image_size // self.patch_size) ** 2
        self.pad_tp_size = 4  # hard code for padding
        # To load the pretrained weights, we still use P+1 as the seqlen
        self.position_embedding = torch.nn.Embedding(
            self.num_patches + 1, self.embed_dim
        )
        self.register_buffer(
            "position_ids",
            torch.arange(self.num_patches + 1).expand((1, -1)),
            persistent=False,
        )
```
**EN:** This method implements `__init__(config: ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 640-660: Method `Step3VisionEmbeddings.forward`
```python
    def forward(self, pixel_values: torch.Tensor) -> torch.Tensor:
        batch_size = pixel_values.shape[0]
        patch_embeds = self.patch_embedding(
            pixel_values
        )  # shape = [*, width, grid, grid]
        patch_embeds = patch_embeds.flatten(2).transpose(1, 2)

        # pad
        class_embeds = self.class_embedding.expand(batch_size, 1, -1)
        embeddings = torch.cat([class_embeds, patch_embeds], dim=1)
        embeddings = embeddings + get_abs_pos(
            self.position_embedding(self.position_ids), patch_embeds.size(1)
        )
        embeddings = torch.cat(
            [
                embeddings[:, 0, :].unsqueeze(1).repeat(1, self.pad_tp_size - 1, 1),
                embeddings,
            ],
            dim=1,
        )
        return embeddings
```
**EN:** This method implements `forward(pixel_values: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(pixel_values: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 663-663: Class `Step3VisionEncoderLayer` overview
```python
class Step3VisionEncoderLayer(nn.Module):
```
**EN:** Defines `Step3VisionEncoderLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Step3VisionEncoderLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 664-677: Method `Step3VisionEncoderLayer.__init__`
```python
    def __init__(self, config, attn_implementation: str = "sdpa") -> None:
        super().__init__()
        self.embed_dim = config.hidden_size
        self.layer_norm1 = LayerNorm(self.embed_dim, eps=1e-6)
        self.layer_norm2 = LayerNorm(self.embed_dim, eps=1e-6)

        self.self_attn = Step3VisionAttention(
            self.embed_dim, num_heads=config.num_attention_heads
        )
        self.mlp = Step3VisionMLP(
            dim=self.embed_dim,
            intermediate_size=config.intermediate_size,
            hidden_act=config.hidden_act,
        )
```
**EN:** This method implements `__init__(config, attn_implementation: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config, attn_implementation: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 679-682: Method `Step3VisionEncoderLayer.forward`
```python
    def forward(self, hidden_states) -> torch.Tensor:
        hidden_states = hidden_states + self.layer_norm1(self.self_attn(hidden_states))
        hidden_states = hidden_states + self.layer_norm2(self.mlp(hidden_states))
        return hidden_states
```
**EN:** This method implements `forward(hidden_states)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 685-685: Class `Step3VisionTransformer` overview
```python
class Step3VisionTransformer(nn.Module):
```
**EN:** Defines `Step3VisionTransformer` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Step3VisionTransformer`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 686-691: Method `Step3VisionTransformer.__init__`
```python
    def __init__(self, config: Step3VisionEncoderConfig):
        super().__init__()
        self.config = config
        self.image_size = config.image_size
        self.embeddings = Step3VisionEmbeddings(config)
        self.transformer = Step3VisionEncoder(config)
```
**EN:** This method implements `__init__(config: ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 693-695: Method `Step3VisionTransformer.dtype`
```python
    @property
    def dtype(self) -> torch.dtype:
        return self.embeddings.patch_embedding.weight.dtype
```
**EN:** This method implements `dtype()` and implements a focused helper that supports the surrounding runtime flow inside `Step3VisionTransformer` Decorators: property.
**CN:** 这个方法实现了 `dtype()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Step3VisionTransformer` 内部调用 装饰器：property。

### Lines 697-703: Method `Step3VisionTransformer.forward`
```python
    def forward(
        self,
        pixel_values: torch.Tensor,
    ):
        hidden_states = self.embeddings(pixel_values)
        hidden_states = self.transformer(inputs_embeds=hidden_states)
        return hidden_states
```
**EN:** This method implements `forward(pixel_values: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(pixel_values: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 706-714: Class `Step3VisionEncoder` overview
```python
class Step3VisionEncoder(nn.Module):
    """
    Transformer encoder consisting of `config.num_hidden_layers` self attention layers. Each layer is a
    [`Step3VisionEncoderLayer`].

    Args:
        config: StepVisionEncoderConfig
    """
```
**EN:** Defines `Step3VisionEncoder` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Step3VisionEncoder`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 715-720: Method `Step3VisionEncoder.__init__`
```python
    def __init__(self, config: Step3VisionEncoderConfig):
        super().__init__()
        self.config = config
        self.layers = nn.ModuleList(
            [Step3VisionEncoderLayer(config) for _ in range(config.num_hidden_layers)]
        )
```
**EN:** This method implements `__init__(config: ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 722-733: Method `Step3VisionEncoder.forward`
```python
    def forward(
        self,
        inputs_embeds,
    ) -> torch.Tensor:

        hidden_states = inputs_embeds
        for encoder_layer in self.layers:
            hidden_states = encoder_layer(
                hidden_states,
            )

        return hidden_states
```
**EN:** This method implements `forward(inputs_embeds)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(inputs_embeds)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 736-737: Class `Step3VLForConditionalGeneration` overview
```python
class Step3VLForConditionalGeneration(nn.Module):
```
**EN:** Defines `Step3VLForConditionalGeneration` as a reusable runtime type derived from nn.Module. The class groups 9 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Step3VLForConditionalGeneration`，其继承关系为 nn.Module。这个类组织了 9 个方法，用于实现模型相关行为。

### Lines 738-790: Method `Step3VLForConditionalGeneration.__init__`
```python
    def __init__(
        self,
        config: Step3VLConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.quant_config = quant_config
        self.model = Step3TextModel(
            config.text_config, quant_config, prefix=add_prefix("model", prefix)
        )

        self.vision_model = Step3VisionTransformer(config.vision_config)

        self.vit_downsampler = nn.Conv2d(
            config.vision_config.hidden_size,
            config.vision_config.output_hidden_size,
            kernel_size=2,
            stride=config.understand_projector_stride,
        )
        self.vit_downsampler2 = nn.Conv2d(
            config.vision_config.output_hidden_size,
            config.vision_config.output_hidden_size * 2,
            kernel_size=3,
            stride=2,
            padding=1,
        )
        self.vit_large_projector = nn.Linear(
            config.vision_config.output_hidden_size * 2,
            config.hidden_size,
            bias=config.projector_bias,
        )

        # TODO: support shared experts fusion
        # self.n_shared_experts = 1
        # self.num_fused_shared_experts = (
        #     0
        #     if global_server_args.disable_shared_experts_fusion
        #     else self.n_shared_experts
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 792-793: Method `Step3VLForConditionalGeneration._get_vision_model_output`
```python
    def _get_vision_model_output(self, input_tensor: torch.Tensor) -> torch.Tensor:
        return self.vision_model(input_tensor)[:, 4:]
```
**EN:** This method implements `_get_vision_model_output(input_tensor: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `_get_vision_model_output(input_tensor: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 795-801: Method `Step3VLForConditionalGeneration._flatten_embeddings`
```python
    def _flatten_embeddings(self, embeddings) -> torch.Tensor:

        if isinstance(embeddings, torch.Tensor):
            # Flatten all but the last dimension.
            return embeddings.flatten(0, -2)

        return torch.cat(tuple(self._flatten_embeddings(t) for t in embeddings))
```
**EN:** This method implements `_flatten_embeddings(embeddings)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `_flatten_embeddings(embeddings)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 803-812: Method `Step3VLForConditionalGeneration._process_image_features`
```python
    def _process_image_features(self, image_features: torch.Tensor) -> torch.Tensor:
        B, P = image_features.shape[:2]
        HW = int(sqrt(P))
        image_features = image_features.permute(0, 2, 1).view(B, -1, HW, HW)
        image_features = self.vit_downsampler(image_features)
        image_features = self.vit_downsampler2(image_features)
        n_dim = image_features.size(1)
        image_features = image_features.view(B, n_dim, -1).permute(0, 2, 1)
        image_features = self.vit_large_projector(image_features)
        return image_features
```
**EN:** This method implements `_process_image_features(image_features: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Step3VLForConditionalGeneration`.
**CN:** 这个方法实现了 `_process_image_features(image_features: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Step3VLForConditionalGeneration` 内部调用。

### Lines 814-855: Method `Step3VLForConditionalGeneration.get_image_feature`
```python
    def get_image_feature(self, items: List[MultimodalDataItem]) -> torch.Tensor:
        assert len(items) == 1  # We only have images.

        item = items[0]
        pixel_values = item.feature.type(self.vision_model.dtype)
        num_patches = item.model_specific_data.get("num_patches")
        patch_pixel_values = item.model_specific_data.get("patch_pixel_values", None)
        if patch_pixel_values is not None:
            patch_pixel_values = patch_pixel_values.type(self.vision_model.dtype)

        if patch_pixel_values is not None:
            patch_pixel_values = patch_pixel_values.to("cuda")

        image_features = self._get_vision_model_output(pixel_values)
        patch_image_features = (
            self._get_vision_model_output(patch_pixel_values)
            if patch_pixel_values is not None
            else None
        )

        image_features = self._process_image_features(image_features)
        patch_image_features = (
            self._process_image_features(patch_image_features)
            if patch_image_features is not None
            else None
        )

        merged_image_features = []
        cur_patch_idx = 0
        for i, num_patch in enumerate(num_patches):
            cur_feature = []
            if num_patch > 0:
                patch_slice = patch_image_features[
                    cur_patch_idx : cur_patch_idx + num_patch
                ]
                cur_feature.append(patch_slice.view(-1, patch_slice.shape[-1]))
            cur_feature.append(image_features[i].view(-1, image_features.shape[-1]))
            cur_patch_idx += num_patch
            merged_image_features.append(
                torch.cat(cur_feature) if len(cur_feature) > 1 else cur_feature[0]
# ... truncated for brevity ...
```
**EN:** This method implements `get_image_feature(items: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_image_feature(items: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 857-859: Method `Step3VLForConditionalGeneration.pad_input_ids`
```python
    def pad_input_ids(self, input_ids: List[int], mm_inputs: MultimodalInputs):
        pattern = MultiModalityDataPaddingPatternMultimodalTokens()
        return pattern.pad_input_tokens(input_ids, mm_inputs)
```
**EN:** This method implements `pad_input_ids(input_ids: ..., mm_inputs: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Step3VLForConditionalGeneration`.
**CN:** 这个方法实现了 `pad_input_ids(input_ids: ..., mm_inputs: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Step3VLForConditionalGeneration` 内部调用。

### Lines 861-881: Method `Step3VLForConditionalGeneration.forward`
```python
    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
    ) -> torch.Tensor:
        hidden_states = general_mm_embed_routine(
            input_ids=input_ids,
            forward_batch=forward_batch,
            language_model=self.model,
            data_embedding_funcs={
                Modality.IMAGE: self.get_image_feature,
            },
            positions=positions,
        )

        return self.logits_processor(
            input_ids, hidden_states, self.lm_head, forward_batch
        )
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 883-992: Method `Step3VLForConditionalGeneration.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            (".qkv_proj", ".q_proj", 0),
            (".qkv_proj", ".k_proj", 1),
            (".qkv_proj", ".v_proj", 2),
            (".gate_up_proj", ".gate_proj", 0),
            (".gate_up_proj", ".up_proj", 1),
        ]

        if self.num_fused_shared_experts > 0:
            assert self.num_fused_shared_experts == 1
            log_info_on_rank0(logger, "Shared experts fusion optimization enabled.")

        expert_params_mapping = FusedMoE.make_expert_params_mapping(
            ckpt_gate_proj_name="gate_proj",
            ckpt_down_proj_name="down_proj",
            ckpt_up_proj_name="up_proj",
            num_experts=self.config.text_config.moe_num_experts
            + self.num_fused_shared_experts,
        )

        params_dict = dict(self.named_parameters())
        loaded_params = set()

        def match_expert_and_shard_ids(name_path: str, weight_path: str) -> bool:
            name_parts = name_path.split(".")
            weight_parts = weight_path.split(".")
            shard_id_matches = name_parts[4] == weight_parts[2]
            return shard_id_matches

        for name, loaded_weight in weights:
            if "vision_model" in name:
                name = name.replace("self_attn", "self_attn.attn")
                name = name.replace("out_proj", "proj")

            # TODO: support vision model
            if self.num_fused_shared_experts > 0 and "share" in name:
                # assert False
                name = name.replace("share_expert", "moe")
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 995-1001: Method `Step3VLForConditionalGeneration.get_model_config_for_expert_location`
```python
    @classmethod
    def get_model_config_for_expert_location(cls, config: Step3VLConfig):
        return ModelConfigForExpertLocation(
            num_layers=config.text_config.num_hidden_layers,
            num_logical_experts=config.text_config.moe_num_experts,
            num_groups=None,
        )
```
**EN:** This method implements `get_model_config_for_expert_location(config: ...)` and returns a cached or derived object used elsewhere in the pipeline Decorators: classmethod.
**CN:** 这个方法实现了 `get_model_config_for_expert_location(config: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果 装饰器：classmethod。

### Lines 1004-1004: Top-level assign
```python
EntryClass = Step3VLForConditionalGeneration
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `logging`
- `math`
- `math.sqrt`
- `typing.Any`
- `typing.Dict`
- `typing.Iterable`
- `typing.List`
- `typing.Optional`
- `typing.Tuple`
- `torch`
- `torch.nn`
- `torch.nn.LayerNorm`
- `torch.nn.functional`
- `transformers.PretrainedConfig`
- `transformers.activations.ACT2FN`
- `sglang.srt.configs.step3_vl.Step3TextConfig`
- `sglang.srt.configs.step3_vl.Step3VisionEncoderConfig`
- `sglang.srt.configs.step3_vl.Step3VLConfig`
- `sglang.srt.distributed.get_tensor_model_parallel_rank`
- `sglang.srt.distributed.get_tensor_model_parallel_world_size`
- `sglang.srt.distributed.tensor_model_parallel_all_reduce`
- `sglang.srt.eplb.expert_location.ModelConfigForExpertLocation`
- `sglang.srt.layers.activation.SiluAndMul`
- `sglang.srt.layers.attention.vision.VisionAttention`
- `sglang.srt.layers.communicator.LayerCommunicator`
- `sglang.srt.layers.communicator.LayerScatterModes`
- `sglang.srt.layers.conv.Conv2dLayer`
- `sglang.srt.layers.dp_attention.get_attention_tp_rank`
- `sglang.srt.layers.dp_attention.get_attention_tp_size`
- `sglang.srt.layers.dp_attention.is_dp_attention_enabled`
- `sglang.srt.layers.layernorm.RMSNorm`
- `sglang.srt.layers.linear.ColumnParallelLinear`
- `sglang.srt.layers.linear.MergedColumnParallelLinear`
- `sglang.srt.layers.linear.ReplicatedLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.moe.get_moe_a2a_backend`
- `sglang.srt.layers.moe.ep_moe.layer.get_moe_impl_class`
- `sglang.srt.layers.moe.fused_moe_triton.FusedMoE`
- `sglang.srt.layers.moe.topk.TopK`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.layers.rotary_embedding.get_rope`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`
- `sglang.srt.managers.mm_utils.MultiModalityDataPaddingPatternMultimodalTokens`
- `sglang.srt.managers.mm_utils.general_mm_embed_routine`
- `sglang.srt.managers.schedule_batch.Modality`
- `sglang.srt.managers.schedule_batch.MultimodalDataItem`
- `sglang.srt.managers.schedule_batch.MultimodalInputs`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.log_info_on_rank0`
- `sglang.srt.utils.make_layers`
- `sglang.srt.utils.hf_transformers_utils.get_rope_config`
