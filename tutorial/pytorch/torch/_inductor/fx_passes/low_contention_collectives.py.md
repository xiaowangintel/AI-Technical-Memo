# low_contention_collectives.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/low_contention_collectives.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It exposes functions such as `_get_collective_info`, `replace_collectives_with_low_contention`, `_enable_symm_mem`, `_replace_collective`, `_get_per_rank_bytes`, `_has_compute_bound_overlap`, and `...+3`.
- **用途（中文）**: 该模块实现 FX 图变换 pass。同时提供 `_get_collective_info`、`replace_collectives_with_low_contention`、`_enable_symm_mem`、`_replace_collective`、`_get_per_rank_bytes`、`_has_compute_bound_overlap`、`另有3项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
from __future__ import annotations

import logging
import warnings

import torch
from torch.utils._ordered_set import OrderedSet


log = logging.getLogger(__name__)


def _get_collective_info(node):
    """Return (is_ag, group_name) if node is an AG/RS collective, else None."""
    from torch._inductor.fx_passes.bucketing import (
        is_all_gather_into_tensor,
        is_reduce_scatter_tensor,
    )
    from torch._inductor.fx_passes.overlap_scheduling import get_group_name

````
- **EN**: Imports dependencies such as `__future__`, `logging`, `warnings`, `torch`, `torch.utils._ordered_set`, `torch._inductor.fx_passes.bucketing`, and `...+1` for the logic in this range. Introduces function `_get_collective_info`. Initializes or updates values such as `log`.
- **CN**: 这里导入了 `__future__`、`logging`、`warnings`、`torch`、`torch.utils._ordered_set`、`torch._inductor.fx_passes.bucketing`、`另有1项` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_get_collective_info`。初始化或更新了 `log` 等值。

### Lines 21-40 / 第 21-40 行
````python
    if is_all_gather_into_tensor(node):
        return True, get_group_name(node)
    if is_reduce_scatter_tensor(node):
        return False, get_group_name(node)
    return None


def replace_collectives_with_low_contention(
    graph: torch.fx.Graph,
) -> None:
    """Replace FSDP collectives with copy-engine symm_mem variants."""
    symm_mem = torch.ops.symm_mem

    collectives = []
    groups: OrderedSet[str] = OrderedSet()
    for node in list(graph.nodes):
        info = _get_collective_info(node)
        if info is None:
            continue
        is_ag, group_name = info
````
- **EN**: Introduces function `replace_collectives_with_low_contention`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `graph`, `symm_mem`, `collectives`, `groups`, and `info`.
- **CN**: 这里定义了函数`replace_collectives_with_low_contention`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `graph`、`symm_mem`、`collectives`、`groups`、`info` 等值。

### Lines 41-60 / 第 41-60 行
````python
        collectives.append((node, is_ag, group_name))
        groups.add(group_name)

    if not collectives:
        return

    # Some group names can't be resolved at compile time — skip them.
    valid_groups: OrderedSet[str] = OrderedSet()
    for group_name in groups:
        if _enable_symm_mem(group_name):
            valid_groups.add(group_name)

    # Filter to collectives whose groups we can actually resolve
    collectives = [
        (node, is_ag, gn) for node, is_ag, gn in collectives if gn in valid_groups
    ]
    if not collectives:
        return

    from torch._inductor import config
````
- **EN**: Imports dependencies such as `torch._inductor` for the logic in this range. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `valid_groups`, and `collectives`.
- **CN**: 这里导入了 `torch._inductor` 等依赖，为后续逻辑提供基础能力。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `valid_groups`、`collectives` 等值。

