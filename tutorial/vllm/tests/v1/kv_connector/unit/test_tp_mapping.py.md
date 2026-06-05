# test_tp_mapping.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/unit/test_tp_mapping.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Tests for TP mapping and transfer plan utilities. / 该文件的文档字符串表明其用途：`tests for tp mapping and transfer plan utilities`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (lines 3-8)
```python
"""Tests for TP mapping and transfer plan utilities.

These tests verify that TP mapping produces correct outputs
(source ranks, split handles, desc IDs).
No GPU or NIXL required.
"""
```
**EN:** Module docstring that declares the scope of the file: Tests for TP mapping and transfer plan utilities.
**CN:** 模块文档字符串直接说明了文件范围：`tests for tp mapping and transfer plan utilities`。

### Imports and setup / 导入与设置 (lines 10-23)
```python
from __future__ import annotations

from types import SimpleNamespace

import pytest

from vllm.distributed.kv_transfer.kv_connector.v1.nixl.tp_mapping import (
    TPMapping,
    compute_tp_mapping,
)
from vllm.distributed.kv_transfer.kv_connector.v1.nixl.worker import (
    NixlConnectorWorker,
)
from vllm.v1.kv_cache_interface import FullAttentionSpec, MambaSpec
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest`. vLLM modules under test include `vllm.distributed.kv_transfer.kv_connector.v1.nixl.tp_mapping, vllm.distributed.kv_transfer.kv_connector.v1.nixl.worker, vllm.v1.kv_cache_interface`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest`。 被测试的 vLLM 模块包括 `vllm.distributed.kv_transfer.kv_connector.v1.nixl.tp_mapping, vllm.distributed.kv_transfer.kv_connector.v1.nixl.worker, vllm.v1.kv_cache_interface`。

### _compute_mapping (lines 30-48)
```python
def _compute_mapping(
    tp_rank: int = 0,
    tp_size: int = 1,
    remote_tp_size: int = 1,
    is_mla: bool = False,
    num_kv_heads: int = 8,
    group_spec_types: tuple[type, ...] = (FullAttentionSpec,),
) -> TPMapping:
    transfer_topology = SimpleNamespace(
        tp_rank=tp_rank,
        tp_size=tp_size,
        is_mla=is_mla,
        total_num_kv_heads=num_kv_heads,
    )
    return compute_tp_mapping(
        transfer_topology=transfer_topology,
        remote_tp_size=remote_tp_size,
        group_spec_types=group_spec_types,
    )
```
**EN:** Helper function `_compute_mapping` encapsulates reusable logic for `compute mapping`. Inputs: `tp_rank, tp_size, remote_tp_size, is_mla, num_kv_heads, group_spec_types`. Key calls include `SimpleNamespace, compute_tp_mapping`.
**CN:** 辅助函数 `_compute_mapping` 封装了与 `compute mapping` 相关的可复用逻辑。 输入参数：`tp_rank, tp_size, remote_tp_size, is_mla, num_kv_heads, group_spec_types`。 关键调用包括 `SimpleNamespace, compute_tp_mapping`。

### TestTPMappingStructure (lines 56-67)
```python
class TestTPMappingStructure:
    def test_source_ranks_homogeneous(self):
        m = _compute_mapping(tp_size=2, tp_rank=1, remote_tp_size=2)
        assert m.all_source_ranks == (1,)

    def test_source_ranks_d_gt_p(self):
        m = _compute_mapping(tp_size=4, tp_rank=2, remote_tp_size=2)
        assert m.all_source_ranks == (1,)

    def test_source_ranks_p_gt_d(self):
        m = _compute_mapping(tp_size=1, tp_rank=0, remote_tp_size=2)
        assert m.all_source_ranks == (0, 1)
```
**EN:** Class `TestTPMappingStructure` groups 3 test method(s). Representative scenarios: `test_source_ranks_homogeneous, test_source_ranks_d_gt_p, test_source_ranks_p_gt_d`.
**CN:** 类 `TestTPMappingStructure` 组织了 3 个测试方法。 代表性场景：`test_source_ranks_homogeneous, test_source_ranks_d_gt_p, test_source_ranks_p_gt_d`。

### _make_mock_worker_for_splits (lines 75-79)
```python
def _make_mock_worker_for_splits(group_spec_types):
    """Build a mock NixlConnectorWorker with _group_spec_types for split tests."""
    worker = object.__new__(NixlConnectorWorker)
    worker._group_spec_types = group_spec_types
    return worker
