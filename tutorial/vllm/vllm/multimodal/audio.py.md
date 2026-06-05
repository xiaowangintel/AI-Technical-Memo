# audio.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/multimodal/audio.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements audio support for the `multimodal` portion of vLLM. / 为 vLLM 的 `multimodal` 子目录实现与 音频 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-12)
```python
import math

from dataclasses import dataclass

from enum import Enum

from typing import Literal

import numpy as np

import numpy.typing as npt

import torch

from vllm.utils.import_utils import PlaceholderModule
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 82-83)
```python
MONO_AUDIO_SPEC = AudioSpec(target_channels=1, channel_reduction=ChannelReduction.MEAN)

PASSTHROUGH_AUDIO_SPEC = AudioSpec(target_channels=None)
```
**EN:** This constant/configuration block defines `MONO_AUDIO_SPEC`, `PASSTHROUGH_AUDIO_SPEC`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `MONO_AUDIO_SPEC`, `PASSTHROUGH_AUDIO_SPEC`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Function `get_audio_duration` (lines 27-38)
```python
def get_audio_duration(*, y: npt.NDArray[np.floating], sr: float = 22050) -> float:
    """Get the duration of an audio array in seconds.

    Args:
        y: Audio time series. Can be 1D (samples,) or 2D (channels, samples).
        sr: Sample rate of the audio in Hz.

    Returns:
        Duration of the audio in seconds.
    """
    n_samples = y.shape[-1]
    return float(n_samples) / sr
```
**EN:** Function `get_audio_duration` works with modality-specific preprocessing or transport logic. The docstring highlights: Get the duration of an audio array in seconds. Key calls such as `float` show the concrete execution path.
**CN:** Function `get_audio_duration` 处理模态相关的预处理或传输逻辑。 文档字符串强调：Get the duration of an audio array in seconds. 像 `float` 这样的关键调用展示了该代码块的具体执行路径。

### Class `ChannelReduction` (lines 41-47)
```python
class ChannelReduction(str, Enum):
    """Method to reduce multi-channel audio to target channels."""

    MEAN = "mean"  # Average across channels (default, preserves energy balance)
    FIRST = "first"  # Take first channel only
    MAX = "max"  # Take max value across channels
    SUM = "sum"  # Sum across channels
```
**EN:** Class `ChannelReduction` is a structured building block in this module. It inherits from `str`, `Enum`. The class docstring says: Method to reduce multi-channel audio to target channels.
**CN:** 类 `ChannelReduction` 是该模块中的结构化构件，继承自 `str`, `Enum`。 类文档说明：Method to reduce multi-channel audio to target channels.

### Class `AudioSpec` (lines 51-78)
```python
class AudioSpec:
    """Specification for target audio format.

    This dataclass defines the expected audio format for a model's feature
    extractor. It is used to normalize audio data before processing.

    Attributes:
        target_channels: Number of output channels. None means passthrough
            (no normalization). 1 = mono, 2 = stereo, etc.
        channel_reduction: Method to reduce channels when input has more
            channels than target. Only used when reducing channels.
    """

    target_channels: int | None = 1
    channel_reduction: ChannelReduction = ChannelReduction.MEAN

    @property
    def needs_normalization(self) -> bool:
        """Whether audio normalization is needed."""
        return self.target_channels is not None

    def __repr__(self) -> str:
        if self.target_channels is None:
    # ... omitted for brevity ...
            f"reduction={self.channel_reduction.value})"
        )
```
**EN:** Class `AudioSpec` is a structured building block in this module and uses a dataclass-style declaration to store explicit state. Key methods include `needs_normalization`, `__repr__`, which define initialization, validation, transformation, or access patterns. The class docstring says: Specification for target audio format.
**CN:** 类 `AudioSpec` 是该模块中的结构化构件，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `needs_normalization`, `__repr__`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Specification for target audio format.

### Method `AudioSpec.needs_normalization` (lines 68-70)
```python
    def needs_normalization(self) -> bool:
        """Whether audio normalization is needed."""
        return self.target_channels is not None
