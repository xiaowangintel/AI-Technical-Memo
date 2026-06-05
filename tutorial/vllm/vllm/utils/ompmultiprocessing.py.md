# ompmultiprocessing.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/utils/ompmultiprocessing.py`
- **Repository**: vllm-project/vllm
- **Purpose**: OMP Aware Multiprocessing manager for running multiprocessing.Process() Copyright (c) 2026 Red Hat Inc Copyright (c) 2026 Cambridge Greys Ltd / 该模块围绕 `ompmultiprocessing` 相关逻辑组织实现，提供与源文件相匹配的核心功能。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-22)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""OMP Aware Multiprocessing manager for running multiprocessing.Process()
Copyright (c) 2026 Red Hat Inc
Copyright (c) 2026 Cambridge Greys Ltd
"""

import os
from collections.abc import Callable
from contextlib import contextmanager
from typing import TYPE_CHECKING

import vllm.utils.cpu_resource_utils as cr_utils
from vllm import envs
from vllm.logger import init_logger
from vllm.platforms import CpuArchEnum, current_platform
from vllm.utils.cpu_resource_utils import LogicalCPUInfo

if TYPE_CHECKING:
    from vllm.config import VllmConfig

logger = init_logger(__name__)
```
**EN:** Sets up the module with standard-library support such as `os`, `collections.abc`, `contextlib`, vLLM modules such as `vllm.utils.cpu_resource_utils`, `vllm`, `vllm.logger`. It prepares the symbols later used by `OMPProcessManager`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.utils.cpu_resource_utils`, `vllm`, `vllm.logger` 等 vLLM 内部依赖。 这些准备工作为后续的 `OMPProcessManager` 提供上下文。

### OMPProcessManager overview (lines 25-289)
```python
class OMPProcessManager:
    def __init__(self, config: "VllmConfig"):
        if not current_platform.is_cpu():
            return

        self.local_world_size = config.parallel_config.local_world_size
        self.local_dp_rank = config.parallel_config.data_parallel_rank_local
        # This is a bit tricky because the internal DP size
        # is always 1 for non-MoE models
        self.internal_dp_size = config.parallel_config._api_process_count

        self.simulate_multi_node = os.environ.get("VLLM_CPU_SIM_MULTI_NUMA", "0") != "0"
        ld_preload_str = os.getenv("LD_PRELOAD", "")
        self.use_iomp = "libiomp" in ld_preload_str or "libomp" in ld_preload_str
        self.use_gomp = "libgomp" in ld_preload_str

        assert not (self.use_iomp and self.use_gomp)

        # at least reserve 1/local_world_size(for ARM/RISC-V) core for scheduler
        # proc as always use MP executor
        # TODO: make scheduler proc sleep when idle
        self.reserve_cpu_num = (
            self.local_world_size
            if current_platform.get_cpu_architecture()
            in (CpuArchEnum.ARM, CpuArchEnum.RISCV)
            else 1
        )
    # ...
```
**EN:** Defines the `OMPProcessManager` class used by this module. Key methods include `__init__`, `configure_omp_envs`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `OMPProcessManager` 是该文件中的核心类，用于封装与 `OMPProcessManager` 相关的状态和行为。 关键方法包括 `__init__`, `configure_omp_envs`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### OMPProcessManager.__init__ (lines 26-68)
```python
    def __init__(self, config: "VllmConfig"):
        if not current_platform.is_cpu():
            return

        self.local_world_size = config.parallel_config.local_world_size
        self.local_dp_rank = config.parallel_config.data_parallel_rank_local
        # This is a bit tricky because the internal DP size
        # is always 1 for non-MoE models
        self.internal_dp_size = config.parallel_config._api_process_count

        self.simulate_multi_node = os.environ.get("VLLM_CPU_SIM_MULTI_NUMA", "0") != "0"
        ld_preload_str = os.getenv("LD_PRELOAD", "")
        self.use_iomp = "libiomp" in ld_preload_str or "libomp" in ld_preload_str
        self.use_gomp = "libgomp" in ld_preload_str

        assert not (self.use_iomp and self.use_gomp)

        # at least reserve 1/local_world_size(for ARM/RISC-V) core for scheduler
        # proc as always use MP executor
        # TODO: make scheduler proc sleep when idle
    # ...
                logger.warning(msg=msg)
            self.reserve_cpu_num = envs.VLLM_CPU_NUM_OF_RESERVED_CPU

        self._parse_omp_threads_bind_env()

        assert not self.simulate_multi_node or self.auto_setup
```
**EN:** `__init__` initializes state required by the module. It mainly works with `config`. Inside the body, it relies on `os.getenv`, `self._parse_omp_threads_bind_env`, `current_platform.is_cpu` to complete the main steps.
**CN:** `__init__` 负责初始化模块所需的状态。 它主要处理 `config` 等参数。 实现过程中会调用 `os.getenv`, `self._parse_omp_threads_bind_env`, `current_platform.is_cpu` 等函数完成关键步骤。

### OMPProcessManager._get_autobind_cpu_ids (lines 187-289)
```python
    def _get_autobind_cpu_ids(
        self, cpu_selector: Callable[[list[LogicalCPUInfo]], list[LogicalCPUInfo]]
    ) -> tuple[list[list[LogicalCPUInfo]], list[LogicalCPUInfo]]:
        """
        Return CPU ids to bind based on NUMA nodes, and CPU ids reserved for
        other processes.
        Currently for rank N, only CPU ids on the N-th node in available NUMA
        node list will be selected.
        Args:
            cpu_selector: a callable object to select CPUs from a CPU list
            of a physical core. The input is a LogicalCPUInfo list contains
            logical CPUs of a physical CPU, sorted by the LogicalCPUInfo.id.
            A selected LogicalCPUInfo list should be returned.
        """

        # this memory node list has been sliced for DP offset
        allowed_numa_nodes = cr_utils.get_visible_memory_node()
        logical_cpu_list = cr_utils.get_allowed_cpu_list()

        local_world_size = self.local_world_size
    # ...
            num = reserve_num_per_rank[i]
            if num > 0:
                reserved_cpu_list.extend(cpu_lists_of_ranks[i][-num:])
                cpu_lists_of_ranks[i] = cpu_lists_of_ranks[i][:-num]

        return cpu_lists_of_ranks, reserved_cpu_list
```
**EN:** `_get_autobind_cpu_ids`: Return CPU ids to bind based on NUMA nodes, and CPU ids reserved for other processes. It mainly works with `cpu_selector`. Inside the body, it relies on `cr_utils.get_visible_memory_node`, `cr_utils.get_allowed_cpu_list`, `core_to_cpus.values` to complete the main steps.
**CN:** `_get_autobind_cpu_ids` 负责获取流水线所需的数据或状态。 它主要处理 `cpu_selector` 等参数。 实现过程中会调用 `cr_utils.get_visible_memory_node`, `cr_utils.get_allowed_cpu_list`, `core_to_cpus.values` 等函数完成关键步骤。

### OMPProcessManager.configure_omp_envs (lines 71-120)
```python
    def configure_omp_envs(self, rank: int, local_rank: int):
        if not current_platform.is_cpu() or self.skip_setup:
            yield
            return

        envs_dict = {}
        cpu_list = [str(i) for i in self.cpu_lists[local_rank]]
        envs_dict["OMP_NUM_THREADS"] = str(len(cpu_list))
        if self.use_iomp:
            # set IOMP envs
            cpu_list_str = ",".join(cpu_list)
            envs_dict["KMP_AFFINITY"] = (
                f"granularity=fine,explicit,proclist=[{cpu_list_str}]"
            )
            # The time(milliseconds) that a thread should wait after
            # completing the execution of a parallel region, before sleeping.
            # A value of 5 masks thread underutilization.
            # Set to 1 when debugging thread utilization issues.
            envs_dict["KMP_BLOCKTIME"] = "5"
            # Prevents the CPU to run into low performance state
    # ...
            # restore old envs
            for k, v in old_envs_dict.items():  # type: ignore
                if v is None:
                    os.environ.pop(k, None)
                else:
                    os.environ[k] = v
```
**EN:** `configure_omp_envs` implements helper logic used by this module. It mainly works with `rank`, `local_rank`. Inside the body, it relies on `join`, `os.environ.get`, `envs_dict.items` to complete the main steps.
**CN:** `configure_omp_envs` 负责实现本模块使用的辅助逻辑。 它主要处理 `rank`, `local_rank` 等参数。 实现过程中会调用 `join`, `os.environ.get`, `envs_dict.items` 等函数完成关键步骤。

### OMPProcessManager._parse_omp_threads_bind_env (lines 122-185)
```python
    def _parse_omp_threads_bind_env(self):
        vllm_mask = envs.VLLM_CPU_OMP_THREADS_BIND
        self.skip_setup = vllm_mask == "nobind"
        self.auto_setup = vllm_mask == "auto"
        self.reserved_cpu_list = []
        self.cpu_lists = []

        if self.auto_setup:
            # auto generate CPU lists
            cpu_arch = current_platform.get_cpu_architecture()
            if cpu_arch == CpuArchEnum.POWERPC:
                # For POWERPC SMT-8/4/2
                cpu_list, reserve_list = self._get_autobind_cpu_ids(
                    lambda cpus: [cpu for cpu in cpus if cpu.id % 8 < 4]
                )
            elif cpu_arch in (CpuArchEnum.X86, CpuArchEnum.S390X):
                # For x86/S390X SMT-2, use 1 logical CPU per physical core
                cpu_list, reserve_list = self._get_autobind_cpu_ids(
                    lambda cpus: cpus[-1:]
                )
    # ...
            f"reserve_cpu_num={self.reserve_cpu_num}\n"
        )
        for i, cpus in enumerate(self.cpu_lists):
            msg += f"\tlocal_rank={i}, core ids={cpus}\n"
        msg += f"\treserved_cpus={self.reserved_cpu_list}"
        logger.info(msg)
```
**EN:** `_parse_omp_threads_bind_env` parses raw inputs into structured objects. Inside the body, it relies on `logger.info`, `current_platform.get_cpu_architecture`, `self._get_autobind_cpu_ids` to complete the main steps.
**CN:** `_parse_omp_threads_bind_env` 负责把原始输入解析为结构化对象。 实现过程中会调用 `logger.info`, `current_platform.get_cpu_architecture`, `self._get_autobind_cpu_ids` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`OMPProcessManager`**: Core class that organizes module behavior. / **`OMPProcessManager`**：组织模块行为的核心类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: os, collections.abc, contextlib, typing
- **Third-party / 第三方**: None / 无
- **Internal vLLM / vLLM 内部依赖**: vllm.utils.cpu_resource_utils, vllm, vllm.logger, vllm.platforms, vllm.config
