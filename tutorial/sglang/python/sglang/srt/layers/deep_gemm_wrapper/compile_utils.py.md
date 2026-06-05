# compile_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/deep_gemm_wrapper/compile_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements linear projections and GEMM backend integration for the SGLang SRT runtime. It exposes symbols such as `update_deep_gemm_config`, `DeepGemmKernelType`, `_maybe_compile_deep_gemm_one_type_all`, and `_compile_deep_gemm_one_type_all` and connects them to backend-specific paths such as `CUDA` and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了线性投影与 GEMM 后端集成。它提供了 `update_deep_gemm_config`、`DeepGemmKernelType`、`_maybe_compile_deep_gemm_one_type_all` 以及 `_compile_deep_gemm_one_type_all` 等符号，并把这些符号连接到 `CUDA` 和 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-44: Imports, conditional backend setup, and runtime guards
```python
import logging
import os
from contextlib import contextmanager, nullcontext
from enum import IntEnum, auto
from typing import Dict, List, Tuple

import torch
from tqdm import tqdm

from sglang.srt.distributed.device_communicators.pynccl_allocator import (
    disable_symmetric_memory_context,
    restore_symmetric_memory_context,
)
from sglang.srt.environ import envs
from sglang.srt.layers.deep_gemm_wrapper.configurer import ENABLE_JIT_DEEPGEMM
from sglang.srt.server_args import ServerArgs
from sglang.srt.utils import ceil_div, get_available_gpu_memory, is_musa

logger = logging.getLogger(__name__)

_is_musa = is_musa()

if ENABLE_JIT_DEEPGEMM:
    import deep_gemm


_BUILTIN_M_LIST = list(range(1, 1024 * 16 + 1))
_ENABLE_JIT_DEEPGEMM_PRECOMPILE = envs.SGLANG_JIT_DEEPGEMM_PRECOMPILE.get()
_DO_COMPILE_ALL = True
_IS_FIRST_RANK_ON_NODE = envs.SGLANG_IS_FIRST_RANK_ON_NODE.get()
_IN_PRECOMPILE_STAGE = envs.SGLANG_IN_DEEPGEMM_PRECOMPILE_STAGE.get()
_FAST_WARMUP = envs.SGLANG_JIT_DEEPGEMM_FAST_WARMUP.get()

# Force redirect deep_gemm cache_dir
os.environ["DG_JIT_CACHE_DIR"] = os.getenv(
    "SGLANG_DG_CACHE_DIR", os.path.join(os.path.expanduser("~"), ".cache", "deep_gemm")
)

# Refer to https://github.com/deepseek-ai/DeepGEMM/commit/d75b218b7b8f4a5dd5406ac87905039ead3ae42f
# NVRTC may have performance loss with some cases.
# And NVCC JIT speed is also 9x faster in the ref commit
os.environ["DG_JIT_USE_NVRTC"] = os.getenv("SGL_DG_USE_NVRTC", "0")
```
**EN:** This section prepares the module namespace. It imports `logging`, `os`, `contextlib.contextmanager`, `contextlib.nullcontext`, `enum.IntEnum`, and `enum.auto`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `logger`, `_is_musa`, `_BUILTIN_M_LIST`, `_ENABLE_JIT_DEEPGEMM_PRECOMPILE`, and `_DO_COMPILE_ALL` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `logging`、`os`、`contextlib.contextmanager`、`contextlib.nullcontext`、`enum.IntEnum` 以及 `enum.auto`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `logger`、`_is_musa`、`_BUILTIN_M_LIST`、`_ENABLE_JIT_DEEPGEMM_PRECOMPILE` 以及 `_DO_COMPILE_ALL` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 45-96: `update_deep_gemm_config` step for deep runtime integration GEMM config
```python
def update_deep_gemm_config(gpu_id: int, server_args: ServerArgs):
    global _BUILTIN_M_LIST
    global _DO_COMPILE_ALL
    global _IS_FIRST_RANK_ON_NODE

    _BUILTIN_M_LIST = []

    if _FAST_WARMUP:
        # In fast warmup mode, only compile a small set of typical Ms

        # First cover all the small bs to ensure decode performance
        _BUILTIN_M_LIST += list(range(1, 1025))

        # Then cover larger batch sizes with gradually increasing steps
        # For example, when chunekd prefill size is 16384
        # The sampled Ms would be:
        #   1024, 1026, ... 2046 (step 2)
        #   2048, 2052, ... 4092 (step 4)
        #   4096, 5004, ... 8184 (step 8)
        #   8192, 9008, ... 16384 (step 16)
        # Totally 1024 + 1024 / 2 + 2048 / 4 + 4096 / 8 + 8192 / 16 = 3072 kernels
        next_m, sample_step = 1024, 2
        max_prefill_bs = (
            min(server_args.chunked_prefill_size, 32 * 1024)
            if server_args.chunked_prefill_size >= 1
            else 16 * 1024
        )
        while next_m < max_prefill_bs:
            _BUILTIN_M_LIST += list(range(next_m, 2 * next_m, sample_step))
            next_m = next_m * 2
            sample_step = sample_step * 2
        _BUILTIN_M_LIST.append(max_prefill_bs)
        _BUILTIN_M_LIST = sorted(list(set(_BUILTIN_M_LIST)))
    else:
        # When fast warmup isn't enabled, generate m_max and compile all the covered Ms.
        m_max = 1024 * 16
        if server_args.chunked_prefill_size < 1:
            m_max = 1024 * 64
        elif server_args.chunked_prefill_size > 8192:
            m_max = server_args.chunked_prefill_size * 2
        m_max = min(1024 * 128, m_max)
        _BUILTIN_M_LIST += list(range(1, m_max + 1))

    _IS_FIRST_RANK_ON_NODE = server_args.base_gpu_id == gpu_id

    # Check if is the first rank on node.
    # Default each rank will try compile all Ms to
    # load all symbols at the launch stages.
    # Avoid loading symbols at the serving stages.
    _DO_COMPILE_ALL = _IS_FIRST_RANK_ON_NODE
```
**EN:** This block defines `update_deep_gemm_config` and contains the main logic for this step. It mainly invokes `list`, `_BUILTIN_M_LIST.append`, `sorted`, `min`, and `range`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `_BUILTIN_M_LIST`, `_IS_FIRST_RANK_ON_NODE`, `_DO_COMPILE_ALL`, `next_m`, and `sample_step` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `update_deep_gemm_config`，并承载这一阶段的核心逻辑。 它主要调用 `list`、`_BUILTIN_M_LIST.append`、`sorted`、`min` 以及 `range`，说明该流程会编排底层辅助函数或计算内核。 像 `_BUILTIN_M_LIST`、`_IS_FIRST_RANK_ON_NODE`、`_DO_COMPILE_ALL`、`next_m` 以及 `sample_step` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 97-105: Class `DeepGemmKernelType` declaration and shared state
```python
class DeepGemmKernelType(IntEnum):
    GROUPED_GEMM_NT_F8F8BF16_MASKED = auto()
    GROUPED_GEMM_NT_F8F8BF16_CONTIG = auto()
    GROUPED_GEMM_NT_BF16_MASKED = auto()
    GROUPED_GEMM_NT_BF16_CONTIG = auto()
    GEMM_NT_F8F8BF16 = auto()
    GEMM_NT_BF16BF16F32 = auto()
```
**EN:** This block introduces class `DeepGemmKernelType` and the state shared by its methods. It inherits from `IntEnum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `DeepGemmKernelType`，并定义其方法共享的状态。 它继承自 `IntEnum`，说明了它在 SRT 层栈中的接入方式。

### Lines 106-109: Module constants and shared configuration
```python
_INITIALIZATION_DICT: Dict[Tuple[DeepGemmKernelType, int, int, int], bool] = dict()


