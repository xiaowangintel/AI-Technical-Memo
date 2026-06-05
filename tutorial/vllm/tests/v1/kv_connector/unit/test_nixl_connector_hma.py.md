# test_nixl_connector_hma.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/unit/test_nixl_connector_hma.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Unit tests for NixlConnectorScheduler with HMA and Mamba N-1 prefill. / 该文件的文档字符串表明其用途：`unit tests for nixlconnectorscheduler with hma and mamba n-1 prefill`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (line 3)
```python
"""Unit tests for NixlConnectorScheduler with HMA and Mamba N-1 prefill."""
```
**EN:** Module docstring that declares the scope of the file: Unit tests for NixlConnectorScheduler with HMA and Mamba N-1 prefill.
**CN:** 模块文档字符串直接说明了文件范围：`unit tests for nixlconnectorscheduler with hma and mamba n-1 prefill`。

### Imports and setup / 导入与设置 (lines 5-23)
```python
import gc
from unittest.mock import patch

import pytest
import torch

from vllm import LLM, SamplingParams
from vllm.config import KVTransferConfig
from vllm.v1.core.single_type_kv_cache_manager import (
    FullAttentionManager,
    SlidingWindowManager,
)

from .utils import (
    create_request,
    create_vllm_config,
    make_kv_cache_config,
    make_nixl_scheduler,
)
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch`. vLLM modules under test include `vllm, vllm.config, vllm.v1.core.single_type_kv_cache_manager, vllm.distributed.kv_transfer.kv_connector.v1.nixl.scheduler, vllm.distributed.kv_transfer.kv_connector.v1.nixl.worker, ...`. Local helpers come from `tests.v1.kv_connector.unit.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch`。 被测试的 vLLM 模块包括 `vllm, vllm.config, vllm.v1.core.single_type_kv_cache_manager, vllm.distributed.kv_transfer.kv_connector.v1.nixl.scheduler, vllm.distributed.kv_transfer.kv_connector.v1.nixl.worker, ...`。 本地测试辅助逻辑来自 `tests.v1.kv_connector.unit.utils`。

### test_sw_sizes (lines 37-60)
```python
def test_sw_sizes(mock_platform, swa_enabled, expected_sw_sizes):
    """Test sw_sizes is correctly computed based on SWA enabled/disabled."""
    from vllm.distributed.kv_transfer.kv_connector.v1.nixl.scheduler import (
        NixlConnectorScheduler,
    )

    mock_platform.device_type = "cpu"

    block_size = 16
    vllm_config = create_vllm_config(block_size=block_size)
    # SW 2048 tokens=>128 blocks
    kv_cache_config = make_kv_cache_config(
        block_size=block_size, swa_enabled=swa_enabled, sw_size=2048
    )

    scheduler = NixlConnectorScheduler(
        vllm_config=vllm_config,
        engine_id="test-engine",
        kv_cache_config=kv_cache_config,
    )
    # in number of blocks
    assert scheduler.blocks_per_sw == expected_sw_sizes, (
        f"Expected sw_sizes={expected_sw_sizes}, got {scheduler.blocks_per_sw}"
    )
```
**EN:** Parameterized test covering `sw sizes`. Parameter axes: `swa_enabled, expected_sw_sizes`. Inputs/fixtures: `mock_platform, swa_enabled, expected_sw_sizes`. It exercises `mark.parametrize, patch, create_vllm_config, make_kv_cache_config, NixlConnectorScheduler`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `sw sizes` 的测试用例。 参数维度：`swa_enabled, expected_sw_sizes`。 输入或 fixture：`mock_platform, swa_enabled, expected_sw_sizes`。 该测试会调用 `mark.parametrize, patch, create_vllm_config, make_kv_cache_config, NixlConnectorScheduler`。 代码主体包含 1 个显式断言。

