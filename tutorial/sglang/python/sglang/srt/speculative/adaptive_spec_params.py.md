# adaptive_spec_params.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/speculative/adaptive_spec_params.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Adaptive speculative decoding parameters. / 该文件实现了推测解码基础设施，例如 worker、元数据对象、CUDA Graph 运行器或共享工具函数。它通过协调草稿模型执行与目标模型校验来提升吞吐。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19: Module header, imports, and shared constants
```python
"""Adaptive speculative decoding parameters.

Adjusts speculative_num_steps at runtime based on observed acceptance lengths.
"""

from __future__ import annotations

import json
import logging
from typing import TYPE_CHECKING

from sglang.srt.utils import log_info_on_rank0

if TYPE_CHECKING:
    from sglang.srt.server_args import ServerArgs

logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 20-52: adaptive unsupported reason function
```python
def adaptive_unsupported_reason(server_args: ServerArgs) -> str | None:
    """Return why adaptive spec cannot run under the given server args, or None if supported."""
    if server_args.speculative_algorithm not in ("EAGLE", "EAGLE3"):
        return (
            f"speculative_algorithm={server_args.speculative_algorithm} "
            "(only EAGLE/EAGLE3 are supported)"
        )
    if server_args.speculative_eagle_topk != 1:
        return (
            f"speculative_eagle_topk={server_args.speculative_eagle_topk} "
            "(only topk=1 is supported)"
        )
    if server_args.enable_dp_attention:
        return (
            "enable_dp_attention=True is not supported "
            "(adaptive tier decisions are not synchronized across DP ranks)"
        )
    if server_args.enable_multi_layer_eagle:
        return (
            "enable_multi_layer_eagle=True is not supported "
            "(MultiLayerEagleWorker does not implement adaptive)"
        )
    if server_args.enable_two_batch_overlap:
        return (
            "enable_two_batch_overlap=True is not supported "
            "(adaptive state swap would discard the TboAttnBackend wrapper)"
        )
    if server_args.enable_pdmux:
        return (
            "enable_pdmux=True is not supported "
            "(adaptive state swap does not update decode_attn_backend_group)"
        )
    return None
```
**EN:** This block uses `adaptive_unsupported_reason` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `adaptive_unsupported_reason` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 55-73: load adaptive config function
```python
def load_adaptive_config(path: str | None) -> dict[str, object]:
    """Load adaptive speculative config from a JSON file.

    The file may contain any subset of the following keys:
        ema_alpha, update_interval, warmup_batches,
        down_hysteresis, up_hysteresis, candidate_steps

    Returns an empty dict when *path* is ``None``.
    """
    if path is None:
        return {}
    with open(path) as f:
        cfg = json.load(f)
    if not isinstance(cfg, dict):
        raise ValueError(
            "speculative_adaptive_config must be a JSON object, "
            f"got {type(cfg).__name__}"
        )
    return cfg
```
**EN:** This block uses `load_adaptive_config` to load resources or weights. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `load_adaptive_config` 来加载资源或权重。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 76-87: AdaptiveSpeculativeParams class declaration
```python
class AdaptiveSpeculativeParams:
    """Tracks acceptance rate via EMA and adapts num_steps accordingly.

    The core idea: if drafts are consistently accepted, try more steps;
    if drafts are consistently rejected early, reduce steps to avoid waste.

    Formula: target_steps = clamp(round(ema_accept_len) + 1, min_steps, max_steps)
    - Probes one step beyond observed acceptance
    - EMA smoothing prevents oscillation
    - Only updates every `update_interval` batches for stability
    """