# TODO improve code
```
**EN:** This section prepares the module namespace. Shared names such as `_INITIALIZATION_DICT` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 像 `_INITIALIZATION_DICT` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 110-154: Internal helper `_maybe_compile_deep_gemm_one_type_all`
```python
def _maybe_compile_deep_gemm_one_type_all(
    kernel_type: DeepGemmKernelType,
    n: int,
    k: int,
    num_groups: int,
) -> None:
    global _INITIALIZATION_DICT
    global _BUILTIN_M_LIST

    query_key = (kernel_type, n, k, num_groups)
    if (
        _ENABLE_JIT_DEEPGEMM_PRECOMPILE
        and _DO_COMPILE_ALL
        and _INITIALIZATION_DICT.get(query_key) is None
    ):
        _INITIALIZATION_DICT[query_key] = True

        # TODO maybe improve logs
        if not _IN_PRECOMPILE_STAGE and _IS_FIRST_RANK_ON_NODE:
            logger.warning(
                "Entering DeepGEMM JIT Pre-Compile session. "
                "It may take a long time (typically 10-20 mins) "
                "if you have not run `sglang.compile_deep_gemm`. "
                "It is recommended to run `sglang.compile_deep_gemm` with same args as `sglang.launch_server`"
                " for pre-compilation to reduce the overhead if you have not run it before. "
                "For example: "
                "`python3 -m sglang.compile_deep_gemm --model deepseek-ai/DeepSeek-V3 --tp 8 --trust-remote-code`"
            )

        logger.info(
            f"Try DeepGEMM JIT Compiling for "
            f"<{kernel_type.name}> N={n}, K={k}, num_groups={num_groups} with all Ms."
            f"{' It only takes a little time (typically 1 sec) if you have run `python3 -m sglang.compile_deep_gemm`. ' if not _IN_PRECOMPILE_STAGE else ''}"
        )

        _compile_deep_gemm_one_type_all(
            kernel_type=kernel_type,
            n=n,
            k=k,
            num_groups=num_groups,
            m_list=_BUILTIN_M_LIST,
        )


