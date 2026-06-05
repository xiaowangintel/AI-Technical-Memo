# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/npu/utils.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements NPU backend support for shared runtime utilities inside the SGLang runtime. / 为 SGLang 运行时提供面向 NPU/Ascend 后端的共享运行时工具支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Module setup and shared state / 模块设置与共享状态
```python
import functools
import logging
import sys
from enum import IntEnum
from typing import TYPE_CHECKING, Callable

import torch

from sglang.srt.environ import envs
from sglang.srt.utils import get_npu_memory_capacity, is_npu

if TYPE_CHECKING:
    from sglang.srt.server_args import ServerArgs

logger = logging.getLogger(__name__)
_is_npu = is_npu()
indexer_weight_stream = None
gva_is_inited = False
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `functools`, `logging`, `sys`, `enum`, `typing`, `torch`. It also defines symbols such as `logger`, `_is_npu`, `indexer_weight_stream`, `gva_is_inited` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `functools`, `logging`, `sys`, `enum`, `typing`, `torch`。 同时定义了 `logger`, `_is_npu`, `indexer_weight_stream`, `gva_is_inited` 等符号，供后续逻辑使用。

### Lines 21-24: Class `NPUACLFormat` declaration / 类 `NPUACLFormat` 声明
```python
class NPUACLFormat(IntEnum):
    ACL_FORMAT_UNDEFINED = -1
    ACL_FORMAT_ND = 2
    ACL_FORMAT_FRACTAL_NZ = 29
```
**EN:** This class establishes `NPUACLFormat` as the main container/coordinator for the surrounding logic. It inherits from `IntEnum`. The main stored fields include `ACL_FORMAT_UNDEFINED`, `ACL_FORMAT_ND`, `ACL_FORMAT_FRACTAL_NZ`.
**CN:** 该类将 `NPUACLFormat` 定义为周边逻辑的主要封装体或协调者。 它继承自 `IntEnum`。 其主要存储字段包括 `ACL_FORMAT_UNDEFINED`, `ACL_FORMAT_ND`, `ACL_FORMAT_FRACTAL_NZ`。

### Lines 27-29: Class `FusedMoEMode` declaration / 类 `FusedMoEMode` 声明
```python
class FusedMoEMode(IntEnum):
    FUSED_DEEP_MOE = 1
    DISPATCH_FFN_COMBINE = 2
```
**EN:** This class establishes `FusedMoEMode` as the main container/coordinator for the surrounding logic. It inherits from `IntEnum`. The main stored fields include `FUSED_DEEP_MOE`, `DISPATCH_FFN_COMBINE`.
**CN:** 该类将 `FusedMoEMode` 定义为周边逻辑的主要封装体或协调者。 它继承自 `IntEnum`。 其主要存储字段包括 `FUSED_DEEP_MOE`, `DISPATCH_FFN_COMBINE`。

### Lines 32-43: Function `_call_once` / 函数 `_call_once`
```python
def _call_once(fn: Callable):

    @functools.wraps(fn)
    def wrapper(*args, **kwargs):
        if getattr(fn, "_has_been_called", False):
            logger.debug("Function {} has already been called.", fn.__name__)
            return

        fn._has_been_called = True
        return fn(*args, **kwargs)

    return wrapper
```
**EN:** This function implements `_call_once`. It primarily calls `functools.wraps`, `getattr`, `fn`, `logger.debug` to complete its work. State updates are written into `fn._has_been_called`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_call_once`。 它主要通过调用 `functools.wraps`, `getattr`, `fn`, `logger.debug` 来完成任务。 状态更新主要写入 `fn._has_been_called`。 实现中使用了条件分支。

