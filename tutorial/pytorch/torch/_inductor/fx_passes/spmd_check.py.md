# spmd_check.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/spmd_check.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It exposes functions such as `_compute_hash`, `_build_diag_fingerprint`, `_format_val_metadata`, `spmd_check`, `_entry_target`, `_entry_metadata`, and `...+1`. Module note: SPMD graph verification for overlap scheduling.
- **用途（中文）**: 该模块实现 FX 图变换 pass。同时提供 `_compute_hash`、`_build_diag_fingerprint`、`_format_val_metadata`、`spmd_check`、`_entry_target`、`_entry_metadata`、`另有1项` 等函数。模块文档字符串给出了额外背景说明。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行
````python
"""SPMD graph verification for overlap scheduling.

Verifies all ranks have identical FX graph structure before collective
reordering passes. Non-SPMD graphs cause NCCL collective ordering
mismatches and hangs.
"""

import hashlib
import logging
from collections import Counter

import torch
from torch._inductor import config
from torch._logging import trace_structured
````
- **EN**: Imports dependencies such as `hashlib`, `logging`, `collections`, `torch`, `torch._inductor`, and `torch._logging` for the logic in this range.
- **CN**: 这里导入了 `hashlib`、`logging`、`collections`、`torch`、`torch._inductor`、`torch._logging` 等依赖，为后续逻辑提供基础能力。

### Lines 15-28 / 第 15-28 行
````python


log = logging.getLogger(__name__)


def _compute_hash(gm: torch.fx.GraphModule) -> int | None:
    """Compute a structural hash of the graph including tensor metadata.

    Uses FxGraphCachePickler(device_id_agnostic=True) to serialize
    (target, val) per call_function node, capturing op targets and
    FakeTensor metadata (dtype, shape, stride, etc.) with device indices
    normalized to 0.

    Returns None if the graph contains unpicklable objects.
````
- **EN**: Introduces function `_compute_hash`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`_compute_hash`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 29-42 / 第 29-42 行
````python
    """
    from torch._inductor.codecache import BypassFxGraphCache, FxGraphCachePickler

    try:
        pickler = FxGraphCachePickler(gm, device_id_agnostic=True)
        data = pickler.dumps(
            tuple(
                (str(n.target), n.meta.get("val"))
                for n in gm.graph.nodes
                if n.op == "call_function"
            )
        )
        digest = hashlib.blake2b(data, digest_size=8).digest()
        return int.from_bytes(digest, "big", signed=True)
````
- **EN**: Imports dependencies such as `torch._inductor.codecache` for the logic in this range. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch._inductor.codecache` 等依赖，为后续逻辑提供基础能力。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 43-56 / 第 43-56 行
````python
    except BypassFxGraphCache:
        # FxGraphCachePickler can't serialize certain objects:
        # mkldnn tensors, BackwardState, torchbind objects, or general
        # pickle failures. Skip the SPMD check gracefully.
        log.warning("SPMD check: skipping, unpicklable graph objects", exc_info=True)
        return None


def _build_diag_fingerprint(
    gm: torch.fx.GraphModule,
) -> tuple[tuple[str, str | None], ...]:
    """Build human-readable fingerprint for mismatch diagnostics.

    Only called on the rare mismatch path.
````
- **EN**: Introduces function `_build_diag_fingerprint`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_build_diag_fingerprint`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 57-70 / 第 57-70 行
````python
    """
    from torch._inductor.codecache import extract_tensor_metadata_for_cache_key

    entries: list[tuple[str, str | None]] = []
    for n in gm.graph.nodes:
        if n.op != "call_function":
            continue
        target_str = str(n.target)
        val = n.meta.get("val")
        entries.append(
            (
                target_str,
                _format_val_metadata(val, extract_tensor_metadata_for_cache_key),
            )
````
- **EN**: Imports dependencies such as `torch._inductor.codecache` for the logic in this range. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `entries`, `target_str`, and `val`.
- **CN**: 这里导入了 `torch._inductor.codecache` 等依赖，为后续逻辑提供基础能力。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含分支、循环或上下文管理等控制流。初始化或更新了 `entries`、`target_str`、`val` 等值。

### Lines 71-84 / 第 71-84 行
````python
        )
    return tuple(entries)


def _format_val_metadata(val: object, extract_fn: object) -> str | None:
    """Format node val metadata for human-readable diagnostics."""
    if val is None:
        return None
    if isinstance(val, torch.Tensor):
        return str(extract_fn(val))  # type: ignore[operator]
    if isinstance(val, (tuple, list)):
        parts = []
        for v in val:
            if isinstance(v, torch.Tensor):
````
- **EN**: Introduces function `_format_val_metadata`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `parts`.
- **CN**: 这里定义了函数`_format_val_metadata`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `parts` 等值。

### Lines 85-98 / 第 85-98 行
````python
                parts.append(str(extract_fn(v)))  # type: ignore[operator]
            else:
                parts.append(str(type(v).__name__))
        return f"({', '.join(parts)})"
    return str(type(val).__name__)


def spmd_check(gm: torch.fx.GraphModule) -> bool:
    """Verify all ranks have identical FX graph structure (SPMD).

    Computes a structural hash (op targets + tensor metadata including
    shapes, dtypes, strides) and compares across ranks.
    On mismatch, emits a diagnostic report to stdout, logging, and
    trace_structured.
