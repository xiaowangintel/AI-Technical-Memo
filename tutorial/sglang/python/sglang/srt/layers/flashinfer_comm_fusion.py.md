# flashinfer_comm_fusion.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/flashinfer_comm_fusion.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements distributed communication and tensor movement for the SGLang SRT runtime. It exposes symbols such as `_should_force_posix_fd_transport`, `_flashinfer_posix_fd_transport_override_if_needed`, `is_flashinfer_allreduce_unavailable`, and `_make_flashinfer_workspace_allocation_prop` and connects them to backend-specific paths such as `CUDA`, `FlashInfer`, `NPU`, and `CPU`. / 该模块为 SGLang 的 SRT 运行时实现了分布式通信与张量搬运。它提供了 `_should_force_posix_fd_transport`、`_flashinfer_posix_fd_transport_override_if_needed`、`is_flashinfer_allreduce_unavailable` 以及 `_make_flashinfer_workspace_allocation_prop` 等符号，并把这些符号连接到 `CUDA`、`FlashInfer`、`NPU` 以及 `CPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-35: Imports, constants, and runtime setup
```python
import contextlib
import logging
import platform
from typing import Optional, Tuple

import torch

from sglang.srt.distributed import (
    get_attn_tensor_model_parallel_rank,
    get_attn_tensor_model_parallel_world_size,
    get_attn_tp_group,
    get_moe_ep_group,
    get_moe_expert_parallel_rank,
    get_moe_expert_parallel_world_size,
    get_moe_tensor_parallel_rank,
    get_moe_tensor_parallel_world_size,
    get_moe_tp_group,
    get_tp_group,
)
from sglang.srt.environ import envs
from sglang.srt.utils import (
    ceil_align,
    get_cuda_driver_bindings,
    is_flashinfer_available,
)
from sglang.srt.utils.custom_op import register_custom_op

logger = logging.getLogger(__name__)

_flashinfer_comm = None
_TorchDistBackend = None
_flashinfer_allreduce_unavailable = False
_posix_transport_override_logged = False
```
**EN:** This section prepares the module namespace. It imports `contextlib`, `logging`, `platform`, `typing.Optional`, `typing.Tuple`, and `torch`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `logger`, `_flashinfer_comm`, `_TorchDistBackend`, `_flashinfer_allreduce_unavailable`, and `_posix_transport_override_logged` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `contextlib`、`logging`、`platform`、`typing.Optional`、`typing.Tuple` 以及 `torch`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `logger`、`_flashinfer_comm`、`_TorchDistBackend`、`_flashinfer_allreduce_unavailable` 以及 `_posix_transport_override_logged` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 36-56: Internal helper `_should_force_posix_fd_transport`
```python
def _should_force_posix_fd_transport() -> bool:
    force_posix_env = envs.SGLANG_FLASHINFER_FORCE_POSIX_FD_TRANSPORT.get()
    if force_posix_env is not None:
        return force_posix_env

    machine = platform.machine().lower()
    if machine not in ("aarch64", "arm64"):
        return False

    if not torch.cuda.is_available():
        return False

    try:
        major, _minor = torch.cuda.get_device_capability(torch.cuda.current_device())
    except Exception as e:
        logger.debug("Failed to get CUDA device capability: %s", e)
        return False

    return major == 10
```
**EN:** This block defines `_should_force_posix_fd_transport` and contains the main logic for this step. It mainly invokes `envs.SGLANG_FLASHINFER_FORCE_POSIX_FD_TRANSPORT.get`, `platform.machine.lower`, `torch.cuda.is_available`, `torch.cuda.get_device_capability`, and `platform.machine`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `force_posix_env`, `machine`, `major`, and `_minor` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_should_force_posix_fd_transport`，并承载这一阶段的核心逻辑。 它主要调用 `envs.SGLANG_FLASHINFER_FORCE_POSIX_FD_TRANSPORT.get`、`platform.machine.lower`、`torch.cuda.is_available`、`torch.cuda.get_device_capability` 以及 `platform.machine`，说明该流程会编排底层辅助函数或计算内核。 像 `force_posix_env`、`machine`、`major` 以及 `_minor` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 57-100: Internal helper `_flashinfer_posix_fd_transport_override_if_needed`
```python
@contextlib.contextmanager
def _flashinfer_posix_fd_transport_override_if_needed():
    # TODO(mmangkad): Remove this temporary override once the
    # FlashInfer unified allreduce-fusion transport issue on
    # GB200/GB300 platforms is fixed and verified resolved.
    global _posix_transport_override_logged

    if not _should_force_posix_fd_transport():
        yield
        return

    try:
        import flashinfer.comm.mnnvl as flashinfer_mnnvl
    except Exception as e:
        logger.debug(
            "Failed to import flashinfer.comm.mnnvl for transport override: %s", e
        )
        yield
        return

    original_checker = getattr(flashinfer_mnnvl, "is_mnnvl_fabric_supported", None)
    if original_checker is None:
        yield
        return

    if not _posix_transport_override_logged:
        logger.warning(
            "Applying FlashInfer transport workaround: forcing PosixFD "
            "symmetric-memory handle exchange on aarch64 + sm10x to avoid "
            "known data corruption with Fabric handle exchange on GB systems. "
            "Set SGLANG_FLASHINFER_FORCE_POSIX_FD_TRANSPORT=0 to disable."
        )
        _posix_transport_override_logged = True

    def _always_disable_fabric(_device_idx: int) -> bool:
        return False

    flashinfer_mnnvl.is_mnnvl_fabric_supported = _always_disable_fabric
    try:
        yield
    finally:
        flashinfer_mnnvl.is_mnnvl_fabric_supported = original_checker
