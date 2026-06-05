# cfg_policy.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/distributed/cfg_policy.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the distributed runtime layer. It centers on `CFGBranch`, `CFGPolicy`, and `_wrap`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于分布式运行时层。它围绕 `CFGBranch`、`CFGPolicy` 和 `_wrap` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: module setup and imports / 模块初始化与导入
```python
from __future__ import annotations

import dataclasses
from dataclasses import dataclass, field
from typing import TYPE_CHECKING, Any

import torch
```
**EN:** This block establishes the module context and imports `__future__`, `dataclasses`, `typing`, and `torch`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `__future__`、`dataclasses`、`typing` 和 `torch`。这些依赖为后续实现提供所需符号。

### Lines 9-10: supporting statements / 辅助语句
```python
if TYPE_CHECKING:
    from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import Req
```
**EN:** This block gathers supporting statements at module scope.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。

### Lines 14-19: `CFGBranch` class overview / `CFGBranch` 类概览
```python
class CFGBranch:
    """Immutable specification of one CFG branch forward pass.

    Built once before the denoising loop; read-only across all steps.
    """
```
**EN:** This block defines class `CFGBranch`. Immutable specification of one CFG branch forward pass. Built once before the denoising loop; read-only across all steps.
**CN:** 该代码块定义了类 `CFGBranch`。 它用于封装 cfgbranch 相关行为。

### Lines 20-22: supporting statements / 辅助语句
```python
    name: str
    is_conditional: bool
    kwargs: dict[str, Any]
```
**EN:** This block gathers supporting statements inside `CFGBranch`. It updates names such as `name`, `is_conditional`, and `kwargs`.
**CN:** 该代码块汇集了位于 `CFGBranch` 内部的辅助语句。 它会更新 `name`、`is_conditional` 和 `kwargs` 等名称。

### Lines 24-30: `configure_batch` implementation / `configure_batch` 实现
```python
    def configure_batch(self, batch: "Req") -> None:
        """Set batch state before this branch's forward pass.

        Override for richer per-branch context (e.g. a branch index instead of
        a single boolean) when a model needs more than two guidance modes.
        """
        batch.is_cfg_negative = not self.is_conditional
```
**EN:** This block defines method `configure_batch` on `CFGBranch`. Set batch state before this branch's forward pass. Override for richer per-branch context (e.g. Parameters such as `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `CFGBranch` 的方法 `configure_batch`。 它用于处理 configure batch 相关逻辑。 本段逻辑主要由 `batch` 等参数驱动。

### Lines 34-44: `CFGPolicy` class overview / `CFGPolicy` 类概览
```python
class CFGPolicy:
    """Owns the CFG branches for one generation run and combines their predictions.

    Built once before the denoising loop via ``build()``, then used read-only
    across all steps.  Subclass and override ``build()`` / ``combine()`` for
    custom CFG schemes (N-branch, multi-output, etc.).

    The default implementation handles standard 2-branch CFG.  With a single
    branch (CFG disabled) ``combine()`` returns the prediction unchanged.
    """
```
**EN:** This block defines class `CFGPolicy`. Owns the CFG branches for one generation run and combines their predictions. Built once before the denoising loop via ``build()``, then used read-only across all steps.
**CN:** 该代码块定义了类 `CFGPolicy`。 它用于封装 cfgpolicy 相关行为。

### Lines 45-45: supporting statements / 辅助语句
```python
    branches: list[CFGBranch] = field(default_factory=list)
```
**EN:** This block gathers supporting statements inside `CFGPolicy`. It updates names such as `branches`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `CFGPolicy` 内部的辅助语句。 它会更新 `branches` 等名称。 代码会与 `field` 协同工作。

### Lines 47-64: `build` implementation / `build` 实现
```python
    def build(
        self,
        batch: "Req",
        image_kwargs: dict[str, Any],
        pos_cond_kwargs: dict[str, Any],
        neg_cond_kwargs: dict[str, Any],
    ) -> "CFGPolicy":
        """Return a new policy with branches populated.

        Called once before the denoising loop.  The returned policy is
        immutable for the lifetime of the run.  Override to declare N branches.
        """
        branches = [CFGBranch("conditional", True, {**image_kwargs, **pos_cond_kwargs})]
        if batch.do_classifier_free_guidance:
            branches.append(
                CFGBranch("unconditional", False, {**image_kwargs, **neg_cond_kwargs})
            )
        return dataclasses.replace(self, branches=branches)
```
**EN:** This block defines method `build` on `CFGPolicy`. Return a new policy with branches populated. Called once before the denoising loop. Key calls include `dataclasses.replace`, `CFGBranch`, and `branches.append`. The implementation branches on conditions. Parameters such as `batch`, `image_kwargs`, `pos_cond_kwargs`, and `neg_cond_kwargs` drive the behavior in this section.
**CN:** 该代码块定义了 `CFGPolicy` 的方法 `build`。 它用于构建函数。 关键调用包括 `dataclasses.replace`、`CFGBranch` 和 `branches.append`。 实现中包含条件分支。 本段逻辑主要由 `batch`、`image_kwargs`、`pos_cond_kwargs` 和 `neg_cond_kwargs` 等参数驱动。