```
**EN:** This block declares the `AdaptiveSpeculativeParams` class, which exists to store configuration or metadata. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `AdaptiveSpeculativeParams` 类，其职责是存储配置或元数据。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 88-131: AdaptiveSpeculativeParams initializer
```python
    def __init__(
        self,
        initial_steps: int,
        config: dict[str, object] | None = None,
    ):
        cfg = config or {}
        # TODO: Wider range of candidate_steps (once lazy init is supported).
        candidates = set(cfg.get("candidate_steps", [1, 3, 7]))

        # Ensure the worker's initial speculative_num_steps is itself a candidate.
        # Otherwise AdaptiveController.register() would store the worker's pre-built
        # runtime state under a key that _activate() never queries, leaking that
        # state's draft attn backend and cuda graph buffers for the process lifetime.
        if initial_steps not in candidates:
            log_info_on_rank0(
                logger,
                f"Adding initial speculative_num_steps={initial_steps} to "
                f"candidate_steps={sorted(candidates)} so the pre-built "
                f"runtime state is reused.",
            )
            candidates.add(initial_steps)

        self.candidate_steps = sorted(candidates)
        assert (
            len(self.candidate_steps) >= 2
        ), "candidate_steps must have at least 2 distinct values"

        self.ema_alpha = cfg.get("ema_alpha", 0.2)
        self.update_interval = cfg.get("update_interval", 5)
        self.warmup_batches = cfg.get("warmup_batches", 10)
        self.down_hysteresis = cfg.get("down_hysteresis", -0.25)
        self.up_hysteresis = cfg.get("up_hysteresis", 0.0)

        self.current_steps = initial_steps

        # Initialize EMA at current steps - 1 (neutral starting point)
        self.ema_accept_len = float(self.current_steps - 1)
        self._batch_count = 0

        log_info_on_rank0(
            logger,
            f"AdaptiveSpeculativeParams initialized: "
            f"steps={self.current_steps}, candidate_steps={self.candidate_steps}",
        )
```
**EN:** This block initializes the `AdaptiveSpeculativeParams` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `AdaptiveSpeculativeParams` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 133-154: AdaptiveSpeculativeParams.update method
```python
    def update(self, num_correct_drafts_per_req: list[int]) -> bool:
        """Update EMA with observed accept lengths. Returns True if params changed.

        Args:
            num_correct_drafts_per_req: Per-request accepted draft token counts from last verify.
        """
        if not num_correct_drafts_per_req:
            return False

        batch_avg = sum(num_correct_drafts_per_req) / len(num_correct_drafts_per_req)
        self.ema_accept_len = (
            1 - self.ema_alpha
        ) * self.ema_accept_len + self.ema_alpha * batch_avg

        self._batch_count += 1
        if self._batch_count <= self.warmup_batches:
            return False

        if (self._batch_count - self.warmup_batches) % self.update_interval != 0:
            return False

        return self._recompute_params()
```
**EN:** This block uses `AdaptiveSpeculativeParams.update` to refresh state using new information. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `AdaptiveSpeculativeParams.update` 来利用新信息刷新状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 156-188: AdaptiveSpeculativeParams._recompute_params method
```python
    def _recompute_params(self) -> bool:
        """Recompute steps from EMA. Returns True if params changed."""
        old_steps = self.current_steps
        current_idx = self.candidate_steps.index(old_steps)

        # TODO: Consider limiting step changes to avoid overshooting.
        while current_idx > 0:
            prev_step = self.candidate_steps[current_idx - 1]
            drop_threshold = prev_step - 0.5 + self.down_hysteresis
            if self.ema_accept_len <= drop_threshold:
                current_idx -= 1
            else:
                break

        while current_idx < len(self.candidate_steps) - 1:
            current_step = self.candidate_steps[current_idx]
            rise_threshold = current_step - 0.5 + self.up_hysteresis
            if self.ema_accept_len > rise_threshold:
                current_idx += 1
            else:
                break

        target = self.candidate_steps[current_idx]

        if target != old_steps:
            self.current_steps = target
            log_info_on_rank0(
                logger,
                f"Adaptive spec params updated: steps {old_steps} -> {target} "
                f"(ema_accept_len={self.ema_accept_len:.2f})",
            )
            return True
        return False
```
**EN:** This block uses `AdaptiveSpeculativeParams._recompute_params` to store configuration or metadata. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `AdaptiveSpeculativeParams._recompute_params` 来存储配置或元数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- Adaptive speculative control / 自适应推测控制
- Speculative decoding / 推测解码

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.server_args`
- `sglang.srt.utils`
### External / 外部
- `__future__`
- `json` (stdlib)
- `logging` (stdlib)
- `typing` (stdlib)
