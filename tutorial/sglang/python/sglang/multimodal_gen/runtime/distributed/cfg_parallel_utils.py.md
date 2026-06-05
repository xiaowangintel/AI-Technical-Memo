# cfg_parallel_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/distributed/cfg_parallel_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the distributed runtime layer. It centers on `_run`, `run_cfg_parallel`, and `run_two_branch_cfg_parallel`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于分布式运行时层。它围绕 `_run`、`run_cfg_parallel` 和 `run_two_branch_cfg_parallel` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: module setup and imports / 模块初始化与导入
```python
from __future__ import annotations

import dataclasses
from typing import TYPE_CHECKING, Callable

import torch

from sglang.multimodal_gen.runtime.distributed import get_local_torch_device
from sglang.multimodal_gen.runtime.distributed.cfg_policy import (
    _apply_cfg_postprocess,
    _unwrap,
    _wrap,
)
from sglang.multimodal_gen.runtime.distributed.communication_op import (
    cfg_model_parallel_all_gather,
    cfg_model_parallel_all_reduce,
)
from sglang.multimodal_gen.runtime.distributed.parallel_state import (
    get_cfg_group,
    get_classifier_free_guidance_rank,
    get_classifier_free_guidance_world_size,
)
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `__future__`, `dataclasses`, `typing`, `torch`, `sglang.multimodal_gen.runtime.distributed`, and `sglang.multimodal_gen.runtime.distributed.cfg_policy`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `__future__`、`dataclasses`、`typing`、`torch`、`sglang.multimodal_gen.runtime.distributed` 和 `sglang.multimodal_gen.runtime.distributed.cfg_policy`。这些依赖为后续实现提供所需符号。

### Lines 25-35: supporting statements / 辅助语句
```python
logger = init_logger(__name__)

if TYPE_CHECKING:
    from sglang.multimodal_gen.runtime.distributed.cfg_policy import (
        CFGBranch,
        CFGPolicy,
    )

# Tracks (n_branches, cfg_world_size, cfg_rank) tuples already logged so the
# dispatch table is printed once per unique configuration, not once per step.
_logged_dispatch_keys: set[tuple[int, int, int]] = set()
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`, and `_logged_dispatch_keys`. The code collaborates with `init_logger`, and `set`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 和 `_logged_dispatch_keys` 等名称。 代码会与 `init_logger` 和 `set` 协同工作。

### Lines 38-53: `_run` implementation / `_run` 实现
```python
def _run(
    predict_fn: Callable[["CFGBranch"], "torch.Tensor | tuple[torch.Tensor, ...]"],
    bid: int,
    branches,
) -> tuple[torch.Tensor, ...]:
    branch = branches[bid]
    device = get_local_torch_device()
    local_branch = dataclasses.replace(
        branch,
        kwargs={
            k: v.to(device) if isinstance(v, torch.Tensor) else v
            for k, v in branch.kwargs.items()
        },
    )
    raw = predict_fn(local_branch)
    return _wrap(raw)
```
**EN:** This block defines function `_run`. It runs function. Key calls include `get_local_torch_device`, `dataclasses.replace`, `predict_fn`, `_wrap`, and `isinstance`. Parameters such as `predict_fn`, `bid`, and `branches` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_run`。 它用于运行函数。 关键调用包括 `get_local_torch_device`、`dataclasses.replace`、`predict_fn`、`_wrap` 和 `isinstance`。 本段逻辑主要由 `predict_fn`、`bid` 和 `branches` 等参数驱动。

