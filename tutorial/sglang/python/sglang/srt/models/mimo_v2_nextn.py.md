# mimo_v2_nextn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/mimo_v2_nextn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the mimo v2 nextn model stack used by SGLang SRT inference, including network blocks, runtime wrappers, and checkpoint-loading utilities. / 该模块实现 SGLang SRT 推理所使用的 mimo v2 nextn 模型栈，包括网络模块、运行时封装以及检查点加载工具。

## Line-by-Line Analysis / 逐行分析

### Lines 15-50: Module imports
```python
import logging
from typing import Iterable, Optional, Tuple

import torch
from torch import nn
from transformers import PretrainedConfig

from sglang.srt.configs.model_config import get_mimo_v2_fused_qkv_expected_tp_size
from sglang.srt.distributed import get_tensor_model_parallel_world_size
from sglang.srt.eplb.expert_distribution import get_global_expert_distribution_recorder
from sglang.srt.layers.communicator import (
    LayerCommunicator,
    LayerScatterModes,
    enable_moe_dense_fully_dp,
)
from sglang.srt.layers.dp_attention import (
    get_attention_tp_rank,
    is_dp_attention_enabled,
)
from sglang.srt.layers.layernorm import RMSNorm
from sglang.srt.layers.logits_processor import LogitsProcessor
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.layers.vocab_parallel_embedding import (
    ParallelLMHead,
    VocabParallelEmbedding,
)
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.model_loader.weight_utils import default_weight_loader
from sglang.srt.models.mimo_v2 import (
    MiMoV2Attention,
    MiMoV2ForCausalLM,
    MiMoV2MLP,
    load_mimo_v2_qkv_proj_weight,
)
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import add_prefix
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 52-52: Top-level assign
```python
MiMoV2Config = None
```
**EN:** Defines or updates MiMoV2Config, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 MiMoV2Config，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 54-54: Top-level assign
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines or updates logger, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 logger，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 57-57: Class `MiMoV2MTPLayer` overview
```python
class MiMoV2MTPLayer(nn.Module):
```
**EN:** Defines `MiMoV2MTPLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiMoV2MTPLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 58-132: Method `MiMoV2MTPLayer.__init__`
```python
    def __init__(
        self,
        config: MiMoV2Config,
        layer_id: int = 0,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.hidden_size = config.hidden_size

        rope_theta = getattr(config, "rope_theta", 10000)
        rope_scaling = getattr(config, "rope_scaling", None)
        if (
            isinstance(rope_scaling, dict)
            and rope_scaling.get("rope_type") == "default"
        ):
            rope_scaling = None
        max_position_embeddings = getattr(
            config,
            "context_len",
            getattr(config, "max_position_embeddings", 32768),
        )

        self.self_attn = MiMoV2Attention(
            hidden_size=self.hidden_size,
            num_heads=config.swa_num_attention_heads,
            num_kv_heads=config.swa_num_key_value_heads,
            head_dim=config.swa_head_dim,
            v_head_dim=getattr(config, "swa_v_head_dim", None),
            v_scale=getattr(config, "attention_value_scale", None),
            sliding_window_size=config.sliding_window_size,
            attention_bias=config.attention_bias,
            attention_sink_bias=getattr(config, "add_swa_attention_sink_bias", False),
            layer_id=layer_id,
            rope_theta=getattr(config, "swa_rope_theta", rope_theta),
            rope_scaling=rope_scaling,
            max_position_embeddings=max_position_embeddings,
            quant_config=quant_config,
            partial_rotary_factor=getattr(config, "partial_rotary_factor", 1.0),
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_id: ...=..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ...=..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 134-162: Method `MiMoV2MTPLayer.forward`
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
        with get_global_expert_distribution_recorder().disable_this_region():
            hidden_states = self.mlp(hidden_states)
        hidden_states, residual = self.layer_communicator.postprocess_layer(
            hidden_states, residual, forward_batch
        )

        return hidden_states, residual
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 165-165: Class `MiMoV2ModelNextN` overview
```python
class MiMoV2ModelNextN(nn.Module):
```
**EN:** Defines `MiMoV2ModelNextN` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiMoV2ModelNextN`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 166-194: Method `MiMoV2ModelNextN.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.vocab_size = config.vocab_size

        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size,
            config.hidden_size,
            use_attn_tp_group=is_dp_attention_enabled(),
            prefix=add_prefix("embed_tokens", prefix),
        )

        self.enorm = RMSNorm(config.hidden_size, eps=config.layernorm_epsilon)
        self.hnorm = RMSNorm(config.hidden_size, eps=config.layernorm_epsilon)

        self.eh_proj = nn.Linear(2 * config.hidden_size, config.hidden_size, bias=False)

        self.mtp_block = MiMoV2MTPLayer(
            config,
            0,
            quant_config=quant_config,
            prefix=add_prefix("decoder", prefix),
        )
        self.final_layernorm = RMSNorm(config.hidden_size, eps=config.layernorm_epsilon)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 196-234: Method `MiMoV2ModelNextN.forward`
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
        if hidden_states.shape[0] > 0:
            hidden_states = self.eh_proj(
                torch.cat(
                    (
                        self.enorm(hidden_states),
                        self.hnorm(forward_batch.spec_info.hidden_states),
                    ),
                    dim=-1,
                )
            )
        hidden_states, residual = self.mtp_block(
            positions=positions,
            hidden_states=hidden_states,
            forward_batch=forward_batch,
            residual=None,
        )
        hidden_states_before_norm = None
        if not forward_batch.forward_mode.is_idle():
            if forward_batch.return_hidden_states_before_norm:
                hidden_states_before_norm = (
                    hidden_states if residual is None else hidden_states + residual
                )
            if residual is not None:
                hidden_states, _ = self.final_layernorm(hidden_states, residual)
            else:
                hidden_states = self.final_layernorm(hidden_states)

        return hidden_states, hidden_states_before_norm
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 237-238: Class `MiMoV2MTP` overview
```python
class MiMoV2MTP(MiMoV2ForCausalLM):
```
**EN:** Defines `MiMoV2MTP` as a reusable runtime type derived from MiMoV2ForCausalLM. The class groups 6 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiMoV2MTP`，其继承关系为 MiMoV2ForCausalLM。这个类组织了 6 个方法，用于实现模型相关行为。