### Lines 66-99: `combine` implementation / `combine` 实现
```python
    def combine(
        self,
        predictions: list[torch.Tensor | tuple[torch.Tensor, ...]],
        batch: "Req",
        cfg_scale: float,
        pipeline_config: Any,
        *,
        cfg_parallel: bool = False,
    ) -> torch.Tensor | tuple[torch.Tensor, ...]:
        """Combine branch predictions into the final noise estimate.

        Default: standard 2-branch CFG formula applied element-wise, followed
        by normalization / rescale / model-specific postprocess.
        Single-branch (CFG disabled): returns the prediction unchanged.
        Override for N-branch or multi-output models.
        """
        if len(predictions) == 1:
            return predictions[0]
        pos_t = _wrap(predictions[0])
        neg_t = _wrap(predictions[1])
        if cfg_parallel:
            # Match the old CFG-parallel calculation: multiply the positive
            # prediction by cfg_scale and the negative prediction by
            # (1 - cfg_scale) before adding them. The serial CFG formula is
            # mathematically equivalent, but bf16 rounding changes WAN outputs.
            results = [
                cfg_scale * p + (1 - cfg_scale) * n for p, n in zip(pos_t, neg_t)
            ]
        else:
            results = [n + cfg_scale * (p - n) for p, n in zip(pos_t, neg_t)]
        results[0] = _apply_cfg_postprocess(
            results[0], pos_t[0], batch, pipeline_config
        )
        return _unwrap(tuple(results))
```
**EN:** This block defines method `combine` on `CFGPolicy`. Combine branch predictions into the final noise estimate. Default: standard 2-branch CFG formula applied element-wise, followed by normalization / rescale / model-specific postprocess. Key calls include `_wrap`, `_apply_cfg_postprocess`, `_unwrap`, `len`, and `tuple`. The implementation branches on conditions. Parameters such as `predictions`, `batch`, `cfg_scale`, and `pipeline_config` drive the behavior in this section.
**CN:** 该代码块定义了 `CFGPolicy` 的方法 `combine`。 它用于处理 combine 相关逻辑。 关键调用包括 `_wrap`、`_apply_cfg_postprocess`、`_unwrap`、`len` 和 `tuple`。 实现中包含条件分支。 本段逻辑主要由 `predictions`、`batch`、`cfg_scale` 和 `pipeline_config` 等参数驱动。

### Lines 105-108: `_wrap` implementation / `_wrap` 实现
```python
def _wrap(
    pred: torch.Tensor | tuple[torch.Tensor, ...],
) -> tuple[torch.Tensor, ...]:
    return pred if isinstance(pred, tuple) else (pred,)
```
**EN:** This block defines function `_wrap`. It handles wrap logic. Key calls include `isinstance`. Parameters such as `pred` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_wrap`。 它用于处理 wrap 相关逻辑。 关键调用包括 `isinstance`。 本段逻辑主要由 `pred` 等参数驱动。

### Lines 111-114: `_unwrap` implementation / `_unwrap` 实现
```python
def _unwrap(
    pred: tuple[torch.Tensor, ...],
) -> torch.Tensor | tuple[torch.Tensor, ...]:
    return pred[0] if len(pred) == 1 else pred
```
**EN:** This block defines function `_unwrap`. It handles unwrap logic. Key calls include `len`. Parameters such as `pred` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_unwrap`。 它用于处理 unwrap 相关逻辑。 关键调用包括 `len`。 本段逻辑主要由 `pred` 等参数驱动。

