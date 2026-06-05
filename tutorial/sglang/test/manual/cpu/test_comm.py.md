# test_comm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/cpu/test_comm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `comm` scenario in `test/manual/cpu`. It uses SGLang's shared test infrastructure to configure models or services and verify expected functional behavior. / 该手动测试模块覆盖 `test/manual/cpu` 中的 `comm` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Imports and shared helpers / 导入与共享辅助项
```python
import copy
import multiprocessing
import os
import traceback
import unittest
from multiprocessing import Process

import torch
import torch.distributed as dist
import torch.multiprocessing as mp

from sglang.test.test_utils import CustomTestCase, find_available_port
```
**EN:** This range imports `copy`, `multiprocessing`, `os` and `traceback`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。

### Lines 15-38: Helper routines around run_distributed_test / 辅助例程
```python
def run_distributed_test(rank, world_size, master_port, output_writer, fn):
    try:
        os.environ["RANK"] = str(rank)
        os.environ["WORLD_SIZE"] = str(world_size)
        os.environ["MASTER_ADDR"] = "localhost"
        os.environ["MASTER_PORT"] = str(master_port)
        os.environ["LOCAL_SIZE"] = str(world_size)

        dist.init_process_group("gloo", rank=rank, world_size=world_size)
        torch.ops.sgl_kernel.initialize(world_size, rank)

        fn(rank, world_size)

        execution_ok = True
    except Exception as e:
        print(f"subprocess[{rank=}] has error: {e}", flush=True)
        traceback.print_exc()
        execution_ok = False

    output_writer.send(execution_ok)
    output_writer.close()

    if dist.is_initialized():
        dist.destroy_process_group()
```
**EN:** This range implements helper routine(s) `run_distributed_test` so setup, transformation, or validation logic can be reused cleanly. Process management appears here to launch, monitor, or stop runtime components. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `init_process_group`, `initialize`, `fn` and `print_exc`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 41-50: Helper routines around all_reduce_fn / 辅助例程
```python
def all_reduce_fn(rank, world_size):
    op = dist.ReduceOp.SUM
    for dtype in [torch.float32, torch.bfloat16, torch.float16]:
        tensor = torch.randn(2, 10, dtype=dtype)
        tensor_shm = copy.deepcopy(tensor)

        dist.all_reduce(tensor, op=op)
        torch.ops.sgl_kernel.shm_allreduce(tensor_shm, op)

        torch.testing.assert_close(tensor, tensor_shm)
```
**EN:** This range implements helper routine(s) `all_reduce_fn` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `randn`, `deepcopy`, `all_reduce` and `shm_allreduce`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 53-77: Helper routines around all_gather_fn / 辅助例程
```python
def all_gather_fn(rank, world_size):
    dim = -1

    for dtype in [torch.float32, torch.bfloat16, torch.float16]:
        tensor = torch.randn(2, 10, dtype=dtype)

        if dim < 0:
            # Convert negative dim to positive.
            dim += tensor.dim()

        input_size = tensor.size()
        output_size = (input_size[0] * world_size,) + input_size[1:]
        output_tensor = torch.empty(
            output_size, dtype=tensor.dtype, device=tensor.device
        )
        dist.all_gather_into_tensor(output_tensor, tensor)
        output_tensor = output_tensor.reshape((world_size,) + input_size)
        output_tensor = output_tensor.movedim(0, dim)
        output_tensor = output_tensor.reshape(
            input_size[:dim] + (world_size * input_size[dim],) + input_size[dim + 1 :]
        )

        output_shm = torch.ops.sgl_kernel.shm_allgather(tensor, dim)

        torch.testing.assert_close(output_tensor, output_shm)
```
**EN:** This range implements helper routine(s) `all_gather_fn` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `randn`, `dim`, `size` and `empty`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 80-80: Class definition for TestComm / 类定义
```python
class TestComm(CustomTestCase):
```
**EN:** This range declares `TestComm`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 81-104: Helper routines around _spawn_and_check / 辅助例程
```python
    def _spawn_and_check(self, fn, world_size=2):
        mp.set_start_method("spawn", force=True)
        master_port = find_available_port(23456)

        processes = []
        output_reader, output_writer = multiprocessing.Pipe(duplex=False)

        for rank in range(world_size):
            p = Process(
                target=run_distributed_test,
                kwargs=dict(
                    rank=rank,
                    world_size=world_size,
                    master_port=master_port,
                    output_writer=output_writer,
                    fn=fn,
                ),
            )
            p.start()
            processes.append(p)

        for _ in range(world_size):
            self.assertTrue(output_reader.recv(), "Subprocess fail. Check logs above.")
```
**EN:** This range implements helper routine(s) `_spawn_and_check` so setup, transformation, or validation logic can be reused cleanly. Process management appears here to launch, monitor, or stop runtime components. Assertions in this block enforce the intended outcome. Representative call sites include `set_start_method`, `find_available_port`, `Pipe` and `Process`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 105-105: Scenario logic / 场景逻辑
```python
        for p in processes:
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 106-106: Scenario logic / 场景逻辑
```python
            p.join()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `join`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 108-109: Test routines around test_all_reduce / 测试例程
```python
    def test_all_reduce(self):
        self._spawn_and_check(all_reduce_fn)
```
**EN:** This range defines concrete test routine(s) `test_all_reduce`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `_spawn_and_check`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 111-112: Test routines around test_all_gather / 测试例程
```python
    def test_all_gather(self):
        self._spawn_and_check(all_gather_fn)
```
**EN:** This range defines concrete test routine(s) `test_all_gather`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `_spawn_and_check`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 113-116: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Process lifecycle management / 进程生命周期管理
- Environment-aware configuration / 环境感知配置

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `copy`, `multiprocessing`, `os`, `traceback`, `unittest`
- **Third-party / 第三方库**: `torch`, `torch.distributed`, `torch.multiprocessing`
- **Project Modules / 项目模块**: `sglang.test.test_utils`
