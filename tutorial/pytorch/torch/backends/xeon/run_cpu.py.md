# run_cpu.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/backends/xeon/run_cpu.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Exposes backend-specific switches, capability probes, and preprocessing helpers for optional runtimes.
- **Purpose (CN)**: 暴露后端相关的开关、能力探测以及面向可选运行时的预处理辅助逻辑。
## Line-by-Line Analysis / 逐行分析

### Lines 1-28 / 第 1-28 行
````python
# mypy: allow-untyped-defs
"""
This is a script for launching PyTorch inference on Intel(R) Xeon(R) Scalable Processors with optimal configurations.

Single instance inference, multi-instance inference are enabled.

Note: term "instance" here doesn't refer to a cloud instance. This script is executed as a single process. It invokes
multiple "instances" which are formed from multiple threads for each. "instance" is kind of group of threads in this
context.

Illustrated as below:

::

    +-----------------------------+----------------------+-------+
    |            process          |        thread        | core  |
    +=============================+======================+=======+
    | torch.backends.xeon.run_cpu | instance 0: thread 0 |   0   |
    |                             |             thread 1 |   1   |
    |                             +----------------------+-------+
    |                             | instance 1: thread 0 |   2   |
    |                             |             thread 1 |   3   |
    |                             +----------------------+-------+
    |                             | ...                  |  ...  |
    |                             +----------------------+-------+
    |                             | instance N: thread 0 |   M   |
    |                             |             thread 1 |  M+1  |
    +-----------------------------+----------------------+-------+
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 30-56 / 第 30-56 行
````python
To get the peak performance on Intel(R) Xeon(R) Scalable Processors, the script optimizes the configuration of thread and memory
management. For thread management, the script configures thread affinity and the preload of Intel OMP library.
For memory management, it configures NUMA binding and preload optimized memory allocation library (e.g. tcmalloc, jemalloc).

Environment variables that will be set by this script:

+------------------+-------------------------------------------------------------------------------------------------+
| Environ Variable |                                             Value                                               |
+==================+=================================================================================================+
|    LD_PRELOAD    | Depending on knobs you set, <lib>/libiomp5.so, <lib>/libjemalloc.so, <lib>/libtcmalloc.so might |
|                  | be appended to LD_PRELOAD.                                                                      |
+------------------+-------------------------------------------------------------------------------------------------+
|   KMP_AFFINITY   | If libiomp5.so is preloaded, KMP_AFFINITY could be set to "granularity=fine,compact,1,0".       |
+------------------+-------------------------------------------------------------------------------------------------+
|   KMP_BLOCKTIME  | If libiomp5.so is preloaded, KMP_BLOCKTIME is set to "1".                                       |
+------------------+-------------------------------------------------------------------------------------------------+
|  OMP_NUM_THREADS | value of ncores_per_instance                                                                    |
+------------------+-------------------------------------------------------------------------------------------------+
|    MALLOC_CONF   | If libjemalloc.so is preloaded, MALLOC_CONF will be set to                                      |
|                  | "oversize_threshold:1,background_thread:true,metadata_thp:auto".                                |
+------------------+-------------------------------------------------------------------------------------------------+

*Note*: This script respects environment variables set preliminarily. I.e. If you set the environment variables
mentioned above before running the script, the script will not overwrite the values in the script.

How to use this module:
~~~~~~~~~~~~~~~~~~~~~~~
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 58-84 / 第 58-84 行
````python
Single instance inference
-------------------------

1. Run single-instance inference on a single node with all CPU nodes.

::

   python -m torch.backends.xeon.run_cpu --throughput-mode script.py args

2. Run single-instance inference on a single CPU node.

::

   python -m torch.backends.xeon.run_cpu --node-id 1 script.py args

Multi-instance inference
------------------------

1. Multi-instance
   By default this tool runs one process per node. If you want to set the instance numbers and core per instance,
   --ninstances and  --ncores-per-instance should be set.

::

   python -m torch.backends.xeon.run_cpu -- python_script args

   eg: on an Intel(R) Xeon(R) Scalable Processor with 14 instance, 4 cores per instance
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 86-111 / 第 86-111 行
````python
::

   python -m torch.backends.xeon.run_cpu --ninstances 14 --ncores-per-instance 4 python_script args

2. Run single-instance inference among multiple instances.
   By default, runs all ninstances. If you want to independently run a single instance among ninstances, specify rank.

   eg: run 0th instance on an Intel(R) Xeon(R) Scalable Processor with 2 instance (i.e., numactl -C 0-27)

::

   python -m torch.backends.xeon.run_cpu --ninstances 2 --rank 0 python_script args

   eg: run 1st instance on an Intel(R) Xeon(R) Scalable Processor with 2 instance (i.e., numactl -C 28-55)

::

   python -m torch.backends.xeon.run_cpu --ninstances 2 --rank 1 python_script args

   eg: run 0th instance on an Intel(R) Xeon(R) Scalable Processor with 2 instance, 2 cores per instance,
   first four cores (i.e., numactl -C 0-1)

::

   python -m torch.backends.xeon.run_cpu --core-list "0, 1, 2, 3" --ninstances 2 --ncores-per-instance 2
   --rank 0 python_script args
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 113-134 / 第 113-134 行
````python
3. To look up what optional arguments this module offers:

::

    python -m torch.backends.xeon.run_cpu --help

Memory allocator
----------------

"--enable-tcmalloc" and "--enable-jemalloc" can be used to enable different memory allocator.

"""

