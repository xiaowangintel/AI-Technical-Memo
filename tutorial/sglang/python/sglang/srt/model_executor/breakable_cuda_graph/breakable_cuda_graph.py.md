# breakable_cuda_graph.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/model_executor/breakable_cuda_graph/breakable_cuda_graph.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the model execution and graph runners part of the SRT runtime and implements logic centered on `breakable_cuda_graph`. The module docstring frames it as: "Breakable CUDA Graph: capture a region as a sequence of ``torch.cuda.CUDAGraph`` segments separated by eager break points." / 该模块属于 SRT 运行时的模型执行与图运行器部分，主要实现围绕 `breakable_cuda_graph` 的逻辑。 它对外提供的主要入口包括 `_check_cuda_bindings`, `get_current_stream`, `_capture_status`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-48: Module imports, constants, and setup
```python
# Copyright 2023-2026 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
"""Breakable CUDA Graph: capture a region as a sequence of
``torch.cuda.CUDAGraph`` segments separated by eager break points.

Each segment is a real ``torch.cuda.CUDAGraph``. Its destructor calls
``releasePool`` on the shared mempool, so the pool's ``use_count`` tracks how
many segments are alive; the pool stays pinned as long as any segment graph
is alive. This lets ``weak_ref_tensor`` views of intermediate pool-allocated
tensors remain valid across replays — we don't need Python-managed bridge
buffers to keep break-point tensors at stable addresses.
"""

import logging
import threading
from contextvars import ContextVar
from typing import Any, Callable

import torch

try:
    from cuda.bindings import runtime as rt
except ImportError:
    rt = None

from sglang.srt.model_executor.breakable_cuda_graph.cuda_utils import checkCudaErrors

logger = logging.getLogger(__name__)

__all__ = [
    "eager_on_graph",
    "BreakableCUDAGraph",
    "BreakableCUDAGraphCapture",
    "break_graph",
]


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; manages graph capture or replay logic.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；管理图捕获或回放逻辑。

### Lines 49-54: Function _check_cuda_bindings
```python
def _check_cuda_bindings():
    if rt is None:
        raise ImportError(
            "Breakable CUDA graph requires the 'cuda-python' package. "
            "Install it with: pip install cuda-python"
        )
```
**EN:** This callable implements `_check_cuda_bindings` and mainly checks preconditions and compatibility. In this range it performs defensive checks on invalid state; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `_check_cuda_bindings`，主要用于检查前置条件与兼容性。 在这一范围内，它会对非法状态执行防御性检查；管理图捕获或回放逻辑。

### Lines 55-70: Module-level constants and helpers
```python


# Active BreakableCUDAGraphCapture context for the currently-capturing thread.
# eager_on_graph's wrapper uses this to split the current torch.cuda.CUDAGraph
# at break points.
_current_capture_var: ContextVar["BreakableCUDAGraphCapture | None"] = ContextVar(
    "current_capture", default=None
)
_current_stream_var: ContextVar[torch.cuda.Stream | None] = ContextVar(
    "current_stream", default=None
)
_forked_streams_var: ContextVar[set[torch.cuda.Stream] | None] = ContextVar(
    "forked_streams", default=None
)


```
**EN:** This range organizes module-level state and shared setup. In this range it manages graph capture or replay logic.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 71-75: Function get_current_stream
```python
def get_current_stream(device: torch.device | None = None) -> torch.cuda.Stream:
    stream = _current_stream_var.get()
    if stream is None:
        return torch.cuda.current_stream(device)
    return stream
```
**EN:** This callable implements `get_current_stream`. It takes `device` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `get_current_stream`。它接收 `device`，主要用于获取某个值或派生视图。

### Lines 78-81: Function _capture_status
```python
def _capture_status(stream_ptr: int) -> "rt.cudaStreamCaptureStatus":
    _check_cuda_bindings()
    status, *_ = checkCudaErrors(rt.cudaStreamGetCaptureInfo(stream_ptr))
    return status
