# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/entrypoints/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the service entrypoint layer. It centers on `SetLoraReq`, `MergeLoraWeightsReq`, and `UnmergeLoraWeightsReq`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: DiffGenerator module for sglang-diffusion. This module provides a consolidated interface for generating videos using diffusion models. / 该文件属于服务入口层。它围绕 `SetLoraReq`、`MergeLoraWeightsReq` 和 `UnmergeLoraWeightsReq` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 4-21: module setup and imports / 模块初始化与导入
```python
"""
DiffGenerator module for sglang-diffusion.

This module provides a consolidated interface for generating videos using
diffusion models.
"""

import os
import shutil
import subprocess
import tempfile
from copy import copy
from dataclasses import dataclass, field
from typing import Any, Callable, List, Optional, Sequence, Union

import imageio
import numpy as np
import torch
```
**EN:** This block establishes the module context and imports `os`, `shutil`, `subprocess`, `tempfile`, `copy`, and `dataclasses`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `os`、`shutil`、`subprocess`、`tempfile`、`copy` 和 `dataclasses`。这些依赖为后续实现提供所需符号。

### Lines 23-42: supporting statements / 辅助语句
```python
try:
    import scipy.io.wavfile as scipy_wavfile
except ImportError:  # pragma: no cover
    scipy_wavfile = None

try:
    import imageio_ffmpeg as _imageio_ffmpeg
except ImportError:  # pragma: no cover
    _imageio_ffmpeg = None

from sglang.multimodal_gen.configs.sample.sampling_params import (
    DataType,
    SamplingParams,
)
from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import Req
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.logging_utils import CYAN, RESET, init_logger
from sglang.srt.observability.trace import TraceReqContext

logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 46-46: `SetLoraReq` class overview / `SetLoraReq` 类概览
```python
class SetLoraReq:
```
**EN:** This block defines class `SetLoraReq`. It configures lora req.
**CN:** 该代码块定义了类 `SetLoraReq`。 它用于设置lora req。

### Lines 47-51: supporting statements / 辅助语句
```python
    lora_nickname: Union[str, List[str]]
    lora_path: Optional[Union[str, List[Optional[str]]]] = None
    target: Union[str, List[str]] = "all"
    strength: Union[float, List[float]] = 1.0
    merge_mode: Optional[str] = None
```
**EN:** This block gathers supporting statements inside `SetLoraReq`. It updates names such as `lora_nickname`, `lora_path`, `target`, `strength`, and `merge_mode`.
**CN:** 该代码块汇集了位于 `SetLoraReq` 内部的辅助语句。 它会更新 `lora_nickname`、`lora_path`、`target`、`strength` 和 `merge_mode` 等名称。

### Lines 55-55: `MergeLoraWeightsReq` class overview / `MergeLoraWeightsReq` 类概览
```python
class MergeLoraWeightsReq:
```
**EN:** This block defines class `MergeLoraWeightsReq`. It merges lora weights req.
**CN:** 该代码块定义了类 `MergeLoraWeightsReq`。 它用于合并lora weights req。

### Lines 56-57: supporting statements / 辅助语句
```python
    target: str = "all"
    strength: float = 1.0
```
**EN:** This block gathers supporting statements inside `MergeLoraWeightsReq`. It updates names such as `target`, and `strength`.
**CN:** 该代码块汇集了位于 `MergeLoraWeightsReq` 内部的辅助语句。 它会更新 `target` 和 `strength` 等名称。

### Lines 61-61: `UnmergeLoraWeightsReq` class overview / `UnmergeLoraWeightsReq` 类概览
```python
class UnmergeLoraWeightsReq:
```
**EN:** This block defines class `UnmergeLoraWeightsReq`. It encapsulates unmerge lora weights req behavior.
**CN:** 该代码块定义了类 `UnmergeLoraWeightsReq`。 它用于封装 unmerge lora weights req 相关行为。

### Lines 62-62: supporting statements / 辅助语句
```python
    target: str = "all"
```
**EN:** This block gathers supporting statements inside `UnmergeLoraWeightsReq`. It updates names such as `target`.
**CN:** 该代码块汇集了位于 `UnmergeLoraWeightsReq` 内部的辅助语句。 它会更新 `target` 等名称。

### Lines 66-66: `ListLorasReq` class overview / `ListLorasReq` 类概览
```python
class ListLorasReq:
```
**EN:** This block defines class `ListLorasReq`. It encapsulates list loras req behavior.
**CN:** 该代码块定义了类 `ListLorasReq`。 它用于封装 list loras req 相关行为。

### Lines 67-67: supporting statements / 辅助语句
```python
    pass
