# test_quick_allreduce.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/test_quick_allreduce.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `quick allreduce` scenario in `test/manual`. It uses SGLang's shared test infrastructure to configure models or services and verify expected behavioral and API behavior. / 该手动测试模块覆盖 `test/manual` 中的 `quick allreduce` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的行为与接口表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: Imports and shared helpers / 导入与共享辅助项
```python
import multiprocessing
import os
import random
import socket
import unittest
from typing import Any

import ray
import torch
import torch.distributed as dist

import sglang.srt.distributed.device_communicators.custom_all_reduce_ops as ops
from sglang.srt.distributed import init_distributed_environment
from sglang.srt.distributed.communication_op import (  # noqa
    tensor_model_parallel_all_reduce,
)
from sglang.srt.distributed.device_communicators.quick_all_reduce import (
    qr_rocm_arch_available,
)
from sglang.srt.distributed.parallel_state import (
    get_tensor_model_parallel_group,
    graph_capture,
    initialize_model_parallel,
)
from sglang.test.test_utils import CustomTestCase

torch.manual_seed(42)
random.seed(44)  # keep the deterministic seed
```
**EN:** This range imports `multiprocessing`, `os`, `random` and `socket`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `manual_seed` and `seed`.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 31-41: Helper routines around get_open_port / 辅助例程
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

### Lines 44-63: Helper routines around multi_process_parallel / 辅助例程
```python
def multi_process_parallel(
    world_size: int, cls: Any, test_target: Any, quant_mode: str
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
            test_target.remote(cls, world_size, rank, distributed_init_port, quant_mode)
        )
    ray.get(refs)

    ray.shutdown()
```
**EN:** This range implements helper routine(s) `multi_process_parallel` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `init`, `get_open_port`, `append` and `remote`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 66-84: Class definition for TestQuickAllReduce / 类定义
```python
class TestQuickAllReduce(CustomTestCase):
    TEST_SIZES = [
        2 * 1024 * 1024,
        4 * 1024 * 1024,
        8 * 1024 * 1024,
        16 * 1024 * 1024,
        32 * 1024 * 1024,
    ]
    TEST_LOOP = 5
    # Too many configurations can lead to a test grid that is too large
    # The tp takes too long to boot,let's just choose 4 out of 12 configurations
    # WORLD_SIZES = [2, 4, 8]
    # QUANT_MODE = ["FP", "INT8", "INT6", "INT4"]
    QUANT_MODE_WORLD_SIZE_PART = [["FP", 8], ["INT4", 4], ["INT8", 2], ["INT6", 2]]

    @unittest.skipIf(
        not qr_rocm_arch_available(),
        "Only test Quick AllReduce on ROCm architectures >= gfx94*",
    )
```
**EN:** This range declares `TestQuickAllReduce`, which organizes the scenario as a reusable test-oriented class. Representative call sites include `skipIf` and `qr_rocm_arch_available`.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 85-90: Test routines around test_graph_allreduce / 测试例程
```python
    def test_graph_allreduce(self):
        for quant_mode_world_size_part in self.QUANT_MODE_WORLD_SIZE_PART:
            quant_mode = quant_mode_world_size_part[0]
            world_size = quant_mode_world_size_part[1]
            if world_size > torch.cuda.device_count():
                continue
```
**EN:** This range defines concrete test routine(s) `test_graph_allreduce`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `device_count`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 91-91: Scenario logic / 场景逻辑
```python
            multi_process_parallel(world_size, self, self.graph_allreduce, quant_mode)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `multi_process_parallel`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 92-96: Scenario logic / 场景逻辑
```python

    @unittest.skipIf(
        not qr_rocm_arch_available(),
        "Only test Quick AllReduce on ROCm architectures >= gfx94*",
    )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skipIf` and `qr_rocm_arch_available`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 97-103: Test routines around test_eager_allreduce / 测试例程
```python
    def test_eager_allreduce(self):
        for quant_mode_world_size_part in self.QUANT_MODE_WORLD_SIZE_PART:
            quant_mode = quant_mode_world_size_part[0]
            world_size = quant_mode_world_size_part[1]
            if world_size > torch.cuda.device_count():
                continue
            multi_process_parallel(world_size, self, self.eager_allreduce, quant_mode)
```
**EN:** This range defines concrete test routine(s) `test_eager_allreduce`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `device_count` and `multi_process_parallel`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 104-105: Scenario logic / 场景逻辑
```python

    @ray.remote(num_gpus=1, max_calls=1)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `remote`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 106-115: Helper routines around graph_allreduce / 辅助例程
