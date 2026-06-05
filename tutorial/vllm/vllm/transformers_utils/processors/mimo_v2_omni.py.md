# mimo_v2_omni.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/transformers_utils/processors/mimo_v2_omni.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] MiMo-Omni multimodal processor for vLLM. / [CN] 实现 Mimo V2 Omni 相关的处理器集成逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 4-7: Module overview
```python
"""MiMo-Omni multimodal processor for vLLM.

Ported from SGLang's MiMoV2OmniProcessor / MiMoVLProcessor implementations.
"""
```
**EN:** The module docstring frames the file's responsibility and intended usage. Its opening line is: MiMo-Omni multimodal processor for vLLM.
**CN:** 模块文档字符串概述了该文件的职责与预期使用方式。

### Lines 9-27: Imports
```python
import contextlib
import copy
import io
import logging
import math
from collections import OrderedDict
from concurrent.futures import ThreadPoolExecutor, as_completed
from dataclasses import dataclass, field
from io import BytesIO
from typing import Any, Literal

import numpy as np
import regex as re
import requests
import torch
import torch.nn.functional as F
from PIL import Image
from transformers import BatchFeature, TensorType
from transformers.processing_utils import ProcessorMixin
```
**EN:** This import block assembles the module's dependencies: standard-library helpers such as `contextlib`, `copy`, `io`, `logging`, `math`, `collections`, `concurrent.futures`, `dataclasses`, ... (+1 more), external APIs such as `numpy`, `regex`, `requests`, `torch`, `torch.nn.functional`, `PIL`, `transformers`, `transformers.processing_utils`, and internal vLLM modules such as none. Together they define the services the rest of the file builds on.
**CN:** 该导入块汇集了模块依赖：标准库工具如 `contextlib`, `copy`, `io`, `logging`, `math`, `collections`, `concurrent.futures`, `dataclasses`, ... (+1 more)，外部 API 如 `numpy`, `regex`, `requests`, `torch`, `torch.nn.functional`, `PIL`, `transformers`, `transformers.processing_utils`，以及 vLLM 内部模块如 无。这些依赖共同构成了后续实现所依赖的基础能力。

### Lines 29-35: Try/except block
```python
try:
    from torchcodec.decoders import AudioDecoder

    _HAS_TORCHCODEC = True
except ImportError:
    AudioDecoder = None
    _HAS_TORCHCODEC = False
```
**EN:** This top-level `try`/`except` block guards module initialization so the file can degrade gracefully when a dependency or capability is unavailable.
**CN:** 这个顶层 `try`/`except` 代码块为模块初始化提供保护，使文件在依赖或能力缺失时可以平滑退化。

### Lines 37-45: Try/except block
```python
try:
    import torchaudio
    from torchaudio.transforms import MelSpectrogram as _MelSpectrogram

    _HAS_TORCHAUDIO = True
except ImportError:
    torchaudio = None  # type: ignore[assignment]
    _MelSpectrogram = None  # type: ignore[assignment,misc]
    _HAS_TORCHAUDIO = False
```
**EN:** This top-level `try`/`except` block guards module initialization so the file can degrade gracefully when a dependency or capability is unavailable.
**CN:** 这个顶层 `try`/`except` 代码块为模块初始化提供保护，使文件在依赖或能力缺失时可以平滑退化。

### Lines 47-55: Module state and constants
```python
logger = logging.getLogger(__name__)

# ---------------------------------------------------------------------------
# Constants
# ---------------------------------------------------------------------------

_PIXEL_MEAN = [123.675, 116.28, 103.53]
_PIXEL_STD = [58.395, 57.12, 57.375]
_mean_std_cache: dict[str, tuple[torch.Tensor, torch.Tensor]] = {}
```
**EN:** This block defines module-level constants/defaults such as `logger`, `_PIXEL_MEAN`, `_PIXEL_STD`, `_mean_std_cache`. They centralize shared parameters for later helpers and classes.
**CN:** 该代码块定义了模块级常量或默认值，例如 `logger`, `_PIXEL_MEAN`, `_PIXEL_STD`, `_mean_std_cache`。它们把后续辅助函数和类会复用的参数集中到一起。

