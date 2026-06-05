# midashenglm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/midashenglm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the midashenglm model stack used by SGLang SRT inference, including network blocks, runtime wrappers, and checkpoint-loading utilities. / 该模块实现 SGLang SRT 推理所使用的 midashenglm 模型栈，包括网络模块、运行时封装以及检查点加载工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28: Module imports
```python
import collections
import collections.abc
import logging
from collections.abc import Callable, Sequence
from typing import Iterable, List, Optional, Tuple, TypeAlias, cast

import torch
import torch.nn as nn
import torchaudio.functional as F
from transformers import PretrainedConfig

from sglang.srt.layers.attention.vision import VisionAttention
from sglang.srt.layers.conv import Conv2dLayer
from sglang.srt.layers.linear import ColumnParallelLinear, RowParallelLinear
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
from sglang.srt.models.qwen2 import Qwen2ForCausalLM
from sglang.srt.utils import add_prefix
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 30-30: Top-level assign
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines or updates logger, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 logger，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 31-31: Top-level annassign
```python
_Tuple2: TypeAlias = int | tuple[int, int] | Sequence[int]
```
**EN:** Defines or updates _Tuple2, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _Tuple2，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 34-40: Function `_resolve_tuple2`
```python
def _resolve_tuple2(x: _Tuple2) -> tuple[int, int]:
    if isinstance(x, collections.abc.Sequence):
        assert (
            len(x) == 2
        ), f"Expected a sequence of length 2, got {x} with length {len(x)}"
        return cast(tuple[int, int], tuple(x))
    return (x, x)
```
**EN:** This function implements `_resolve_tuple2(x: ...)` and resolves runtime selections into concrete classes or configuration values.
**CN:** 这个函数实现了 `_resolve_tuple2(x: ...)`，其作用是把运行时选择解析为具体类或配置值。

### Lines 43-59: Function `calculate_mel_frames_dasheng`
```python
def calculate_mel_frames_dasheng(
    audio_length_samples: int,
    n_fft: int = 512,
    hop_size: int = 160,
    dasheng_subsampling: int = 4,
    center=True,
    model_subsampling: int = 5,
) -> int:
    """Calculate the number of Mel-spectrogram frames."""
    if center:
        audio_length_samples = audio_length_samples + n_fft

    return (
        int(1 + ((audio_length_samples - n_fft) / hop_size))
        // dasheng_subsampling
        // model_subsampling
    )
