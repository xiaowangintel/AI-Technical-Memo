# step3p5_mtp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/step3p5_mtp.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the step3p5 mtp model stack used by SGLang SRT inference, including network blocks, runtime wrappers, and checkpoint-loading utilities. / 该模块实现 SGLang SRT 推理所使用的 step3p5 mtp 模型栈，包括网络模块、运行时封装以及检查点加载工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20: Module imports
```python
import logging
from collections.abc import Iterable
from typing import Optional

import torch
import torch.nn as nn
from transformers import PretrainedConfig

from sglang.srt.distributed import get_tensor_model_parallel_world_size
from sglang.srt.layers.layernorm import GemmaRMSNorm
from sglang.srt.layers.logits_processor import LogitsProcessor
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.layers.vocab_parallel_embedding import (
    ParallelLMHead,
    VocabParallelEmbedding,
)
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.model_loader.weight_utils import default_weight_loader
from sglang.srt.models.step3p5 import Step3p5DecoderLayer, Step3p5ForCausalLM
from sglang.srt.utils import add_prefix
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 22-22: Top-level assign
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines or updates logger, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 logger，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 25-40: Function `get_spec_layer_idx_from_weight_name`
```python
def get_spec_layer_idx_from_weight_name(
    config: PretrainedConfig, weight_name: str
) -> Optional[int]:
    """Return MTP/nextn layer index if this weight belongs to spec layers.

    Step3p5 MTP/nextn checkpoints append extra layers after the main decoder:
      model.layers.[num_hidden_layers ... num_hidden_layers + num_nextn_predict_layers)
    """
    if hasattr(config, "num_nextn_predict_layers") and (
        getattr(config, "num_nextn_predict_layers", 0) > 0
    ):
        base = config.num_hidden_layers
        for i in range(config.num_nextn_predict_layers):
            if weight_name.startswith(f"model.layers.{base + i}."):
                return base + i
    return None
