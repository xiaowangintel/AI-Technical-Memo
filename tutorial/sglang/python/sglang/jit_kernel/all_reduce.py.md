# all_reduce.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/all_reduce.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from __future__ import annotations". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from __future__ import annotations”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports and module setup
```python
from __future__ import annotations

import enum
from typing import TYPE_CHECKING, List, NamedTuple, Optional, Tuple, cast

import torch
import tvm_ffi
from tvm_ffi import Module

from sglang.jit_kernel.utils import (
    cache_once,
    is_arch_support_pdl,
    load_jit,
    make_cpp_args,
)
from sglang.kernel_api_logging import debug_kernel_api
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 19-23: Class `ConfigResult`
```python
class ConfigResult(NamedTuple):
    num_blocks: int
    num_threads: int
```
**EN:** This block declares the `ConfigResult` class and establishes the behavior or state it encapsulates. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段声明了 `ConfigResult` 类，并建立其封装的行为或状态。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 24-95: Class `AllReduceAlgo`
```python
class AllReduceAlgo(enum.Enum):
    ONE_SHOT_PUSH = enum.auto()
    ONE_SHOT_PULL = enum.auto()
    TWO_SHOT_PULL = enum.auto()

    def is_push(self) -> bool:
        return self == AllReduceAlgo.ONE_SHOT_PUSH

    @property
    def shot(self) -> int:
        return 2 if self == AllReduceAlgo.TWO_SHOT_PULL else 1


if TYPE_CHECKING:
    CUSTOM_AR_HANDLE = List[int]
    CUSTOM_AR_PAIR = Tuple[int, CUSTOM_AR_HANDLE]

    class CustomAllReduceObj:
        def __init__(
            self,
            rank: int,
            world_size: int,
            pull_buffer_bytes: int,
            push_buffer_bytes: int,
            graph_input_count: int,
            *,
            max_pull_blocks: Optional[int] = None,
            max_push_blocks: Optional[int] = None,
        ) -> None:
            """
            Create a CustomAllReduceObj instance.

            :param rank: The rank of the current process.
            :param world_size: The total number of processes in the group.
            :param pull_buffer_bytes: The size of the buffer (in bytes) used for pull-based all-reduce.
            :param push_buffer_bytes: The size of the buffer (in bytes) used for push-based all-reduce.
            :param graph_input_count: The maximum number of inputs in all CUDA graphs.
            :param max_pull_blocks: The maximum number of thread blocks to launch for pull-based all-reduce.
                                    If None, it will be determined by the implementation.
            :param max_push_blocks: The maximum number of thread blocks to launch for push-based all-reduce.
                                    If None, it will be determined by the implementation.
            """

        @property
        def world_size(self) -> int: ...
        def share_storage(self) -> CUSTOM_AR_HANDLE: ...
        def share_graph_inputs(self) -> List[CUSTOM_AR_PAIR]: ...
        def post_init(self, handles: List[CUSTOM_AR_HANDLE]) -> None: ...
        def register_inputs(self, handles: List[List[CUSTOM_AR_PAIR]]) -> None: ...
        def set_cuda_graph_capture(self, is_capturing: bool) -> None: ...
        def free(self, tp_cpu_group: torch.distributed.ProcessGroup) -> None: ...
        def all_reduce(
            self, input: torch.Tensor, algo: AllReduceAlgo
        ) -> tvm_ffi.Tensor: ...
        def config_pull(
            self, num_blocks: int = -1, num_threads: int = -1
        ) -> ConfigResult:
            """
            Configure the CUDA kernel's grid and block dimensions.
            This provides only the upper bound of the configuration,
            and the actual launch configuration may be determined by implementation.
            Note that push-based all-reduce can not be configured currently.

            :param num_blocks: The maximum number of thread blocks to launch. -1 means no limit.
            :param num_threads: The maximum number of threads per block. -1 means no limit.

            :return: The previous configuration as a ConfigResult named tuple.
            """
            ...


@cache_once
```
**EN:** This block declares the `AllReduceAlgo` class and establishes the behavior or state it encapsulates. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段声明了 `AllReduceAlgo` 类，并建立其封装的行为或状态。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 96-107: Function `_jit_custom_all_reduce_pull_module`
```python
def _jit_custom_all_reduce_pull_module(dtype: torch.dtype, world_size: int) -> Module:
    args = make_cpp_args(dtype, world_size, is_arch_support_pdl())
    return load_jit(
        "custom_all_reduce_pull",
        *args,
        extra_ldflags=["-lcuda"],
        cuda_files=["distributed/custom_all_reduce_pull.cuh"],
        cuda_wrappers=[("all_reduce", f"custom_all_reduce<{args}>")],
    )


@cache_once
```
**EN:** This block defines `_jit_custom_all_reduce_pull_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_custom_all_reduce_pull_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 108-119: Function `_jit_custom_all_reduce_push_module`
```python
def _jit_custom_all_reduce_push_module(dtype: torch.dtype, world_size: int) -> Module:
    args = make_cpp_args(dtype, world_size, is_arch_support_pdl())
    return load_jit(
        "custom_all_reduce_push",
        *args,
        extra_ldflags=["-lcuda"],
        cuda_files=["distributed/custom_all_reduce_push.cuh"],
        cuda_wrappers=[("all_reduce", f"custom_all_reduce<{args}>")],
    )


@cache_once
```
**EN:** This block defines `_jit_custom_all_reduce_push_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_custom_all_reduce_push_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 120-137: Function `_jit_fused_parallel_qknorm_module`
```python
def _jit_fused_parallel_qknorm_module(
    dtype: torch.dtype, world_size: int, q_dim: int, k_dim: int
) -> Module:
    args = make_cpp_args(dtype, world_size, q_dim, k_dim, is_arch_support_pdl())
    cls_name = f"FusedParallelQKNormAcrossHead<{args}>"
    return load_jit(
        "tp_qknorm",
        *args,
        extra_ldflags=["-lcuda"],
        cuda_files=["distributed/tp_qknorm.cuh"],
        cuda_wrappers=[
            ("fused_parallel_qknorm", f"{cls_name}::run"),
            ("get_max_occupancy", f"{cls_name}::get_max_occupancy"),
        ],
    )


