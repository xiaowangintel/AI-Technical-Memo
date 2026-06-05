# moriep.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/token_dispatcher/moriep.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements distributed communication and tensor movement with Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `MoriEPPDispatchHooks`, `MoriEPNormalDispatchOutput`, `MoriEPLLDispatchOutput`, and `MoriEPNormalCombineInput` and connects them to backend-specific paths such as `CUDA`, `NPU`, and `CPU`. / 该模块为 SGLang 的 SRT 运行时实现了分布式通信与张量搬运，并结合混合专家路由与专家执行。它提供了 `MoriEPPDispatchHooks`、`MoriEPNormalDispatchOutput`、`MoriEPLLDispatchOutput` 以及 `MoriEPNormalCombineInput` 等符号，并把这些符号连接到 `CUDA`、`NPU` 以及 `CPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-55: Imports, conditional backend setup, and runtime guards
```python
from __future__ import annotations

import logging
import os
from dataclasses import dataclass
from typing import TYPE_CHECKING, List, NamedTuple, Optional, Tuple

from sglang.srt.layers.dp_attention import get_is_extend_in_batch
from sglang.srt.layers.moe.token_dispatcher.base import (
    BaseDispatcher,
    CombineInput,
    CombineInputFormat,
    DispatchOutput,
    DispatchOutputFormat,
)
from sglang.srt.layers.moe.token_dispatcher.deepep import DeepEPPDispatchHooks
from sglang.srt.layers.moe.topk import TopKOutput
from sglang.srt.layers.moe.utils import (
    DeepEPMode,
    is_tbo_enabled,
)
from sglang.srt.utils import (
    get_bool_env_var,
    get_int_env_var,
    is_hip,
)

if TYPE_CHECKING:
    from sglang.srt.single_batch_overlap import CombineOverlapArgs
    import mori

from enum import Enum, auto
from functools import lru_cache

import torch

from sglang.srt.distributed import (
    get_moe_expert_parallel_rank,
    get_moe_expert_parallel_world_size,
)
from sglang.srt.layers.quantization.fp8_kernel import fp8_dtype

# Blockwise quantization group sizes: number of elements sharing one scale factor
FP8_BLOCK_SIZE = 128
MXFP4_BLOCK_SIZE = 32

_is_hip = is_hip()
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip

if _use_aiter:
    from aiter import QuantType, get_hip_quant

logger = logging.getLogger(__name__)
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `logging`, `os`, `dataclasses.dataclass`, `typing.TYPE_CHECKING`, and `typing.List`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `FP8_BLOCK_SIZE`, `MXFP4_BLOCK_SIZE`, `_is_hip`, `_use_aiter`, and `logger` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`logging`、`os`、`dataclasses.dataclass`、`typing.TYPE_CHECKING` 以及 `typing.List`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `FP8_BLOCK_SIZE`、`MXFP4_BLOCK_SIZE`、`_is_hip`、`_use_aiter` 以及 `logger` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 56-57: Class `MoriEPPDispatchHooks` declaration and shared state
```python
class MoriEPPDispatchHooks(DeepEPPDispatchHooks):
```
**EN:** This block introduces class `MoriEPPDispatchHooks` and the state shared by its methods. It inherits from `DeepEPPDispatchHooks`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `MoriEPPDispatchHooks`，并定义其方法共享的状态。 它继承自 `DeepEPPDispatchHooks`，说明了它在 SRT 层栈中的接入方式。

### Lines 58-62: `MoriEPPDispatchHooks.__call__` callable execution entry
```python
    def __call__(self, dispatcher: BaseDispatcher):
        for hook_fun in self.hook_dict.values():
            hook_fun(dispatcher)
```
**EN:** This block defines `MoriEPPDispatchHooks.__call__` and contains the main logic for this step. It mainly invokes `self.hook_dict.values` and `hook_fun`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `MoriEPPDispatchHooks.__call__`，并承载这一阶段的核心逻辑。 它主要调用 `self.hook_dict.values` 和 `hook_fun`，说明该流程会编排底层辅助函数或计算内核。

### Lines 63-74: Class `MoriEPNormalDispatchOutput` declaration and shared state
```python
class MoriEPNormalDispatchOutput(NamedTuple):
    """Mori EP normal dispatch output."""

    hidden_states: torch.Tensor
    hidden_states_scale: Optional[torch.Tensor]
    topk_ids: torch.Tensor
    topk_weights: torch.Tensor
    num_recv_tokens_per_expert: List[int]
    origin_topk_ids: torch.Tensor
    origin_topk_weights: torch.Tensor
    out_dtype: torch.dtype
```
**EN:** This block introduces class `MoriEPNormalDispatchOutput` and the state shared by its methods. It inherits from `NamedTuple`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Mori EP normal dispatch output.
**CN:** 该代码块引入类 `MoriEPNormalDispatchOutput`，并定义其方法共享的状态。 它继承自 `NamedTuple`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 75-79: Function `MoriEPNormalDispatchOutput.format` and its core logic
```python
    @property
    def format(self) -> DispatchOutputFormat:
        return DispatchOutputFormat.DEEPEP_NORMAL
```
**EN:** This block defines `MoriEPNormalDispatchOutput.format` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoriEPNormalDispatchOutput.format`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 80-91: Class `MoriEPLLDispatchOutput` declaration and shared state
```python
class MoriEPLLDispatchOutput(NamedTuple):
    """Mori EP low latency dispatch output."""

    hidden_states: torch.Tensor
    hidden_states_scale: Optional[torch.Tensor]
    topk_ids: torch.Tensor
    topk_weights: torch.Tensor
    num_recv_tokens_per_expert: List[int]
    origin_topk_ids: torch.Tensor
    origin_topk_weights: torch.Tensor
    out_dtype: torch.dtype
```
**EN:** This block introduces class `MoriEPLLDispatchOutput` and the state shared by its methods. It inherits from `NamedTuple`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Mori EP low latency dispatch output.
**CN:** 该代码块引入类 `MoriEPLLDispatchOutput`，并定义其方法共享的状态。 它继承自 `NamedTuple`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 92-96: Function `MoriEPLLDispatchOutput.format` and its core logic
```python
    @property
    def format(self) -> DispatchOutputFormat:
        return DispatchOutputFormat.DEEPEP_LL
```
**EN:** This block defines `MoriEPLLDispatchOutput.format` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoriEPLLDispatchOutput.format`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 97-100: Module-level helper logic
```python
assert isinstance(MoriEPNormalDispatchOutput, DispatchOutput)
assert isinstance(MoriEPLLDispatchOutput, DispatchOutput)
```
**EN:** This section prepares the module namespace.
**CN:** 该部分负责准备模块命名空间。

### Lines 101-107: Class `MoriEPNormalCombineInput` declaration and shared state
```python
class MoriEPNormalCombineInput(NamedTuple):
    """Mori EP combine input."""

    hidden_states: torch.Tensor
    topk_ids: torch.Tensor
    topk_weights: torch.Tensor
```
**EN:** This block introduces class `MoriEPNormalCombineInput` and the state shared by its methods. It inherits from `NamedTuple`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Mori EP combine input.
**CN:** 该代码块引入类 `MoriEPNormalCombineInput`，并定义其方法共享的状态。 它继承自 `NamedTuple`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 108-112: Function `MoriEPNormalCombineInput.format` and its core logic
```python
    @property
    def format(self) -> CombineInputFormat:
        return CombineInputFormat.DEEPEP_NORMAL
```
**EN:** This block defines `MoriEPNormalCombineInput.format` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoriEPNormalCombineInput.format`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 113-119: Class `MoriEPLLCombineInput` declaration and shared state
```python
class MoriEPLLCombineInput(NamedTuple):
    """Mori EP combine input."""

    hidden_states: torch.Tensor
    topk_ids: torch.Tensor
    topk_weights: torch.Tensor
```
**EN:** This block introduces class `MoriEPLLCombineInput` and the state shared by its methods. It inherits from `NamedTuple`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Mori EP combine input.
**CN:** 该代码块引入类 `MoriEPLLCombineInput`，并定义其方法共享的状态。 它继承自 `NamedTuple`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 120-124: Function `MoriEPLLCombineInput.format` and its core logic
```python
    @property
    def format(self) -> CombineInputFormat:
        return CombineInputFormat.DEEPEP_LL
```
**EN:** This block defines `MoriEPLLCombineInput.format` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoriEPLLCombineInput.format`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 125-128: Module-level helper logic
```python
assert isinstance(MoriEPNormalCombineInput, CombineInput)
assert isinstance(MoriEPLLCombineInput, CombineInput)
```
**EN:** This section prepares the module namespace.
**CN:** 该部分负责准备模块命名空间。

### Lines 129-134: Class `EpMode` declaration and shared state
```python
class EpMode(Enum):
    INTRA_NODE = "intra_node"
    INTER_NODE = "inter_node"
    LOW_LATENCY = "low_latency"
```
**EN:** This block introduces class `EpMode` and the state shared by its methods. It inherits from `Enum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `EpMode`，并定义其方法共享的状态。 它继承自 `Enum`，说明了它在 SRT 层栈中的接入方式。