```
**EN:** This function implements `get_spec_layer_idx_from_weight_name(config: ..., weight_name: ...)` and Return MTP/nextn layer index if this weight belongs to spec layers.
**CN:** 这个函数实现了 `get_spec_layer_idx_from_weight_name(config: ..., weight_name: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 43-44: Class `SharedHead` overview
```python
class SharedHead(nn.Module):
```
**EN:** Defines `SharedHead` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `SharedHead`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 45-55: Method `SharedHead.__init__`
```python
    def __init__(
        self,
        config,
        quant_config=None,
    ) -> None:
        super().__init__()
        self.norm = GemmaRMSNorm(config.hidden_size, config.rms_norm_eps)
        self.head = ParallelLMHead(
            config.vocab_size, config.hidden_size, quant_config=quant_config
        )
        self.lm_head = self.head
```
**EN:** This method implements `__init__(config, quant_config=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config, quant_config=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 57-58: Method `SharedHead.forward`
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        return self.norm(hidden_states)
```
**EN:** This method implements `forward(hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 61-61: Class `Step3p5AMultiTokenPredictor` overview
```python
class Step3p5AMultiTokenPredictor(nn.Module):
```
**EN:** Defines `Step3p5AMultiTokenPredictor` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Step3p5AMultiTokenPredictor`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 62-86: Method `Step3p5AMultiTokenPredictor.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size,
            config.hidden_size,
        )
        self.mtp_start_layer_idx = config.num_hidden_layers
        self.num_mtp_layers = config.num_nextn_predict_layers

        layer_id = 45  # FIXME

        self.enorm = GemmaRMSNorm(config.hidden_size, config.rms_norm_eps)
        self.hnorm = GemmaRMSNorm(config.hidden_size, config.rms_norm_eps)
        self.eh_proj = nn.Linear(config.hidden_size * 2, config.hidden_size, bias=False)
        self.shared_head = SharedHead(config=config, quant_config=quant_config)
        self.mtp_block = Step3p5DecoderLayer(
            config=config, layer_id=layer_id, prefix=f"{prefix}.mtp_block"
        )
        self.lm_head = self.shared_head.head
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 88-127: Method `Step3p5AMultiTokenPredictor.forward`
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
            # if forward_batch.return_hidden_states_before_norm:
            hidden_states_before_norm = (
                hidden_states if residual is None else hidden_states + residual
            )
            if residual is not None:
                hidden_states, _ = self.shared_head.norm(hidden_states, residual)
            else:
                hidden_states = self.shared_head.norm(hidden_states)

        return hidden_states, hidden_states_before_norm
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 129-130: Method `Step3p5AMultiTokenPredictor.embed_input_ids`
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.embed_tokens(input_ids)
```
**EN:** This method implements `embed_input_ids(input_ids: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `embed_input_ids(input_ids: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 141-141: Class `Step3p5MTP` overview
```python
class Step3p5MTP(Step3p5ForCausalLM):
```
**EN:** Defines `Step3p5MTP` as a reusable runtime type derived from Step3p5ForCausalLM. The class groups 7 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Step3p5MTP`，其继承关系为 Step3p5ForCausalLM。这个类组织了 7 个方法，用于实现模型相关行为。

### Lines 142-159: Method `Step3p5MTP.__init__`
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
        self.draft_model_idx = draft_model_idx

        self.model = Step3p5AMultiTokenPredictor(
            config=config, quant_config=quant_config, prefix=add_prefix("model", prefix)
        )
        self.logits_processor = LogitsProcessor(config)
        self.lm_head = self.model.lm_head
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., draft_model_idx: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., draft_model_idx: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 161-162: Method `Step3p5MTP.embed_input_ids`
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.model.embed_input_ids(input_ids)
```
**EN:** This method implements `embed_input_ids(input_ids: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `embed_input_ids(input_ids: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 164-179: Method `Step3p5MTP.forward`
```python
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
            self.model.shared_head.head,
            forward_batch,
            hidden_states_before_norm=hidden_states_before_norm,
        )
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 181-182: Method `Step3p5MTP.get_embed_and_head`
```python
    def get_embed_and_head(self):
        return self.model.embed_tokens.weight, self.model.shared_head.head.weight
```
**EN:** This method implements `get_embed_and_head()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_embed_and_head()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 184-185: Method `Step3p5MTP.set_embed_and_head`
```python
    def set_embed_and_head(self, embed, head):
        return
```
**EN:** This method implements `set_embed_and_head(embed, head)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `set_embed_and_head(embed, head)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 187-293: Method `Step3p5MTP.load_weights`
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
        ]

        expert_params_mapping = [
            (".moe.experts.w13_weight", ".moe.gate_proj.weight", "w1"),
            (".moe.experts.w13_weight", ".moe.up_proj.weight", "w3"),
            (".moe.experts.w2_weight", ".moe.down_proj.weight", "w2"),
        ]

        params_dict = dict(self.named_parameters())
        loaded_params: set[str] = set()
        for name, loaded_weight in weights:
            if "rotary_emb.inv_freq" in name:
                continue
            spec_layer = get_spec_layer_idx_from_weight_name(self.config, name)
            if spec_layer is not None and spec_layer != (
                self.config.num_hidden_layers + self.draft_model_idx
            ):
                continue
            if "embed_tokens" not in name and spec_layer is None:
                continue
            name = self._rewrite_spec_layer_name(spec_layer, name)
            for param_name, weight_name, shard_id in stacked_params_mapping:
                # Skip non-stacked layers and experts (experts handled below).
                if weight_name not in name:
                    continue
                # We have mlp.experts[0].gate_proj in the checkpoint.
                # Since we handle the experts below in expert_params_mapping,
                # we need to skip here BEFORE we update the name, otherwise
                # name will be updated to mlp.experts[0].gate_up_proj, which
                # will then be updated below in expert_params_mapping
                # for mlp.experts[0].gate_gate_up_proj, which breaks load.
                if ("mlp.experts." in name) and name not in params_dict:
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 295-332: Method `Step3p5MTP._rewrite_spec_layer_name`
```python
    def _rewrite_spec_layer_name(self, spec_layer: Optional[int], name: str) -> str:
        """
        Rewrite the weight name to match the format of the original model.
        Add .mtp_block for modules in transformer layer block for spec layer
        """
        if spec_layer is None:
            return name

        # Some checkpoints place MTP weights under "model.layers.<id>.transformer.*".
        # Our modules use "model.layers.<id>.*", so drop the ".transformer." segment.
        transformer_prefix = f"model.layers.{spec_layer}.transformer."
        if name.startswith(transformer_prefix):
            name = name.replace(".transformer.", ".", 1)

        spec_layer_weight_names = [
            "embed_tokens",
            "enorm",
            "hnorm",
            "eh_proj",
            "shared_head",
        ]
        spec_layer_weight = False
        for weight_name in spec_layer_weight_names:
            if weight_name in name:
                spec_layer_weight = True
                break
        if not spec_layer_weight:
            # treat rest weights as weights for transformer layer block
            name = name.replace(
                f"model.layers.{spec_layer}.", f"model.layers.{spec_layer}.mtp_block."
            )

        # NEW: drop "layers.<idx>." from the rewritten name (minimal change).
        layers_prefix = f"model.layers.{spec_layer}."
        if name.startswith(layers_prefix):
            name = name.replace(layers_prefix, "model.", 1)

        return name
```
**EN:** This method implements `_rewrite_spec_layer_name(spec_layer: ..., name: ...)` and Rewrite the weight name to match the format of the original model.
**CN:** 这个方法实现了 `_rewrite_spec_layer_name(spec_layer: ..., name: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 335-335: Top-level assign
```python
EntryClass = [Step3p5MTP]
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
- `collections.abc.Iterable`
- `typing.Optional`
- `torch`
- `torch.nn`
- `transformers.PretrainedConfig`
- `sglang.srt.distributed.get_tensor_model_parallel_world_size`
- `sglang.srt.layers.layernorm.GemmaRMSNorm`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.models.step3p5.Step3p5DecoderLayer`
- `sglang.srt.models.step3p5.Step3p5ForCausalLM`
- `sglang.srt.utils.add_prefix`
