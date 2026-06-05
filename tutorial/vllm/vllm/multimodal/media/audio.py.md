# audio.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/multimodal/media/audio.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements audio support for the `media` portion of vLLM. / 为 vLLM 的 `media` 子目录实现与 音频 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-17)
```python
import math

from io import BytesIO

from pathlib import Path

import numpy as np

import numpy.typing as npt

import pybase64

import torch

from vllm.logger import init_logger

from vllm.multimodal.audio import resample_audio_pyav

from vllm.utils.import_utils import PlaceholderModule

from vllm.utils.serial_utils import tensor2base64

from .base import MediaIO
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 19-42)
```python
logger = init_logger(__name__)

_BAD_SF_CODES = {0, 1, 3, 4}
```
**EN:** This constant/configuration block defines `logger`, `_BAD_SF_CODES`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `logger`, `_BAD_SF_CODES`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Function `load_audio_pyav` (lines 45-109)
```python
def load_audio_pyav(
    path: BytesIO | Path | str,
    *,
    sr: float | None = 22050,
    mono: bool = True,
) -> tuple[npt.NDArray, float]:
    """Load an audio file using PyAV (FFmpeg), returning float32 mono waveform.

    Decodes the audio stream at its native sample rate. Channel reduction to
    mono is performed by averaging across channels.  Resampling to a
    model-specific rate is left to the downstream :class:`AudioResampler`.

    Args:
        path: A :class:`~io.BytesIO` buffer, a filesystem
            :class:`~pathlib.Path`, or a string path.

    Returns:
        ``(waveform, sample_rate)`` where *waveform* is a 1-D float32
        NumPy array and *sample_rate* is the native sample rate in Hz.
    """
    native_sr = None
    try:
        with av.open(path) as container:
            if not container.streams.audio:
                raise ValueError("No audio stream found.")
    # ... omitted for brevity ...

    return audio, sr
```
**EN:** Function `load_audio_pyav` handles loading or retrieval of external/internal data. The docstring highlights: Load an audio file using PyAV (FFmpeg), returning float32 mono waveform. Key calls such as `av.open`, `ValueError`, `math.isclose`, `float`, `av.AudioResampler` show the concrete execution path.
**CN:** Function `load_audio_pyav` 负责加载或获取外部/内部数据。 文档字符串强调：Load an audio file using PyAV (FFmpeg), returning float32 mono waveform. 像 `av.open`, `ValueError`, `math.isclose`, `float`, `av.AudioResampler` 这样的关键调用展示了该代码块的具体执行路径。

### Function `load_audio_soundfile` (lines 112-129)
```python
def load_audio_soundfile(
    path: BytesIO | Path | str,
    *,
    sr: float | None = 22050,
    mono: bool = True,
) -> tuple[np.ndarray, int]:
    """Load audio via soundfile"""
    with soundfile.SoundFile(path) as f:
        native_sr = f.samplerate
        y = f.read(dtype="float32", always_2d=False).T

    if mono and y.ndim > 1:
        y = np.mean(y, axis=tuple(range(y.ndim - 1)))

    if sr is not None and sr != native_sr:
        y = resample_audio_pyav(y, orig_sr=native_sr, target_sr=sr)
        return y, int(sr)
    return y, native_sr
```
**EN:** Function `load_audio_soundfile` handles loading or retrieval of external/internal data. The docstring highlights: Load audio via soundfile Key calls such as `soundfile.SoundFile`, `f.read`, `np.mean`, `tuple`, `range` show the concrete execution path.
**CN:** Function `load_audio_soundfile` 负责加载或获取外部/内部数据。 文档字符串强调：Load audio via soundfile 像 `soundfile.SoundFile`, `f.read`, `np.mean`, `tuple`, `range` 这样的关键调用展示了该代码块的具体执行路径。

### Function `load_audio` (lines 132-160)
```python
def load_audio(
    path: BytesIO | Path | str,
    *,
    sr: float | None = 22050,
    mono: bool = True,
):
    try:
        return load_audio_soundfile(path, sr=sr, mono=mono)
    except ImportError as exc:
        # soundfile (or resampy) is not installed — fall through to pyav.
        # NOTE: this clause must stay BEFORE ``soundfile.LibsndfileError``
        # because when soundfile is a PlaceholderModule, evaluating
        # ``soundfile.LibsndfileError`` itself raises ImportError.
        logger.error("Failed to load audio via soundfile: %r", exc)
    except soundfile.LibsndfileError as exc:
        # Only fall back for known format-detection failures.
        # Re-raise anything else (e.g. corrupt but recognised format).
        if exc.code not in _BAD_SF_CODES:
            raise
    # soundfile may have advanced the BytesIO seek position before failing;
    # reset it so PyAV can read from the beginning.
    if isinstance(path, BytesIO):
        path.seek(0)
    try:
        return load_audio_pyav(path, sr=sr, mono=mono)
    # ... omitted for brevity ...
    except Exception as pyav_exc:
        raise ValueError("Invalid or unsupported audio file.") from pyav_exc
