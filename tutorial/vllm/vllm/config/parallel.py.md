# parallel.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/config/parallel.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements parallel support for the `config` portion of vLLM. / 为 vLLM 的 `config` 子目录实现与 并行 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-19)
```python
import os

import socket

from collections.abc import Callable

from typing import TYPE_CHECKING, Any, Literal, overload

import regex as re

import torch

from pydantic import Field, field_validator, model_validator

from torch.distributed import ProcessGroup, ReduceOp, Store

from typing_extensions import Self

import vllm.envs as envs

from vllm.config.utils import config

from vllm.logger import init_logger

from vllm.platforms import current_platform

from vllm.utils.network_utils import get_open_ports_list
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 31-51)
```python
logger = init_logger(__name__)

_NUMACTL_CPUSET_PATTERN = re.compile(r"^\d+(?:-\d+)?(?:,\d+(?:-\d+)?)*$")

ExpertPlacementStrategy = Literal["linear", "round_robin"]

DistributedExecutorBackend = Literal["ray", "mp", "uni", "external_launcher"]

DataParallelBackend = Literal["ray", "mp"]

EPLBPolicyOption = Literal["default"]

DCPCommBackend = Literal["ag_rs", "a2a"]

EPLBCommunicatorBackend = Literal["torch_nccl", "torch_gloo", "nixl", "pynccl"]