```
**EN:** This block gathers supporting statements inside `ListLorasReq`.
**CN:** 该代码块汇集了位于 `ListLorasReq` 内部的辅助语句。

### Lines 71-71: `ShutdownReq` class overview / `ShutdownReq` 类概览
```python
class ShutdownReq:
```
**EN:** This block defines class `ShutdownReq`. It encapsulates shutdown req behavior.
**CN:** 该代码块定义了类 `ShutdownReq`。 它用于封装 shutdown req 相关行为。

### Lines 72-72: supporting statements / 辅助语句
```python
    pass
```
**EN:** This block gathers supporting statements inside `ShutdownReq`.
**CN:** 该代码块汇集了位于 `ShutdownReq` 内部的辅助语句。

### Lines 76-78: `GetDisaggStatsReq` class overview / `GetDisaggStatsReq` 类概览
```python
class GetDisaggStatsReq:
    """Request to get disagg pipeline metrics from the scheduler."""
```
**EN:** This block defines class `GetDisaggStatsReq`. Request to get disagg pipeline metrics from the scheduler.
**CN:** 该代码块定义了类 `GetDisaggStatsReq`。 它用于获取disagg stats req。

### Lines 79-79: supporting statements / 辅助语句
```python
    pass
```
**EN:** This block gathers supporting statements inside `GetDisaggStatsReq`.
**CN:** 该代码块汇集了位于 `GetDisaggStatsReq` 内部的辅助语句。

### Lines 82-104: `format_lora_message` implementation / `format_lora_message` 实现
```python
def format_lora_message(
    lora_nickname: Union[str, List[str]],
    target: Union[str, List[str]],
    strength: Union[float, List[float]],
) -> tuple[str, str, str]:
    """Format success message for single or multiple LoRAs."""
    if isinstance(lora_nickname, list):
        nickname_str = ", ".join(lora_nickname)
        target_str = ", ".join(target) if isinstance(target, list) else target
        strength_str = (
            ", ".join(f"{s:.2f}" for s in strength)
            if isinstance(strength, list)
            else f"{strength:.2f}"
        )
    else:
        nickname_str = lora_nickname
        target_str = target if isinstance(target, str) else ", ".join(target)
        strength_str = (
            f"{strength:.2f}"
            if isinstance(strength, (int, float))
            else ", ".join(f"{s:.2f}" for s in strength)
        )
    return nickname_str, target_str, strength_str
```
**EN:** This block defines function `format_lora_message`. Format success message for single or multiple LoRAs. Key calls include `isinstance`, and `join`. The implementation branches on conditions. Parameters such as `lora_nickname`, `target`, and `strength` drive the behavior in this section.
**CN:** 该代码块定义了函数 `format_lora_message`。 它用于处理 format lora message 相关逻辑。 关键调用包括 `isinstance` 和 `join`。 实现中包含条件分支。 本段逻辑主要由 `lora_nickname`、`target` 和 `strength` 等参数驱动。

### Lines 108-110: `GenerationResult` class overview / `GenerationResult` 类概览
```python
class GenerationResult:
    """Result of a single generation request from DiffGenerator."""
```
**EN:** This block defines class `GenerationResult`. Result of a single generation request from DiffGenerator.
**CN:** 该代码块定义了类 `GenerationResult`。 它用于封装 generation result 相关行为。

### Lines 111-124: supporting statements / 辅助语句
```python
    samples: Any = None
    frames: Any = None
    audio: Any = None
    prompt: str | None = None
    size: tuple | None = None  # (height, width, num_frames)
    generation_time: float = 0.0
    peak_memory_mb: float = 0.0
    metrics: dict = field(default_factory=dict)
    trajectory_latents: Any = None
    trajectory_timesteps: Any = None
    rollout_trajectory_data: Any = None
    trajectory_decoded: Any = None
    prompt_index: int = 0
    output_file_path: str | None = None
```
**EN:** This block gathers supporting statements inside `GenerationResult`. It updates names such as `samples`, `frames`, `audio`, `prompt`, `size`, and `generation_time`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `GenerationResult` 内部的辅助语句。 它会更新 `samples`、`frames`、`audio`、`prompt`、`size` 和 `generation_time` 等名称。 代码会与 `field` 协同工作。

### Lines 127-157: `normalize_output_seeds` implementation / `normalize_output_seeds` 实现
```python
def normalize_output_seeds(
    seed: int | list[int],
    *,
    num_outputs_per_prompt: int,
    num_prompts: int = 1,
    prompt_index: int = 0,
) -> list[int]:
    """
    return a list of seed with size equal to `num_outputs_per_prompt`
    """
    if num_outputs_per_prompt <= 0:
        raise ValueError(
            f"num_outputs_per_prompt must be positive, got {num_outputs_per_prompt}"
        )

    if isinstance(seed, list):
        seeds = [int(item) for item in seed]
        total_outputs = num_outputs_per_prompt * num_prompts
        if len(seeds) == num_outputs_per_prompt:
            return seeds
        if len(seeds) == total_outputs:
            start = prompt_index * num_outputs_per_prompt
            return seeds[start : start + num_outputs_per_prompt]
        raise ValueError(
            "seed list length must match num_outputs_per_prompt "
            f"({num_outputs_per_prompt}) or total outputs ({total_outputs}), "
            f"got {len(seeds)}"
        )

    base_seed = int(seed)
    return [base_seed + i for i in range(num_outputs_per_prompt)]
