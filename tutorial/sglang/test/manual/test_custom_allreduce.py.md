# test_custom_allreduce.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/test_custom_allreduce.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `custom allreduce` scenario in `test/manual`. It uses SGLang's shared test infrastructure to configure models or services and verify expected behavioral and API behavior. / 该手动测试模块覆盖 `test/manual` 中的 `custom allreduce` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的行为与接口表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Imports and shared helpers / 导入与共享辅助项
```python
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
)
from sglang.srt.server_args import ServerArgs, set_global_server_args_for_scheduler
from sglang.test.test_utils import CustomTestCase
```
**EN:** This range imports `os`, `random`, `socket` and `unittest`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. Environment variables are read here so the scenario adapts to the local machine and accelerator topology.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。

### Lines 24-34: Helper routines around get_open_port / 辅助例程
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

### Lines 37-56: Helper routines around multi_process_parallel / 辅助例程
```python
def multi_process_parallel(
    world_size: int,
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
        refs.append(test_target.remote(cls, world_size, rank, distributed_init_port))
    ray.get(refs)

    ray.shutdown()
```
**EN:** This range implements helper routine(s) `multi_process_parallel` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `init`, `get_open_port`, `append` and `remote`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 59-73: Class definition for TestCustomAllReduce / 类定义
```python
class TestCustomAllReduce(CustomTestCase):
    TEST_SIZES = [
        512,
        4096,
        32768,
        262144,
        2097152,
        16777216,
        33554432,
        67108864,
    ]  # 512B...32MB
    WORLD_SIZES = [2, 4, 6, 8]
    TEST_LOOP = 10

    @classmethod
```
**EN:** This range declares `TestCustomAllReduce`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 74-75: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        random.seed(42)  # keep the deterministic seed
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `seed`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 77-81: Test routines around test_graph_allreduce / 测试例程
```python
    def test_graph_allreduce(self):
        for world_size in self.WORLD_SIZES:
            if world_size > torch.cuda.device_count():
                continue
            multi_process_parallel(world_size, self, self.graph_allreduce)
```
**EN:** This range defines concrete test routine(s) `test_graph_allreduce`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `device_count` and `multi_process_parallel`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 83-83: Test routines around test_eager_allreduce / 测试例程
```python
    def test_eager_allreduce(self):
```
**EN:** This range defines concrete test routine(s) `test_eager_allreduce`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 84-87: Scenario logic / 场景逻辑
```python
        for world_size in self.WORLD_SIZES:
            if world_size > torch.cuda.device_count():
                continue
            multi_process_parallel(world_size, self, self.eager_allreduce)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `device_count` and `multi_process_parallel`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 88-89: Scenario logic / 场景逻辑
```python

    @ray.remote(num_gpus=1, max_calls=1)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `remote`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 90-108: Helper routines around graph_allreduce / 辅助例程
```python
    def graph_allreduce(self, world_size, rank, distributed_init_port):
        del os.environ["CUDA_VISIBLE_DEVICES"]
        device = torch.device(f"cuda:{rank}")
        torch.cuda.set_device(device)
        distributed_init_method = f"tcp://localhost:{distributed_init_port}"
        init_distributed_environment(
            world_size=world_size,
            rank=rank,
            distributed_init_method=distributed_init_method,
            local_rank=rank,
        )
        initialize_model_parallel(tensor_model_parallel_size=world_size)
        group = get_tensor_model_parallel_group().device_group

        # Set global server args to avoid "Global server args is not set yet!" error
        set_global_server_args_for_scheduler(ServerArgs(model_path="dummy"))

        # A small all_reduce for warmup.
        # this is needed because device communicators might be created lazily
```
**EN:** This range implements helper routine(s) `graph_allreduce` so setup, transformation, or validation logic can be reused cleanly. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `device`, `set_device`, `init_distributed_environment` and `initialize_model_parallel`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 109-114: Scenario logic / 场景逻辑
```python
        # (e.g. NCCL). This will ensure that the communicator is initialized
        # before any communication happens, so that this group can be used for
        # graph capture immediately.
        data = torch.zeros(1)
        data = data.to(device=device)
        torch.distributed.all_reduce(data, group=group)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `zeros`, `to` and `all_reduce`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 115-133: Scenario logic / 场景逻辑
```python
        torch.cuda.synchronize()
        del data

        for sz in self.TEST_SIZES:
            for dtype in [torch.float32, torch.float16, torch.bfloat16]:
                for _ in range(self.TEST_LOOP):
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
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `synchronize`, `graph_capture`, `randint` and `current_device`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 134-139: Scenario logic / 场景逻辑
```python
                            dtype=dtype,
                            device=torch.cuda.current_device(),
                        )
                        torch.cuda.synchronize()
                        graph = torch.cuda.CUDAGraph()
                        with torch.cuda.graph(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `current_device`, `synchronize`, `CUDAGraph` and `graph`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 140-150: Assertions and result checks / 断言与结果检查
```python
                            graph, stream=graph_capture_context.stream
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

### Lines 151-152: Scenario logic / 场景逻辑
```python

    @ray.remote(num_gpus=1, max_calls=1)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `remote`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 153-158: Helper routines around eager_allreduce / 辅助例程
```python
    def eager_allreduce(self, world_size, rank, distributed_init_port):
        del os.environ["CUDA_VISIBLE_DEVICES"]
        device = torch.device(f"cuda:{rank}")
        torch.cuda.set_device(device)
        distributed_init_method = f"tcp://localhost:{distributed_init_port}"
        init_distributed_environment(
```
**EN:** This range implements helper routine(s) `eager_allreduce` so setup, transformation, or validation logic can be reused cleanly. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `device`, `set_device` and `init_distributed_environment`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 159-177: Scenario logic / 场景逻辑
```python
            world_size=world_size,
            rank=rank,
            distributed_init_method=distributed_init_method,
            local_rank=rank,
        )
        initialize_model_parallel(tensor_model_parallel_size=world_size)
        group = get_tensor_model_parallel_group().device_group

        # Set global server args to avoid "Global server args is not set yet!" error
        set_global_server_args_for_scheduler(ServerArgs(model_path="dummy"))

        for sz in self.TEST_SIZES:
            for dtype in [torch.float32, torch.float16, torch.bfloat16]:
                for _ in range(self.TEST_LOOP):
                    inp1 = torch.randint(
                        1, 16, (sz,), dtype=dtype, device=torch.cuda.current_device()
                    )
                    out1 = tensor_model_parallel_all_reduce(inp1)
                    dist.all_reduce(inp1, group=group)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `initialize_model_parallel`, `get_tensor_model_parallel_group`, `set_global_server_args_for_scheduler` and `ServerArgs`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 178-178: Assertions and result checks / 断言与结果检查
```python
                    torch.testing.assert_close(out1, inp1)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assert_close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 179-182: Script entry point / 脚本入口
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
- **Project Modules / 项目模块**: `sglang.srt.distributed`, `sglang.srt.distributed.communication_op`, `sglang.srt.distributed.parallel_state`, `sglang.srt.server_args`, `sglang.test.test_utils`
