# parallel_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/parallel_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Support module for kernel tests in kernels / moe / parallel_utils; it provides reference logic, fixtures, or utilities reused by multiple scenarios. / kernels / moe / parallel_utils 对应的内核测试支撑模块；它提供会被多个场景复用的参考逻辑、fixture 或工具函数。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-5)
```python
"""
DeepEP test utilities
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 7-19)
```python
import dataclasses
import os
import traceback
from collections.abc import Callable
from typing import Concatenate

import torch
from torch.distributed import ProcessGroup
from torch.multiprocessing import spawn  # pyright: ignore[reportPrivateImportUsage]
from typing_extensions import ParamSpec

from vllm.utils.import_utils import has_deep_ep
from vllm.utils.network_utils import get_open_port
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as dataclasses, os, traceback, collections.abc; and vLLM components like vllm.utils.import_utils, vllm.utils.network_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 dataclasses、os、traceback、collections.abc；vLLM 内部组件，例如 vllm.utils.import_utils、vllm.utils.network_utils。

### Top-level block starting at line 21 (lines 21-27)
```python
if has_deep_ep():
    from vllm.model_executor.layers.fused_moe.prepare_finalize.deepep_ht import (
        DeepEPHTPrepareAndFinalize,
    )
    from vllm.model_executor.layers.fused_moe.prepare_finalize.deepep_ll import (
        DeepEPLLPrepareAndFinalize,
    )
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Constants and module state (lines 31-31)
```python
P = ParamSpec("P")
```
**EN:** This block centralizes shared constants and parameter grids, including P. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 P。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Class `ProcessGroupInfo` (lines 34-41)
```python
@dataclasses.dataclass
class ProcessGroupInfo:
    world_size: int
    world_local_size: int
    rank: int
    node_rank: int
    local_rank: int
    device: torch.device
```
**EN:** This dataclass packages the fields needed to describe ProcessGroupInfo. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 ProcessGroupInfo 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Function `_worker_parallel_launch` (lines 44-85)
```python
def _worker_parallel_launch(
    local_rank: int,
    world_size: int,
    world_local_size: int,
    node_rank: int,
    init_method: str,
    worker: Callable[Concatenate[ProcessGroupInfo, P], None],
    *args: P.args,
    **kwargs: P.kwargs,
) -> None:
    rank = node_rank * world_local_size + local_rank
    torch.accelerator.set_device_index(local_rank)
    device = torch.device("cuda", local_rank)
    torch.distributed.init_process_group(
        backend="cpu:gloo,cuda:nccl",
        init_method=init_method,
        rank=rank,
        world_size=world_size,
        device_id=device,
    )
    barrier = torch.tensor([rank], device=device)
    torch.distributed.all_reduce(barrier)

    try:
        worker(
            ProcessGroupInfo(
                world_size=world_size,
                world_local_size=world_local_size,
                rank=rank,
                node_rank=node_rank,
                local_rank=local_rank,
                device=device,
            ),
            *args,
            **kwargs,
        )
    except Exception as ex:
        print(ex)
        traceback.print_exc()
        raise
    finally:
        torch.distributed.destroy_process_group()
```
**EN:** This helper function implements the shared logic for worker parallel launch. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 worker parallel launch 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `parallel_launch` (lines 88-107)
```python
def parallel_launch(
    world_size: int,
    worker: Callable[Concatenate[ProcessGroupInfo, P], None],
    *args: P.args,
    **kwargs: P.kwargs,
) -> None:
    assert not kwargs
    spawn(
        _worker_parallel_launch,
        args=(
            world_size,
            world_size,
            0,
            f"tcp://{os.getenv('LOCALHOST', 'localhost')}:{get_open_port()}",
            worker,
        )
        + args,
        nprocs=world_size,
        join=True,
    )
```
**EN:** This helper function implements the shared logic for parallel launch. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 parallel launch 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Class `DeepEPHTArgs` (lines 113-115)
```python
@dataclasses.dataclass
class DeepEPHTArgs:
    num_local_experts: int
```
**EN:** This dataclass packages the fields needed to describe DeepEPHTArgs. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 DeepEPHTArgs 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Class `DeepEPLLArgs` (lines 118-123)
```python
@dataclasses.dataclass
class DeepEPLLArgs:
    max_tokens_per_rank: int
    hidden_size: int
    num_experts: int
    use_fp8_dispatch: bool