```python
    def graph_allreduce(self, world_size, rank, distributed_init_port, quant_mode):
        os.environ.pop("CUDA_VISIBLE_DEVICES", None)
        os.environ["ROCM_QUICK_REDUCE_QUANTIZATION"] = quant_mode
        os.environ["ROCM_QUICK_REDUCE_CAST_BF16_TO_FP16"] = "0"
        device = torch.device(f"cuda:{rank}")
        torch.cuda.set_device(device)
        distributed_init_method = f"tcp://localhost:{distributed_init_port}"
        init_distributed_environment(
            world_size=world_size,
            rank=rank,
```
**EN:** This range implements helper routine(s) `graph_allreduce` so setup, transformation, or validation logic can be reused cleanly. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `pop`, `device`, `set_device` and `init_distributed_environment`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 116-130: Scenario logic / 场景逻辑
```python
            distributed_init_method=distributed_init_method,
            local_rank=rank,
        )
        initialize_model_parallel(tensor_model_parallel_size=world_size)
        group = get_tensor_model_parallel_group().device_group

        # A small all_reduce for warmup.
        # this is needed because device communicators might be created lazily
        # (e.g. NCCL). This will ensure that the communicator is initialized
        # before any communication happens, so that this group can be used for
        # graph capture immediately.
        data = torch.zeros(1)
        data = data.to(device=device)
        torch.distributed.all_reduce(data, group=group)
        torch.cuda.synchronize()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `initialize_model_parallel`, `get_tensor_model_parallel_group`, `zeros` and `to`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 131-140: Scenario logic / 场景逻辑
```python
        del data

        for sz in self.TEST_SIZES:
            for dtype in [torch.float16, torch.bfloat16]:
                for _ in range(self.TEST_LOOP):
                    with graph_capture() as graph_capture_context:
                        # use integers so result matches NCCL exactly
                        inp1 = torch.randint(
                            1,
                            23,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `graph_capture` and `randint`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 141-155: Scenario logic / 场景逻辑
```python
                            (sz,),
                            dtype=dtype,
                            device=torch.cuda.current_device(),
                        )
                        inp2 = torch.randint(
                            -23,
                            1,
                            (sz,),
                            dtype=dtype,
                            device=torch.cuda.current_device(),
                        )
                        torch.cuda.synchronize()
                        graph = torch.cuda.CUDAGraph()
                        with torch.cuda.graph(
                            graph, stream=graph_capture_context.stream
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `current_device`, `randint`, `synchronize` and `CUDAGraph`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 156-165: Scenario logic / 场景逻辑
```python
                        ):
                            out1 = tensor_model_parallel_all_reduce(inp1)
                            # the input buffer is immediately modified to test
                            # synchronization
                            dist.all_reduce(inp1, group=group)
                            out2 = tensor_model_parallel_all_reduce(inp2)
                            dist.all_reduce(inp2, group=group)
                    graph.replay()
                    atol = 1.25 * world_size
                    rtol = 0.5 * world_size
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `tensor_model_parallel_all_reduce`, `all_reduce` and `replay`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 166-167: Assertions and result checks / 断言与结果检查
```python
                    for inp, out in [[inp1, out1], [inp2, out2]]:
                        torch.testing.assert_close(out, inp, atol=atol, rtol=rtol)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assert_close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 168-174: Assertions and result checks / 断言与结果检查
```python
                        # try:
                        #     torch.testing.assert_close(out, inp, atol=atol, rtol=rtol)
                        # except AssertionError as e:
                        #     print("Max abs diff:", (out - inp).abs().max())
                        #     print("Max rel diff:", ((out - inp).abs() / inp.abs().clamp(min=1e-5)).max())

    @ray.remote(num_gpus=1, max_calls=1)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assert_close`, `abs`, `clamp` and `remote`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 175-190: Helper routines around eager_allreduce / 辅助例程
```python
    def eager_allreduce(self, world_size, rank, distributed_init_port, quant_mode):
        os.environ.pop("CUDA_VISIBLE_DEVICES", None)
        os.environ["ROCM_QUICK_REDUCE_QUANTIZATION"] = quant_mode
        os.environ["ROCM_QUICK_REDUCE_CAST_BF16_TO_FP16"] = "0"
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
```
**EN:** This range implements helper routine(s) `eager_allreduce` so setup, transformation, or validation logic can be reused cleanly. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `pop`, `device`, `set_device` and `init_distributed_environment`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 191-199: Scenario logic / 场景逻辑
```python
        for sz in self.TEST_SIZES:
            for dtype in [torch.float16, torch.bfloat16]:
                for _ in range(self.TEST_LOOP):
                    inp1 = torch.randint(
                        1,
                        23,
                        (sz,),
                        dtype=dtype,
                        device=torch.cuda.current_device(),
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randint` and `current_device`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 200-205: Assertions and result checks / 断言与结果检查
```python
                    )
                    out1 = tensor_model_parallel_all_reduce(inp1)
                    dist.all_reduce(inp1, group=group)
                    atol = 1.25 * world_size
                    rtol = 0.5 * world_size
                    torch.testing.assert_close(out1, inp1, atol=atol, rtol=rtol)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `tensor_model_parallel_all_reduce`, `all_reduce` and `assert_close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 206-212: Assertions and result checks / 断言与结果检查
```python
                    # try:
                    #     torch.testing.assert_close(out1, inp1, atol=atol, rtol=rtol)
                    # except AssertionError as e:
                    #     print("Max abs diff:", (out1 - inp1).abs().max())
                    #     print("Max rel diff:", ((out1 - inp1).abs() / inp1.abs().clamp(min=1e-5)).max())
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assert_close`, `abs` and `clamp`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 213-237: Helper routines around qr_variable_input / 辅助例程
```python
def qr_variable_input(rank, world_size):
    device = torch.device(f"cuda:{rank}")
    torch.cuda.set_device(device)
    qr_max_size = None  # MB
    _ptr = ops.init_custom_qr(rank, world_size, qr_max_size)
    ranks = []
    for i in range(world_size):
        ranks.append(i)
    dist.init_process_group(
        backend="nccl",
        init_method="tcp://127.0.0.1:29500",
        rank=rank,
        world_size=world_size,
    )
    cpu_group = torch.distributed.new_group(ranks, backend="nccl")

    handle = ops.qr_get_handle(_ptr)
    world_size = dist.get_world_size(group=cpu_group)
    handles = [None] * world_size
    dist.all_gather_object(handles, handle, group=cpu_group)
    ops.qr_open_handles(_ptr, handles)

    num = 1
    s1 = 1024
    while num < 50000:  # 50000 is sufficient to identify issues.
