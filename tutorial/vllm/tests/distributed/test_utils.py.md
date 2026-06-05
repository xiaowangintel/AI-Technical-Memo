# test_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/distributed/test_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Utils behavior in the Distributed test area through focused pytest scenarios. It focuses on scenarios such as Cudadevicecountstatelesstestactor, CUDA Device Count Stateless, CPU Worker. / 该文件在 Distributed 测试域中，通过有针对性的 pytest 场景验证 Utils 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-17)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import socket

import pytest
import ray
import torch

import vllm.envs as envs
from vllm.distributed.device_communicators.pynccl import PyNcclCommunicator
from vllm.distributed.utils import StatelessProcessGroup
from vllm.platforms import current_platform
from vllm.utils.network_utils import get_open_port
from vllm.utils.system_utils import update_environment_variables

from ..utils import multi_gpu_test
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `socket`, `multiprocessing`, `pytest`, `ray`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Class: _CUDADeviceCountStatelessTestActor (lines 20-29)
```python
@ray.remote
class _CUDADeviceCountStatelessTestActor:
    def get_count(self):
        return current_platform.device_count()

    def set_cuda_visible_devices(self, cuda_visible_devices: str):
        update_environment_variables({"CUDA_VISIBLE_DEVICES": cuda_visible_devices})

    def get_cuda_visible_devices(self):
        return envs.CUDA_VISIBLE_DEVICES
```
**EN:** Groups related scenarios for Cudadevicecountstatelesstestactor.
**CN:** 该类把与 Cudadevicecountstatelesstestactor 相关的场景组织在一起。

### Test: test_cuda_device_count_stateless (lines 32-45)
```python
def test_cuda_device_count_stateless():
    """Test that cuda_device_count_stateless changes return value if
    CUDA_VISIBLE_DEVICES is changed."""
    if current_platform.is_rocm():
        pytest.skip("Skip for ROCm because Ray uses HIP_VISIBLE_DEVICES.")
    actor = _CUDADeviceCountStatelessTestActor.options(  # type: ignore
        num_gpus=2
    ).remote()
    assert len(sorted(ray.get(actor.get_cuda_visible_devices.remote()).split(","))) == 2
    assert ray.get(actor.get_count.remote()) == 2
    ray.get(actor.set_cuda_visible_devices.remote("0"))
    assert ray.get(actor.get_count.remote()) == 1
    ray.get(actor.set_cuda_visible_devices.remote(""))
    assert ray.get(actor.get_count.remote()) == 0
```
**EN:** Test that cuda_device_count_stateless changes return value if CUDA_VISIBLE_DEVICES is changed. The body exercises logic via `current_platform.is_rocm`, `_CUDADeviceCountStatelessTestActor.options(num_gpus=2).remote`, `ray.get` before asserting the expected outcome.
**CN:** 该测试用例验证 CUDA Device Count Stateless 在特定场景下的行为。 函数体会先通过 `current_platform.is_rocm`, `_CUDADeviceCountStatelessTestActor.options(num_gpus=2).remote`, `ray.get` 驱动目标逻辑，再断言预期结果。

### Helper: cpu_worker (lines 48-64)
```python
def cpu_worker(rank, WORLD_SIZE, port1, port2):
    pg1 = StatelessProcessGroup.create(
        host="127.0.0.1", port=port1, rank=rank, world_size=WORLD_SIZE
    )
    if rank <= 2:
        pg2 = StatelessProcessGroup.create(
            host="127.0.0.1", port=port2, rank=rank, world_size=3
        )
    data = torch.tensor([rank])
    data = pg1.broadcast_obj(data, src=2)
    assert data.item() == 2
    if rank <= 2:
        data = torch.tensor([rank + 1])
        data = pg2.broadcast_obj(data, src=2)
        assert data.item() == 3
        pg2.barrier()
    pg1.barrier()
```
**EN:** Implements a reusable helper for CPU Worker, reducing duplication across related tests. It coordinates operations such as `StatelessProcessGroup.create`, `torch.tensor`, `pg1.broadcast_obj`.
**CN:** 该辅助函数为 CPU Worker 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `StatelessProcessGroup.create`, `torch.tensor`, `pg1.broadcast_obj` 等操作。