```
**EN:** This block defines `_flashinfer_posix_fd_transport_override_if_needed` and contains the main logic for this step. Decorators like `contextlib.contextmanager` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `getattr`, `_should_force_posix_fd_transport`, `logger.warning`, and `logger.debug`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `original_checker` and `_posix_transport_override_logged` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_flashinfer_posix_fd_transport_override_if_needed`，并承载这一阶段的核心逻辑。 像 `contextlib.contextmanager` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `getattr`、`_should_force_posix_fd_transport`、`logger.warning` 以及 `logger.debug`，说明该流程会编排底层辅助函数或计算内核。 像 `original_checker` 和 `_posix_transport_override_logged` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 101-159: Module-level helper logic
```python
if is_flashinfer_available():
    try:
        import flashinfer.comm as comm

        if hasattr(comm, "allreduce_fusion") and hasattr(
            comm, "create_allreduce_fusion_workspace"
        ):
            _flashinfer_comm = comm
        else:
            _flashinfer_allreduce_unavailable = True
            logger.warning(
                "flashinfer.comm unified allreduce_fusion API is not available, "
                "falling back to standard implementation"
            )
    except ImportError:
        _flashinfer_allreduce_unavailable = True
        logger.warning(
            "flashinfer.comm is not available, falling back to standard "
            "implementation"
        )

    try:
        from flashinfer.comm.mnnvl import TorchDistBackend

        class _FixedTorchDistBackend(TorchDistBackend):
            """Workaround for FlashInfer TorchDistBackend issues.

            1. bcast fix: TorchDistBackend.bcast passes the in-group rank
               directly as `src` to broadcast_object_list, which expects a
               global rank.
            2. Graph-capture fix: initialize with NCCL device_group (so
               the backend derives correct device_idx / GPU mapping), but
               broadcast via GLOO cpu_group (to avoid NCCL collectives
               that interfere with CUDA graph capture).
            """

            def __init__(self, device_group, cpu_group):
                super().__init__(group=device_group)
                self._cpu_group = cpu_group

            def bcast(self, data, root):
                import torch.distributed as dist

                group_ranks = dist.get_process_group_ranks(self._cpu_group)
                global_root = group_ranks[root]
                object_list = [data]
                dist.broadcast_object_list(
                    object_list, src=global_root, group=self._cpu_group
                )
                return object_list[0]

        _TorchDistBackend = _FixedTorchDistBackend
    except ImportError:
        logger.debug(
            "flashinfer.comm.mnnvl.TorchDistBackend is not available, "
            "allreduce fusion will use the default process group"
        )
```
**EN:** This section prepares the module namespace. It imports `flashinfer.comm`, `flashinfer.comm.mnnvl.TorchDistBackend`, and `torch.distributed`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_TorchDistBackend`, `_flashinfer_comm`, `_flashinfer_allreduce_unavailable`, `group_ranks`, and `global_root` capture configuration, cached handles, or feature flags. Control structures like `If` and `Try` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `flashinfer.comm`、`flashinfer.comm.mnnvl.TorchDistBackend` 以及 `torch.distributed`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_TorchDistBackend`、`_flashinfer_comm`、`_flashinfer_allreduce_unavailable`、`group_ranks` 以及 `global_root` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 和 `Try` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 160-163: `is_flashinfer_allreduce_unavailable` predicate for is FlashInfer all-reduce unavailable
```python
def is_flashinfer_allreduce_unavailable() -> bool:
    return _flashinfer_allreduce_unavailable
```
**EN:** This block defines `is_flashinfer_allreduce_unavailable` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `is_flashinfer_allreduce_unavailable`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 164-190: Internal helper `_make_flashinfer_workspace_allocation_prop`
```python
def _make_flashinfer_workspace_allocation_prop(cuda_driver):
    if _should_force_posix_fd_transport():
        handle_type = (
            cuda_driver.CUmemAllocationHandleType.CU_MEM_HANDLE_TYPE_POSIX_FILE_DESCRIPTOR
        )
    else:
        from flashinfer.comm.mnnvl import is_mnnvl_fabric_supported

        if is_mnnvl_fabric_supported(torch.cuda.current_device()):
            handle_type = (
                cuda_driver.CUmemAllocationHandleType.CU_MEM_HANDLE_TYPE_FABRIC
            )
        else:
            handle_type = (
                cuda_driver.CUmemAllocationHandleType.CU_MEM_HANDLE_TYPE_POSIX_FILE_DESCRIPTOR
            )

    prop = cuda_driver.CUmemAllocationProp()
    prop.requestedHandleTypes = handle_type
    prop.type = cuda_driver.CUmemAllocationType.CU_MEM_ALLOCATION_TYPE_PINNED
    prop.location = cuda_driver.CUmemLocation()
    prop.location.type = cuda_driver.CUmemLocationType.CU_MEM_LOCATION_TYPE_DEVICE
    prop.location.id = torch.cuda.current_device()
    prop.allocFlags.gpuDirectRDMACapable = 1
    return prop
```
**EN:** This block defines `_make_flashinfer_workspace_allocation_prop` and contains the main logic for this step. It mainly invokes `_should_force_posix_fd_transport`, `cuda_driver.CUmemAllocationProp`, `cuda_driver.CUmemLocation`, `torch.cuda.current_device`, and `is_mnnvl_fabric_supported`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `prop` and `handle_type` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_make_flashinfer_workspace_allocation_prop`，并承载这一阶段的核心逻辑。 它主要调用 `_should_force_posix_fd_transport`、`cuda_driver.CUmemAllocationProp`、`cuda_driver.CUmemLocation`、`torch.cuda.current_device` 以及 `is_mnnvl_fabric_supported`，说明该流程会编排底层辅助函数或计算内核。 像 `prop` 和 `handle_type` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 191-252: Internal helper `_flashinfer_trtllm_workspace_allocation_sizes`
```python
def _flashinfer_trtllm_workspace_allocation_sizes(
    cuda_driver,
    prop,
    world_size: int,
    max_token_num: int,
    hidden_dim: int,
    dtype: torch.dtype,
) -> list[int]:
    """Mirror FlashInfer TRTLLM SymmDeviceMemory local allocation sizes."""
    elem_size = 4 if dtype == torch.float32 else 2
    buffer_size = world_size * max_token_num * hidden_dim * 2
    flag_size = world_size * 256 * 4

    max_comm_size = 2147483647 & ~((1 << 21) - 1)
    lamport_comm_size = min(
        world_size * max_token_num * hidden_dim * elem_size,
        max_comm_size,
    )
    lamport_buffer_size = lamport_comm_size * 3

    # trtllm_create_ipc_workspace_for_all_reduce_fusion rounds each logical
    # buffer to 2 MiB before passing it to SymmDeviceMemory.
    buffer_sizes = (
        ceil_align(size, 1 << 21)
        for size in (buffer_size, flag_size, lamport_buffer_size)
    )

    signal_pad_size = 2048
    allocation_sizes = []
    for buffer_size in buffer_sizes:
        err, alloc_granularity = cuda_driver.cuMemGetAllocationGranularity(
            prop,
            cuda_driver.CUmemAllocationGranularity_flags.CU_MEM_ALLOC_GRANULARITY_RECOMMENDED,
        )
        if err != cuda_driver.CUresult.CUDA_SUCCESS:
            raise RuntimeError(
                "cuMemGetAllocationGranularity failed for FlashInfer "
                f"workspace preflight: {err}"
            )

        allocation_size = ceil_align(buffer_size + signal_pad_size, alloc_granularity)

        mc_prop = cuda_driver.CUmulticastObjectProp()
        mc_prop.numDevices = world_size
        mc_prop.size = allocation_size
        mc_prop.handleTypes = prop.requestedHandleTypes

        err, mc_granularity = cuda_driver.cuMulticastGetGranularity(
            mc_prop,
            cuda_driver.CUmulticastGranularity_flags.CU_MULTICAST_GRANULARITY_RECOMMENDED,
        )
        if err != cuda_driver.CUresult.CUDA_SUCCESS:
            raise RuntimeError(
                "cuMulticastGetGranularity failed for FlashInfer "
                f"workspace preflight: {err}"
            )

        allocation_size = ceil_align(allocation_size, mc_granularity)
        allocation_sizes.append(allocation_size)
    return allocation_sizes