```
**EN:** This range implements helper routine(s) `qr_variable_input` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `device`, `set_device`, `init_custom_qr` and `append`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 238-258: Assertions and result checks / 断言与结果检查
```python
        dtype = torch.float16
        if num % 2 == 0:
            s2 = 1024
            inp1 = torch.zeros(
                (s1, s2), dtype=dtype, device=torch.cuda.current_device()
            )
        else:
            s2 = 2048
            inp1 = torch.ones((s1, s2), dtype=dtype, device=torch.cuda.current_device())
        result = torch.empty_like(inp1)
        # FP = 0 INT8 = 1 INT6 = 2 INT4 = 3 NONE = 4
        ops.qr_all_reduce(_ptr, inp1, result, 3, cast_bf2half=True)
        try:
            if inp1[0, 0] == 0:
                assert torch.all(result == 0)
            else:
                assert torch.all(result == world_size)
        except AssertionError:
            print("Assertion failed! Allreduce results are incorrect.")
            raise
        num += 1
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `zeros`, `current_device`, `ones` and `empty_like`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 261-273: Class definition for TestQuickreduceVariableInput / 类定义
```python
class TestQuickreduceVariableInput(CustomTestCase):
    """
    When the tensor parallelism is set to 4 or 8, frequent changes
    in the input shape can cause QuickReduce to hang (this issue
    has been observed with the gpt_oss model).
    """

    TP_SIZES = [4, 8]

    @unittest.skipIf(
        not qr_rocm_arch_available(),
        "Only test Quick AllReduce on ROCm architectures >= gfx94*",
    )
```
**EN:** This range declares `TestQuickreduceVariableInput`, which organizes the scenario as a reusable test-oriented class. Representative call sites include `hang`, `skipIf` and `qr_rocm_arch_available`.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 274-285: Test routines around test_custom_quick_allreduce_variable_input / 测试例程
```python
    def test_custom_quick_allreduce_variable_input(self):
        for tp_size in self.TP_SIZES:
            world_size = tp_size
            if world_size > torch.cuda.device_count():
                return

            multiprocessing.set_start_method("spawn", force=True)
            # 90s is enough
            timeout = 90
            processes = []
            for rank in range(tp_size):
                p = multiprocessing.Process(
```
**EN:** This range defines concrete test routine(s) `test_custom_quick_allreduce_variable_input`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `device_count`, `set_start_method` and `Process`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 286-298: Process control logic / 进程控制逻辑
```python
                    target=qr_variable_input, args=(rank, tp_size)
                )
                p.start()
                processes.append((rank, p))
            for rank, p in processes:
                p.join(timeout=timeout)
                if p.is_alive():
                    for r, proc in processes:
                        if proc.is_alive():
                            proc.terminate()
                            proc.join()
                    raise RuntimeError(
                        f"QuickReduce hang detected after {timeout} seconds!"
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `start`, `append`, `join` and `is_alive`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 299-299: Scenario logic / 场景逻辑
```python
                    )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 300-303: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- Process lifecycle management / 进程生命周期管理
- Environment-aware configuration / 环境感知配置
- Streaming responses / 流式响应

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `multiprocessing`, `os`, `random`, `socket`, `typing`, `unittest`
- **Third-party / 第三方库**: `ray`, `torch`, `torch.distributed`
- **Project Modules / 项目模块**: `sglang.srt.distributed`, `sglang.srt.distributed.communication_op`, `sglang.srt.distributed.device_communicators.custom_all_reduce_ops`, `sglang.srt.distributed.device_communicators.quick_all_reduce`, `sglang.srt.distributed.parallel_state`, `sglang.test.test_utils`