import glob
import logging
import os
import platform
import re
import subprocess
import sys
from argparse import ArgumentParser, RawTextHelpFormatter, REMAINDER
from os.path import expanduser
````
- **EN**: This block assembles module dependencies, pulling in standard-library helpers such as glob, logging, os, ....
- **CN**: 这一段组织模块依赖，引入了标准库辅助模块，如 glob、logging、os、...。

### Lines 136-163 / 第 136-163 行
````python
from torch.distributed.elastic.multiprocessing import (
    DefaultLogsSpecs as _DefaultLogsSpecs,
    start_processes,
    Std,
)


format_str = "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
logging.basicConfig(level=logging.INFO, format=format_str)
logger = logging.getLogger(__name__)


class _CPUinfo:
    """Get CPU information, such as cores list and NUMA information."""

    def __init__(self, test_input=""):
        self.cpuinfo = []
        if platform.system() in ["Windows", "Darwin"]:
            raise RuntimeError(f"{platform.system()} is not supported!!!")
        elif platform.system() == "Linux":
            # Sample output of: `lscpu --parse=CPU,Core,Socket,Node`
            #
            # # The following is the parsable format, which can be fed to other
            # # programs. Each different item in every column has an unique ID
            # # starting from zero.
            # # CPU,Core,Socket,Node
            # 0,0,0,0
            # 1,1,0,0
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.distributed.elastic.multiprocessing. It introduces or extends `_CPUinfo`, which hold the main object-oriented state for this portion of the file. This chunk defines `__init__`, which initializes object state and binds construction-time configuration. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.distributed.elastic.multiprocessing。 它引入或扩展了 `_CPUinfo`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 164-186 / 第 164-186 行
````python
            # ...
            if test_input == "":
                lscpu_cmd = ["lscpu", "--parse=CPU,Core,Socket,Node"]
                lscpu_info = subprocess.check_output(
                    lscpu_cmd, universal_newlines=True
                ).split("\n")
            else:
                lscpu_info = test_input.split("\n")

            # Get information about  cpu, core, socket and node
            for line in lscpu_info:
                pattern = r"^([\d]+,[\d]+,[\d]+,[\d]?)"
                regex_out = re.search(pattern, line)
                if regex_out:
                    self.cpuinfo.append(regex_out.group(1).strip().split(","))

            # physical cores := core column in lscpu output
            #  logical cores :=  cPU column in lscpu output
            self.node_nums = int(max(line[3] for line in self.cpuinfo)) + 1
            self.node_physical_cores: list[list[int]] = []  # node_id is index
            self.node_logical_cores: list[list[int]] = []  # node_id is index
            self.physical_core_node_map = {}  # physical core to numa node id
            self.logical_core_node_map = {}  # logical core to numa node id