```
**EN:** This dataclass packages the fields needed to describe DeepEPLLArgs. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 DeepEPLLArgs 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Function `make_deepep_ht_a2a` (lines 126-151)
```python
def make_deepep_ht_a2a(
    pg: ProcessGroup,
    pgi: ProcessGroupInfo,
    dp_size: int,
    ht_args: DeepEPHTArgs,
    q_dtype: torch.dtype | None = None,
    block_shape: list[int] | None = None,
):
    import deep_ep

    # high throughput a2a
    num_nvl_bytes = 1024 * 1024 * 1024  # 1GB
    num_rdma_bytes, low_latency_mode, num_qps_per_rank = 0, False, 1
    buffer = deep_ep.Buffer(
        group=pg,
        num_nvl_bytes=num_nvl_bytes,
        num_rdma_bytes=num_rdma_bytes,
        low_latency_mode=low_latency_mode,
        num_qps_per_rank=num_qps_per_rank,
    )
    return DeepEPHTPrepareAndFinalize(
        buffer=buffer,
        num_dispatchers=pgi.world_size,
        dp_size=dp_size,
        rank_expert_offset=pgi.rank * ht_args.num_local_experts,
    )
```
**EN:** This helper function implements the shared logic for deepep ht a2a. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 deepep ht a2a 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `make_deepep_ll_a2a` (lines 154-183)
```python
def make_deepep_ll_a2a(
    pg: ProcessGroup,
    pgi: ProcessGroupInfo,
    deepep_ll_args: DeepEPLLArgs,
    q_dtype: torch.dtype | None = None,
    block_shape: list[int] | None = None,
):
    import deep_ep

    # low-latency a2a
    num_rdma_bytes = deep_ep.Buffer.get_low_latency_rdma_size_hint(
        deepep_ll_args.max_tokens_per_rank,
        deepep_ll_args.hidden_size,
        pgi.world_size,
        deepep_ll_args.num_experts,
    )

    buffer = deep_ep.Buffer(
        group=pg,
        num_rdma_bytes=num_rdma_bytes,
        low_latency_mode=True,
        num_qps_per_rank=deepep_ll_args.num_experts // pgi.world_size,
    )

    return DeepEPLLPrepareAndFinalize(
        buffer=buffer,
        num_dispatchers=pgi.world_size,
        max_tokens_per_rank=deepep_ll_args.max_tokens_per_rank,
        use_fp8_dispatch=deepep_ll_args.use_fp8_dispatch,
    )
```
**EN:** This helper function implements the shared logic for deepep ll a2a. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 deepep ll a2a 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `make_deepep_a2a` (lines 186-202)
```python
def make_deepep_a2a(
    pg: ProcessGroup,
    pgi: ProcessGroupInfo,
    dp_size: int,
    deepep_ht_args: DeepEPHTArgs | None,
    deepep_ll_args: DeepEPLLArgs | None,
    q_dtype: torch.dtype | None = None,
    block_shape: list[int] | None = None,
):
    if deepep_ht_args is not None:
        assert deepep_ll_args is None
        return make_deepep_ht_a2a(
            pg, pgi, dp_size, deepep_ht_args, q_dtype, block_shape
        )

    assert deepep_ll_args is not None
    return make_deepep_ll_a2a(pg, pgi, deepep_ll_args, q_dtype, block_shape)
```
**EN:** This helper function implements the shared logic for deepep a2a. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 deepep a2a 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

## Key Concepts / 关键概念
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。
- **Shared test utilities / 共享测试工具:** The module factors repeated setup or reference logic out of the individual test files. / 该模块把重复的初始化或参考逻辑从单个测试文件中抽离出来复用。

## Dependencies / 依赖关系
- `dataclasses`
- `os`
- `traceback`
- `collections.abc -> Callable`
- `typing -> Concatenate`
- `torch`
- `torch.distributed -> ProcessGroup`
- `torch.multiprocessing -> spawn`
- `typing_extensions -> ParamSpec`
- `vllm.utils.import_utils -> has_deep_ep`
- `vllm.utils.network_utils -> get_open_port`