### Lines 135-140: Class `DispatchDtype` declaration and shared state
```python
class DispatchDtype(Enum):
    bf16 = "bfloat16"
    fp8 = "float8_blockwise"
    fp4 = "mxfp4_blockwise"
```
**EN:** This block introduces class `DispatchDtype` and the state shared by its methods. It inherits from `Enum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `DispatchDtype`，并定义其方法共享的状态。 它继承自 `Enum`，说明了它在 SRT 层栈中的接入方式。

### Lines 141-146: Class `CombineDtype` declaration and shared state
```python
class CombineDtype(Enum):
    bf16 = "bfloat16"
    fp8 = "float8_blockwise"
    fp8_direct_cast = "float8_direct_cast"
```
**EN:** This block introduces class `CombineDtype` and the state shared by its methods. It inherits from `Enum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `CombineDtype`，并定义其方法共享的状态。 它继承自 `Enum`，说明了它在 SRT 层栈中的接入方式。

### Lines 147-154: Class `EpDispatchConfig` declaration and shared state
```python
@dataclass(frozen=True)
class EpDispatchConfig:
    kernel_type: mori.ops.EpDispatchCombineKernelType
    warp_num_per_block: int
    block_num: int
    rdma_block_num: int
```
**EN:** This block introduces class `EpDispatchConfig` and the state shared by its methods.
**CN:** 该代码块引入类 `EpDispatchConfig`，并定义其方法共享的状态。

### Lines 155-195: `get_ep_dispatch_configs` getter for expert parallel dispatch configs
```python
def get_ep_dispatch_configs(num_max_dispatch_tokens_per_rank: int = 4096):
    import mori

    # Selects the inter-node kernel. `InterNodeV1LL` is used if `num_max_dispatch_tokens_per_rank`
    # is less than or equal to the threshold, otherwise `InterNodeV1` is used. The threshold defaults to 256.
    inter_kernel_switch_threshold = get_int_env_var(
        "SGLANG_MORI_DISPATCH_INTER_KERNEL_SWITCH_THRESHOLD", 256
    )

    inter_kernel_type = (
        mori.ops.EpDispatchCombineKernelType.InterNodeV1LL
        if num_max_dispatch_tokens_per_rank <= inter_kernel_switch_threshold
        else mori.ops.EpDispatchCombineKernelType.InterNodeV1
    )

    return {
        # TODO(billishyahao): need to tune different configs for intra node async
        # Also could be tuned for different AMD platform
        EpMode.INTRA_NODE: EpDispatchConfig(
            kernel_type=mori.ops.EpDispatchCombineKernelType.IntraNode,
            warp_num_per_block=16,
            block_num=80,
            rdma_block_num=0,
        ),
        EpMode.INTER_NODE: EpDispatchConfig(
            kernel_type=inter_kernel_type,
            warp_num_per_block=8,
            block_num=64,
            rdma_block_num=32,
        ),
        EpMode.LOW_LATENCY: EpDispatchConfig(
            kernel_type=mori.ops.EpDispatchCombineKernelType.AsyncLL,
            warp_num_per_block=8,
            block_num=64,
            rdma_block_num=32,
        ),
    }


# init_mori_op only needs do once in model initial stage
# use lru_cache to reuse the same mori_op instance to avoid the init overhead for mori
```
**EN:** This block defines `get_ep_dispatch_configs` and contains the main logic for this step. It mainly invokes `get_int_env_var` and `EpDispatchConfig`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inter_kernel_switch_threshold` and `inter_kernel_type` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_ep_dispatch_configs`，并承载这一阶段的核心逻辑。 它主要调用 `get_int_env_var` 和 `EpDispatchConfig`，说明该流程会编排底层辅助函数或计算内核。 像 `inter_kernel_switch_threshold` 和 `inter_kernel_type` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 196-331: Function `init_mori_op` and its core logic
```python
@lru_cache(maxsize=4)
def init_mori_op(
    group,
    router_topk,
    num_experts,
    num_local_experts,
    hidden_size,
    params_dtype,
    num_max_dispatch_tokens_per_rank,
    deepep_mode,
    instance_id=0,
    dispatch_dtype=DispatchDtype.bf16,
    combine_dtype=CombineDtype.bf16,
    enable_sdma=False,
):

    import mori

    world_size = get_moe_expert_parallel_world_size()
    rank = get_moe_expert_parallel_rank()

    gpu_per_node = 8 if world_size >= 8 else world_size

    group_name = f"mori"
    cpu_group = group.cpu_group
    try:
        torch._C._distributed_c10d._register_process_group(group_name, cpu_group)
    except Exception as e:
        if "already registered" in str(e):
            logger.info(
                f"[MORI init] The same process group is already "
                f"registered. Ignoring [{str(e)}]"
            )
        else:
            raise
    else:
        # If new group is newly registered then need to init mori shmem. However
        # if the group is registered already then need to skip init mori shmem
        # and reuse the previous one.
        mori.shmem.shmem_torch_process_group_init(group_name)

    mode = EpMode.INTRA_NODE if world_size <= 8 else EpMode.INTER_NODE
    async_mode = deepep_mode.enable_low_latency() or enable_sdma
    if async_mode:
        mode = EpMode.LOW_LATENCY

    cfg = get_ep_dispatch_configs(num_max_dispatch_tokens_per_rank)[mode]

    kernel_type = cfg.kernel_type
    warp_num_per_block = cfg.warp_num_per_block
    block_num = cfg.block_num
    rdma_block_num = cfg.rdma_block_num

    hidden_dim = hidden_size
    scale_dim = 1
    data_type = fp8_dtype
    scale_type_size = torch.float32.itemsize

    if dispatch_dtype == DispatchDtype.fp8:
        scale_dim = hidden_size // FP8_BLOCK_SIZE
    elif dispatch_dtype == DispatchDtype.fp4:
        # FP4 kernel still takes the original hidden size and do quantization
        # internally, so hidden_dim is not reduced. The reason is that for FP4
        # quantization, we need to keep the original hidden size to calculate
        # the quantization scale correctly. Don't use packed hidden size for FP4 kernel.
        hidden_dim = hidden_size
        scale_dim = hidden_size // MXFP4_BLOCK_SIZE
        data_type = torch.float4_e2m1fn_x2
        scale_type_size = torch.float8_e8m0fnu.itemsize

        if mode == EpMode.INTRA_NODE:
            if num_max_dispatch_tokens_per_rank < 128:
                block_num = 225
                warp_num_per_block = 5
            else:
                block_num = 256
                warp_num_per_block = 16

    # Fp8 blockwise combine uses its own internal scale_dim driven which can be
    # overridden by env ``MORI_FP8_COMBINE_SCALE_DIM`` (default 56)
    # See https://github.com/ROCm/mori/blob/96ffa169710f214e76e07abe5008d686fe54522b/python/mori/ops/dispatch_combine.py#L81-L84
    combine_quant_type = "none"
    if combine_dtype == CombineDtype.fp8:
        combine_quant_type = "fp8_blockwise"
    elif combine_dtype == CombineDtype.fp8_direct_cast:
        combine_quant_type = "fp8_direct_cast"

    logger.info(
        f"[MORI init] {world_size=} {rank=} {hidden_size=} {params_dtype=} "
        f"{num_max_dispatch_tokens_per_rank=} {num_local_experts=} "
        f"{router_topk=} {mode=} {dispatch_dtype=} {combine_dtype=} "
    )

    def check_mori_compatibility(kwargs: dict) -> None:
        """Remove kwargs not accepted by the installed mori's EpDispatchCombineConfig."""
        import dataclasses

        config_cls = mori.ops.EpDispatchCombineConfig
        valid_kwargs = {f.name for f in dataclasses.fields(config_cls)}

        invalid_kwargs = set(kwargs.keys()) - valid_kwargs
        for arg in invalid_kwargs:
            logger.warning(f"[MORI compat] Removing incompatible argument {arg} ")
            del kwargs[arg]

    # Definition refer to https://github.com/ROCm/mori/blob/f9be5ee2e5ac87256b9523399ae9d4d0e8a54f53/python/mori/ops/dispatch_combine.py#L66-L121
    common_kwargs = dict(
        data_type=data_type,
        rank=rank,
        world_size=world_size,
        hidden_dim=hidden_dim,
        scale_dim=scale_dim,
        scale_type_size=scale_type_size,
        max_token_type_size=params_dtype.itemsize,
        max_num_inp_token_per_rank=num_max_dispatch_tokens_per_rank,
        num_experts_per_rank=num_local_experts,
        num_experts_per_token=router_topk,
        warp_num_per_block=warp_num_per_block,
        block_num=block_num,
        max_total_recv_tokens=get_int_env_var(
            "SGLANG_MORI_PREALLOC_MAX_RECV_TOKENS", 0
        ),
        kernel_type=kernel_type,
        gpu_per_node=gpu_per_node,
        rdma_block_num=rdma_block_num,
        num_qp_per_pe=2,  # Number of queue pairs per processing element
        quant_type=combine_quant_type,
    )

    check_mori_compatibility(common_kwargs)

    mori_config = mori.ops.EpDispatchCombineConfig(**common_kwargs)
    mori_op = mori.ops.EpDispatchCombineOp(mori_config)
    return mori_op
```
**EN:** This block defines `init_mori_op` and contains the main logic for this step. Decorators like `lru_cache` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `lru_cache`, `get_moe_expert_parallel_world_size`, `get_moe_expert_parallel_rank`, `logger.info`, and `dict`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `world_size`, `rank`, `gpu_per_node`, `group_name`, and `cpu_group` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `init_mori_op`，并承载这一阶段的核心逻辑。 像 `lru_cache` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `lru_cache`、`get_moe_expert_parallel_world_size`、`get_moe_expert_parallel_rank`、`logger.info` 以及 `dict`，说明该流程会编排底层辅助函数或计算内核。 像 `world_size`、`rank`、`gpu_per_node`、`group_name` 以及 `cpu_group` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 332-334: Class `CommStreamPool` declaration and shared state
```python
class CommStreamPool:
    _streams = {}  # key -> torch.cuda.Stream
```
**EN:** This block introduces class `CommStreamPool` and the state shared by its methods.
**CN:** 该代码块引入类 `CommStreamPool`，并定义其方法共享的状态。