### Lines 64-68: Class `ImageInput`
```python
class ImageInput:
    # PIL.Image | str (path/url/base64) | bytes | torch.Tensor (C,H,W)
    image: Any
    max_pixels: int | None = None
    min_pixels: int | None = None
```
**EN:** Defines `ImageInput`, a processor-related class. Class attributes such as `image`, `max_pixels`, `min_pixels` encode defaults or metadata.
**CN:** 定义 `ImageInput`，这是一个处理器相关类。 类属性如 `image`, `max_pixels`, `min_pixels` 编码了默认值或元数据。

### Lines 72-85: Class `VideoInput`
```python
class VideoInput:
    # tuple[frames_TCHW: torch.Tensor, timestamps_T: torch.Tensor]
    video: Any
    min_pixels: int | None = None
    max_pixels: int | None = None
    total_max_pixels: int | None = None
    fps: float | None = None
    num_frames: int | None = None
    max_frames: int | None = None
    min_frames: int | None = None
    do_include_last_frame: bool | None = False
    start_time: float | None = None
    end_time: float | None = None
    segment_type: Literal["individual", "partial"] = "individual"
```
**EN:** Defines `VideoInput`, a processor-related class. Class attributes such as `video`, `min_pixels`, `max_pixels`, `total_max_pixels`, `fps`, ... (+7 more) encode defaults or metadata.
**CN:** 定义 `VideoInput`，这是一个处理器相关类。 类属性如 `video`, `min_pixels`, `max_pixels`, `total_max_pixels`, `fps`, ... (+7 more) 编码了默认值或元数据。

### Lines 89-92: Class `AudioInput`
```python
class AudioInput:
    # str (path/url/base64) | bytes | tuple[waveform_1D, sr]
    # | np.ndarray | torch.Tensor (T,n_vq)
    audio: Any
```
**EN:** Defines `AudioInput`, a processor-related class. Class attributes such as `audio` encode defaults or metadata.
**CN:** 定义 `AudioInput`，这是一个处理器相关类。 类属性如 `audio` 编码了默认值或元数据。

### Lines 96-109: Class `VideoAudioInput`
```python
class VideoAudioInput:
    video: Any  # same as VideoInput.video
    audio: Any  # same as AudioInput.audio
    min_pixels: int | None = None
    max_pixels: int | None = None
    total_max_pixels: int | None = None
    fps: float | None = None
    num_frames: int | None = None
    max_frames: int | None = None
    min_frames: int | None = None
    do_include_last_frame: bool | None = False
    start_time: float | None = None
    end_time: float | None = None
    segment_type: Literal["individual", "partial"] = "individual"
```
**EN:** Defines `VideoAudioInput`, a processor-related class. Class attributes such as `video`, `audio`, `min_pixels`, `max_pixels`, `total_max_pixels`, ... (+8 more) encode defaults or metadata.
**CN:** 定义 `VideoAudioInput`，这是一个处理器相关类。 类属性如 `video`, `audio`, `min_pixels`, `max_pixels`, `total_max_pixels`, ... (+8 more) 编码了默认值或元数据。

### Lines 113-116: Class `Content`
```python
class Content:
    type: Literal["text", "image", "video", "audio", "video_audio"]
    content: Any
    is_target: bool | None = None
```
**EN:** Defines `Content`, a processor-related class. Class attributes such as `type`, `content`, `is_target` encode defaults or metadata.
**CN:** 定义 `Content`，这是一个处理器相关类。 类属性如 `type`, `content`, `is_target` 编码了默认值或元数据。

### Lines 120-136: Class `MiMoVLInputSample`
```python
class MiMoVLInputSample:
    input_ids: torch.Tensor
    labels: torch.Tensor | None
    pixel_values: list[torch.Tensor]
    pixel_values_videos: list[torch.Tensor]
    image_thw_grids: list[torch.Tensor]
    video_thw_grids: list[torch.Tensor]
    audio_inputs: list[torch.Tensor]
    second_per_grid_ts: list[float] = field(default_factory=list)
    video_start_times: list[float] = field(default_factory=list)
    audio_token_lens: list[int] = field(default_factory=list)
    va_audio_inputs: list[torch.Tensor] = field(default_factory=list)
    video_audio_n_segs: list[int] = field(default_factory=list)
    video_audio_seg_lens: list[int] = field(default_factory=list)
    position_ids: torch.Tensor | None = None
    rope_deltas: torch.Tensor | None = None
    extra: dict = field(default_factory=dict)
```
**EN:** Defines `MiMoVLInputSample`, a processor-related class. Class attributes such as `input_ids`, `labels`, `pixel_values`, `pixel_values_videos`, `image_thw_grids`, ... (+11 more) encode defaults or metadata.
**CN:** 定义 `MiMoVLInputSample`，这是一个处理器相关类。 类属性如 `input_ids`, `labels`, `pixel_values`, `pixel_values_videos`, `image_thw_grids`, ... (+11 more) 编码了默认值或元数据。