### Helper: gpu_worker (lines 67-91)
```python
def gpu_worker(rank, WORLD_SIZE, port1, port2):
    torch.accelerator.set_device_index(rank)
    pg1 = StatelessProcessGroup.create(
        host="127.0.0.1", port=port1, rank=rank, world_size=WORLD_SIZE
    )
    pynccl1 = PyNcclCommunicator(pg1, device=rank)
    if rank <= 2:
        pg2 = StatelessProcessGroup.create(
            host="127.0.0.1", port=port2, rank=rank, world_size=3
        )
        pynccl2 = PyNcclCommunicator(pg2, device=rank)
    data = torch.tensor([rank]).cuda()
    pynccl1.all_reduce(data)
    pg1.barrier()
    torch.accelerator.synchronize()
    if rank <= 2:
        pynccl2.all_reduce(data)
        pg2.barrier()
        torch.accelerator.synchronize()
    item = data[0].item()
    print(f"rank: {rank}, item: {item}")
    if rank == 3:
        assert item == 6
    else:
        assert item == 18
```
**EN:** Implements a reusable helper for GPU Worker, reducing duplication across related tests. It coordinates operations such as `torch.accelerator.set_device_index`, `StatelessProcessGroup.create`, `PyNcclCommunicator`.
**CN:** 该辅助函数为 GPU Worker 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `torch.accelerator.set_device_index`, `StatelessProcessGroup.create`, `PyNcclCommunicator` 等操作。

### Helper: broadcast_worker (lines 94-103)
```python
def broadcast_worker(rank, WORLD_SIZE, port1, port2):
    pg1 = StatelessProcessGroup.create(
        host="127.0.0.1", port=port1, rank=rank, world_size=WORLD_SIZE
    )
    if rank == 2:
        pg1.broadcast_obj("secret", src=2)
    else:
        obj = pg1.broadcast_obj(None, src=2)
        assert obj == "secret"
    pg1.barrier()
```
**EN:** Implements a reusable helper for Broadcast Worker, reducing duplication across related tests. It coordinates operations such as `StatelessProcessGroup.create`, `pg1.barrier`, `pg1.broadcast_obj`.
**CN:** 该辅助函数为 Broadcast Worker 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `StatelessProcessGroup.create`, `pg1.barrier`, `pg1.broadcast_obj` 等操作。

### Helper: allgather_worker (lines 106-112)
```python
def allgather_worker(rank, WORLD_SIZE, port1, port2):
    pg1 = StatelessProcessGroup.create(
        host="127.0.0.1", port=port1, rank=rank, world_size=WORLD_SIZE
    )
    data = pg1.all_gather_obj(rank)
    assert data == list(range(WORLD_SIZE))
    pg1.barrier()
```
**EN:** Implements a reusable helper for Allgather Worker, reducing duplication across related tests. It coordinates operations such as `StatelessProcessGroup.create`, `pg1.all_gather_obj`, `pg1.barrier`.
**CN:** 该辅助函数为 Allgather Worker 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `StatelessProcessGroup.create`, `pg1.all_gather_obj`, `pg1.barrier` 等操作。

### Test: test_stateless_process_group (lines 115-141)
```python
@pytest.mark.skip(reason="This test is flaky and prone to hang.")
@multi_gpu_test(num_gpus=4)
@pytest.mark.parametrize(
    "worker", [cpu_worker, gpu_worker, broadcast_worker, allgather_worker]
)
def test_stateless_process_group(worker):
    port1 = get_open_port()
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.bind(("", port1))
        port2 = get_open_port()
    WORLD_SIZE = 4
    from multiprocessing import get_context

    ctx = get_context("fork")
    processes = []
    for i in range(WORLD_SIZE):
        rank = i
        processes.append(
            ctx.Process(target=worker, args=(rank, WORLD_SIZE, port1, port2))
        )
    for p in processes:
        p.start()
    for p in processes:
        p.join()
    for p in processes:
        assert not p.exitcode
    print("All processes finished.")
```
**EN:** Checks Stateless Process Group under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.skip`, `multi_gpu_test`, `pytest.mark.parametrize` before asserting the expected outcome.
**CN:** 该测试用例验证 Stateless Process Group 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.skip`, `multi_gpu_test`, `pytest.mark.parametrize` 驱动目标逻辑，再断言预期结果。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Distributed coordination / 分布式协同**
  - **EN:** The file validates multi-process or multi-worker coordination primitives such as collectives and rank-aware setup.
  - **CN:** 该文件验证多进程或多工作节点之间的协同原语，例如集合通信和基于 rank 的初始化。
- **Configuration / 配置**
  - **EN:** The file checks how options, environment variables, and defaults are interpreted.
  - **CN:** 该文件检查选项、环境变量与默认值如何被解析和生效。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `socket`, `multiprocessing`
- **Third-party / 第三方依赖**: `pytest`, `ray`, `torch`
- **vLLM internal / vLLM 内部依赖**: `vllm.envs`, `vllm.distributed.device_communicators.pynccl`, `vllm.distributed.utils`, `vllm.platforms`, `vllm.utils.network_utils`, `vllm.utils.system_utils`
- **Local test utilities / 本地测试辅助**: `..utils`