### Lines 61-80 / 第 61-80 行
````python

    min_bytes = config.aten_distributed_optimizations.low_contention_min_bytes_per_rank

    node_positions = {n: i for i, n in enumerate(graph.nodes)}

    replacements = 0
    skipped_small = 0
    skipped_no_overlap = 0
    skipped_nvlink_contention = 0
    for node, is_ag, group_name in collectives:
        coll_type = "AG" if is_ag else "RS"

        # Size filter: LC barrier overhead dominates for small messages
        if min_bytes > 0:
            per_rank_bytes = _get_per_rank_bytes(node, is_ag)
            if per_rank_bytes is not None and per_rank_bytes < min_bytes:
                skipped_small += 1
                log.debug(
                    "LC skip %s %s: size %d < min_bytes %d",
                    coll_type,
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `min_bytes`, `node_positions`, `replacements`, `skipped_small`, `skipped_no_overlap`, `skipped_nvlink_contention`, and `...+2`. This range continues the implementation of function `replace_collectives_with_low_contention`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `min_bytes`、`node_positions`、`replacements`、`skipped_small`、`skipped_no_overlap`、`skipped_nvlink_contention`、`另有2项` 等值。这一段延续了函数`replace_collectives_with_low_contention` 的具体实现。

### Lines 81-100 / 第 81-100 行
````python
                    node.name,
                    per_rank_bytes,
                    min_bytes,
                )
                continue

        # Skip collectives with no compute to hide behind
        if not _has_compute_bound_overlap(node, graph, node_positions):
            skipped_no_overlap += 1
            log.debug("LC skip %s %s: no compute-bound overlap", coll_type, node.name)
            continue

        # Skip if other groups' NCCL collectives overlap on NVLink
        if _has_other_group_collectives(node, group_name, graph, node_positions):
            skipped_nvlink_contention += 1
            log.debug(
                "LC skip %s %s: overlaps other-group collectives (NVLink contention)",
                coll_type,
                node.name,
            )
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `replace_collectives_with_low_contention`.
- **CN**: 包含分支、循环或上下文管理等控制流。这一段延续了函数`replace_collectives_with_low_contention` 的具体实现。

### Lines 101-120 / 第 101-120 行
````python
            continue

        _replace_collective(node, graph, symm_mem, is_ag, group_name)
        replacements += 1

    log.info(
        "Replaced %d/%d FSDP collectives "
        "(skipped_small=%d, skipped_no_overlap=%d, "
        "skipped_nvlink_contention=%d, min_bytes=%d)",
        replacements,
        len(collectives),
        skipped_small,
        skipped_no_overlap,
        skipped_nvlink_contention,
        min_bytes,
    )


def _enable_symm_mem(group_name):
    """Try to enable symmetric memory for a group. Returns True on success."""
````
- **EN**: Introduces function `_enable_symm_mem`.
- **CN**: 这里定义了函数`_enable_symm_mem`。

### Lines 121-140 / 第 121-140 行
````python
    from torch.distributed._symmetric_memory import (
        enable_symm_mem_for_group,
        is_symm_mem_enabled_for_group,
    )

    if is_symm_mem_enabled_for_group(group_name):
        return True
    try:
        with warnings.catch_warnings():
            warnings.simplefilter("ignore", FutureWarning)
            enable_symm_mem_for_group(group_name)
        return True
    except (TypeError, RuntimeError, KeyError) as e:
        log.debug("LC: cannot enable symm_mem for group %s: %s", group_name, e)
        return False


def _replace_collective(node, graph, symm_mem, is_ag, group_name):
    input_node = node.args[0]
    if is_ag:
````
- **EN**: Imports dependencies such as `torch.distributed._symmetric_memory` for the logic in this range. Introduces function `_replace_collective`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch.distributed._symmetric_memory` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_replace_collective`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 141-160 / 第 141-160 行
````python
        target = symm_mem._low_contention_all_gather.default
        args = (input_node, group_name)
    else:
        reduce_op = node.args[1]
        target = symm_mem._low_contention_reduce_scatter.default
        args = (input_node, reduce_op, group_name)

    with graph.inserting_before(node):
        new_node = graph.call_function(target, args=args)
    new_node.meta.update(node.meta)
    node.replace_all_uses_with(new_node)
    graph.erase_node(node)


def _get_per_rank_bytes(node, is_ag):
    """Return per-rank message bytes for a collective, or None if unknown."""
    input_val = node.args[0].meta.get("val") if node.args else None
    if not isinstance(input_val, torch.Tensor):
        return None
    total_bytes = input_val.nelement() * input_val.element_size()
````
- **EN**: Introduces function `_get_per_rank_bytes`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `target`, `args`, `else`, `reduce_op`, `new_node`, `input_val`, and `...+1`.
- **CN**: 这里定义了函数`_get_per_rank_bytes`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `target`、`args`、`else`、`reduce_op`、`new_node`、`input_val`、`另有1项` 等值。

### Lines 161-180 / 第 161-180 行
````python
    if is_ag:
        return total_bytes
    # For RS, input is the full tensor; per-rank = total / group_size
    group_size = node.args[2] if len(node.args) > 2 else None
    if not isinstance(group_size, int) or group_size <= 0:
        return None
    return total_bytes // group_size