```
**EN:** This callable implements `_capture_status`. It takes `stream_ptr` and mainly implements capture status. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `_capture_status`。它接收 `stream_ptr`，主要用于实现 capture status 相关逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 84-89: Function _is_capturing
```python
def _is_capturing(stream_ptr: int) -> bool:
    _check_cuda_bindings()
    return (
        _capture_status(stream_ptr)
        == rt.cudaStreamCaptureStatus.cudaStreamCaptureStatusActive
    )
```
**EN:** This callable implements `_is_capturing`. It takes `stream_ptr` and mainly implements is capturing. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `_is_capturing`。它接收 `stream_ptr`，主要用于实现 is capturing 相关逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 90-100: Module-level constants and helpers
```python


# Hook torch.cuda.Stream.wait_stream to track side-stream forks/joins that happen
# during breakable capture. We need this because capture_end() on a torch
# CUDAGraph fails if there are still side streams participating in the capture
# — so before ending each segment we auto-join any forked-but-not-rejoined streams.
_original_wait_stream: Callable | None = None
_hook_lock = threading.Lock()
_hook_refcount = 0


```
**EN:** This range organizes module-level state and shared setup. In this range it manages graph capture or replay logic.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 101-128: Function _hooked_wait_stream
```python
def _hooked_wait_stream(self: torch.cuda.Stream, other: torch.cuda.Stream):
    assert _original_wait_stream is not None
    forked = _forked_streams_var.get()
    if forked is None:
        _original_wait_stream(self, other)
        return
    capturing = _current_stream_var.get()
    if capturing is None:
        _original_wait_stream(self, other)
        return

    cap_ptr = capturing.cuda_stream
    is_self_cap = self is capturing or self.cuda_stream == cap_ptr
    is_other_cap = other is capturing or other.cuda_stream == cap_ptr

    if is_self_cap and not is_other_cap:
        if (
            _capture_status(other.cuda_stream)
            != rt.cudaStreamCaptureStatus.cudaStreamCaptureStatusActive
        ):
            return
        _original_wait_stream(self, other)
        forked.discard(other)
    elif is_other_cap and not is_self_cap:
        _original_wait_stream(self, other)
        forked.add(self)
    else:
        _original_wait_stream(self, other)
```
**EN:** This callable implements `_hooked_wait_stream`. It takes `other` and mainly implements hooked wait stream. In this range it performs defensive checks on invalid state; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `_hooked_wait_stream`。它接收 `other`，主要用于实现 hooked wait stream 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；管理图捕获或回放逻辑。

### Lines 131-137: Function _install_wait_stream_hook
```python
def _install_wait_stream_hook():
    global _original_wait_stream, _hook_refcount
    with _hook_lock:
        if _hook_refcount == 0:
            _original_wait_stream = torch.cuda.Stream.wait_stream
            torch.cuda.Stream.wait_stream = _hooked_wait_stream  # type: ignore[assignment]
        _hook_refcount += 1
```
**EN:** This callable implements `_install_wait_stream_hook` and mainly implements install wait stream hook.
**CN:** 这一可调用对象实现了 `_install_wait_stream_hook`，主要用于实现 install wait stream hook 相关逻辑。

### Lines 140-147: Function _uninstall_wait_stream_hook
```python
def _uninstall_wait_stream_hook():
    global _original_wait_stream, _hook_refcount
    with _hook_lock:
        _hook_refcount -= 1
        if _hook_refcount == 0:
            assert _original_wait_stream is not None, "wait_stream hook not installed"
            torch.cuda.Stream.wait_stream = _original_wait_stream  # type: ignore[assignment]
            _original_wait_stream = None