### Lines 335-338: Internal helper `CommStreamPool._make_key`
```python
    @classmethod
    def _make_key(cls, group):
        return (torch.cuda.current_device(), id(group))
```
**EN:** This block defines `CommStreamPool._make_key` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `torch.cuda.current_device` and `id`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CommStreamPool._make_key`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `torch.cuda.current_device` 和 `id`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 339-347: `CommStreamPool.get_stream_from_pool` getter for stream from pool
```python
    @classmethod
    def get_stream_from_pool(cls, group) -> torch.cuda.Stream:
        key = cls._make_key(group)
        stream = cls._streams.get(key)
        if stream is None:
            stream = torch.cuda.Stream(priority=0)
            cls._streams[key] = stream
        return stream
```
**EN:** This block defines `CommStreamPool.get_stream_from_pool` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `cls._make_key`, `cls._streams.get`, and `torch.cuda.Stream`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `key` and `stream` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CommStreamPool.get_stream_from_pool`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `cls._make_key`、`cls._streams.get` 以及 `torch.cuda.Stream`，说明该流程会编排底层辅助函数或计算内核。 像 `key` 和 `stream` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 348-353: Function `CommStreamPool.clear_group` and its core logic
```python
    @classmethod
    def clear_group(cls, group):
        key = (torch.cuda.current_device(), id(group))
        cls._streams.pop(key, None)
```
**EN:** This block defines `CommStreamPool.clear_group` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `cls._streams.pop`, `torch.cuda.current_device`, and `id`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `key` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `CommStreamPool.clear_group`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `cls._streams.pop`、`torch.cuda.current_device` 以及 `id`，说明该流程会编排底层辅助函数或计算内核。 像 `key` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 354-354: Class `_MoriEPDispatcherImplBase` declaration and shared state
```python
class _MoriEPDispatcherImplBase:
```
**EN:** This block introduces class `_MoriEPDispatcherImplBase` and the state shared by its methods.
**CN:** 该代码块引入类 `_MoriEPDispatcherImplBase`，并定义其方法共享的状态。

### Lines 355-395: `_MoriEPDispatcherImplBase` initialization and state setup
```python
    def __init__(
        self,
        group: torch.distributed.ProcessGroup,
        router_topk: int,
        permute_fusion: bool,
        num_experts: int,
        num_local_experts: int,
        hidden_size: int,
        params_dtype: torch.dtype,
        deepep_mode: DeepEPMode,
        instance_id: int = 0,
    ):
        try:
            import mori  # noqa: F401
        except ImportError:
            raise ImportError("Mori EP is not installed. Please install.")
        self.group = group
        self.router_topk = router_topk
        self.permute_fusion = permute_fusion
        self.num_experts = num_experts
        self.num_local_experts = num_local_experts
        self.hidden_size = hidden_size
        self.params_dtype = params_dtype
        self.deepep_mode = deepep_mode
        self.instance_id = instance_id

        self.num_max_dispatch_tokens_per_rank = get_int_env_var(
            "SGLANG_MORI_NUM_MAX_DISPATCH_TOKENS_PER_RANK", 4096
        )

        self.enable_sdma = get_bool_env_var("MORI_ENABLE_SDMA", "false")

        self._mori_op = None
        self.dispatch_dtype = DispatchDtype.bf16
        self.combine_dtype = CombineDtype.bf16

        self.quant_config: Optional[dict] = None

        self.overlap_args: Optional[CombineOverlapArgs] = None
        self.meta_overlap_args: Optional[dict] = None
```
**EN:** This block defines `_MoriEPDispatcherImplBase.__init__` and contains the main logic for this step. It mainly invokes `get_int_env_var`, `get_bool_env_var`, and `ImportError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.group`, `self.router_topk`, `self.permute_fusion`, `self.num_experts`, and `self.num_local_experts` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplBase.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `get_int_env_var`、`get_bool_env_var` 以及 `ImportError`，说明该流程会编排底层辅助函数或计算内核。 像 `self.group`、`self.router_topk`、`self.permute_fusion`、`self.num_experts` 以及 `self.num_local_experts` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 396-417: Function `_MoriEPDispatcherImplBase.mori_op` and its core logic
```python
    @property
    def mori_op(self):
        if self._mori_op is None:
            # If set_quant_config was never called, apply env var override now
            if self.quant_config is None:
                self._apply_dispatch_dtype_override()
            self._mori_op = init_mori_op(
                self.group,
                self.router_topk,
                self.num_experts,
                self.num_local_experts,
                self.hidden_size,
                self.params_dtype,
                self.num_max_dispatch_tokens_per_rank,
                self.deepep_mode,
                self.instance_id,
                self.dispatch_dtype,
                self.combine_dtype,
                self.enable_sdma,
            )
        return self._mori_op
```
**EN:** This block defines `_MoriEPDispatcherImplBase.mori_op` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `init_mori_op` and `self._apply_dispatch_dtype_override`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self._mori_op` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplBase.mori_op`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `init_mori_op` 和 `self._apply_dispatch_dtype_override`，说明该流程会编排底层辅助函数或计算内核。 像 `self._mori_op` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 418-461: Internal helper `_MoriEPDispatcherImplBase._apply_dispatch_dtype_override`
```python
    def _apply_dispatch_dtype_override(self):
        """Apply env var override to fp8_dispatch/fp4_dispatch/fp8_combine flags."""
        if "SGLANG_MORI_DISPATCH_DTYPE" in os.environ:
            dispatch_dtype = os.environ["SGLANG_MORI_DISPATCH_DTYPE"].lower()
            if dispatch_dtype != "auto":
                if dispatch_dtype == "bf16":
                    self.dispatch_dtype = DispatchDtype.bf16
                elif dispatch_dtype == "fp8":
                    self.dispatch_dtype = DispatchDtype.fp8
                elif dispatch_dtype == "fp4":
                    self.dispatch_dtype = DispatchDtype.fp4
        elif (
            "SGLANG_MORI_FP8_DISP" in os.environ or "SGLANG_MORI_FP4_DISP" in os.environ
        ):
            # Deprecated: will be removed in a future release
            logger.warning_once(
                "SGLANG_MORI_FP8_DISP and SGLANG_MORI_FP4_DISP are deprecated "
                "and will be removed in a future release. "
                "Use SGLANG_MORI_DISPATCH_DTYPE=auto|bf16|fp8|fp4 instead."
            )
            if get_bool_env_var("SGLANG_MORI_FP8_DISP", "False"):
                self.dispatch_dtype = DispatchDtype.fp8
            if get_bool_env_var("SGLANG_MORI_FP4_DISP", "False"):
                self.dispatch_dtype = DispatchDtype.fp4

        if "SGLANG_MORI_COMBINE_DTYPE" in os.environ:
            combine_dtype = os.environ["SGLANG_MORI_COMBINE_DTYPE"].lower()
            if combine_dtype != "auto":
                if combine_dtype == "fp8":
                    self.combine_dtype = CombineDtype.fp8
                elif combine_dtype == "bf16":
                    self.combine_dtype = CombineDtype.bf16
                elif combine_dtype == "fp8_direct_cast":
                    self.combine_dtype = CombineDtype.fp8_direct_cast
        elif "SGLANG_MORI_FP8_COMB" in os.environ:
            # Deprecated: will be removed in a future release
            logger.warning_once(
                "SGLANG_MORI_FP8_COMB is deprecated "
                "and will be removed in a future release. "
                "Use SGLANG_MORI_COMBINE_DTYPE=auto|bf16|fp8|fp8_direct_cast instead."
            )
            if get_bool_env_var("SGLANG_MORI_FP8_COMB", "False"):
                self.combine_dtype = CombineDtype.fp8
```
**EN:** This block defines `_MoriEPDispatcherImplBase._apply_dispatch_dtype_override` and contains the main logic for this step. It mainly invokes `os.environ.lower`, `logger.warning_once`, and `get_bool_env_var`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `dispatch_dtype`, `combine_dtype`, `self.dispatch_dtype`, and `self.combine_dtype` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplBase._apply_dispatch_dtype_override`，并承载这一阶段的核心逻辑。 它主要调用 `os.environ.lower`、`logger.warning_once` 以及 `get_bool_env_var`，说明该流程会编排底层辅助函数或计算内核。 像 `dispatch_dtype`、`combine_dtype`、`self.dispatch_dtype` 以及 `self.combine_dtype` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 462-468: `_MoriEPDispatcherImplBase.dispatch_a` step for a
```python
    def dispatch_a(
        self,
        hidden_states: torch.Tensor,
        topk_output: TopKOutput,
    ):
        raise NotImplementedError
```
**EN:** This block defines `_MoriEPDispatcherImplBase.dispatch_a` and contains the main logic for this step.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplBase.dispatch_a`，并承载这一阶段的核心逻辑。