### test_logical_to_kernel_block_ids_with_hma (lines 64-91)
```python
def test_logical_to_kernel_block_ids_with_hma():
    """Test _logical_to_kernel_block_ids expands blocks when HMA is enabled.

    When HMA is enabled, the logical block size may differ from the kernel
    block size. Each logical block maps to multiple kernel blocks.
    """
    from vllm.distributed.kv_transfer.kv_connector.v1.nixl.worker import (
        NixlConnectorWorker,
    )

    # Create a mock worker with just the required attributes
    # (use __new__ to skip __init__)
    worker = object.__new__(NixlConnectorWorker)

    # Simulate HMA scenario: logical block size = 32, kernel block size = 16
    # So each logical block maps to 2 kernel blocks eg [0]->[0,1]
    worker._physical_blocks_per_logical_kv_block = 2
    # FA + SW groups (neither is MambaSpec, so both get expanded)
    worker.kv_cache_config = make_kv_cache_config(block_size=16, swa_enabled=True)

    # Test conversion: FA + SW group
    logical_block_ids = [[0, 1, 2], [3, 4]]
    kernel_block_ids = worker._logical_to_kernel_block_ids(logical_block_ids)

    expected_kernel_block_ids = [[0, 1, 2, 3, 4, 5], [6, 7, 8, 9]]
    assert kernel_block_ids == expected_kernel_block_ids, (
        f"Expected {expected_kernel_block_ids}, got {kernel_block_ids}"
    )
```
**EN:** Test case covering `logical to kernel block ids with hma`. It exercises `object.__new__, make_kv_cache_config, worker._logical_to_kernel_block_ids`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `logical to kernel block ids with hma` 的测试用例。 该测试会调用 `object.__new__, make_kv_cache_config, worker._logical_to_kernel_block_ids`。 代码主体包含 1 个显式断言。

### test_read_blocks_for_req_expands_remote_ids (lines 124-206)
```python
def test_read_blocks_for_req_expands_remote_ids(
    group_spec_types,
    remote_physical_per_logical,
    local_physical_per_logical,
    tp_ratio,
    remote_block_ids,
    expected_remote_block_ids,
):
    """_read_blocks_for_req must expand remote logical block IDs to kernel
    block IDs when kernel block size != logical block size.

    The hot path always calls _logical_to_remote_kernel_block_ids with
    remote_info.remote_physical_blocks_per_logical (model-agnostic).
    """
    from unittest.mock import MagicMock
    from vllm.distributed.kv_transfer.kv_connector.v1.nixl.metadata import (
        NixlConnectorMetadata,
    # ... excerpt omitted for brevity ...
    worker.transfer_topo = MagicMock()
    remote_info = MagicMock()
    mock_plan = MagicMock(spec=TPMapping)
    meta = metadata.reqs_to_recv["test-req"]
    worker._read_blocks_for_req("test-req", meta)
    assert meta.remote.block_ids == expected_remote_block_ids, (
        f"Expected {expected_remote_block_ids}, got {meta.remote.block_ids}"
    )
```
**EN:** Parameterized test covering `read blocks for req expands remote ids`. Parameter axes: `group_spec_types, remote_physical_per_logical, local_physical_per_logical, tp_ratio, remote_block_ids, expected_remote_block_ids`. Inputs/fixtures: `group_spec_types, remote_physical_per_logical, local_physical_per_logical, tp_ratio, remote_block_ids, expected_remote_block_ids`. It exercises `mark.parametrize, tuple, object.__new__, any, make_kv_cache_config, MagicMock`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `read blocks for req expands remote ids` 的测试用例。 参数维度：`group_spec_types, remote_physical_per_logical, local_physical_per_logical, tp_ratio, remote_block_ids, expected_remote_block_ids`。 输入或 fixture：`group_spec_types, remote_physical_per_logical, local_physical_per_logical, tp_ratio, remote_block_ids, expected_remote_block_ids`。 该测试会调用 `mark.parametrize, tuple, object.__new__, any, make_kv_cache_config, MagicMock`。 代码主体包含 1 个显式断言。