# NOTE(alcanderian): get_num_sms should be change when 2-batch-overlap is introduced
```
**EN:** This block defines `_maybe_compile_deep_gemm_one_type_all` and contains the main logic for this step. It mainly invokes `logger.info`, `_compile_deep_gemm_one_type_all`, `_INITIALIZATION_DICT.get`, and `logger.warning`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `query_key` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_maybe_compile_deep_gemm_one_type_all`，并承载这一阶段的核心逻辑。 它主要调用 `logger.info`、`_compile_deep_gemm_one_type_all`、`_INITIALIZATION_DICT.get` 以及 `logger.warning`，说明该流程会编排底层辅助函数或计算内核。 像 `query_key` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 155-225: Internal helper `_compile_deep_gemm_one_type_all`
```python
def _compile_deep_gemm_one_type_all(
    kernel_type: DeepGemmKernelType,
    n: int,
    k: int,
    num_groups: int,
    m_list: List[int],
) -> None:
    # Symmetric memory allocation performs a collective operation across all the GPUs.
    # Temporary disable symmetric memory during compilation since it only runs on the first rank.
    saved_context = disable_symmetric_memory_context()
    try:
        if kernel_type == DeepGemmKernelType.GROUPED_GEMM_NT_F8F8BF16_CONTIG:
            m_alignment = deep_gemm.get_mk_alignment_for_contiguous_layout()
            m_list = sorted(list(set(m for m in m_list if m % m_alignment == 0)))
        elif kernel_type == DeepGemmKernelType.GROUPED_GEMM_NT_BF16_CONTIG:
            m_alignment = deep_gemm.get_mk_alignment_for_contiguous_layout()
            m_list = sorted(list(set(m for m in m_list if m % m_alignment == 0)))

        # Here the precompilation is only run on the first rank, so gpu_id should be 0
        memory_budget = get_available_gpu_memory(device="cuda", gpu_id=0)

        # If the memory budget is less memory requirement, we need to reduce max_m to avoid out of memory, which might further cause hanging during warmup
        max_m = max(m_list)
        required_memory = _BaseWarmupExecutor.get_memory_requirement(
            kernel_type, max_m=max_m, n=n, k=k, num_groups=num_groups
        )
        logger.info(
            f"Required memory for warmup: {required_memory}GB, Available memory: {memory_budget}GB"
        )
        if memory_budget < required_memory:
            # TODO: Maybe compute the max_m based on the memory budget
            while (
                _BaseWarmupExecutor.get_memory_requirement(
                    kernel_type, max_m=max_m, n=n, k=k, num_groups=num_groups
                )
                > memory_budget
                and max_m > 4096
            ):
                max_m = max_m // 2
            logger.warning(
                f"Available memory {memory_budget}GB is less than required memory {required_memory}GB for warmup, reducing max_m to {max_m} to avoid out of memory"
            )
            m_list = [m for m in m_list if m <= max_m]

        # Need some methods to estimate needed memory for warmup
        executor = _BaseWarmupExecutor.create(
            kernel_type, max_m=max_m, n=n, k=k, num_groups=num_groups
        )

        has_compile_mode_api = hasattr(deep_gemm, "get_compile_mode") and hasattr(
            deep_gemm, "set_compile_mode"
        )
        if has_compile_mode_api:
            old_compile_mode = deep_gemm.get_compile_mode()
            deep_gemm.set_compile_mode(1)

        # TODO can use multi thread
        for m in tqdm(m_list, desc=f"DeepGEMM warmup"):
            executor.execute(m=m)
        if has_compile_mode_api:
            deep_gemm.set_compile_mode(old_compile_mode)

        # clean up input buffers
        torch.cuda.current_stream().synchronize()
        del executor
        torch.cuda.empty_cache()
    finally:
        # Restore symmetric memory context
        restore_symmetric_memory_context(saved_context)
```
**EN:** This block defines `_compile_deep_gemm_one_type_all` and contains the main logic for this step. It mainly invokes `disable_symmetric_memory_context`, `get_available_gpu_memory`, `max`, `_BaseWarmupExecutor.get_memory_requirement`, and `logger.info`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `saved_context`, `memory_budget`, `max_m`, `required_memory`, and `executor` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_compile_deep_gemm_one_type_all`，并承载这一阶段的核心逻辑。 它主要调用 `disable_symmetric_memory_context`、`get_available_gpu_memory`、`max`、`_BaseWarmupExecutor.get_memory_requirement` 以及 `logger.info`，说明该流程会编排底层辅助函数或计算内核。 像 `saved_context`、`memory_budget`、`max_m`、`required_memory` 以及 `executor` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 226-226: Class `_BaseWarmupExecutor` declaration and shared state
```python
class _BaseWarmupExecutor:
```
**EN:** This block introduces class `_BaseWarmupExecutor` and the state shared by its methods.
**CN:** 该代码块引入类 `_BaseWarmupExecutor`，并定义其方法共享的状态。

### Lines 227-237: Function `_BaseWarmupExecutor.create` and its core logic
```python
    @staticmethod
    def create(kernel_type: DeepGemmKernelType, **kwargs):
        return {
            DeepGemmKernelType.GEMM_NT_F8F8BF16: _NormalWarmupExecutor,
            DeepGemmKernelType.GROUPED_GEMM_NT_F8F8BF16_CONTIG: _GroupedContWarmupExecutor,
            DeepGemmKernelType.GROUPED_GEMM_NT_F8F8BF16_MASKED: _GroupedMaskedWarmupExecutor,
            DeepGemmKernelType.GEMM_NT_BF16BF16F32: _BF16F32WarmupExecutor,
            DeepGemmKernelType.GROUPED_GEMM_NT_BF16_CONTIG: _BF16GroupedContWarmupExecutor,
            DeepGemmKernelType.GROUPED_GEMM_NT_BF16_MASKED: _BF16GroupedMaskedWarmupExecutor,
        }[kernel_type](**kwargs)