### Lines 239-261: Method `MiMoV2MTP.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        draft_model_idx: Optional[int] = None,
        prefix: str = "",
    ) -> None:
        nn.Module.__init__(self)
        self.config = config
        self.tp_size = get_tensor_model_parallel_world_size()
        self.quant_config = quant_config

        self.model = MiMoV2ModelNextN(
            config, quant_config, prefix=add_prefix("model", prefix)
        )
        self.lm_head = ParallelLMHead(
            config.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=add_prefix("lm_head", prefix),
            use_attn_tp_group=get_global_server_args().enable_dp_lm_head,
        )
        self.logits_processor = LogitsProcessor(config)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., draft_model_idx: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., draft_model_idx: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 263-279: Method `MiMoV2MTP.forward`
```python
    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        hidden_states, hidden_states_before_norm = self.model(
            input_ids, positions, forward_batch
        )
        return self.logits_processor(
            input_ids,
            hidden_states,
            self.lm_head,
            forward_batch,
            hidden_states_before_norm=hidden_states_before_norm,
        )
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 281-358: Method `MiMoV2MTP.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]], is_nextn=False):
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
        ]

        params_dict = dict(self.named_parameters())
        for name, loaded_weight in weights:
            if "rotary_emb.inv_freq" in name or "projector" in name:
                continue
            if "rotary_emb.cos_cached" in name or "rotary_emb.sin_cached" in name:
                # Models trained using ColossalAI may include these tensors in
                # the checkpoint. Skip them.
                continue
            if self.config.tie_word_embeddings and "lm_head.weight" in name:
                continue
            if name.startswith("model.vision_tower") and name not in params_dict:
                continue
            name = self.map_model_name_to_mtp_param_name(name)

            # Support fused qkv_proj checkpoint (Pro format)
            if "qkv_proj" in name:
                if name in params_dict:
                    param = params_dict[name]
                    load_mimo_v2_qkv_proj_weight(
                        name,
                        param,
                        loaded_weight,
                        expected_fused_tp_size=get_mimo_v2_fused_qkv_expected_tp_size(
                            self.config
                        ),
                    )
                continue

            for param_name, weight_name, shard_id in stacked_params_mapping:

# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ..., is_nextn=...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ..., is_nextn=...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 360-380: Method `MiMoV2MTP.map_model_name_to_mtp_param_name`
```python
    def map_model_name_to_mtp_param_name(self, name: str) -> str:
        import re

        if "pre_mlp_layernorm" in name:
            name = name.replace("pre_mlp_layernorm", "post_attention_layernorm")

        name_without_prefix = [
            "enorm",
            "hnorm",
            "eh_proj",
            "final_layernorm",
        ]
        pattern = r"model.mtp.layers.(\d+)."
        group = re.match(pattern, name)
        if group is not None:
            for sub_name in name_without_prefix:
                if sub_name in name:
                    name = name.replace(group.group(), "model.")
                    return name
            name = name.replace(group.group(), "model.mtp_block.")
        return name
```
**EN:** This method implements `map_model_name_to_mtp_param_name(name: ...)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2MTP`.
**CN:** 这个方法实现了 `map_model_name_to_mtp_param_name(name: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2MTP` 内部调用。

### Lines 382-383: Method `MiMoV2MTP.get_embed_and_head`
```python
    def get_embed_and_head(self):
        return self.model.embed_tokens.weight, self.lm_head.weight
```
**EN:** This method implements `get_embed_and_head()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_embed_and_head()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 385-391: Method `MiMoV2MTP.set_embed_and_head`
```python
    def set_embed_and_head(self, embed, head):
        del self.model.embed_tokens.weight
        del self.lm_head.weight
        self.model.embed_tokens.weight = embed
        self.lm_head.weight = head
        torch.cuda.empty_cache()
        torch.cuda.synchronize()
```
**EN:** This method implements `set_embed_and_head(embed, head)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `set_embed_and_head(embed, head)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 394-394: Top-level assign
```python
EntryClass = MiMoV2MTP
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
- `typing.Iterable`
- `typing.Optional`
- `typing.Tuple`
- `torch`
- `torch.nn`
- `transformers.PretrainedConfig`
- `sglang.srt.configs.model_config.get_mimo_v2_fused_qkv_expected_tp_size`
- `sglang.srt.distributed.get_tensor_model_parallel_world_size`
- `sglang.srt.eplb.expert_distribution.get_global_expert_distribution_recorder`
- `sglang.srt.layers.communicator.LayerCommunicator`
- `sglang.srt.layers.communicator.LayerScatterModes`
- `sglang.srt.layers.communicator.enable_moe_dense_fully_dp`
- `sglang.srt.layers.dp_attention.get_attention_tp_rank`
- `sglang.srt.layers.dp_attention.is_dp_attention_enabled`
- `sglang.srt.layers.layernorm.RMSNorm`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.models.mimo_v2.MiMoV2Attention`
- `sglang.srt.models.mimo_v2.MiMoV2ForCausalLM`
- `sglang.srt.models.mimo_v2.MiMoV2MLP`
- `sglang.srt.models.mimo_v2.load_mimo_v2_qkv_proj_weight`
- `sglang.srt.server_args.get_global_server_args`
- `sglang.srt.utils.add_prefix`
- `re`