```
**EN:** This block defines function `normalize_output_seeds`. return a list of seed with size equal to `num_outputs_per_prompt` Key calls include `isinstance`, `int`, `ValueError`, `len`, and `range`. The implementation branches on conditions. Parameters such as `seed` drive the behavior in this section.
**CN:** 该代码块定义了函数 `normalize_output_seeds`。 它用于处理 normalize output seeds 相关逻辑。 关键调用包括 `isinstance`、`int`、`ValueError`、`len` 和 `range`。 实现中包含条件分支。 本段逻辑主要由 `seed` 等参数驱动。

### Lines 160-162: `_with_output_index_suffix` implementation / `_with_output_index_suffix` 实现
```python
def _with_output_index_suffix(output_file_name: str, output_index: int) -> str:
    base, ext = os.path.splitext(output_file_name)
    return f"{base}_{output_index}{ext}"
```
**EN:** This block defines function `_with_output_index_suffix`. It handles with output index suffix logic. Key calls include `os.path.splitext`. Parameters such as `output_file_name`, and `output_index` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_with_output_index_suffix`。 它用于处理 with output index suffix 相关逻辑。 关键调用包括 `os.path.splitext`。 本段逻辑主要由 `output_file_name` 和 `output_index` 等参数驱动。

### Lines 165-176: `_copy_trace_ctx_for_output` implementation / `_copy_trace_ctx_for_output` 实现
```python
def _copy_trace_ctx_for_output(req: Req, request_id: str | None, output_index: int):
    trace_ctx = req.trace_ctx
    if output_index == 0 or not trace_ctx.tracing_enable:
        return trace_ctx

    output_trace_ctx = TraceReqContext(
        rid=request_id,
        module_name=trace_ctx.module_name,
        external_trace_header=trace_ctx.external_trace_header,
    )
    output_trace_ctx.trace_req_start()
    return output_trace_ctx
```
**EN:** This block defines function `_copy_trace_ctx_for_output`. It handles copy trace ctx for output logic. Key calls include `TraceReqContext`, and `output_trace_ctx.trace_req_start`. The implementation branches on conditions. Parameters such as `req`, `request_id`, and `output_index` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_copy_trace_ctx_for_output`。 它用于处理 copy trace ctx for output 相关逻辑。 关键调用包括 `TraceReqContext` 和 `output_trace_ctx.trace_req_start`。 实现中包含条件分支。 本段逻辑主要由 `req`、`request_id` 和 `output_index` 等参数驱动。

### Lines 179-190: `_copy_req_for_output` implementation / `_copy_req_for_output` 实现
```python
def _copy_req_for_output(
    req: Req,
    *,
    request_id: str | None,
    output_index: int,
) -> Req:
    """Create a lightweight per-output ``Req`` without deep-copying tensors."""
    output_req = copy(req)
    output_req.sampling_params = copy(req.sampling_params)
    output_req.extra = dict(req.extra)
    output_req.trace_ctx = _copy_trace_ctx_for_output(req, request_id, output_index)
    return output_req
```
**EN:** This block defines function `_copy_req_for_output`. Create a lightweight per-output ``Req`` without deep-copying tensors. Key calls include `copy`, `dict`, and `_copy_trace_ctx_for_output`. Parameters such as `req` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_copy_req_for_output`。 它用于处理 copy req for output 相关逻辑。 关键调用包括 `copy`、`dict` 和 `_copy_trace_ctx_for_output`。 本段逻辑主要由 `req` 等参数驱动。