### Lines 231-249: Function `_fetch_image`
```python
def _fetch_image(src: Any) -> Image.Image:
    if isinstance(src, Image.Image):
        return _to_rgb(src)
    if isinstance(src, bytes):
        return _to_rgb(copy.deepcopy(Image.open(BytesIO(src))))
    if isinstance(src, str):
        if src.startswith(("http://", "https://")):
            r = requests.get(src, timeout=30)
            r.raise_for_status()
            return _to_rgb(copy.deepcopy(Image.open(BytesIO(r.content))))
        if src.startswith("file://"):
            return _to_rgb(Image.open(src[7:]))
        if src.startswith("data:image"):
            import pybase64 as _b64

            _, b64 = src.split("base64,", 1)
            return _to_rgb(copy.deepcopy(Image.open(BytesIO(_b64.b64decode(b64)))))
        return _to_rgb(Image.open(src))
    raise ValueError(f"Unrecognized image source: {type(src)}")
```
**EN:** This private function implements `_fetch_image`. Main inputs include `src`.
**CN:** 该私有函数实现 `_fetch_image` 相关逻辑。 主要输入参数包括 `src`。

### Lines 257-890: Class `MiMoVLProcessor`
```python
class MiMoVLProcessor:
    """Core MiMo-VL multimodal processor.

    Handles image/video/audio preprocessing and token sequence construction.
    Ported from SGLang's MiMoVLProcessor.
    """

    def __init__(
        self,
        tokenizer: Any,
        patch_size: int = 14,
        merge_size: int = 2,
        temporal_patch_size: int = 2,
        temporal_compression_ratio: int = 1,
        use_video_timestamps: bool = True,
        video_audio_interleave_length: int = 0,
        audio_kernel_size: int = 3,
        audio_stride_size: int = 2,
        audio_avg_pooler: int = 2,
        audio_sampling_rate: int = 24000,
        audio_nfft: int = 960,
        audio_hop_length: int = 240,
        audio_window_size: int = 960,
        audio_fmin: float = 0.0,
        audio_fmax: float | None = None,
        audio_n_mels: int = 128,
        audio_segment_size: int = 6000,
        audio_channels: int = 8,
        audio_group_size: int = 4,
        audio_input_id_per_second: float = 25.0,
        audio_zeroemb_idx: int = 4096,
        image_min_pixels: int | None = None,
        image_max_pixels: int | None = None,
        video_min_pixels: int | None = None,
        video_max_pixels: int | None = None,
        video_total_max_pixels: int | None = None,
        fps: float | None = None,
        num_frames: int | None = None,
        max_frames: int | None = None,
        min_frames: int | None = None,
# ... omitted for brevity ...
            video_thw_grids=vid_grids,
            audio_inputs=audio_inputs,
            second_per_grid_ts=second_per_grid_ts,
            video_start_times=video_start_times,
            audio_token_lens=audio_token_lens,
            va_audio_inputs=va_audio_inputs,
            video_audio_n_segs=video_audio_n_segs,
            video_audio_seg_lens=video_audio_seg_lens,
            position_ids=position_ids,
            rope_deltas=rope_deltas,
            extra=extra,
        )
```
**EN:** Defines `MiMoVLProcessor`, a processor-related class. The class docstring highlights that Core MiMo-VL multimodal processor. Key methods include `__init__`, `mel_spectrogram`, `_resolve_img_kw`, `_resolve_vid_kw`, `preprocess_audio`, ... (+5 more).
**CN:** 定义 `MiMoVLProcessor`，这是一个处理器相关类。 类文档字符串进一步说明了它的职责。 关键方法包括 `__init__`, `mel_spectrogram`, `_resolve_img_kw`, `_resolve_vid_kw`, `preprocess_audio`, ... (+5 more)。