All2AllBackend = Literal[
    "naive",
    "pplx",
    "deepep_high_throughput",
    "deepep_low_latency",
    "mori",
    "nixl_ep",
    "allgather_reducescatter",
    "flashinfer_all2allv",  # temporary alias for flashinfer_nvlink_two_sided
    "flashinfer_nvlink_two_sided",
    "flashinfer_nvlink_one_sided",
]
# ... omitted for brevity ...
```
**EN:** This constant/configuration block defines `logger`, `_NUMACTL_CPUSET_PATTERN`, `ExpertPlacementStrategy`, `DistributedExecutorBackend`, `DataParallelBackend`, `EPLBPolicyOption`, `DCPCommBackend`, `EPLBCommunicatorBackend`, `All2AllBackend`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `logger`, `_NUMACTL_CPUSET_PATTERN`, `ExpertPlacementStrategy`, `DistributedExecutorBackend`, `DataParallelBackend`, `EPLBPolicyOption`, `DCPCommBackend`, `EPLBCommunicatorBackend`, `All2AllBackend`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Class `EPLBConfig` (lines 55-104)
```python
class EPLBConfig:
    """Configuration for Expert Parallel Load Balancing (EP)."""

    window_size: int = Field(default=1000, gt=0)
    """Window size for expert load recording."""
    step_interval: int = Field(default=3000, gt=0)
    """
    Interval for rearranging experts in expert parallelism.

    Note that if this is greater than the EPLB window size, only the metrics
    of the last `lb_window_size` steps will be used for rearranging experts.
    """

    num_redundant_experts: int = Field(default=0, ge=0)
    """Number of redundant experts to use for expert parallelism."""

    log_balancedness: bool = False
    """
    Log the balancedness each step of expert parallelism.
    This is turned off by default since it will cause communication overhead.
    """
    log_balancedness_interval: int = Field(default=1, gt=0)
    """
    # ... omitted for brevity ...
            raise ValueError("log_balancedness_interval must be greater than 0.")
        return self
```
**EN:** Class `EPLBConfig` is a structured building block in this module. Key methods include `_validate_eplb_config`, which define initialization, validation, transformation, or access patterns. The class docstring says: Configuration for Expert Parallel Load Balancing (EP).
**CN:** 类 `EPLBConfig` 是该模块中的结构化构件。 关键方法包括 `_validate_eplb_config`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Configuration for Expert Parallel Load Balancing (EP).

### Method `EPLBConfig._validate_eplb_config` (lines 99-104)
```python
    def _validate_eplb_config(self) -> Self:
        if self.use_async and self.policy != "default":
            raise ValueError("Async EPLB is only supported with the default policy.")
        if self.log_balancedness and self.log_balancedness_interval <= 0:
            raise ValueError("log_balancedness_interval must be greater than 0.")
        return self
```
**EN:** Method `EPLBConfig._validate_eplb_config` parses configuration, arguments, or structured metadata. Key calls such as `ValueError`, `model_validator` show the concrete execution path.
**CN:** Method `EPLBConfig._validate_eplb_config` 负责解析配置、参数或结构化元数据。 像 `ValueError`, `model_validator` 这样的关键调用展示了该代码块的具体执行路径。

### Class `ParallelConfig` (lines 108-957)
```python
class ParallelConfig:
    """Configuration for the distributed execution."""

    pipeline_parallel_size: int = 1
    """Number of pipeline parallel groups."""
    tensor_parallel_size: int = 1
    """Number of tensor parallel groups."""
    prefill_context_parallel_size: int = 1
    """Number of prefill context parallel groups."""
    data_parallel_size: int = 1
    """Number of data parallel groups. MoE layers will be sharded according to
    the product of the tensor parallel size and data parallel size."""
    data_parallel_size_local: int = 1
    """Number of local data parallel groups."""
    data_parallel_rank: int = 0
    """Rank of the data parallel group."""
    data_parallel_rank_local: int | None = None
    """Local rank of the data parallel group,
    set only in SPMD mode."""
    data_parallel_master_ip: str = "127.0.0.1"
    """IP of the data parallel master."""
    data_parallel_rpc_port: int = 29550
    """Port for data parallel messaging."""
    # ... omitted for brevity ...

        return self
```
**EN:** Class `ParallelConfig` is a structured building block in this module. Key methods include `_skip_none_validation`, `_validate_numa_bind_nodes`, `_validate_numa_bind_cpus`, `_validate_parallel_config`, `world_size_across_dp`, `use_ubatching`, which define initialization, validation, transformation, or access patterns. The class docstring says: Configuration for the distributed execution.
**CN:** 类 `ParallelConfig` 是该模块中的结构化构件。 关键方法包括 `_skip_none_validation`, `_validate_numa_bind_nodes`, `_validate_numa_bind_cpus`, `_validate_parallel_config`, `world_size_across_dp`, `use_ubatching`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Configuration for the distributed execution.

### Method `ParallelConfig.get_next_dp_init_port` (lines 509-524)
```python
    def get_next_dp_init_port(self) -> int:
        """
        We might need to initialize process groups in multiple
        processes that is related to data parallelism,
        e.g. both in the worker and in the engine, which
        can live in different processes. To avoid port conflicts, we
        pop a new port from the prepared port list each time we need to
        initialize a new process group related to data parallelism.
        """
        if self._data_parallel_master_port_list:
            answer = self._data_parallel_master_port_list.pop()
        else:
            answer = self.data_parallel_master_port
            self.data_parallel_master_port += 1

        return answer
```
**EN:** Method `ParallelConfig.get_next_dp_init_port` constructs derived objects, runtime state, or helper structures. The docstring highlights: We might need to initialize process groups in multiple processes that is related to data parallelism, e.g. Key calls such as `self._data_parallel_master_port_list.pop` show the concrete execution path.
**CN:** Method `ParallelConfig.get_next_dp_init_port` 负责构造派生对象、运行时状态或辅助结构。 文档字符串强调：We might need to initialize process groups in multiple processes that is related to data parallelism, e.g. 像 `self._data_parallel_master_port_list.pop` 这样的关键调用展示了该代码块的具体执行路径。

### Method `ParallelConfig.stateless_init_dp_group` (lines 561-602)
```python
    def stateless_init_dp_group(
        self, return_store: bool = False
    ) -> ProcessGroup | tuple[ProcessGroup, Store]:
        # NOTE: In high-concurrency scenarios multiple processes
        # can pick the same (currently free) port through a race
        # condition when calling `get_open_port()`. When the first
        # process binds the port the others will subsequently fail
        # with `torch.distributed.DistNetworkError: EADDRINUSE`.
        # To make the initialization more robust we retry a few times
        # with a fresh port whenever this specific error is observed.
        from torch.distributed import DistNetworkError

        from vllm.distributed.utils import (
            stateless_init_torch_distributed_process_group,
        )

        max_retries = 5
        last_exc: Exception | None = None
        for _ in range(max_retries):
    # ... omitted for brevity ...
        assert last_exc is not None
        raise last_exc
```
**EN:** Method `ParallelConfig.stateless_init_dp_group` constructs derived objects, runtime state, or helper structures. Key calls such as `range`, `self._pick_stateless_dp_port`, `stateless_init_torch_distributed_process_group`, `str`, `logger.warning` show the concrete execution path.
**CN:** Method `ParallelConfig.stateless_init_dp_group` 负责构造派生对象、运行时状态或辅助结构。 像 `range`, `self._pick_stateless_dp_port`, `stateless_init_torch_distributed_process_group`, `str`, `logger.warning` 这样的关键调用展示了该代码块的具体执行路径。

### Method `ParallelConfig.use_sequence_parallel_moe` (lines 613-626)
```python
    def use_sequence_parallel_moe(self) -> bool:
        return (
            self.all2all_backend
            in (
                "allgather_reducescatter",
                "deepep_high_throughput",
                "deepep_low_latency",
                "mori",
                "nixl_ep",
            )
            and self.enable_expert_parallel
            and self.tensor_parallel_size > 1
            and self.data_parallel_size > 1
        )
```
**EN:** Method `ParallelConfig.use_sequence_parallel_moe` exposes a computed property so callers can access derived state without duplicating logic.
**CN:** Method `ParallelConfig.use_sequence_parallel_moe` 暴露计算属性，使调用方无需重复逻辑即可访问派生状态。

### Method `ParallelConfig.sync_dp_state` (lines 669-693)
```python
    def sync_dp_state(
        dp_group: ProcessGroup, has_unfinished: bool, pending_pause: bool
    ) -> tuple[bool, bool]:
        """Combined all-reduce for DP state synchronization.

        Uses a single SUM all-reduce on a 2-element tensor:
          [0] = 1 if this rank has unfinished work, else 0.
                SUM > 0 ≡ logical OR across ranks → any rank has work.
          [1] = 1 if this rank has a pending pause request, else 0.
                SUM == dp_size ≡ all ranks reached pause consensus.

        has_unfinished_global is true if any rank has unfinished work,
        or if some ranks are waiting for a pause consensus.

        Returns:
            (has_unfinished_global, pause_consensus)
        """
        tensor = torch.tensor(
            [int(has_unfinished), int(pending_pause)], dtype=torch.int32, device="cpu"
    # ... omitted for brevity ...
        has_unfinished_global = tensor[0].item() > 0 or pause_count % dp_size != 0
        return has_unfinished_global, pause_count == dp_size
```
**EN:** Method `ParallelConfig.sync_dp_state` provides a reusable helper around the module's main workflow. The docstring highlights: Combined all-reduce for DP state synchronization. Key calls such as `torch.tensor`, `int`, `torch.distributed.all_reduce`, `dp_group.size`, `tensor[1].item` show the concrete execution path.
**CN:** Method `ParallelConfig.sync_dp_state` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Combined all-reduce for DP state synchronization. 像 `torch.tensor`, `int`, `torch.distributed.all_reduce`, `dp_group.size`, `tensor[1].item` 这样的关键调用展示了该代码块的具体执行路径。

### Method `ParallelConfig.compute_hash` (lines 705-758)
```python
    def compute_hash(self):
        """
        Provide a hash that uniquely identifies all the configs
        that affect the structure of the computation
        graph from input ids/embeddings to the final hidden states,
        excluding anything before input ids/embeddings and after
        the final hidden states.

        This hash is also used for DP worker configuration validation
        to prevent hangs from mismatched collective communication patterns.
        """
        ignored_factors = {
            # Derived/runtime topology, networking, or launch details
            "data_parallel_rank",
            "data_parallel_rank_local",
            "data_parallel_size_local",
            "data_parallel_index",
            "data_parallel_backend",
            "data_parallel_external_lb",
    # ... omitted for brevity ...
        factors = get_hash_factors(self, ignored_factors)
        return hash_factors(factors)
```
**EN:** Method `ParallelConfig.compute_hash` computes stable identifiers or fingerprints for caching/comparison. The docstring highlights: Provide a hash that uniquely identifies all the configs that affect the structure of the computation graph from input ids/embeddings to the final hidden states, excluding anythi... Key calls such as `get_hash_factors`, `hash_factors` show the concrete execution path.
**CN:** Method `ParallelConfig.compute_hash` 负责计算稳定标识或指纹以支持缓存/比较。 文档字符串强调：Provide a hash that uniquely identifies all the configs that affect the structure of the computation graph from input ids/embeddings to the final hidden states, excluding anythi... 像 `get_hash_factors`, `hash_factors` 这样的关键调用展示了该代码块的具体执行路径。

### Method `ParallelConfig.__post_init__` (lines 760-904)
```python
    def __post_init__(self) -> None:
        # Continue with the rest of the initialization
        self.world_size = (
            self.pipeline_parallel_size
            * self.tensor_parallel_size
            * self.prefill_context_parallel_size
        )

        if self.distributed_executor_backend == "external_launcher":
            logger.info("Using external launcher for distributed inference.")
            self.world_size *= self.data_parallel_size

        if self.enable_elastic_ep:
            if not self.enable_eplb:
                raise ValueError("Elastic EP is only supported with enable_eplb=True.")
            if self.pipeline_parallel_size > 1:
                raise ValueError(
                    "Elastic EP is not supported with pipeline parallelism "
                    f"(pipeline_parallel_size={self.pipeline_parallel_size})."
    # ... omitted for brevity ...
            else:
                self.eplb_config.communicator = "torch_nccl"
```
**EN:** Method `ParallelConfig.__post_init__` constructs derived objects, runtime state, or helper structures. Key calls such as `logger.info`, `ValueError`, `NotImplementedError`, `int`, `get_open_ports_list` show the concrete execution path.
**CN:** Method `ParallelConfig.__post_init__` 负责构造派生对象、运行时状态或辅助结构。 像 `logger.info`, `ValueError`, `NotImplementedError`, `int`, `get_open_ports_list` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。
- **Caching / 缓存**
  - **EN:** Some definitions are designed to reuse computed state and avoid repeated work.
  - **CN:** 部分定义旨在复用已计算状态，避免重复工作。
- **Benchmarking/data prep / 基准测试/数据准备**
  - **EN:** The file includes utilities for preparing benchmark inputs or measuring runtime behavior.
  - **CN:** 该文件包含用于准备基准测试输入或测量运行时行为的工具。
- **Configuration / 配置**
  - **EN:** Configuration objects and validation rules centralize feature toggles and runtime settings.
  - **CN:** 配置对象与校验规则集中管理功能开关和运行时设置。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import os`, `import socket`, `from collections.abc import Callable`, `from typing import TYPE_CHECKING, Any, Literal, overload`
- **Third-party / 第三方**: `import regex as re`, `import torch`, `from pydantic import Field, field_validator, model_validator`, `from torch.distributed import ProcessGroup, ReduceOp, Store`, `from typing_extensions import Self`
- **vLLM internal / vLLM 内部依赖**: `import vllm.envs as envs`, `from vllm.config.utils import config`, `from vllm.logger import init_logger`, `from vllm.platforms import current_platform`, `from vllm.utils.network_utils import get_open_ports_list`