```
**EN:** This block defines `_flashinfer_trtllm_workspace_allocation_sizes` and contains the main logic for this step. It mainly invokes `min`, `ceil_align`, `cuda_driver.cuMemGetAllocationGranularity`, `cuda_driver.CUmulticastObjectProp`, and `cuda_driver.cuMulticastGetGranularity`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `elem_size`, `buffer_size`, `flag_size`, `max_comm_size`, and `lamport_comm_size` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_flashinfer_trtllm_workspace_allocation_sizes`，并承载这一阶段的核心逻辑。 它主要调用 `min`、`ceil_align`、`cuda_driver.cuMemGetAllocationGranularity`、`cuda_driver.CUmulticastObjectProp` 以及 `cuda_driver.cuMulticastGetGranularity`，说明该流程会编排底层辅助函数或计算内核。 像 `elem_size`、`buffer_size`、`flag_size`、`max_comm_size` 以及 `lamport_comm_size` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 253-266: Internal helper `_probe_cumem_create_sequence`
```python
def _probe_cumem_create_sequence(cuda_driver, allocation_sizes, prop) -> bool:
    handles = []
    try:
        for allocation_size in allocation_sizes:
            err, handle = cuda_driver.cuMemCreate(allocation_size, prop, 0)
            if err != cuda_driver.CUresult.CUDA_SUCCESS:
                return False
            handles.append(handle)
        return True
    finally:
        for handle in reversed(handles):
            cuda_driver.cuMemRelease(handle)
```
**EN:** This block defines `_probe_cumem_create_sequence` and contains the main logic for this step. It mainly invokes `reversed`, `cuda_driver.cuMemCreate`, `handles.append`, and `cuda_driver.cuMemRelease`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `handles`, `err`, and `handle` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_probe_cumem_create_sequence`，并承载这一阶段的核心逻辑。 它主要调用 `reversed`、`cuda_driver.cuMemCreate`、`handles.append` 以及 `cuda_driver.cuMemRelease`，说明该流程会编排底层辅助函数或计算内核。 像 `handles`、`err` 以及 `handle` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 267-332: Internal helper `_preflight_check_workspace_memory`
```python
def _preflight_check_workspace_memory(
    world_size: int,
    max_token_num: int,
    hidden_dim: int,
    dtype: torch.dtype,
    cpu_group: Optional["torch.distributed.ProcessGroup"] = None,
) -> bool:
    """Collectively decide whether to enter FlashInfer workspace creation.

    FlashInfer TRTLLM workspaces allocate several SymmDeviceMemory buffers and
    then exchange handles across ranks. If one rank fails local cuMemCreate and
    exits while peers enter handle exchange, peers can hang until the watchdog
    aborts. Probe the same handle type and allocation sequence first, then vote
    on a CPU group so all ranks proceed or skip together.
    """
    import torch.distributed as dist

    group = cpu_group
    if group is None:
        tp_group = get_tp_group()
        if tp_group.world_size <= 1:
            return True
        group = tp_group.cpu_group

    allocation_sizes = []
    try:
        cuda_driver = get_cuda_driver_bindings()
        prop = _make_flashinfer_workspace_allocation_prop(cuda_driver)
        allocation_sizes = _flashinfer_trtllm_workspace_allocation_sizes(
            cuda_driver,
            prop,
            world_size,
            max_token_num,
            hidden_dim,
            dtype,
        )
        local_ok = _probe_cumem_create_sequence(cuda_driver, allocation_sizes, prop)
    except Exception as e:
        logger.warning(
            "FlashInfer workspace preflight probe failed (%s). "
            "Skipping allreduce fusion.",
            e,
        )
        local_ok = False

    flag = torch.tensor([1 if local_ok else 0], dtype=torch.int32)
    dist.all_reduce(flag, op=dist.ReduceOp.BAND, group=group)

    logger.debug(
        "FlashInfer workspace preflight [rank %s]: probe=%.2f GB, "
        "local_probe=%s, vote=%s",
        dist.get_rank(group=group),
        sum(allocation_sizes) / 1e9,
        "OK" if local_ok else "FAIL",
        "PROCEED" if flag.item() == 1 else "SKIP",
    )
    if flag.item() == 0:
        logger.warning(
            "FlashInfer workspace preflight: cuMemCreate probe failed on at "
            "least one rank. Skipping allreduce fusion to avoid cross-rank "
            "desync inside the flashinfer collective."
        )
        return False
    return True
