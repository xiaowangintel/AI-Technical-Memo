# video_decoder.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/video_decoder.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable SGLang runtime helpers for video decoding and preprocessing helpers. / 为 SGLang 运行时提供面向视频解码与预处理辅助逻辑的可复用辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Module setup and shared state / 模块设置与共享状态
```python
"""Unified video decoder: torchcodec preferred, decord as fallback."""

import logging

import numpy as np

logger = logging.getLogger(__name__)

try:
    from torchcodec.decoders import VideoDecoder

    _BACKEND = "torchcodec"
except (ImportError, RuntimeError):
    _BACKEND = "decord"


_cuda_backend_enabled: bool | None = None
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `logging`, `numpy`. It also defines symbols such as `logger`, `_BACKEND`, `_cuda_backend_enabled` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `logging`, `numpy`。 同时定义了 `logger`, `_BACKEND`, `_cuda_backend_enabled` 等符号，供后续逻辑使用。

### Lines 20-32: Function `_try_cuda_backend` / 函数 `_try_cuda_backend`
```python
def _try_cuda_backend() -> bool:
    """Try to enable torchcodec CUDA backend. Caches result after first call."""
    global _cuda_backend_enabled
    if _cuda_backend_enabled is not None:
        return _cuda_backend_enabled
    try:
        from torchcodec.decoders import set_cuda_backend

        set_cuda_backend("beta")
        _cuda_backend_enabled = True
    except Exception:
        _cuda_backend_enabled = False
    return _cuda_backend_enabled
```
**EN:** This function implements `_try_cuda_backend`. It primarily calls `set_cuda_backend` to complete its work. State updates are written into `_cuda_backend_enabled`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `_try_cuda_backend`。 它主要通过调用 `set_cuda_backend` 来完成任务。 状态更新主要写入 `_cuda_backend_enabled`。 实现中使用了条件分支、错误处理。

### Lines 35-39: Class `VideoDecoderWrapper` declaration / 类 `VideoDecoderWrapper` 声明
```python
class VideoDecoderWrapper:
    """Unified video decoder that uses torchcodec when available, decord as fallback.

    All frames are returned in NHWC uint8 numpy format for consistency.
    """
```
**EN:** This class establishes `VideoDecoderWrapper` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `__len__`, `__getitem__`, `avg_fps`, `get_frames_at`, `get_frames_as_tensor`.
**CN:** 该类将 `VideoDecoderWrapper` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `__len__`, `__getitem__`, `avg_fps`, `get_frames_at`, `get_frames_as_tensor` 等方法。

### Lines 41-76: Method `VideoDecoderWrapper.__init__` / 方法 `VideoDecoderWrapper.__init__`
```python
    def __init__(self, source, device: str = "cpu"):
        """source: file path (str) or video bytes.
        device: "cpu" or "cuda". GPU decoding only supported with torchcodec.
        """
        self._source_bytes = source if isinstance(source, bytes) else None
        self._source_path = source if isinstance(source, str) else None
        self._tmp_path = None
        if _BACKEND == "torchcodec":
            kwargs = {"dimension_order": "NHWC"}
            if device == "cuda" and _try_cuda_backend():
                kwargs["device"] = "cuda"
            try:
                self._decoder = VideoDecoder(source, **kwargs)
            except RuntimeError:
                if "device" in kwargs:
                    logger.warning("CUDA video decoding failed, falling back to CPU.")
                    kwargs.pop("device")
                    self._decoder = VideoDecoder(source, **kwargs)
                else:
                    raise
        else:
            from decord import VideoReader, cpu

            if isinstance(source, bytes):
                import os
                import tempfile

                fd, tmp_path = tempfile.mkstemp(suffix=".mp4")
                try:
                    os.write(fd, source)
                finally:
                    os.close(fd)
                self._tmp_path = tmp_path
                self._decoder = VideoReader(tmp_path, ctx=cpu(0))
            else:
                self._decoder = VideoReader(source, ctx=cpu(0))