```
**EN:** This callable implements `_uninstall_wait_stream_hook` and mainly implements uninstall wait stream hook. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `_uninstall_wait_stream_hook`，主要用于实现 uninstall wait stream hook 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 150-163: Function _weak_ref_if_tensor
```python
def _weak_ref_if_tensor(x):
    """Return a weak-ref tensor view (shared storage, no refcount) for tensors;
    pass-through for non-tensors. Weak-ref'ing captured args lets the shared
    mempool reclaim per-layer intermediates between segments — storage stays
    alive for each segment CUDAGraph's lifetime via its pool use_count.

    ``weak_ref_tensors`` is imported lazily: the module hard-raises on
    non-CUDA/NPU platforms, and we only reach this code during an active
    BCG capture (which can't happen on CPU-only runners anyway)."""
    if torch.is_tensor(x):
        from sglang.srt.compilation.weak_ref_tensor import weak_ref_tensors

        return weak_ref_tensors(x)
    return x
```
**EN:** This callable implements `_weak_ref_if_tensor`. It takes `x` and mainly implements weak ref if tensor. The docstring states: "Return a weak-ref tensor view (shared storage, no refcount) for tensors; pass-through for non-tensors." In this range it sets up imports and shared symbols; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `_weak_ref_if_tensor`。它接收 `x`，主要用于实现 weak ref if tensor 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；管理图捕获或回放逻辑。

### Lines 166-195: Function _copy_output
```python
def _copy_output(dst: Any, src: Any) -> Any:
    """Copy src output into dst in-place where possible.

    Handles plain tensors, dataclass/object with tensor attributes,
    and dicts of tensors. Returns dst if in-place copy succeeded,
    otherwise returns src.
    """
    if torch.is_tensor(dst) and torch.is_tensor(src):
        dst.copy_(src)
        return dst

    if hasattr(dst, "__dict__") and hasattr(src, "__dict__"):
        for key, src_val in src.__dict__.items():
            dst_val = getattr(dst, key, None)
            if torch.is_tensor(dst_val) and torch.is_tensor(src_val):
                dst_val.copy_(src_val)
            else:
                setattr(dst, key, src_val)
        return dst

    if isinstance(dst, dict) and isinstance(src, dict):
        for key, src_val in src.items():
            dst_val = dst.get(key)
            if torch.is_tensor(dst_val) and torch.is_tensor(src_val):
                dst_val.copy_(src_val)
            else:
                dst[key] = src_val
        return dst

    return src
```
**EN:** This callable implements `_copy_output`. It takes `dst`, `src` and mainly implements copy output. The docstring states: "Copy src output into dst in-place where possible."
**CN:** 这一可调用对象实现了 `_copy_output`。它接收 `dst`, `src`，主要用于实现 copy output 相关逻辑。

### Lines 198-237: Function eager_on_graph
```python
def eager_on_graph(enable: bool):
    def decorator(inner: Callable):
        if not enable:
            return inner

        def wrapper(*args, **kwargs):
            capture = _current_capture_var.get()
            if capture is None:
                return inner(*args, **kwargs)

            logger.debug("Break graph due to function: %s", inner.__name__)

            # End the segment that captured up to this break point.
            capture._end_current_segment()

            # Run the eager function once so it allocates its outputs and
            # writes real data into them.
            output = inner(*args, **kwargs)

            # Weak-ref the closure state. Storage lives with the segment
            # CUDAGraphs' mempool pin; Python refs don't need to prevent
            # pool reuse across layers.
            captured_inner = inner
            captured_args = tuple(_weak_ref_if_tensor(a) for a in args)
            captured_kwargs = {k: _weak_ref_if_tensor(v) for k, v in kwargs.items()}
            captured_output = _weak_ref_if_tensor(output)

            def replay_fn():
                new_out = captured_inner(*captured_args, **captured_kwargs)
                return _copy_output(captured_output, new_out)

            capture.cuda_graph._break_fns.append(replay_fn)

            # Start a fresh CUDAGraph segment for the remainder of the forward.
            capture._begin_new_segment()
            return output

        return wrapper

    return decorator
```
**EN:** This callable implements `eager_on_graph`. It takes `enable` and mainly implements eager on graph. In this range it emits logs for diagnostics; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `eager_on_graph`。它接收 `enable`，主要用于实现 eager on graph 相关逻辑。 在这一范围内，它会输出日志以便诊断；管理图捕获或回放逻辑。

### Lines 240-243: Class BreakableCUDAGraph
```python
class BreakableCUDAGraph:
    """Container holding one ``torch.cuda.CUDAGraph`` per segment plus an
    eager break function between consecutive segments."""

