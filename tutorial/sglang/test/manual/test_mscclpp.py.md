# test_mscclpp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/test_mscclpp.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `mscclpp` scenario in `test/manual`. It uses SGLang's shared test infrastructure to configure models or services and verify expected behavioral and API behavior. / 该手动测试模块覆盖 `test/manual` 中的 `mscclpp` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的行为与接口表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: Environment-driven configuration / 环境驱动配置
```python
"""For Now, MSCCL is only supported on TP16 and TP8 case

if [[ $RANK -eq 0  ]]; then
    ray start --block --head --port=6379 &
    python3 test_mscclpp.py;
else
    ray start --block --address=${MASTER_ADDR}:6379;
fi
"""

import os
import random
import socket
import unittest
from typing import Any

import ray
import torch
import torch.distributed as dist

from sglang.srt.distributed import init_distributed_environment
from sglang.srt.distributed.communication_op import (  # noqa
    tensor_model_parallel_all_reduce,
)
from sglang.srt.distributed.parallel_state import (
    get_tensor_model_parallel_group,
    graph_capture,
    initialize_model_parallel,
    set_custom_all_reduce,
    set_mscclpp_all_reduce,
)
from sglang.test.test_utils import CustomTestCase
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Environment variables are read here so the scenario adapts to the local machine and accelerator topology.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。

### Lines 35-45: Helper routines around get_open_port / 辅助例程
```python
def get_open_port() -> int:
    # try ipv4
    try:
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
            s.bind(("", 0))
            return s.getsockname()[1]
    except OSError:
        # try ipv6
        with socket.socket(socket.AF_INET6, socket.SOCK_STREAM) as s:
            s.bind(("", 0))
            return s.getsockname()[1]
```
**EN:** This range implements helper routine(s) `get_open_port` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `socket`, `bind` and `getsockname`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 48-72: Helper routines around multi_process_parallel / 辅助例程
```python
def multi_process_parallel(
    world_size: int,
    master_addr: str,
    cls: Any,
    test_target: Any,
) -> None:

    # Using ray helps debugging the error when it failed
    # as compared to multiprocessing.
    # NOTE: We need to set working_dir for distributed tests,
    # otherwise we may get import errors on ray workers

    ray.init(log_to_driver=True)

    distributed_init_port = get_open_port()
    refs = []
    for rank in range(world_size):
        refs.append(
            test_target.remote(
                cls, world_size, master_addr, rank, distributed_init_port
            )
        )
    ray.get(refs)

    ray.shutdown()
```
**EN:** This range implements helper routine(s) `multi_process_parallel` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `init`, `get_open_port`, `append` and `remote`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 75-76: Class definition for TestMSCCLAllReduce / 类定义
```python
class TestMSCCLAllReduce(CustomTestCase):
    @classmethod
```
**EN:** This range declares `TestMSCCLAllReduce`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 77-85: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        random.seed(42)
        # 1KB to 1MB
        cls.test_sizes = [512, 4096, 32768, 262144, 524288]
        cls.world_sizes = [8]
        TEST_TP16 = int(os.getenv("SGL_MSCCLPP_TEST_TP16", "0"))
        if TEST_TP16:
            cls.world_sizes = [16]
        cls.test_loop = 10
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `seed` and `getenv`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 87-94: Test routines around test_graph_allreduce / 测试例程
```python
    def test_graph_allreduce(self):
        TEST_MASTER_ADDR = os.getenv("SGL_MSCCLPP_TEST_MASTER_ADDR", "localhost")
        for world_size in self.world_sizes:
            if world_size not in [8, 16]:
                continue
            multi_process_parallel(
                world_size, TEST_MASTER_ADDR, self, self.graph_allreduce
            )
```
**EN:** This range defines concrete test routine(s) `test_graph_allreduce`. The logic drives the target scenario and encodes the expected acceptance criteria. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `getenv` and `multi_process_parallel`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 96-99: Test routines around test_eager_allreduce / 测试例程
```python
    def test_eager_allreduce(self):
        TEST_MASTER_ADDR = os.getenv("SGL_MSCCLPP_TEST_MASTER_ADDR", "localhost")
        for world_size in self.world_sizes:
            if world_size not in [8, 16]:
```
**EN:** This range defines concrete test routine(s) `test_eager_allreduce`. The logic drives the target scenario and encodes the expected acceptance criteria. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `getenv`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 100-103: Scenario logic / 场景逻辑
```python
                continue
            multi_process_parallel(
                world_size, TEST_MASTER_ADDR, self, self.eager_allreduce
            )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `multi_process_parallel`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 104-105: Scenario logic / 场景逻辑
