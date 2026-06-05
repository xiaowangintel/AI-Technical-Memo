# test_custom_allreduce.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_custom_allreduce.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `custom allreduce` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `custom allreduce` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Imports and module setup
````python
import ctypes
import multiprocessing as mp
import random
import socket
import unittest
from typing import Any, List, Optional

import sgl_kernel.allreduce as custom_ops
import torch
import torch.distributed as dist
from torch.distributed import ProcessGroup

from sglang.srt.distributed.device_communicators.cuda_wrapper import CudaRTLibrary
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 16-66: `_run_correctness_worker` definition
````python
def _run_correctness_worker(world_size, rank, distributed_init_port, test_sizes):
    device = torch.device(f"cuda:{rank}")
    torch.cuda.set_device(device)
    distributed_init_method = f"tcp://localhost:{distributed_init_port}"
    dist.init_process_group(
        backend="nccl",
        init_method=distributed_init_method,
        rank=rank,
        world_size=world_size,
    )
    group = dist.group.WORLD

    try:
        device = torch.device(f"cuda:{rank}")
        max_size = 8192 * 1024
        meta_ptrs = TestCustomAllReduce.create_shared_buffer(
            custom_ops.meta_size() + max_size, group=group
        )

        rank_data = torch.empty(8 * 1024 * 1024, dtype=torch.uint8, device=device)
        buffer_ptrs = TestCustomAllReduce.create_shared_buffer(max_size, group=group)

        custom_ptr = custom_ops.init_custom_ar(meta_ptrs, rank_data, rank, True)
        custom_ops.register_buffer(custom_ptr, buffer_ptrs)

        test_loop = 10
        for sz in test_sizes:
            for dtype in [torch.float32, torch.float16, torch.bfloat16]:
                for _ in range(test_loop):
                    inp1 = torch.randint(1, 16, (sz,), dtype=dtype, device=device)
                    inp1_ref = inp1.clone()
                    out1 = torch.empty_like(inp1)

                    custom_ops.all_reduce(
                        custom_ptr, inp1, out1, buffer_ptrs[rank], max_size
                    )

                    dist.all_reduce(inp1_ref, group=group)

                    torch.testing.assert_close(out1, inp1_ref)

    finally:
        dist.barrier(group=group)
        if custom_ptr is not None:
            custom_ops.dispose(custom_ptr)
        if buffer_ptrs:
            TestCustomAllReduce.free_shared_buffer(buffer_ptrs, group)
        if meta_ptrs:
            TestCustomAllReduce.free_shared_buffer(meta_ptrs, group)

        dist.destroy_process_group(group=group)
````
**EN:** This section defines `_run_correctness_worker` and implements the core logic associated with run correctness worker.
**CN:** 该部分定义 `_run_correctness_worker`，并实现与 run correctness worker 相关的核心逻辑。

### Lines 69-77: `get_open_port` definition
````python
def get_open_port() -> int:
    try:
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
            s.bind(("127.0.0.1", 0))
            return s.getsockname()[1]
    except OSError:
        with socket.socket(socket.AF_INET6, socket.SOCK_STREAM) as s:
            s.bind(("::1", 0))
            return s.getsockname()[1]
````
**EN:** This section defines `get_open_port`. It retrieves or computes the `open port` path used by the module.
**CN:** 该部分定义 `get_open_port`。它负责获取或计算模块中与 `open port` 相关的处理路径。

### Lines 80-97: `multi_process_parallel` definition
````python
def multi_process_parallel(
    world_size: int, test_target: Any, target_args: tuple = ()
) -> None:
    mp.set_start_method("spawn", force=True)

    procs = []
    distributed_init_port = get_open_port()
    for i in range(world_size):
        proc_args = (world_size, i, distributed_init_port) + target_args
        proc = mp.Process(target=test_target, args=proc_args, name=f"Worker-{i}")
        proc.start()
        procs.append(proc)

    for i in range(world_size):
        procs[i].join()
        assert (
            procs[i].exitcode == 0
        ), f"Process {i} failed with exit code {procs[i].exitcode}"