```
**EN:** This block defines `_BaseWarmupExecutor.create` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_BaseWarmupExecutor.create`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 238-271: `_BaseWarmupExecutor.get_memory_requirement` getter for memory requirement
```python
    @staticmethod
    def get_memory_requirement(
        kernel_type: DeepGemmKernelType, max_m: int, n: int, k: int, num_groups: int
    ) -> int:
        # Return the required memory space in GB for warmup executor
        _GB = 1 << 30
        if kernel_type == DeepGemmKernelType.GEMM_NT_F8F8BF16:
            return (max_m * k + n * k + max_m * n * 2) / _GB
        elif kernel_type == DeepGemmKernelType.GROUPED_GEMM_NT_F8F8BF16_CONTIG:
            return (max_m * k + num_groups * n * k + max_m * 4 + max_m * n * 2) / _GB
        elif kernel_type == DeepGemmKernelType.GROUPED_GEMM_NT_BF16_CONTIG:
            return (
                max_m * k * 2 + num_groups * n * k * 2 + max_m * 4 + max_m * n * 2
            ) / _GB
        elif kernel_type == DeepGemmKernelType.GROUPED_GEMM_NT_F8F8BF16_MASKED:
            return (
                num_groups * max_m * k
                + num_groups * n * k
                + num_groups * 4
                + num_groups * max_m * n * 2
            ) / _GB
        elif kernel_type == DeepGemmKernelType.GEMM_NT_BF16BF16F32:
            # bf16 lhs + bf16 rhs + fp32 out
            return (max_m * k * 2 + n * k * 2 + max_m * n * 4) / _GB
        elif kernel_type == DeepGemmKernelType.GROUPED_GEMM_NT_BF16_MASKED:
            return (
                num_groups * max_m * k * 2
                + num_groups * n * k * 2
                + num_groups * 4
                + num_groups * max_m * n * 2
            ) / _GB
        else:
            raise ValueError(f"Invalid kernel type: {kernel_type}")
```
**EN:** This block defines `_BaseWarmupExecutor.get_memory_requirement` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `ValueError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `_GB` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_BaseWarmupExecutor.get_memory_requirement`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `ValueError`，说明该流程会编排底层辅助函数或计算内核。 像 `_GB` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 272-275: Function `_BaseWarmupExecutor.execute` and its core logic
```python
    def execute(self, m):
        raise NotImplementedError
```
**EN:** This block defines `_BaseWarmupExecutor.execute` and contains the main logic for this step.
**CN:** 该代码块定义了 `_BaseWarmupExecutor.execute`，并承载这一阶段的核心逻辑。

