# qwen3_asr.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/qwen3_asr.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Qwen3-ASR model compatible with HuggingFace weights. This file provides runtime-ready implementation details for SGLang inference. / 该文件围绕上述主题实现了可直接用于 SGLang 推理运行时的具体逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module docstring
```python
"""Qwen3-ASR model compatible with HuggingFace weights"""
```
**EN:** States the module scope and documents the high-level intention of the implementation.
**CN:** 说明模块范围，并给出实现的高层设计意图。

### Lines 3-25: Module imports
```python
import logging
from typing import Any, Iterable, List, Optional, Tuple

import torch
import torch.nn as nn

from sglang.srt.configs.qwen3_asr import Qwen3ASRConfig
from sglang.srt.configs.qwen3_omni import Qwen3OmniMoeAudioEncoderConfig
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.managers.mm_utils import (
    MultiModalityDataPaddingPatternMultimodalTokens,
    general_mm_embed_routine,
)
from sglang.srt.managers.schedule_batch import (
    Modality,
    MultimodalDataItem,
    MultimodalInputs,
)
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.model_loader.weight_utils import default_weight_loader
from sglang.srt.models.qwen3 import Qwen3ForCausalLM
from sglang.srt.models.qwen3_omni_moe import Qwen3OmniMoeAudioEncoder
from sglang.srt.utils import add_prefix
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、SGLang 运行时组件。

### Lines 27-27: Top-level assign
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines or updates logger, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 logger，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 30-30: Class `Qwen3ASRForConditionalGeneration` overview
```python
class Qwen3ASRForConditionalGeneration(nn.Module):
```
**EN:** Defines `Qwen3ASRForConditionalGeneration` as a reusable runtime type derived from nn.Module. The class groups 5 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3ASRForConditionalGeneration`，其继承关系为 nn.Module。这个类组织了 5 个方法，用于实现模型相关行为。

### Lines 31-46: Class `Qwen3ASRForConditionalGeneration` attributes
```python
    default_bitsandbytes_target_modules = [
        ".gate_proj.",
        ".down_proj.",
        ".up_proj.",
        ".q_proj.",
        ".k_proj.",
        ".v_proj.",
        ".o_proj.",
    ]
    bitsandbytes_stacked_params_mapping = {
        "q_proj": ("qkv_proj", 0),
        "k_proj": ("qkv_proj", 1),
        "v_proj": ("qkv_proj", 2),
        "gate_proj": ("gate_up_proj", 0),
        "up_proj": ("gate_up_proj", 1),
    }
```
**EN:** Defines class-level attributes and metadata that shape how `Qwen3ASRForConditionalGeneration` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `Qwen3ASRForConditionalGeneration` 在运行时的行为。

### Lines 48-67: Method `Qwen3ASRForConditionalGeneration.__init__`
```python
    def __init__(
        self,
        config: Qwen3ASRConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        thinker_config = config.thinker_config

        if getattr(thinker_config, "audio_config", None) is None:
            thinker_config.audio_config = Qwen3OmniMoeAudioEncoderConfig()

        self.audio_tower = Qwen3OmniMoeAudioEncoder(thinker_config.audio_config)
        self.language_model = Qwen3ForCausalLM(
            thinker_config.text_config,
            quant_config,
            prefix=add_prefix("language_model", prefix),
        )
        self.pattern = MultiModalityDataPaddingPatternMultimodalTokens()
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 69-70: Method `Qwen3ASRForConditionalGeneration.pad_input_ids`
```python
    def pad_input_ids(self, input_ids: List[int], mm_inputs: MultimodalInputs):
        return self.pattern.pad_input_tokens(input_ids, mm_inputs)