```
**EN:** Function `load_audio` handles loading or retrieval of external/internal data. Key calls such as `load_audio_soundfile`, `logger.error`, `isinstance`, `path.seek`, `load_audio_pyav` show the concrete execution path.
**CN:** Function `load_audio` 负责加载或获取外部/内部数据。 像 `load_audio_soundfile`, `logger.error`, `isinstance`, `path.seek`, `load_audio_pyav` 这样的关键调用展示了该代码块的具体执行路径。

### Class `AudioMediaIO` (lines 163-205)
```python
class AudioMediaIO(MediaIO[tuple[npt.NDArray, float]]):
    """Configuration values can be user-provided either by --media-io-kwargs or
    by the runtime API field "media_io_kwargs". Ensure proper validation and
    error handling.
    """

    def __init__(self, **kwargs) -> None:
        super().__init__()

        # `kwargs` contains custom arguments from
        # --media-io-kwargs for this modality, merged with
        # per-request runtime media_io_kwargs via merge_kwargs().
        # They can be passed to the underlying
        # media loaders (e.g. custom implementations)
        # for flexible control.
        self.kwargs = kwargs

    def load_bytes(self, data: bytes) -> tuple[npt.NDArray, float]:
        return load_audio(BytesIO(data), sr=None)

    def load_base64(
        self,
        media_type: str,
    # ... omitted for brevity ...

        return pybase64.b64encode(data).decode("utf-8")
```
**EN:** Class `AudioMediaIO` is a structured building block in this module. It inherits from `MediaIO[tuple[npt.NDArray, float]]`. Key methods include `__init__`, `load_bytes`, `load_base64`, `load_file`, `encode_base64`, which define initialization, validation, transformation, or access patterns. The class docstring says: Configuration values can be user-provided either by --media-io-kwargs or by the runtime API field "media_io_kwargs".
**CN:** 类 `AudioMediaIO` 是该模块中的结构化构件，继承自 `MediaIO[tuple[npt.NDArray, float]]`。 关键方法包括 `__init__`, `load_bytes`, `load_base64`, `load_file`, `encode_base64`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Configuration values can be user-provided either by --media-io-kwargs or by the runtime API field "media_io_kwargs".

### Method `AudioMediaIO.__init__` (lines 169-178)
```python
    def __init__(self, **kwargs) -> None:
        super().__init__()

        # `kwargs` contains custom arguments from
        # --media-io-kwargs for this modality, merged with
        # per-request runtime media_io_kwargs via merge_kwargs().
        # They can be passed to the underlying
        # media loaders (e.g. custom implementations)
        # for flexible control.
        self.kwargs = kwargs
```
**EN:** Method `AudioMediaIO.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `super().__init__`, `super` show the concrete execution path.
**CN:** Method `AudioMediaIO.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `super().__init__`, `super` 这样的关键调用展示了该代码块的具体执行路径。

### Method `AudioMediaIO.load_bytes` (lines 180-181)
```python
    def load_bytes(self, data: bytes) -> tuple[npt.NDArray, float]:
        return load_audio(BytesIO(data), sr=None)
```
**EN:** Method `AudioMediaIO.load_bytes` handles loading or retrieval of external/internal data. Key calls such as `load_audio`, `BytesIO` show the concrete execution path.
**CN:** Method `AudioMediaIO.load_bytes` 负责加载或获取外部/内部数据。 像 `load_audio`, `BytesIO` 这样的关键调用展示了该代码块的具体执行路径。

### Method `AudioMediaIO.load_base64` (lines 183-188)
```python
    def load_base64(
        self,
        media_type: str,
        data: str,
    ) -> tuple[npt.NDArray, float]:
        return self.load_bytes(pybase64.b64decode(data))
```
**EN:** Method `AudioMediaIO.load_base64` handles loading or retrieval of external/internal data. Key calls such as `self.load_bytes`, `pybase64.b64decode` show the concrete execution path.
**CN:** Method `AudioMediaIO.load_base64` 负责加载或获取外部/内部数据。 像 `self.load_bytes`, `pybase64.b64decode` 这样的关键调用展示了该代码块的具体执行路径。

### Method `AudioMediaIO.load_file` (lines 190-191)
```python
    def load_file(self, filepath: Path) -> tuple[npt.NDArray, float]:
        return load_audio(filepath, sr=None)
```
**EN:** Method `AudioMediaIO.load_file` handles loading or retrieval of external/internal data. Key calls such as `load_audio` show the concrete execution path.
**CN:** Method `AudioMediaIO.load_file` 负责加载或获取外部/内部数据。 像 `load_audio` 这样的关键调用展示了该代码块的具体执行路径。

### Method `AudioMediaIO.encode_base64` (lines 193-205)
```python
    def encode_base64(
        self,
        media: tuple[npt.NDArray, int],
        *,
        audio_format: str = "WAV",
    ) -> str:
        audio, sr = media

        with BytesIO() as buffer:
            soundfile.write(buffer, audio, sr, format=audio_format)
            data = buffer.getvalue()

        return pybase64.b64encode(data).decode("utf-8")
```
**EN:** Method `AudioMediaIO.encode_base64` implements an encoding/decoding or token-transformation step. Key calls such as `BytesIO`, `soundfile.write`, `buffer.getvalue`, `pybase64.b64encode(data).decode`, `pybase64.b64encode` show the concrete execution path.
**CN:** Method `AudioMediaIO.encode_base64` 实现编码/解码或 Token 变换步骤。 像 `BytesIO`, `soundfile.write`, `buffer.getvalue`, `pybase64.b64encode(data).decode`, `pybase64.b64encode` 这样的关键调用展示了该代码块的具体执行路径。

### Class `AudioEmbeddingMediaIO` (lines 208-236)
```python
class AudioEmbeddingMediaIO(MediaIO[torch.Tensor]):
    """Configuration values can be user-provided either by --media-io-kwargs or
    by the runtime API field "media_io_kwargs". Ensure proper validation and
    error handling.
    """

    def __init__(self) -> None:
        super().__init__()

    def load_bytes(self, data: bytes) -> torch.Tensor:
        buffer = BytesIO(data)
        # Enable sparse tensor integrity checks to prevent out-of-bounds
        # writes from maliciously crafted tensors
        with torch.sparse.check_sparse_tensor_invariants():
            tensor = torch.load(buffer, weights_only=True)
            return tensor.to_dense()

    def load_base64(self, media_type: str, data: str) -> torch.Tensor:
        return self.load_bytes(pybase64.b64decode(data, validate=True))

    def load_file(self, filepath: Path) -> torch.Tensor:
        # Enable sparse tensor integrity checks to prevent out-of-bounds
        # writes from maliciously crafted tensors
    # ... omitted for brevity ...
    def encode_base64(self, media: torch.Tensor) -> str:
        return tensor2base64(media)
```
**EN:** Class `AudioEmbeddingMediaIO` is a structured building block in this module. It inherits from `MediaIO[torch.Tensor]`. Key methods include `__init__`, `load_bytes`, `load_base64`, `load_file`, `encode_base64`, which define initialization, validation, transformation, or access patterns. The class docstring says: Configuration values can be user-provided either by --media-io-kwargs or by the runtime API field "media_io_kwargs".
**CN:** 类 `AudioEmbeddingMediaIO` 是该模块中的结构化构件，继承自 `MediaIO[torch.Tensor]`。 关键方法包括 `__init__`, `load_bytes`, `load_base64`, `load_file`, `encode_base64`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Configuration values can be user-provided either by --media-io-kwargs or by the runtime API field "media_io_kwargs".

### Method `AudioEmbeddingMediaIO.__init__` (lines 214-215)
```python
    def __init__(self) -> None:
        super().__init__()
```
**EN:** Method `AudioEmbeddingMediaIO.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `super().__init__`, `super` show the concrete execution path.
**CN:** Method `AudioEmbeddingMediaIO.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `super().__init__`, `super` 这样的关键调用展示了该代码块的具体执行路径。

### Method `AudioEmbeddingMediaIO.load_bytes` (lines 217-223)
```python
    def load_bytes(self, data: bytes) -> torch.Tensor:
        buffer = BytesIO(data)
        # Enable sparse tensor integrity checks to prevent out-of-bounds
        # writes from maliciously crafted tensors
        with torch.sparse.check_sparse_tensor_invariants():
            tensor = torch.load(buffer, weights_only=True)
            return tensor.to_dense()
```
**EN:** Method `AudioEmbeddingMediaIO.load_bytes` handles loading or retrieval of external/internal data. Key calls such as `BytesIO`, `torch.sparse.check_sparse_tensor_invariants`, `torch.load`, `tensor.to_dense` show the concrete execution path.
**CN:** Method `AudioEmbeddingMediaIO.load_bytes` 负责加载或获取外部/内部数据。 像 `BytesIO`, `torch.sparse.check_sparse_tensor_invariants`, `torch.load`, `tensor.to_dense` 这样的关键调用展示了该代码块的具体执行路径。

### Method `AudioEmbeddingMediaIO.load_base64` (lines 225-226)
```python
    def load_base64(self, media_type: str, data: str) -> torch.Tensor:
        return self.load_bytes(pybase64.b64decode(data, validate=True))
```
**EN:** Method `AudioEmbeddingMediaIO.load_base64` handles loading or retrieval of external/internal data. Key calls such as `self.load_bytes`, `pybase64.b64decode` show the concrete execution path.
**CN:** Method `AudioEmbeddingMediaIO.load_base64` 负责加载或获取外部/内部数据。 像 `self.load_bytes`, `pybase64.b64decode` 这样的关键调用展示了该代码块的具体执行路径。

### Method `AudioEmbeddingMediaIO.load_file` (lines 228-233)
```python
    def load_file(self, filepath: Path) -> torch.Tensor:
        # Enable sparse tensor integrity checks to prevent out-of-bounds
        # writes from maliciously crafted tensors
        with torch.sparse.check_sparse_tensor_invariants():
            tensor = torch.load(filepath, weights_only=True)
            return tensor.to_dense()
```
**EN:** Method `AudioEmbeddingMediaIO.load_file` handles loading or retrieval of external/internal data. Key calls such as `torch.sparse.check_sparse_tensor_invariants`, `torch.load`, `tensor.to_dense` show the concrete execution path.
**CN:** Method `AudioEmbeddingMediaIO.load_file` 负责加载或获取外部/内部数据。 像 `torch.sparse.check_sparse_tensor_invariants`, `torch.load`, `tensor.to_dense` 这样的关键调用展示了该代码块的具体执行路径。

### Method `AudioEmbeddingMediaIO.encode_base64` (lines 235-236)
```python
    def encode_base64(self, media: torch.Tensor) -> str:
        return tensor2base64(media)
```
**EN:** Method `AudioEmbeddingMediaIO.encode_base64` implements an encoding/decoding or token-transformation step. Key calls such as `tensor2base64` show the concrete execution path.
**CN:** Method `AudioEmbeddingMediaIO.encode_base64` 实现编码/解码或 Token 变换步骤。 像 `tensor2base64` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。
- **Multimodality / 多模态**
  - **EN:** The code contains modality-specific paths for text, image, audio, or video data.
  - **CN:** 代码包含面向文本、图像、音频或视频数据的模态专用路径。
- **Configuration / 配置**
  - **EN:** Configuration objects and validation rules centralize feature toggles and runtime settings.
  - **CN:** 配置对象与校验规则集中管理功能开关和运行时设置。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import math`, `from io import BytesIO`, `from pathlib import Path`
- **Third-party / 第三方**: `import numpy as np`, `import numpy.typing as npt`, `import pybase64`, `import torch`
- **vLLM internal / vLLM 内部依赖**: `from vllm.logger import init_logger`, `from vllm.multimodal.audio import resample_audio_pyav`, `from vllm.utils.import_utils import PlaceholderModule`, `from vllm.utils.serial_utils import tensor2base64`, `from .base import MediaIO`
