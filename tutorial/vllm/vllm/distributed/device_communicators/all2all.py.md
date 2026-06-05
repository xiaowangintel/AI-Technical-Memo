# all2all.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/device_communicators/all2all.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements distributed device-communication helpers or backend adapters. / 实现分布式设备通信辅助逻辑或后端适配器。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化
```python
import threading
from typing import Any

import torch
import torch.distributed as dist

import vllm.envs as envs
from vllm.distributed import get_dp_group, get_ep_group
from vllm.forward_context import get_forward_context
from vllm.logger import init_logger
from vllm.utils.flashinfer import (
    has_flashinfer_nvlink_one_sided,
    has_flashinfer_nvlink_two_sided,
)
from vllm.utils.import_utils import has_deep_ep, has_mori

from .base_device_communicator import All2AllManagerBase, Cache
```
**EN:** This block imports `threading`, `typing`, `torch`, `torch.distributed`, `vllm.envs`, `vllm.distributed` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `threading`, `typing`, `torch`, `torch.distributed`, `vllm.envs`, `vllm.distributed`，为后续实现准备运行时、类型与辅助 API。

### Conditional block / 条件代码块
```python
if has_flashinfer_nvlink_two_sided():
    from flashinfer.comm import Mapping  # type: ignore[import-not-found]
    from flashinfer.comm.mnnvl import MnnvlConfig  # type: ignore[import-not-found]
    from flashinfer.comm.trtllm_alltoall import (
        MnnvlMoe,  # type: ignore[import-not-found]
    )
```
**EN:** This top-level conditional adapts module behavior to a runtime capability, configuration flag, or platform detail.
**CN:** 该顶层条件分支会根据运行时能力、配置开关或平台细节调整模块行为。

### Conditional block / 条件代码块
```python
if has_flashinfer_nvlink_one_sided():
    from flashinfer.comm import Mapping  # type: ignore[import-not-found]
    from flashinfer.comm.mnnvl import MnnvlConfig  # type: ignore[import-not-found]
    from flashinfer.comm.trtllm_moe_alltoall import (
        MoeAlltoAll,  # type: ignore[import-not-found]
        moe_a2a_get_workspace_size_per_rank,
    )
```
**EN:** This top-level conditional adapts module behavior to a runtime capability, configuration flag, or platform detail.
**CN:** 该顶层条件分支会根据运行时能力、配置开关或平台细节调整模块行为。

### Module constants / 模块常量
```python
logger = init_logger(__name__)
```
**EN:** This section defines module-level aliases, constants, or shared state such as `logger`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `logger`，供后续代码复用。

### Class `AgRsAll2AllManager` / 类 `AgRsAll2AllManager`
```python
class AgRsAll2AllManager(All2AllManagerBase):
    """
    An implementation of all2all communication based on
    all-gather (dispatch) and reduce-scatter (combine).
    """

    def __init__(self, cpu_group, tcp_store_group=None):
        super().__init__(cpu_group, tcp_store_group)

    def dispatch_router_logits(
        self,
        hidden_states: torch.Tensor,
        router_logits: torch.Tensor,
        is_sequence_parallel: bool = False,
        extra_tensors: list[torch.Tensor] | None = None,
    ) -> (
        tuple[torch.Tensor, torch.Tensor]
        | tuple[torch.Tensor, torch.Tensor, list[torch.Tensor]]
    ):
        """
        Gather hidden_states and router_logits from all dp ranks.
        """
        dp_metadata = get_forward_context().dp_metadata
        assert dp_metadata is not None
        sizes = dp_metadata.get_chunk_sizes_across_dp_rank()
        assert sizes is not None
        dist_group = get_ep_group() if is_sequence_parallel else get_dp_group()
        assert sizes[dist_group.rank_in_group] == hidden_states.shape[0]

        tensors_to_gather = [hidden_states, router_logits]
        if extra_tensors is not None:
            tensors_to_gather.extend(extra_tensors)

        gathered_tensors = dist_group.all_gatherv(
            tensors_to_gather,
            dim=0,
            sizes=sizes,
        )

        if extra_tensors is not None:
            return (gathered_tensors[0], gathered_tensors[1], gathered_tensors[2:])
        return gathered_tensors[0], gathered_tensors[1]

    def dispatch(
        self,
# ... truncated for analysis ...
        assert dp_metadata is not None
        sizes = dp_metadata.get_chunk_sizes_across_dp_rank()
        assert sizes is not None

        dist_group = get_ep_group() if is_sequence_parallel else get_dp_group()
        hidden_states = dist_group.reduce_scatterv(hidden_states, dim=0, sizes=sizes)
        return hidden_states

    def destroy(self):
        pass
```
**EN:** Declares `AgRsAll2AllManager`, a class derived from `All2AllManagerBase`. Key methods include `__init__`, `dispatch_router_logits`, `dispatch`, `combine`, `destroy`. The docstring summarizes its role as: An implementation of all2all communication based on all-gather (dispatch) and reduce-scatter (combine).
**CN:** 声明 `AgRsAll2AllManager`，它是一个类，继承自 `All2AllManagerBase`。 关键方法包括 `__init__`, `dispatch_router_logits`, `dispatch`, `combine`, `destroy`。 文档字符串概括了它在整体流程中的职责。