```
**EN:** This block defines `_preflight_check_workspace_memory` and contains the main logic for this step. It mainly invokes `torch.tensor`, `dist.all_reduce`, `logger.debug`, `get_tp_group`, and `get_cuda_driver_bindings`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `group`, `allocation_sizes`, `flag`, `tp_group`, and `cuda_driver` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_preflight_check_workspace_memory`，并承载这一阶段的核心逻辑。 它主要调用 `torch.tensor`、`dist.all_reduce`、`logger.debug`、`get_tp_group` 以及 `get_cuda_driver_bindings`，说明该流程会编排底层辅助函数或计算内核。 像 `group`、`allocation_sizes`、`flag`、`tp_group` 以及 `cuda_driver` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 333-333: Class `FlashInferWorkspaceManager` declaration and shared state
```python
class FlashInferWorkspaceManager:
```
**EN:** This block introduces class `FlashInferWorkspaceManager` and the state shared by its methods.
**CN:** 该代码块引入类 `FlashInferWorkspaceManager`，并定义其方法共享的状态。

### Lines 334-343: `FlashInferWorkspaceManager` initialization and state setup
```python
    def __init__(self):
        self.workspace = None
        self.world_size = None
        self.rank = None
        self.group = None
        self.max_token_num = None
        self.hidden_dim = None
        self.dtype = None
        self.initialized = False
```
**EN:** This block defines `FlashInferWorkspaceManager.__init__` and contains the main logic for this step. Intermediate names such as `self.workspace`, `self.world_size`, `self.rank`, `self.group`, and `self.max_token_num` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `FlashInferWorkspaceManager.__init__`，并承载这一阶段的核心逻辑。 像 `self.workspace`、`self.world_size`、`self.rank`、`self.group` 以及 `self.max_token_num` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 344-427: Function `FlashInferWorkspaceManager.initialize` and its core logic
```python
    def initialize(
        self,
        world_size: int,
        rank: int,
        max_token_num: int,
        hidden_dim: int,
        dtype: torch.dtype,
        use_oneshot: Optional[bool] = None,
        device_group: Optional["torch.distributed.ProcessGroup"] = None,
        cpu_group: Optional["torch.distributed.ProcessGroup"] = None,
    ):
        """Initialize workspace"""
        if _flashinfer_comm is None:
            logger.warning(
                "FlashInfer comm not available, skipping workspace initialization"
            )
            return

        self.cleanup()

        global _flashinfer_allreduce_unavailable
        if not _preflight_check_workspace_memory(
            world_size=world_size,
            max_token_num=max_token_num,
            hidden_dim=hidden_dim,
            dtype=dtype,
            cpu_group=cpu_group,
        ):
            _flashinfer_allreduce_unavailable = True
            self.workspace = None
            self.initialized = False
            return

        try:
            kwargs = dict(
                backend="trtllm",
                world_size=world_size,
                rank=rank,
                max_token_num=max_token_num,
                hidden_dim=hidden_dim,
                dtype=dtype,
                force_oneshot_support=bool(use_oneshot),
                # Pin the symmetric-memory rendezvous to the actual
                # subgroup. Without this, flashinfer >=0.6.10 falls back
                # to WORLD and TP/EP/CP subgroup peers get addressed
                # incorrectly (kernel hangs in cuda-graph warmup).
                group=device_group,
            )
            if (
                _TorchDistBackend is not None
                and device_group is not None
                and cpu_group is not None
            ):
                kwargs["comm_backend"] = _TorchDistBackend(
                    device_group=device_group, cpu_group=cpu_group
                )
            with _flashinfer_posix_fd_transport_override_if_needed():
                self.workspace = _flashinfer_comm.create_allreduce_fusion_workspace(
                    **kwargs
                )
        except Exception as e:
            _flashinfer_allreduce_unavailable = True
            logger.warning(
                f"Failed to initialize FlashInfer workspace: {e}. "
                "Disabling flashinfer allreduce fusion permanently."
            )
            self.workspace = None
            self.initialized = False
            return

        self.world_size = world_size
        self.rank = rank
        self.group = (device_group, cpu_group)
        self.max_token_num = max_token_num
        self.hidden_dim = hidden_dim
        self.dtype = dtype
        self.initialized = True

        backend = getattr(self.workspace, "backend", "unknown")
        logger.info(
            f"FlashInfer workspace initialized for rank {rank}, "
            f"world_size {world_size}, backend {backend}"
        )
```
**EN:** This block defines `FlashInferWorkspaceManager.initialize` and contains the main logic for this step. It mainly invokes `self.cleanup`, `getattr`, `logger.info`, `logger.warning`, and `_preflight_check_workspace_memory`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.world_size`, `self.rank`, `self.group`, `self.max_token_num`, and `self.hidden_dim` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `FlashInferWorkspaceManager.initialize`，并承载这一阶段的核心逻辑。 它主要调用 `self.cleanup`、`getattr`、`logger.info`、`logger.warning` 以及 `_preflight_check_workspace_memory`，说明该流程会编排底层辅助函数或计算内核。 像 `self.world_size`、`self.rank`、`self.group`、`self.max_token_num` 以及 `self.hidden_dim` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 428-448: `FlashInferWorkspaceManager.is_buffer_size_sufficient` predicate for is buffer size sufficient
```python
    def is_buffer_size_sufficient(
        self,
        token_num: int,
        hidden_dim: int,
        dtype: torch.dtype,
        use_oneshot: Optional[bool] = None,
    ) -> bool:
        if not self.initialized or self.workspace is None:
            return False
        try:
            return self.workspace.is_buffer_size_sufficient(
                tp_size=self.world_size,
                num_tokens=token_num,
                hidden_dim=hidden_dim,
                dtype=dtype,
                use_oneshot=use_oneshot,
            )
        except Exception as e:
            logger.debug(f"FlashInfer workspace size check failed: {e}")
            return False