```
**EN:** This method implements `__init__` on `VideoDecoderWrapper`. It primarily calls `isinstance`, `_try_cuda_backend`, `VideoDecoder`, `tempfile.mkstemp`, `VideoReader`, `os.write` to complete its work. State updates are written into `self._source_bytes`, `self._source_path`, `self._tmp_path`, `kwargs`, `self._decoder`. The implementation relies on conditional branches, error handling.
**CN:** 该方法（属于 `VideoDecoderWrapper`）实现了 `__init__`。 它主要通过调用 `isinstance`, `_try_cuda_backend`, `VideoDecoder`, `tempfile.mkstemp`, `VideoReader`, `os.write` 来完成任务。 状态更新主要写入 `self._source_bytes`, `self._source_path`, `self._tmp_path`, `kwargs`, `self._decoder`。 实现中使用了条件分支、错误处理。

### Lines 78-79: Method `VideoDecoderWrapper.__len__` / 方法 `VideoDecoderWrapper.__len__`
```python
    def __len__(self):
        return len(self._decoder)
```
**EN:** This method implements `__len__` on `VideoDecoderWrapper`. It primarily calls `len` to complete its work.
**CN:** 该方法（属于 `VideoDecoderWrapper`）实现了 `__len__`。 它主要通过调用 `len` 来完成任务。

### Lines 81-87: Method `VideoDecoderWrapper.__getitem__` / 方法 `VideoDecoderWrapper.__getitem__`
```python
    def __getitem__(self, idx):
        """Return single frame as numpy NHWC uint8."""
        if _BACKEND == "torchcodec":
            return self._decoder[idx].numpy()
        else:
            frame = self._decoder[idx]
            return frame.asnumpy() if hasattr(frame, "asnumpy") else np.array(frame)
```
**EN:** This method implements `__getitem__` on `VideoDecoderWrapper`. It primarily calls `self._decoder.numpy`, `hasattr`, `frame.asnumpy`, `np.array` to complete its work. State updates are written into `frame`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `VideoDecoderWrapper`）实现了 `__getitem__`。 它主要通过调用 `self._decoder.numpy`, `hasattr`, `frame.asnumpy`, `np.array` 来完成任务。 状态更新主要写入 `frame`。 实现中使用了条件分支。

### Lines 89-94: Method `VideoDecoderWrapper.avg_fps` / 方法 `VideoDecoderWrapper.avg_fps`
```python
    @property
    def avg_fps(self) -> float:
        if _BACKEND == "torchcodec":
            return self._decoder.metadata.average_fps
        else:
            return self._decoder.get_avg_fps()
```
**EN:** This method implements `avg_fps` on `VideoDecoderWrapper`. It primarily calls `self._decoder.get_avg_fps` to complete its work. The implementation relies on conditional branches.
**CN:** 该方法（属于 `VideoDecoderWrapper`）实现了 `avg_fps`。 它主要通过调用 `self._decoder.get_avg_fps` 来完成任务。 实现中使用了条件分支。

### Lines 96-102: Method `VideoDecoderWrapper.get_frames_at` / 方法 `VideoDecoderWrapper.get_frames_at`
```python
    def get_frames_at(self, indices: list) -> np.ndarray:
        """Return frames at given indices as numpy array with shape (N, H, W, C)."""
        if _BACKEND == "torchcodec":
            batch = self._decoder.get_frames_at(indices)
            return batch.data.numpy()
        else:
            return self._decoder.get_batch(indices).asnumpy()
```
**EN:** This method implements `get_frames_at` on `VideoDecoderWrapper`. It primarily calls `self._decoder.get_frames_at`, `batch.data.numpy`, `self._decoder.get_batch.asnumpy`, `self._decoder.get_batch` to complete its work. State updates are written into `batch`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `VideoDecoderWrapper`）实现了 `get_frames_at`。 它主要通过调用 `self._decoder.get_frames_at`, `batch.data.numpy`, `self._decoder.get_batch.asnumpy`, `self._decoder.get_batch` 来完成任务。 状态更新主要写入 `batch`。 实现中使用了条件分支。

### Lines 104-113: Method `VideoDecoderWrapper.get_frames_as_tensor` / 方法 `VideoDecoderWrapper.get_frames_as_tensor`
```python
    def get_frames_as_tensor(self, indices: list):
        """Return frames at given indices as a torch tensor (NHWC, uint8, pinned memory)."""
        import torch

        if _BACKEND == "torchcodec":
            batch = self._decoder.get_frames_at(indices)
            return batch.data.pin_memory()
        else:
            arr = self._decoder.get_batch(indices).asnumpy()
            return torch.from_numpy(arr).pin_memory()