```
**EN:** This function implements `calculate_mel_frames_dasheng(audio_length_samples: ..., n_fft: ...=..., hop_size: ...=..., dasheng_subsampling: ...=..., center=..., model_subsampling: ...=...)` and Calculate the number of Mel-spectrogram frames.
**CN:** 这个函数实现了 `calculate_mel_frames_dasheng(audio_length_samples: ..., n_fft: ...=..., hop_size: ...=..., dasheng_subsampling: ...=..., center=..., model_subsampling: ...=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 62-62: Class `AudioPatchEmbed` overview
```python
class AudioPatchEmbed(nn.Module):
```
**EN:** Defines `AudioPatchEmbed` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `AudioPatchEmbed`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 63-89: Method `AudioPatchEmbed.__init__`
```python
    def __init__(
        self,
        input_size: _Tuple2 = 64,
        patch_size: _Tuple2 = 16,
        patch_stride: _Tuple2 = 16,
        in_chans: int = 1,
        embed_dim: int = 768,
        norm_layer: Callable | None = None,
        flatten: bool = False,
    ):
        super().__init__()
        self.input_size = _resolve_tuple2(input_size)
        self.patch_size = _resolve_tuple2(patch_size)
        self.patch_stride = _resolve_tuple2(patch_stride)
        self.grid_size = (
            self.input_size[0] // self.patch_stride[0],
            self.input_size[1] // self.patch_stride[1],
        )
        self.num_patches = self.grid_size[0] * self.grid_size[1]
        self.flatten = flatten
        self.proj = Conv2dLayer(
            in_chans,
            embed_dim,
            kernel_size=self.patch_size,
            stride=self.patch_stride,
        )
        self.norm = norm_layer(embed_dim) if norm_layer else nn.Identity()
```
**EN:** This method implements `__init__(input_size: ...=..., patch_size: ...=..., patch_stride: ...=..., in_chans: ...=..., embed_dim: ...=..., norm_layer: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(input_size: ...=..., patch_size: ...=..., patch_stride: ...=..., in_chans: ...=..., embed_dim: ...=..., norm_layer: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 91-96: Method `AudioPatchEmbed.forward`
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x = self.proj(x)
        if self.flatten:
            x = torch.permute(torch.flatten(x, 2, 3), (0, 2, 1))
        x = self.norm(x)
        return x
```
**EN:** This method implements `forward(x: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 99-99: Class `LayerScale` overview
```python
class LayerScale(nn.Module):
```
**EN:** Defines `LayerScale` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `LayerScale`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 100-103: Method `LayerScale.__init__`
```python
    def __init__(self, dim, init_values=1e-5, inplace=False):
        super().__init__()
        self.inplace = inplace
        self.gamma = nn.Parameter(init_values * torch.ones(dim))
```
**EN:** This method implements `__init__(dim, init_values=..., inplace=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(dim, init_values=..., inplace=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 105-106: Method `LayerScale.forward`
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return x.mul_(self.gamma) if self.inplace else x * self.gamma
```
**EN:** This method implements `forward(x: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 109-109: Class `DashengMlp` overview
```python
class DashengMlp(nn.Module):
```
**EN:** Defines `DashengMlp` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `DashengMlp`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 110-135: Method `DashengMlp.__init__`
```python
    def __init__(
        self,
        in_features: int,
        hidden_features: int | None = None,
        out_features: int | None = None,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        out_features = out_features or in_features
        hidden_features = hidden_features or in_features
        self.fc1 = ColumnParallelLinear(
            input_size=in_features,
            output_size=hidden_features,
            bias=True,
            quant_config=quant_config,
            prefix=add_prefix("fc1", prefix),
        )
        self.act = nn.GELU()
        self.fc2 = RowParallelLinear(
            input_size=hidden_features,
            output_size=out_features,
            bias=True,
            quant_config=quant_config,
            prefix=add_prefix("fc2", prefix),
        )
```
**EN:** This method implements `__init__(in_features: ..., hidden_features: ...=..., out_features: ...=..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(in_features: ..., hidden_features: ...=..., out_features: ...=..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 137-141: Method `DashengMlp.forward`
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x, _ = self.fc1(x)
        x = self.act(x)
        x, _ = self.fc2(x)
        return x
```
**EN:** This method implements `forward(x: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 144-146: Class `DashengAttention` overview
```python
class DashengAttention(nn.Module):
    """Audio encoder attention using VisionAttention for compatibility."""
```
**EN:** Defines `DashengAttention` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `DashengAttention`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 147-174: Method `DashengAttention.__init__`
```python
    def __init__(
        self,
        dim: int,
        num_heads: int = 8,
        qkv_bias: bool = False,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        assert dim % num_heads == 0, "dim should be divisible by num_heads"
        self.embed_dim = dim
        self.num_heads = num_heads
        self.head_dim = self.embed_dim // self.num_heads
        self.scale = self.head_dim**-0.5

        self.attn = VisionAttention(
            embed_dim=dim,
            num_heads=num_heads,
            projection_size=dim,
            use_qkv_parallel=True,
            proj_bias=True,
            qkv_bias=qkv_bias,
            qkv_backend="sdpa",
            softmax_in_single_precision=False,
            flatten_batch=False,
            quant_config=quant_config,
            prefix=prefix,
        )
```
**EN:** This method implements `__init__(dim: ..., num_heads: ...=..., qkv_bias: ...=..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(dim: ..., num_heads: ...=..., qkv_bias: ...=..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 176-189: Method `DashengAttention.forward`
```python
    def forward(self, x: torch.Tensor, mask: torch.Tensor | None = None):
        """
        Args:
            x: [B, N, C] tensor
            mask: [B, N] boolean mask
        """
        attn_mask = None
        if mask is not None:
            attn_mask = mask.unsqueeze(1).unsqueeze(2)  # [B, 1, 1, N]
            attn_mask = attn_mask.float()
            attn_mask = (1.0 - attn_mask) * -10000.0

        x = self.attn(x, attn_mask=attn_mask)
        return x
```
**EN:** This method implements `forward(x: ..., mask: ...=...)` and Args:.
**CN:** 这个方法实现了 `forward(x: ..., mask: ...=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 192-192: Class `DashengBlock` overview
```python
class DashengBlock(nn.Module):
```
**EN:** Defines `DashengBlock` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `DashengBlock`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 193-224: Method `DashengBlock.__init__`
```python
    def __init__(
        self,
        dim: int,
        num_heads: int,
        mlp_ratio: float = 4.0,
        qkv_bias: bool = False,
        init_values: float | None = None,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.norm1 = nn.LayerNorm(dim, eps=1e-6)
        self.attn = DashengAttention(
            dim,
            num_heads=num_heads,
            qkv_bias=qkv_bias,
            quant_config=quant_config,
            prefix=add_prefix("attn", prefix),
        )
        self.ls1 = (
            LayerScale(dim, init_values=init_values) if init_values else nn.Identity()
        )
        self.norm2 = nn.LayerNorm(dim, eps=1e-6)
        self.mlp = DashengMlp(
            in_features=dim,
            hidden_features=int(dim * mlp_ratio),
            quant_config=quant_config,
            prefix=add_prefix("mlp", prefix),
        )
        self.ls2 = (
            LayerScale(dim, init_values=init_values) if init_values else nn.Identity()
        )
```
**EN:** This method implements `__init__(dim: ..., num_heads: ..., mlp_ratio: ...=..., qkv_bias: ...=..., init_values: ...=..., quant_config: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(dim: ..., num_heads: ..., mlp_ratio: ...=..., qkv_bias: ...=..., init_values: ...=..., quant_config: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 226-233: Method `DashengBlock.forward`
```python
    def forward(
        self,
        x: torch.Tensor,
        mask: torch.Tensor | None = None,
    ) -> torch.Tensor:
        x = x + self.ls1(self.attn(self.norm1(x), mask))
        x = x + self.ls2(self.mlp(self.norm2(x)))
        return x
```
**EN:** This method implements `forward(x: ..., mask: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x: ..., mask: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 236-238: Class `DashengFrontend` overview
```python
class DashengFrontend(nn.Module):
    """Audio frontend that converts waveforms to log mel-spectrograms."""
```
**EN:** Defines `DashengFrontend` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `DashengFrontend`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 239-260: Method `DashengFrontend.__init__`
```python
    def __init__(self, config: PretrainedConfig):
        super().__init__()
        self.n_fft = config.n_fft
        self.hop_length = config.hop_length
        self.win_length = config.win_length
        self.center = config.center
        spectrogram_window = torch.hann_window(config.win_length)
        self.register_buffer(
            "spectrogram_window",
            spectrogram_window,
            persistent=False,
        )
        self.spectrogram_window: torch.Tensor
        melscale_fbanks = F.melscale_fbanks(
            n_freqs=config.n_fft // 2 + 1,
            f_min=config.f_min,
            f_max=config.f_max,
            n_mels=config.n_mels,
            sample_rate=config.sample_rate,
        )
        self.register_buffer("melscale_fbanks", melscale_fbanks, persistent=False)
        self.melscale_fbanks: torch.Tensor
```
**EN:** This method implements `__init__(config: ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 262-290: Method `DashengFrontend.forward`
```python
    def forward(self, waveform: torch.Tensor) -> torch.Tensor:
        """Convert waveform to log mel-spectrogram.

        Args:
            waveform: [B, T] tensor of audio samples

        Returns:
            log_mel_spectrogram: [B, n_mels, time] tensor
        """
        spectrogram = F.spectrogram(
            waveform=waveform.to(torch.float32),
            pad=0,
            window=self.spectrogram_window,
            n_fft=self.n_fft,
            hop_length=self.hop_length,
            win_length=self.win_length,
            power=2,
            normalized=False,
            center=self.center,
        )
        mel_spectrogram = (spectrogram.mT @ self.melscale_fbanks.to(torch.float32)).mT
        log_mel_spectrogram = F.amplitude_to_DB(
            mel_spectrogram.unsqueeze(1),
            multiplier=10,
            amin=1e-10,
            db_multiplier=0,
            top_db=120,
        ).squeeze(1)
        return log_mel_spectrogram.to(waveform.dtype)
```
**EN:** This method implements `forward(waveform: ...)` and Convert waveform to log mel-spectrogram.
**CN:** 这个方法实现了 `forward(waveform: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 293-295: Class `DashengAudioTransformer` overview
```python
class DashengAudioTransformer(nn.Module):
    """Audio encoder transformer."""
```
**EN:** Defines `DashengAudioTransformer` as a reusable runtime type derived from nn.Module. The class groups 4 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `DashengAudioTransformer`，其继承关系为 nn.Module。这个类组织了 4 个方法，用于实现模型相关行为。

### Lines 296-333: Method `DashengAudioTransformer.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.target_length = config.target_length
        self.hop_length = config.hop_length
        self.front_end = DashengFrontend(config)
        self.init_bn = nn.BatchNorm2d(config.n_mels, momentum=0.01)
        self.patch_embed = AudioPatchEmbed(
            input_size=(config.n_mels, config.target_length),
            embed_dim=config.embed_dim,
            in_chans=config.input_channels,
            patch_size=config.patch_size,
            flatten=False,
            patch_stride=config.patch_stride,
        )
        self.time_pos_embed = nn.Parameter(
            torch.empty(1, config.embed_dim, 1, self.patch_embed.grid_size[1])
        )
        self.freq_pos_embed = nn.Parameter(
            torch.empty(1, config.embed_dim, self.patch_embed.grid_size[0], 1)
        )
        self.blocks = nn.ModuleList(
            DashengBlock(
                dim=config.embed_dim,
                num_heads=config.num_heads,
                mlp_ratio=config.mlp_ratio,
                qkv_bias=config.qkv_bias,
                init_values=config.init_values,
                quant_config=quant_config,
                prefix=add_prefix(f"blocks.{i}", prefix),
            )
            for i in range(config.depth)
        )
        self.norm = nn.LayerNorm(config.embed_dim, eps=1e-6)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 335-347: Method `DashengAudioTransformer.forward_features`
```python
    def forward_features(
        self,
        x: torch.Tensor,
        mask: torch.Tensor | None = None,
    ) -> torch.Tensor:
        t = x.shape[-1]
        x = x + self.time_pos_embed[:, :, :, :t]
        x = x + self.freq_pos_embed[:, :, :, :]
        x = torch.permute(torch.flatten(x, 2, 3), (0, 2, 1))
        for block in self.blocks:
            x = block(x, mask)
        x = self.norm(x)
        return x
```
**EN:** This method implements `forward_features(x: ..., mask: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_features(x: ..., mask: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 349-354: Method `DashengAudioTransformer._to_mask`
```python
    def _to_mask(self, lengths: torch.Tensor, max_length: int) -> torch.Tensor:
        batch_size = len(lengths)
        idx = torch.arange(max_length, device=lengths.device)
        idx = idx.repeat(batch_size).view(batch_size, max_length)
        mask = (idx < lengths.unsqueeze(-1)).bool()
        return mask
```
**EN:** This method implements `_to_mask(lengths: ..., max_length: ...)` and implements a focused helper that supports the surrounding runtime flow inside `DashengAudioTransformer`.
**CN:** 这个方法实现了 `_to_mask(lengths: ..., max_length: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `DashengAudioTransformer` 内部调用。

### Lines 356-398: Method `DashengAudioTransformer.forward`
```python
    def forward(
        self,
        x: torch.Tensor,
        x_length: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        """
        Args:
            x: [B, T] audio waveform tensor
            x_length: [B] tensor of audio lengths

        Returns:
            x: [B, seq_len, embed_dim] encoded features
            mask: [B, seq_len] mask tensor
        """
        x = self.front_end(x)
        x = x.to(self.time_pos_embed.dtype)
        target_length_in_patches = self.target_length // 4
        x = x.unsqueeze(1)
        x = torch.permute(x, (0, 2, 1, 3))
        x = self.init_bn(x)
        x = torch.permute(x, (0, 2, 1, 3))
        x = self.patch_embed(x)
        t = x.shape[-1]
        input_splits = x.split(target_length_in_patches, dim=-1)
        if x_length is not None:
            assert len(x_length) == len(
                x
            ), "batchsizes of input x and x_length need to be same"
            assert x_length.ndim == 1, "Lengths are of size (B,)"
            scaled_lengths = (x_length / (self.hop_length * 4)).long()
            mask = self._to_mask(max_length=t, lengths=scaled_lengths)
            split_masks = mask.split(target_length_in_patches, dim=-1)
        else:
            mask = None
            split_masks = [None] * len(input_splits)
        outputs = []
        for split_x, split_mask in zip(input_splits, split_masks):
            forward_kwargs = {}
            forward_kwargs["mask"] = split_mask
            split_x = self.forward_features(split_x, **forward_kwargs)
# ... truncated for brevity ...
```
**EN:** This method implements `forward(x: ..., x_length: ...=...)` and Args:.
**CN:** 这个方法实现了 `forward(x: ..., x_length: ...=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 401-403: Class `AudioProjectorSubsample` overview
```python
class AudioProjectorSubsample(nn.Module):
    """Audio projector with subsampling."""
```
**EN:** Defines `AudioProjectorSubsample` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `AudioProjectorSubsample`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 404-429: Method `AudioProjectorSubsample.__init__`
```python
    def __init__(
        self,
        in_dim: int,
        out_dim: int,
        downsample_rate=5,
        dtype: torch.dtype | None = None,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.k = downsample_rate
        self.fc1 = ColumnParallelLinear(
            input_size=in_dim * self.k,
            output_size=out_dim,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("net.0", prefix),
        )
        self.act = nn.GELU()
        self.fc2 = RowParallelLinear(
            input_size=out_dim,
            output_size=out_dim,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("net.2", prefix),
        )
```
**EN:** This method implements `__init__(in_dim: ..., out_dim: ..., downsample_rate=..., dtype: ...=..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(in_dim: ..., out_dim: ..., downsample_rate=..., dtype: ...=..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 431-446: Method `AudioProjectorSubsample.forward`
```python
    def forward(self, x, mask=None):
        batch_size, seq_len, dim = x.shape
        num_frames_to_discard = seq_len % self.k
        if num_frames_to_discard > 0:
            x = x[:, :-num_frames_to_discard, :]
            if mask is not None:
                mask = mask[:, :-num_frames_to_discard]
        if mask is None:
            mask = torch.ones(x.shape[:-1], dtype=torch.long, device=x.device)
        x = x.reshape(batch_size, -1, self.k * dim)
        x, _ = self.fc1(x)
        x = self.act(x)
        x, _ = self.fc2(x)
        mask = mask.reshape(batch_size, -1, self.k)
        mask = mask.any(dim=-1).long()
        return x, mask
```
**EN:** This method implements `forward(x, mask=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x, mask=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 449-451: Class `MiDashengLMModel` overview
```python
class MiDashengLMModel(nn.Module):
    """MiDashengLM model for audio-language processing."""
```
**EN:** Defines `MiDashengLMModel` as a reusable runtime type derived from nn.Module. The class groups 7 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiDashengLMModel`，其继承关系为 nn.Module。这个类组织了 7 个方法，用于实现模型相关行为。

### Lines 452-469: Class `MiDashengLMModel` attributes
```python
    default_bitsandbytes_target_modules = [
        ".fc1.",
        ".fc2.",
        ".gate_up_proj.",
        ".down_proj.",
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
**EN:** Defines class-level attributes and metadata that shape how `MiDashengLMModel` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `MiDashengLMModel` 在运行时的行为。

### Lines 471-503: Method `MiDashengLMModel.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        rope_scaling = config.text_config.rope_parameters
        if rope_scaling:
            if "mrope_section" in rope_scaling:
                # Remove mrope_section from rope_parameters so downstream
                # code treats this as standard rotary embedding.
                del rope_scaling["mrope_section"]
        self.audio_encoder = DashengAudioTransformer(
            config.audio_encoder_config,
            quant_config=quant_config,
            prefix=add_prefix("audio_encoder", prefix),
        )
        self.audio_projector = AudioProjectorSubsample(
            in_dim=config.audio_encoder_config.embed_dim,
            out_dim=config.text_config.hidden_size,
            downsample_rate=config.subsample_factor,
            quant_config=quant_config,
            prefix=add_prefix("audio_projector", prefix),
        )
        self.language_model = Qwen2ForCausalLM(
            config.text_config,
            quant_config=quant_config,
            prefix=add_prefix("decoder", prefix),
        )
        self.logits_processor = self.language_model.logits_processor
        self.quant_config = quant_config
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 505-508: Method `MiDashengLMModel.pad_input_ids`
```python
    def pad_input_ids(self, input_ids: List[int], mm_inputs: MultimodalInputs):
        """Pad input IDs with multimodal tokens."""
        pattern = MultiModalityDataPaddingPatternMultimodalTokens()
        return pattern.pad_input_tokens(input_ids, mm_inputs)
```
**EN:** This method implements `pad_input_ids(input_ids: ..., mm_inputs: ...)` and Pad input IDs with multimodal tokens.
**CN:** 这个方法实现了 `pad_input_ids(input_ids: ..., mm_inputs: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 510-558: Method `MiDashengLMModel.get_audio_feature`
```python
    def get_audio_feature(self, items: List[MultimodalDataItem]) -> torch.Tensor:
        """Process audio inputs and return embeddings.

        Args:
            items: List of multimodal data items containing audio features

        Returns:
            audio_embeddings: Concatenated audio embeddings
        """
        logger.debug("=" * 80)
        logger.debug(f"get_audio_feature called with {len(items)} items")
        logger.debug("=" * 80)
        for i, item in enumerate(items):
            logger.debug(f"Item {i} feature shape: {item.feature.shape}")
            logger.debug(
                f"Item {i} audio_length: {getattr(item, 'audio_length', 'NOT SET')}"
            )
            logger.debug(f"Item {i} pad_value: {getattr(item, 'pad_value', 'NOT SET')}")
            logger.debug(f"Item {i} hash: {getattr(item, 'hash', 'NOT SET')}")
        input_values = torch.cat([item.feature for item in items], dim=0)
        logger.debug(f"Concatenated input_values shape: {input_values.shape}")
        audio_lengths = []
        for item in items:
            if hasattr(item, "audio_length") and item.audio_length is not None:
                audio_lengths.append(item.audio_length)
            else:
                audio_lengths.append(item.feature.shape[-1])
        audio_length = torch.tensor(audio_lengths, device=input_values.device)
        logger.debug(f"audio_length: {audio_length}")
        encoder_out, encoder_atts = self.audio_encoder(input_values, audio_length)
        logger.debug(f"Encoder output shape: {encoder_out.shape}")
        audio_embeddings, _ = self.audio_projector(encoder_out, encoder_atts)
        audio_embeddings = audio_embeddings.to(input_values.dtype)
        logger.debug(f"Projector output shape: {audio_embeddings.shape}")
        batch_size, max_audio_tokens, embed_dim = audio_embeddings.shape
        logger.debug(f"Using all {max_audio_tokens} audio tokens from projector output")
        masked_audio_features = audio_embeddings.reshape(-1, embed_dim)
        logger.debug(f"Final output shape: {masked_audio_features.shape}")
        logger.debug(
            f"Stats: min={masked_audio_features.min().item():.4f}, max={masked_audio_features.max().item():.4f}"
# ... truncated for brevity ...
```
**EN:** This method implements `get_audio_feature(items: ...)` and Process audio inputs and return embeddings.
**CN:** 这个方法实现了 `get_audio_feature(items: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 560-561: Method `MiDashengLMModel.get_input_embeddings`
```python
    def get_input_embeddings(self):
        return self.language_model.model.embed_tokens
```
**EN:** This method implements `get_input_embeddings()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 563-606: Method `MiDashengLMModel.forward`
```python
    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        **kwargs,
    ):
        """Run forward pass for MiDashengLM.

        Args:
            input_ids: Flattened (concatenated) input_ids corresponding to a batch.
            positions: Flattened (concatenated) position ids corresponding to a batch.
            forward_batch: Forward batch information including multimodal data.
        """
        if forward_batch.contains_mm_inputs():
            logger.debug("=" * 80)
            logger.debug(f"input_ids shape: {input_ids.shape}")
            logger.debug(f"input_ids first 20: {input_ids[:20].tolist()}")
            logger.debug(
                f"input_ids unique values count: {len(torch.unique(input_ids))}"
            )
            if forward_batch.mm_inputs and len(forward_batch.mm_inputs) > 0:
                mm_input = forward_batch.mm_inputs[0]
                if mm_input and len(mm_input.mm_items) > 0:
                    pad_value = mm_input.mm_items[0].pad_value
                    logger.debug(f"Expected pad_value: {pad_value}")
                    logger.debug(
                        f"Count of pad_value in input_ids: {(input_ids == pad_value).sum().item()}"
                    )
                    if hasattr(mm_input, "audio_token_id") and mm_input.audio_token_id:
                        logger.debug(f"audio_token_id: {mm_input.audio_token_id}")
                        logger.debug(
                            f"Count of audio_token_id in input_ids: {(input_ids == mm_input.audio_token_id).sum().item()}"
                        )
            logger.debug("=" * 80)

        return general_mm_embed_routine(
            input_ids=input_ids,
            forward_batch=forward_batch,
# ... truncated for brevity ...
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., **kwargs)` and Run forward pass for MiDashengLM. Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., **kwargs)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责 装饰器：torch.no_grad()。

### Lines 608-690: Method `MiDashengLMModel.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        """Load model weights."""
        params_dict = dict(self.named_parameters(remove_duplicate=False))
        buffers_dict = dict(self.named_buffers())
        audio_encoder_loaded = []
        audio_projector_loaded = []
        skipped_weights = []
        decoder_weights = []
        for name, loaded_weight in weights:
            if "rotary_emb.inv_freq" in name:
                continue
            if "rotary_emb.cos_cached" in name or "rotary_emb.sin_cached" in name:
                continue
            if name.startswith("decoder"):
                decoder_weights.append((name, loaded_weight))
                continue
            original_name = name
            if "audio_encoder.front_end" in name:
                if ".mel_scale.fb" in name:
                    name = name.replace(".mel_scale.fb", ".melscale_fbanks")
                elif ".spectrogram.window" in name:
                    name = name.replace(".spectrogram.window", ".spectrogram_window")
            if "audio_encoder" in name and ".attn.qkv." in name:
                name = name.replace(".attn.qkv.", ".attn.attn.qkv_proj.")
            if "audio_encoder" in name and ".attn.proj." in name:
                name = name.replace(".attn.proj.", ".attn.attn.proj.")
            if "audio_projector" in name:
                name = name.replace(".net.0.", ".fc1.")
                name = name.replace(".net.2.", ".fc2.")
            if (
                name.endswith(".bias")
                and name not in params_dict
                and name not in buffers_dict
            ):
                skipped_weights.append(f"{original_name} (bias not in params/buffers)")
                continue
            if name in params_dict:
                param = params_dict[name]
                weight_loader = getattr(param, "weight_loader", default_weight_loader)
                weight_loader(param, loaded_weight)
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and Load model weights.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 692-696: Method `MiDashengLMModel.get_embed_and_head`
```python
    def get_embed_and_head(self):
        return (
            self.language_model.model.embed_tokens.weight,
            self.language_model.lm_head.weight,
        )
```
**EN:** This method implements `get_embed_and_head()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_embed_and_head()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 699-699: Top-level assign
```python
EntryClass = [MiDashengLMModel]
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `collections`
- `collections.abc`
- `logging`
- `collections.abc.Callable`
- `collections.abc.Sequence`
- `typing.Iterable`
- `typing.List`
- `typing.Optional`
- `typing.Tuple`
- `typing.TypeAlias`
- `typing.cast`
- `torch`
- `torch.nn`
- `torchaudio.functional`
- `transformers.PretrainedConfig`
- `sglang.srt.layers.attention.vision.VisionAttention`
- `sglang.srt.layers.conv.Conv2dLayer`
- `sglang.srt.layers.linear.ColumnParallelLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.managers.mm_utils.MultiModalityDataPaddingPatternMultimodalTokens`
- `sglang.srt.managers.mm_utils.general_mm_embed_routine`
- `sglang.srt.managers.schedule_batch.Modality`
- `sglang.srt.managers.schedule_batch.MultimodalDataItem`
- `sglang.srt.managers.schedule_batch.MultimodalInputs`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.models.qwen2.Qwen2ForCausalLM`
- `sglang.srt.utils.add_prefix`