### Lines 193-242: `expand_request_outputs` implementation / `expand_request_outputs` 实现
```python
def expand_request_outputs(
    req: Req,
    *,
    num_prompts: int = 1,
    prompt_index: int = 0,
) -> list[Req]:
    """
    Expand a req to a list with size equal to `num_prompts`
    """
    num_outputs = int(req.num_outputs_per_prompt)
    # each req must has different seed
    seeds = normalize_output_seeds(
        req.seed,
        num_outputs_per_prompt=num_outputs,
        num_prompts=num_prompts,
        prompt_index=prompt_index,
    )

    if num_outputs == 1:
        req.seed = seeds[0]
        req.seeds = None
        req.generator = None
        return [req]

    expanded: list[Req] = []
    for output_index, seed in enumerate(seeds):
        output_request_id = (
            f"{req.request_id}:{output_index}" if req.request_id is not None else None
        )
        output_req = _copy_req_for_output(
            req, request_id=output_request_id, output_index=output_index
        )
        output_req.seed = seed
        output_req.num_outputs_per_prompt = 1
        output_req.seeds = None
        output_req.generator = None
        output_req.extra["parent_request_id"] = req.request_id
        output_req.extra["output_index"] = output_index

        if output_request_id is not None:
            output_req.request_id = output_request_id

        if req.output_file_name:
            output_req.output_file_name = _with_output_index_suffix(
                req.output_file_name, output_index
            )
        output_req.validate()
        expanded.append(output_req)

    return expanded
```
**EN:** This block defines function `expand_request_outputs`. Expand a req to a list with size equal to `num_prompts` Key calls include `int`, `normalize_output_seeds`, `enumerate`, `_copy_req_for_output`, and `output_req.validate`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `req` drive the behavior in this section.
**CN:** 该代码块定义了函数 `expand_request_outputs`。 它用于处理 expand request outputs 相关逻辑。 关键调用包括 `int`、`normalize_output_seeds`、`enumerate`、`_copy_req_for_output` 和 `output_req.validate`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `req` 等参数驱动。

### Lines 245-269: `_normalize_audio_to_numpy` implementation / `_normalize_audio_to_numpy` 实现
```python
def _normalize_audio_to_numpy(audio: Any) -> np.ndarray | None:
    """Convert audio (torch / numpy) into a float32 numpy array in [-1, 1], best-effort."""
    if audio is None:
        return None
    if isinstance(audio, torch.Tensor):
        audio_np = audio.detach().float().clamp(-1.0, 1.0).cpu().numpy()
    elif isinstance(audio, np.ndarray):
        audio_np = audio.astype(np.float32, copy=False)
        audio_np = np.clip(audio_np, -1.0, 1.0)
    else:
        return None

    # 1. Squeeze leading singleton dimensions (Batch, etc.)
    while audio_np.ndim > 1 and audio_np.shape[0] == 1:
        audio_np = audio_np.squeeze(0)

    # 2. Handle (C, L) -> (L, C)
    if audio_np.ndim == 2 and audio_np.shape[0] < audio_np.shape[1]:
        audio_np = audio_np.transpose(1, 0)

    # 3. Final safety check: if still 2D and channels (dim 1) is huge, something is wrong
    if audio_np.ndim == 2 and audio_np.shape[1] > 256 and audio_np.shape[0] == 1:
        audio_np = audio_np.flatten()

    return audio_np
```
**EN:** This block defines function `_normalize_audio_to_numpy`. Convert audio (torch / numpy) into a float32 numpy array in [-1, 1], best-effort. Key calls include `isinstance`, `audio.detach.float.clamp.cpu.numpy`, `audio_np.squeeze`, `audio_np.transpose`, and `audio_np.flatten`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `audio` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_normalize_audio_to_numpy`。 它用于处理 normalize audio to numpy 相关逻辑。 关键调用包括 `isinstance`、`audio.detach.float.clamp.cpu.numpy`、`audio_np.squeeze`、`audio_np.transpose` 和 `audio_np.flatten`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `audio` 等参数驱动。

### Lines 272-296: `_pick_audio_sample_rate` implementation / `_pick_audio_sample_rate` 实现
```python
def _pick_audio_sample_rate(
    *,
    audio_np: np.ndarray,
    audio_sample_rate: Optional[int],
    fps: int,
    num_frames: int,
) -> int:
    """Pick a plausible sample rate, falling back to inferring from video duration."""
    selected_sr = int(audio_sample_rate) if audio_sample_rate is not None else None
    if selected_sr is None or not (8000 <= selected_sr <= 192000):
        selected_sr = 24000
        try:
            duration_s = float(num_frames) / float(fps) if fps else 0.0
            if duration_s > 0:
                audio_len = (
                    int(audio_np.shape[0])
                    if audio_np.ndim == 2
                    else int(audio_np.shape[-1])
                )
                inferred_sr = int(round(float(audio_len) / duration_s))
                if 8000 <= inferred_sr <= 192000:
                    selected_sr = inferred_sr
        except Exception:
            pass
    return selected_sr