### Class `DeepEPAll2AllManagerBase` / 类 `DeepEPAll2AllManagerBase`
```python
class DeepEPAll2AllManagerBase(All2AllManagerBase):
    """
    All2All communication based on DeepEP High-Throughput kernels.
    """

    def __init__(self, cpu_group, tcp_store_group=None):
        assert has_deep_ep(), (
            "DeepEP kernels not found. Please follow https://github.com/vllm-project/vllm/blob/main/tools/ep_kernels/README.md"
            " to install DeepEP kernels."
        )  # noqa
        super().__init__(cpu_group, tcp_store_group)
        self.handle_cache = Cache()

        # This is the DeepEP default. Stick to it till we can establish
        # reasonable defaults based on profiling.
        self.num_sms = 20

    def get_handle(self, kwargs):
        raise NotImplementedError

    def dispatch_router_logits(
        self,
        hidden_states: torch.Tensor,
        router_logits: torch.Tensor,
        is_sequence_parallel: bool = False,
        extra_tensors: list[torch.Tensor] | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        raise NotImplementedError

    def dispatch(
        self,
        hidden_states: torch.Tensor,
        topk_weights: torch.Tensor,
        topk_ids: torch.Tensor,
        is_sequence_parallel: bool = False,
        extra_tensors: list[torch.Tensor] | None = None,
    ) -> (
        tuple[torch.Tensor, torch.Tensor, torch.Tensor]
        | tuple[torch.Tensor, torch.Tensor, torch.Tensor, list[torch.Tensor]]
    ):
        raise NotImplementedError

    def combine(
        self, hidden_states: torch.Tensor, is_sequence_parallel: bool = False
    ) -> torch.Tensor:
        raise NotImplementedError

    def destroy(self):
        with self.handle_cache._lock:
            for _, handle in self.handle_cache._cache.items():
                handle.destroy()
            self.handle_cache._cache.clear()
```
**EN:** Declares `DeepEPAll2AllManagerBase`, a class derived from `All2AllManagerBase`. Key methods include `__init__`, `get_handle`, `dispatch_router_logits`, `dispatch`, `combine`. The docstring summarizes its role as: All2All communication based on DeepEP High-Throughput kernels.
**CN:** 声明 `DeepEPAll2AllManagerBase`，它是一个类，继承自 `All2AllManagerBase`。 关键方法包括 `__init__`, `get_handle`, `dispatch_router_logits`, `dispatch`, `combine`。 文档字符串概括了它在整体流程中的职责。