```
**EN:** Method `AudioSpec.needs_normalization` exposes a computed property so callers can access derived state without duplicating logic. The docstring highlights: Whether audio normalization is needed.
**CN:** Method `AudioSpec.needs_normalization` 暴露计算属性，使调用方无需重复逻辑即可访问派生状态。 文档字符串强调：Whether audio normalization is needed.

### Method `AudioSpec.__repr__` (lines 72-78)
```python
    def __repr__(self) -> str:
        if self.target_channels is None:
            return "AudioSpec(passthrough)"
        return (
            f"AudioSpec(channels={self.target_channels}, "
            f"reduction={self.channel_reduction.value})"
        )
```
**EN:** Method `AudioSpec.__repr__` provides a reusable helper around the module's main workflow.
**CN:** Method `AudioSpec.__repr__` 为模块主流程提供可复用的辅助逻辑。

### Function `normalize_audio` (lines 86-161)
```python
def normalize_audio(
    audio: npt.NDArray[np.floating] | torch.Tensor,
    spec: AudioSpec,
) -> npt.NDArray[np.floating] | torch.Tensor:
    """Normalize audio to the specified format.

    This function handles channel reduction for multi-channel audio,
    supporting both numpy arrays and torch tensors.

    Args:
        audio: Input audio data. Can be:
            - 1D array/tensor: (time,) - already mono
            - 2D array/tensor: (channels, time) - standard format from torchaudio
            - 2D array/tensor: (time, channels) - format from soundfile
              (will be auto-detected and transposed if time > channels)
        spec: AudioSpec defining the target format.

    Returns:
        Normalized audio in the same type as input (numpy or torch).
        For mono output (target_channels=1), returns 1D array/tensor.

    Raises:
        ValueError: If audio has unsupported dimensions or channel expansion
            is requested (e.g., mono to stereo).
    """
    # ... omitted for brevity ...
        # For now, just take first N channels
        return audio[: spec.target_channels]
```
**EN:** Function `normalize_audio` works with modality-specific preprocessing or transport logic. The docstring highlights: Normalize audio to the specified format. Key calls such as `ValueError`, `isinstance`, `np.mean`, `audio.mean`, `np.max` show the concrete execution path.
**CN:** Function `normalize_audio` 处理模态相关的预处理或传输逻辑。 文档字符串强调：Normalize audio to the specified format. 像 `ValueError`, `isinstance`, `np.mean`, `audio.mean`, `np.max` 这样的关键调用展示了该代码块的具体执行路径。

### Function `resample_audio_pyav` (lines 169-224)
```python
def resample_audio_pyav(
    audio: npt.NDArray[np.floating],
    *,
    orig_sr: float,
    target_sr: float,
) -> npt.NDArray[np.floating]:
    """Resample audio using PyAV (libswresample via FFmpeg).

    Args:
        audio: Input audio. Can be:
            - 1D array ``(samples,)``: mono audio
            - 2D array ``(channels, samples)``: stereo audio
        orig_sr: Original sample rate in Hz.
        target_sr: Target sample rate in Hz.

    Returns:
        Resampled audio with the same shape as the input (1D → 1D, 2D → 2D).
    """
    orig_sr_int = int(round(orig_sr))
    target_sr_int = int(round(target_sr))

    if orig_sr_int == target_sr_int:
        return audio

    if audio.ndim == 2:
    # ... omitted for brevity ...
    result = np.concatenate([f.to_ndarray() for f in out_frames], axis=1).squeeze(0)
    return result[:expected_len]