```
**EN:** This block defines `FlashInferWorkspaceManager.is_buffer_size_sufficient` and contains the main logic for this step. It mainly invokes `self.workspace.is_buffer_size_sufficient` and `logger.debug`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `FlashInferWorkspaceManager.is_buffer_size_sufficient`，并承载这一阶段的核心逻辑。 它主要调用 `self.workspace.is_buffer_size_sufficient` 和 `logger.debug`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 449-466: Function `FlashInferWorkspaceManager.cleanup` and its core logic
```python
    def cleanup(self):
        """Clean up workspace"""
        if self.workspace is not None:
            try:
                self.workspace.destroy()
            except Exception as e:
                logger.warning(f"Failed to cleanup FlashInfer workspace: {e}")
            finally:
                self.workspace = None
                self.initialized = False
                self.world_size = None
                self.rank = None
                self.group = None
                self.max_token_num = None
                self.hidden_dim = None
                self.dtype = None
```
**EN:** This block defines `FlashInferWorkspaceManager.cleanup` and contains the main logic for this step. It mainly invokes `self.workspace.destroy` and `logger.warning`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.workspace`, `self.initialized`, `self.world_size`, `self.rank`, and `self.group` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `FlashInferWorkspaceManager.cleanup`，并承载这一阶段的核心逻辑。 它主要调用 `self.workspace.destroy` 和 `logger.warning`，说明该流程会编排底层辅助函数或计算内核。 像 `self.workspace`、`self.initialized`、`self.world_size`、`self.rank` 以及 `self.group` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 467-470: Module constants and shared configuration
```python
_attn_tp_workspace_manager = FlashInferWorkspaceManager()
_moe_tp_workspace_manager = FlashInferWorkspaceManager()
```
**EN:** This section prepares the module namespace. Shared names such as `_attn_tp_workspace_manager` and `_moe_tp_workspace_manager` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 像 `_attn_tp_workspace_manager` 和 `_moe_tp_workspace_manager` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 471-476: Internal helper `_get_workspace_manager`
```python
def _get_workspace_manager(use_attn_tp_group: bool) -> FlashInferWorkspaceManager:
    return (
        _attn_tp_workspace_manager if use_attn_tp_group else _moe_tp_workspace_manager
    )
```
**EN:** This block defines `_get_workspace_manager` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_get_workspace_manager`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 477-507: Internal helper `_sync_allreduce_unavailable_across_tp`
```python
def _sync_allreduce_unavailable_across_tp():
    """Synchronize _flashinfer_allreduce_unavailable across all TP ranks.

    If workspace initialization fails on any rank, all ranks must agree to
    disable fusion. Otherwise ranks diverge during CUDA graph capture: some
    use FlashInfer fusion (skipping custom allreduce), others fall back to
    standard allreduce (calling register_buffer collectives), causing a hang
    in register_graph_buffers.
    """
    global _flashinfer_allreduce_unavailable
    try:
        import torch.distributed as dist

        tp_group = get_tp_group()
        if tp_group.world_size <= 1:
            return
        flag = torch.tensor(
            [1 if _flashinfer_allreduce_unavailable else 0],
            dtype=torch.int32,
        )
        dist.all_reduce(flag, op=dist.ReduceOp.MAX, group=tp_group.cpu_group)
        if flag.item() > 0 and not _flashinfer_allreduce_unavailable:
            _flashinfer_allreduce_unavailable = True
            logger.warning(
                "FlashInfer allreduce fusion disabled globally because "
                "workspace initialization failed on at least one rank."
            )
    except Exception as e:
        logger.debug(f"Failed to sync flashinfer unavailable flag: {e}")