### Lines 46-90: Function `set_default_server_args` / 函数 `set_default_server_args`
```python
def set_default_server_args(args: "ServerArgs"):
    """
    Set default server arguments for NPU backend.
    """

    # NPU only works with "ascend" attention backend for now
    args.attention_backend = "ascend"
    args.prefill_attention_backend = "ascend"
    args.decode_attention_backend = "ascend"
    if args.page_size is None:
        args.page_size = 128

    # NPU memory settings
    npu_mem = get_npu_memory_capacity()
    if npu_mem <= 32 * 1024:
        # Ascend 910B4,910B4_1
        # (chunked_prefill_size 4k, cuda_graph_max_bs 16 if tp < 4 else 64)
        if args.chunked_prefill_size is None:
            args.chunked_prefill_size = 4 * 1024
        if args.cuda_graph_max_bs is None:
            if args.tp_size < 4:
                args.cuda_graph_max_bs = 16
            else:
                args.cuda_graph_max_bs = 64
    elif npu_mem <= 64 * 1024:
        # Ascend 910B1,910B2,910B2C,910B3,910_9391,910_9392,910_9381,910_9382,910_9372,910_9362
        # (chunked_prefill_size 8k, cuda_graph_max_bs 64 if tp < 4 else 256)
        if args.chunked_prefill_size is None:
# ... omitted for brevity ...
    if args.enable_hierarchical_cache:
        args.hicache_io_backend = "kernel_ascend"
        if args.use_mla_backend():
            args.hicache_mem_layout = "page_first_kv_split"
        else:
            args.hicache_mem_layout = "page_first_direct"
```
**EN:** This function implements `set_default_server_args`. It primarily calls `get_npu_memory_capacity`, `args.use_mla_backend` to complete its work. State updates are written into `args.attention_backend`, `args.prefill_attention_backend`, `args.decode_attention_backend`, `npu_mem`, `args.disable_custom_all_reduce`, `args.page_size`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `set_default_server_args`。 它主要通过调用 `get_npu_memory_capacity`, `args.use_mla_backend` 来完成任务。 状态更新主要写入 `args.attention_backend`, `args.prefill_attention_backend`, `args.decode_attention_backend`, `npu_mem`, `args.disable_custom_all_reduce`, `args.page_size`。 实现中使用了条件分支。

### Lines 93-109: Function `init_npu_backend` / 函数 `init_npu_backend`
```python
@_call_once
def init_npu_backend():
    """
    Initialize NPU backend. This function should be called only once.
    """

    assert _is_npu, "NPU backend initialization called on non-NPU device."

    import sgl_kernel_npu  # noqa: F401
    import torch_npu
    from torch_npu.contrib import transfer_to_npu  # noqa: F401

    # Re-mock torch.cuda.is_available cuz transfer_to_npu mocks it True
    torch.cuda.is_available = lambda: False

    torch_npu.npu.config.allow_internal_format = True
    torch_npu.npu.set_compile_mode(jit_compile=False)
```
**EN:** This function implements `init_npu_backend`. It primarily calls `torch_npu.npu.set_compile_mode` to complete its work. State updates are written into `torch.cuda.is_available`, `torch_npu.npu.config.allow_internal_format`.
**CN:** 该函数实现了 `init_npu_backend`。 它主要通过调用 `torch_npu.npu.set_compile_mode` 来完成任务。 状态更新主要写入 `torch.cuda.is_available`, `torch_npu.npu.config.allow_internal_format`。

### Lines 112-131: Function `_is_nz_aligned` / 函数 `_is_nz_aligned`
```python
def _is_nz_aligned(tensor: torch.Tensor) -> bool:
    """Check whether the last two dims satisfy FRACTAL_NZ alignment rules.

    Ascend FRACTAL_NZ requires:
      BF16 / FP16 : both dims divisible by 16
      INT8         : k % 16 == 0  and  n % 32 == 0
      INT4         : k % 16 == 0  and  n % 64 == 0
      FP4          : both dims divisible by 64
    """
    if tensor.dim() < 2:
        return False
    k, n = tensor.shape[-2], tensor.shape[-1]
    if tensor.dtype in (torch.bfloat16, torch.float16):
        return k % 16 == 0 and n % 16 == 0
    if tensor.dtype == torch.int8:
        return k % 16 == 0 and n % 32 == 0
    if tensor.dtype in (torch.uint8, torch.int32):
        # INT4 is typically packed into uint8/int32; be conservative
        return k % 16 == 0 and n % 64 == 0
    return True
```
**EN:** This function implements `_is_nz_aligned`. It primarily calls `tensor.dim` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_is_nz_aligned`。 它主要通过调用 `tensor.dim` 来完成任务。 实现中使用了条件分支。

### Lines 134-179: Function `npu_format_cast` / 函数 `npu_format_cast`
```python
def npu_format_cast(
    tensor: torch.Tensor,
    acl_format: NPUACLFormat = NPUACLFormat.ACL_FORMAT_FRACTAL_NZ,
) -> torch.Tensor:
    """
    Cast a tensor to a specific NPU ACL format.

    Args:
        tensor (torch.Tensor): The input tensor.
        acl_format (NPUACLFormat): The target NPU ACL format.

    Returns:
        torch.Tensor: The tensor cast to the specified NPU ACL format.
    """

    if not _is_npu:
        return tensor

    if envs.SGLANG_NPU_DISABLE_ACL_FORMAT_WEIGHT.get():
        return tensor

    if tensor.device == torch.device("cpu"):
        logger.warning_once(
            "Warning: The conversion from 'ND' to 'NZ' does not work on the CPU. "
            "Please disable offloading, otherwise the performance will be "
            "significantly reduced. --dit-cpu-offload false"
        )
        return tensor