```
**EN:** This block defines function `_pick_audio_sample_rate`. Pick a plausible sample rate, falling back to inferring from video duration. Key calls include `int`, `float`, and `round`. The implementation branches on conditions, handles exceptional paths.
**CN:** 该代码块定义了函数 `_pick_audio_sample_rate`。 它用于处理 pick audio sample rate 相关逻辑。 关键调用包括 `int`、`float` 和 `round`。 实现中包含条件分支，处理异常路径。

### Lines 299-318: `_resolve_ffmpeg_exe` implementation / `_resolve_ffmpeg_exe` 实现
```python
def _resolve_ffmpeg_exe() -> str:
    ffmpeg_exe = "ffmpeg"
    ffmpeg_on_path = shutil.which("ffmpeg")
    if ffmpeg_on_path:
        ffmpeg_exe = ffmpeg_on_path
    try:
        if _imageio_ffmpeg is not None:
            ffmpeg_exe = _imageio_ffmpeg.get_ffmpeg_exe()
    except Exception:
        pass

    ffmpeg_ok = False
    if ffmpeg_exe:
        if os.path.isabs(ffmpeg_exe):
            ffmpeg_ok = os.path.exists(ffmpeg_exe)
        else:
            ffmpeg_ok = shutil.which(ffmpeg_exe) is not None
    if not ffmpeg_ok:
        raise RuntimeError("ffmpeg not found")
    return ffmpeg_exe
```
**EN:** This block defines function `_resolve_ffmpeg_exe`. It resolves ffmpeg exe. Key calls include `shutil.which`, `os.path.isabs`, `RuntimeError`, `_imageio_ffmpeg.get_ffmpeg_exe`, and `os.path.exists`. The implementation branches on conditions, handles exceptional paths.
**CN:** 该代码块定义了函数 `_resolve_ffmpeg_exe`。 它用于解析ffmpeg exe。 关键调用包括 `shutil.which`、`os.path.isabs`、`RuntimeError`、`_imageio_ffmpeg.get_ffmpeg_exe` 和 `os.path.exists`。 实现中包含条件分支，处理异常路径。

### Lines 321-369: `_mux_audio_np_into_mp4` implementation / `_mux_audio_np_into_mp4` 实现
```python
def _mux_audio_np_into_mp4(
    *,
    save_file_path: str,
    audio_np: np.ndarray,
    sample_rate: int,
    ffmpeg_exe: str,
) -> None:
    merged_path = save_file_path.rsplit(".", 1)[0] + ".tmp_mux.mp4"
    tmp_wav_path = None
    try:
        if scipy_wavfile is None:
            raise RuntimeError(
                "scipy is required to mux audio into mp4 (pip install scipy)"
            )
        with tempfile.NamedTemporaryFile(suffix=".wav", delete=False) as f:
            tmp_wav_path = f.name
        scipy_wavfile.write(tmp_wav_path, sample_rate, audio_np)
        subprocess.run(
            [
                ffmpeg_exe,
                "-y",
                "-i",
                save_file_path,
                "-i",
                tmp_wav_path,
                "-c:v",
                "copy",
                "-c:a",
                "aac",
                "-strict",
                "experimental",
                merged_path,
            ],
            check=True,
            stdout=subprocess.DEVNULL,
            stderr=subprocess.DEVNULL,
        )
        os.replace(merged_path, save_file_path)
    finally:
        if tmp_wav_path:
            try:
                os.remove(tmp_wav_path)
            except OSError:
                pass
        if os.path.exists(merged_path):
            try:
                os.remove(merged_path)
            except OSError:
                pass
```
**EN:** This block defines function `_mux_audio_np_into_mp4`. It handles mux audio np into mp4 logic. Key calls include `scipy_wavfile.write`, `subprocess.run`, `os.replace`, `os.path.exists`, and `save_file_path.rsplit`. The implementation branches on conditions, handles exceptional paths, uses context-managed resources.
**CN:** 该代码块定义了函数 `_mux_audio_np_into_mp4`。 它用于处理 mux audio np into mp4 相关逻辑。 关键调用包括 `scipy_wavfile.write`、`subprocess.run`、`os.replace`、`os.path.exists` 和 `save_file_path.rsplit`。 实现中包含条件分支，处理异常路径，使用上下文管理资源。

### Lines 372-406: `_maybe_mux_audio_into_mp4` implementation / `_maybe_mux_audio_into_mp4` 实现
```python
def _maybe_mux_audio_into_mp4(
    *,
    save_file_path: str,
    audio: Any,
    frames: list,
    fps: int,
    audio_sample_rate: Optional[int],
) -> None:
    """Best-effort mux audio into an already-written mp4 at save_file_path.

    Any failure should keep the silent video and only log a warning.
    """
    audio_np = _normalize_audio_to_numpy(audio)
    if audio_np is None:
        return
    selected_sr = _pick_audio_sample_rate(
        audio_np=audio_np,
        audio_sample_rate=audio_sample_rate,
        fps=fps,
        num_frames=len(frames),
    )

    try:
        ffmpeg_exe = _resolve_ffmpeg_exe()
        _mux_audio_np_into_mp4(
            save_file_path=save_file_path,
            audio_np=audio_np,
            sample_rate=selected_sr,
            ffmpeg_exe=ffmpeg_exe,
        )
    except Exception as e:
        logger.warning(
            "Failed to mux audio into mp4 (saved silent video): %s",
            str(e),
        )