### Lines 276-285: Internal helper `_empty_token_fp8`
```python
def _empty_token_fp8(size):
    *dims, k = size
    return (
        torch.empty(size, device="cuda", dtype=torch.float8_e4m3fn),
        torch.empty(
            (*dims, ceil_div(k, _BLOCK_SIZE)), device="cuda", dtype=torch.float32
        ),
    )
```
**EN:** This block defines `_empty_token_fp8` and contains the main logic for this step. It mainly invokes `torch.empty` and `ceil_div`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `k` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_empty_token_fp8`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty` 和 `ceil_div`，说明该流程会编排底层辅助函数或计算内核。 像 `k` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 286-297: Internal helper `_empty_block_fp8`
```python
def _empty_block_fp8(size):
    *dims, n, k = size
    return (
        torch.empty(size, device="cuda", dtype=torch.float8_e4m3fn),
        torch.empty(
            (*dims, ceil_div(n, _BLOCK_SIZE), ceil_div(k, _BLOCK_SIZE)),
            device="cuda",
            dtype=torch.float32,
        ),
    )
```
**EN:** This block defines `_empty_block_fp8` and contains the main logic for this step. It mainly invokes `torch.empty` and `ceil_div`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `n` and `k` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_empty_block_fp8`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty` 和 `ceil_div`，说明该流程会编排底层辅助函数或计算内核。 像 `n` 和 `k` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 298-300: Module constants and shared configuration
```python
_BLOCK_SIZE = 128
```
**EN:** This section prepares the module namespace. Shared names such as `_BLOCK_SIZE` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 像 `_BLOCK_SIZE` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 301-301: Class `_NormalWarmupExecutor` declaration and shared state
```python
class _NormalWarmupExecutor(_BaseWarmupExecutor):
```
**EN:** This block introduces class `_NormalWarmupExecutor` and the state shared by its methods. It inherits from `_BaseWarmupExecutor`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `_NormalWarmupExecutor`，并定义其方法共享的状态。 它继承自 `_BaseWarmupExecutor`，说明了它在 SRT 层栈中的接入方式。

### Lines 302-306: `_NormalWarmupExecutor` initialization and state setup
```python
    def __init__(self, max_m: int, n: int, k: int, num_groups: int):
        self.lhs_q, self.lhs_s = _empty_token_fp8((max_m, k))
        self.rhs_q, self.rhs_s = _empty_block_fp8((n, k))
        self.out = torch.empty((max_m, n), device="cuda", dtype=torch.bfloat16)
```
**EN:** This block defines `_NormalWarmupExecutor.__init__` and contains the main logic for this step. It mainly invokes `_empty_token_fp8`, `_empty_block_fp8`, and `torch.empty`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.lhs_q`, `self.lhs_s`, `self.rhs_q`, `self.rhs_s`, and `self.out` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_NormalWarmupExecutor.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `_empty_token_fp8`、`_empty_block_fp8` 以及 `torch.empty`，说明该流程会编排底层辅助函数或计算内核。 像 `self.lhs_q`、`self.lhs_s`、`self.rhs_q`、`self.rhs_s` 以及 `self.out` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 307-314: Function `_NormalWarmupExecutor.execute` and its core logic
```python
    def execute(self, m):
        deep_gemm.fp8_gemm_nt(
            (self.lhs_q[:m], self.lhs_s[:m]),
            (self.rhs_q, self.rhs_s),
            self.out[:m],
        )
```
**EN:** This block defines `_NormalWarmupExecutor.execute` and contains the main logic for this step. It mainly invokes `deep_gemm.fp8_gemm_nt`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `_NormalWarmupExecutor.execute`，并承载这一阶段的核心逻辑。 它主要调用 `deep_gemm.fp8_gemm_nt`，说明该流程会编排底层辅助函数或计算内核。

### Lines 315-315: Class `_GroupedContWarmupExecutor` declaration and shared state
```python
class _GroupedContWarmupExecutor(_BaseWarmupExecutor):
```
**EN:** This block introduces class `_GroupedContWarmupExecutor` and the state shared by its methods. It inherits from `_BaseWarmupExecutor`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `_GroupedContWarmupExecutor`，并定义其方法共享的状态。 它继承自 `_BaseWarmupExecutor`，说明了它在 SRT 层栈中的接入方式。

### Lines 316-321: `_GroupedContWarmupExecutor` initialization and state setup
```python
    def __init__(self, max_m: int, n: int, k: int, num_groups: int):
        self.lhs_q, self.lhs_s = _empty_token_fp8((max_m, k))
        self.rhs_q, self.rhs_s = _empty_block_fp8((num_groups, n, k))
        self.m_indices = torch.zeros((max_m,), device="cuda", dtype=torch.int32)
        self.out = torch.empty((max_m, n), device="cuda", dtype=torch.bfloat16)