### test_apply_prefix_caching_mamba_hybrid (lines 243-274)
```python
def test_apply_prefix_caching_mamba_hybrid(
    local_physical_per_logical,
    remote_physical_per_logical,
    local_block_ids,
    remote_block_ids,
    expected_local,
    expected_remote,
):
    """_apply_prefix_caching front-trims FA groups to
    min(local, remote) for Mamba hybrid models with heterogeneous TP.
    """
    from vllm.distributed.kv_transfer.kv_connector.v1.nixl.worker import (
        NixlConnectorWorker,
    )
    from vllm.v1.kv_cache_interface import FullAttentionSpec, MambaSpec

    worker = object.__new__(NixlConnectorWorker)
    worker._has_mamba = True
    worker._physical_blocks_per_logical_kv_block = local_physical_per_logical
    worker._group_spec_types = (FullAttentionSpec, MambaSpec)
    worker.kv_cache_config = make_kv_cache_config(block_size=16, mamba_enabled=True)

    aligned_local, aligned_remote = worker._apply_prefix_caching(
        local_block_ids, remote_block_ids, remote_physical_per_logical
    )

    assert aligned_local == expected_local, (
        f"Expected local {expected_local}, got {aligned_local}"
    )
    assert aligned_remote == expected_remote, (
        f"Expected remote {expected_remote}, got {aligned_remote}"
    )
```
**EN:** Parameterized test covering `apply prefix cachingMamba hybrid`. Parameter axes: `local_physical_per_logical, remote_physical_per_logical, local_block_ids, remote_block_ids, expected_local, expected_remote`. Inputs/fixtures: `local_physical_per_logical, remote_physical_per_logical, local_block_ids, remote_block_ids, expected_local, expected_remote`. It exercises `mark.parametrize, object.__new__, make_kv_cache_config, worker._apply_prefix_caching, pytest.param, list`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `apply prefix cachingmamba hybrid` 的测试用例。 参数维度：`local_physical_per_logical, remote_physical_per_logical, local_block_ids, remote_block_ids, expected_local, expected_remote`。 输入或 fixture：`local_physical_per_logical, remote_physical_per_logical, local_block_ids, remote_block_ids, expected_local, expected_remote`。 该测试会调用 `mark.parametrize, object.__new__, make_kv_cache_config, worker._apply_prefix_caching, pytest.param, list`。 代码主体包含 2 个显式断言。

### test_mismatched_physical_per_logical_fails_with_prefix_caching (lines 319-368)
```python
def test_mismatched_physical_per_logical_fails_with_prefix_caching(
    local_physical_per_logical,
    remote_physical_per_logical,
    remote_fa_blocks,
    local_fa_blocks,
    ssm_blocks,
    correct_remote_fa,
    correct_local_fa,
):
    """Demonstrate that _apply_prefix_caching front-trims ([:N])
    in the Mamba hybrid path, which fails when prefix caching produces
    suffix-only local blocks.

    Prefix caching operates at logical block granularity. When a logical
    block is cached locally, the decode side only allocates kernel blocks
    for the uncached suffix. The front-trim pairs remote prefix blocks
    with local suffix slots — a silent data corruption.
    """
    # ... excerpt omitted for brevity ...
    assert (
    ), (
        f"Prefix caching with mismatched physical_per_logical should not "
        f"produce correct transfer ids: "
        f"remote={aligned_remote[0]}, local={aligned_local[0]}, "
        f"correct_remote={correct_remote_fa}, correct_local={correct_local_fa}"
    )
```
**EN:** Parameterized test covering `mismatched physical per logical fails with prefix caching`. Parameter axes: `local_physical_per_logical, remote_physical_per_logical, remote_fa_blocks, local_fa_blocks, ssm_blocks, correct_remote_fa, correct_local_fa`. Inputs/fixtures: `local_physical_per_logical, remote_physical_per_logical, remote_fa_blocks, local_fa_blocks, ssm_blocks, correct_remote_fa, correct_local_fa`. It exercises `mark.parametrize, object.__new__, make_kv_cache_config, tuple, worker._apply_prefix_caching, pytest.param`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `mismatched physical per logical fails with 前缀缓存` 的测试用例。 参数维度：`local_physical_per_logical, remote_physical_per_logical, remote_fa_blocks, local_fa_blocks, ssm_blocks, correct_remote_fa, correct_local_fa`。 输入或 fixture：`local_physical_per_logical, remote_physical_per_logical, remote_fa_blocks, local_fa_blocks, ssm_blocks, correct_remote_fa, correct_local_fa`。 该测试会调用 `mark.parametrize, object.__new__, make_kv_cache_config, tuple, worker._apply_prefix_caching, pytest.param`。 代码主体包含 1 个显式断言。

