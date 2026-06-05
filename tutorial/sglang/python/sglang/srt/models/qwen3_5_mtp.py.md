# qwen3_5_mtp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/qwen3_5_mtp.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Inference-only Qwen3_5 MTP model. This file provides runtime-ready implementation details for SGLang inference. / 该文件围绕上述主题实现了可直接用于 SGLang 推理运行时的具体逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 15-15: Module docstring
```python
"""Inference-only Qwen3_5 MTP model."""
```
**EN:** States the module scope and documents the high-level intention of the implementation.
**CN:** 说明模块范围，并给出实现的高层设计意图。

### Lines 17-35: Module imports
```python
import logging
from typing import Iterable, Optional, Tuple

import torch
from torch import nn
from transformers import PretrainedConfig

from sglang.srt.distributed import get_pp_group, get_tensor_model_parallel_world_size
from sglang.srt.eplb.expert_distribution import get_global_expert_distribution_recorder
from sglang.srt.eplb.expert_location import ModelConfigForExpertLocation
from sglang.srt.layers.layernorm import GemmaRMSNorm
from sglang.srt.layers.logits_processor import LogitsProcessor
from sglang.srt.layers.moe.fused_moe_triton.layer import FusedMoE
from sglang.srt.layers.vocab_parallel_embedding import ParallelLMHead
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.model_loader.weight_utils import default_weight_loader
from sglang.srt.models.qwen3_5 import Qwen3_5ForCausalLM
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import add_prefix, is_npu
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 37-37: Top-level assign
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines or updates logger, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 logger，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 40-41: Class `Qwen3_5ForCausalLMMTP` overview
```python
class Qwen3_5ForCausalLMMTP(nn.Module):
```
**EN:** Defines `Qwen3_5ForCausalLMMTP` as a reusable runtime type derived from nn.Module. The class groups 6 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3_5ForCausalLMMTP`，其继承关系为 nn.Module。这个类组织了 6 个方法，用于实现模型相关行为。

### Lines 42-106: Method `Qwen3_5ForCausalLMMTP.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config=None,
        prefix: str = "",
    ) -> None:
        nn.Module.__init__(self)

        self.is_multimodal = hasattr(config, "text_config")
        if self.is_multimodal:
            config = config.text_config

        # The MTP model is unquantized in the nvfp4 checkpoint.
        if quant_config and quant_config.get_name() == "modelopt_fp4":
            quant_config = None
        if (
            is_npu()
            and get_global_server_args().speculative_draft_model_quantization is None
        ):
            quant_config = None

        # Quark-quantized Qwen3.5 MXFP4 checkpoints ship the MTP module in
        # bf16; every `mtp.*` layer appears under the quantization exclude
        # list. Detect that and skip quantization here so linear/MoE weight
        # loaders allocate bf16 shapes (see sgl-project/sglang#23113).
        if quant_config and quant_config.get_name() == "quark":
            exclude_layers = getattr(quant_config, "exclude_layers", [])
            if any(
                isinstance(layer, str) and layer.startswith("mtp.")
                for layer in exclude_layers
            ):
                quant_config = None

        self.config = config
        self.tp_size = get_tensor_model_parallel_world_size()
        self.quant_config = quant_config
        self.pp_group = get_pp_group()

        self.fc = nn.Linear(2 * config.hidden_size, config.hidden_size, bias=False)
        RMSNorm_cls = GemmaRMSNorm
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 108-115: Method `Qwen3_5ForCausalLMMTP.get_model_config_for_expert_location`
```python
    @classmethod
    def get_model_config_for_expert_location(cls, config):
        text_config = getattr(config, "text_config", config)
        return ModelConfigForExpertLocation(
            num_layers=text_config.num_hidden_layers,
            num_logical_experts=text_config.num_experts,
            num_groups=None,
        )
```
**EN:** This method implements `get_model_config_for_expert_location(config)` and returns a cached or derived object used elsewhere in the pipeline Decorators: classmethod.
**CN:** 这个方法实现了 `get_model_config_for_expert_location(config)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果 装饰器：classmethod。

### Lines 117-118: Method `Qwen3_5ForCausalLMMTP.get_embed_and_head`
```python
    def get_embed_and_head(self):
        return self.model.embed_tokens.weight, self.lm_head.weight