### Lines 469-471: `_MoriEPDispatcherImplBase.dispatch_b` step for b
```python
    def dispatch_b(self, *args, **kwargs):
        raise NotImplementedError
```
**EN:** This block defines `_MoriEPDispatcherImplBase.dispatch_b` and contains the main logic for this step.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplBase.dispatch_b`，并承载这一阶段的核心逻辑。

### Lines 472-479: Function `_MoriEPDispatcherImplBase.combine_a` and its core logic
```python
    def combine_a(
        self,
        hidden_states: torch.Tensor,
        topk_ids: torch.Tensor,
        topk_weights: torch.Tensor,
    ):
        raise NotImplementedError
```
**EN:** This block defines `_MoriEPDispatcherImplBase.combine_a` and contains the main logic for this step.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplBase.combine_a`，并承载这一阶段的核心逻辑。

### Lines 480-482: Function `_MoriEPDispatcherImplBase.combine_b` and its core logic
```python
    def combine_b(self, *args, **kwargs):
        raise NotImplementedError
```
**EN:** This block defines `_MoriEPDispatcherImplBase.combine_b` and contains the main logic for this step.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplBase.combine_b`，并承载这一阶段的核心逻辑。

### Lines 483-498: `_MoriEPDispatcherImplBase.set_quant_config` setter for quantization config
```python
    def set_quant_config(self, quant_config: dict) -> None:
        self.quant_config = quant_config
        # Auto-detect dispatch quantization from weight dtype
        weight_dtype = quant_config.get("weight_dtype", None)
        if weight_dtype in (torch.float8_e4m3fn, torch.float8_e4m3fnuz):
            self.dispatch_dtype = DispatchDtype.fp8
            self.combine_dtype = CombineDtype.bf16
        elif weight_dtype == torch.float4_e2m1fn_x2:
            self.dispatch_dtype = DispatchDtype.fp4
            self.combine_dtype = CombineDtype.fp8
        else:
            self.dispatch_dtype = DispatchDtype.bf16
            self.combine_dtype = CombineDtype.bf16
        # Apply env var override immediately so dispatch_a sees correct flags
        self._apply_dispatch_dtype_override()
```
**EN:** This block defines `_MoriEPDispatcherImplBase.set_quant_config` and contains the main logic for this step. It mainly invokes `quant_config.get` and `self._apply_dispatch_dtype_override`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.quant_config`, `weight_dtype`, `self.dispatch_dtype`, and `self.combine_dtype` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplBase.set_quant_config`，并承载这一阶段的核心逻辑。 它主要调用 `quant_config.get` 和 `self._apply_dispatch_dtype_override`，说明该流程会编排底层辅助函数或计算内核。 像 `self.quant_config`、`weight_dtype`、`self.dispatch_dtype` 以及 `self.combine_dtype` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 499-504: `_MoriEPDispatcherImplBase.set_overlap_args` setter for overlap args
```python
    def set_overlap_args(
        self, combine_overlap_args: CombineOverlapArgs, meta_overlap_args: dict
    ) -> None:
        self.overlap_args = combine_overlap_args
        self.meta_overlap_args = meta_overlap_args
```
**EN:** This block defines `_MoriEPDispatcherImplBase.set_overlap_args` and contains the main logic for this step. Intermediate names such as `self.overlap_args` and `self.meta_overlap_args` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplBase.set_overlap_args`，并承载这一阶段的核心逻辑。 像 `self.overlap_args` 和 `self.meta_overlap_args` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 505-509: Function `_MoriEPDispatcherImplBase.clear_overlap_args` and its core logic
```python
    def clear_overlap_args(self) -> None:
        self.overlap_args = None
        self.meta_overlap_args = None
```
**EN:** This block defines `_MoriEPDispatcherImplBase.clear_overlap_args` and contains the main logic for this step. Intermediate names such as `self.overlap_args` and `self.meta_overlap_args` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplBase.clear_overlap_args`，并承载这一阶段的核心逻辑。 像 `self.overlap_args` 和 `self.meta_overlap_args` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 510-510: Class `_MoriEPDispatcherImplNormal` declaration and shared state
```python
class _MoriEPDispatcherImplNormal(_MoriEPDispatcherImplBase):
```
**EN:** This block introduces class `_MoriEPDispatcherImplNormal` and the state shared by its methods. It inherits from `_MoriEPDispatcherImplBase`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `_MoriEPDispatcherImplNormal`，并定义其方法共享的状态。 它继承自 `_MoriEPDispatcherImplBase`，说明了它在 SRT 层栈中的接入方式。

### Lines 511-522: `_MoriEPDispatcherImplNormal` initialization and state setup
```python
    def __init__(self, async_finish: bool, **kwargs):
        super().__init__(**kwargs)

        self.async_finish = async_finish
        self.quant_config = {}
        self.fp8_quant_func = get_hip_quant(QuantType.per_1x128)
        self.fp4_quant_func = get_hip_quant(QuantType.per_1x32)
        self.enable_dual_stream = is_tbo_enabled()
        self._comm_stream = None
        if self.enable_dual_stream:
            self._comm_stream = CommStreamPool.get_stream_from_pool(self.group)
```
**EN:** This block defines `_MoriEPDispatcherImplNormal.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `get_hip_quant`, `is_tbo_enabled`, and `CommStreamPool.get_stream_from_pool`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.async_finish`, `self.quant_config`, `self.fp8_quant_func`, `self.fp4_quant_func`, and `self.enable_dual_stream` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplNormal.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`get_hip_quant`、`is_tbo_enabled` 以及 `CommStreamPool.get_stream_from_pool`，说明该流程会编排底层辅助函数或计算内核。 像 `self.async_finish`、`self.quant_config`、`self.fp8_quant_func`、`self.fp4_quant_func` 以及 `self.enable_dual_stream` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 523-530: Internal helper `_MoriEPDispatcherImplNormal._capture_event_if_async`
```python
    def _capture_event_if_async(self) -> Optional[torch.cuda.Event]:
        assert self.enable_dual_stream, "dual stream must be enabled"
        if not self.async_finish:
            return None
        ev = torch.cuda.Event(blocking=False, interprocess=False)
        ev.record(torch.cuda.current_stream())
        return ev
```
**EN:** This block defines `_MoriEPDispatcherImplNormal._capture_event_if_async` and contains the main logic for this step. It mainly invokes `torch.cuda.Event`, `ev.record`, and `torch.cuda.current_stream`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `ev` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplNormal._capture_event_if_async`，并承载这一阶段的核心逻辑。 它主要调用 `torch.cuda.Event`、`ev.record` 以及 `torch.cuda.current_stream`，说明该流程会编排底层辅助函数或计算内核。 像 `ev` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 531-586: `_MoriEPDispatcherImplNormal.dispatch_a` step for a
```python
    def dispatch_a(
        self,
        hidden_states: torch.Tensor,
        topk_output: TopKOutput,
    ):
        topk_weights, topk_ids = topk_output.topk_weights, topk_output.topk_ids

        num_token = hidden_states.shape[0]
        output_dtype = hidden_states.dtype
        scale = None

        if self.dispatch_dtype == DispatchDtype.fp8:
            # FP8 quant
            if num_token > 0:
                # NOTE: aiter is able to handle token=0 case in UT. But for some
                # reason it failed at e2e case. Root cause TBD.
                hidden_states, scale = self.fp8_quant_func(
                    hidden_states, quant_dtype=fp8_dtype
                )
            else:
                hidden_states = torch.empty(
                    hidden_states.shape, dtype=fp8_dtype, device=hidden_states.device
                )
                scale = torch.empty(
                    (0, self.hidden_size // FP8_BLOCK_SIZE),
                    dtype=torch.float32,
                    device=hidden_states.device,
                )

        elif self.dispatch_dtype == DispatchDtype.fp4:
            # FP4 quant
            if num_token > 0:
                hidden_states, scale = self.fp4_quant_func(hidden_states, shuffle=False)
            else:
                hidden_states = torch.empty(
                    (0, self.hidden_size // 2),
                    dtype=torch.float4_e2m1fn_x2,
                    device=hidden_states.device,
                )
                scale = torch.empty(
                    (0, self.hidden_size // MXFP4_BLOCK_SIZE),
                    dtype=torch.float8_e8m0fnu,
                    device=hidden_states.device,
                )

        previous_event = self._capture_event_if_async() if self._comm_stream else None

        return (
            hidden_states,
            topk_weights,
            topk_ids,
            scale,
            output_dtype,
            previous_event,
        )
```
**EN:** This block defines `_MoriEPDispatcherImplNormal.dispatch_a` and contains the main logic for this step. It mainly invokes `self._capture_event_if_async`, `self.fp8_quant_func`, `torch.empty`, and `self.fp4_quant_func`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `topk_weights`, `topk_ids`, `num_token`, `output_dtype`, and `scale` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplNormal.dispatch_a`，并承载这一阶段的核心逻辑。 它主要调用 `self._capture_event_if_async`、`self.fp8_quant_func`、`torch.empty` 以及 `self.fp4_quant_func`，说明该流程会编排底层辅助函数或计算内核。 像 `topk_weights`、`topk_ids`、`num_token`、`output_dtype` 以及 `scale` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 587-625: `_MoriEPDispatcherImplNormal.dispatch_b` step for b
```python
    def dispatch_b(
        self,
        hidden_states,
        topk_weights,
        topk_ids,
        scale,
        output_dtype,
        previous_event,
    ):

        (
            packed_recv_hidden,
            recv_topk_weights,
            recv_scales,
            recv_topk_ids,
            packed_recv_count,
            done_event,
        ) = self._dispatch_core(
            hidden_states,
            topk_weights,
            topk_ids,
            scale=scale,
            previous_event=previous_event,
        )

        if self._comm_stream and self.async_finish and done_event is not None:
            torch.cuda.current_stream().wait_event(done_event)

        return MoriEPNormalDispatchOutput(
            hidden_states=packed_recv_hidden,
            hidden_states_scale=recv_scales,
            topk_ids=recv_topk_ids,
            topk_weights=recv_topk_weights,
            num_recv_tokens_per_expert=packed_recv_count,
            origin_topk_ids=topk_ids,
            origin_topk_weights=topk_weights,
            out_dtype=output_dtype,
        )