```
**EN:** Function `resample_audio_pyav` samples, slices, or reshapes data for downstream use. The docstring highlights: Resample audio using PyAV (libswresample via FFmpeg). Key calls such as `int`, `round`, `np.stack`, `resample_audio_pyav`, `math.ceil` show the concrete execution path.
**CN:** Function `resample_audio_pyav` 负责为下游流程采样、切片或重组数据。 文档字符串强调：Resample audio using PyAV (libswresample via FFmpeg). 像 `int`, `round`, `np.stack`, `resample_audio_pyav`, `math.ceil` 这样的关键调用展示了该代码块的具体执行路径。

### Function `resample_audio_scipy` (lines 227-245)
```python
def resample_audio_scipy(
    audio: npt.NDArray[np.floating],
    *,
    orig_sr: float,
    target_sr: float,
) -> npt.NDArray[np.floating]:
    orig_sr_int = int(round(orig_sr))
    target_sr_int = int(round(target_sr))

    if orig_sr_int == target_sr_int:
        return audio

    gcd = math.gcd(orig_sr_int, target_sr_int)
    return scipy_signal.resample_poly(
        audio,
        target_sr_int // gcd,
        orig_sr_int // gcd,
        axis=-1,
    )
```
**EN:** Function `resample_audio_scipy` samples, slices, or reshapes data for downstream use. Key calls such as `int`, `round`, `math.gcd`, `scipy_signal.resample_poly` show the concrete execution path.
**CN:** Function `resample_audio_scipy` 负责为下游流程采样、切片或重组数据。 像 `int`, `round`, `math.gcd`, `scipy_signal.resample_poly` 这样的关键调用展示了该代码块的具体执行路径。

### Class `AudioResampler` (lines 248-286)
```python
class AudioResampler:
    """Resample audio data to a target sample rate."""

    def __init__(
        self,
        target_sr: float | None = None,
        method: Literal["pyav", "scipy"] = "pyav",
    ):
        self.target_sr = target_sr
        self.method = method

    def resample(
        self,
        audio: npt.NDArray[np.floating],
        *,
        orig_sr: float,
    ) -> npt.NDArray[np.floating]:
        if self.target_sr is None:
            raise RuntimeError(
                "Audio resampling is not supported when `target_sr` is not provided"
            )
        if math.isclose(
            float(orig_sr),
    # ... omitted for brevity ...
                "Supported methods are 'pyav' and 'scipy'."
            )
```
**EN:** Class `AudioResampler` is a structured building block in this module. Key methods include `__init__`, `resample`, which define initialization, validation, transformation, or access patterns. The class docstring says: Resample audio data to a target sample rate.
**CN:** 类 `AudioResampler` 是该模块中的结构化构件。 关键方法包括 `__init__`, `resample`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Resample audio data to a target sample rate.

### Method `AudioResampler.__init__` (lines 251-257)
```python
    def __init__(
        self,
        target_sr: float | None = None,
        method: Literal["pyav", "scipy"] = "pyav",
    ):
        self.target_sr = target_sr
        self.method = method
```
**EN:** Method `AudioResampler.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls.
**CN:** Method `AudioResampler.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。

### Method `AudioResampler.resample` (lines 259-286)
```python
    def resample(
        self,
        audio: npt.NDArray[np.floating],
        *,
        orig_sr: float,
    ) -> npt.NDArray[np.floating]:
        if self.target_sr is None:
            raise RuntimeError(
                "Audio resampling is not supported when `target_sr` is not provided"
            )
        if math.isclose(
            float(orig_sr),
            float(self.target_sr),
            rel_tol=0.0,
            abs_tol=1e-6,
        ):
            return audio
        if self.method == "pyav":
            return resample_audio_pyav(audio, orig_sr=orig_sr, target_sr=self.target_sr)
    # ... omitted for brevity ...
                "Supported methods are 'pyav' and 'scipy'."
            )