````
- **EN**: Introduces function `spmd_check`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`.
- **CN**: 这里定义了函数`spmd_check`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。

### Lines 99-112 / 第 99-112 行
````python

    Returns True if graphs match (SPMD), False on mismatch.
    """
    import torch.distributed as dist

    if not dist.is_initialized() or dist.get_world_size() <= 1:
        return True

    structure_hash = _compute_hash(gm)
    if structure_hash is None:
        return True

    from torch._subclasses.fake_tensor import unset_fake_temporarily
    from torch.distributed.distributed_c10d import _get_default_group
````
- **EN**: Imports dependencies such as `torch.distributed`, `torch._subclasses.fake_tensor`, and `torch.distributed.distributed_c10d` for the logic in this range. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `structure_hash`.
- **CN**: 这里导入了 `torch.distributed`、`torch._subclasses.fake_tensor`、`torch.distributed.distributed_c10d` 等依赖，为后续逻辑提供基础能力。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `structure_hash` 等值。

### Lines 113-126 / 第 113-126 行
````python

    pg = _get_default_group()
    world_size = dist.get_world_size()
    rank = dist.get_rank()

    with unset_fake_temporarily():
        all_hashes: list[int] = [0] * world_size
        dist.all_gather_object(all_hashes, structure_hash, pg)

    if all(h == all_hashes[0] for h in all_hashes):
        return True

    # Mismatch detected — build and gather diagnostic fingerprints
    fingerprint = _build_diag_fingerprint(gm)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `pg`, `world_size`, `rank`, `all_hashes`, and `fingerprint`. This range continues the implementation of function `spmd_check`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `pg`、`world_size`、`rank`、`all_hashes`、`fingerprint` 等值。这一段延续了函数`spmd_check` 的具体实现。

### Lines 127-140 / 第 127-140 行
````python
    with unset_fake_temporarily():
        all_fingerprints: list[tuple[object, ...]] = [() for _ in range(world_size)]
        dist.all_gather_object(all_fingerprints, fingerprint, pg)

    report = _build_mismatch_report(all_fingerprints, rank, world_size)

    print(report, flush=True)
    log.warning("\n%s", report)

    trace_structured(
        "artifact",
        metadata_fn=lambda: {
            "name": "inductor_spmd_graph_mismatch",
            "encoding": "string",
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `all_fingerprints`, `report`, and `metadata_fn`. This range continues the implementation of function `spmd_check`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `all_fingerprints`、`report`、`metadata_fn` 等值。这一段延续了函数`spmd_check` 的具体实现。

### Lines 141-154 / 第 141-154 行
````python
        },
        payload_fn=lambda: report,
    )

    if config.aten_distributed_optimizations.spmd_mismatch == "error":
        raise RuntimeError(
            "SPMD graph verification failed. "
            'Set aten_distributed_optimizations.spmd_mismatch="warn" '
            "to warn instead of fail.\n" + report
        )

    return False


````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `payload_fn`. This range continues the implementation of function `spmd_check`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `payload_fn` 等值。这一段延续了函数`spmd_check` 的具体实现。

### Lines 155-168 / 第 155-168 行
````python
def _entry_target(entry: object) -> str:
    """Extract the target string from a fingerprint entry."""
    if isinstance(entry, tuple):
        return str(entry[0])
    return str(entry)


def _entry_metadata(entry: object) -> str:
    """Format metadata from a fingerprint entry, if present."""
    if isinstance(entry, tuple) and len(entry) >= 2:
        meta = entry[1]
        if meta is not None:
            return f" meta={meta}"
    return ""
````
- **EN**: Introduces function `_entry_target`, function `_entry_metadata`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `meta`.
- **CN**: 这里定义了函数`_entry_target`、函数`_entry_metadata`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `meta` 等值。

### Lines 169-182 / 第 169-182 行
````python


def _build_mismatch_report(
    all_fingerprints: list[tuple[object, ...]],
    rank: int,
    world_size: int,
) -> str:
    """Build diagnostic report for SPMD graph mismatch."""
    lines = [
        "=" * 80,
        f"SPMD GRAPH MISMATCH — rank {rank}, world_size={world_size}",
        "=" * 80,
    ]

````
- **EN**: Introduces function `_build_mismatch_report`. Initializes or updates values such as `all_fingerprints`, `rank`, `world_size`, and `lines`.
- **CN**: 这里定义了函数`_build_mismatch_report`。初始化或更新了 `all_fingerprints`、`rank`、`world_size`、`lines` 等值。

### Lines 183-196 / 第 183-196 行
````python
    # Node count per rank
    counts = [len(t) for t in all_fingerprints]
    lines.append("NODE COUNTS PER RANK:")
    for r in range(world_size):
        marker = " <--" if counts[r] != counts[0] else ""
        lines.append(f"  rank {r}: {counts[r]} call_function nodes{marker}")
    lines.append("")

    # Find entries that differ
    ref = all_fingerprints[0]
    for r in range(1, world_size):
        other = all_fingerprints[r]
        if other == ref:
            continue
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `counts`, `marker`, `ref`, and `other`. This range continues the implementation of function `_build_mismatch_report`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `counts`、`marker`、`ref`、`other` 等值。这一段延续了函数`_build_mismatch_report` 的具体实现。

### Lines 197-210 / 第 197-210 行
````python
        lines.append(f"DIFFS rank 0 vs rank {r}:")

        # Show first few positional differences
        max_diffs = 10
        shown = 0
        for i, (a, b) in enumerate(zip(ref, other)):
            if a != b and shown < max_diffs:
                lines.append(f"  node {i}:")
                lines.append(f"    rank 0: {_entry_target(a)}{_entry_metadata(a)}")
                lines.append(f"    rank {r}: {_entry_target(b)}{_entry_metadata(b)}")
                shown += 1

        # Also show count-based diffs for op targets
        ref_targets = [_entry_target(e) for e in ref]
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `max_diffs`, `shown`, and `ref_targets`. This range continues the implementation of function `_build_mismatch_report`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `max_diffs`、`shown`、`ref_targets` 等值。这一段延续了函数`_build_mismatch_report` 的具体实现。

### Lines 211-224 / 第 211-224 行
````python
        other_targets = [_entry_target(e) for e in other]

        ref_counts = Counter(ref_targets)
        other_counts = Counter(other_targets)
        only_ref = ref_counts - other_counts
        only_other = other_counts - ref_counts
        if only_ref:
            lines.append("  Only on rank 0:")
            for op, cnt in only_ref.most_common(10):
                lines.append(f"    {op} (x{cnt})")
        if only_other:
            lines.append(f"  Only on rank {r}:")
            for op, cnt in only_other.most_common(10):
                lines.append(f"    {op} (x{cnt})")
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `other_targets`, `ref_counts`, `other_counts`, `only_ref`, and `only_other`. This range continues the implementation of function `_build_mismatch_report`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `other_targets`、`ref_counts`、`other_counts`、`only_ref`、`only_other` 等值。这一段延续了函数`_build_mismatch_report` 的具体实现。

### Lines 225-228 / 第 225-228 行
````python
        lines.append("")

    lines.append("=" * 80)
    return "\n".join(lines)
````
- **EN**: Includes returns or checks that define the contract of this code path. This range continues the implementation of function `_build_mismatch_report`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。这一段延续了函数`_build_mismatch_report` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary functions: `_compute_hash`, `_build_diag_fingerprint`, `_format_val_metadata`, `spmd_check`, `_entry_target`, `_entry_metadata`, and `...+1`  
  **CN**: 主要函数：`_compute_hash`、`_build_diag_fingerprint`、`_format_val_metadata`、`spmd_check`、`_entry_target`、`_entry_metadata`、`另有1项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `hashlib`, `logging`, `collections`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor`, `torch._logging`, `torch._inductor.codecache`, `torch.distributed`, `torch._subclasses.fake_tensor`, `torch.distributed.distributed_c10d`