# ... omitted for brevity ...

    # Skip format cast for meta tensors (used in offloader)
    if tensor.device.type == "meta":
        return tensor

    return torch.ops.npu.npu_format_cast(tensor, acl_format.value)
```
**EN:** This function implements `npu_format_cast`. It primarily calls `envs.SGLANG_NPU_DISABLE_ACL_FORMAT_WEIGHT.get`, `torch.ops.npu.npu_format_cast`, `torch.device`, `logger.warning_once`, `_is_nz_aligned` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `npu_format_cast`。 它主要通过调用 `envs.SGLANG_NPU_DISABLE_ACL_FORMAT_WEIGHT.get`, `torch.ops.npu.npu_format_cast`, `torch.device`, `logger.warning_once`, `_is_nz_aligned` 来完成任务。 实现中使用了条件分支。

### Lines 182-186: Function `get_indexer_weight_stream` / 函数 `get_indexer_weight_stream`
```python
def get_indexer_weight_stream():
    global indexer_weight_stream
    if indexer_weight_stream is None:
        indexer_weight_stream = torch.npu.Stream()
    return indexer_weight_stream
```
**EN:** This function implements `get_indexer_weight_stream`. It primarily calls `torch.npu.Stream` to complete its work. State updates are written into `indexer_weight_stream`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `get_indexer_weight_stream`。 它主要通过调用 `torch.npu.Stream` 来完成任务。 状态更新主要写入 `indexer_weight_stream`。 实现中使用了条件分支。

### Lines 189-222: Function `init_zbal` / 函数 `init_zbal`
```python
def init_zbal(world_size, gpu_id, world_rank, do_check=True):
    """
    init zbal, if is mix alloc mode, only register for sma & comm
    """
    zbal_mem_size = envs.SGLANG_ZBAL_LOCAL_MEM_SIZE.get()
    if not zbal_mem_size > 0:
        return 1

    global gva_is_inited
    from zbal import is_mix_alloc, switch_to_allocator, zbal_init

    if is_mix_alloc():
        switch_to_allocator()
        # use lazy init for mix alloc
        return 1
    else:
        if envs.SGLANG_ZBAL_BOOTSTRAP_URL.get():
            ret = zbal_init(
                world_size,
                gpu_id,
                world_rank,
                zbal_mem_size * (1024**2),
                ip_port=envs.SGLANG_ZBAL_BOOTSTRAP_URL.get(),
            )
        else:
            ret = zbal_init(world_size, gpu_id, world_rank, zbal_mem_size * (1024**2))

        gva_is_inited = True

        if do_check and not ret:
            logger.error(f"[ZBAL] zbal init failed!")
            sys.exit(-1)

        return ret