```
**EN:** This method implements `get_embed_and_head()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_embed_and_head()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 120-128: Method `Qwen3_5ForCausalLMMTP.set_embed_and_head`
```python
    def set_embed_and_head(self, embed, head):
        del self.model.embed_tokens.weight
        if not self.config.tie_word_embeddings:
            del self.lm_head.weight

        self.model.embed_tokens.weight = embed
        self.lm_head.weight = head
        torch.cuda.empty_cache()
        torch.cuda.synchronize()
```
**EN:** This method implements `set_embed_and_head(embed, head)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `set_embed_and_head(embed, head)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 130-174: Method `Qwen3_5ForCausalLMMTP.forward`
```python
    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: Optional[torch.Tensor] = None,
        **kwargs,
    ):

        assert input_embeds is None
        input_embeds = forward_batch.mm_input_embeds
        if (
            forward_batch.forward_mode.is_extend()
            and forward_batch.contains_mm_inputs()
            and not forward_batch.forward_mode.is_draft_extend(include_v2=True)
        ):
            assert input_embeds is not None
            input_embeds = torch.cat(
                [input_embeds[:-1], self.model.embed_tokens(input_ids[-1].unsqueeze(0))]
            )

        if input_embeds is None:
            input_embeds = self.model.embed_tokens(input_ids)

        hidden_states = forward_batch.spec_info.hidden_states

        if not forward_batch.forward_mode.is_idle():
            input_embeds = self.pre_fc_norm_embedding(input_embeds)
            hidden_states = self.pre_fc_norm_hidden(hidden_states)
        hidden_states = torch.cat([input_embeds, hidden_states], dim=-1)

        hidden_states = self.fc(hidden_states)

        with get_global_expert_distribution_recorder().disable_this_region():
            hidden_states = self.model(
                input_ids,
                positions,
                forward_batch,
                hidden_states,
# ... truncated for brevity ...
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., **kwargs)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., **kwargs)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 176-376: Method `Qwen3_5ForCausalLMMTP.load_weights`
```python
    def load_weights(
        self, weights: Iterable[Tuple[str, torch.Tensor]], is_mtp: bool = False
    ):
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
        ]

        # Params for MoE experts (non-fused/fused)
        num_experts = getattr(self.config, "num_experts", None)
        if num_experts is not None:
            expert_params_mapping = FusedMoE.make_expert_params_mapping(
                ckpt_gate_proj_name="gate_proj",
                ckpt_down_proj_name="down_proj",
                ckpt_up_proj_name="up_proj",
                num_experts=num_experts,
            )
        else:
            expert_params_mapping = []

        # Skip loading extra parameters for GPTQ/modelopt models.
        ignore_suffixes = (
            ".bias",
            "_bias",
            ".k_scale",
            "_k_scale",
            ".v_scale",
            "_v_scale",
            ".weight_scale",
            "_weight_scale",
            ".input_scale",
            "_input_scale",
        )

        # fused experts: experts.w13_weight / experts.w2_weight
        is_fused_expert = False
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ..., is_mtp: ...=...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ..., is_mtp: ...=...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 379-379: Top-level assign
```python
EntryClass = [Qwen3_5ForCausalLMMTP]
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
- `sglang.srt.distributed.get_pp_group`
- `sglang.srt.distributed.get_tensor_model_parallel_world_size`
- `sglang.srt.eplb.expert_distribution.get_global_expert_distribution_recorder`
- `sglang.srt.eplb.expert_location.ModelConfigForExpertLocation`
- `sglang.srt.layers.layernorm.GemmaRMSNorm`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.moe.fused_moe_triton.layer.FusedMoE`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.models.qwen3_5.Qwen3_5ForCausalLM`
- `sglang.srt.server_args.get_global_server_args`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.is_npu`