```
**EN:** This range introduces `BreakableCUDAGraph` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Container holding one ``torch.cuda.CUDAGraph`` per segment plus an eager break function between consecutive segments." In this range it manages graph capture or replay logic.
**CN:** 这一段引入 `BreakableCUDAGraph`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 244-246: Method BreakableCUDAGraph.__init__
```python
    def __init__(self) -> None:
        self._segments: list[torch.cuda.CUDAGraph] = []
        self._break_fns: list[Callable[[], Any]] = []
```
**EN:** This callable implements `BreakableCUDAGraph.__init__` and mainly initializes instance state and defaults. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `BreakableCUDAGraph.__init__`，主要用于初始化实例状态与默认值。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 248-257: Method BreakableCUDAGraph.replay
```python
    def replay(self) -> None:
        stream = torch.cuda.current_stream()
        token = _current_stream_var.set(stream)
        try:
            for i, seg in enumerate(self._segments):
                seg.replay()
                if i < len(self._break_fns):
                    self._break_fns[i]()
        finally:
            _current_stream_var.reset(token)
```
**EN:** This callable implements `BreakableCUDAGraph.replay` and mainly implements replay.
**CN:** 这一可调用对象实现了 `BreakableCUDAGraph.replay`，主要用于实现 replay 相关逻辑。

### Lines 260-270: Class BreakableCUDAGraphCapture
```python
class BreakableCUDAGraphCapture:
    """Context manager that captures the enclosed code as one or more
    ``torch.cuda.CUDAGraph`` segments separated by eager break points.

    Each segment shares the supplied ``pool`` (``MempoolId_t`` tuple) so
    pool-allocated intermediates can be reused across segments. While any
    segment is alive, its ``beginAllocateToPool`` call keeps the mempool's
    ``use_count`` > 0, which makes ``weak_ref_tensor`` of segment-allocated
    tensors safe across subsequent replays.
    """

```
**EN:** This range introduces `BreakableCUDAGraphCapture` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Context manager that captures the enclosed code as one or more ``torch.cuda.CUDAGraph`` segments separated by eager break points." In this range it manages graph capture or replay logic.
**CN:** 这一段引入 `BreakableCUDAGraphCapture`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 271-288: Method BreakableCUDAGraphCapture.__init__
```python
    def __init__(
        self,
        cuda_graph: BreakableCUDAGraph,
        pool=None,
        stream: torch.cuda.Stream | None = None,
        capture_error_mode: str = "global",
    ):
        assert isinstance(
            cuda_graph, BreakableCUDAGraph
        ), "cuda_graph must be a BreakableCUDAGraph"
        self.cuda_graph = cuda_graph
        self._pool = pool if pool is not None else (0, 0)
        self._stream = stream
        self._capture_error_mode = capture_error_mode
        self._stream_ctx = None
        self._capture_token = None
        self._stream_token = None
        self._forked_token = None
```
**EN:** This callable implements `BreakableCUDAGraphCapture.__init__`. It takes `cuda_graph`, `pool`, `stream`, `capture_error_mode` and mainly initializes instance state and defaults. In this range it performs defensive checks on invalid state; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `BreakableCUDAGraphCapture.__init__`。它接收 `cuda_graph`, `pool`, `stream`, `capture_error_mode`，主要用于初始化实例状态与默认值。 在这一范围内，它会对非法状态执行防御性检查；管理图捕获或回放逻辑。

### Lines 290-301: Method BreakableCUDAGraphCapture.__enter__
```python
    def __enter__(self):
        _install_wait_stream_hook()
        if self._stream is not None:
            self._stream_ctx = torch.cuda.stream(self._stream)
            self._stream_ctx.__enter__()
        self._capture_token = _current_capture_var.set(self)
        self._stream_token = _current_stream_var.set(
            self._stream or torch.cuda.current_stream()
        )
        self._forked_token = _forked_streams_var.set(set())
        self._begin_new_segment()
        return self