```
**EN:** This block defines `_GroupedContWarmupExecutor.__init__` and contains the main logic for this step. It mainly invokes `_empty_token_fp8`, `_empty_block_fp8`, `torch.zeros`, and `torch.empty`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.lhs_q`, `self.lhs_s`, `self.rhs_q`, `self.rhs_s`, and `self.m_indices` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_GroupedContWarmupExecutor.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `_empty_token_fp8`、`_empty_block_fp8`、`torch.zeros` 以及 `torch.empty`，说明该流程会编排底层辅助函数或计算内核。 像 `self.lhs_q`、`self.lhs_s`、`self.rhs_q`、`self.rhs_s` 以及 `self.m_indices` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 322-330: Function `_GroupedContWarmupExecutor.execute` and its core logic
```python
    def execute(self, m):
        deep_gemm.m_grouped_fp8_gemm_nt_contiguous(
            (self.lhs_q[:m], self.lhs_s[:m]),
            (self.rhs_q, self.rhs_s),
            self.out[:m],
            self.m_indices[:m],
        )
```
**EN:** This block defines `_GroupedContWarmupExecutor.execute` and contains the main logic for this step. It mainly invokes `deep_gemm.m_grouped_fp8_gemm_nt_contiguous`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `_GroupedContWarmupExecutor.execute`，并承载这一阶段的核心逻辑。 它主要调用 `deep_gemm.m_grouped_fp8_gemm_nt_contiguous`，说明该流程会编排底层辅助函数或计算内核。

### Lines 331-331: Class `_BF16GroupedContWarmupExecutor` declaration and shared state
```python
class _BF16GroupedContWarmupExecutor(_BaseWarmupExecutor):
```
**EN:** This block introduces class `_BF16GroupedContWarmupExecutor` and the state shared by its methods. It inherits from `_BaseWarmupExecutor`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `_BF16GroupedContWarmupExecutor`，并定义其方法共享的状态。 它继承自 `_BaseWarmupExecutor`，说明了它在 SRT 层栈中的接入方式。

### Lines 332-337: `_BF16GroupedContWarmupExecutor` initialization and state setup
```python
    def __init__(self, max_m: int, n: int, k: int, num_groups: int):
        self.a = torch.empty((max_m, k), device="cuda", dtype=torch.bfloat16)
        self.b = torch.empty((num_groups, n, k), device="cuda", dtype=torch.bfloat16)
        self.m_indices = torch.zeros((max_m,), device="cuda", dtype=torch.int32)
        self.out = torch.empty((max_m, n), device="cuda", dtype=torch.bfloat16)
```
**EN:** This block defines `_BF16GroupedContWarmupExecutor.__init__` and contains the main logic for this step. It mainly invokes `torch.empty` and `torch.zeros`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.a`, `self.b`, `self.m_indices`, and `self.out` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_BF16GroupedContWarmupExecutor.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty` 和 `torch.zeros`，说明该流程会编排底层辅助函数或计算内核。 像 `self.a`、`self.b`、`self.m_indices` 以及 `self.out` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 338-346: Function `_BF16GroupedContWarmupExecutor.execute` and its core logic
```python
    def execute(self, m):
        deep_gemm.m_grouped_bf16_gemm_nt_contiguous(
            self.a[:m],
            self.b,
            self.out[:m],
            m_indices=self.m_indices[:m],
        )
```
**EN:** This block defines `_BF16GroupedContWarmupExecutor.execute` and contains the main logic for this step. It mainly invokes `deep_gemm.m_grouped_bf16_gemm_nt_contiguous`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `_BF16GroupedContWarmupExecutor.execute`，并承载这一阶段的核心逻辑。 它主要调用 `deep_gemm.m_grouped_bf16_gemm_nt_contiguous`，说明该流程会编排底层辅助函数或计算内核。

### Lines 347-347: Class `_GroupedMaskedWarmupExecutor` declaration and shared state
```python
class _GroupedMaskedWarmupExecutor(_BaseWarmupExecutor):
```
**EN:** This block introduces class `_GroupedMaskedWarmupExecutor` and the state shared by its methods. It inherits from `_BaseWarmupExecutor`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `_GroupedMaskedWarmupExecutor`，并定义其方法共享的状态。 它继承自 `_BaseWarmupExecutor`，说明了它在 SRT 层栈中的接入方式。

### Lines 348-355: `_GroupedMaskedWarmupExecutor` initialization and state setup
```python
    def __init__(self, max_m: int, n: int, k: int, num_groups: int):
        self.lhs_q, self.lhs_s = _empty_token_fp8((num_groups, max_m, k))
        self.rhs_q, self.rhs_s = _empty_block_fp8((num_groups, n, k))
        self.masked_m = torch.zeros((num_groups,), device="cuda", dtype=torch.int32)
        self.out = torch.empty(
            (num_groups, max_m, n), device="cuda", dtype=torch.bfloat16
        )
```
**EN:** This block defines `_GroupedMaskedWarmupExecutor.__init__` and contains the main logic for this step. It mainly invokes `_empty_token_fp8`, `_empty_block_fp8`, `torch.zeros`, and `torch.empty`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.lhs_q`, `self.lhs_s`, `self.rhs_q`, `self.rhs_s`, and `self.masked_m` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_GroupedMaskedWarmupExecutor.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `_empty_token_fp8`、`_empty_block_fp8`、`torch.zeros` 以及 `torch.empty`，说明该流程会编排底层辅助函数或计算内核。 像 `self.lhs_q`、`self.lhs_s`、`self.rhs_q`、`self.rhs_s` 以及 `self.masked_m` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 356-366: Function `_GroupedMaskedWarmupExecutor.execute` and its core logic
```python
    def execute(self, m):
        deep_gemm.fp8_m_grouped_gemm_nt_masked(
            (self.lhs_q, self.lhs_s),
            (self.rhs_q, self.rhs_s),
            self.out,
            masked_m=self.masked_m,
            # DeepGEMM uses `expect_m` instead of input shape for `get_best_config`
            expected_m=m,
        )
