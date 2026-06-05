# encoder_cudagraph_defs.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/encoder_cudagraph_defs.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Data transfer objects for encoder CUDA graph management. / 该模块位于 `worker` 子系统，主要围绕 `EncoderCudaGraphConfig`, `EncoderCudaGraphCaptureInputs`, `EncoderCudaGraphReplayBuffers` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""Data transfer objects for encoder CUDA graph management."""

from dataclasses import dataclass
from typing import Any

import torch
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。

### `EncoderCudaGraphConfig` class / `EncoderCudaGraphConfig` 类
```python
@dataclass
class EncoderCudaGraphConfig:
    """Configuration for encoder CUDA graph management.

    Provided by the model at init time via
    ``get_encoder_cudagraph_config()``. Values are fixed for the
    lifetime of the manager.
    """

    modalities: list[str]
    """Supported modalities (e.g. ["image"])."""

    input_key_by_modality: dict[str, str]
    """Per-modality input tensor key mapping, e.g.
    {"image": "pixel_values", "video": "pixel_values_videos"}.
    """

    buffer_keys: list[str]
    """Keys for the tensor buffers recorded into the CUDA graph.
    Before replay the manager zeros then slice-copies new data
    into these buffers."""

    out_hidden_size: int
    """Output hidden dim of the vision encoder.
    Used for DP gather buffer allocation."""
```
**EN:** Uses `@dataclass` to package related state for `EncoderCudaGraphConfig`. Typical fields include `modalities`, `input_key_by_modality`, `buffer_keys`, `out_hidden_size`.
**CN:** `EncoderCudaGraphConfig` 使用 `@dataclass` 打包相关状态。典型字段包括 `modalities`, `input_key_by_modality`, `buffer_keys`, `out_hidden_size`。

### `EncoderCudaGraphCaptureInputs` class / `EncoderCudaGraphCaptureInputs` 类
```python
@dataclass
class EncoderCudaGraphCaptureInputs:
    """Everything needed for one CUDA graph capture.

    Returned by ``prepare_encoder_cudagraph_capture_inputs()``.
    """

    mm_kwargs: dict[str, Any]
    """Dummy forward inputs (model-specific keys).
    For Qwen3-VL this contains pixel_values and grid_thw."""

    buffers: dict[str, torch.Tensor]
    """Precomputed tensor buffers that will be recorded into the
    CUDA graph.  The manager stores references to these exact
    tensor objects and copies new data into them before each
    ``graph.replay()`` call (buffer identity invariant)."""
```
**EN:** Uses `@dataclass` to package related state for `EncoderCudaGraphCaptureInputs`. Typical fields include `mm_kwargs`, `buffers`.
**CN:** `EncoderCudaGraphCaptureInputs` 使用 `@dataclass` 打包相关状态。典型字段包括 `mm_kwargs`, `buffers`。

### `EncoderCudaGraphReplayBuffers` class / `EncoderCudaGraphReplayBuffers` 类
```python
@dataclass
class EncoderCudaGraphReplayBuffers:
    """New buffer values for graph replay, computed by the model from
    actual batch inputs.

    Returned by ``prepare_encoder_cudagraph_replay_buffers()``.
    Keys match ``EncoderCudaGraphConfig.buffer_keys``.
    """

    buffers: dict[str, torch.Tensor | None]
    """Data to copy into the captured buffers before replay.
    ``None`` values leave the corresponding captured buffer
    unchanged."""
```
**EN:** Uses `@dataclass` to package related state for `EncoderCudaGraphReplayBuffers`. Typical fields include `buffers`.
**CN:** `EncoderCudaGraphReplayBuffers` 使用 `@dataclass` 打包相关状态。典型字段包括 `buffers`。

## Key Concepts / 关键概念
- `EncoderCudaGraphConfig`: central class or interface in this module. / `EncoderCudaGraphConfig`：本模块中的核心类或接口。
- `EncoderCudaGraphCaptureInputs`: central class or interface in this module. / `EncoderCudaGraphCaptureInputs`：本模块中的核心类或接口。
- `EncoderCudaGraphReplayBuffers`: central class or interface in this module. / `EncoderCudaGraphReplayBuffers`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`, `typing`
- External / 外部依赖: `torch`