```
**EN:** This block defines function `_maybe_mux_audio_into_mp4`. Best-effort mux audio into an already-written mp4 at save_file_path. Any failure should keep the silent video and only log a warning. Key calls include `_normalize_audio_to_numpy`, `_pick_audio_sample_rate`, `_resolve_ffmpeg_exe`, `_mux_audio_np_into_mp4`, and `len`. The implementation branches on conditions, handles exceptional paths.
**CN:** 该代码块定义了函数 `_maybe_mux_audio_into_mp4`。 它用于处理 maybe mux audio into mp4 相关逻辑。 关键调用包括 `_normalize_audio_to_numpy`、`_pick_audio_sample_rate`、`_resolve_ffmpeg_exe`、`_mux_audio_np_into_mp4` 和 `len`。 实现中包含条件分支，处理异常路径。

### Lines 409-447: `prepare_request` implementation / `prepare_request` 实现
```python
def prepare_request(
    server_args: ServerArgs,
    sampling_params: SamplingParams,
    external_trace_header: dict[str, str] | None = None,
) -> Req:
    """
    Create a Req object with sampling_params as a parameter.
    """
    req = Req(
        sampling_params=sampling_params,
        VSA_sparsity=server_args.attention_backend_config.VSA_sparsity,
    )
    sampling_params.apply_request_extra(req)
    diffusers_kwargs = getattr(sampling_params, "diffusers_kwargs", None)
    if diffusers_kwargs and "max_sequence_length" in diffusers_kwargs:
        req.max_sequence_length = diffusers_kwargs["max_sequence_length"]

    req.adjust_size(server_args)

    if not isinstance(req.prompt, str):
        raise TypeError(f"`prompt` must be a string, but got {type(req.prompt)}")

    if (req.width is not None and req.width <= 0) or (
        req.height is not None and req.height <= 0
    ):
        raise ValueError(
            f"Height and width must be positive, got height={req.height}, width={req.width}"
        )

    if server_args.enable_trace:
        trace_ctx = TraceReqContext(
            rid=sampling_params.request_id,
            module_name="diffusion",
            external_trace_header=external_trace_header,
        )
        trace_ctx.trace_req_start()
        req.trace_ctx = trace_ctx

    return req
```
**EN:** This block defines function `prepare_request`. Create a Req object with sampling_params as a parameter. Key calls include `Req`, `sampling_params.apply_request_extra`, `getattr`, `req.adjust_size`, and `isinstance`. The implementation branches on conditions. Parameters such as `server_args`, `sampling_params`, and `external_trace_header` drive the behavior in this section.
**CN:** 该代码块定义了函数 `prepare_request`。 它用于准备request。 关键调用包括 `Req`、`sampling_params.apply_request_extra`、`getattr`、`req.adjust_size` 和 `isinstance`。 实现中包含条件分支。 本段逻辑主要由 `server_args`、`sampling_params` 和 `external_trace_header` 等参数驱动。

### Lines 450-467: `attach_audio_to_video_sample` implementation / `attach_audio_to_video_sample` 实现
```python
def attach_audio_to_video_sample(
    sample: Any,
    audio: Any,
    output_idx: int,
) -> Any:
    """Attach per-sample audio for video outputs when available."""
    if audio is None:
        return sample
    if isinstance(audio, torch.Tensor) and audio.ndim >= 2:
        audio = audio[output_idx] if audio.shape[0] > output_idx else None
    elif isinstance(audio, np.ndarray) and audio.ndim >= 2:
        audio = audio[output_idx] if audio.shape[0] > output_idx else None

    if audio is not None and not (
        isinstance(sample, (tuple, list)) and len(sample) == 2
    ):
        return (sample, audio)
    return sample
