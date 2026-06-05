# io_struct.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/entrypoints/post_training/io_struct.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the service entrypoint layer. It centers on `UpdateWeightFromDiskReqInput`, `GetWeightsChecksumReqInput`, and `RolloutRequest`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Request/response data structures for post-training APIs. / 该文件属于服务入口层。它围绕 `UpdateWeightFromDiskReqInput`、`GetWeightsChecksumReqInput` 和 `RolloutRequest` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: module setup and imports / 模块初始化与导入
```python
"""Request/response data structures for post-training APIs."""

from __future__ import annotations

from dataclasses import dataclass
from typing import Any, Optional

from pydantic import BaseModel
```
**EN:** This block establishes the module context and imports `__future__`, `dataclasses`, `typing`, and `pydantic`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `__future__`、`dataclasses`、`typing` 和 `pydantic`。这些依赖为后续实现提供所需符号。

### Lines 12-14: `UpdateWeightFromDiskReqInput` class overview / `UpdateWeightFromDiskReqInput` 类概览
```python
class UpdateWeightFromDiskReqInput:
    """Request to update model weights from disk for diffusion models."""
```
**EN:** This block defines class `UpdateWeightFromDiskReqInput`. Request to update model weights from disk for diffusion models.
**CN:** 该代码块定义了类 `UpdateWeightFromDiskReqInput`。 它用于更新weight from disk req input。

### Lines 15-17: supporting statements / 辅助语句
```python
    model_path: str
    flush_cache: bool = True
    target_modules: list[str] | None = None
```
**EN:** This block gathers supporting statements inside `UpdateWeightFromDiskReqInput`. It updates names such as `model_path`, `flush_cache`, and `target_modules`.
**CN:** 该代码块汇集了位于 `UpdateWeightFromDiskReqInput` 内部的辅助语句。 它会更新 `model_path`、`flush_cache` 和 `target_modules` 等名称。

### Lines 21-23: `GetWeightsChecksumReqInput` class overview / `GetWeightsChecksumReqInput` 类概览
```python
class GetWeightsChecksumReqInput:
    """Compute SHA-256 checksum of loaded module weights for verification."""
```
**EN:** This block defines class `GetWeightsChecksumReqInput`. Compute SHA-256 checksum of loaded module weights for verification.
**CN:** 该代码块定义了类 `GetWeightsChecksumReqInput`。 它用于获取weights checksum req input。

### Lines 24-24: supporting statements / 辅助语句
```python
    module_names: list[str] | None = None
```
**EN:** This block gathers supporting statements inside `GetWeightsChecksumReqInput`. It updates names such as `module_names`.
**CN:** 该代码块汇集了位于 `GetWeightsChecksumReqInput` 内部的辅助语句。 它会更新 `module_names` 等名称。

### Lines 27-27: `RolloutRequest` class overview / `RolloutRequest` 类概览
```python
class RolloutRequest(BaseModel):
```
**EN:** This block defines class `RolloutRequest`. It encapsulates rollout request behavior. It inherits from `BaseModel`.
**CN:** 该代码块定义了类 `RolloutRequest`。 它用于封装 rollout request 相关行为。 它继承自 `BaseModel`。

### Lines 28-63: supporting statements / 辅助语句
```python
    prompt: str
    negative_prompt: Optional[str] = None
    seed: Optional[int] = None
    generator_device: str = "cuda"

    width: Optional[int] = None
    height: Optional[int] = None
    num_inference_steps: Optional[int] = None
    num_outputs_per_prompt: Optional[int] = None

    guidance_scale: Optional[float] = None
    true_cfg_scale: Optional[float] = None

    # video-specific (ignored by image pipelines)
    num_frames: Optional[int] = None
    fps: Optional[int] = None

    rollout: bool = True
    rollout_sde_type: str = "sde"
    rollout_noise_level: float = 0.7
    rollout_log_prob_no_const: bool = False
    rollout_debug_mode: bool = True

    rollout_return_denoising_env: bool = False
    rollout_return_dit_trajectory: bool = False

    # 0-indexed denoising-loop step filters. None = all steps.
    rollout_sde_step_indices: Optional[list[int]] = None
    rollout_return_step_indices: Optional[list[int]] = None

    image_path: Optional[list[str]] = None

    # suppress verbose per-request logging (also gates peak_memory_mb collection)
    suppress_logs: bool = False

    extra_sampling_params: Optional[dict[str, Any]] = None
```
**EN:** This block gathers supporting statements inside `RolloutRequest`. It updates names such as `prompt`, `negative_prompt`, `seed`, `generator_device`, `width`, and `height`.
**CN:** 该代码块汇集了位于 `RolloutRequest` 内部的辅助语句。 它会更新 `prompt`、`negative_prompt`、`seed`、`generator_device`、`width` 和 `height` 等名称。

### Lines 66-66: `RolloutResponse` class overview / `RolloutResponse` 类概览
```python
class RolloutResponse(BaseModel):
```
**EN:** This block defines class `RolloutResponse`. It encapsulates rollout response behavior. It inherits from `BaseModel`.
**CN:** 该代码块定义了类 `RolloutResponse`。 它用于封装 rollout response 相关行为。 它继承自 `BaseModel`。

### Lines 67-79: supporting statements / 辅助语句
```python
    request_id: str
    prompt: str
    seed: int

    generated_output: Any = None

    rollout_log_probs: Optional[dict[str, Any]] = None
    rollout_debug_tensors: Optional[dict[str, Any]] = None
    denoising_env: Optional[dict[str, Any]] = None
    dit_trajectory: Optional[dict[str, Any]] = None

    inference_time_s: Optional[float] = None
    peak_memory_mb: Optional[float] = None
```
**EN:** This block gathers supporting statements inside `RolloutResponse`. It updates names such as `request_id`, `prompt`, `seed`, `generated_output`, `rollout_log_probs`, and `rollout_debug_tensors`.
**CN:** 该代码块汇集了位于 `RolloutResponse` 内部的辅助语句。 它会更新 `request_id`、`prompt`、`seed`、`generated_output`、`rollout_log_probs` 和 `rollout_debug_tensors` 等名称。

## Key Concepts / 关键概念
- `UpdateWeightFromDiskReqInput`: Request to update model weights from disk for diffusion models. / 核心类，用于更新weight from disk req input。
- `GetWeightsChecksumReqInput`: Compute SHA-256 checksum of loaded module weights for verification. / 核心类，用于获取weights checksum req input。
- `RolloutRequest`: Primary class that encapsulates rollout request behavior. / 核心类，用于封装 rollout request 相关行为。
- `RolloutResponse`: Primary class that encapsulates rollout response behavior. / 核心类，用于封装 rollout response 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `pydantic`

- **Total lines / 总行数**: 79