### Lines 56-135: `run_cfg_parallel` implementation / `run_cfg_parallel` 实现
```python
def run_cfg_parallel(
    policy: "CFGPolicy",
    predict_fn: Callable[["CFGBranch"], "torch.Tensor | tuple[torch.Tensor, ...]"],
) -> "list[torch.Tensor | tuple[torch.Tensor, ...]]":
    """Dispatch CFG branches across ranks, all-gather results, return in branch order.

    ``predict_fn`` is a closure capturing all step-varying state
    (latent_model_input, timestep, model, etc.).  It is called with each
    assigned ``CFGBranch`` and must return the raw ``_predict_noise`` output.

    Idle ranks (cfg_world_size > n_branches) run branch 0 as a dummy forward
    to obtain tensor shapes for the all-gather.

    Returns a list indexed to match ``policy.branches``, identical on every rank.
    """

    cfg_rank = get_classifier_free_guidance_rank()
    cfg_world_size = get_classifier_free_guidance_world_size()
    branches = policy.branches
    n_branches = len(branches)
    assignments = dispatch_branches(n_branches, cfg_world_size)
    branches_assigned_to_local_rank = assignments[cfg_rank]
    max_num_branches_per_rank = max(len(a) for a in assignments)

    if cfg_world_size > n_branches:
        logger.warning_once(
            "cfg_parallel_size=%d > n_branches=%d; %d GPU(s) will be idle for CFG",
            cfg_world_size,
            n_branches,
            cfg_world_size - n_branches,
        )

    dispatch_key = (n_branches, cfg_world_size, cfg_rank)
    if dispatch_key not in _logged_dispatch_keys:
        _logged_dispatch_keys.add(dispatch_key)
        branch_names = (
            [branches[i].name for i in branches_assigned_to_local_rank]
            if branches_assigned_to_local_rank
            else ["(idle)"]
        )
        logger.info(
            "CFG parallel dispatch: rank %d/%d -> [%s]",
            cfg_rank,
            cfg_world_size,
            ", ".join(branch_names),
        )

    # perform the forward for local branches
    predicts_from_local_branches: list[tuple[torch.Tensor, ...]] = [
        _run(predict_fn, bid, branches) for bid in branches_assigned_to_local_rank
    ]

    if not predicts_from_local_branches:  # idle rank: run branch 0 for tensor shapes
        predicts_from_local_branches.append(_run(predict_fn, 0, branches))

    # pad the predicts to the length of max_num_branches_per_rank, to prepare for the all-gather later
    ref = predicts_from_local_branches[0]
    while len(predicts_from_local_branches) < max_num_branches_per_rank:
        # TODO: cache this zero
        predicts_from_local_branches.append(tuple(torch.zeros_like(t) for t in ref))

    # All-gather each slot and output element with separate_tensors=True.
    # all_slots[slot][elem] = list[Tensor] indexed by CFG rank; no reshape.
    all_slots: list[list[list[torch.Tensor]]] = [
        [
            cfg_model_parallel_all_gather(p, dim=0, separate_tensors=True)
            for p in slot_pred
        ]
        for slot_pred in predicts_from_local_branches
    ]

    # reorder the results in branch order: branch bid -> owner rank, slot.
    n_elems = len(ref)
    final: list[torch.Tensor | tuple[torch.Tensor, ...]] = []
    for bid in range(n_branches):
        owner = bid % cfg_world_size
        slot = bid // cfg_world_size
        elems = tuple(all_slots[slot][ei][owner] for ei in range(n_elems))
        final.append(_unwrap(elems))
    return final
```
**EN:** This block defines function `run_cfg_parallel`. Dispatch CFG branches across ranks, all-gather results, return in branch order. ``predict_fn`` is a closure capturing all step-varying state (latent_model_input, timestep, model, etc.). Key calls include `get_classifier_free_guidance_rank`, `get_classifier_free_guidance_world_size`, `len`, `dispatch_branches`, and `max`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `policy`, and `predict_fn` drive the behavior in this section.
**CN:** 该代码块定义了函数 `run_cfg_parallel`。 它用于运行cfg parallel。 关键调用包括 `get_classifier_free_guidance_rank`、`get_classifier_free_guidance_world_size`、`len`、`dispatch_branches` 和 `max`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `policy` 和 `predict_fn` 等参数驱动。