```
**EN:** This block defines `_GroupedMaskedWarmupExecutor.execute` and contains the main logic for this step. It mainly invokes `deep_gemm.fp8_m_grouped_gemm_nt_masked`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `_GroupedMaskedWarmupExecutor.execute`，并承载这一阶段的核心逻辑。 它主要调用 `deep_gemm.fp8_m_grouped_gemm_nt_masked`，说明该流程会编排底层辅助函数或计算内核。

### Lines 367-367: Class `_BF16F32WarmupExecutor` declaration and shared state
```python
class _BF16F32WarmupExecutor(_BaseWarmupExecutor):
```
**EN:** This block introduces class `_BF16F32WarmupExecutor` and the state shared by its methods. It inherits from `_BaseWarmupExecutor`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `_BF16F32WarmupExecutor`，并定义其方法共享的状态。 它继承自 `_BaseWarmupExecutor`，说明了它在 SRT 层栈中的接入方式。

### Lines 368-372: `_BF16F32WarmupExecutor` initialization and state setup
```python
    def __init__(self, max_m: int, n: int, k: int, num_groups: int):
        self.lhs = torch.empty((max_m, k), device="cuda", dtype=torch.bfloat16)
        self.rhs = torch.empty((n, k), device="cuda", dtype=torch.bfloat16)
        self.out = torch.empty((max_m, n), device="cuda", dtype=torch.float32)
```
**EN:** This block defines `_BF16F32WarmupExecutor.__init__` and contains the main logic for this step. It mainly invokes `torch.empty`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.lhs`, `self.rhs`, and `self.out` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_BF16F32WarmupExecutor.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty`，说明该流程会编排底层辅助函数或计算内核。 像 `self.lhs`、`self.rhs` 以及 `self.out` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 373-376: Function `_BF16F32WarmupExecutor.execute` and its core logic
```python
    def execute(self, m):
        deep_gemm.bf16_gemm_nt(self.lhs[:m], self.rhs, self.out[:m])
```
**EN:** This block defines `_BF16F32WarmupExecutor.execute` and contains the main logic for this step. It mainly invokes `deep_gemm.bf16_gemm_nt`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `_BF16F32WarmupExecutor.execute`，并承载这一阶段的核心逻辑。 它主要调用 `deep_gemm.bf16_gemm_nt`，说明该流程会编排底层辅助函数或计算内核。

### Lines 377-377: Class `_BF16GroupedMaskedWarmupExecutor` declaration and shared state
```python
class _BF16GroupedMaskedWarmupExecutor(_BaseWarmupExecutor):
```
**EN:** This block introduces class `_BF16GroupedMaskedWarmupExecutor` and the state shared by its methods. It inherits from `_BaseWarmupExecutor`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `_BF16GroupedMaskedWarmupExecutor`，并定义其方法共享的状态。 它继承自 `_BaseWarmupExecutor`，说明了它在 SRT 层栈中的接入方式。

### Lines 378-387: `_BF16GroupedMaskedWarmupExecutor` initialization and state setup
```python
    def __init__(self, max_m: int, n: int, k: int, num_groups: int):
        self.a = torch.empty(
            (num_groups, max_m, k), device="cuda", dtype=torch.bfloat16
        )
        self.b = torch.empty((num_groups, n, k), device="cuda", dtype=torch.bfloat16)
        self.masked_m = torch.zeros((num_groups,), device="cuda", dtype=torch.int32)
        self.out = torch.empty(
            (num_groups, max_m, n), device="cuda", dtype=torch.bfloat16
        )
```
**EN:** This block defines `_BF16GroupedMaskedWarmupExecutor.__init__` and contains the main logic for this step. It mainly invokes `torch.empty` and `torch.zeros`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.a`, `self.b`, `self.masked_m`, and `self.out` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_BF16GroupedMaskedWarmupExecutor.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty` 和 `torch.zeros`，说明该流程会编排底层辅助函数或计算内核。 像 `self.a`、`self.b`、`self.masked_m` 以及 `self.out` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 388-398: Function `_BF16GroupedMaskedWarmupExecutor.execute` and its core logic
```python
    def execute(self, m):
        deep_gemm.m_grouped_bf16_gemm_nt_masked(
            self.a,
            self.b,
            self.out,
            masked_m=self.masked_m,
            # DeepGEMM uses `expect_m` instead of input shape for `get_best_config`
            expected_m=m,
        )
