# tp_mapping.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/kv_transfer/kv_connector/v1/nixl/tp_mapping.py`
- **Repository**: vllm-project/vllm
- **Purpose**: TP mapping computation for NIXL KV cache transfers / 实现 v1 KV 传输连接器、接口或辅助模块。

## Line-by-Line Analysis / 逐行分析
### Module overview / 模块概览
```python
"""TP mapping computation for NIXL KV cache transfers."""
```
**EN:** The opening docstring defines the module scope and high-level contract. It highlights: TP mapping computation for NIXL KV cache transfers.
**CN:** 开头的文档字符串给出了模块范围与高层契约。 它重点概括了文件的职责、参与方以及主要接口。

### Imports and setup / 导入与初始化
```python
from __future__ import annotations

from dataclasses import dataclass

import numpy as np

from vllm.distributed.kv_transfer.kv_connector.utils import (
    BlockIds,
    TransferTopology,
)
from vllm.v1.kv_cache_interface import AttentionSpec, KVCacheSpec, MambaSpec
```
**EN:** This block imports `__future__`, `dataclasses`, `numpy`, `vllm.distributed.kv_transfer.kv_connector.utils`, `vllm.v1.kv_cache_interface` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `__future__`, `dataclasses`, `numpy`, `vllm.distributed.kv_transfer.kv_connector.utils`, `vllm.v1.kv_cache_interface`，为后续实现准备运行时、类型与辅助 API。

### Class `ReadSpec` / 类 `ReadSpec`
```python
@dataclass(frozen=True)
class ReadSpec:
    """Specification for a single remote block read operation."""

    remote_rank: int
    local_block_ids: BlockIds
    remote_block_ids: BlockIds
```
**EN:** Declares `ReadSpec`, a class. The docstring summarizes its role as: Specification for a single remote block read operation.
**CN:** 声明 `ReadSpec`，它是一个类。 文档字符串概括了它在整体流程中的职责。

### Function `_is_attention_spec` / 函数 `_is_attention_spec`
```python
def _is_attention_spec(spec_type: type[KVCacheSpec]) -> bool:
    return issubclass(spec_type, AttentionSpec)
```
**EN:** `_is_attention_spec` is a thin wrapper around `issubclass`, exposing that operation through a module-level helper. It primarily works with arguments like `spec_type`. Key calls include `issubclass`.
**CN:** `_is_attention_spec` 是对 `issubclass` 的轻量封装，通过模块级辅助函数暴露该操作。 它主要处理诸如 `spec_type` 这样的参数。 关键调用包括 `issubclass`。

### Function `_is_ssm_spec` / 函数 `_is_ssm_spec`
```python
def _is_ssm_spec(spec_type: type[KVCacheSpec]) -> bool:
    return issubclass(spec_type, MambaSpec)
```
**EN:** `_is_ssm_spec` is a thin wrapper around `issubclass`, exposing that operation through a module-level helper. It primarily works with arguments like `spec_type`. Key calls include `issubclass`.
**CN:** `_is_ssm_spec` 是对 `issubclass` 的轻量封装，通过模块级辅助函数暴露该操作。 它主要处理诸如 `spec_type` 这样的参数。 关键调用包括 `issubclass`。

### Class `TPMapping` / 类 `TPMapping`
```python
@dataclass(frozen=True)
class TPMapping:
    """Complete local-to-remote TP mapping for one remote engine.

    Generated once per remote engine during handshake.
    """

    # Remote TP ranks that this local rank reads from, per group.
    # Position = local piece index.
    source_ranks_per_group: tuple[tuple[int, ...], ...]

    # Superset of all source ranks (union of all groups).
    all_source_ranks: tuple[int, ...]

    # Maps each source rank to its FA head slot index.
    rank_to_attention_slot: dict[int, int]

    # FA head offset factor for hetero-TP (D_TP > P_TP).
    rank_offset_factor: int
```
**EN:** Declares `TPMapping`, a class. The docstring summarizes its role as: Complete local-to-remote TP mapping for one remote engine.
**CN:** 声明 `TPMapping`，它是一个类。 文档字符串概括了它在整体流程中的职责。