````
**EN:** This section defines `multi_process_parallel` and implements the core logic associated with multi process parallel. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义 `multi_process_parallel`，并实现与 multi process parallel 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。

### Lines 100-181: `TestCustomAllReduce` definition
````python
class TestCustomAllReduce(unittest.TestCase):
    test_sizes = [
        512,
        2560,
        4096,
        5120,
        7680,
        32768,
        262144,
        524288,
        1048576,
        2097152,
    ]
    world_sizes = [2, 4, 8]

    @staticmethod
    def create_shared_buffer(
        size_in_bytes: int, group: Optional[ProcessGroup] = None
    ) -> List[int]:
        lib = CudaRTLibrary()
        pointer = lib.cudaMalloc(size_in_bytes)
        handle = lib.cudaIpcGetMemHandle(pointer)
        if group is None:
            group = dist.group.WORLD
        world_size = dist.get_world_size(group=group)
        rank = dist.get_rank(group=group)

        handle_bytes = ctypes.string_at(ctypes.addressof(handle), ctypes.sizeof(handle))
        input_tensor = torch.ByteTensor(list(handle_bytes)).to(f"cuda:{rank}")
        gathered_tensors = [torch.empty_like(input_tensor) for _ in range(world_size)]
        dist.all_gather(gathered_tensors, input_tensor, group=group)

        handles = []
        handle_type = type(handle)
        for tensor in gathered_tensors:
            bytes_list = tensor.cpu().tolist()
            bytes_data = bytes(bytes_list)
            handle_obj = handle_type()
            ctypes.memmove(ctypes.addressof(handle_obj), bytes_data, len(bytes_data))
            handles.append(handle_obj)

        pointers: List[int] = []
        for i, h in enumerate(handles):
            if i == rank:
                pointers.append(pointer.value)
            else:
                try:
                    opened_ptr = lib.cudaIpcOpenMemHandle(h)
                    pointers.append(opened_ptr.value)
                except Exception as e:
                    print(f"Rank {rank}: Failed to open IPC handle from rank {i}: {e}")
                    raise

        dist.barrier(group=group)
        return pointers

    @staticmethod
    def free_shared_buffer(
        pointers: List[int], group: Optional[ProcessGroup] = None
    ) -> None:
        if group is None:
            group = dist.group.WORLD
        rank = dist.get_rank(group=group)
        lib = CudaRTLibrary()
        if pointers and len(pointers) > rank and pointers[rank] is not None:
            lib.cudaFree(ctypes.c_void_p(pointers[rank]))
        dist.barrier(group=group)

    def test_correctness(self):
        for world_size in self.world_sizes:
            available_gpus = torch.cuda.device_count()
            if world_size > available_gpus:
                print(
                    f"Skipping world_size={world_size}, requires {world_size} GPUs, found {available_gpus}"
                )
                continue

            print(f"Running test for world_size={world_size}")
            multi_process_parallel(
                world_size, _run_correctness_worker, target_args=(self.test_sizes,)
            )
            print(f"custom allreduce tp = {world_size}: OK")
````
**EN:** This section defines the class `TestCustomAllReduce`, grouping related state and behavior around test Custom All Reduce.
**CN:** 该部分定义类 `TestCustomAllReduce`，把与 test Custom All Reduce 相关的状态和行为组织在一起。

### Lines 184-185: Command-line entry point
````python
if __name__ == "__main__":
    unittest.main()
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `_run_correctness_worker`, `get_open_port`, `multi_process_parallel`, `TestCustomAllReduce`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel.allreduce`, `sglang.srt.distributed.device_communicators.cuda_wrapper`
- **External / 外部**: `ctypes`, `multiprocessing`, `random`, `socket`, `torch`, `torch.distributed`, `typing`, `unittest`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
