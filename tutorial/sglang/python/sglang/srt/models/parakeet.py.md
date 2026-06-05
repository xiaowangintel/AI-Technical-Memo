# parakeet.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/parakeet.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the parakeet model stack used by SGLang SRT inference, including network blocks, runtime wrappers, and checkpoint-loading utilities. / 该模块实现 SGLang SRT 推理所使用的 parakeet 模型栈，包括网络模块、运行时封装以及检查点加载工具。

## Line-by-Line Analysis / 逐行分析

### Lines 20-32: Module imports
```python
from collections.abc import Iterable
from dataclasses import asdict

import numpy as np
import torch
import torch.nn as nn
from transformers import ParakeetEncoder as HFParakeetEncoder
from transformers import ParakeetFeatureExtractor, PretrainedConfig

from sglang.srt.configs.parakeet import ExtractorConfig, ParakeetConfig
from sglang.srt.layers.activation import ReLU2
from sglang.srt.layers.layernorm import RMSNorm
from sglang.srt.model_loader.weight_utils import default_weight_loader
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 35-35: Class `ParakeetProjection` overview
```python
class ParakeetProjection(nn.Module):
```
**EN:** Defines `ParakeetProjection` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `ParakeetProjection`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 36-46: Method `ParakeetProjection.__init__`
```python
    def __init__(self, config: ParakeetConfig) -> None:
        super().__init__()
        sound_hidden_size = config.hidden_size
        proj_hidden_size = config.projection_hidden_size
        llm_hidden_size = config.llm_hidden_size
        bias = config.projection_bias

        self.norm = RMSNorm(sound_hidden_size, eps=config.projection_eps)
        self.linear1 = nn.Linear(sound_hidden_size, proj_hidden_size, bias=bias)
        self.activation = ReLU2()
        self.linear2 = nn.Linear(proj_hidden_size, llm_hidden_size, bias=bias)