```
**EN:** This method implements `get_frames_as_tensor` on `VideoDecoderWrapper`. It primarily calls `self._decoder.get_frames_at`, `batch.data.pin_memory`, `self._decoder.get_batch.asnumpy`, `torch.from_numpy.pin_memory`, `self._decoder.get_batch`, `torch.from_numpy` to complete its work. State updates are written into `batch`, `arr`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `VideoDecoderWrapper`）实现了 `get_frames_as_tensor`。 它主要通过调用 `self._decoder.get_frames_at`, `batch.data.pin_memory`, `self._decoder.get_batch.asnumpy`, `torch.from_numpy.pin_memory`, `self._decoder.get_batch`, `torch.from_numpy` 来完成任务。 状态更新主要写入 `batch`, `arr`。 实现中使用了条件分支。

### Lines 115-127: Method `VideoDecoderWrapper.source_bytes` / 方法 `VideoDecoderWrapper.source_bytes`
```python
    @property
    def source_bytes(self) -> bytes | None:
        """Return raw video bytes if available (needed for audio extraction)."""
        if self._source_bytes is not None:
            return self._source_bytes
        path = self._tmp_path or self._source_path
        if path is not None:
            import os

            if os.path.isfile(path):
                with open(path, "rb") as f:
                    return f.read()
        return None
```
**EN:** This method implements `source_bytes` on `VideoDecoderWrapper`. It primarily calls `os.path.isfile`, `open`, `f.read` to complete its work. State updates are written into `path`. The implementation relies on conditional branches, context-managed resources.
**CN:** 该方法（属于 `VideoDecoderWrapper`）实现了 `source_bytes`。 它主要通过调用 `os.path.isfile`, `open`, `f.read` 来完成任务。 状态更新主要写入 `path`。 实现中使用了条件分支、上下文管理资源。

### Lines 129-136: Method `VideoDecoderWrapper.close` / 方法 `VideoDecoderWrapper.close`
```python
    def close(self):
        """Explicitly clean up temporary files."""
        if self._tmp_path is not None:
            import os

            if os.path.exists(self._tmp_path):
                os.unlink(self._tmp_path)
            self._tmp_path = None
```
**EN:** This method implements `close` on `VideoDecoderWrapper`. It primarily calls `os.path.exists`, `os.unlink` to complete its work. State updates are written into `self._tmp_path`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `VideoDecoderWrapper`）实现了 `close`。 它主要通过调用 `os.path.exists`, `os.unlink` 来完成任务。 状态更新主要写入 `self._tmp_path`。 实现中使用了条件分支。

### Lines 138-139: Method `VideoDecoderWrapper.__del__` / 方法 `VideoDecoderWrapper.__del__`
```python
    def __del__(self):
        self.close()
```
**EN:** This method implements `__del__` on `VideoDecoderWrapper`. It primarily calls `self.close` to complete its work.
**CN:** 该方法（属于 `VideoDecoderWrapper`）实现了 `__del__`。 它主要通过调用 `self.close` 来完成任务。

### Lines 141-142: Method `VideoDecoderWrapper.__enter__` / 方法 `VideoDecoderWrapper.__enter__`
```python
    def __enter__(self):
        return self
```
**EN:** This method implements `__enter__` on `VideoDecoderWrapper`.
**CN:** 该方法（属于 `VideoDecoderWrapper`）实现了 `__enter__`。

### Lines 144-145: Method `VideoDecoderWrapper.__exit__` / 方法 `VideoDecoderWrapper.__exit__`
```python
    def __exit__(self, *args):
        self.close()
```
**EN:** This method implements `__exit__` on `VideoDecoderWrapper`. It primarily calls `self.close` to complete its work.
**CN:** 该方法（属于 `VideoDecoderWrapper`）实现了 `__exit__`。 它主要通过调用 `self.close` 来完成任务。

## Key Concepts / 关键概念
- **Classes / 类**: `VideoDecoderWrapper`
- **Functions / 函数**: `_try_cuda_backend`, `__init__`, `__len__`, `__getitem__`, `avg_fps`, `get_frames_at`, `get_frames_as_tensor`, `source_bytes`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: None / 无
- **External / 外部依赖**: `numpy`, `torchcodec.decoders`, `torch`, `decord`
- **Standard library / 标准库**: `logging`, `os`, `tempfile`