### Class `DeepEPHTAll2AllManager` / 类 `DeepEPHTAll2AllManager`
```python
class DeepEPHTAll2AllManager(DeepEPAll2AllManagerBase):
    """
    All2All communication based on DeepEP High-Throughput kernels.
    """

    def __init__(self, cpu_group, tcp_store_group=None):
        super().__init__(cpu_group, tcp_store_group)

    def _make_all2all_kwargs(self) -> dict[Any, Any]:
        # Defaults for internode and intranode are taken from DeepEP tests.
        num_nvl_bytes = envs.VLLM_DEEPEP_BUFFER_SIZE_MB * 1024 * 1024
        num_rdma_bytes = None
        num_qps_per_rank = None

        if self.internode and not envs.VLLM_DEEPEP_HIGH_THROUGHPUT_FORCE_INTRA_NODE:
            num_rdma_bytes = envs.VLLM_DEEPEP_BUFFER_SIZE_MB * 1024 * 1024
            num_qps_per_rank = self.num_sms // 2
        else:
            num_rdma_bytes = 0
            num_qps_per_rank = 1

        assert num_rdma_bytes is not None
        assert num_qps_per_rank is not None
        # TODO: remove platform-specific logic
        # once ROCm DeepEP is updated with the latest APIs.
        kwargs = dict(
            group=self.cpu_group,
            num_nvl_bytes=num_nvl_bytes,
            num_rdma_bytes=num_rdma_bytes,
            low_latency_mode=False,
            num_qps_per_rank=num_qps_per_rank,
            explicitly_destroy=True,
        )
        return kwargs

    def get_handle(self, kwargs):
        assert len(kwargs) == 0, (
            "DeepEPHTAll2AllManager expects no arguments. All the required "
            "args are computed in the Manager itself."
        )

        import deep_ep  # type: ignore[import-not-found]

        buffer_kwargs = self._make_all2all_kwargs()
        logger.debug("DeepEP all2all args %s", buffer_kwargs)
        handle: deep_ep.Buffer = self.handle_cache.get_or_create(
            buffer_kwargs, deep_ep.Buffer
        )
        return handle

    def set_num_sms(self, num_sms: int):
        import deep_ep  # type: ignore[import-not-found]

        # Right now the buffers are sized for only what the kernels were
        # created with. So we can only reduce the number of SMS used
        # but not increase it.
        if num_sms > self.num_sms:
            num_sms = self.num_sms
        deep_ep.Buffer.set_num_sms(num_sms)
```
**EN:** Declares `DeepEPHTAll2AllManager`, a class derived from `DeepEPAll2AllManagerBase`. Key methods include `__init__`, `_make_all2all_kwargs`, `get_handle`, `set_num_sms`. The docstring summarizes its role as: All2All communication based on DeepEP High-Throughput kernels.
**CN:** 声明 `DeepEPHTAll2AllManager`，它是一个类，继承自 `DeepEPAll2AllManagerBase`。 关键方法包括 `__init__`, `_make_all2all_kwargs`, `get_handle`, `set_num_sms`。 文档字符串概括了它在整体流程中的职责。

### Class `DeepEPLLAll2AllManager` / 类 `DeepEPLLAll2AllManager`
```python
class DeepEPLLAll2AllManager(DeepEPAll2AllManagerBase):
    """
    All2All communication based on DeepEP Low-Latency kernels.
    """

    def __init__(self, cpu_group, tcp_store_group=None):
        super().__init__(cpu_group, tcp_store_group)

    def _make_all2all_kwargs(
        self,
        max_num_tokens_per_dp_rank: int,
        token_hidden_size: int,
        num_ep_ranks: int,
        num_global_experts: int,
        num_local_experts: int,
    ) -> dict[Any, Any]:
        """
        max_num_tokens_per_dp_rank : the maximum number of tokens a DP rank
          can dispatch all the ranks must hold the same value.
        token_hidden_size: the hidden dimension of each token.
        num_ep_ranks: the number of EP group ranks.
        num_global_experts: Number of experts in the model.
        num_local_experts: Number of experts in an EP rank.
        """
        import deep_ep  # type: ignore[import-not-found]

        # Defaults for internode and intranode are taken from DeepEP tests.
        num_nvl_bytes = envs.VLLM_DEEPEP_BUFFER_SIZE_MB * 1024 * 1024
        num_qps_per_rank = num_local_experts
        num_rdma_bytes = deep_ep.Buffer.get_low_latency_rdma_size_hint(
            num_max_dispatch_tokens_per_rank=max_num_tokens_per_dp_rank,
            hidden=token_hidden_size,
            num_ranks=num_ep_ranks,
            num_experts=num_global_experts,
        )

        assert num_rdma_bytes is not None
        # TODO: remove platform-specific logic
        # once ROCm DeepEP is updated with the latest APIs.
        kwargs = dict(
            group=self.cpu_group,
            num_nvl_bytes=num_nvl_bytes,
            num_rdma_bytes=num_rdma_bytes,
            low_latency_mode=True,
            num_qps_per_rank=num_qps_per_rank,
# ... truncated for analysis ...
        buffer_kwargs = self._make_all2all_kwargs(**kwargs)
        logger.debug("DeepEP all2all args %s", buffer_kwargs)
        handle: deep_ep.Buffer = self.handle_cache.get_or_create(
            buffer_kwargs, deep_ep.Buffer
        )
        return handle

    # DeepEP LL uses RDMA so no SMs are used for communication
    def max_sms_used(self) -> int | None:
        return 0
```
**EN:** Declares `DeepEPLLAll2AllManager`, a class derived from `DeepEPAll2AllManagerBase`. Key methods include `__init__`, `_make_all2all_kwargs`, `get_handle`, `max_sms_used`. The docstring summarizes its role as: All2All communication based on DeepEP Low-Latency kernels.
**CN:** 声明 `DeepEPLLAll2AllManager`，它是一个类，继承自 `DeepEPAll2AllManagerBase`。 关键方法包括 `__init__`, `_make_all2all_kwargs`, `get_handle`, `max_sms_used`。 文档字符串概括了它在整体流程中的职责。