```
**EN:** This block defines `_MoriEPDispatcherImplNormal.dispatch_b` and contains the main logic for this step. It mainly invokes `self._dispatch_core`, `MoriEPNormalDispatchOutput`, `torch.cuda.current_stream.wait_event`, and `torch.cuda.current_stream`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `packed_recv_hidden`, `recv_topk_weights`, `recv_scales`, `recv_topk_ids`, and `packed_recv_count` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplNormal.dispatch_b`，并承载这一阶段的核心逻辑。 它主要调用 `self._dispatch_core`、`MoriEPNormalDispatchOutput`、`torch.cuda.current_stream.wait_event` 以及 `torch.cuda.current_stream`，说明该流程会编排底层辅助函数或计算内核。 像 `packed_recv_hidden`、`recv_topk_weights`、`recv_scales`、`recv_topk_ids` 以及 `packed_recv_count` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 626-704: Internal helper `_MoriEPDispatcherImplNormal._dispatch_core`
```python
    def _dispatch_core(
        self,
        hidden_states: torch.Tensor,
        topk_weights: torch.Tensor,
        topk_ids: torch.Tensor,
        scale: Optional[torch.Tensor] = None,
        previous_event: Optional[torch.cuda.Event] = None,
    ):
        done_event: Optional[torch.cuda.Event] = None

        if self._comm_stream:
            compute_stream = torch.cuda.current_stream()
            comm_stream = self._comm_stream  # comm stream

            for t in (hidden_states, topk_weights, topk_ids):
                t.record_stream(comm_stream)
            if scale is not None:
                scale.record_stream(comm_stream)

            with torch.cuda.stream(comm_stream):
                # if (previous_event) stream_wait(comm_stream, previous_event)
                # else stream_wait(comm_stream, compute_stream)

                if previous_event is not None:
                    comm_stream.wait_event(previous_event)
                else:
                    comm_stream.wait_stream(compute_stream)

                dispatch_fn = (
                    self.mori_op.dispatch_send
                    if self.enable_sdma
                    else self.mori_op.dispatch
                )
                (
                    packed_recv_hidden,
                    recv_topk_weights,
                    recv_scales,
                    recv_topk_ids,
                    packed_recv_count,
                ) = dispatch_fn(hidden_states, topk_weights, scale, topk_ids)
                if self.enable_sdma:
                    self.mori_op.dispatch_recv()

                if self.async_finish:
                    done_event = torch.cuda.Event(blocking=False, interprocess=False)
                    done_event.record(comm_stream)
                else:
                    compute_stream.wait_stream(comm_stream)

            for t in (
                packed_recv_hidden,
                recv_topk_weights,
                recv_scales,
                recv_topk_ids,
            ):
                if t is not None:
                    t.record_stream(comm_stream)
        else:

            (
                packed_recv_hidden,
                recv_topk_weights,
                recv_scales,
                recv_topk_ids,
                packed_recv_count,
            ) = self.mori_op.dispatch(hidden_states, topk_weights, scale, topk_ids)

        # TODO(billishyahao): EPLB
        # get_global_expert_distribution_recorder().on_deepep_dispatch_normal(

        return (
            packed_recv_hidden,
            recv_topk_weights,
            recv_scales,
            recv_topk_ids,
            packed_recv_count,
            done_event,
        )
```
**EN:** This block defines `_MoriEPDispatcherImplNormal._dispatch_core` and contains the main logic for this step. It mainly invokes `torch.cuda.current_stream`, `self.mori_op.dispatch`, `t.record_stream`, `scale.record_stream`, and `torch.cuda.stream`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `done_event`, `compute_stream`, `comm_stream`, `packed_recv_hidden`, and `recv_topk_weights` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplNormal._dispatch_core`，并承载这一阶段的核心逻辑。 它主要调用 `torch.cuda.current_stream`、`self.mori_op.dispatch`、`t.record_stream`、`scale.record_stream` 以及 `torch.cuda.stream`，说明该流程会编排底层辅助函数或计算内核。 像 `done_event`、`compute_stream`、`comm_stream`、`packed_recv_hidden` 以及 `recv_topk_weights` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 705-713: Function `_MoriEPDispatcherImplNormal.combine_a` and its core logic
```python
    def combine_a(
        self,
        hidden_states: torch.Tensor,
        topk_ids: torch.Tensor,
        topk_weights: torch.Tensor,
    ):
        previous_event = self._capture_event_if_async() if self._comm_stream else None
        return hidden_states, topk_ids, topk_weights, previous_event
```
**EN:** This block defines `_MoriEPDispatcherImplNormal.combine_a` and contains the main logic for this step. It mainly invokes `self._capture_event_if_async`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `previous_event` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplNormal.combine_a`，并承载这一阶段的核心逻辑。 它主要调用 `self._capture_event_if_async`，说明该流程会编排底层辅助函数或计算内核。 像 `previous_event` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 714-724: Function `_MoriEPDispatcherImplNormal.combine_b` and its core logic
```python
    def combine_b(self, hidden_states, topk_ids, topk_weights, previous_event):

        hidden_states, done_event = self._combine_core(
            hidden_states, topk_ids, topk_weights, previous_event
        )

        if self._comm_stream and self.async_finish and done_event is not None:
            torch.cuda.current_stream().wait_event(done_event)

        return hidden_states
```
**EN:** This block defines `_MoriEPDispatcherImplNormal.combine_b` and contains the main logic for this step. It mainly invokes `self._combine_core`, `torch.cuda.current_stream.wait_event`, and `torch.cuda.current_stream`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states` and `done_event` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplNormal.combine_b`，并承载这一阶段的核心逻辑。 它主要调用 `self._combine_core`、`torch.cuda.current_stream.wait_event` 以及 `torch.cuda.current_stream`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states` 和 `done_event` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 725-770: Internal helper `_MoriEPDispatcherImplNormal._combine_core`
```python
    def _combine_core(
        self,
        hidden_states: torch.Tensor,
        topk_ids: torch.Tensor,
        topk_weights: torch.Tensor,
        previous_event: Optional[torch.cuda.Event],
    ):
        done_event: Optional[torch.cuda.Event] = None

        if self._comm_stream:
            compute_stream = torch.cuda.current_stream()
            comm_stream = self._comm_stream

            for t in (hidden_states, topk_ids, topk_weights):
                t.record_stream(comm_stream)

            with torch.cuda.stream(comm_stream):
                if previous_event is not None:
                    comm_stream.wait_event(previous_event)
                else:
                    comm_stream.wait_stream(compute_stream)

                combine_fn = (
                    self.mori_op.combine_send
                    if self.enable_sdma
                    else self.mori_op.combine
                )
                combined_hidden_states = combine_fn(hidden_states, None, topk_ids)[0]
                if self.enable_sdma:
                    self.mori_op.combine_recv()

                if self.async_finish:
                    done_event = torch.cuda.Event(blocking=False, interprocess=False)
                    done_event.record(comm_stream)
                else:
                    compute_stream.wait_stream(comm_stream)

            combined_hidden_states.record_stream(comm_stream)

        else:
            combined_hidden_states = self.mori_op.combine(
                hidden_states, None, topk_ids
            )[0]

        return combined_hidden_states, done_event
```
**EN:** This block defines `_MoriEPDispatcherImplNormal._combine_core` and contains the main logic for this step. It mainly invokes `torch.cuda.current_stream`, `combined_hidden_states.record_stream`, `t.record_stream`, `torch.cuda.stream`, and `self.mori_op.combine`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `done_event`, `compute_stream`, `comm_stream`, `combined_hidden_states`, and `combine_fn` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplNormal._combine_core`，并承载这一阶段的核心逻辑。 它主要调用 `torch.cuda.current_stream`、`combined_hidden_states.record_stream`、`t.record_stream`、`torch.cuda.stream` 以及 `self.mori_op.combine`，说明该流程会编排底层辅助函数或计算内核。 像 `done_event`、`compute_stream`、`comm_stream`、`combined_hidden_states` 以及 `combine_fn` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 771-774: `_MoriEPDispatcherImplNormal.set_quant_config` setter for quantization config
```python
    def set_quant_config(self, quant_config: dict):
        super().set_quant_config(quant_config)
```
**EN:** This block defines `_MoriEPDispatcherImplNormal.set_quant_config` and contains the main logic for this step. It mainly invokes `super.set_quant_config`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplNormal.set_quant_config`，并承载这一阶段的核心逻辑。 它主要调用 `super.set_quant_config`，说明该流程会编排底层辅助函数或计算内核。

### Lines 775-775: Class `_MoriEPDispatcherImplLowLatency` declaration and shared state
```python
class _MoriEPDispatcherImplLowLatency(_MoriEPDispatcherImplBase):
```
**EN:** This block introduces class `_MoriEPDispatcherImplLowLatency` and the state shared by its methods. It inherits from `_MoriEPDispatcherImplBase`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `_MoriEPDispatcherImplLowLatency`，并定义其方法共享的状态。 它继承自 `_MoriEPDispatcherImplBase`，说明了它在 SRT 层栈中的接入方式。

### Lines 776-781: `_MoriEPDispatcherImplLowLatency` initialization and state setup
```python
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.quant_config = {}
        self.fp8_quant_func = get_hip_quant(QuantType.per_1x128)
        self.fp4_quant_func = get_hip_quant(QuantType.per_1x32)