### Function `compute_tp_mapping` / 函数 `compute_tp_mapping`
```python
def compute_tp_mapping(
    transfer_topology: TransferTopology,
    remote_tp_size: int,
    group_spec_types: tuple[type[KVCacheSpec], ...],
) -> TPMapping:
    """Build the complete local-to-remote TP mapping.

    Computes source ranks, head slot assignments, and the rank offset
    factor in a single pass.
    """
    tp_rank = transfer_topology.tp_rank
    tp_size = transfer_topology.tp_size
    total_num_kv_heads = transfer_topology.total_num_kv_heads
    # --- Attention source ranks ---
    if transfer_topology.is_mla or tp_size >= remote_tp_size:
        # D (local TP) > P (remote TP): multiple local ranks read different chunks from
        # *one* remote rank, corresponding to different kv heads.
        # For MLA, we only need one remote since cache is duplicated. When P TP=k*TP k,
        # this will spread mla ranks to read from remote k*tp_rank.
        attn_ranks = [tp_rank * remote_tp_size // tp_size]
    else:
        # P (remote TP) > D (local TP): one local rank
        # reads from multiple remote ranks.
        # GQA dedup: when K < remote_tp_size, several remote ranks
        # hold the same KV head.  np.unique keeps only the first
        # rank per unique head so we don't issue redundant reads.
        abs_tp = remote_tp_size // tp_size
        start = tp_rank * abs_tp
        heads = np.arange(start, start + abs_tp) * total_num_kv_heads // remote_tp_size
        _, unique_idx = np.unique(heads, return_index=True)
        attn_ranks = (start + np.sort(unique_idx)).tolist()

    # --- SSM source ranks ---
    has_ssm = any(_is_ssm_spec(t) for t in group_spec_types)
    if has_ssm:
        if tp_size < remote_tp_size:
            abs_tp = remote_tp_size // tp_size
            ssm_ranks = list(range(tp_rank * abs_tp, (tp_rank + 1) * abs_tp))
        else:
            ssm_ranks = list(attn_ranks)
    else:
        ssm_ranks = []

    all_ranks = sorted(set(attn_ranks) | set(ssm_ranks))
# ... truncated for analysis ...
    else:
        # D TP > P TP: we index into remote to read different heads depending on rank.
        rank_offset_factor = tp_rank % (tp_size // remote_tp_size)

    return TPMapping(
        source_ranks_per_group=source_ranks_per_group,
        all_source_ranks=tuple(all_ranks),
        rank_to_attention_slot=rank_to_attention_slot,
        rank_offset_factor=rank_offset_factor,
    )
```
**EN:** `compute_tp_mapping` implements a focused helper routine for this module. The docstring frames it as: Build the complete local-to-remote TP mapping. It primarily works with arguments like `transfer_topology`, `remote_tp_size`, `group_spec_types`. Key calls include `any`, `sorted`, `tuple`.
**CN:** `compute_tp_mapping` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `transfer_topology`, `remote_tp_size`, `group_spec_types` 这样的参数。 关键调用包括 `any`, `sorted`, `tuple`。

## Key Concepts / 关键概念
- KV cache movement across workers / 跨工作进程的 KV 缓存迁移
- `ReadSpec`: class interface or data carrier / `ReadSpec`：类接口或数据载体
- `_is_attention_spec`: module-level helper or API entry / `_is_attention_spec`：模块级辅助函数或 API 入口
- `_is_ssm_spec`: module-level helper or API entry / `_is_ssm_spec`：模块级辅助函数或 API 入口
- `TPMapping`: class interface or data carrier / `TPMapping`：类接口或数据载体
- `compute_tp_mapping`: module-level helper or API entry / `compute_tp_mapping`：模块级辅助函数或 API 入口

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `dataclasses`
- **Third-party / 第三方**: `numpy`
- **Internal modules / 内部模块**: `vllm.distributed.kv_transfer.kv_connector.utils`, `vllm.v1.kv_cache_interface`
