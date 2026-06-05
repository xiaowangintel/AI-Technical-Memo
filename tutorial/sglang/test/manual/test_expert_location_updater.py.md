# test_expert_location_updater.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/test_expert_location_updater.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `expert location updater` scenario in `test/manual`. It uses SGLang's shared test infrastructure to configure models or services and verify expected functional behavior. / 该手动测试模块覆盖 `test/manual` 中的 `expert location updater` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports and shared helpers / 导入与共享辅助项
```python
import os
import traceback
import unittest
from dataclasses import dataclass
from typing import List

import torch
import torch.distributed
import torch.multiprocessing as mp
from torch.multiprocessing import Process

from sglang.srt.eplb import expert_location_updater
from sglang.srt.utils import get_device
from sglang.test.test_utils import CustomTestCase, find_available_port
from sglang.utils import is_in_ci


@dataclass
```
**EN:** This range imports `os`, `traceback`, `unittest` and `dataclasses`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。

### Lines 19-23: Class definition for _TestInfo / 类定义
```python
class _TestInfo:
    nnodes: int
    num_logical_experts: int
    num_physical_experts: int
    num_repeat: int = 5000
```
**EN:** This range declares `_TestInfo`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 26-27: Class definition for TestExpertLocationUpdater / 类定义
```python
class TestExpertLocationUpdater(CustomTestCase):
    @classmethod
```
**EN:** This range declares `TestExpertLocationUpdater`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 28-29: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        mp.set_start_method("spawn", force=True)
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `set_start_method`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 31-44: Test routines around test_cpu / 测试例程
```python
    def test_cpu(self):
        self._test_common(device="cpu")
        self._test_core(
            num_gpus=32,
            device="cpu",
            infos=[
                _TestInfo(
                    nnodes=4,
                    num_logical_experts=256,
                    num_physical_experts=288,
                    num_repeat=10000,
                )
            ],
        )
```
**EN:** This range defines concrete test routine(s) `test_cpu`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `_test_common`, `_test_core` and `_TestInfo`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 46-50: Test routines around test_cpu_slow / 测试例程
```python
    def test_cpu_slow(self):
        if is_in_ci():
            return
        self._test_core(
            num_gpus=144,
```
**EN:** This range defines concrete test routine(s) `test_cpu_slow`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `is_in_ci` and `_test_core`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 51-60: Scenario logic / 场景逻辑
```python
            device="cpu",
            infos=[
                _TestInfo(
                    nnodes=18,
                    num_logical_experts=256,
                    num_physical_experts=288,
                    num_repeat=10000,
                )
            ],
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_TestInfo`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 62-65: Test routines around test_gpu / 测试例程
```python
    def test_gpu(self):
        if is_in_ci():
            return
        self._test_common(device=get_device())
```
**EN:** This range defines concrete test routine(s) `test_gpu`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `is_in_ci`, `_test_common` and `get_device`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 67-75: Helper routines around _test_common / 辅助例程
```python
    def _test_common(self, device):
        infos = []

        for nnodes in [1, 2, 4]:
            for num_logical_experts in [2, 5, 20, 256]:
                for num_physical_experts in [8, 16, 256, 288]:
                    if num_logical_experts > num_physical_experts:
                        continue
                    infos.append(
```
**EN:** This range implements helper routine(s) `_test_common` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `append`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 76-83: Scenario logic / 场景逻辑
```python
                        _TestInfo(
                            nnodes=nnodes,
                            num_logical_experts=num_logical_experts,
                            num_physical_experts=num_physical_experts,
                        )
                    )

        self._test_core(num_gpus=8, device=device, infos=infos)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_TestInfo` and `_test_core`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 85-100: Helper routines around _test_core / 辅助例程
```python
    def _test_core(
        self,
        num_gpus: int,
        device: str,
        infos: List[_TestInfo],
    ):
        master_port = find_available_port(23456)

        processes = []
        output_reader, output_writer = mp.Pipe(duplex=False)
        for rank in range(num_gpus):
            p = Process(
                target=_run_subprocess,
                kwargs=dict(
                    rank=rank,
                    num_gpus=num_gpus,
```
**EN:** This range implements helper routine(s) `_test_core` so setup, transformation, or validation logic can be reused cleanly. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `find_available_port`, `Pipe` and `Process`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 101-109: Scenario logic / 场景逻辑
```python
                    output_writer=output_writer,
                    master_port=master_port,
                    device=device,
                    infos=infos,
                ),
            )
            p.start()
            processes.append(p)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `start` and `append`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 110-116: Assertions and result checks / 断言与结果检查
```python
        for _ in range(num_gpus):
            self.assertTrue(
                output_reader.recv(), f"Subprocess has error, please see logs above."
            )

        for p in processes:
            p.join()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Assertions in this block enforce the intended outcome. Representative call sites include `assertTrue`, `recv` and `join`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 119-143: Helper routines around _run_subprocess / 辅助例程