```
**EN:** Helper function `_make_mock_worker_for_splits` encapsulates reusable logic for `mock worker for splits`. Inputs: `group_spec_types`. Key calls include `object.__new__`.
**CN:** 辅助函数 `_make_mock_worker_for_splits` 封装了与 `mock worker for splits` 相关的可复用逻辑。 输入参数：`group_spec_types`。 关键调用包括 `object.__new__`。

### TestBuildSrcSplitHandles (lines 82-109)
```python
class TestBuildSrcSplitHandles:
    @pytest.mark.parametrize("remote_tp_size", [2, 4])
    def test_build_src_split_handles(self, remote_tp_size):
        tp_rank = 0
        tp_size = 1

        plan = _compute_mapping(
            tp_rank=tp_rank,
            tp_size=tp_size,
            remote_tp_size=remote_tp_size,
        )

        worker = _make_mock_worker_for_splits((FullAttentionSpec,))
        src_blocks_data = [(0x2000 + i * 1024, 1024, 0) for i in range(8)]
        num_descs = len(src_blocks_data)
        splits = list(
            worker._build_local_splits_from_plan(
                plan,
                src_blocks_data,
                num_descs,
            )
        )

        assert len(splits) == remote_tp_size
        for handle in splits:
            assert len(handle) == len(src_blocks_data)
            for _, length, _ in handle:
                assert length == 1024 // remote_tp_size
```
**EN:** Class `TestBuildSrcSplitHandles` groups 1 test method(s). Representative scenarios: `test_build_src_split_handles`.
**CN:** 类 `TestBuildSrcSplitHandles` 组织了 1 个测试方法。 代表性场景：`test_build_src_split_handles`。

### TestMambaPlanSplitHandles (lines 112-146)
```python
class TestMambaPlanSplitHandles:
    """Verify split handles for Mamba with FA/SSM distinction."""

    def test_fa_and_ssm_different_split_factors(self):
        """Section 0 split by num_attn_reads, section 1 by abs_tp."""
        fa_readers = (0,)
        ssm_readers = (0, 1)
        plan = TPMapping(
            source_ranks_per_group=(fa_readers, ssm_readers),
            all_source_ranks=(0, 1),
            rank_to_attention_slot={0: 0, 1: 0},
            rank_offset_factor=0,
        )
        worker = _make_mock_worker_for_splits((FullAttentionSpec, MambaSpec))
        # 2 FA descs + 1 SSM desc
        src_blocks_data = [
            (1000, 200, 0),  # FA desc 0
    # ... excerpt omitted for brevity ...
        assert len(splits) == 2  # 2 source ranks
        assert splits[0] == [(1000, 200, 0), (2000, 200, 0), (3000, 200, 0)]
        # Rank 1 (not FA source, p_idx=1):
        # FA: chunk=200//1=200, slot=0 (skip_fa) → (1000, 200, 0), (2000, 200, 0)
        # SSM: chunk=400//2=200, idx=1 → (3200, 200, 0)
        assert splits[1] == [(1000, 200, 0), (2000, 200, 0), (3200, 200, 0)]
```
**EN:** Class `TestMambaPlanSplitHandles` groups 1 test method(s). Representative scenarios: `test_fa_and_ssm_different_split_factors`.
**CN:** 类 `TestMambaPlanSplitHandles` 组织了 1 个测试方法。 代表性场景：`test_fa_and_ssm_different_split_factors`。

## Key Concepts / 关键概念
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为
- **EN:** Connector contracts and transfer paths
- **CN:** 连接器契约与传输路径
- **EN:** Worker-side execution and device coordination
- **CN:** worker 侧执行与设备协同

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest`.
- **CN:** 外部库：`pytest`。
- **EN:** vLLM modules under test: `vllm.distributed.kv_transfer.kv_connector.v1.nixl.tp_mapping, vllm.distributed.kv_transfer.kv_connector.v1.nixl.worker, vllm.v1.kv_cache_interface`.
- **CN:** 被测试的 vLLM 模块：`vllm.distributed.kv_transfer.kv_connector.v1.nixl.tp_mapping, vllm.distributed.kv_transfer.kv_connector.v1.nixl.worker, vllm.v1.kv_cache_interface`。
- **EN:** Standard-library support: `__future__, types`.
- **CN:** 标准库支持：`__future__, types`。