### test_fewer_blocks_with_hma (lines 372-445)
```python
def test_fewer_blocks_with_hma(monkeypatch, model_name, sw_size):
    """Test that a prefill instance returns fewer "remote blocks" for the SWA groups
    when sequence exceeds the sliding window.
    """
    kv_transfer_config = KVTransferConfig(
        kv_connector="NixlConnector",
        kv_role="kv_both",
    )
    block_size = 16
    llm_kwargs = {
        "model": model_name,
        "enforce_eager": True,
        "gpu_memory_utilization": 0.3,
        "kv_transfer_config": kv_transfer_config,
        "max_model_len": 2048,
        "max_num_seqs": 1,
        # NOTE: Make sure HMA is enabled
        "disable_hybrid_kv_cache_manager": False,
    # ... excerpt omitted for brevity ...
        assert len(kv_managers) > 2
            assert isinstance(kv_manager, (SlidingWindowManager, FullAttentionManager))
        assert len(req_to_blocks) == 0
        expected_num_remote_blocks = sw_size // block_size + 1
        assert (
            == expected_num_remote_blocks
        try:
            run_hma_test(llm)
        finally:
            llm.llm_engine.engine_core.shutdown()

    run_test_and_cleanup()
```
**EN:** Parameterized test covering `fewer blocks with hma`. Parameter axes: `model_name, sw_size`. Inputs/fixtures: `monkeypatch, model_name, sw_size`. It exercises `mark.parametrize, KVTransferConfig, monkeypatch.setenv, run_test_and_cleanup, SamplingParams, llm.generate`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `fewer blocks with hma` 的测试用例。 参数维度：`model_name, sw_size`。 输入或 fixture：`monkeypatch, model_name, sw_size`。 该测试会调用 `mark.parametrize, KVTransferConfig, monkeypatch.setenv, run_test_and_cleanup, SamplingParams, llm.generate`。 代码主体包含 5 个显式断言。

### test_nixl_metadata_hma_block_ids_structure (lines 449-489)
```python
def test_nixl_metadata_hma_block_ids_structure():
    """
    Test that NixlConnectorMetadata correctly stores block IDs for multiple
    KV cache groups when HMA is enabled.
    from vllm.distributed.kv_transfer.kv_connector.v1.nixl.metadata import (
        NixlConnectorMetadata,
    )

    metadata = NixlConnectorMetadata()
    # Add request with block IDs for 2 groups (FA + SW)
    fa_blocks = [0, 1, 2, 3, 4, 5, 6, 7]  # 8 blocks for FA
    sw_blocks = [8, 9, 10, 11]  # 4 blocks for SW (clipped)
    metadata.add_new_req_to_recv(
        request_id="test-req-hma",
        local_block_ids=(fa_blocks, sw_blocks),
    # ... excerpt omitted for brevity ...
    assert "test-req-hma" in metadata.reqs_to_recv
    assert len(req_meta.local_block_ids) == 2
    assert list(req_meta.local_block_ids[0]) == fa_blocks
    assert list(req_meta.local_block_ids[1]) == sw_blocks
    # Verify remote block IDs structure
    assert req_meta.remote is not None
    assert len(req_meta.remote.block_ids) == 2
    assert list(req_meta.remote.block_ids[0]) == [10, 11, 12, 13, 14, 15, 16, 17]
    assert list(req_meta.remote.block_ids[1]) == [18, 19, 20, 21]
```
**EN:** Test case covering `NIXL metadata hma block ids structure`. It exercises `NixlConnectorMetadata, metadata.add_new_req_to_recv, len, list`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `nixl metadata hma block ids structure` 的测试用例。 该测试会调用 `NixlConnectorMetadata, metadata.add_new_req_to_recv, len, list`。 代码主体包含 8 个显式断言。

### _make_mock_worker_for_desc_ids (lines 492-513)
```python
def _make_mock_worker_for_desc_ids(
    num_regions: int,
    has_mamba: bool,
    group_spec_types: tuple,
    block_len_per_layer: list[int] | None = None,
):
    """Build a mock NixlConnectorWorker with attrs needed by _compute_desc_ids."""
    from unittest.mock import MagicMock

    from vllm.distributed.kv_transfer.kv_connector.v1.nixl.worker import (
        NixlConnectorWorker,
    )

    worker = MagicMock(spec=NixlConnectorWorker)
    worker.num_regions = num_regions
    worker._has_mamba = has_mamba
    worker._group_spec_types = group_spec_types
    worker.block_len_per_layer = block_len_per_layer or [100]
    worker._compute_desc_ids = NixlConnectorWorker._compute_desc_ids.__get__(
        worker, NixlConnectorWorker
    )
    return worker
```
**EN:** Helper function `_make_mock_worker_for_desc_ids` encapsulates reusable logic for `mock worker for desc ids`. Inputs: `num_regions, has_mamba, group_spec_types, block_len_per_layer`. Key calls include `MagicMock, _compute_desc_ids.__get__`.
**CN:** 辅助函数 `_make_mock_worker_for_desc_ids` 封装了与 `mock worker for desc ids` 相关的可复用逻辑。 输入参数：`num_regions, has_mamba, group_spec_types, block_len_per_layer`。 关键调用包括 `MagicMock, _compute_desc_ids.__get__`。