### Lines 898-1285: Class `MiMoOmniProcessor`
```python
class MiMoOmniProcessor(ProcessorMixin):
    """HuggingFace-compatible ProcessorMixin wrapper for MiMo-Omni.

    Accepts PIL images, pre-decoded video tuples (frames_TCHW, timestamps_T),
    and audio (file path / bytes / (waveform, sr) tuple / numpy array).
    """

    attributes = ["tokenizer"]
    tokenizer_class = "AutoTokenizer"

    # Single or multi-pad placeholders produced by the chat template / prior expansion
    _IMG_RE = re.compile(r"<\|vision_start\|>(?:<\|image_pad\|>)+<\|vision_end\|>")
    _VID_RE = re.compile(r"<\|vision_start\|>(?:<\|video_pad\|>)+<\|vision_end\|>")
    _AUD_RE = re.compile(
        r"<\|mimo_audio_start\|>(?:<\|audio_pad\|>)+<\|mimo_audio_end\|>"
    )

    _MM_RE = re.compile(
        r"(<\|vision_start\|>(?:<\|image_pad\|>)+<\|vision_end\|>"
        r"|<\|vision_start\|>(?:<\|video_pad\|>)+<\|vision_end\|>"
        r"|<\|mimo_audio_start\|>(?:<\|audio_pad\|>)+<\|mimo_audio_end\|>)"
    )

    def __init__(
        self,
        tokenizer: Any,
        *,
        patch_size: int = 14,
        merge_size: int = 2,
        temporal_patch_size: int = 2,
        temporal_compression_ratio: int = 1,
        image_min_pixels: int | None = None,
        image_max_pixels: int | None = None,
        video_min_pixels: int | None = None,
        video_max_pixels: int | None = None,
        video_total_max_pixels: int | None = None,
        fps: float = 2.0,
        num_frames: int | None = None,
        max_frames: int = 256,
        min_frames: int = 8,
# ... omitted for brevity ...
            if sample.audio_token_lens:
                data["audio_token_lens"] = torch.tensor(
                    sample.audio_token_lens, dtype=torch.long
                )

        bf = BatchFeature(data=data, tensor_type=return_tensors)
        if audio_features is not None:
            bf["audio_features"] = audio_features
        # va_audio_features: list of mel-spec tensors (one per video_audio item)
        if sample.va_audio_inputs:
            bf["va_audio_features"] = sample.va_audio_inputs
        return bf
```
**EN:** Defines `MiMoOmniProcessor`, a processor-related class derived from `ProcessorMixin`. The class docstring highlights that HuggingFace-compatible ProcessorMixin wrapper for MiMo-Omni. Key methods include `__init__`, `from_hf_config`, `image_token`, `video_token`, `image_processor`, ... (+2 more).
**CN:** 定义 `MiMoOmniProcessor`，这是一个处理器相关类，继承自 `ProcessorMixin`。 类文档字符串进一步说明了它的职责。 关键方法包括 `__init__`, `from_hf_config`, `image_token`, `video_token`, `image_processor`, ... (+2 more)。

## Key Concepts / 关键概念
- **EN:** Processor wrappers unify tokenizers and modality-specific preprocessors behind a single serving-oriented interface.
- **CN:** 处理器封装把 tokenizer 与模态专用预处理器统一到面向服务的单一接口之下。
- **EN:** The module is tightly coupled to Hugging Face Transformers APIs, so version compatibility is often an explicit concern.
- **CN:** 该模块与 Hugging Face Transformers API 紧密耦合，因此版本兼容性通常是显式关注点。
- **EN:** Torch types and tensor conventions influence preprocessing, dtype selection, or runtime metadata in this module.
- **CN:** Torch 的类型与张量约定会影响本模块中的预处理、数据类型选择或运行时元数据。

## Dependencies / 依赖关系
- **EN:** Standard library modules: `contextlib`, `copy`, `io`, `logging`, `math`, `collections`, `concurrent.futures`, `dataclasses`, `typing`.
- **CN:** 标准库模块：`contextlib`, `copy`, `io`, `logging`, `math`, `collections`, `concurrent.futures`, `dataclasses`, `typing`。
- **EN:** External packages: `numpy`, `regex`, `requests`, `torch`, `torch.nn.functional`, `PIL`, `transformers`, `transformers.processing_utils`, `torchcodec.decoders`, `torchaudio`, ... (+2 more).
- **CN:** 外部依赖包：`numpy`, `regex`, `requests`, `torch`, `torch.nn.functional`, `PIL`, `transformers`, `transformers.processing_utils`, `torchcodec.decoders`, `torchaudio`, ... (+2 more)。
- **EN:** Internal modules: none.
- **CN:** 内部模块：无。