```
**EN:** This method implements `pad_input_ids(input_ids: ..., mm_inputs: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3ASRForConditionalGeneration`.
**CN:** 这个方法实现了 `pad_input_ids(input_ids: ..., mm_inputs: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3ASRForConditionalGeneration` 内部调用。

### Lines 72-109: Method `Qwen3ASRForConditionalGeneration.get_audio_feature`
```python
    def get_audio_feature(self, items: List[MultimodalDataItem]) -> torch.Tensor:
        device = next(self.audio_tower.parameters()).device

        input_features = (
            torch.cat([item.feature for item in items])
            .type(self.audio_tower.dtype)
            .to(device)
        )

        has_mask = all(
            getattr(item, "feature_attention_mask", None) is not None for item in items
        )

        if has_mask:
            feature_attention_mask = (
                torch.cat([item.feature_attention_mask for item in items], dim=0)
                .type(torch.long)
                .to(device)
            )
            audio_feature_lengths = torch.sum(feature_attention_mask, dim=1)
            input_features = input_features.permute(0, 2, 1)[
                feature_attention_mask.bool()
            ].permute(1, 0)
        else:
            audio_feature_lengths = torch.tensor(
                [input_features.shape[-1]] * input_features.shape[0],
                dtype=torch.long,
                device=device,
            )
            input_features = input_features.permute(0, 2, 1).reshape(
                -1, input_features.shape[1]
            )

        audio_outputs = self.audio_tower(
            input_features,
            feature_lens=audio_feature_lengths,
        )
        return audio_outputs.last_hidden_state
```
**EN:** This method implements `get_audio_feature(items: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_audio_feature(items: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 111-127: Method `Qwen3ASRForConditionalGeneration.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        **kwargs: Any,
    ) -> torch.Tensor:
        hidden_states = general_mm_embed_routine(
            input_ids=input_ids,
            forward_batch=forward_batch,
            language_model=self.language_model,
            data_embedding_funcs={
                Modality.AUDIO: self.get_audio_feature,
            },
            positions=positions,
        )
        return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., **kwargs)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., **kwargs)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 129-196: Method `Qwen3ASRForConditionalGeneration.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        llm_stacked_params = [
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
        ]
        # Audio tower has separate q/k/v in checkpoint → stack into qkv_proj
        audio_stacked_params = [
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
        ]
        params_dict = dict(self.named_parameters(remove_duplicate=False))

        for name, loaded_weight in weights:
            if "rotary_emb.inv_freq" in name:
                continue
            if "rotary_emb.cos_cached" in name or "rotary_emb.sin_cached" in name:
                continue

            if (
                getattr(
                    self.config.thinker_config.text_config, "tie_word_embeddings", False
                )
                and "lm_head.weight" in name
            ):
                continue

            if "talker" in name or "code2wav" in name:
                continue

            if name.startswith("thinker.audio_tower."):
                name = name.replace("thinker.audio_tower.", "audio_tower.", 1)
            elif name.startswith("thinker.lm_head."):
                name = name.replace("thinker.lm_head.", "language_model.lm_head.", 1)
            elif name.startswith("thinker.model."):
                name = name.replace("thinker.model.", "language_model.model.", 1)

# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 199-199: Top-level assign
```python
EntryClass = Qwen3ASRForConditionalGeneration
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
- `typing.Any`
- `typing.Iterable`
- `typing.List`
- `typing.Optional`
- `typing.Tuple`
- `torch`
- `torch.nn`
- `sglang.srt.configs.qwen3_asr.Qwen3ASRConfig`
- `sglang.srt.configs.qwen3_omni.Qwen3OmniMoeAudioEncoderConfig`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.managers.mm_utils.MultiModalityDataPaddingPatternMultimodalTokens`
- `sglang.srt.managers.mm_utils.general_mm_embed_routine`
- `sglang.srt.managers.schedule_batch.Modality`
- `sglang.srt.managers.schedule_batch.MultimodalDataItem`
- `sglang.srt.managers.schedule_batch.MultimodalInputs`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.models.qwen3.Qwen3ForCausalLM`
- `sglang.srt.models.qwen3_omni_moe.Qwen3OmniMoeAudioEncoder`
- `sglang.srt.utils.add_prefix`