```python

    @ray.remote(num_gpus=1, max_calls=1)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `remote`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 106-124: Helper routines around graph_allreduce / 辅助例程
```python
    def graph_allreduce(self, world_size, master_addr, rank, distributed_init_port):
        del os.environ["CUDA_VISIBLE_DEVICES"]
        device = torch.device(f"cuda:{rank % torch.cuda.device_count()}")
        torch.cuda.set_device(device)
        distributed_init_method = f"tcp://{master_addr}:{distributed_init_port}"
        set_mscclpp_all_reduce(True)
        set_custom_all_reduce(False)
        init_distributed_environment(
            world_size=world_size,
            rank=rank,
            distributed_init_method=distributed_init_method,
            local_rank=rank % torch.cuda.device_count(),
        )
        initialize_model_parallel(tensor_model_parallel_size=world_size)
        group = get_tensor_model_parallel_group().device_group

        # A small all_reduce for warmup.
        # this is needed because device communicators might be created lazily
        # (e.g. NCCL). This will ensure that the communicator is initialized
```
**EN:** This range implements helper routine(s) `graph_allreduce` so setup, transformation, or validation logic can be reused cleanly. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `device`, `device_count`, `set_device` and `set_mscclpp_all_reduce`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 125-130: Scenario logic / 场景逻辑
```python
        # before any communication happens, so that this group can be used for
        # graph capture immediately.
        data = torch.zeros(1)
        data = data.to(device=device)
        torch.distributed.all_reduce(data, group=group)
        torch.cuda.synchronize()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `zeros`, `to`, `all_reduce` and `synchronize`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 131-149: Scenario logic / 场景逻辑
```python
        del data

        for sz in self.test_sizes:
            for dtype in [torch.float32, torch.float16, torch.bfloat16]:
                for _ in range(self.test_loop):
                    with graph_capture() as graph_capture_context:
                        # use integers so result matches NCCL exactly
                        inp1 = torch.randint(
                            1,
                            16,
                            (sz,),
                            dtype=dtype,
                            device=torch.cuda.current_device(),
                        )
                        inp2 = torch.randint(
                            1,
                            16,
                            (sz,),
                            dtype=dtype,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `graph_capture`, `randint` and `current_device`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 150-155: Scenario logic / 场景逻辑
```python
                            device=torch.cuda.current_device(),
                        )
                        torch.cuda.synchronize()
                        graph = torch.cuda.CUDAGraph()
                        with torch.cuda.graph(
                            graph, stream=graph_capture_context.stream
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `current_device`, `synchronize`, `CUDAGraph` and `graph`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 156-165: Assertions and result checks / 断言与结果检查
```python
                        ):
                            out1 = tensor_model_parallel_all_reduce(inp1)
                            # the input buffer is immediately modified to test
                            # synchronization
                            dist.all_reduce(inp1, group=group)
                            out2 = tensor_model_parallel_all_reduce(inp2)
                            dist.all_reduce(inp2, group=group)
                    graph.replay()
                    torch.testing.assert_close(out1, inp1)
                    torch.testing.assert_close(out2, inp2)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `tensor_model_parallel_all_reduce`, `all_reduce`, `replay` and `assert_close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 166-167: Scenario logic / 场景逻辑
```python

    @ray.remote(num_gpus=1, max_calls=1)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `remote`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 168-174: Helper routines around eager_allreduce / 辅助例程
```python
    def eager_allreduce(self, world_size, master_addr, rank, distributed_init_port):
        del os.environ["CUDA_VISIBLE_DEVICES"]
        device = torch.device(f"cuda:{rank % torch.cuda.device_count()}")
        torch.cuda.set_device(device)
        distributed_init_method = f"tcp://{master_addr}:{distributed_init_port}"
        set_mscclpp_all_reduce(True)
        set_custom_all_reduce(False)
```
**EN:** This range implements helper routine(s) `eager_allreduce` so setup, transformation, or validation logic can be reused cleanly. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `device`, `device_count`, `set_device` and `set_mscclpp_all_reduce`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 175-192: Assertions and result checks / 断言与结果检查
```python
        init_distributed_environment(
            world_size=world_size,
            rank=rank,
            distributed_init_method=distributed_init_method,
            local_rank=rank,
        )
        initialize_model_parallel(tensor_model_parallel_size=world_size)
        group = get_tensor_model_parallel_group().device_group

        for sz in self.test_sizes:
            for dtype in [torch.float32, torch.float16, torch.bfloat16]:
                for _ in range(self.test_loop):
                    inp1 = torch.randint(
                        1, 16, (sz,), dtype=dtype, device=torch.cuda.current_device()
                    )
                    out1 = tensor_model_parallel_all_reduce(inp1)
                    dist.all_reduce(inp1, group=group)
                    torch.testing.assert_close(out1, inp1)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Assertions in this block enforce the intended outcome. Representative call sites include `init_distributed_environment`, `initialize_model_parallel`, `get_tensor_model_parallel_group` and `randint`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 193-196: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Multi-GPU orchestration / 多 GPU 编排
- Environment-aware configuration / 环境感知配置
- Streaming responses / 流式响应

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `os`, `random`, `socket`, `typing`, `unittest`
- **Third-party / 第三方库**: `ray`, `torch`, `torch.distributed`
- **Project Modules / 项目模块**: `sglang.srt.distributed`, `sglang.srt.distributed.communication_op`, `sglang.srt.distributed.parallel_state`, `sglang.test.test_utils`