### Lines 138-165: `run_two_branch_cfg_parallel` implementation / `run_two_branch_cfg_parallel` 实现
```python
def run_two_branch_cfg_parallel(
    policy: "CFGPolicy",
    predict_fn: Callable[["CFGBranch"], "torch.Tensor | tuple[torch.Tensor, ...]"],
    cfg_scale: float,
    batch,
    pipeline_config,
) -> "torch.Tensor | tuple[torch.Tensor, ...]":
    """Run standard two-pass CFG with the old all-reduce combine.

    This keeps the existing WAN baselines: it avoids gathering both branch
    predictions, and it preserves the bf16 arithmetic order used before the
    multi-branch CFG dispatcher was added.
    """

    cfg_rank = get_classifier_free_guidance_rank()
    pred_t = _run(predict_fn, cfg_rank, policy.branches)

    if cfg_rank == 0:
        partial = tuple(cfg_scale * p for p in pred_t)
        cond_t = pred_t
    else:
        partial = tuple((1 - cfg_scale) * p for p in pred_t)
        cond_t = tuple(torch.empty_like(p) for p in pred_t)

    results = [cfg_model_parallel_all_reduce(p) for p in partial]
    cond_t = tuple(get_cfg_group().broadcast(p, src=0) for p in cond_t)
    results[0] = _apply_cfg_postprocess(results[0], cond_t[0], batch, pipeline_config)
    return _unwrap(tuple(results))
```
**EN:** This block defines function `run_two_branch_cfg_parallel`. Run standard two-pass CFG with the old all-reduce combine. This keeps the existing WAN baselines: it avoids gathering both branch predictions, and it preserves the bf16 arithmetic order used before the multi-branch CFG dispatcher was added. Key calls include `get_classifier_free_guidance_rank`, `_run`, `tuple`, `_apply_cfg_postprocess`, and `_unwrap`. The implementation branches on conditions. Parameters such as `policy`, `predict_fn`, `cfg_scale`, `batch`, and `pipeline_config` drive the behavior in this section.
**CN:** 该代码块定义了函数 `run_two_branch_cfg_parallel`。 它用于运行two branch cfg parallel。 关键调用包括 `get_classifier_free_guidance_rank`、`_run`、`tuple`、`_apply_cfg_postprocess` 和 `_unwrap`。 实现中包含条件分支。 本段逻辑主要由 `policy`、`predict_fn`、`cfg_scale`、`batch` 和 `pipeline_config` 等参数驱动。

### Lines 168-181: `dispatch_branches` implementation / `dispatch_branches` 实现
```python
def dispatch_branches(n_branches: int, n_ranks: int) -> list[list[int]]:
    """Assign branches to ranks in Round-robin fashion

    Returns a list of length ``n_ranks`` where element ``r`` contains the
    branch indices assigned to rank ``r``.  Branch ``i`` goes to rank
    ``i % n_ranks``.

    Example: 4 passes, 2 GPUs:
        rank 0 -> [0, 2],  rank 1 -> [1, 3]
    """
    assignments: list[list[int]] = [[] for _ in range(n_ranks)]
    for i in range(n_branches):
        assignments[i % n_ranks].append(i)
    return assignments
```
**EN:** This block defines function `dispatch_branches`. Assign branches to ranks in Round-robin fashion Returns a list of length ``n_ranks`` where element ``r`` contains the branch indices assigned to rank ``r``. Branch ``i`` goes to rank ``i % n_ranks``. Key calls include `range`, and `assignments.append`. The implementation iterates over collections or steps. Parameters such as `n_branches`, and `n_ranks` drive the behavior in this section.
**CN:** 该代码块定义了函数 `dispatch_branches`。 它用于处理 dispatch branches 相关逻辑。 关键调用包括 `range` 和 `assignments.append`。 实现中会遍历集合或步骤。 本段逻辑主要由 `n_branches` 和 `n_ranks` 等参数驱动。

## Key Concepts / 关键概念
- `_run`: Top-level function that runs function. / 顶层函数，用于运行函数。
- `run_cfg_parallel`: Dispatch CFG branches across ranks, all-gather results, return in branch order. / 顶层函数，用于运行cfg parallel。
- `run_two_branch_cfg_parallel`: Run standard two-pass CFG with the old all-reduce combine. / 顶层函数，用于运行two branch cfg parallel。
- `dispatch_branches`: Assign branches to ranks in Round-robin fashion Returns a list of length ``n_ranks`` where element ``r`` contains the branch indices assigned to rank ``r``. / 顶层函数，用于处理 dispatch branches 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.distributed.cfg_policy`, `sglang.multimodal_gen.runtime.distributed.communication_op`, `sglang.multimodal_gen.runtime.distributed.parallel_state`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 181