### test_get_block_descs_ids_hybrid_ssm (lines 517-539)
```python
def test_get_block_descs_ids_hybrid_ssm():
    """Test _compute_desc_ids uses per-group strides for hybrid
    FA+SSM when ratio=1 (no kernel block size mismatch)."""
    from vllm.v1.kv_cache_interface import FullAttentionSpec, MambaSpec

    worker = _make_mock_worker_for_desc_ids(
        num_regions=2,
        has_mamba=True,
        group_spec_types=(FullAttentionSpec, MambaSpec),
        block_len_per_layer=[100],
    )

    fa_blocks = [3, 5]
    ssm_blocks = [1, 2]
    result = worker._compute_desc_ids(
        block_ids=(fa_blocks, ssm_blocks),
        dst_num_blocks=100,
        block_size_ratio=None,
        physical_blocks_per_logical=1,
    )

    expected = [3, 5, 103, 105, 201, 202, 301, 302, 401, 402, 501, 502]
    assert list(result) == expected, f"Expected {expected}, got {list(result)}"
```
**EN:** Test case covering `get block descs ids hybrid ssm`. It exercises `_make_mock_worker_for_desc_ids, worker._compute_desc_ids, list`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `get block descs ids hybrid ssm` 的测试用例。 该测试会调用 `_make_mock_worker_for_desc_ids, worker._compute_desc_ids, list`。 代码主体包含 1 个显式断言。

### test_get_block_descs_ids_kernel_block_mismatch (lines 543-569)
```python
def test_get_block_descs_ids_kernel_block_mismatch():
    """Test _compute_desc_ids uses different strides for FA
    (kernel blocks) vs SSM (logical blocks) when ratio > 1."""
    from vllm.v1.kv_cache_interface import FullAttentionSpec, MambaSpec

    ratio = 4
    logical_blocks = 100
    num_blocks = logical_blocks * ratio  # 400 kernel blocks

    worker = _make_mock_worker_for_desc_ids(
        num_regions=2,
        has_mamba=True,
        group_spec_types=(FullAttentionSpec, MambaSpec),
        block_len_per_layer=[100],
    )

    fa_blocks = [3, 7]
    ssm_blocks = [1, 2]
    result = worker._compute_desc_ids(
        block_ids=(fa_blocks, ssm_blocks),
        dst_num_blocks=num_blocks,
        block_size_ratio=None,
        physical_blocks_per_logical=ratio,
    )

    expected = [3, 7, 403, 407, 801, 802, 901, 902, 1001, 1002, 1101, 1102]
    assert list(result) == expected, f"Expected {expected}, got {list(result)}"
```
**EN:** Test case covering `get block descs ids kernel block mismatch`. It exercises `_make_mock_worker_for_desc_ids, worker._compute_desc_ids, list`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `get block descs ids kernel block mismatch` 的测试用例。 该测试会调用 `_make_mock_worker_for_desc_ids, worker._compute_desc_ids, list`。 代码主体包含 1 个显式断言。

### test_nixl_metadata_hybrid_ssm_block_ids (lines 573-613)
```python
def test_nixl_metadata_hybrid_ssm_block_ids():
    """Test NixlConnectorMetadata correctly stores block IDs for FA + SSM
    groups with different block counts (kernel mismatch active)."""
    from vllm.distributed.kv_transfer.kv_connector.v1.nixl.metadata import (
        NixlConnectorMetadata,
    )

    metadata = NixlConnectorMetadata()
    # FA: 8 kernel blocks (2 logical * ratio=4), SSM: 2 logical blocks
    fa_blocks = [0, 1, 2, 3, 4, 5, 6, 7]
    ssm_blocks = [0, 1]
    metadata.add_new_req_to_recv(
        request_id="test-req-hybrid",
        local_block_ids=(fa_blocks, ssm_blocks),
        kv_transfer_params={
            "remote_block_ids": ([10, 11, 12, 13, 14, 15, 16, 17], [20, 21]),
    # ... excerpt omitted for brevity ...
    assert "test-req-hybrid" in metadata.reqs_to_recv
    assert len(req_meta.local_block_ids) == 2
    assert list(req_meta.local_block_ids[0]) == fa_blocks
    assert list(req_meta.local_block_ids[1]) == ssm_blocks
    assert len(req_meta.local_block_ids[0]) != len(req_meta.local_block_ids[1])
    # Verify remote block IDs: same asymmetry preserved
    assert req_meta.remote is not None
    assert len(req_meta.remote.block_ids) == 2
    assert list(req_meta.remote.block_ids[0]) == [10, 11, 12, 13, 14, 15, 16, 17]
    assert list(req_meta.remote.block_ids[1]) == [20, 21]
    assert len(req_meta.remote.block_ids[0]) != len(req_meta.remote.block_ids[1])
```
**EN:** Test case covering `NIXL metadata hybrid ssm block ids`. It exercises `NixlConnectorMetadata, metadata.add_new_req_to_recv, len, list`. The body contains 10 explicit assertion(s).
**CN:** 该代码块是覆盖 `nixl metadata hybrid ssm block ids` 的测试用例。 该测试会调用 `NixlConnectorMetadata, metadata.add_new_req_to_recv, len, list`。 代码主体包含 10 个显式断言。