def _has_compute_bound_overlap(start_node, graph, node_positions):
    """Check if compute-bound ops exist between collective start and wait."""
    from torch._inductor.fx_passes.overlap_scheduling import is_compute_node

    wait_node = _find_wait_for_collective(start_node)
    if wait_node is None:
        return False

    start_pos = node_positions[start_node]
    wait_pos = node_positions[wait_node]

````
- **EN**: Imports dependencies such as `torch._inductor.fx_passes.overlap_scheduling` for the logic in this range. Introduces function `_has_compute_bound_overlap`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch._inductor.fx_passes.overlap_scheduling` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_has_compute_bound_overlap`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 181-200 / 第 181-200 行
````python
    for node in graph.nodes:
        pos = node_positions[node]
        if pos <= start_pos or pos >= wait_pos:
            continue
        if is_compute_node(node):
            return True
    return False


def _has_other_group_collectives(start_node, group_name, graph, node_positions):
    """Check if other groups' collectives overlap, competing for NVLink."""
    wait_node = _find_wait_for_collective(start_node)
    if wait_node is None:
        return False

    start_pos = node_positions[start_node]
    wait_pos = node_positions[wait_node]

    for node in graph.nodes:
        pos = node_positions[node]
````
- **EN**: Introduces function `_has_other_group_collectives`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `pos`, `wait_node`, `start_pos`, and `wait_pos`.
- **CN**: 这里定义了函数`_has_other_group_collectives`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `pos`、`wait_node`、`start_pos`、`wait_pos` 等值。

### Lines 201-220 / 第 201-220 行
````python
        if pos <= start_pos or pos >= wait_pos:
            continue
        info = _get_collective_info(node)
        if info is not None:
            _, other_group = info
            if other_group != group_name:
                log.debug(
                    "LC contention %s: found %s (group %s) between start/wait",
                    start_node.name,
                    node.name,
                    other_group,
                )
                return True
    return False


def _is_wait_tensor(node):
    """Check if node is a wait_tensor op (direct or wrapped in ControlDeps)."""
    if node.op != "call_function":
        return False
````
- **EN**: Introduces function `_is_wait_tensor`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `info`.
- **CN**: 这里定义了函数`_is_wait_tensor`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `info` 等值。

### Lines 221-240 / 第 221-240 行
````python
    if node.target is torch.ops._c10d_functional.wait_tensor.default:
        return True
    # Handles public namespace (c10d_functional.wait_tensor) and
    # ControlDeps-wrapped wait_tensor (from TBB manual scheduling)
    return "wait_tensor" in node.name


def _find_wait_for_collective(start_node):
    """Find the wait_tensor node for a collective.

    Handles multiple graph patterns:
    1. Direct: start -> wait_tensor(start)
    2. _out variant: start(out=buf) -> wait_tensor(buf)
    3. ControlDeps-wrapped: start -> control_deps(wait_tensor_subgraph, start)
    """
    for user in start_node.users:
        if _is_wait_tensor(user):
            return user

    # For _out variants, check users of the out-buffer keyword argument.
````
- **EN**: Introduces function `_find_wait_for_collective`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_find_wait_for_collective`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 241-252 / 第 241-252 行
````python
    c10d = torch.ops._c10d_functional
    if start_node.target in (
        c10d.all_gather_into_tensor_out.default,
        c10d.reduce_scatter_tensor_out.default,
    ):
        out_buf = start_node.kwargs.get("out")
        if isinstance(out_buf, torch.fx.Node):
            for user in out_buf.users:
                if _is_wait_tensor(user):
                    return user

    return None
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `c10d`, and `out_buf`. This range continues the implementation of function `_find_wait_for_collective`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `c10d`、`out_buf` 等值。这一段延续了函数`_find_wait_for_collective` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Primary functions: `_get_collective_info`, `replace_collectives_with_low_contention`, `_enable_symm_mem`, `_replace_collective`, `_get_per_rank_bytes`, `_has_compute_bound_overlap`, and `...+3`  
  **CN**: 主要函数：`_get_collective_info`、`replace_collectives_with_low_contention`、`_enable_symm_mem`、`_replace_collective`、`_get_per_rank_bytes`、`_has_compute_bound_overlap`、`另有3项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `warnings`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.utils._ordered_set`, `torch._inductor.fx_passes.bucketing`, `torch._inductor.fx_passes.overlap_scheduling`, `torch._inductor`, `torch.distributed._symmetric_memory`