### Class `NixlEPAll2AllManager` / 类 `NixlEPAll2AllManager`
```python
class NixlEPAll2AllManager(All2AllManagerBase):
    """
    All2All communication based on NIXL EP kernels.
    This backend supports elastic EP with dynamic rank connection/disconnection.
    """

    # (nixl_ep_buffer, ep_size)
    _buffer: tuple[Any, int] | None = None
    _lock = threading.Lock()

    def __init__(self, cpu_group, tcp_store_group=None):
        assert tcp_store_group is not None
        super().__init__(cpu_group, tcp_store_group)

        self.max_num_ep_ranks = envs.VLLM_NIXL_EP_MAX_NUM_RANKS

    def _init_buffer(
        self,
        max_num_tokens_per_dp_rank: int,
        token_hidden_size: int,
        num_experts_per_rank: int,
    ) -> None:
        from nixl_ep import Buffer  # type: ignore[import-not-found]

        max_num_global_experts = self.max_num_ep_ranks * num_experts_per_rank
        num_rdma_bytes = Buffer.get_rdma_size_hint(
            num_max_dispatch_tokens_per_rank=max_num_tokens_per_dp_rank,
            hidden=token_hidden_size,
            num_ranks=self.max_num_ep_ranks,
            num_experts=max_num_global_experts,
        )
        assert NixlEPAll2AllManager._buffer is None, (
            "NIXL EP buffer already initialized"
        )
        buffer = Buffer(
            rank=self.rank,
            tcp_store_group=self.tcp_store_group.store,
        )
        buffer.update_memory_buffers(
            num_ranks=self.max_num_ep_ranks,
            num_experts_per_rank=num_experts_per_rank,
            num_rdma_bytes=num_rdma_bytes,
        )
        ranks_to_connect = list(range(self.cpu_group.size()))
        buffer.connect_ranks(ranks_to_connect)
# ... truncated for analysis ...
    def destroy(self):
        # NOTE(yongji): NIXLEPAll2AllManager instance is recreated during
        # scale-up/down, so we cannot destroy the persistent buffer here.
        assert NixlEPAll2AllManager._buffer is not None
        buffer = NixlEPAll2AllManager._buffer[0]
        buffer.set_tcp_store_group(None)

    # NIXL EP uses RDMA so no SMs are used for communication
    def max_sms_used(self) -> int | None:
        return 0
```
**EN:** Declares `NixlEPAll2AllManager`, a class derived from `All2AllManagerBase`. Key methods include `__init__`, `_init_buffer`, `_update_buffer`, `get_handle`, `dispatch`. The docstring summarizes its role as: All2All communication based on NIXL EP kernels. This backend supports elastic EP with dynamic rank connection/disconnection.
**CN:** 声明 `NixlEPAll2AllManager`，它是一个类，继承自 `All2AllManagerBase`。 关键方法包括 `__init__`, `_init_buffer`, `_update_buffer`, `get_handle`, `dispatch`。 文档字符串概括了它在整体流程中的职责。