```
**EN:** This block defines `_MoriEPDispatcherImplLowLatency.__init__` and contains the main logic for this step. It mainly invokes `super.__init__` and `get_hip_quant`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.quant_config`, `self.fp8_quant_func`, and `self.fp4_quant_func` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplLowLatency.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__` 和 `get_hip_quant`，说明该流程会编排底层辅助函数或计算内核。 像 `self.quant_config`、`self.fp8_quant_func` 以及 `self.fp4_quant_func` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 782-852: `_MoriEPDispatcherImplLowLatency.dispatch_a` step for a
```python
    def dispatch_a(
        self,
        hidden_states: torch.Tensor,
        topk_output: TopKOutput,
    ):
        import mori

        assert (
            self.mori_op.config.kernel_type
            is mori.ops.EpDispatchCombineKernelType.AsyncLL
        ), "mori asyncll mismatch"

        num_tokens = hidden_states.shape[0]
        output_dtype = hidden_states.dtype
        scale = None

        if self.dispatch_dtype == DispatchDtype.fp8:
            # FP8 quant
            if num_tokens > 0:
                # NOTE: aiter is able to handle token=0 case in UT. But for some
                # reason it failed at e2e case. Root cause TBD.
                hidden_states, scale = self.fp8_quant_func(
                    hidden_states, quant_dtype=fp8_dtype
                )
            else:
                hidden_states = torch.empty(
                    hidden_states.shape, dtype=fp8_dtype, device=hidden_states.device
                )
                scale = torch.empty(
                    (0, self.hidden_size // FP8_BLOCK_SIZE),
                    dtype=torch.float32,
                    device=hidden_states.device,
                )

        elif self.dispatch_dtype == DispatchDtype.fp4:
            # FP4 quant
            if num_tokens > 0:
                hidden_states, scale = self.fp4_quant_func(hidden_states, shuffle=False)
            else:
                hidden_states = torch.empty(
                    (0, self.hidden_size // 2),
                    dtype=torch.float4_e2m1fn_x2,
                    device=hidden_states.device,
                )
                scale = torch.empty(
                    (0, self.hidden_size // MXFP4_BLOCK_SIZE),
                    dtype=torch.float8_e8m0fnu,
                    device=hidden_states.device,
                )

        topk_weights, topk_ids = topk_output.topk_weights, topk_output.topk_ids

        (
            packed_recv_hidden,
            recv_topk_weights,
            recv_scales,
            recv_topk_ids,
            packed_recv_count,
        ) = self._dispatch_core(hidden_states, topk_weights, topk_ids, scale=scale)

        return (
            packed_recv_hidden,
            recv_topk_weights,
            recv_topk_ids,
            recv_scales,
            packed_recv_count,
            topk_weights,
            topk_ids,
            output_dtype,
        )
```
**EN:** This block defines `_MoriEPDispatcherImplLowLatency.dispatch_a` and contains the main logic for this step. It mainly invokes `self._dispatch_core`, `self.fp8_quant_func`, `torch.empty`, and `self.fp4_quant_func`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_tokens`, `output_dtype`, `scale`, `topk_weights`, and `topk_ids` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplLowLatency.dispatch_a`，并承载这一阶段的核心逻辑。 它主要调用 `self._dispatch_core`、`self.fp8_quant_func`、`torch.empty` 以及 `self.fp4_quant_func`，说明该流程会编排底层辅助函数或计算内核。 像 `num_tokens`、`output_dtype`、`scale`、`topk_weights` 以及 `topk_ids` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 853-885: `_MoriEPDispatcherImplLowLatency.dispatch_b` step for b
```python
    def dispatch_b(
        self,
        hidden_states,
        recv_topk_weights,
        recv_topk_ids,
        recv_scales,
        packed_recv_count,
        topk_weights,
        topk_ids,
        output_dtype,
    ):

        ##TODO(billishyahao): add assertion here to check async
        import mori

        assert (
            self.mori_op.config.kernel_type
            is mori.ops.EpDispatchCombineKernelType.AsyncLL
        ), "mori asyncll mismatch"

        self.mori_op.dispatch_recv()

        return MoriEPLLDispatchOutput(
            hidden_states=hidden_states,
            hidden_states_scale=recv_scales,
            topk_ids=recv_topk_ids,
            topk_weights=recv_topk_weights,
            num_recv_tokens_per_expert=packed_recv_count,
            origin_topk_ids=topk_ids,
            origin_topk_weights=topk_weights,
            out_dtype=output_dtype,
        )
```
**EN:** This block defines `_MoriEPDispatcherImplLowLatency.dispatch_b` and contains the main logic for this step. It mainly invokes `self.mori_op.dispatch_recv` and `MoriEPLLDispatchOutput`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplLowLatency.dispatch_b`，并承载这一阶段的核心逻辑。 它主要调用 `self.mori_op.dispatch_recv` 和 `MoriEPLLDispatchOutput`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 886-910: Internal helper `_MoriEPDispatcherImplLowLatency._dispatch_core`
```python
    def _dispatch_core(
        self,
        hidden_states: torch.Tensor,
        topk_weights: torch.Tensor,
        topk_ids: torch.Tensor,
        scale: Optional[torch.Tensor] = None,
    ):
        ##TODO(billishyahao): add assertion here to check async

        (
            packed_recv_hidden,
            recv_topk_weights,
            recv_scales,
            recv_topk_ids,
            packed_recv_count,
        ) = self.mori_op.dispatch_send(hidden_states, topk_weights, scale, topk_ids)

        return (
            packed_recv_hidden,
            recv_topk_weights,
            recv_scales,
            recv_topk_ids,
            packed_recv_count,
        )
```
**EN:** This block defines `_MoriEPDispatcherImplLowLatency._dispatch_core` and contains the main logic for this step. It mainly invokes `self.mori_op.dispatch_send`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `packed_recv_hidden`, `recv_topk_weights`, `recv_scales`, `recv_topk_ids`, and `packed_recv_count` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplLowLatency._dispatch_core`，并承载这一阶段的核心逻辑。 它主要调用 `self.mori_op.dispatch_send`，说明该流程会编排底层辅助函数或计算内核。 像 `packed_recv_hidden`、`recv_topk_weights`、`recv_scales`、`recv_topk_ids` 以及 `packed_recv_count` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 911-925: Function `_MoriEPDispatcherImplLowLatency.combine_a` and its core logic
```python
    def combine_a(
        self,
        hidden_states: torch.Tensor,
        topk_ids: torch.Tensor,
        topk_weights: torch.Tensor,
        overlap_args: Optional[CombineOverlapArgs] = None,
    ):
        hidden_states = self._combine_core(
            hidden_states,
            topk_ids,
            topk_weights,
            overlap_args=overlap_args,
        )
        return hidden_states, topk_ids, topk_weights, overlap_args
```
**EN:** This block defines `_MoriEPDispatcherImplLowLatency.combine_a` and contains the main logic for this step. It mainly invokes `self._combine_core`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplLowLatency.combine_a`，并承载这一阶段的核心逻辑。 它主要调用 `self._combine_core`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 926-931: Function `_MoriEPDispatcherImplLowLatency.combine_b` and its core logic
```python
    def combine_b(self, hidden_states, topk_ids, topk_weights, previous_event):

        self.mori_op.combine_recv()

        return hidden_states[0]
```
**EN:** This block defines `_MoriEPDispatcherImplLowLatency.combine_b` and contains the main logic for this step. It mainly invokes `self.mori_op.combine_recv`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplLowLatency.combine_b`，并承载这一阶段的核心逻辑。 它主要调用 `self.mori_op.combine_recv`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 932-944: Internal helper `_MoriEPDispatcherImplLowLatency._combine_core`
```python
    def _combine_core(
        self,
        hidden_states: torch.Tensor,
        topk_ids: torch.Tensor,
        topk_weights: torch.Tensor,
        overlap_args: Optional[CombineOverlapArgs] = None,
    ):
        combined_hidden_states = self.mori_op.combine_send(
            hidden_states, None, topk_ids
        )

        return combined_hidden_states
```
**EN:** This block defines `_MoriEPDispatcherImplLowLatency._combine_core` and contains the main logic for this step. It mainly invokes `self.mori_op.combine_send`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `combined_hidden_states` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplLowLatency._combine_core`，并承载这一阶段的核心逻辑。 它主要调用 `self.mori_op.combine_send`，说明该流程会编排底层辅助函数或计算内核。 像 `combined_hidden_states` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 945-948: `_MoriEPDispatcherImplLowLatency.set_quant_config` setter for quantization config
```python
    def set_quant_config(self, quant_config: dict):
        super().set_quant_config(quant_config)
```
**EN:** This block defines `_MoriEPDispatcherImplLowLatency.set_quant_config` and contains the main logic for this step. It mainly invokes `super.set_quant_config`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `_MoriEPDispatcherImplLowLatency.set_quant_config`，并承载这一阶段的核心逻辑。 它主要调用 `super.set_quant_config`，说明该流程会编排底层辅助函数或计算内核。

### Lines 949-956: Class `_Stage` declaration and shared state
```python
@dataclass
class _Stage(Enum):
    INITIAL = auto()
    AFTER_DISPATCH_A = auto()
    AFTER_DISPATCH_B = auto()
    AFTER_COMBINE_A = auto()
```
**EN:** This block introduces class `_Stage` and the state shared by its methods. It inherits from `Enum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `_Stage`，并定义其方法共享的状态。 它继承自 `Enum`，说明了它在 SRT 层栈中的接入方式。

### Lines 957-957: Class `MoriEPDispatcher` declaration and shared state
```python
class MoriEPDispatcher(BaseDispatcher):
```
**EN:** This block introduces class `MoriEPDispatcher` and the state shared by its methods. It inherits from `BaseDispatcher`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `MoriEPDispatcher`，并定义其方法共享的状态。 它继承自 `BaseDispatcher`，说明了它在 SRT 层栈中的接入方式。

### Lines 958-1027: `MoriEPDispatcher` initialization and state setup
```python
    def __init__(
        self,
        group: torch.distributed.ProcessGroup,
        router_topk: int,
        permute_fusion: bool = False,
        num_experts: int = None,
        num_local_experts: int = None,
        hidden_size: int = None,
        params_dtype: torch.dtype = None,
        deepep_mode: DeepEPMode = DeepEPMode.AUTO,
        async_finish: bool = False,
        return_recv_hook: bool = False,
        instance_id: int = 0,
    ):
        super().__init__()

        self.deepep_mode = deepep_mode

        async_mode = self.deepep_mode.enable_low_latency()
        if get_bool_env_var("SGLANG_ROCM_USE_MULTI_STREAM") and not async_mode:
            logger.warning_once(
                "SGLANG_ROCM_USE_MULTI_STREAM=1 is set but Mori AsyncLL is "
                "not enabled (--deepep-mode=%s). The alt-stream overlap only "
                "frees up CUs when dispatch/combine runs on the AsyncLL "
                "copy-engine kernel; otherwise it stays on CUs and competes "
                "with the alt-stream work. Pass --deepep-mode low_latency "
                "(or auto) to enable the AsyncLL kernel.",
                self.deepep_mode.value,
            )

        common_kwargs = dict(
            group=group,
            router_topk=router_topk,
            permute_fusion=permute_fusion,
            num_experts=num_experts,
            num_local_experts=num_local_experts,
            hidden_size=hidden_size,
            params_dtype=params_dtype,
            deepep_mode=deepep_mode,
            instance_id=instance_id,
        )

        if self.deepep_mode.enable_low_latency():
            self._low_latency_dispatcher = _MoriEPDispatcherImplLowLatency(
                **common_kwargs,
            )

        if self.deepep_mode.enable_normal():
            self._normal_dispatcher = _MoriEPDispatcherImplNormal(
                async_finish=async_finish,
                **common_kwargs,
            )

        self._stage = _Stage.INITIAL
        self._deepep_dispatch_hooks = MoriEPPDispatchHooks()

        # Mori dispatch produces global topk_ids in [0, num_experts); mask out
        # experts that are not local to this rank.
        self.expert_mask_gpu = None
        if _use_aiter and num_experts is not None and num_local_experts is not None:
            ep_rank = get_moe_expert_parallel_rank()
            expert_mask = torch.zeros(
                num_experts,
                device=torch.cuda.current_device(),
                dtype=torch.int32,
            )
            start = ep_rank * num_local_experts
            expert_mask[start : start + num_local_experts] = 1
            self.expert_mask_gpu = expert_mask
```
**EN:** This block defines `MoriEPDispatcher.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `self.deepep_mode.enable_low_latency`, `dict`, `self.deepep_mode.enable_normal`, and `MoriEPPDispatchHooks`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.deepep_mode`, `async_mode`, `common_kwargs`, `self._stage`, and `self._deepep_dispatch_hooks` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MoriEPDispatcher.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`self.deepep_mode.enable_low_latency`、`dict`、`self.deepep_mode.enable_normal` 以及 `MoriEPPDispatchHooks`，说明该流程会编排底层辅助函数或计算内核。 像 `self.deepep_mode`、`async_mode`、`common_kwargs`、`self._stage` 以及 `self._deepep_dispatch_hooks` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1028-1039: Function `MoriEPDispatcher.dispatch` and its core logic
```python
    def dispatch(
        self,
        hidden_states: torch.Tensor,
        topk_output: TopKOutput,
    ) -> DispatchOutput:
        self._num_tokens = hidden_states.shape[0]
        self.dispatch_a(hidden_states, topk_output)
        if self._deepep_dispatch_hooks is not None:
            self._deepep_dispatch_hooks(self)
        ret = self.dispatch_b()
        return ret
```
**EN:** This block defines `MoriEPDispatcher.dispatch` and contains the main logic for this step. It mainly invokes `self.dispatch_a`, `self.dispatch_b`, and `self._deepep_dispatch_hooks`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self._num_tokens` and `ret` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoriEPDispatcher.dispatch`，并承载这一阶段的核心逻辑。 它主要调用 `self.dispatch_a`、`self.dispatch_b` 以及 `self._deepep_dispatch_hooks`，说明该流程会编排底层辅助函数或计算内核。 像 `self._num_tokens` 和 `ret` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1040-1051: `MoriEPDispatcher.dispatch_a` step for a
```python
    def dispatch_a(
        self,
        hidden_states: torch.Tensor,
        topk_output: TopKOutput,
    ):
        self._update_stage(_Stage.INITIAL, _Stage.AFTER_DISPATCH_A)
        inner_state = self._get_impl().dispatch_a(
            hidden_states=hidden_states,
            topk_output=topk_output,
        )
        self._dispatch_intermediate_state = inner_state
```
**EN:** This block defines `MoriEPDispatcher.dispatch_a` and contains the main logic for this step. It mainly invokes `self._update_stage`, `self._get_impl.dispatch_a`, and `self._get_impl`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inner_state` and `self._dispatch_intermediate_state` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MoriEPDispatcher.dispatch_a`，并承载这一阶段的核心逻辑。 它主要调用 `self._update_stage`、`self._get_impl.dispatch_a` 以及 `self._get_impl`，说明该流程会编排底层辅助函数或计算内核。 像 `inner_state` 和 `self._dispatch_intermediate_state` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1052-1057: `MoriEPDispatcher.dispatch_b` step for b
```python
    def dispatch_b(self):
        self._update_stage(_Stage.AFTER_DISPATCH_A, _Stage.AFTER_DISPATCH_B)
        inner_state = self._dispatch_intermediate_state
        del self._dispatch_intermediate_state
        return self._get_impl().dispatch_b(*inner_state)
```
**EN:** This block defines `MoriEPDispatcher.dispatch_b` and contains the main logic for this step. It mainly invokes `self._update_stage`, `self._get_impl.dispatch_b`, and `self._get_impl`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inner_state` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoriEPDispatcher.dispatch_b`，并承载这一阶段的核心逻辑。 它主要调用 `self._update_stage`、`self._get_impl.dispatch_b` 以及 `self._get_impl`，说明该流程会编排底层辅助函数或计算内核。 像 `inner_state` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1058-1065: Function `MoriEPDispatcher.combine` and its core logic
```python
    def combine(
        self,
        combine_input: CombineInput,
    ) -> Tuple:
        self.combine_a(combine_input)
        hidden_states = self.combine_b()
        return hidden_states[: self._num_tokens]
```
**EN:** This block defines `MoriEPDispatcher.combine` and contains the main logic for this step. It mainly invokes `self.combine_a` and `self.combine_b`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoriEPDispatcher.combine`，并承载这一阶段的核心逻辑。 它主要调用 `self.combine_a` 和 `self.combine_b`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1066-1078: Function `MoriEPDispatcher.combine_a` and its core logic
```python
    def combine_a(
        self,
        combine_input: CombineInput,
    ):
        hidden_states, topk_ids, topk_weights = combine_input
        self._update_stage(_Stage.AFTER_DISPATCH_B, _Stage.AFTER_COMBINE_A)
        inner_state = self._get_impl().combine_a(
            hidden_states=hidden_states,
            topk_ids=topk_ids,
            topk_weights=topk_weights,
        )
        self._combine_intermediate_state = inner_state
```
**EN:** This block defines `MoriEPDispatcher.combine_a` and contains the main logic for this step. It mainly invokes `self._update_stage`, `self._get_impl.combine_a`, and `self._get_impl`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `topk_ids`, `topk_weights`, `inner_state`, and `self._combine_intermediate_state` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MoriEPDispatcher.combine_a`，并承载这一阶段的核心逻辑。 它主要调用 `self._update_stage`、`self._get_impl.combine_a` 以及 `self._get_impl`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`topk_ids`、`topk_weights`、`inner_state` 以及 `self._combine_intermediate_state` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1079-1084: Function `MoriEPDispatcher.combine_b` and its core logic
```python
    def combine_b(self):
        self._update_stage(_Stage.AFTER_COMBINE_A, _Stage.INITIAL)
        inner_state = self._combine_intermediate_state
        del self._combine_intermediate_state
        return self._get_impl().combine_b(*inner_state)
```
**EN:** This block defines `MoriEPDispatcher.combine_b` and contains the main logic for this step. It mainly invokes `self._update_stage`, `self._get_impl.combine_b`, and `self._get_impl`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inner_state` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoriEPDispatcher.combine_b`，并承载这一阶段的核心逻辑。 它主要调用 `self._update_stage`、`self._get_impl.combine_b` 以及 `self._get_impl`，说明该流程会编排底层辅助函数或计算内核。 像 `inner_state` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1085-1094: Internal helper `MoriEPDispatcher._get_impl`
```python
    def _get_impl(self) -> _MoriEPDispatcherImplBase:
        is_extend_in_batch = get_is_extend_in_batch()
        resolved_deepep_mode = self.deepep_mode.resolve(is_extend_in_batch)
        if resolved_deepep_mode == DeepEPMode.NORMAL:
            return self._normal_dispatcher
        elif resolved_deepep_mode == DeepEPMode.LOW_LATENCY:
            return self._low_latency_dispatcher
        else:
            raise ValueError(f"Invalid deepep_mode: {self.deepep_mode}")
```
**EN:** This block defines `MoriEPDispatcher._get_impl` and contains the main logic for this step. It mainly invokes `get_is_extend_in_batch`, `self.deepep_mode.resolve`, and `ValueError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `is_extend_in_batch` and `resolved_deepep_mode` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoriEPDispatcher._get_impl`，并承载这一阶段的核心逻辑。 它主要调用 `get_is_extend_in_batch`、`self.deepep_mode.resolve` 以及 `ValueError`，说明该流程会编排底层辅助函数或计算内核。 像 `is_extend_in_batch` 和 `resolved_deepep_mode` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1095-1098: Internal helper `MoriEPDispatcher._update_stage`
```python
    def _update_stage(self, old_stage, new_stage):
        assert self._stage == old_stage
        self._stage = new_stage
```
**EN:** This block defines `MoriEPDispatcher._update_stage` and contains the main logic for this step. Intermediate names such as `self._stage` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MoriEPDispatcher._update_stage`，并承载这一阶段的核心逻辑。 像 `self._stage` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1099-1105: `MoriEPDispatcher.set_quant_config` setter for quantization config
```python
    def set_quant_config(self, quant_config: dict):
        super().set_quant_config(quant_config)
        if self.deepep_mode.enable_low_latency():
            self._low_latency_dispatcher.set_quant_config(quant_config)
        if self.deepep_mode.enable_normal():
            self._normal_dispatcher.set_quant_config(quant_config)
```
**EN:** This block defines `MoriEPDispatcher.set_quant_config` and contains the main logic for this step. It mainly invokes `super.set_quant_config`, `self.deepep_mode.enable_low_latency`, `self.deepep_mode.enable_normal`, `self._low_latency_dispatcher.set_quant_config`, and `self._normal_dispatcher.set_quant_config`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `MoriEPDispatcher.set_quant_config`，并承载这一阶段的核心逻辑。 它主要调用 `super.set_quant_config`、`self.deepep_mode.enable_low_latency`、`self.deepep_mode.enable_normal`、`self._low_latency_dispatcher.set_quant_config` 以及 `self._normal_dispatcher.set_quant_config`，说明该流程会编排底层辅助函数或计算内核。

### Lines 1106-1118: `MoriEPDispatcher.set_overlap_args` setter for overlap args
```python
    def set_overlap_args(
        self, combine_overlap_args: CombineOverlapArgs, meta_overlap_args: dict
    ):
        super().set_overlap_args(combine_overlap_args, meta_overlap_args)
        if self.deepep_mode.enable_low_latency():
            self._low_latency_dispatcher.set_overlap_args(
                combine_overlap_args, meta_overlap_args
            )
        if self.deepep_mode.enable_normal():
            self._normal_dispatcher.set_overlap_args(
                combine_overlap_args, meta_overlap_args
            )
```
**EN:** This block defines `MoriEPDispatcher.set_overlap_args` and contains the main logic for this step. It mainly invokes `super.set_overlap_args`, `self.deepep_mode.enable_low_latency`, `self.deepep_mode.enable_normal`, `self._low_latency_dispatcher.set_overlap_args`, and `self._normal_dispatcher.set_overlap_args`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `MoriEPDispatcher.set_overlap_args`，并承载这一阶段的核心逻辑。 它主要调用 `super.set_overlap_args`、`self.deepep_mode.enable_low_latency`、`self.deepep_mode.enable_normal`、`self._low_latency_dispatcher.set_overlap_args` 以及 `self._normal_dispatcher.set_overlap_args`，说明该流程会编排底层辅助函数或计算内核。

### Lines 1119-1125: Function `MoriEPDispatcher.clear_overlap_args` and its core logic
```python
    def clear_overlap_args(self):
        super().clear_overlap_args()
        if self.deepep_mode.enable_low_latency():
            self._low_latency_dispatcher.clear_overlap_args()
        if self.deepep_mode.enable_normal():
            self._normal_dispatcher.clear_overlap_args()
```
**EN:** This block defines `MoriEPDispatcher.clear_overlap_args` and contains the main logic for this step. It mainly invokes `super.clear_overlap_args`, `self.deepep_mode.enable_low_latency`, `self.deepep_mode.enable_normal`, `self._low_latency_dispatcher.clear_overlap_args`, and `self._normal_dispatcher.clear_overlap_args`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `MoriEPDispatcher.clear_overlap_args`，并承载这一阶段的核心逻辑。 它主要调用 `super.clear_overlap_args`、`self.deepep_mode.enable_low_latency`、`self.deepep_mode.enable_normal`、`self._low_latency_dispatcher.clear_overlap_args` 以及 `self._normal_dispatcher.clear_overlap_args`，说明该流程会编排底层辅助函数或计算内核。

### Lines 1126-1127: Function `MoriEPDispatcher.register_deepep_dispatch_hook` and its core logic
```python
    def register_deepep_dispatch_hook(self, hook):
        return self._deepep_dispatch_hooks.register_hook(hook)
```
**EN:** This block defines `MoriEPDispatcher.register_deepep_dispatch_hook` and contains the main logic for this step. It mainly invokes `self._deepep_dispatch_hooks.register_hook`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoriEPDispatcher.register_deepep_dispatch_hook`，并承载这一阶段的核心逻辑。 它主要调用 `self._deepep_dispatch_hooks.register_hook`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `MoriEPPDispatchHooks`, `MoriEPNormalDispatchOutput`, `MoriEPLLDispatchOutput`, `MoriEPNormalCombineInput`, and `MoriEPLLCombineInput`. / **主要符号**：核心入口包括 `MoriEPPDispatchHooks`、`MoriEPNormalDispatchOutput`、`MoriEPLLDispatchOutput`、`MoriEPNormalCombineInput` 以及 `MoriEPLLCombineInput`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `logging`, `os`, `dataclasses.dataclass`, `typing.TYPE_CHECKING`, `typing.List`, `typing.NamedTuple`, `typing.Optional`, `typing.Tuple`, and `enum.Enum` / **标准库**：`__future__.annotations`、`logging`、`os`、`dataclasses.dataclass`、`typing.TYPE_CHECKING`、`typing.List`、`typing.NamedTuple`、`typing.Optional`、`typing.Tuple` 以及 `enum.Enum`
- **Third-party**: `torch`, `mori`, `aiter.QuantType`, and `aiter.get_hip_quant` / **第三方依赖**：`torch`、`mori`、`aiter.QuantType` 以及 `aiter.get_hip_quant`
- **Internal SGLang modules**: `sglang.srt.layers.dp_attention.get_is_extend_in_batch`, `sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcher`, `sglang.srt.layers.moe.token_dispatcher.base.CombineInput`, `sglang.srt.layers.moe.token_dispatcher.base.CombineInputFormat`, `sglang.srt.layers.moe.token_dispatcher.base.DispatchOutput`, `sglang.srt.layers.moe.token_dispatcher.base.DispatchOutputFormat`, `sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPPDispatchHooks`, `sglang.srt.layers.moe.topk.TopKOutput`, `sglang.srt.layers.moe.utils.DeepEPMode`, `sglang.srt.layers.moe.utils.is_tbo_enabled`, `sglang.srt.utils.get_bool_env_var`, and `sglang.srt.utils.get_int_env_var` / **SGLang 内部模块**：`sglang.srt.layers.dp_attention.get_is_extend_in_batch`、`sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcher`、`sglang.srt.layers.moe.token_dispatcher.base.CombineInput`、`sglang.srt.layers.moe.token_dispatcher.base.CombineInputFormat`、`sglang.srt.layers.moe.token_dispatcher.base.DispatchOutput`、`sglang.srt.layers.moe.token_dispatcher.base.DispatchOutputFormat`、`sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPPDispatchHooks`、`sglang.srt.layers.moe.topk.TopKOutput`、`sglang.srt.layers.moe.utils.DeepEPMode`、`sglang.srt.layers.moe.utils.is_tbo_enabled`、`sglang.srt.utils.get_bool_env_var` 以及 `sglang.srt.utils.get_int_env_var`