### test_mamba_n1_d_side (lines 629-636)
```python
def test_mamba_n1_d_side(has_mamba, is_hma_required, expected_count):
    """D-side: Mamba gets N-1 matched tokens, non-Mamba gets N."""
    sched = make_nixl_scheduler(has_mamba=has_mamba, is_hma_required=is_hma_required)
    req = create_request(num_tokens=10, do_remote_prefill=True)

    count, is_async = sched.get_num_new_matched_tokens(req, num_computed_tokens=0)
    assert count == expected_count
    assert is_async is True
```
**EN:** Parameterized test covering `Mamba n1 d side`. Parameter axes: `has_mamba, is_hma_required, expected_count`. Inputs/fixtures: `has_mamba, is_hma_required, expected_count`. It exercises `mark.parametrize, make_nixl_scheduler, create_request, sched.get_num_new_matched_tokens`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `mamba n1 d side` 的测试用例。 参数维度：`has_mamba, is_hma_required, expected_count`。 输入或 fixture：`has_mamba, is_hma_required, expected_count`。 该测试会调用 `mark.parametrize, make_nixl_scheduler, create_request, sched.get_num_new_matched_tokens`。 代码主体包含 2 个显式断言。

### test_mamba_n1_p_side_truncation (lines 640-671)
```python
def test_mamba_n1_p_side_truncation():
    """P-side: Mamba truncates prompt to N-1, sets max_tokens=1.

    Also verifies idempotency (calling again is a no-op) which is
    needed for preemption safety via the _p_side_truncated guard,
    and that non-Mamba models skip truncation entirely.
    """
    sched = make_nixl_scheduler(has_mamba=True, is_hma_required=True)
    req = create_request(num_tokens=10, do_remote_decode=True)
    req.max_tokens = 128
    original_len = len(req.prompt_token_ids)

    count, is_async = sched.get_num_new_matched_tokens(req, num_computed_tokens=0)

    assert count == 0
    assert is_async is False
    assert len(req.prompt_token_ids) == original_len - 1
    assert req.num_prompt_tokens == original_len - 1
    assert req.max_tokens == 1
    assert req.kv_transfer_params["_p_side_truncated"] is True

    # Idempotency: second call must not truncate further
    sched.get_num_new_matched_tokens(req, num_computed_tokens=0)
    assert len(req.prompt_token_ids) == original_len - 1

    # Non-Mamba: truncation is skipped
    fa_sched = make_nixl_scheduler(has_mamba=False, is_hma_required=False)
    fa_req = create_request(num_tokens=10, do_remote_decode=True)
    fa_original = len(fa_req.prompt_token_ids)

    fa_sched.get_num_new_matched_tokens(fa_req, num_computed_tokens=0)
    assert len(fa_req.prompt_token_ids) == fa_original
```
**EN:** Test case covering `Mamba n1 p side truncation`. It exercises `make_nixl_scheduler, create_request, len, sched.get_num_new_matched_tokens, fa_sched.get_num_new_matched_tokens`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `mamba n1 p side truncation` 的测试用例。 该测试会调用 `make_nixl_scheduler, create_request, len, sched.get_num_new_matched_tokens, fa_sched.get_num_new_matched_tokens`。 代码主体包含 8 个显式断言。