```
**EN:** This block defines `_sync_allreduce_unavailable_across_tp` and contains the main logic for this step. It mainly invokes `get_tp_group`, `torch.tensor`, `dist.all_reduce`, `logger.warning`, and `logger.debug`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `tp_group`, `flag`, and `_flashinfer_allreduce_unavailable` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_sync_allreduce_unavailable_across_tp`，并承载这一阶段的核心逻辑。 它主要调用 `get_tp_group`、`torch.tensor`、`dist.all_reduce`、`logger.warning` 以及 `logger.debug`，说明该流程会编排底层辅助函数或计算内核。 像 `tp_group`、`flag` 以及 `_flashinfer_allreduce_unavailable` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 508-578: Function `ensure_workspace_initialized` and its core logic
```python
def ensure_workspace_initialized(
    max_token_num: int = 2048,
    hidden_dim: int = 4096,
    dtype: torch.dtype = torch.float16,
    token_num: Optional[int] = None,
    use_oneshot: Optional[bool] = None,
    use_attn_tp_group: bool = True,
):
    """Ensure workspace is initialized"""
    if _flashinfer_allreduce_unavailable:
        return False

    if not is_flashinfer_available() or _flashinfer_comm is None:
        return False

    if use_attn_tp_group:
        world_size = get_attn_tensor_model_parallel_world_size()
        rank = get_attn_tensor_model_parallel_rank()
        coordinator = get_attn_tp_group()
    else:
        if get_moe_expert_parallel_world_size() > 1:
            world_size = get_moe_expert_parallel_world_size()
            rank = get_moe_expert_parallel_rank()
            coordinator = get_moe_ep_group()
        else:
            world_size = get_moe_tensor_parallel_world_size()
            rank = get_moe_tensor_parallel_rank()
            coordinator = get_moe_tp_group()

    # Always pass the coordinator's groups: flashinfer >=0.6.10 reads the
    # rendezvous group from `group=...` (falling back to WORLD when None),
    # so leaving it None silently rendezvouses on WORLD and the kernel ends
    # up addressing the wrong peers in TP/EP/CP subgroup setups.
    device_group = coordinator.device_group
    cpu_group = coordinator.cpu_group

    if world_size <= 1:
        return False

    workspace_manager = _get_workspace_manager(use_attn_tp_group)
    token_num = token_num or max_token_num
    group_key = (device_group, cpu_group)

    if (
        not workspace_manager.initialized
        or workspace_manager.world_size != world_size
        or workspace_manager.rank != rank
        or workspace_manager.group != group_key
        or not workspace_manager.is_buffer_size_sufficient(
            token_num=token_num,
            hidden_dim=hidden_dim,
            dtype=dtype,
            use_oneshot=use_oneshot,
        )
    ):
        workspace_manager.initialize(
            world_size=world_size,
            rank=rank,
            max_token_num=max_token_num,
            hidden_dim=hidden_dim,
            dtype=dtype,
            use_oneshot=use_oneshot,
            device_group=device_group,
            cpu_group=cpu_group,
        )

        _sync_allreduce_unavailable_across_tp()

    return workspace_manager.initialized
```
**EN:** This block defines `ensure_workspace_initialized` and contains the main logic for this step. It mainly invokes `_get_workspace_manager`, `get_attn_tensor_model_parallel_world_size`, `get_attn_tensor_model_parallel_rank`, `get_attn_tp_group`, and `workspace_manager.initialize`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `device_group`, `cpu_group`, `workspace_manager`, `token_num`, and `group_key` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `ensure_workspace_initialized`，并承载这一阶段的核心逻辑。 它主要调用 `_get_workspace_manager`、`get_attn_tensor_model_parallel_world_size`、`get_attn_tensor_model_parallel_rank`、`get_attn_tp_group` 以及 `workspace_manager.initialize`，说明该流程会编排底层辅助函数或计算内核。 像 `device_group`、`cpu_group`、`workspace_manager`、`token_num` 以及 `group_key` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 579-594: Function `fake_flashinfer_allreduce_residual_rmsnorm` and its core logic
```python
def fake_flashinfer_allreduce_residual_rmsnorm(
    input_tensor: torch.Tensor,
    residual: torch.Tensor,
    weight: torch.Tensor,
    eps: float = 1e-6,
    max_token_num: int = 16384,
    use_oneshot: Optional[bool] = None,
    trigger_completion_at_end: bool = False,
    fp32_acc: bool = False,
    use_attn_tp_group: bool = True,
) -> Tuple[torch.Tensor, torch.Tensor]:
    residual_out = torch.empty_like(residual)
    norm_out = torch.empty_like(input_tensor)
    return norm_out, residual_out
```
**EN:** This block defines `fake_flashinfer_allreduce_residual_rmsnorm` and contains the main logic for this step. It mainly invokes `torch.empty_like`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `residual_out` and `norm_out` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fake_flashinfer_allreduce_residual_rmsnorm`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty_like`，说明该流程会编排底层辅助函数或计算内核。 像 `residual_out` 和 `norm_out` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 595-689: Function `flashinfer_allreduce_residual_rmsnorm` and its core logic
```python
@register_custom_op(
    mutates_args=["input_tensor", "residual", "weight"],
    fake_impl=fake_flashinfer_allreduce_residual_rmsnorm,
)
def flashinfer_allreduce_residual_rmsnorm(
    input_tensor: torch.Tensor,
    residual: torch.Tensor,
    weight: torch.Tensor,
    eps: float = 1e-6,
    max_token_num: int = 2048,
    use_oneshot: Optional[bool] = None,
    trigger_completion_at_end: bool = False,
    fp32_acc: bool = False,
    use_attn_tp_group: bool = True,
) -> Tuple[torch.Tensor, torch.Tensor]:
    """
    Use FlashInfer's fused allreduce + residual + RMS norm operation

    Args:
        input_tensor: Input tensor that needs allreduce
        residual: Residual tensor
        weight: RMS norm weight
        eps: RMS norm epsilon
        max_token_num: Maximum token number
        use_oneshot: Whether to use oneshot mode
        trigger_completion_at_end: Whether to trigger completion at end
        fp32_acc: Whether to use fp32 precision
        use_attn_tp_group: If True, use attention TP group; otherwise use MoE TP group

    Returns:
        Tuple[torch.Tensor, torch.Tensor]: (norm_output, residual_output)
    """
    if not is_flashinfer_available() or _flashinfer_comm is None:
        logger.debug(
            "FlashInfer not available, falling back to standard implementation"
        )
        return None, None

    if use_attn_tp_group:
        world_size = get_attn_tensor_model_parallel_world_size()
    else:
        # If MoE expert parallel world size > 1, use expert parallel group
        # Otherwise, use tensor parallel group
        # The two values cannot be larger than 1 at the same time
        if get_moe_expert_parallel_world_size() > 1:
            world_size = get_moe_expert_parallel_world_size()
        else:
            world_size = get_moe_tensor_parallel_world_size()

    if world_size <= 1:
        logger.debug("Single GPU, no need for allreduce fusion")
        return None, None

    assert input_tensor.shape[0] <= max_token_num
    if (
        not input_tensor.is_contiguous()
        or not residual.is_contiguous()
        or not weight.is_contiguous()
    ):
        logger.debug("Non-contiguous tensors, skipping FlashInfer allreduce fusion")
        return None, None

    if not ensure_workspace_initialized(
        max_token_num=max_token_num,
        hidden_dim=input_tensor.shape[-1],
        dtype=input_tensor.dtype,
        token_num=input_tensor.shape[0],
        use_oneshot=use_oneshot,
        use_attn_tp_group=use_attn_tp_group,
    ):
        logger.debug("FlashInfer workspace not available")
        return None, None

    residual_out = torch.empty_like(residual)
    norm_out = torch.empty_like(input_tensor)

    workspace_manager = _get_workspace_manager(use_attn_tp_group)
    _flashinfer_comm.allreduce_fusion(
        input=input_tensor,
        workspace=workspace_manager.workspace,
        pattern=_flashinfer_comm.AllReduceFusionPattern.kARResidualRMSNorm,
        launch_with_pdl=True,
        trigger_completion_at_end=trigger_completion_at_end,
        residual_out=residual_out,
        norm_out=norm_out,
        residual_in=residual,
        rms_gamma=weight,
        rms_eps=eps,
        use_oneshot=use_oneshot,
        fp32_acc=fp32_acc,
    )

    return norm_out, residual_out
```
**EN:** This block defines `flashinfer_allreduce_residual_rmsnorm` and contains the main logic for this step. Decorators like `register_custom_op` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_custom_op`, `torch.empty_like`, `_get_workspace_manager`, `_flashinfer_comm.allreduce_fusion`, and `logger.debug`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `residual_out`, `norm_out`, `workspace_manager`, and `world_size` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `flashinfer_allreduce_residual_rmsnorm`，并承载这一阶段的核心逻辑。 像 `register_custom_op` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_custom_op`、`torch.empty_like`、`_get_workspace_manager`、`_flashinfer_comm.allreduce_fusion` 以及 `logger.debug`，说明该流程会编排底层辅助函数或计算内核。 像 `residual_out`、`norm_out`、`workspace_manager` 以及 `world_size` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 690-723: Function `pre_initialize_workspaces` and its core logic
```python
def pre_initialize_workspaces(
    max_token_num: int,
    hidden_dim: int,
    dtype: torch.dtype,
    use_oneshot: Optional[bool] = None,
):
    """Pre-initialize flashinfer workspaces before CUDA graph capture.

    This must be called before graph capture to avoid collective operations
    (broadcasts, barriers) inside the graph capture context, which can
    deadlock with custom_all_reduce.register_graph_buffers.
    """
    if _flashinfer_allreduce_unavailable or _flashinfer_comm is None:
        return

    # Initialize MoE workspace
    ensure_workspace_initialized(
        max_token_num=max_token_num,
        hidden_dim=hidden_dim,
        dtype=dtype,
        use_oneshot=use_oneshot,
        use_attn_tp_group=False,
    )

    # Initialize attention workspace
    ensure_workspace_initialized(
        max_token_num=max_token_num,
        hidden_dim=hidden_dim,
        dtype=dtype,
        use_oneshot=use_oneshot,
        use_attn_tp_group=True,
    )
```
**EN:** This block defines `pre_initialize_workspaces` and contains the main logic for this step. It mainly invokes `ensure_workspace_initialized`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `pre_initialize_workspaces`，并承载这一阶段的核心逻辑。 它主要调用 `ensure_workspace_initialized`，说明该流程会编排底层辅助函数或计算内核。