```
**EN:** This function implements `init_zbal`. It primarily calls `envs.SGLANG_ZBAL_LOCAL_MEM_SIZE.get`, `is_mix_alloc`, `switch_to_allocator`, `envs.SGLANG_ZBAL_BOOTSTRAP_URL.get`, `zbal_init`, `logger.error` to complete its work. State updates are written into `zbal_mem_size`, `gva_is_inited`, `ret`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `init_zbal`。 它主要通过调用 `envs.SGLANG_ZBAL_LOCAL_MEM_SIZE.get`, `is_mix_alloc`, `switch_to_allocator`, `envs.SGLANG_ZBAL_BOOTSTRAP_URL.get`, `zbal_init`, `logger.error` 来完成任务。 状态更新主要写入 `zbal_mem_size`, `gva_is_inited`, `ret`。 实现中使用了条件分支。

### Lines 225-276: Function `lazy_init_zbal_gva_mem` / 函数 `lazy_init_zbal_gva_mem`
```python
def lazy_init_zbal_gva_mem(
    device, gpu_id, world_rank, world_size, cpu_group=None, do_check=True
):
    """
    lazy init zbal gva mem, keep weights and kv remains alloc by dma vmm to avoid memory fragment
    """
    from zbal import is_mix_alloc, zbal_init

    if not is_mix_alloc():
        logger.info(
            "lazy init is supported only in mix alloc mode, this action will be passed"
        )
        return 1

    global gva_is_inited
    from sglang.srt.utils.common import get_available_gpu_memory

    # TODO need to use allgather if you want use total_memory stats from mem_get_info as unbalance os
    total_memory = 61.2  # 2.5GB for other (workspace & os) outside torch
    free_gpu_memory = get_available_gpu_memory(
        device,
        gpu_id,
        distributed=world_size > 1,
        cpu_group=cpu_group,
        empty_cache=True,
    )

    used_memory = total_memory - free_gpu_memory
# ... omitted for brevity ...

    gva_is_inited = True
    if do_check and not res:
        logger.error(f"[ZBAL] zbal lazy init failed!")
        sys.exit(-1)
    return res
```
**EN:** This function implements `lazy_init_zbal_gva_mem`. It primarily calls `get_available_gpu_memory`, `int`, `print`, `envs.SGLANG_ZBAL_BOOTSTRAP_URL.get`, `is_mix_alloc`, `logger.info` to complete its work. State updates are written into `total_memory`, `free_gpu_memory`, `used_memory`, `used_memory_in_mb`, `gva_in_mb`, `gva_is_inited`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `lazy_init_zbal_gva_mem`。 它主要通过调用 `get_available_gpu_memory`, `int`, `print`, `envs.SGLANG_ZBAL_BOOTSTRAP_URL.get`, `is_mix_alloc`, `logger.info` 来完成任务。 状态更新主要写入 `total_memory`, `free_gpu_memory`, `used_memory`, `used_memory_in_mb`, `gva_in_mb`, `gva_is_inited`。 实现中使用了条件分支。

### Lines 279-280: Constants and shared state / 常量与共享状态
```python
share_stream = None
routed_stream = None
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `share_stream`, `routed_stream`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `share_stream`, `routed_stream`。

### Lines 283-285: Function `get_share_stream` / 函数 `get_share_stream`
```python
def get_share_stream():
    global share_stream
    return share_stream
```
**EN:** This function implements `get_share_stream`.
**CN:** 该函数实现了 `get_share_stream`。

### Lines 288-290: Function `set_share_stream` / 函数 `set_share_stream`
```python
def set_share_stream(stream):
    global share_stream
    share_stream = stream
```
**EN:** This function implements `set_share_stream`. State updates are written into `share_stream`.
**CN:** 该函数实现了 `set_share_stream`。 状态更新主要写入 `share_stream`。

### Lines 295-297: Function `get_routed_stream` / 函数 `get_routed_stream`
```python
def get_routed_stream():
    global routed_stream
    return routed_stream
```
**EN:** This function implements `get_routed_stream`.
**CN:** 该函数实现了 `get_routed_stream`。

### Lines 300-302: Function `set_routed_stream` / 函数 `set_routed_stream`
```python
def set_routed_stream(stream):
    global routed_stream
    routed_stream = stream
```
**EN:** This function implements `set_routed_stream`. State updates are written into `routed_stream`.
**CN:** 该函数实现了 `set_routed_stream`。 状态更新主要写入 `routed_stream`。

### Lines 307-311: Function `wait_share_stream` / 函数 `wait_share_stream`
```python
def wait_share_stream():
    stream = get_share_stream()
    if stream is not None:
        cur_stream = torch.get_device_module().current_stream()
        cur_stream.wait_stream(stream)
```
**EN:** This function implements `wait_share_stream`. It primarily calls `get_share_stream`, `torch.get_device_module.current_stream`, `cur_stream.wait_stream`, `torch.get_device_module` to complete its work. State updates are written into `stream`, `cur_stream`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `wait_share_stream`。 它主要通过调用 `get_share_stream`, `torch.get_device_module.current_stream`, `cur_stream.wait_stream`, `torch.get_device_module` 来完成任务。 状态更新主要写入 `stream`, `cur_stream`。 实现中使用了条件分支。