### Class `FlashInferNVLinkTwoSidedManager` / 类 `FlashInferNVLinkTwoSidedManager`
```python
class FlashInferNVLinkTwoSidedManager(All2AllManagerBase):
    """
    All2All communication based on flashinfer all2allv/two-sided NVLink kernels.
    """

    # This type lint could be removed after all of the work in
    # https://github.com/vllm-project/vllm/issues/26533 done.
    rank: int
    world_size: int

    def __init__(self, cpu_group, tcp_store_group=None):
        assert has_flashinfer_nvlink_two_sided(), (
            "flashinfer all2all module not found. Please install/check flashinfer"
        )  # noqa
        super().__init__(cpu_group, tcp_store_group)
        logger.debug(
            "Initialize for flashinfer All2All rank=%d, world size=%d",
            self.rank,
            self.world_size,
        )
        self.initialized = False
        self.alltoall_info = None

    def initialize(
        self,
        world_size: int,
        rank: int,
        gpus_per_node: int,
    ):
        """Initialize workspace"""
        if self.initialized:
            return

        self.cleanup()
        logger.debug("making map: rank=%d, world size=%d", rank, world_size)
        self.mapping = Mapping(
            world_size,
            rank,
            gpus_per_node,
            tp_size=world_size,
        )

        from vllm.distributed.device_communicators.mnnvl_compat import (
            CustomCommunicator,
        )
# ... truncated for analysis ...
            try:
                del self.workspace_tensor
                del self.prepare_workspace_tensor
            except Exception as e:
                logger.warning("Failed to cleanup FlashInfer workspace: %s", e)
            finally:
                self.workspace_tensor = None
                self.prepare_workspace_tensor = None
                self.mapping = None
                self.initialized = False
```
**EN:** Declares `FlashInferNVLinkTwoSidedManager`, a class derived from `All2AllManagerBase`. Key methods include `__init__`, `initialize`, `ensure_alltoall_workspace_initialized`, `get_handle`, `cleanup`. The docstring summarizes its role as: All2All communication based on flashinfer all2allv/two-sided NVLink kernels.
**CN:** 声明 `FlashInferNVLinkTwoSidedManager`，它是一个类，继承自 `All2AllManagerBase`。 关键方法包括 `__init__`, `initialize`, `ensure_alltoall_workspace_initialized`, `get_handle`, `cleanup`。 文档字符串概括了它在整体流程中的职责。

### Class `FlashInferNVLinkOneSidedManager` / 类 `FlashInferNVLinkOneSidedManager`
```python
class FlashInferNVLinkOneSidedManager(All2AllManagerBase):
    """
    All2All communication based on FlashInfer's MoeAlltoAll/One-sided NVLink kernel.
    This is a newer kernel from trtllm that should perform better than the kernel
    used by flashinfer_nvlink_two_sided.
    """

    rank: int
    world_size: int

    def __init__(self, cpu_group):
        assert has_flashinfer_nvlink_one_sided(), (
            "flashinfer trtllm_moe_alltoall module not found. "
            "Please install/check flashinfer"
        )
        super().__init__(cpu_group)
        logger.debug(
            "Initialize FlashInfer One-sided NVLink rank=%d, world size=%d",
            self.rank,
            self.world_size,
        )
        self.initialized = False
        self.moe_alltoall: MoeAlltoAll | None = None
        self.mapping = None

    def initialize(
        self,
        max_num_tokens: int,
        top_k: int,
        num_experts: int,
        hidden_size: int,
        dispatch_dtype_bytes_per_elem: int = 0,
        dispatch_scale_bytes_per_token: int = 0,
    ):
        """Initialize the MoeAlltoAll workspace."""
        if self.initialized:
            return

        self.cleanup()
        gpus_per_node = torch.accelerator.device_count()
        logger.debug(
            "Making One-sided NVLink mapping: rank=%d, world size=%d",
            self.rank,
            self.world_size,
        )
# ... truncated for analysis ...
            try:
                del self.moe_alltoall
            except Exception as e:
                logger.warning(
                    "Failed to cleanup FlashInfer One-sided NVLink workspace: %s", e
                )
            finally:
                self.moe_alltoall = None
                self.mapping = None
                self.initialized = False
```
**EN:** Declares `FlashInferNVLinkOneSidedManager`, a class derived from `All2AllManagerBase`. Key methods include `__init__`, `initialize`, `get_handle`, `cleanup`. The docstring summarizes its role as: All2All communication based on FlashInfer's MoeAlltoAll/One-sided NVLink kernel. This is a newer kernel from trtllm that should perform....
**CN:** 声明 `FlashInferNVLinkOneSidedManager`，它是一个类，继承自 `All2AllManagerBase`。 关键方法包括 `__init__`, `initialize`, `get_handle`, `cleanup`。 文档字符串概括了它在整体流程中的职责。