### Lines 724-732: Function `cleanup_flashinfer_workspace` and its core logic
```python
def cleanup_flashinfer_workspace():
    global _attn_tp_workspace_manager, _moe_tp_workspace_manager
    if _attn_tp_workspace_manager is not None:
        _attn_tp_workspace_manager.cleanup()
    if (
        _moe_tp_workspace_manager is not None
        and _moe_tp_workspace_manager is not _attn_tp_workspace_manager
    ):
        _moe_tp_workspace_manager.cleanup()
```
**EN:** This block defines `cleanup_flashinfer_workspace` and contains the main logic for this step. It mainly invokes `_attn_tp_workspace_manager.cleanup` and `_moe_tp_workspace_manager.cleanup`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `cleanup_flashinfer_workspace`，并承载这一阶段的核心逻辑。 它主要调用 `_attn_tp_workspace_manager.cleanup` 和 `_moe_tp_workspace_manager.cleanup`，说明该流程会编排底层辅助函数或计算内核。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `_should_force_posix_fd_transport`, `_flashinfer_posix_fd_transport_override_if_needed`, `is_flashinfer_allreduce_unavailable`, `_make_flashinfer_workspace_allocation_prop`, and `_flashinfer_trtllm_workspace_allocation_sizes`. / **主要符号**：核心入口包括 `_should_force_posix_fd_transport`、`_flashinfer_posix_fd_transport_override_if_needed`、`is_flashinfer_allreduce_unavailable`、`_make_flashinfer_workspace_allocation_prop` 以及 `_flashinfer_trtllm_workspace_allocation_sizes`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Acceleration stack**: The module depends on external acceleration libraries and wraps them behind Python entry points. / **加速栈**：该模块依赖外部加速库，并通过 Python 入口对它们进行封装。