```
**EN:** This block defines function `attach_audio_to_video_sample`. Attach per-sample audio for video outputs when available. Key calls include `isinstance`, and `len`. The implementation branches on conditions. Parameters such as `sample`, `audio`, and `output_idx` drive the behavior in this section.
**CN:** 该代码块定义了函数 `attach_audio_to_video_sample`。 它用于处理 attach audio to video sample 相关逻辑。 关键调用包括 `isinstance` 和 `len`。 实现中包含条件分支。 本段逻辑主要由 `sample`、`audio` 和 `output_idx` 等参数驱动。

### Lines 470-531: `save_outputs` implementation / `save_outputs` 实现
```python
def save_outputs(
    outputs: Sequence[Any],
    data_type: DataType,
    fps: int,
    save_output: bool,
    build_output_path: Callable[[int], str],
    *,
    audio: Any = None,
    audio_sample_rate: Optional[int] = None,
    samples_out: Optional[list[Any]] = None,
    audios_out: Optional[list[Any]] = None,
    frames_out: Optional[list[Any]] = None,
    output_compression: Optional[int] = None,
    enable_frame_interpolation: bool = False,
    frame_interpolation_exp: int = 1,
    frame_interpolation_scale: float = 1.0,
    frame_interpolation_model_path: Optional[str] = None,
    enable_upscaling: bool = False,
    upscaling_model_path: Optional[str] = None,
    upscaling_scale: int = 4,
) -> list[str]:
    """Save outputs to files and return the list of file paths."""
    output_paths: list[str] = []
    for idx, output in enumerate(outputs):
        save_file_path = build_output_path(idx)
        sample = output
        if data_type == DataType.VIDEO:
            sample = attach_audio_to_video_sample(sample, audio, idx)

        frames = post_process_sample(
            sample,
            data_type,
            fps,
            save_output,
            save_file_path,
            audio_sample_rate=audio_sample_rate,
            output_compression=output_compression,
            enable_frame_interpolation=enable_frame_interpolation,
            frame_interpolation_exp=frame_interpolation_exp,
            frame_interpolation_scale=frame_interpolation_scale,
            frame_interpolation_model_path=frame_interpolation_model_path,
            enable_upscaling=enable_upscaling,
            upscaling_model_path=upscaling_model_path,
            upscaling_scale=upscaling_scale,
        )

        if samples_out is not None:
            samples_out.append(sample)
        if audios_out is not None:
            if data_type == DataType.VIDEO:
                audio_item = audio
                if isinstance(audio, torch.Tensor) and audio.ndim >= 2:
                    audio_item = audio[idx] if audio.shape[0] > idx else None
                elif isinstance(audio, np.ndarray) and audio.ndim >= 2:
                    audio_item = audio[idx] if audio.shape[0] > idx else None
                audios_out.append(audio_item)
            else:
                audios_out.append(audio)
        if frames_out is not None:
            frames_out.append(frames)
        output_paths.append(save_file_path)
    return output_paths