```
**EN:** This method implements `__init__(config: ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 48-53: Method `ParakeetProjection.forward`
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states = self.norm(hidden_states)
        hidden_states = self.linear1(hidden_states)
        hidden_states = self.activation(hidden_states)
        hidden_states = self.linear2(hidden_states)
        return hidden_states
```
**EN:** This method implements `forward(hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 56-56: Class `ProjectedParakeet` overview
```python
class ProjectedParakeet(nn.Module):
```
**EN:** Defines `ProjectedParakeet` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `ProjectedParakeet`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 57-72: Method `ProjectedParakeet.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        *,
        dtype: torch.dtype,
        llm_hidden_size: int,
        max_model_len: int,
    ) -> None:
        super().__init__()
        self.config = ParakeetConfig.from_hf_config(
            config, llm_hidden_size=llm_hidden_size, max_model_len=max_model_len
        )
        self.encoder = HFParakeetEncoder(self.config)
        self.encoder = self.encoder.to(dtype)
        self.projection = ParakeetProjection(self.config)
        self.projection = self.projection.to(dtype)
```
**EN:** This method implements `__init__(config: ..., *, dtype: ..., llm_hidden_size: ..., max_model_len: ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., *, dtype: ..., llm_hidden_size: ..., max_model_len: ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 74-82: Method `ProjectedParakeet.forward`
```python
    def forward(
        self, input_features: torch.Tensor, attention_mask: torch.Tensor | None = None
    ) -> torch.Tensor:
        outputs = self.encoder(
            input_features=input_features, attention_mask=attention_mask
        )
        outputs = outputs.last_hidden_state
        outputs = self.projection(outputs)
        return outputs
```
**EN:** This method implements `forward(input_features: ..., attention_mask: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_features: ..., attention_mask: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 84-114: Method `ProjectedParakeet.load_weights`
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        loaded_params: set[str] = set()
        params_dict = dict(self.named_parameters())
        buffers_dict = dict(self.named_buffers())

        if isinstance(weights, dict):
            weights_list = list(weights.items())
        else:
            weights_list = list(weights)

        for name, weight in weights_list:
            if name.startswith("sound_encoder.encoder.feature_extractor."):
                continue
            if name.startswith("sound_encoder."):
                target_name = name[len("sound_encoder.") :]
            elif name.startswith("sound_projection."):
                target_name = f"projection.{name[len('sound_projection.'):]}"
            else:
                continue

            target = params_dict.get(target_name)
            if target is None:
                target = buffers_dict.get(target_name)
            if target is None:
                continue
            weight_loader = getattr(target, "weight_loader", default_weight_loader)
            with torch.no_grad():
                weight_loader(target, weight)
            loaded_params.add(target_name)

        return loaded_params
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 117-117: Class `ParakeetExtractor` overview
```python
class ParakeetExtractor(ParakeetFeatureExtractor):
```
**EN:** Defines `ParakeetExtractor` as a reusable runtime type derived from ParakeetFeatureExtractor. The class groups 7 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `ParakeetExtractor`，其继承关系为 ParakeetFeatureExtractor。这个类组织了 7 个方法，用于实现模型相关行为。

### Lines 118-126: Method `ParakeetExtractor.__init__`
```python
    def __init__(self, config: PretrainedConfig) -> None:
        self.config = ExtractorConfig.from_hf_config(config)
        super().__init__(**asdict(self.config))
        self._clip_target_samples = int(
            round(self.config.clip_duration_s * self.sampling_rate)
        )
        self._tail_min_samples = int(
            round(self.config.clip_min_duration_s * self.sampling_rate)
        )
```
**EN:** This method implements `__init__(config: ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 128-134: Method `ParakeetExtractor._clip_sizes`
```python
    def _clip_sizes(self, audio_len: int) -> list[int]:
        audio_len = max(audio_len, self._tail_min_samples)
        num_full_clips, remainder = divmod(audio_len, self._clip_target_samples)
        clip_sizes = [self._clip_target_samples] * num_full_clips
        if remainder > 0:
            clip_sizes.append(max(remainder, self._tail_min_samples))
        return clip_sizes
```
**EN:** This method implements `_clip_sizes(audio_len: ...)` and implements a focused helper that supports the surrounding runtime flow inside `ParakeetExtractor`.
**CN:** 这个方法实现了 `_clip_sizes(audio_len: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `ParakeetExtractor` 内部调用。

### Lines 136-145: Method `ParakeetExtractor._subsampling_output_length`
```python
    def _subsampling_output_length(self, length: int) -> int:
        import math

        kernel_size = self.config.subsampling_conv_kernel_size
        stride = self.config.subsampling_conv_stride
        num_layers = int(math.log2(self.config.subsampling_factor))
        add_pad = (kernel_size - 1) // 2 * 2 - kernel_size
        for _ in range(num_layers):
            length = int(math.floor((length + add_pad) / stride + 1.0))
        return max(1, length)
```
**EN:** This method implements `_subsampling_output_length(length: ...)` and implements a focused helper that supports the surrounding runtime flow inside `ParakeetExtractor`.
**CN:** 这个方法实现了 `_subsampling_output_length(length: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `ParakeetExtractor` 内部调用。

### Lines 147-152: Method `ParakeetExtractor.audio_token_count`
```python
    def audio_token_count(self, audio_len: int) -> int:
        total_tokens = 0
        for clip_size in self._clip_sizes(audio_len):
            num_frames = clip_size // self.hop_length
            total_tokens += self._subsampling_output_length(num_frames)
        return max(1, total_tokens)
```
**EN:** This method implements `audio_token_count(audio_len: ...)` and implements a focused helper that supports the surrounding runtime flow inside `ParakeetExtractor`.
**CN:** 这个方法实现了 `audio_token_count(audio_len: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `ParakeetExtractor` 内部调用。

### Lines 154-167: Method `ParakeetExtractor.split_audio_into_clips`
```python
    def split_audio_into_clips(self, audio: np.ndarray) -> list[np.ndarray]:
        assert audio.ndim == 1
        audio_len = int(audio.shape[0])
        clip_sizes = self._clip_sizes(audio_len)
        target_len = sum(clip_sizes)
        if audio_len < target_len:
            audio = np.pad(audio, (0, target_len - audio_len))

        clips = list[np.ndarray]()
        offset = 0
        for clip_size in clip_sizes:
            clips.append(audio[offset : offset + clip_size])
            offset += clip_size
        return clips
```
**EN:** This method implements `split_audio_into_clips(audio: ...)` and implements a focused helper that supports the surrounding runtime flow inside `ParakeetExtractor`.
**CN:** 这个方法实现了 `split_audio_into_clips(audio: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `ParakeetExtractor` 内部调用。

### Lines 169-179: Method `ParakeetExtractor.__call__`
```python
    def __call__(self, raw_speech: list[np.ndarray], *args, **kwargs):
        audio_clips = list[np.ndarray]()
        audio_num_clips = list[int]()
        for audio in raw_speech:
            clips = self.split_audio_into_clips(audio)
            audio_clips.extend(clips)
            audio_num_clips.append(len(clips))

        outputs = super().__call__(audio_clips, *args, **kwargs)
        outputs["audio_num_clips"] = audio_num_clips
        return outputs
```
**EN:** This method implements `__call__(raw_speech: ..., *args, **kwargs)` and implements a focused helper that supports the surrounding runtime flow inside `ParakeetExtractor`.
**CN:** 这个方法实现了 `__call__(raw_speech: ..., *args, **kwargs)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `ParakeetExtractor` 内部调用。

### Lines 181-184: Method `ParakeetExtractor.audio_length`
```python
    @staticmethod
    def audio_length(raw_config: PretrainedConfig, audio_tokens: int) -> int:
        config = ExtractorConfig.from_hf_config(raw_config)
        return int(audio_tokens * config.subsampling_factor * config.hop_length)
```
**EN:** This method implements `audio_length(audio_tokens: ...)` and implements a focused helper that supports the surrounding runtime flow inside `ParakeetExtractor` Decorators: staticmethod.
**CN:** 这个方法实现了 `audio_length(audio_tokens: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `ParakeetExtractor` 内部调用 装饰器：staticmethod。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Multimodal feature processing / **CN:** 多模态特征处理
- **EN:** Checkpoint remapping and weight loading / **CN:** 检查点重映射与权重加载
- **EN:** Distributed/parallel inference layout / **CN:** 分布式/并行推理布局

## Dependencies / 依赖关系
- `collections.abc.Iterable`
- `dataclasses.asdict`
- `numpy`
- `torch`
- `torch.nn`
- `transformers.ParakeetEncoder`
- `transformers.ParakeetFeatureExtractor`
- `transformers.PretrainedConfig`
- `sglang.srt.configs.parakeet.ExtractorConfig`
- `sglang.srt.configs.parakeet.ParakeetConfig`
- `sglang.srt.layers.activation.ReLU2`
- `sglang.srt.layers.layernorm.RMSNorm`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `math`