## Dependencies / 依赖关系
- **Standard library**: `contextlib`, `logging`, `platform`, `typing.Optional`, and `typing.Tuple` / **标准库**：`contextlib`、`logging`、`platform`、`typing.Optional` 以及 `typing.Tuple`
- **Third-party**: `torch`, `torch.distributed`, `flashinfer.comm.mnnvl`, `flashinfer.comm`, `flashinfer.comm.mnnvl.TorchDistBackend`, and `flashinfer.comm.mnnvl.is_mnnvl_fabric_supported` / **第三方依赖**：`torch`、`torch.distributed`、`flashinfer.comm.mnnvl`、`flashinfer.comm`、`flashinfer.comm.mnnvl.TorchDistBackend` 以及 `flashinfer.comm.mnnvl.is_mnnvl_fabric_supported`
- **Internal SGLang modules**: `sglang.srt.distributed.get_attn_tensor_model_parallel_rank`, `sglang.srt.distributed.get_attn_tensor_model_parallel_world_size`, `sglang.srt.distributed.get_attn_tp_group`, `sglang.srt.distributed.get_moe_ep_group`, `sglang.srt.distributed.get_moe_expert_parallel_rank`, `sglang.srt.distributed.get_moe_expert_parallel_world_size`, `sglang.srt.distributed.get_moe_tensor_parallel_rank`, `sglang.srt.distributed.get_moe_tensor_parallel_world_size`, `sglang.srt.distributed.get_moe_tp_group`, `sglang.srt.distributed.get_tp_group`, `sglang.srt.environ.envs`, and `sglang.srt.utils.ceil_align` / **SGLang 内部模块**：`sglang.srt.distributed.get_attn_tensor_model_parallel_rank`、`sglang.srt.distributed.get_attn_tensor_model_parallel_world_size`、`sglang.srt.distributed.get_attn_tp_group`、`sglang.srt.distributed.get_moe_ep_group`、`sglang.srt.distributed.get_moe_expert_parallel_rank`、`sglang.srt.distributed.get_moe_expert_parallel_world_size`、`sglang.srt.distributed.get_moe_tensor_parallel_rank`、`sglang.srt.distributed.get_moe_tensor_parallel_world_size`、`sglang.srt.distributed.get_moe_tp_group`、`sglang.srt.distributed.get_tp_group`、`sglang.srt.environ.envs` 以及 `sglang.srt.utils.ceil_align`