```
**EN:** This block defines function `save_outputs`. Save outputs to files and return the list of file paths. Key calls include `enumerate`, `build_output_path`, `post_process_sample`, `output_paths.append`, and `attach_audio_to_video_sample`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `outputs`, `data_type`, `fps`, `save_output`, and `build_output_path` drive the behavior in this section.
**CN:** 该代码块定义了函数 `save_outputs`。 它用于保存outputs。 关键调用包括 `enumerate`、`build_output_path`、`post_process_sample`、`output_paths.append` 和 `attach_audio_to_video_sample`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `outputs`、`data_type`、`fps`、`save_output` 和 `build_output_path` 等参数驱动。

### Lines 534-655: `post_process_sample` implementation / `post_process_sample` 实现
```python
def post_process_sample(
    sample: Any,
    data_type: DataType,
    fps: int,
    save_output: bool = True,
    save_file_path: Optional[str] = None,
    audio_sample_rate: Optional[int] = None,
    output_compression: Optional[int] = None,
    enable_frame_interpolation: bool = False,
    frame_interpolation_exp: int = 1,
    frame_interpolation_scale: float = 1.0,
    frame_interpolation_model_path: Optional[str] = None,
    enable_upscaling: bool = False,
    upscaling_model_path: Optional[str] = None,
    upscaling_scale: int = 4,
):
    """
    Process sample output, optionally interpolate video frames, and save.
    """
    audio = None
    if isinstance(sample, (tuple, list)) and len(sample) == 2:
        sample, audio = sample

    # 1. Convert tensor / array to list of uint8 HWC frames
    frames = None
    if isinstance(sample, torch.Tensor):
        if sample.dim() == 3:
            sample = sample.unsqueeze(1)
        sample = (sample * 255).clamp(0, 255).to(torch.uint8)
        videos = sample.permute(1, 2, 3, 0).cpu().numpy()
        frames = list(videos)
    else:
        if not isinstance(sample, np.ndarray):
            raise TypeError(f"Unsupported sample type: {type(sample)}")

        arr = sample
        if arr.ndim == 3:
            if arr.shape[-1] in (1, 3, 4):
                arr = arr[None, ...]
            else:
                arr = arr[..., None]
        if arr.ndim != 4:
            raise ValueError(f"Unexpected numpy sample shape: {tuple(arr.shape)}")

        if arr.shape[-1] not in (1, 3, 4) and arr.shape[0] in (1, 3, 4):
            t = torch.from_numpy(arr)
            if t.dim() == 3:
                t = t.unsqueeze(1)
            t = (t * 255).clamp(0, 255).to(torch.uint8)
            videos = t.permute(1, 2, 3, 0).cpu().numpy()
            frames = list(videos)
        else:
            if arr.dtype != np.uint8:
                arr = (np.clip(arr, 0.0, 1.0) * 255.0).astype(np.uint8)
            frames = list(arr)

    # 2. Frame interpolation (video only)
    if enable_frame_interpolation and data_type == DataType.VIDEO and len(frames) > 1:
        from sglang.multimodal_gen.runtime.postprocess import (
            interpolate_video_frames,
        )

        frames, multiplier = interpolate_video_frames(
            frames,
            exp=frame_interpolation_exp,
            scale=frame_interpolation_scale,
            model_path=frame_interpolation_model_path,
        )
        fps = fps * multiplier

    # 3. Upscaling (images and videos)
    if enable_upscaling and frames:
        from sglang.multimodal_gen.runtime.postprocess import upscale_frames

        frames = upscale_frames(
            frames,
            model_path=upscaling_model_path,
            scale=upscaling_scale,
        )

    # 4. Save outputs if requested
    if save_output:
        if save_file_path:
            os.makedirs(os.path.dirname(save_file_path), exist_ok=True)
            if data_type == DataType.VIDEO:
                quality = (
                    output_compression / 10 if output_compression is not None else 5
                )
                imageio.mimsave(
                    save_file_path,
                    frames,
                    fps=fps,
                    format=data_type.get_default_extension(),
                    codec="libx264",
                    quality=quality,
                )

                _maybe_mux_audio_into_mp4(
                    save_file_path=save_file_path,
                    audio=audio,
                    frames=frames,
                    fps=fps,
                    audio_sample_rate=audio_sample_rate,
                )

            else:
                quality = output_compression if output_compression is not None else 75
                if len(frames) > 1:
                    for i, image in enumerate(frames):
                        parts = save_file_path.rsplit(".", 1)
                        if len(parts) == 2:
                            indexed_path = f"{parts[0]}_{i}.{parts[1]}"
                        else:
                            indexed_path = f"{save_file_path}_{i}"
                        imageio.imwrite(indexed_path, image, quality=quality)
                else:
                    imageio.imwrite(save_file_path, frames[0], quality=quality)
            logger.info(f"Output saved to {CYAN}{save_file_path}{RESET}")
        else:
            logger.info(f"No output path provided, output not saved")

    return frames
```
**EN:** This block defines function `post_process_sample`. Process sample output, optionally interpolate video frames, and save. Key calls include `isinstance`, `clamp.to`, `sample.permute.cpu.numpy`, `list`, and `interpolate_video_frames`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `sample`, `data_type`, `fps`, `save_output`, and `save_file_path` drive the behavior in this section.
**CN:** 该代码块定义了函数 `post_process_sample`。 它用于后处理process sample。 关键调用包括 `isinstance`、`clamp.to`、`sample.permute.cpu.numpy`、`list` 和 `interpolate_video_frames`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `sample`、`data_type`、`fps`、`save_output` 和 `save_file_path` 等参数驱动。

## Key Concepts / 关键概念
- `SetLoraReq`: Primary class that configures lora req. / 核心类，用于设置lora req。
- `MergeLoraWeightsReq`: Primary class that merges lora weights req. / 核心类，用于合并lora weights req。
- `UnmergeLoraWeightsReq`: Primary class that encapsulates unmerge lora weights req behavior. / 核心类，用于封装 unmerge lora weights req 相关行为。
- `ListLorasReq`: Primary class that encapsulates list loras req behavior. / 核心类，用于封装 list loras req 相关行为。
- `ShutdownReq`: Primary class that encapsulates shutdown req behavior. / 核心类，用于封装 shutdown req 相关行为。
- `GetDisaggStatsReq`: Request to get disagg pipeline metrics from the scheduler. / 核心类，用于获取disagg stats req。
- `format_lora_message`: Format success message for single or multiple LoRAs. / 顶层函数，用于处理 format lora message 相关逻辑。
- `GenerationResult`: Result of a single generation request from DiffGenerator. / 核心类，用于封装 generation result 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `shutil`, `subprocess`, `tempfile`, `copy`, `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `imageio`, `numpy`, `torch`, `scipy.io.wavfile`, `imageio_ffmpeg`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.sample.sampling_params`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.srt.observability.trace`, `sglang.multimodal_gen.runtime.postprocess`

- **Total lines / 总行数**: 655