### Class `MoriAll2AllManager` / 类 `MoriAll2AllManager`
```python
class MoriAll2AllManager(All2AllManagerBase):
    def __init__(self, cpu_group):
        assert has_mori(), (
            "MoRI kernels not found. Please follow https://github.com/ROCm/mori/blob/main/README.md"
            " to install MoRI kernels."
        )  # noqa
        import mori

        super().__init__(cpu_group)
        self.handle_cache = Cache()

        torch._C._distributed_c10d._register_process_group("mori", cpu_group)
        mori.shmem.shmem_torch_process_group_init("mori")

    def _make_all2all_kwargs(
        self,
        rank: int,
        num_ep_ranks: int,
        input_dtype: torch.dtype,
        quant_dtype: torch.dtype,
        token_hidden_size: int,
        scale_dim: int,
        scale_type_size: int,
        max_num_tokens_per_dp_rank: int,
        num_local_experts: int,
        num_experts_per_token: int,
    ):
        import mori  # type: ignore[import-not-found]

        from vllm.platforms.rocm import on_gfx942, on_gfx950

        assert on_gfx942() or on_gfx950(), (
            "mori currently only support arch gfx942 and gfx950"
        )

        if not self.internode:
            # single node
            kernel_type = mori.ops.EpDispatchCombineKernelType.IntraNode
            rdma_block_num = 0
            warp_num_per_block = 16
            block_num = 80
        else:
            # multi node
            kernel_type = mori.ops.EpDispatchCombineKernelType.InterNodeV1
            if on_gfx942():
# ... truncated for analysis ...

    def get_handle(self, kwargs):
        import mori  # type: ignore[import-not-found]

        mori_kwargs = self._make_all2all_kwargs(**kwargs)
        logger.debug("MoRI all2all args %s", mori_kwargs)
        handle: mori.ops.EpDispatchCombineOp = self.handle_cache.get_or_create(
            mori_kwargs, self._make_handle
        )
        return handle
```
**EN:** Declares `MoriAll2AllManager`, a class derived from `All2AllManagerBase`. Key methods include `__init__`, `_make_all2all_kwargs`, `_make_handle`, `get_handle`.
**CN:** 声明 `MoriAll2AllManager`，它是一个类，继承自 `All2AllManagerBase`。 关键方法包括 `__init__`, `_make_all2all_kwargs`, `_make_handle`, `get_handle`。

## Key Concepts / 关键概念
- Collective communication backends / collective 通信后端
- `AgRsAll2AllManager`: class interface or data carrier / `AgRsAll2AllManager`：类接口或数据载体
- `DeepEPAll2AllManagerBase`: class interface or data carrier / `DeepEPAll2AllManagerBase`：类接口或数据载体
- `DeepEPHTAll2AllManager`: class interface or data carrier / `DeepEPHTAll2AllManager`：类接口或数据载体
- `DeepEPLLAll2AllManager`: class interface or data carrier / `DeepEPLLAll2AllManager`：类接口或数据载体
- `NixlEPAll2AllManager`: class interface or data carrier / `NixlEPAll2AllManager`：类接口或数据载体

## Dependencies / 依赖关系
- **Standard library / 标准库**: `threading`, `typing`
- **Third-party / 第三方**: `torch`, `torch.distributed`, `flashinfer.comm`, `flashinfer.comm.mnnvl`, `flashinfer.comm.trtllm_alltoall`, `flashinfer.comm.trtllm_moe_alltoall`, `deep_ep`, `nixl_ep`, `mori`
- **Internal modules / 内部模块**: `vllm.envs`, `vllm.distributed`, `vllm.forward_context`, `vllm.logger`, `vllm.utils.flashinfer`, `vllm.utils.import_utils`, `.base_device_communicator`, `vllm.distributed.device_communicators.mnnvl_compat`, `vllm.platforms.rocm`