```
**EN:** Method `AudioResampler.resample` samples, slices, or reshapes data for downstream use. Key calls such as `RuntimeError`, `math.isclose`, `float`, `resample_audio_pyav`, `resample_audio_scipy` show the concrete execution path.
**CN:** Method `AudioResampler.resample` 负责为下游流程采样、切片或重组数据。 像 `RuntimeError`, `math.isclose`, `float`, `resample_audio_pyav`, `resample_audio_scipy` 这样的关键调用展示了该代码块的具体执行路径。

### Function `split_audio` (lines 294-354)
```python
def split_audio(
    audio_data: np.ndarray,
    sample_rate: int,
    max_clip_duration_s: float,
    overlap_duration_s: float,
    min_energy_window_size: int,
) -> list[np.ndarray]:
    """Split audio into chunks with intelligent split points.

    Splits long audio into smaller chunks at low-energy regions to minimize
    cutting through speech. Uses overlapping windows to find quiet moments
    for splitting.

    Args:
        audio_data: Audio array to split. Can be 1D (mono) or multi-dimensional.
                   Splits along the last dimension (time axis).
        sample_rate: Sample rate of the audio in Hz.
        max_clip_duration_s: Maximum duration of each chunk in seconds.
        overlap_duration_s: Overlap duration in seconds between consecutive chunks.
                           Used to search for optimal split points.
        min_energy_window_size: Window size in samples for finding low-energy regions.

    Returns:
        List of audio chunks. Each chunk is a numpy array with the same shape
        as the input except for the last (time) dimension.
    # ... omitted for brevity ...

    return chunks
```
**EN:** Function `split_audio` works with modality-specific preprocessing or transport logic. The docstring highlights: Split audio into chunks with intelligent split points. Key calls such as `int`, `chunks.append`, `min`, `find_split_point` show the concrete execution path.
**CN:** Function `split_audio` 处理模态相关的预处理或传输逻辑。 文档字符串强调：Split audio into chunks with intelligent split points. 像 `int`, `chunks.append`, `min`, `find_split_point` 这样的关键调用展示了该代码块的具体执行路径。

### Function `find_split_point` (lines 357-404)
```python
def find_split_point(
    wav: np.ndarray,
    start_idx: int,
    end_idx: int,
    min_energy_window: int,
) -> int:
    """Find the best point to split audio by looking for silence or low amplitude.

    Searches for the quietest region within a specified range by calculating
    RMS energy in sliding windows.

    Args:
        wav: Audio array. Can be 1D or multi-dimensional.
        start_idx: Start index of search region (inclusive).
        end_idx: End index of search region (exclusive).
        min_energy_window: Window size in samples for energy calculation.

    Returns:
        Index of the quietest point within the search region. This is the
        recommended split point to minimize audio artifacts.

    Example:
        >>> audio = np.random.randn(32000)
        >>> # Insert quiet region
        >>> audio[16000:17600] = 0.01
    # ... omitted for brevity ...

    return quietest_idx
```
**EN:** Function `find_split_point` provides a reusable helper around the module's main workflow. The docstring highlights: Find the best point to split audio by looking for silence or low amplitude. Key calls such as `range`, `len`, `(window ** 2).mean` show the concrete execution path.
**CN:** Function `find_split_point` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Find the best point to split audio by looking for silence or low amplitude. 像 `range`, `len`, `(window ** 2).mean` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。
- **Multimodality / 多模态**
  - **EN:** The code contains modality-specific paths for text, image, audio, or video data.
  - **CN:** 代码包含面向文本、图像、音频或视频数据的模态专用路径。
- **Structured types / 结构化类型**
  - **EN:** Dataclasses, typed dictionaries, or aliases are used to make runtime contracts explicit.
  - **CN:** 通过 dataclass、类型化字典或别名来显式表达运行时契约。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import math`, `from dataclasses import dataclass`, `from enum import Enum`, `from typing import Literal`
- **Third-party / 第三方**: `import numpy as np`, `import numpy.typing as npt`, `import torch`
- **vLLM internal / vLLM 内部依赖**: `from vllm.utils.import_utils import PlaceholderModule`