### Lines 314-318: Function `wait_routed_stream` / 函数 `wait_routed_stream`
```python
def wait_routed_stream():
    stream = get_routed_stream()
    if stream is not None:
        cur_stream = torch.get_device_module().current_stream()
        cur_stream.wait_stream(stream)
```
**EN:** This function implements `wait_routed_stream`. It primarily calls `get_routed_stream`, `torch.get_device_module.current_stream`, `cur_stream.wait_stream`, `torch.get_device_module` to complete its work. State updates are written into `stream`, `cur_stream`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `wait_routed_stream`。 它主要通过调用 `get_routed_stream`, `torch.get_device_module.current_stream`, `cur_stream.wait_stream`, `torch.get_device_module` 来完成任务。 状态更新主要写入 `stream`, `cur_stream`。 实现中使用了条件分支。

### Lines 321-329: Function `process_shared_expert` / 函数 `process_shared_expert`
```python
def process_shared_expert(hidden_states, forward_func):
    stream = get_share_stream()
    if stream is None:
        stream = torch.get_device_module().Stream()
        set_share_stream(stream)
    stream.wait_stream(torch.get_device_module().current_stream())
    with torch.get_device_module().stream(stream):
        shared_output = forward_func(hidden_states)
    return shared_output
```
**EN:** This function implements `process_shared_expert`. It primarily calls `get_share_stream`, `stream.wait_stream`, `torch.get_device_module.Stream`, `set_share_stream`, `torch.get_device_module.current_stream`, `torch.get_device_module.stream` to complete its work. State updates are written into `stream`, `shared_output`. The implementation relies on conditional branches, context-managed resources.
**CN:** 该函数实现了 `process_shared_expert`。 它主要通过调用 `get_share_stream`, `stream.wait_stream`, `torch.get_device_module.Stream`, `set_share_stream`, `torch.get_device_module.current_stream`, `torch.get_device_module.stream` 来完成任务。 状态更新主要写入 `stream`, `shared_output`。 实现中使用了条件分支、上下文管理资源。

### Lines 332-340: Function `process_routed_expert` / 函数 `process_routed_expert`
```python
def process_routed_expert(hidden_states, topk_output, forward_func):
    stream = get_routed_stream()
    if stream is None:
        stream = torch.get_device_module().Stream()
        set_routed_stream(stream)
    stream.wait_stream(torch.get_device_module().current_stream())
    with torch.get_device_module().stream(stream):
        shared_output = forward_func(hidden_states, topk_output)
    return shared_output
```
**EN:** This function implements `process_routed_expert`. It primarily calls `get_routed_stream`, `stream.wait_stream`, `torch.get_device_module.Stream`, `set_routed_stream`, `torch.get_device_module.current_stream`, `torch.get_device_module.stream` to complete its work. State updates are written into `stream`, `shared_output`. The implementation relies on conditional branches, context-managed resources.
**CN:** 该函数实现了 `process_routed_expert`。 它主要通过调用 `get_routed_stream`, `stream.wait_stream`, `torch.get_device_module.Stream`, `set_routed_stream`, `torch.get_device_module.current_stream`, `torch.get_device_module.stream` 来完成任务。 状态更新主要写入 `stream`, `shared_output`。 实现中使用了条件分支、上下文管理资源。

## Key Concepts / 关键概念
- **Classes / 类**: `NPUACLFormat`, `FusedMoEMode`
- **Functions / 函数**: `_call_once`, `set_default_server_args`, `init_npu_backend`, `_is_nz_aligned`, `npu_format_cast`, `get_indexer_weight_stream`, `init_zbal`, `lazy_init_zbal_gva_mem`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.environ`, `sglang.srt.utils`, `sglang.srt.server_args`, `sglang.srt.utils.common`
- **External / 外部依赖**: `torch`, `sgl_kernel_npu`, `torch_npu`, `torch_npu.contrib`, `zbal`
- **Standard library / 标准库**: `functools`, `logging`, `sys`, `enum`, `typing`