````
- **EN**: This chunk continues `_CPUinfo` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_CPUinfo`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 188-213 / 第 188-213 行
````python
            for node_id in range(self.node_nums):
                cur_node_physical_core = []
                cur_node_logical_core = []
                for cpuinfo in self.cpuinfo:
                    nid = cpuinfo[3] if cpuinfo[3] != "" else "0"
                    if node_id == int(nid):
                        if int(cpuinfo[1]) not in cur_node_physical_core:
                            cur_node_physical_core.append(int(cpuinfo[1]))
                            self.physical_core_node_map[int(cpuinfo[1])] = int(node_id)
                        cur_node_logical_core.append(int(cpuinfo[0]))
                        self.logical_core_node_map[int(cpuinfo[0])] = int(node_id)
                self.node_physical_cores.append(cur_node_physical_core)
                self.node_logical_cores.append(cur_node_logical_core)

    def _physical_core_nums(self):
        return len(self.node_physical_cores) * len(self.node_physical_cores[0])

    def _logical_core_nums(self):
        return len(self.node_logical_cores) * len(self.node_logical_cores[0])

    def get_node_physical_cores(self, node_id):
        if node_id < 0 or node_id > self.node_nums - 1:
            raise ValueError(
                f"Invalid node id: {node_id}. Valid node ids: {list(range(len(self.node_physical_cores)))}"
            )
        return self.node_physical_cores[node_id]
````
- **EN**: This chunk defines `get_node_physical_cores`, which retrieves runtime state and exposes it through a Python-friendly accessor. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `get_node_physical_cores`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 215-236 / 第 215-236 行
````python
    def get_node_logical_cores(self, node_id):
        if node_id < 0 or node_id > self.node_nums - 1:
            raise ValueError(
                f"Invalid node id: {node_id}. Valid node ids: {list(range(len(self.node_physical_cores)))}"
            )
        return self.node_logical_cores[node_id]

    def get_all_physical_cores(self):
        all_cores = []
        for cores in self.node_physical_cores:
            all_cores.extend(cores)
        return all_cores

    def get_all_logical_cores(self):
        all_cores = []
        for cores in self.node_logical_cores:
            all_cores.extend(cores)
        return all_cores

    def numa_aware_check(self, core_list):
        """
        Check whether all cores in core_list are in the same NUMA node.
````
- **EN**: This chunk defines `numa_aware_check`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `numa_aware_check`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 238-263 / 第 238-263 行
````python
        Cross NUMA will reduce performance.
        We strongly advice to not use cores on different nodes.
        """
        cores_numa_map = self.logical_core_node_map
        numa_ids = []
        for core in core_list:
            numa_id = cores_numa_map[core]
            if numa_id not in numa_ids:
                numa_ids.append(numa_id)
        if len(numa_ids) > 1:
            logger.warning(
                "Numa Aware: cores:%s on different NUMA nodes:%s. To avoid \
this behavior, please use --ncores-per-instance knob to make sure number of cores is divisible by --ncores-per-\
instance. Alternatively, please use --skip-cross-node-cores knob.",
                core_list,
                numa_ids,
            )
        if len(numa_ids) == 0:
            raise RuntimeError(
                "invalid number of NUMA nodes; please make sure numa_ids >= 1"
            )
        return numa_ids


class _Launcher:
    r"""Class for launcher."""
````
- **EN**: It introduces or extends `_Launcher`, which hold the main object-oriented state for this portion of the file. This chunk continues `_Launcher` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 它引入或扩展了 `_Launcher`，这些类承载了本段涉及的主要面向对象状态。 这一段延续了 `_Launcher`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 265-288 / 第 265-288 行
````python
    msg_lib_notfound = (
        f"Unable to find the {{0}} library file lib{{1}}.so in $CONDA_PREFIX/lib or $VIRTUAL_ENV/lib \
or /.local/lib/ or /usr/local/lib/ or /usr/local/lib64/ or /usr/lib or /usr/lib64 or \
{expanduser('~')}/.local/lib/ so the LD_PRELOAD environment variable will not be set."
    )

    def __init__(self) -> None:
        self.cpuinfo = _CPUinfo()

    def add_lib_preload(self, lib_type):
        """Enable TCMalloc/JeMalloc/intel OpenMP."""
        library_paths = []
        if "CONDA_PREFIX" in os.environ:
            library_paths.append(f"{os.environ['CONDA_PREFIX']}/lib")
        if "VIRTUAL_ENV" in os.environ:
            library_paths.append(f"{os.environ['VIRTUAL_ENV']}/lib")

        library_paths += [
            f"{expanduser('~')}/.local/lib",
            "/usr/local/lib",
            "/usr/local/lib64",
            "/usr/lib",
            "/usr/lib64",
        ]
````
- **EN**: This chunk defines `add_lib_preload`, which serializes or reconstructs state across a Python-visible boundary. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `add_lib_preload`，其作用是在 Python 可见边界上序列化或重建状态。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 290-311 / 第 290-311 行
````python
        lib_find = False
        lib_set = False
        for item in os.getenv("LD_PRELOAD", "").split(":"):
            if item.endswith(f"lib{lib_type}.so"):
                lib_set = True
                break
        if not lib_set:
            for lib_path in library_paths:
                # pyrefly: ignore [unbound-name]
                library_file = os.path.join(lib_path, f"lib{lib_type}.so")
                matches = glob.glob(library_file)
                if len(matches) > 0:
                    # pyrefly: ignore [unbound-name]
                    ld_preloads = [f"{matches[0]}", os.getenv("LD_PRELOAD", "")]
                    # pyrefly: ignore [unbound-name]
                    os.environ["LD_PRELOAD"] = os.pathsep.join(
                        # pyrefly: ignore [unbound-name]
                        [p.strip(os.pathsep) for p in ld_preloads if p]
                    )
                    lib_find = True
                    break
        return lib_set or lib_find
````
- **EN**: This chunk continues `add_lib_preload` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `add_lib_preload`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 313-334 / 第 313-334 行
````python
    def is_numactl_available(self):
        numactl_available = False
        try:
            cmd = ["numactl", "-C", "0", "-m", "0", "hostname"]
            r = subprocess.run(
                cmd,
                env=os.environ,
                stdout=subprocess.DEVNULL,
                stderr=subprocess.DEVNULL,
                check=False,
            )
            if r.returncode == 0:
                numactl_available = True
        except Exception:
            pass
        return numactl_available

    def set_memory_allocator(
        self, enable_tcmalloc=True, enable_jemalloc=False, use_default_allocator=False
    ):
        """
        Enable TCMalloc/JeMalloc with LD_PRELOAD and set configuration for JeMalloc.
````
- **EN**: This chunk defines `set_memory_allocator`, which mutates configuration or backend state that affects later execution. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `set_memory_allocator`，其作用是修改会影响后续执行的配置或后端状态。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 336-362 / 第 336-362 行
````python
        By default, PTMalloc will be used for PyTorch, but TCMalloc and JeMalloc can get better
        memory reuse and reduce page fault to improve performance.
        """
        if enable_tcmalloc and enable_jemalloc:
            raise RuntimeError(
                "Unable to enable TCMalloc and JEMalloc at the same time."
            )

        if enable_tcmalloc:
            find_tc = self.add_lib_preload(lib_type="tcmalloc")
            if not find_tc:
                msg = f'{self.msg_lib_notfound} you can use "conda install -c conda-forge gperftools" to install {{0}}'
                logger.warning(msg.format("TCmalloc", "tcmalloc"))
            else:
                logger.info("Use TCMalloc memory allocator")

        elif enable_jemalloc:
            find_je = self.add_lib_preload(lib_type="jemalloc")
            if not find_je:
                msg = f'{self.msg_lib_notfound} you can use "conda install -c conda-forge jemalloc" to install {{0}}'
                logger.warning(msg.format("Jemalloc", "jemalloc"))
            else:
                logger.info("Use JeMalloc memory allocator")
                self.set_env(
                    "MALLOC_CONF",
                    "oversize_threshold:1,background_thread:true,metadata_thp:auto",
                )
````
- **EN**: This chunk continues `set_memory_allocator` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `set_memory_allocator`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 364-386 / 第 364-386 行
````python
        elif use_default_allocator:
            pass

        else:
            find_tc = self.add_lib_preload(lib_type="tcmalloc")
            if find_tc:
                logger.info("Use TCMalloc memory allocator")
                return
            find_je = self.add_lib_preload(lib_type="jemalloc")
            if find_je:
                logger.info("Use JeMalloc memory allocator")
                return
            logger.warning(
                """Neither TCMalloc nor JeMalloc is found in $CONDA_PREFIX/lib or $VIRTUAL_ENV/lib
                            or /.local/lib/ or /usr/local/lib/ or /usr/local/lib64/ or /usr/lib or /usr/lib64 or
                           %s/.local/lib/ so the LD_PRELOAD environment variable will not be set.
                           This may drop the performance""",
                expanduser("~"),
            )

    def log_env_var(self, env_var_name=""):
        if env_var_name in os.environ:
            logger.info("%s=%s", env_var_name, os.environ[env_var_name])
````
- **EN**: This chunk defines `log_env_var`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `log_env_var`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 388-415 / 第 388-415 行
````python
    def set_env(self, env_name, env_value):
        if not env_value:
            logger.warning("%s is None", env_name)
        if env_name not in os.environ:
            os.environ[env_name] = env_value
        elif os.environ[env_name] != env_value:
            logger.warning(
                "Overriding value with the one set in environment variable: %s. \
Value applied: %s. Value ignored: %s",
                env_name,
                os.environ[env_name],
                env_value,
            )
        self.log_env_var(env_name)

    # set_kmp_affinity is used to control whether to set KMP_AFFINITY or not.
    # In scenario that use all cores on all nodes, including logical cores, setting KMP_AFFINITY disables logical cores.
    # In this case, KMP_AFFINITY should not be set.
    def set_multi_thread_and_allocator(
        self,
        ncores_per_instance,
        disable_iomp=False,
        set_kmp_affinity=True,
        enable_tcmalloc=True,
        enable_jemalloc=False,
        use_default_allocator=False,
    ):
        """
````
- **EN**: This chunk defines `set_multi_thread_and_allocator`, which mutates configuration or backend state that affects later execution. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `set_multi_thread_and_allocator`，其作用是修改会影响后续执行的配置或后端状态。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 416-439 / 第 416-439 行
````python
        Set multi-thread configuration and enable Intel openMP and TCMalloc/JeMalloc.

        By default, GNU openMP and PTMalloc are used in PyTorch. but Intel openMP and TCMalloc/JeMalloc are better alternatives
        to get performance benefit.
        """
        self.set_memory_allocator(
            enable_tcmalloc, enable_jemalloc, use_default_allocator
        )
        self.set_env("OMP_NUM_THREADS", str(ncores_per_instance))
        if not disable_iomp:
            find_iomp = self.add_lib_preload(lib_type="iomp5")
            if not find_iomp:
                msg = f'{self.msg_lib_notfound} you can use "conda install mkl" to install {{0}}'
                logger.warning(msg.format("iomp", "iomp5"))
            else:
                logger.info("Using Intel OpenMP")
                if set_kmp_affinity:
                    self.set_env("KMP_AFFINITY", "granularity=fine,compact,1,0")
                self.set_env("KMP_BLOCKTIME", "1")
        self.log_env_var("LD_PRELOAD")

    r"""
     Launcher for single instance and multi-instance
     """
````
- **EN**: This chunk continues `set_multi_thread_and_allocator` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `set_multi_thread_and_allocator`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 441-462 / 第 441-462 行
````python
    def launch(self, args):
        cores = []
        set_kmp_affinity = True
        enable_taskset = False
        if args.core_list:  # user specify what cores will be used by params
            cores = [int(x) for x in args.core_list.split(",")]
            if args.ncores_per_instance == -1:
                raise RuntimeError(
                    'please specify the "--ncores-per-instance" if you have pass the --core-list params'
                )
            elif (
                args.ninstances > 1
                and args.ncores_per_instance * args.ninstances < len(cores)
            ):
                logger.warning(
                    "only first %s cores will be used, \
but you specify %s cores in core_list",
                    args.ncores_per_instance * args.ninstances,
                    len(cores),
                )
            else:
                args.ninstances = len(cores) // args.ncores_per_instance
````
- **EN**: This chunk defines `launch`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `launch`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 464-491 / 第 464-491 行
````python
        else:
            if args.use_logical_core:
                if args.node_id != -1:
                    cores = self.cpuinfo.get_node_logical_cores(args.node_id)
                else:
                    cores = self.cpuinfo.get_all_logical_cores()
                    # When using all cores on all nodes, including logical cores,
                    # setting KMP_AFFINITY disables logical cores. Thus, KMP_AFFINITY should not be set.
                    set_kmp_affinity = False
            else:
                if args.node_id != -1:
                    cores = self.cpuinfo.get_node_physical_cores(args.node_id)
                else:
                    cores = self.cpuinfo.get_all_physical_cores()
            if (
                not args.multi_instance
                and args.ninstances == -1
                and args.ncores_per_instance == -1
            ):
                args.ninstances = 1
                args.ncores_per_instance = len(cores)
            elif (
                args.multi_instance
                and args.ninstances == -1
                and args.ncores_per_instance == -1
            ):
                args.throughput_mode = True
            elif args.ncores_per_instance == -1 and args.ninstances != -1:
````
- **EN**: This chunk continues `launch` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `launch`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 492-519 / 第 492-519 行
````python
                if args.ninstances > len(cores):
                    raise RuntimeError(
                        f"there are {len(cores)} total cores but you specify {args.ninstances} ninstances; \
please make sure ninstances <= total_cores)"
                    )
                else:
                    args.ncores_per_instance = len(cores) // args.ninstances
            elif args.ncores_per_instance != -1 and args.ninstances == -1:
                if not args.skip_cross_node_cores:
                    args.ninstances = len(cores) // args.ncores_per_instance
                else:
                    ncore_per_node = len(self.cpuinfo.node_physical_cores[0])
                    num_leftover_cores = ncore_per_node % args.ncores_per_instance
                    if args.ncores_per_instance > ncore_per_node:
                        # too many ncores_per_instance to skip cross-node cores
                        logger.warning(
                            "there are %s core(s) per socket, but you specify %s ncores_per_instance and \
skip_cross_node_cores. Please make sure --ncores-per-instance < core(s) per \
socket",
                            ncore_per_node,
                            args.ncores_per_instance,
                        )
                        sys.exit(-1)
                    elif num_leftover_cores == 0:
                        # aren't any cross-node cores
                        logger.info(
                            "--skip-cross-node-cores is set, but there are no cross-node cores."
                        )
````
- **EN**: This chunk continues `launch` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `launch`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 520-547 / 第 520-547 行
````python
                        args.ninstances = len(cores) // args.ncores_per_instance
                    else:
                        # skip cross-node cores
                        if args.ninstances != -1:
                            logger.warning(
                                "--skip-cross-node-cores is exclusive to --ninstances. --ninstances \
won't take effect even if it is set explicitly."
                            )

                        i = 1
                        leftover_cores = set()
                        while ncore_per_node * i <= len(cores):
                            leftover_cores.update(
                                cores[
                                    ncore_per_node * i
                                    - num_leftover_cores : ncore_per_node * i
                                ]
                            )
                            i += 1
                        cores = list(set(cores) - leftover_cores)
                        if len(cores) % args.ncores_per_instance != 0:
                            raise AssertionError(
                                f"Number of cores ({len(cores)}) must be divisible by "
                                f"ncores_per_instance ({args.ncores_per_instance})"
                            )
                        args.ninstances = len(cores) // args.ncores_per_instance
            else:
                if args.ninstances * args.ncores_per_instance > len(cores):
````
- **EN**: This chunk continues `launch` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `launch`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 548-574 / 第 548-574 行
````python
                    raise RuntimeError(
                        "Please make sure ninstances * ncores_per_instance <= total_cores"
                    )
            if args.latency_mode:
                logger.warning(
                    "--latency-mode is exclusive to --ninstances, --ncores-per-instance, --node-id and \
--use-logical-core. They won't take effect even they are set explicitly."
                )
                args.ncores_per_instance = 4
                cores = self.cpuinfo.get_all_physical_cores()
                args.ninstances = len(cores) // args.ncores_per_instance

            if args.throughput_mode:
                logger.warning(
                    "--throughput-mode is exclusive to --ninstances, --ncores-per-instance, --node-id and \
--use-logical-core. They won't take effect even they are set explicitly."
                )
                args.ninstances = self.cpuinfo.node_nums
                cores = self.cpuinfo.get_all_physical_cores()
                args.ncores_per_instance = len(cores) // args.ninstances

        if args.ninstances > 1 and args.rank != -1:
            logger.info(
                "assigning %s cores for instance %s",
                args.ncores_per_instance,
                args.rank,
            )
````
- **EN**: This chunk continues `launch` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `launch`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 576-594 / 第 576-594 行
````python
        if not args.disable_numactl:
            numactl_available = self.is_numactl_available()
            if not numactl_available:
                if not args.disable_taskset:
                    logger.warning(
                        "Core binding with numactl is not available. Disabling numactl and using taskset instead. \
                    This may affect performance in multi-socket system; please use numactl if memory binding is needed."
                    )
                    args.disable_numactl = True
                    enable_taskset = True
                else:
                    logger.warning(
                        "Core binding with numactl is not available, and --disable_taskset is set. \
                    Please unset --disable_taskset to use taskset instead of numactl."
                    )
                    sys.exit(-1)

        if not args.disable_taskset:
            enable_taskset = True
````
- **EN**: This chunk continues `launch` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `launch`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 596-623 / 第 596-623 行
````python
        self.set_multi_thread_and_allocator(
            args.ncores_per_instance,
            args.disable_iomp,
            set_kmp_affinity,
            args.enable_tcmalloc,
            args.enable_jemalloc,
            args.use_default_allocator,
        )
        entrypoint = ""
        launch_args = {}
        launch_envs: dict[int, dict] = {}
        launch_tee = {}
        # check whether is launched from torchrun with --nproc-per-node <num workers>
        local_size = int(os.environ.get("LOCAL_WORLD_SIZE", 1))
        local_rank = int(os.environ.get("LOCAL_RANK", 0))
        for i in range(args.ninstances):
            cmd = []
            cur_process_cores = ""
            if not args.disable_numactl or enable_taskset:
                if not args.disable_numactl:
                    cmd = ["numactl"]
                elif enable_taskset:
                    cmd = ["taskset"]
                cores = sorted(cores)
                if (
                    args.rank == -1
                ):  # sequentially assign ncores_per_instance to ninstances
                    core_list = cores[
````
- **EN**: This chunk continues `launch` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `launch`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 624-651 / 第 624-651 行
````python
                        i * args.ncores_per_instance : (i + 1)
                        * args.ncores_per_instance
                    ]
                else:  # assign ncores_per_instance from rank
                    core_list = cores[
                        args.rank * args.ncores_per_instance : (args.rank + 1)
                        * args.ncores_per_instance
                    ]

                core_ranges: list[dict] = []
                if local_size > 1:
                    total_num_cores = len(core_list)
                    cores_per_rank = total_num_cores // local_size
                    if cores_per_rank < 1:
                        raise AssertionError(
                            f"At least one core needs to be assigned to each rank, "
                            f"got {total_num_cores} cores for {local_size} ranks"
                        )
                    core_list = core_list[
                        cores_per_rank * local_rank : cores_per_rank * (local_rank + 1)
                    ]
                for core in core_list:
                    if len(core_ranges) == 0:
                        range_elem = {"start": core, "end": core}
                        core_ranges.append(range_elem)
                    else:
                        if core - core_ranges[-1]["end"] == 1:
                            core_ranges[-1]["end"] = core
````
- **EN**: This chunk continues `launch` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `launch`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 652-679 / 第 652-679 行
````python
                        else:
                            range_elem = {"start": core, "end": core}
                            core_ranges.append(range_elem)
                for r in core_ranges:
                    cur_process_cores = f"{cur_process_cores}{r['start']}-{r['end']},"
                cur_process_cores = cur_process_cores[:-1]
                if not args.disable_numactl:
                    numa_params = f"-C {cur_process_cores} "
                    numa_ids = ",".join(
                        [
                            str(numa_id)
                            for numa_id in self.cpuinfo.numa_aware_check(core_list)
                        ]
                    )
                    numa_params += f"-m {numa_ids}"
                    cmd.extend(numa_params.split())
                elif enable_taskset:
                    taskset_params = f"-c {cur_process_cores} "
                    cmd.extend(taskset_params.split())
            with_python = not args.no_python
            if with_python:
                cmd.append(sys.executable)
                cmd.append("-u")
            if args.module:
                cmd.append("-m")
            cmd.append(args.program)
            cmd.extend(args.program_args)
            cmd_s = " ".join(cmd)
````
- **EN**: This chunk continues `launch` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `launch`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 680-698 / 第 680-698 行
````python
            logger.info(cmd_s)
            if entrypoint == "":
                entrypoint = cmd[0]
            del cmd[0]
            launch_args[i] = tuple(cmd)
            launch_envs[i] = {}
            launch_tee[i] = Std.ALL

            if args.rank != -1:  # launches single instance, rank, only
                break

        ctx = start_processes(
            name=args.log_file_prefix,
            entrypoint=entrypoint,
            args=launch_args,
            envs=launch_envs,
            logs_specs=_DefaultLogsSpecs(log_dir=args.log_path, tee=launch_tee),
        )
        ctx.wait()
````
- **EN**: This chunk continues `launch` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `launch`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 701-724 / 第 701-724 行
````python
def _add_memory_allocator_params(parser):
    group = parser.add_argument_group("Memory Allocator Parameters")
    # allocator control
    group.add_argument(
        "--enable-tcmalloc",
        "--enable_tcmalloc",
        action="store_true",
        default=False,
        help="Enable tcmalloc allocator",
    )
    group.add_argument(
        "--enable-jemalloc",
        "--enable_jemalloc",
        action="store_true",
        default=False,
        help="Enable jemalloc allocator",
    )
    group.add_argument(
        "--use-default-allocator",
        "--use_default_allocator",
        action="store_true",
        default=False,
        help="Use default memory allocator",
    )
````
- **EN**: This chunk defines `_add_memory_allocator_params`, which implements a focused helper used by the surrounding module.
- **CN**: 这一段定义了 `_add_memory_allocator_params`，其作用是实现周边模块使用的关键辅助逻辑。

### Lines 727-754 / 第 727-754 行
````python
def _add_multi_instance_params(parser):
    group = parser.add_argument_group("Multi-instance Parameters")
    # multi-instance control
    group.add_argument(
        "--ncores-per-instance",
        "--ncores_per_instance",
        metavar="\b",
        default=-1,
        type=int,
        help="Cores per instance",
    )
    group.add_argument(
        "--ninstances",
        metavar="\b",
        default=-1,
        type=int,
        help="For multi-instance, you should give the cores number you used for per instance.",
    )
    group.add_argument(
        "--skip-cross-node-cores",
        "--skip_cross_node_cores",
        action="store_true",
        default=False,
        help="If specified --ncores-per-instance, skips cross-node cores.",
    )
    group.add_argument(
        "--rank",
        metavar="\b",
````
- **EN**: This chunk defines `_add_multi_instance_params`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `_add_multi_instance_params`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 755-782 / 第 755-782 行
````python
        default="-1",
        type=int,
        help="Specify instance index to assign ncores_per_instance for rank; \
otherwise ncores_per_instance will be assigned sequentially to ninstances. Please refer to \
https://github.com/intel/intel-extension-for-pytorch/blob/master/docs/tutorials/performance_tuning/launch_script.md",
    )
    group.add_argument(
        "--latency-mode",
        "--latency_mode",
        action="store_true",
        default=False,
        help="By default 4 core per instance and use all physical cores",
    )
    group.add_argument(
        "--throughput-mode",
        "--throughput_mode",
        action="store_true",
        default=False,
        help="By default one instance per node and use all physical cores",
    )
    group.add_argument(
        "--node-id",
        "--node_id",
        metavar="\b",
        default=-1,
        type=int,
        help="node id for multi-instance, by default all nodes will be used",
    )
````
- **EN**: This chunk continues `_add_multi_instance_params` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `_add_multi_instance_params`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 783-810 / 第 783-810 行
````python
    group.add_argument(
        "--use-logical-core",
        "--use_logical_core",
        action="store_true",
        default=False,
        help="Whether only use physical cores",
    )
    group.add_argument(
        "--disable-numactl",
        "--disable_numactl",
        action="store_true",
        default=False,
        help="Disable numactl",
    )
    group.add_argument(
        "--disable-taskset",
        "--disable_taskset",
        action="store_true",
        default=False,
        help="Disable taskset",
    )
    group.add_argument(
        "--core-list",
        "--core_list",
        metavar="\b",
        default=None,
        type=str,
        help='Specify the core list as "core_id, core_id, ....", otherwise, all the cores will be used.',
````
- **EN**: This chunk continues `_add_multi_instance_params` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `_add_multi_instance_params`，进一步展开其内部控制流或状态更新。

### Lines 811-828 / 第 811-828 行
````python
    )
    group.add_argument(
        "--log-path",
        "--log_path",
        metavar="\b",
        default="",
        type=str,
        help="The log file directory. Default path is "
        ", which means disable logging to files.",
    )
    group.add_argument(
        "--log-file-prefix",
        "--log_file_prefix",
        metavar="\b",
        default="run",
        type=str,
        help="log file prefix",
    )
````
- **EN**: This chunk continues `_add_multi_instance_params` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `_add_multi_instance_params`，进一步展开其内部控制流或状态更新。

### Lines 831-855 / 第 831-855 行
````python
def _add_kmp_iomp_params(parser):
    group = parser.add_argument_group("IOMP Parameters")
    group.add_argument(
        "--disable-iomp",
        "--disable_iomp",
        action="store_true",
        default=False,
        help="By default, we use Intel OpenMP and libiomp5.so will be add to LD_PRELOAD",
    )


def create_args(parser=None):
    """
    Parse the command line options.

    @retval ArgumentParser
    """
    # pyrefly: ignore [missing-attribute]
    parser.add_argument(
        "--multi-instance",
        "--multi_instance",
        action="store_true",
        default=False,
        help="Enable multi-instance, by default one instance per node",
    )
````
- **EN**: This chunk defines `create_args`, which implements a focused helper used by the surrounding module. Decorators such as `retval` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks.
- **CN**: 这一段定义了 `create_args`，其作用是实现周边模块使用的关键辅助逻辑。 像 `retval` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。

### Lines 857-879 / 第 857-879 行
````python
    # pyrefly: ignore [missing-attribute]
    parser.add_argument(
        "-m",
        "--module",
        default=False,
        action="store_true",
        help="Changes each process to interpret the launch script "
        "as a python module, executing with the same behavior as"
        '"python -m".',
    )

    # pyrefly: ignore [missing-attribute]
    parser.add_argument(
        "--no-python",
        "--no_python",
        default=False,
        action="store_true",
        help='Do not prepend the --program script with "python" - just exec '
        "it directly. Useful when the script is not a Python script.",
    )

    _add_memory_allocator_params(parser)
    _add_kmp_iomp_params(parser)
````
- **EN**: This chunk continues `create_args` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 这一段延续了 `create_args`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 881-904 / 第 881-904 行
````python
    _add_multi_instance_params(parser)
    # positional
    # pyrefly: ignore [missing-attribute]
    parser.add_argument(
        "program",
        type=str,
        help="The full path to the program/script to be launched. "
        "followed by all the arguments for the script",
    )

    # rest from the training program
    # pyrefly: ignore [missing-attribute]
    parser.add_argument("program_args", nargs=REMAINDER)


def main(args):
    env_before = set(os.environ.keys())
    if platform.system() in ["Windows", "Darwin"]:
        raise RuntimeError(f"{platform.system()} is not supported!!!")

    if args.log_path:
        os.makedirs(args.log_path, exist_ok=True)
    else:
        args.log_path = os.devnull
````
- **EN**: This chunk defines `main`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `main`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 906-929 / 第 906-929 行
````python
    if args.latency_mode and args.throughput_mode:
        raise RuntimeError(
            "Either args.latency_mode or args.throughput_mode should be set"
        )

    if not args.no_python and not args.program.endswith(".py"):
        raise RuntimeError(
            'For non Python script, you should use "--no-python" parameter.'
        )

    # Verify LD_PRELOAD
    if "LD_PRELOAD" in os.environ:
        lst_valid = []
        tmp_ldpreload = os.environ["LD_PRELOAD"]
        for item in tmp_ldpreload.split(":"):
            matches = glob.glob(item)
            if len(matches) > 0:
                lst_valid.append(item)
            else:
                logger.warning("%s doesn't exist. Removing it from LD_PRELOAD.", item)
        if len(lst_valid) > 0:
            os.environ["LD_PRELOAD"] = ":".join(lst_valid)
        else:
            os.environ["LD_PRELOAD"] = ""
````
- **EN**: This chunk continues `main` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 931-957 / 第 931-957 行
````python
    launcher = _Launcher()
    launcher.launch(args)
    for x in sorted(set(os.environ.keys()) - env_before):
        logger.debug("%s=%s", x, os.environ[x])


if __name__ == "__main__":
    parser = ArgumentParser(
        description="This is a script for launching PyTorch inference on Intel(R) Xeon(R) Scalable "
        "Processors with optimal configurations. Single instance inference, "
        "multi-instance inference are enable. To get the peak performance on Intel(R) "
        "Xeon(R) Scalable Processors, the script optimizes the configuration "
        "of thread and memory management. For thread management, the script configures thread "
        "affinity and the preload of Intel OMP library. For memory management, it configures "
        "NUMA binding and preload optimized memory allocation library (e.g. tcmalloc, jemalloc) "
        "\n################################# Basic usage ############################# \n"
        "\n 1. single instance\n"
        "\n   >>> python -m torch.backends.xeon.run_cpu python_script args \n"
        "\n2. multi-instance \n"
        "\n   >>> python -m torch.backends.xeon.run_cpu --ninstances xxx "
        "--ncores-per-instance xx python_script args\n"
        "\n############################################################################# \n",
        formatter_class=RawTextHelpFormatter,
    )
    create_args(parser)
    args = parser.parse_args()
    main(args)
````
- **EN**: This chunk continues `main` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

## Key Concepts / 关键概念

- **Backend configuration**
  - EN: Exposes optional-runtime toggles and capability checks that alter low-level execution behavior.
  - CN: 暴露可选运行时开关与能力检查，以改变底层执行行为。
- **_CPUinfo**
  - EN: `_CPUinfo` is one of the main symbols declared or implemented in this file.
  - CN: `_CPUinfo` 是本文件声明或实现的主要符号之一。
- **_Launcher**
  - EN: `_Launcher` is one of the main symbols declared or implemented in this file.
  - CN: `_Launcher` 是本文件声明或实现的主要符号之一。
- **Memory management**
  - EN: The file manages caches, pools, or allocation-related state exposed to Python.
  - CN: 该文件管理暴露给 Python 的缓存、内存池或分配相关状态。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch.distributed.elastic.multiprocessing`
- **Standard library / 标准库**: `glob`, `logging`, `os`, `platform`, `re`, `subprocess`, `sys`, `argparse`, `os.path`
- **Primary symbols in this file / 本文件核心符号**: `_CPUinfo`, `_Launcher`, `_add_memory_allocator_params`, `_add_multi_instance_params`, `_add_kmp_iomp_params`, `create_args`, `main`
