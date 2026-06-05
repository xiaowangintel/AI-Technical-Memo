# mindspore_runner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/model_executor/mindspore_runner.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the model execution and graph runners part of the SRT runtime and implements logic centered on `mindspore_runner`. The module docstring frames it as: "ms_runner launch MindSpore distributed modules." / 该模块属于 SRT 运行时的模型执行与图运行器部分，主要实现围绕 `mindspore_runner` 的逻辑。 它对外提供的主要入口包括 `_Tmp`, `_get_host_and_ip`, `run_scheduler_init`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Module imports, constants, and setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the SGLang project
"""ms_runner launch MindSpore distributed modules."""

import logging
import multiprocessing as mp
import os
import sys
from pathlib import Path

import mindspore as ms
import torch
from mindspore._c_expression import GroupOptions
from mindspore.communication import create_group

from sglang.srt.distributed.parallel_state import _groups

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断。

### Lines 21-21: Class _Tmp
```python
class _Tmp:
```
**EN:** This range introduces `_Tmp` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `_Tmp`，并定义其后续方法依赖的结构或元数据。

### Lines 22-23: Method _Tmp.__init__
```python
    def __init__(self):
        self.sched_p = None
```
**EN:** This callable implements `_Tmp.__init__` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `_Tmp.__init__`，主要用于初始化实例状态与默认值。

### Lines 25-26: Method _Tmp.set_sched_process
```python
    def set_sched_process(self, p):
        self.sched_p = p
```
**EN:** This callable implements `_Tmp.set_sched_process`. It takes `p` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `_Tmp.set_sched_process`。它接收 `p`，主要用于将配置写入可变状态。

### Lines 28-30: Method _Tmp.__del__
```python
    def __del__(self):
        if self.sched_p:
            self.sched_p.kill()
```
**EN:** This callable implements `_Tmp.__del__` and mainly implements del.
**CN:** 这一可调用对象实现了 `_Tmp.__del__`，主要用于实现 del 相关逻辑。

### Lines 31-35: Module-level constants and helpers
```python


_tmp = _Tmp()


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 36-47: Function _get_host_and_ip
```python
def _get_host_and_ip(distributed_init_method):
    try:
        _, ip_str, port_str = distributed_init_method.split(":")
        ip = ip_str.split("/")[-1]
        port = int(port_str)
    except Exception as e:
        raise RuntimeError(
            "Cannot get host and port information from %s, error: %s!"
            % (distributed_init_method, str(e))
        )

    return ip, port
```
**EN:** This callable implements `_get_host_and_ip`. It takes `distributed_init_method` and mainly retrieves a value or derived view. In this range it sets up imports and shared symbols; performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `_get_host_and_ip`。它接收 `distributed_init_method`，主要用于获取某个值或派生视图。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查。

### Lines 50-65: Function run_scheduler_init
```python
def run_scheduler_init(rank, local_rank, world_size, master_addr, master_port):
    with open(str(Path() / "schedule.log"), "w") as scheduler_f:
        # For Python outputs.
        sys.stdout = scheduler_f
        sys.stderr = scheduler_f
        # For C++ outputs.
        os.dup2(scheduler_f.fileno(), 1)
        os.dup2(scheduler_f.fileno(), 2)
        os.environ["DEVICE_ID"] = str(local_rank)
        os.environ["MS_WORKER_NUM"] = str(world_size)
        os.environ["MS_ROLE"] = "MS_SCHED"
        os.environ["MS_NODE_ID"] = str(rank)
        os.environ["MS_SCHED_HOST"] = str(master_addr)
        os.environ["MS_SCHED_PORT"] = str(master_port)
        # This function is blocked until the whole cluster exits.
        ms.communication.init()