### test_has_mamba_init (lines 685-716)
```python
def test_has_mamba_init(
    mock_platform,
    swa_enabled,
    mamba_enabled,
    expected_has_mamba,
    expected_is_hma,
):
    """Test _has_mamba / _is_hma_required derived from kv_cache_groups."""
    from vllm.distributed.kv_transfer.kv_connector.v1.nixl.scheduler import (
        NixlConnectorScheduler,
    )

    mock_platform.device_type = "cpu"

    block_size = 16
    vllm_config = create_vllm_config(block_size=block_size)
    # VllmConfig.__post_init__ auto-disables HMA when kv_transfer_config
    # is set; override so we can test the scheduler's own derivation.
    vllm_config.scheduler_config.disable_hybrid_kv_cache_manager = False
    kv_cache_config = make_kv_cache_config(
        block_size=block_size,
        swa_enabled=swa_enabled,
        mamba_enabled=mamba_enabled,
    )

    scheduler = NixlConnectorScheduler(
        vllm_config=vllm_config,
        engine_id="test-engine",
        kv_cache_config=kv_cache_config,
    )
    assert scheduler._has_mamba is expected_has_mamba
    assert scheduler._is_hma_required is expected_is_hma
```
**EN:** Parameterized test covering `hasMamba init`. Parameter axes: `swa_enabled, mamba_enabled, expected_has_mamba, expected_is_hma`. Inputs/fixtures: `mock_platform, swa_enabled, mamba_enabled, expected_has_mamba, expected_is_hma`. It exercises `mark.parametrize, patch, create_vllm_config, make_kv_cache_config, NixlConnectorScheduler`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `hasmamba init` 的测试用例。 参数维度：`swa_enabled, mamba_enabled, expected_has_mamba, expected_is_hma`。 输入或 fixture：`mock_platform, swa_enabled, mamba_enabled, expected_has_mamba, expected_is_hma`。 该测试会调用 `mark.parametrize, patch, create_vllm_config, make_kv_cache_config, NixlConnectorScheduler`。 代码主体包含 2 个显式断言。

### test_compute_physical_blocks_per_logical (lines 731-742)
```python
def test_compute_physical_blocks_per_logical(ssm_sizes, block_len, expected_ratio):
    """Verify that compute_physical_blocks_per_logical is TP-dependent.

    With dimension-sharded Mamba state, the ratio differs across TP sizes
    (e.g. TP=1 → 261, TP=4 → 131 for Nemotron 30B). This is why
    _physical_blocks_per_logical must be stored per-engine.
    """
    from vllm.distributed.kv_transfer.kv_connector.v1.ssm_conv_transfer_utils import (
        compute_physical_blocks_per_logical,
    )

    assert compute_physical_blocks_per_logical(ssm_sizes, block_len) == expected_ratio
```
**EN:** Parameterized test covering `compute physical blocks per logical`. Parameter axes: `ssm_sizes, block_len, expected_ratio`. Inputs/fixtures: `ssm_sizes, block_len, expected_ratio`. It exercises `mark.parametrize, compute_physical_blocks_per_logical`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `compute physical blocks per logical` 的测试用例。 参数维度：`ssm_sizes, block_len, expected_ratio`。 输入或 fixture：`ssm_sizes, block_len, expected_ratio`。 该测试会调用 `mark.parametrize, compute_physical_blocks_per_logical`。 代码主体包含 1 个显式断言。

### test_derive_mamba_conv_split (lines 833-868)
```python
def test_derive_mamba_conv_split(
    monkeypatch,
    mamba_type,
    local_tp,
    conv_dim_local,
    conv_rows,
    temporal_shape,
    expected_proj_dims,
):
    """Parametrized test for derive_mamba_conv_split with real model configs.

    Values generated by verify_conv_split.py which loads HuggingFace configs
    and calls vLLM's derive_mamba_conv_split directly.
    """
    from vllm.distributed.kv_transfer.kv_connector.v1.ssm_conv_transfer_utils import (
        derive_mamba_conv_split,
    )
    from vllm.v1.attention.backends.registry import MambaAttentionBackendEnum
    # ... excerpt omitted for brevity ...
        dtypes=(torch.bfloat16, torch.bfloat16),
        mamba_type=mamba_type_enum,
    out = derive_mamba_conv_split(spec, local_tp=local_tp)
    assert out.local_proj_dims == expected_proj_dims
    assert out.conv_rows == conv_rows
```
**EN:** Parameterized test covering `deriveMamba conv split`. Parameter axes: `mamba_type, local_tp, conv_dim_local, conv_rows, temporal_shape, expected_proj_dims`. Inputs/fixtures: `monkeypatch, mamba_type, local_tp, conv_dim_local, conv_rows, temporal_shape, expected_proj_dims`. It exercises `mark.parametrize, monkeypatch.setenv, MambaSpec, derive_mamba_conv_split, pytest.param`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `derivemamba conv split` 的测试用例。 参数维度：`mamba_type, local_tp, conv_dim_local, conv_rows, temporal_shape, expected_proj_dims`。 输入或 fixture：`monkeypatch, mamba_type, local_tp, conv_dim_local, conv_rows, temporal_shape, expected_proj_dims`。 该测试会调用 `mark.parametrize, monkeypatch.setenv, MambaSpec, derive_mamba_conv_split, pytest.param`。 代码主体包含 2 个显式断言。