@cache_once
```
**EN:** This block defines `_jit_fused_parallel_qknorm_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_fused_parallel_qknorm_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 138-220: Function `get_custom_all_reduce_cls`
```python
def get_custom_all_reduce_cls() -> type[CustomAllReduceObj]:
    module = load_jit(
        "custom_all_reduce_base",
        extra_ldflags=["-lcuda"],
        cuda_files=["distributed/custom_all_reduce_base.cuh"],
        cuda_wrappers=[("register_once", "register_custom_all_reduce")],
    )
    module.register_once()
    device = torch.cuda.current_device()
    props = torch.cuda.get_device_properties(device)
    NUM_CTA = props.multi_processor_count
    MAX_THREADS = 512

    @tvm_ffi.register_object("sgl.CustomAllReduce")
    class CustomAllReduceObjReal(tvm_ffi.Object):
        __slots__ = ("__dict__",)

        def __init__(
            self,
            rank: int,
            world_size: int,
            pull_buffer_bytes: int,
            push_buffer_bytes: int,
            graph_input_count: int,
            *,
            max_pull_blocks: Optional[int] = None,
            max_push_blocks: Optional[int] = None,
        ) -> None:
            max_pull_blocks = NUM_CTA if max_pull_blocks is None else max_pull_blocks
            max_push_blocks = NUM_CTA if max_push_blocks is None else max_push_blocks
            self.__ffi_init__(
                rank,
                world_size,
                max_pull_blocks,
                max_push_blocks,
                pull_buffer_bytes,
                push_buffer_bytes,
                graph_input_count,
            )
            self._world_size = world_size
            self._pull_config = ConfigResult(min(NUM_CTA, max_pull_blocks), MAX_THREADS)
            if max_pull_blocks > 0:  # special case: cannot configure 0 blocks
                self.configure_pull(*self._pull_config)  # type: ignore

        @property
        def world_size(self) -> int:
            return self._world_size

        @debug_kernel_api
        def all_reduce(
            self,
            input: torch.Tensor,
            algo: AllReduceAlgo,
        ) -> tvm_ffi.Tensor:
            compile_fn = (
                _jit_custom_all_reduce_push_module
                if algo.is_push()
                else _jit_custom_all_reduce_pull_module
            )
            module = compile_fn(input.dtype, self._world_size)
            return module.all_reduce(self, input, algo.shot)

        def config_pull(
            self, num_blocks: int = -1, num_threads: int = -1
        ) -> ConfigResult:
            old_config = self._pull_config
            num_blocks = num_blocks if num_blocks != -1 else old_config.num_blocks
            num_threads = num_threads if num_threads != -1 else old_config.num_threads
            new_config = ConfigResult(num_blocks, num_threads)
            if new_config != old_config:
                result = ConfigResult(*self.configure_pull(*new_config))  # type: ignore
                assert result == self._pull_config
                self._pull_config = new_config
            return old_config

        def free(self, tp_cpu_group: torch.distributed.ProcessGroup) -> None:
            self.free_ipc_handles()  # type: ignore
            torch.distributed.barrier(group=tp_cpu_group)
            self.free_storage()  # type: ignore

# ...
```
**EN:** This block defines `get_custom_all_reduce_cls`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `get_custom_all_reduce_cls`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 221-227: Function `get_fused_parallel_qknorm_max_occupancy`
```python
def get_fused_parallel_qknorm_max_occupancy(
    dtype: torch.dtype, world_size: int, q_dim: int, k_dim: int
) -> int:
    module = _jit_fused_parallel_qknorm_module(dtype, world_size, q_dim, k_dim)
    return module.get_max_occupancy()
```
**EN:** This block defines `get_fused_parallel_qknorm_max_occupancy`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `get_fused_parallel_qknorm_max_occupancy`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 228-240: Function `fused_parallel_qknorm`
```python
def fused_parallel_qknorm(
    custom_ar: CustomAllReduceObj,
    q: torch.Tensor,
    k: torch.Tensor,
    q_weight: torch.Tensor,
    k_weight: torch.Tensor,
    eps: float = 1e-6,
) -> None:
    world_size = custom_ar.world_size
    q_dim = q.shape[-1] * world_size
    k_dim = k.shape[-1] * world_size
    module = _jit_fused_parallel_qknorm_module(q.dtype, world_size, q_dim, k_dim)
    module.fused_parallel_qknorm(custom_ar, q, k, q_weight, k_weight, eps)
```
**EN:** This block defines `fused_parallel_qknorm`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `fused_parallel_qknorm`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `enum`
- `typing -> TYPE_CHECKING`
- `torch`
- `tvm_ffi`
- `tvm_ffi -> Module`
- `sglang.jit_kernel.utils -> (`
- `sglang.kernel_api_logging -> debug_kernel_api`