```
**EN:** This block defines `_BF16GroupedMaskedWarmupExecutor.execute` and contains the main logic for this step. It mainly invokes `deep_gemm.m_grouped_bf16_gemm_nt_masked`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `_BF16GroupedMaskedWarmupExecutor.execute`，并承载这一阶段的核心逻辑。 它主要调用 `deep_gemm.m_grouped_bf16_gemm_nt_masked`，说明该流程会编排底层辅助函数或计算内核。

### Lines 399-407: Function `deep_gemm_execution_hook` and its core logic
```python
def deep_gemm_execution_hook(
    m: int, n: int, k: int, num_groups: int, kernel_type: DeepGemmKernelType
):
    if _is_musa:
        return nullcontext()

    return _deep_gemm_execution_hook(m, n, k, num_groups, kernel_type)
```
**EN:** This block defines `deep_gemm_execution_hook` and contains the main logic for this step. It mainly invokes `_deep_gemm_execution_hook` and `nullcontext`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `deep_gemm_execution_hook`，并承载这一阶段的核心逻辑。 它主要调用 `_deep_gemm_execution_hook` 和 `nullcontext`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 408-414: Internal helper `_deep_gemm_execution_hook`
```python
@contextmanager
def _deep_gemm_execution_hook(
    m: int, n: int, k: int, num_groups: int, kernel_type: DeepGemmKernelType
):
    if m > 0:
        _maybe_compile_deep_gemm_one_type_all(kernel_type, n, k, num_groups)
    yield
```
**EN:** This block defines `_deep_gemm_execution_hook` and contains the main logic for this step. Decorators like `contextmanager` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `_maybe_compile_deep_gemm_one_type_all`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `_deep_gemm_execution_hook`，并承载这一阶段的核心逻辑。 像 `contextmanager` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `_maybe_compile_deep_gemm_one_type_all`，说明该流程会编排底层辅助函数或计算内核。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `update_deep_gemm_config`, `DeepGemmKernelType`, `_maybe_compile_deep_gemm_one_type_all`, `_compile_deep_gemm_one_type_all`, and `_BaseWarmupExecutor`. / **主要符号**：核心入口包括 `update_deep_gemm_config`、`DeepGemmKernelType`、`_maybe_compile_deep_gemm_one_type_all`、`_compile_deep_gemm_one_type_all` 以及 `_BaseWarmupExecutor`。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Projection layers**: Focuses on matrix multiplication, weight layout, and projection-oriented wrappers. / **投影层**：关注矩阵乘法、权重布局与面向投影的包装层。

## Dependencies / 依赖关系
- **Standard library**: `logging`, `os`, `contextlib.contextmanager`, `contextlib.nullcontext`, `enum.IntEnum`, `enum.auto`, `typing.Dict`, `typing.List`, and `typing.Tuple` / **标准库**：`logging`、`os`、`contextlib.contextmanager`、`contextlib.nullcontext`、`enum.IntEnum`、`enum.auto`、`typing.Dict`、`typing.List` 以及 `typing.Tuple`
- **Third-party**: `torch`, `tqdm.tqdm`, and `deep_gemm` / **第三方依赖**：`torch`、`tqdm.tqdm` 以及 `deep_gemm`
- **Internal SGLang modules**: `sglang.srt.distributed.device_communicators.pynccl_allocator.disable_symmetric_memory_context`, `sglang.srt.distributed.device_communicators.pynccl_allocator.restore_symmetric_memory_context`, `sglang.srt.environ.envs`, `sglang.srt.layers.deep_gemm_wrapper.configurer.ENABLE_JIT_DEEPGEMM`, `sglang.srt.server_args.ServerArgs`, `sglang.srt.utils.ceil_div`, `sglang.srt.utils.get_available_gpu_memory`, and `sglang.srt.utils.is_musa` / **SGLang 内部模块**：`sglang.srt.distributed.device_communicators.pynccl_allocator.disable_symmetric_memory_context`、`sglang.srt.distributed.device_communicators.pynccl_allocator.restore_symmetric_memory_context`、`sglang.srt.environ.envs`、`sglang.srt.layers.deep_gemm_wrapper.configurer.ENABLE_JIT_DEEPGEMM`、`sglang.srt.server_args.ServerArgs`、`sglang.srt.utils.ceil_div`、`sglang.srt.utils.get_available_gpu_memory` 以及 `sglang.srt.utils.is_musa`