### test_logical_to_remote_kernel_block_ids (lines 949-984)
```python
def test_logical_to_remote_kernel_block_ids(
    mamba_enabled,
    swa_enabled,
    local_physical_per_logical,
    remote_physical_per_logical,
    logical_block_ids,
    expected_kernel_block_ids,
):
    """Verify _logical_to_remote_kernel_block_ids uses the remote
    physical_per_logical for FA expansion, not the local one.

    This was the root cause of silent accuracy corruption in Qwen3.5
    heterogeneous TP (e.g. 4P2D): the old code used local physical_per_logical
    for the expansion arange, producing wrong kernel block indices.
    Qwen3.5-0.8B values verified by verify_conv_split.py (issue #13).
    """
    from vllm.distributed.kv_transfer.kv_connector.v1.nixl.worker import (
    # ... excerpt omitted for brevity ...
        logical_block_ids,
        remote_physical_per_logical,
    )
    assert list(result) == expected_kernel_block_ids, (
        f"Expected {expected_kernel_block_ids}, got {result}"
```
**EN:** Parameterized test covering `logical to remote kernel block ids`. Parameter axes: `mamba_enabled, swa_enabled, local_physical_per_logical, remote_physical_per_logical, logical_block_ids, expected_kernel_block_ids`. Inputs/fixtures: `mamba_enabled, swa_enabled, local_physical_per_logical, remote_physical_per_logical, logical_block_ids, expected_kernel_block_ids`. It exercises `mark.parametrize, object.__new__, make_kv_cache_config, worker._logical_to_remote_kernel_block_ids, list, pytest.param`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `logical to remote kernel block ids` 的测试用例。 参数维度：`mamba_enabled, swa_enabled, local_physical_per_logical, remote_physical_per_logical, logical_block_ids, expected_kernel_block_ids`。 输入或 fixture：`mamba_enabled, swa_enabled, local_physical_per_logical, remote_physical_per_logical, logical_block_ids, expected_kernel_block_ids`。 该测试会调用 `mark.parametrize, object.__new__, make_kv_cache_config, worker._logical_to_remote_kernel_block_ids, list, pytest.param`。 代码主体包含 1 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Attention backend and metadata coverage
- **CN:** 注意力后端与元数据覆盖
- **EN:** Scheduler state transitions and queue management
- **CN:** 调度器状态迁移与队列管理

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch`.
- **CN:** 外部库：`pytest, torch`。
- **EN:** vLLM modules under test: `vllm, vllm.config, vllm.v1.core.single_type_kv_cache_manager, vllm.distributed.kv_transfer.kv_connector.v1.nixl.scheduler, vllm.distributed.kv_transfer.kv_connector.v1.nixl.worker, vllm.distributed.kv_transfer.kv_connector.v1.nixl.metadata, vllm.distributed.kv_transfer.kv_connector.v1.nixl.tp_mapping, vllm.v1.kv_cache_interface, ...`.
- **CN:** 被测试的 vLLM 模块：`vllm, vllm.config, vllm.v1.core.single_type_kv_cache_manager, vllm.distributed.kv_transfer.kv_connector.v1.nixl.scheduler, vllm.distributed.kv_transfer.kv_connector.v1.nixl.worker, vllm.distributed.kv_transfer.kv_connector.v1.nixl.metadata, vllm.distributed.kv_transfer.kv_connector.v1.nixl.tp_mapping, vllm.v1.kv_cache_interface, ...`。
- **EN:** Local test helpers: `tests.v1.kv_connector.unit.utils`.
- **CN:** 本地测试辅助模块：`tests.v1.kv_connector.unit.utils`。
- **EN:** Standard-library support: `gc, unittest.mock`.
- **CN:** 标准库支持：`gc, unittest.mock`。