### Lines 117-131: `_apply_cfg_postprocess` implementation / `_apply_cfg_postprocess` 实现
```python
def _apply_cfg_postprocess(
    noise_pred: torch.Tensor,
    noise_pred_cond: torch.Tensor,
    batch: "Req",
    pipeline_config: Any,
) -> torch.Tensor:
    if batch.cfg_normalization and float(batch.cfg_normalization) > 0:
        noise_pred = _apply_cfg_normalization(
            noise_pred, noise_pred_cond, float(batch.cfg_normalization)
        )
    if batch.guidance_rescale > 0.0:
        noise_pred = _rescale_noise_cfg(
            noise_pred, noise_pred_cond, guidance_rescale=batch.guidance_rescale
        )
    return pipeline_config.postprocess_cfg_noise(batch, noise_pred, noise_pred_cond)
```
**EN:** This block defines function `_apply_cfg_postprocess`. It applies cfg postprocess. Key calls include `pipeline_config.postprocess_cfg_noise`, `_apply_cfg_normalization`, `_rescale_noise_cfg`, and `float`. The implementation branches on conditions. Parameters such as `noise_pred`, `noise_pred_cond`, `batch`, and `pipeline_config` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_apply_cfg_postprocess`。 它用于应用cfg postprocess。 关键调用包括 `pipeline_config.postprocess_cfg_noise`、`_apply_cfg_normalization`、`_rescale_noise_cfg` 和 `float`。 实现中包含条件分支。 本段逻辑主要由 `noise_pred`、`noise_pred_cond`、`batch` 和 `pipeline_config` 等参数驱动。

### Lines 134-146: `_apply_cfg_normalization` implementation / `_apply_cfg_normalization` 实现
```python
def _apply_cfg_normalization(
    noise_pred: torch.Tensor,
    noise_pred_cond: torch.Tensor,
    cfg_normalization: float,
) -> torch.Tensor:
    cond_f = noise_pred_cond.float()
    pred_f = noise_pred.float()
    ori_norm = torch.linalg.vector_norm(cond_f)
    new_norm = torch.linalg.vector_norm(pred_f)
    max_norm = ori_norm * cfg_normalization
    if new_norm > max_norm:
        noise_pred = noise_pred * (max_norm / new_norm)
    return noise_pred
```
**EN:** This block defines function `_apply_cfg_normalization`. It applies cfg normalization. Key calls include `noise_pred_cond.float`, `noise_pred.float`, and `torch.linalg.vector_norm`. The implementation branches on conditions. Parameters such as `noise_pred`, `noise_pred_cond`, and `cfg_normalization` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_apply_cfg_normalization`。 它用于应用cfg normalization。 关键调用包括 `noise_pred_cond.float`、`noise_pred.float` 和 `torch.linalg.vector_norm`。 实现中包含条件分支。 本段逻辑主要由 `noise_pred`、`noise_pred_cond` 和 `cfg_normalization` 等参数驱动。

### Lines 149-159: `_rescale_noise_cfg` implementation / `_rescale_noise_cfg` 实现
```python
def _rescale_noise_cfg(
    noise_cfg: torch.Tensor,
    noise_pred_text: torch.Tensor,
    guidance_rescale: float = 0.0,
) -> torch.Tensor:
    std_text = noise_pred_text.std(
        dim=list(range(1, noise_pred_text.ndim)), keepdim=True
    )
    std_cfg = noise_cfg.std(dim=list(range(1, noise_cfg.ndim)), keepdim=True)
    noise_pred_rescaled = noise_cfg * (std_text / std_cfg)
    return guidance_rescale * noise_pred_rescaled + (1 - guidance_rescale) * noise_cfg
```
**EN:** This block defines function `_rescale_noise_cfg`. It handles rescale noise cfg logic. Key calls include `noise_pred_text.std`, `noise_cfg.std`, `list`, and `range`. Parameters such as `noise_cfg`, `noise_pred_text`, and `guidance_rescale` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_rescale_noise_cfg`。 它用于处理 rescale noise cfg 相关逻辑。 关键调用包括 `noise_pred_text.std`、`noise_cfg.std`、`list` 和 `range`。 本段逻辑主要由 `noise_cfg`、`noise_pred_text` 和 `guidance_rescale` 等参数驱动。

## Key Concepts / 关键概念
- `CFGBranch`: Immutable specification of one CFG branch forward pass. / 核心类，用于封装 cfgbranch 相关行为。
- `CFGPolicy`: Owns the CFG branches for one generation run and combines their predictions. / 核心类，用于封装 cfgpolicy 相关行为。
- `_wrap`: Top-level function that handles wrap logic. / 顶层函数，用于处理 wrap 相关逻辑。
- `_unwrap`: Top-level function that handles unwrap logic. / 顶层函数，用于处理 unwrap 相关逻辑。
- `_apply_cfg_postprocess`: Top-level function that applies cfg postprocess. / 顶层函数，用于应用cfg postprocess。
- `_apply_cfg_normalization`: Top-level function that applies cfg normalization. / 顶层函数，用于应用cfg normalization。
- `_rescale_noise_cfg`: Top-level function that handles rescale noise cfg logic. / 顶层函数，用于处理 rescale noise cfg 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`

- **Total lines / 总行数**: 159