```
**EN:** This callable implements `run_scheduler_init`. It takes `rank`, `local_rank`, `world_size`, `master_addr` and mainly executes the main workflow. In this range it reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `run_scheduler_init`。它接收 `rank`, `local_rank`, `world_size`, `master_addr`，主要用于执行主要流程。 在这一范围内，它会读取环境变量驱动的配置。

### Lines 68-88: Function set_ms_parallel_env
```python
def set_ms_parallel_env(rank, local_rank, world_size, init_method):
    master_addr, master_port = _get_host_and_ip(init_method)
    # change port avoiding port conflicts with torch
    master_port = master_port + 35 if master_port < 65500 else master_port - 35
    if not os.getenv("MS_ROLE"):
        if rank == 0:
            # Create a subprocess for scheduler of MindSpore, just for internal collaboration, not for collective communication
            sched_p = mp.Process(
                target=run_scheduler_init,
                args=(rank, local_rank, world_size, master_addr, master_port),
            )
            sched_p.start()
            global _tmp
            _tmp.set_sched_process(sched_p)

        os.environ["DEVICE_ID"] = str(local_rank)
        os.environ["MS_WORKER_NUM"] = str(world_size)
        os.environ["MS_ROLE"] = "MS_WORKER"
        os.environ["MS_NODE_ID"] = str(rank)
        os.environ["MS_SCHED_HOST"] = str(master_addr)
        os.environ["MS_SCHED_PORT"] = str(master_port)
```
**EN:** This callable implements `set_ms_parallel_env`. It takes `rank`, `local_rank`, `world_size`, `init_method` and mainly applies configuration to mutable state. In this range it reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `set_ms_parallel_env`。它接收 `rank`, `local_rank`, `world_size`, `init_method`，主要用于将配置写入可变状态。 在这一范围内，它会读取环境变量驱动的配置。

### Lines 91-105: Function reuse_hccl_comm
```python
def reuse_hccl_comm():
    for group_name, group in _groups.items():
        # Torch ProcessGroupHccl
        device_group = group().device_group
        hccl_comm_handle = device_group._get_backend(torch.device("npu")).get_hccl_comm(
            group().local_rank
        )
        logger.info(
            f"MindSpore reuse torch group: {device_group}, group_name: {group_name}, local rank: {group().local_rank},"
            f"hccl communicator handle: {hex(hccl_comm_handle)}",
        )
        # Create MS communication group by hccl comm handle to reuse Torch group.
        group_options = GroupOptions()
        group_options.hccl_config = {"hccl_comm": hccl_comm_handle}
        create_group(group_name, group().ranks, group_options)
```
**EN:** This callable implements `reuse_hccl_comm` and mainly implements reuse hccl comm. In this range it emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `reuse_hccl_comm`，主要用于实现 reuse hccl comm 相关逻辑。 在这一范围内，它会输出日志以便诊断；协调分布式通信。

### Lines 108-120: Function init_ms_distributed
```python
def init_ms_distributed(world_size, rank, local_rank, server_args, port):
    if server_args.dist_init_addr:
        dist_init_method = f"tcp://{server_args.dist_init_addr}"
    else:
        dist_init_method = f"tcp://{server_args.host}:{port}"
    set_ms_parallel_env(rank, local_rank, world_size, dist_init_method)

    ms.set_context(infer_boost="on", jit_level="O0")
    ms.set_context(mode=ms.context.PYNATIVE_MODE)
    ms.set_device("Ascend", local_rank)
    ms.communication.init("hccl")
    # After distributed job is initialized, reuse hccl comms for MindSpore.
    reuse_hccl_comm()
```
**EN:** This callable implements `init_ms_distributed`. It takes `world_size`, `rank`, `local_rank`, `server_args` and mainly implements init ms distributed.
**CN:** 这一可调用对象实现了 `init_ms_distributed`。它接收 `world_size`, `rank`, `local_rank`, `server_args`，主要用于实现 init ms distributed 相关逻辑。

## Key Concepts / 关键概念
- `_Tmp`: core class or state container / 核心类或状态容器
- `_get_host_and_ip`: retrieves a value or derived view / 获取某个值或派生视图
- `run_scheduler_init`: executes the main workflow / 执行主要流程
- `set_ms_parallel_env`: applies configuration to mutable state / 将配置写入可变状态
- `reuse_hccl_comm`: implements reuse hccl comm / 实现 reuse hccl comm 相关逻辑
- `init_ms_distributed`: implements init ms distributed / 实现 init ms distributed 相关逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `multiprocessing`, `os`, `sys`, `pathlib`
- **Third-party / 第三方**: `mindspore`, `torch`, `mindspore._c_expression`, `mindspore.communication`
- **Internal modules / 内部模块**: `sglang.srt.distributed.parallel_state`