```
**EN:** This callable implements `BreakableCUDAGraphCapture.__enter__` and mainly implements enter. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `BreakableCUDAGraphCapture.__enter__`，主要用于实现 enter 相关逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 303-314: Method BreakableCUDAGraphCapture.__exit__
```python
    def __exit__(self, *args: object):
        try:
            self._end_current_segment()
        finally:
            _forked_streams_var.reset(self._forked_token)
            _current_stream_var.reset(self._stream_token)
            _current_capture_var.reset(self._capture_token)
            if self._stream_ctx is not None:
                self._stream_ctx.__exit__(*args)
                self._stream_ctx = None
            _uninstall_wait_stream_hook()
        return False
```
**EN:** This callable implements `BreakableCUDAGraphCapture.__exit__`. It takes `*args` and mainly implements exit. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `BreakableCUDAGraphCapture.__exit__`。它接收 `*args`，主要用于实现 exit 相关逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 316-321: Method BreakableCUDAGraphCapture._begin_new_segment
```python
    def _begin_new_segment(self) -> None:
        graph = torch.cuda.CUDAGraph()
        graph.capture_begin(
            pool=self._pool, capture_error_mode=self._capture_error_mode
        )
        self.cuda_graph._segments.append(graph)
```
**EN:** This callable implements `BreakableCUDAGraphCapture._begin_new_segment` and mainly implements begin new segment. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `BreakableCUDAGraphCapture._begin_new_segment`，主要用于实现 begin new segment 相关逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 323-333: Method BreakableCUDAGraphCapture._end_current_segment
```python
    def _end_current_segment(self) -> None:
        # Auto-join any side streams forked during this segment but not joined.
        main_stream = get_current_stream()
        forked = _forked_streams_var.get()
        if forked:
            assert _original_wait_stream is not None
            for side in list(forked):
                if _is_capturing(side.cuda_stream):
                    _original_wait_stream(main_stream, side)
            forked.clear()
        self.cuda_graph._segments[-1].capture_end()
```
**EN:** This callable implements `BreakableCUDAGraphCapture._end_current_segment` and mainly implements end current segment. In this range it performs defensive checks on invalid state; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `BreakableCUDAGraphCapture._end_current_segment`，主要用于实现 end current segment 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；管理图捕获或回放逻辑。

### Lines 336-340: Function break_graph
```python
@eager_on_graph(True)
def break_graph() -> None:
    """Insert a graph break. The @eager_on_graph decorator does the actual
    segment split; this function body intentionally does nothing."""
    pass
```
**EN:** This callable implements `break_graph` and mainly implements break graph. The docstring states: "Insert a graph break." In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `break_graph`，主要用于实现 break graph 相关逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

## Key Concepts / 关键概念
- `_check_cuda_bindings`: checks preconditions and compatibility / 检查前置条件与兼容性
- `get_current_stream`: retrieves a value or derived view / 获取某个值或派生视图
- `_capture_status`: implements capture status / 实现 capture status 相关逻辑
- `_is_capturing`: implements is capturing / 实现 is capturing 相关逻辑
- `_hooked_wait_stream`: implements hooked wait stream / 实现 hooked wait stream 相关逻辑
- `_install_wait_stream_hook`: implements install wait stream hook / 实现 install wait stream hook 相关逻辑
- `_uninstall_wait_stream_hook`: implements uninstall wait stream hook / 实现 uninstall wait stream hook 相关逻辑
- `_weak_ref_if_tensor`: implements weak ref if tensor / 实现 weak ref if tensor 相关逻辑
- `_copy_output`: implements copy output / 实现 copy output 相关逻辑
- `eager_on_graph`: implements eager on graph / 实现 eager on graph 相关逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `threading`, `contextvars`, `typing`
- **Third-party / 第三方**: `torch`, `cuda.bindings`
- **Internal modules / 内部模块**: `sglang.srt.model_executor.breakable_cuda_graph.cuda_utils`, `sglang.srt.compilation.weak_ref_tensor`