```python
def _run_subprocess(
    rank: int,
    num_gpus: int,
    master_port: int,
    device: str,
    infos: List[_TestInfo],
    output_writer,
):
    try:
        os.environ["MASTER_ADDR"] = "localhost"
        os.environ["MASTER_PORT"] = str(master_port)

        torch.random.manual_seed(42)
        torch.distributed.init_process_group(
            rank=rank,
            world_size=num_gpus,
            backend={"cpu": "gloo", "cuda": None}[device],
        )
        if device == "cuda":
            torch.cuda.set_device(f"cuda:{rank}")
        if device == "xpu":
            torch.xpu.set_device(f"xpu:{rank}")

        for info in infos:
            _execute_test(info, rank=rank, num_gpus=num_gpus, device=device)
```
**EN:** This range implements helper routine(s) `_run_subprocess` so setup, transformation, or validation logic can be reused cleanly. Process management appears here to launch, monitor, or stop runtime components. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `manual_seed`, `init_process_group`, `set_device` and `_execute_test`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 144-152: Process control logic / 进程控制逻辑
```python

        execution_ok = True
    except Exception as e:
        print(f"subprocess[{rank=}] has error: {e}", flush=True)
        traceback.print_exc()
        execution_ok = False

    output_writer.send(execution_ok)
    output_writer.close()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `print_exc`, `send` and `close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 155-179: Helper routines around _execute_test / 辅助例程
```python
def _execute_test(info: _TestInfo, rank: int, num_gpus: int, device: str):
    if rank == 0:
        print(f"Test: {num_gpus=} {info=}", flush=True)

    assert info.num_physical_experts % num_gpus == 0
    num_local_physical_experts = info.num_physical_experts // num_gpus
    assert num_gpus % info.nnodes == 0
    num_gpu_per_node = num_gpus // info.nnodes

    def _create_routed_experts_weights(physical_to_logical_map):
        local_logical_expert_ids = physical_to_logical_map[
            rank * num_local_physical_experts : (rank + 1) * num_local_physical_experts
        ].cpu()
        return [
            local_logical_expert_ids.to(device).clone(),
            torch.tensor(
                [
                    [local_logical_expert_id * 10, local_logical_expert_id * 100]
                    for local_logical_expert_id in local_logical_expert_ids.tolist()
                ],
                device=device,
            ),
        ]

    def _create_physical_to_logical_map():
```
**EN:** This range implements helper routine(s) `_execute_test`, `_create_routed_experts_weights` and `_create_physical_to_logical_map` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `cpu`, `to`, `clone` and `tensor`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 180-204: Assertions and result checks / 断言与结果检查
```python
        if rank == 0:
            ans = torch.concat(
                [
                    torch.arange(0, info.num_logical_experts),
                    torch.randint(
                        0,
                        info.num_logical_experts,
                        (info.num_physical_experts - info.num_logical_experts,),
                    ),
                ]
            )
            ans = ans[torch.randperm(ans.shape[0])]
        else:
            ans = torch.empty((info.num_physical_experts,), dtype=torch.int64)

        assert ans.dtype == torch.int64 and ans.shape == (info.num_physical_experts,)
        ans = ans.to(device)
        torch.distributed.broadcast(ans, src=0)

        return ans.cpu()

    physical_to_logical_map = _create_physical_to_logical_map()
    routed_experts_weights = _create_routed_experts_weights(physical_to_logical_map)

    for i in range(info.num_repeat):
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `concat`, `arange`, `randint` and `randperm`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 205-229: Scenario logic / 场景逻辑
```python
        if rank == 0 and ((i % 500 == 0) or (i == info.num_repeat - 1)):
            print(f"Step {i}/{info.num_repeat}", flush=True)

        new_physical_to_logical_map = _create_physical_to_logical_map()
        expect_new_weights = _create_routed_experts_weights(new_physical_to_logical_map)

        output_logs = expert_location_updater.update_expert_weights_single_layer(
            routed_experts_weights=routed_experts_weights,
            temp_buffers=expert_location_updater.create_temp_buffers(
                routed_experts_weights
            ),
            old_physical_to_logical_map=physical_to_logical_map.tolist(),
            new_physical_to_logical_map=new_physical_to_logical_map.tolist(),
            num_local_physical_experts=num_local_physical_experts,
            num_gpu_per_node=num_gpu_per_node,
            rank=rank,
            debug=True,
        )

        local_has_error = not all(
            torch.all(x == y)
            for x, y in zip(routed_experts_weights, expect_new_weights, strict=True)
        )
        global_has_error = torch.tensor(local_has_error, device=device)
        torch.distributed.all_reduce(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `and`, `or`, `_create_physical_to_logical_map` and `_create_routed_experts_weights`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 230-254: Assertions and result checks / 断言与结果检查
```python
            global_has_error, op=torch.distributed.ReduceOp.MAX
        )

        if global_has_error.cpu().item():
            output_logs_str = "\n".join(output_logs)
            local_message = (
                f"===================== rank {rank} ============================\n"
                f"{num_gpus=} {info=}\n"
                f"{routed_experts_weights[0].tolist()=}\n"
                f"{expect_new_weights[0].tolist()=}\n"
                f"{physical_to_logical_map.tolist()=}\n"
                f"{new_physical_to_logical_map.tolist()=}\n"
                f"===logs===\n"
                f"{output_logs_str}\n"
                f"==============================================================\n"
            )

            global_messages = ([None] * num_gpus) if rank == 0 else None
            torch.distributed.gather_object(local_message, global_messages, dst=0)

            if rank == 0:
                print("\n\n".join(global_messages), flush=True)
            raise AssertionError(f"Error happens, see logs above")

        physical_to_logical_map = new_physical_to_logical_map
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `cpu`, `item`, `join` and `tolist`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 255-258: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Multi-GPU orchestration / 多 GPU 编排
- Process lifecycle management / 进程生命周期管理
- Environment-aware configuration / 环境感知配置

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `dataclasses`, `os`, `traceback`, `typing`, `unittest`
- **Third-party / 第三方库**: `torch`, `torch.distributed`, `torch.multiprocessing`
- **Project Modules / 项目模块**: `sglang.srt.eplb`, `sglang.srt.utils`, `sglang.test.test_utils`, `sglang.utils`
