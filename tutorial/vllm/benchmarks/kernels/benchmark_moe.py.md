# benchmark_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, mixture-of-experts benchmarks, centered around `FP8_DTYPE`, `_CACHE_CLEAR_INTERVAL_ENV`, `TRITON_CACHE_CLEAR_INTERVAL`, `clear_triton_cache`. / 实现与基准测试编排、混合专家基准相关的逻辑，核心符号包括 `FP8_DTYPE`, `_CACHE_CLEAR_INTERVAL_ENV`, `TRITON_CACHE_CLEAR_INTERVAL`, `clear_triton_cache`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-37)
```python
import argparse
import gc
import json
import os
import time
from contextlib import nullcontext
from datetime import datetime
from itertools import product
from typing import Any, TypedDict

import ray
import torch
from ray.experimental.tqdm_ray import tqdm

from vllm.model_executor.layers.fused_moe import fused_topk
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.all2all_utils import (
    maybe_make_prepare_finalize,
)
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEParallelConfig,
    FusedMoEQuantConfig,
    RoutingMethodType,
    _get_config_dtype_str,
)
from vllm.model_executor.layers.fused_moe.experts.triton_deep_gemm_moe import (
    TritonOrDeepGemmExperts,
)
from vllm.model_executor.layers.fused_moe.fused_moe import *
from vllm.transformers_utils.config import get_config
from vllm.triton_utils import triton
from vllm.utils.argparse_utils import FlexibleArgumentParser
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block gathers standard-library helpers such as `argparse`, `gc`, `json`, `os`, `time`, `contextlib`; third-party packages such as `ray`, `torch`, `ray.experimental.tqdm_ray`; project-local modules such as `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.all2all_utils`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.triton_deep_gemm_moe`, `vllm.model_executor.layers.fused_moe.fused_moe`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`, `gc`, `json`, `os`, `time`, `contextlib`；第三方依赖，如 `ray`, `torch`, `ray.experimental.tqdm_ray`；项目内部模块，如 `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.all2all_utils`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.triton_deep_gemm_moe`, `vllm.model_executor.layers.fused_moe.fused_moe`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 39-44)
```python
FP8_DTYPE = current_platform.fp8_dtype()

# Default interval for clearing Triton JIT cache during tuning
# Set to 0 to disable automatic cache clearing
_CACHE_CLEAR_INTERVAL_ENV = "VLLM_MOE_TUNE_CACHE_CLEAR_INTERVAL"
TRITON_CACHE_CLEAR_INTERVAL = int(os.environ.get(_CACHE_CLEAR_INTERVAL_ENV, "50"))
```
**EN:** This top-level block prepares shared state such as `FP8_DTYPE`, `_CACHE_CLEAR_INTERVAL_ENV`, `TRITON_CACHE_CLEAR_INTERVAL`. It uses `current_platform.fp8_dtype`, `int`, `os.environ.get` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `FP8_DTYPE`, `_CACHE_CLEAR_INTERVAL_ENV`, `TRITON_CACHE_CLEAR_INTERVAL`。它借助 `current_platform.fp8_dtype`, `int`, `os.environ.get` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `clear_triton_cache` (lines 47-77)
```python
def clear_triton_cache():
    """Clear Triton JIT compilation cache and Python/CUDA memory.

    This helps prevent OOM during tuning with large models (many experts).
    """
    # Force Python garbage collection
    gc.collect()

    # Clear CUDA memory cache
    if torch.cuda.is_available():
        torch.accelerator.empty_cache()

    # Try to clear Triton's runtime cache
    try:
        if (
            hasattr(triton, "runtime")
            and hasattr(triton.runtime, "cache")
            and hasattr(triton.runtime.cache, "clear")
        ):
            triton.runtime.cache.clear()
    except ImportError:
        # Triton not installed, skip cache clearing
        pass
    except AttributeError:
        # Triton version doesn't have expected cache API
        pass
    except Exception as e:
        print(f"Warning: Failed to clear Triton cache: {e}")

    # Additional garbage collection after clearing caches
    gc.collect()
```
**EN:** `clear_triton_cache` Clear Triton JIT compilation cache and Python/CUDA memory. It mainly works with no explicit parameters and relies on `gc.collect`, `torch.cuda.is_available`, `torch.accelerator.empty_cache`, `hasattr`, `triton.runtime.cache.clear`, `print` plus branching and error handling to move data through this part of the benchmark pipeline.
**CN:** `clear_triton_cache` 的职责是：Clear Triton JIT compilation cache and Python/CUDA memory。 它主要处理 无显式参数，并结合 `gc.collect`, `torch.cuda.is_available`, `torch.accelerator.empty_cache`, `hasattr`, `triton.runtime.cache.clear`, `print` 以及 条件分支、异常处理 来完成这一段基准测试流程。

### Function `ensure_divisibility` (lines 80-84)
```python
def ensure_divisibility(numerator, denominator, text):
    """Ensure that numerator is divisible by the denominator."""
    assert numerator % denominator == 0, "{} {} is not divisible by tp {}.".format(
        text, numerator, denominator
    )
```
**EN:** `ensure_divisibility` Ensure that numerator is divisible by the denominator. It mainly works with `numerator`, `denominator`, `text` and relies on `format` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `ensure_divisibility` 的职责是：Ensure that numerator is divisible by the denominator。 它主要处理 `numerator`, `denominator`, `text`，并结合 `format` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Class `BenchmarkConfig` (lines 87-93)
```python
class BenchmarkConfig(TypedDict):
    BLOCK_SIZE_M: int
    BLOCK_SIZE_N: int
    BLOCK_SIZE_K: int
    GROUP_SIZE_M: int
    num_warps: int
    num_stages: int
```
**EN:** Class `BenchmarkConfig` packages shared state and related operations for this benchmark module. It extends `TypedDict` and exposes methods such as its methods.
**CN:** 类 `BenchmarkConfig` 用于封装该基准模块的共享状态与相关操作。它继承自 `TypedDict`，并提供 its methods 等方法。

### Function `benchmark_config` (lines 96-336)
```python
def benchmark_config(
    config: BenchmarkConfig,
    num_tokens: int,
    num_experts: int,
    shard_intermediate_size: int,
    hidden_size: int,
    topk: int,
    dtype: torch.dtype,
    use_fp8_w8a8: bool,
    use_int8_w8a16: bool,
    use_int4_w4a16: bool = False,
    num_iters: int = 100,
    block_quant_shape: list[int] = None,
    use_deep_gemm: bool = False,
) -> float:
    init_dtype = torch.float16 if use_fp8_w8a8 else dtype
    x = torch.randn(num_tokens, hidden_size, dtype=dtype)
    if use_int4_w4a16:
        # Int4 packed weights: 2 int4 values per uint8 byte
        # K dimension is packed (halved)
        intermediate_size = shard_intermediate_size // 2  # after silu_and_mul
        w1 = torch.randint(
            0,
            255,
            (
                num_experts,
                shard_intermediate_size,
                hidden_size // 2,  # int4 packing
            ),
            dtype=torch.uint8,
        )
        w2 = torch.randint(
            0,
            255,
    # ... omitted for brevity ...
        start_event.record()
        graph.replay()
        end_event.record()
        end_event.synchronize()
        latencies.append(start_event.elapsed_time(end_event))
    avg = sum(latencies) / (num_iters * 10) * 1000  # us
    graph.reset()
    return avg
```
**EN:** `benchmark_config` coordinates or measures benchmark orchestration. It mainly works with `config`, `num_tokens`, `num_experts`, `shard_intermediate_size`, `hidden_size`, ... and relies on `torch.randn`, `torch.randint`, `ValueError`, `torch.rand`, `w1.to`, `w2.to` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `benchmark_config` 负责协调或测量基准测试编排。 它主要处理 `config`, `num_tokens`, `num_experts`, `shard_intermediate_size`, `hidden_size`, ...，并结合 `torch.randn`, `torch.randint`, `ValueError`, `torch.rand`, `w1.to`, `w2.to` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Function `get_rocm_tuning_space` (lines 339-364)
```python
def get_rocm_tuning_space(use_fp16):
    block_mn_range = [16, 32, 64, 128, 256]
    block_k_range = [16, 32, 64, 128, 256]
    if not use_fp16:
        block_k_range.remove(16)  # BLOCK_K=16 not supported for fp8
    num_warps_range = [1, 2, 4, 8]
    group_m_range = [1, 4, 8, 16, 32]
    num_stage_range = [2]
    waves_per_eu_range = [0, 1, 2, 4]
    matrix_instr_nonkdim_range = [16, 32] if use_fp16 else []
    kpack_range = [1, 2] if use_fp16 else []

    param_ranges = {
        "BLOCK_SIZE_M": block_mn_range,
        "BLOCK_SIZE_N": block_mn_range,
        "BLOCK_SIZE_K": block_k_range,
        "GROUP_SIZE_M": group_m_range,
        "num_warps": num_warps_range,
        "num_stages": num_stage_range,
        "waves_per_eu": waves_per_eu_range,
    }
    if use_fp16:
        param_ranges["matrix_instr_nonkdim"] = matrix_instr_nonkdim_range
        param_ranges["kpack"] = kpack_range

    return param_ranges
```
**EN:** `get_rocm_tuning_space` implements a helper used by `benchmark_moe.py`. It mainly works with `use_fp16` and relies on `block_k_range.remove` plus branching to move data through this part of the benchmark pipeline.
**CN:** `get_rocm_tuning_space` 负责实现 `benchmark_moe.py` 使用的辅助逻辑。 它主要处理 `use_fp16`，并结合 `block_k_range.remove` 以及 条件分支 来完成这一段基准测试流程。

### Function `get_configs_compute_bound` (lines 367-408)
```python
def get_configs_compute_bound(use_fp16, block_quant_shape) -> list[dict[str, int]]:
    configs: list[BenchmarkConfig] = []

    if current_platform.is_rocm():
        param_ranges = get_rocm_tuning_space(use_fp16)
    else:
        # Reduced search space for faster tuning.
        # TODO(woosuk): Increase the search space and use a performance model to
        # prune the search space.
        block_m_range = [16, 32, 64, 128, 256]
        block_n_range = [32, 64, 128, 256]
        block_k_range = [64, 128, 256]
        num_warps_range = [4, 8]
        group_m_range = [1, 16, 32, 64]
        num_stage_range = [2, 3, 4, 5]

        param_ranges = {
            "BLOCK_SIZE_M": block_m_range,
            "BLOCK_SIZE_N": block_n_range,
            "BLOCK_SIZE_K": block_k_range,
            "GROUP_SIZE_M": group_m_range,
            "num_warps": num_warps_range,
            "num_stages": num_stage_range,
        }

    keys, values = zip(*param_ranges.items())
    for config_values in product(*values):
        config = dict(zip(keys, config_values))
        configs.append(config)

    # Remove configs that are not compatible with fp8 block quantization
    # BLOCK_SIZE_K must be a multiple of block_k
    # BLOCK_SIZE_N must be a multiple of block_n
    if block_quant_shape is not None and not use_fp16:
        block_n, block_k = block_quant_shape[0], block_quant_shape[1]
        for config in configs[:]:
            if (
                config["BLOCK_SIZE_K"] % block_k != 0
                or config["BLOCK_SIZE_N"] % block_n != 0
            ):
                configs.remove(config)
    return configs
```
**EN:** `get_configs_compute_bound` implements a helper used by `benchmark_moe.py`. It mainly works with `use_fp16`, `block_quant_shape` and relies on `current_platform.is_rocm`, `get_rocm_tuning_space`, `zip`, `param_ranges.items`, `product`, `dict` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `get_configs_compute_bound` 负责实现 `benchmark_moe.py` 使用的辅助逻辑。 它主要处理 `use_fp16`, `block_quant_shape`，并结合 `current_platform.is_rocm`, `get_rocm_tuning_space`, `zip`, `param_ranges.items`, `product`, `dict` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `prune_rocm_search_space` (lines 411-423)
```python
def prune_rocm_search_space(
    num_tokens, shard_intermediate_size, hidden_size, search_space, is_fp16, topk
):
    N1, K1 = shard_intermediate_size, hidden_size
    N2, K2 = hidden_size, shard_intermediate_size // 2
    pruned_space_1 = prune_rocm_configs(
        num_tokens * topk, N1, K1, search_space, is_fp16
    )
    pruned_space_2 = prune_rocm_configs(
        num_tokens * topk, N2, K2, search_space, is_fp16
    )
    search_space = merge_unique_dicts(pruned_space_1, pruned_space_2)
    return search_space
```
**EN:** `prune_rocm_search_space` implements a helper used by `benchmark_moe.py`. It mainly works with `num_tokens`, `shard_intermediate_size`, `hidden_size`, `search_space`, `is_fp16`, ... and relies on `prune_rocm_configs`, `merge_unique_dicts` plus value production to move data through this part of the benchmark pipeline.
**CN:** `prune_rocm_search_space` 负责实现 `benchmark_moe.py` 使用的辅助逻辑。 它主要处理 `num_tokens`, `shard_intermediate_size`, `hidden_size`, `search_space`, `is_fp16`, ...，并结合 `prune_rocm_configs`, `merge_unique_dicts` 以及 结果返回 来完成这一段基准测试流程。

### Function `prune_rocm_configs` (lines 428-505)
```python
def prune_rocm_configs(M, N, K, configs, is_fp16=True):
    pruned_configs = []
    elemBytes_a = 2 if is_fp16 else 1
    elemBytes_b = 2 if is_fp16 else 1

    mfma = 16 if M < 32 or N < 32 else 32

    # TODO (zhanglx): figure out the boundary between large and small gemms
    large_gemm = False
    if M >= 2048 and N >= 2048:
        large_gemm = True

    for config in configs:
        BLOCK_SIZE_M = config.get("BLOCK_SIZE_M")
        BLOCK_SIZE_N = config.get("BLOCK_SIZE_N")
        BLOCK_SIZE_K = config.get("BLOCK_SIZE_K")
        num_warps = config.get("num_warps")

        if is_fp16:
            matrix_instr_nonkdim = config.get("matrix_instr_nonkdim")
            if matrix_instr_nonkdim > mfma:
                continue
        if mfma == 4 and BLOCK_SIZE_K < 64:
            continue
        # some layouts could not work properly in case
        # number elements per thread is less 1
        if BLOCK_SIZE_M * BLOCK_SIZE_N < 64:
            continue
        SPLIT_K = config.get("SPLIT_K", 1)
        GROUP_M = config.get("GROUP_SIZE_M")
        if is_fp16:
            if (
                matrix_instr_nonkdim > BLOCK_SIZE_M
                or matrix_instr_nonkdim > BLOCK_SIZE_N
    # ... omitted for brevity ...
            if BLOCK_SIZE_K < 64:
                continue
            if num_warps < 4:
                continue

        pruned_configs.append(config)

    return pruned_configs
```
**EN:** `prune_rocm_configs` implements a helper used by `benchmark_moe.py`. It mainly works with `M`, `N`, `K`, `configs`, `is_fp16` and relies on `config.get`, `need_split_k`, `pruned_configs.append` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `prune_rocm_configs` 负责实现 `benchmark_moe.py` 使用的辅助逻辑。 它主要处理 `M`, `N`, `K`, `configs`, `is_fp16`，并结合 `config.get`, `need_split_k`, `pruned_configs.append` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `need_split_k` (lines 508-509)
```python
def need_split_k(SIZE_M, SIZE_N, SIZE_K):
    return (SIZE_M < 64 or SIZE_N < 64) and SIZE_K > 1024
```
**EN:** `need_split_k` implements a helper used by `benchmark_moe.py`. It mainly works with `SIZE_M`, `SIZE_N`, `SIZE_K` and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `need_split_k` 负责实现 `benchmark_moe.py` 使用的辅助逻辑。 它主要处理 `SIZE_M`, `SIZE_N`, `SIZE_K`，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Function `merge_unique_dicts` (lines 512-519)
```python
def merge_unique_dicts(list1, list2):
    result = []
    combined_list = list1.copy()
    combined_list.extend(list2)
    for dictionary in combined_list:
        if dictionary not in result:
            result.append(dictionary)
    return result
```
**EN:** `merge_unique_dicts` implements a helper used by `benchmark_moe.py`. It mainly works with `list1`, `list2` and relies on `list1.copy`, `combined_list.extend`, `result.append` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `merge_unique_dicts` 负责实现 `benchmark_moe.py` 使用的辅助逻辑。 它主要处理 `list1`, `list2`，并结合 `list1.copy`, `combined_list.extend`, `result.append` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Class `BenchmarkWorker` (lines 523-673)
```python
class BenchmarkWorker:
    def __init__(self, seed: int) -> None:
        torch.set_default_device("cuda")
        set_random_seed(seed)
        self.seed = seed
        # Get the device ID to allocate tensors and kernels
        # on the respective GPU. This is required for Ray to work
        # correctly with multi-GPU tuning on the ROCm platform.
        self.device_id = int(ray.get_gpu_ids()[0])

    def benchmark(
        self,
        num_tokens: int,
        num_experts: int,
        shard_intermediate_size: int,
        hidden_size: int,
        topk: int,
        dtype: torch.dtype,
        use_fp8_w8a8: bool,
        use_int8_w8a16: bool,
        use_int4_w4a16: bool = False,
        block_quant_shape: list[int] = None,
        use_deep_gemm: bool = False,
    ) -> tuple[dict[str, int], float]:
        # local import to allow serialization by ray

        set_random_seed(self.seed)
        dtype_str = _get_config_dtype_str(
            dtype,
            use_int8_w8a16=use_int8_w8a16,
            use_fp8_w8a8=use_fp8_w8a8,
            use_int4_w4a16=use_int4_w4a16,
        )
        # NOTE(woosuk): The current naming convention uses w2.shape[2], which
    # ... omitted for brevity ...

        # Final cleanup after tuning completes
        clear_triton_cache()

        now = datetime.now()
        print(f"{now.ctime()}] Completed tuning for batch_size={num_tokens}")
        assert best_config is not None
        return best_config
```
**EN:** Class `BenchmarkWorker` packages shared state and related operations for this benchmark module. It extends `object` and exposes methods such as `__init__`, `benchmark`, `tune`.
**CN:** 类 `BenchmarkWorker` 用于封装该基准模块的共享状态与相关操作。它继承自 `object`，并提供 `__init__`, `benchmark`, `tune` 等方法。

### Method `BenchmarkWorker.__init__` (lines 524-531)
```python
    def __init__(self, seed: int) -> None:
        torch.set_default_device("cuda")
        set_random_seed(seed)
        self.seed = seed
        # Get the device ID to allocate tensors and kernels
        # on the respective GPU. This is required for Ray to work
        # correctly with multi-GPU tuning on the ROCm platform.
        self.device_id = int(ray.get_gpu_ids()[0])
```
**EN:** `__init__` implements a helper used by `benchmark_moe.py`. It mainly works with `seed` and relies on `torch.set_default_device`, `set_random_seed`, `int`, `ray.get_gpu_ids` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `__init__` 负责实现 `benchmark_moe.py` 使用的辅助逻辑。 它主要处理 `seed`，并结合 `torch.set_default_device`, `set_random_seed`, `int`, `ray.get_gpu_ids` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Method `BenchmarkWorker.benchmark` (lines 533-590)
```python
    def benchmark(
        self,
        num_tokens: int,
        num_experts: int,
        shard_intermediate_size: int,
        hidden_size: int,
        topk: int,
        dtype: torch.dtype,
        use_fp8_w8a8: bool,
        use_int8_w8a16: bool,
        use_int4_w4a16: bool = False,
        block_quant_shape: list[int] = None,
        use_deep_gemm: bool = False,
    ) -> tuple[dict[str, int], float]:
        # local import to allow serialization by ray

        set_random_seed(self.seed)
        dtype_str = _get_config_dtype_str(
            dtype,
            use_int8_w8a16=use_int8_w8a16,
            use_fp8_w8a8=use_fp8_w8a8,
            use_int4_w4a16=use_int4_w4a16,
        )
        # NOTE(woosuk): The current naming convention uses w2.shape[2], which
        # is the intermediate size after silu_and_mul.
        block_n = block_quant_shape[0] if block_quant_shape else None
        block_k = block_quant_shape[1] if block_quant_shape else None
        op_config = get_moe_configs(
            num_experts, shard_intermediate_size // 2, dtype_str, block_n, block_k
        )
        if op_config is None:
            config = get_default_config(
                num_tokens,
                num_experts,
        # ... omitted for brevity ...
            use_fp8_w8a8,
            use_int8_w8a16,
            use_int4_w4a16=use_int4_w4a16,
            num_iters=100,
            block_quant_shape=block_quant_shape,
            use_deep_gemm=use_deep_gemm,
        )
        return config, kernel_time
```
**EN:** `benchmark` coordinates or measures benchmark orchestration. It mainly works with `num_tokens`, `num_experts`, `shard_intermediate_size`, `hidden_size`, `topk`, ... and relies on `set_random_seed`, `_get_config_dtype_str`, `get_moe_configs`, `get_default_config`, `min`, `op_config.keys` plus branching to move data through this part of the benchmark pipeline.
**CN:** `benchmark` 负责协调或测量基准测试编排。 它主要处理 `num_tokens`, `num_experts`, `shard_intermediate_size`, `hidden_size`, `topk`, ...，并结合 `set_random_seed`, `_get_config_dtype_str`, `get_moe_configs`, `get_default_config`, `min`, `op_config.keys` 以及 条件分支 来完成这一段基准测试流程。

### Method `BenchmarkWorker.tune` (lines 592-673)
```python
    def tune(
        self,
        num_tokens: int,
        num_experts: int,
        shard_intermediate_size: int,
        hidden_size: int,
        topk: int,
        dtype: torch.dtype,
        use_fp8_w8a8: bool,
        use_int8_w8a16: bool,
        use_int4_w4a16: bool,
        search_space: list[dict[str, int]],
        block_quant_shape: list[int],
        use_deep_gemm: bool,
    ) -> dict[str, int]:
        # local import to allow serialization by ray
        from vllm.platforms import current_platform

        best_config = None
        best_time = float("inf")
        if current_platform.is_rocm():
            is_fp16 = not (use_fp8_w8a8 or use_int8_w8a16 or use_int4_w4a16)
            search_space = prune_rocm_search_space(
                num_tokens,
                shard_intermediate_size,
                hidden_size,
                search_space,
                is_fp16,
                topk,
            )

        need_device_guard = False
        if current_platform.is_rocm():
            visible_device = os.environ.get("ROCR_VISIBLE_DEVICES", None)
        # ... omitted for brevity ...

        # Final cleanup after tuning completes
        clear_triton_cache()

        now = datetime.now()
        print(f"{now.ctime()}] Completed tuning for batch_size={num_tokens}")
        assert best_config is not None
        return best_config
```
**EN:** `tune` implements a helper used by `benchmark_moe.py`. It mainly works with `num_tokens`, `num_experts`, `shard_intermediate_size`, `hidden_size`, `topk`, ... and relies on `float`, `current_platform.is_rocm`, `prune_rocm_search_space`, `os.environ.get`, `torch.accelerator.device_index`, `nullcontext` plus iteration, branching, error handling and context management to move data through this part of the benchmark pipeline.
**CN:** `tune` 负责实现 `benchmark_moe.py` 使用的辅助逻辑。 它主要处理 `num_tokens`, `num_experts`, `shard_intermediate_size`, `hidden_size`, `topk`, ...，并结合 `float`, `current_platform.is_rocm`, `prune_rocm_search_space`, `os.environ.get`, `torch.accelerator.device_index`, `nullcontext` 以及 循环迭代、条件分支、异常处理、上下文管理 来完成这一段基准测试流程。

### Function `sort_config` (lines 676-694)
```python
def sort_config(config: BenchmarkConfig) -> BenchmarkConfig:
    return {
        "BLOCK_SIZE_M": config["BLOCK_SIZE_M"],
        "BLOCK_SIZE_N": config["BLOCK_SIZE_N"],
        "BLOCK_SIZE_K": config["BLOCK_SIZE_K"],
        "GROUP_SIZE_M": config["GROUP_SIZE_M"],
        "num_warps": config["num_warps"],
        "num_stages": config["num_stages"],
        **(
            {"waves_per_eu": config["waves_per_eu"]} if "waves_per_eu" in config else {}
        ),
        **(
            {"matrix_instr_nonkdim": config["matrix_instr_nonkdim"]}
            if "matrix_instr_nonkdim" in config
            else {}
        ),
        **({"kpack": config["kpack"]} if "kpack" in config else {}),
        **({"SPLIT_K": config["SPLIT_K"]} if "SPLIT_K" in config else {}),
    }
```
**EN:** `sort_config` implements a helper used by `benchmark_moe.py`. It mainly works with `config` and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `sort_config` 负责实现 `benchmark_moe.py` 使用的辅助逻辑。 它主要处理 `config`，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Function `save_configs` (lines 697-727)
```python
def save_configs(
    configs: dict[int, BenchmarkConfig],
    num_experts: int,
    shard_intermediate_size: int,
    hidden_size: int,
    topk: int,
    dtype: torch.dtype,
    use_fp8_w8a8: bool,
    use_int8_w8a16: bool,
    use_int4_w4a16: bool,
    block_quant_shape: list[int],
    save_dir: str,
) -> None:
    dtype_str = _get_config_dtype_str(
        dtype,
        use_int8_w8a16=use_int8_w8a16,
        use_fp8_w8a8=use_fp8_w8a8,
        use_int4_w4a16=use_int4_w4a16,
    )

    # NOTE(woosuk): The current naming convention uses w2.shape[2], which
    # is the intermediate size after silu_and_mul.
    filename = get_config_file_name(
        num_experts, shard_intermediate_size // 2, dtype_str, block_quant_shape
    )
    os.makedirs(save_dir, exist_ok=True)
    filename = os.path.join(save_dir, filename)
    print(f"Writing best config to {filename}...")
    with open(filename, "w") as f:
        json.dump({"triton_version": triton.__version__, **configs}, f, indent=4)
        f.write("\n")
```
**EN:** `save_configs` writes benchmark outputs to persistent storage. It mainly works with `configs`, `num_experts`, `shard_intermediate_size`, `hidden_size`, `topk`, ... and relies on `_get_config_dtype_str`, `get_config_file_name`, `os.makedirs`, `os.path.join`, `print`, `open` plus context management to move data through this part of the benchmark pipeline.
**CN:** `save_configs` 负责将基准结果写入持久化存储。 它主要处理 `configs`, `num_experts`, `shard_intermediate_size`, `hidden_size`, `topk`, ...，并结合 `_get_config_dtype_str`, `get_config_file_name`, `os.makedirs`, `os.path.join`, `print`, `open` 以及 上下文管理 来完成这一段基准测试流程。

### Function `get_compressed_tensors_block_structure` (lines 730-737)
```python
def get_compressed_tensors_block_structure(config, default_value=None):
    config_groups = config.get("config_groups", {})
    if len(config_groups) != 1:
        return default_value
    group = next(iter(config_groups.values()))
    weights = group.get("weights", {})
    block_structure = weights.get("block_structure", default_value)
    return block_structure
```
**EN:** `get_compressed_tensors_block_structure` implements a helper used by `benchmark_moe.py`. It mainly works with `config`, `default_value` and relies on `config.get`, `len`, `next`, `iter`, `config_groups.values`, `group.get` plus branching to move data through this part of the benchmark pipeline.
**CN:** `get_compressed_tensors_block_structure` 负责实现 `benchmark_moe.py` 使用的辅助逻辑。 它主要处理 `config`, `default_value`，并结合 `config.get`, `len`, `next`, `iter`, `config_groups.values`, `group.get` 以及 条件分支 来完成这一段基准测试流程。

### Function `get_weight_block_size_safety` (lines 740-748)
```python
def get_weight_block_size_safety(config, default_value=None):
    quantization_config = getattr(config, "quantization_config", {})
    if isinstance(quantization_config, dict):
        if "weight_block_size" in quantization_config:
            return quantization_config["weight_block_size"]
        return get_compressed_tensors_block_structure(
            quantization_config, default_value
        )
    return default_value
```
**EN:** `get_weight_block_size_safety` implements a helper used by `benchmark_moe.py`. It mainly works with `config`, `default_value` and relies on `getattr`, `isinstance`, `get_compressed_tensors_block_structure` plus branching to move data through this part of the benchmark pipeline.
**CN:** `get_weight_block_size_safety` 负责实现 `benchmark_moe.py` 使用的辅助逻辑。 它主要处理 `config`, `default_value`，并结合 `getattr`, `isinstance`, `get_compressed_tensors_block_structure` 以及 条件分支 来完成这一段基准测试流程。

### Function `get_model_params` (lines 751-820)
```python
def get_model_params(config):
    architectures = getattr(config, "architectures", None) or [type(config).__name__]
    architecture = architectures[0]

    if architecture == "DbrxForCausalLM":
        E = config.ffn_config.moe_num_experts
        topk = config.ffn_config.moe_top_k
        intermediate_size = config.ffn_config.ffn_hidden_size
        hidden_size = config.hidden_size
    elif architecture == "JambaForCausalLM":
        E = config.num_experts
        topk = config.num_experts_per_tok
        intermediate_size = config.intermediate_size
        hidden_size = config.hidden_size
    elif architecture in (
        "DeepseekV2ForCausalLM",
        "DeepseekV3ForCausalLM",
        "DeepseekV32ForCausalLM",
        "GlmMoeDsaForCausalLM",
        "Glm4MoeForCausalLM",
        "Glm4MoeLiteForCausalLM",
        "NemotronHForCausalLM",
        "MistralLarge3ForCausalLM",
    ):
        E = config.n_routed_experts
        topk = config.num_experts_per_tok
        intermediate_size = config.moe_intermediate_size
        hidden_size = config.hidden_size
    elif architecture in (
        "Qwen2MoeForCausalLM",
        "Qwen3MoeForCausalLM",
        "Qwen3NextForCausalLM",
    ):
        E = config.num_experts
    # ... omitted for brevity ...
        # Support for llama4
        config = config.get_text_config()
        # Default: Mixtral.
        E = config.num_local_experts
        topk = config.num_experts_per_tok
        intermediate_size = config.intermediate_size
        hidden_size = config.hidden_size
    return E, topk, intermediate_size, hidden_size
```
**EN:** `get_model_params` implements a helper used by `benchmark_moe.py`. It mainly works with `config` and relies on `getattr`, `type`, `config.get_text_config`, `get_model_params` plus branching to move data through this part of the benchmark pipeline.
**CN:** `get_model_params` 负责实现 `benchmark_moe.py` 使用的辅助逻辑。 它主要处理 `config`，并结合 `getattr`, `type`, `config.get_text_config`, `get_model_params` 以及 条件分支 来完成这一段基准测试流程。

### Function `resolve_dtype` (lines 823-837)
```python
def resolve_dtype(config) -> torch.dtype:
    if current_platform.is_rocm():
        return torch.float16

    dtype = getattr(config, "dtype", None)
    if dtype is not None:
        return dtype

    if hasattr(config, "get_text_config"):
        text_config = config.get_text_config()
        dtype = getattr(text_config, "dtype", None)
        if dtype is not None:
            return dtype

    return torch.bfloat16
```
**EN:** `resolve_dtype` implements a helper used by `benchmark_moe.py`. It mainly works with `config` and relies on `current_platform.is_rocm`, `getattr`, `hasattr`, `config.get_text_config` plus branching to move data through this part of the benchmark pipeline.
**CN:** `resolve_dtype` 负责实现 `benchmark_moe.py` 使用的辅助逻辑。 它主要处理 `config`，并结合 `current_platform.is_rocm`, `getattr`, `hasattr`, `config.get_text_config` 以及 条件分支 来完成这一段基准测试流程。

### Function `get_quantization_group_size` (lines 840-869)
```python
def get_quantization_group_size(config) -> int | None:
    """Extract the quantization group size from the HF model config.

    This reads directly from the HuggingFace config object (as returned by
    ``get_config()``), not from vLLM's quantization config classes.

    Supports AWQ/GPTQ-style configs (direct 'group_size' key) and
    compressed-tensors configs (nested inside 'config_groups').
    """
    quantization_config = getattr(config, "quantization_config", {})
    if not isinstance(quantization_config, dict):
        return None
    # AWQ / GPTQ style: group_size is a top-level key
    gs = quantization_config.get("group_size")
    if gs is not None:
        return gs
    # compressed-tensors style: group_size is nested in config_groups
    config_groups = quantization_config.get("config_groups", {})
    if not isinstance(config_groups, dict):
        return None
    for group_cfg in config_groups.values():
        if not isinstance(group_cfg, dict):
            continue
        weights = group_cfg.get("weights", {})
        if not isinstance(weights, dict):
            continue
        gs = weights.get("group_size")
        if gs is not None:
            return gs
    return None
```
**EN:** `get_quantization_group_size` Extract the quantization group size from the HF model config. It mainly works with `config` and relies on `getattr`, `isinstance`, `quantization_config.get`, `config_groups.values`, `group_cfg.get`, `weights.get` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `get_quantization_group_size` 的职责是：Extract the quantization group size from the HF model config。 它主要处理 `config`，并结合 `getattr`, `isinstance`, `quantization_config.get`, `config_groups.values`, `group_cfg.get`, `weights.get` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `main` (lines 872-1039)
```python
def main(args: argparse.Namespace):
    print(args)

    config = get_config(model=args.model, trust_remote_code=args.trust_remote_code)
    if args.model_prefix:
        config = getattr(config, args.model_prefix)
    E, topk, intermediate_size, hidden_size = get_model_params(config)
    enable_ep = bool(args.enable_expert_parallel)
    if enable_ep:
        ensure_divisibility(E, args.tp_size, "Number of experts")
        E = E // args.tp_size
        shard_intermediate_size = 2 * intermediate_size
    else:
        ensure_divisibility(intermediate_size, args.tp_size, "intermediate_size")
        shard_intermediate_size = 2 * intermediate_size // args.tp_size
    dtype = resolve_dtype(config)
    use_fp8_w8a8 = args.dtype == "fp8_w8a8"
    use_int8_w8a16 = args.dtype == "int8_w8a16"
    use_int4_w4a16 = args.dtype == "int4_w4a16"
    block_quant_shape = get_weight_block_size_safety(config)
    if use_int4_w4a16:
        group_size = get_quantization_group_size(config)
        if group_size is None:
            raise ValueError(
                "Could not determine group_size from model config. "
                "The model's quantization_config must contain a 'group_size' "
                "field (AWQ/GPTQ) or 'config_groups.*.weights.group_size' "
                "(compressed-tensors)."
            )
        # For int4_w4a16, block_shape = [0, group_size]
        # block_shape[0]=0 means no block quantization on N dimension
        block_quant_shape = [0, group_size]

    if args.batch_size is None:
    # ... omitted for brevity ...
                )
                for batch_size in batch_sizes
            ],
        )

        for batch_size, (config, kernel_time) in zip(batch_sizes, outputs):
            print(f"Batch size: {batch_size}, config: {config}")
            print(f"Kernel time: {kernel_time:.2f} us")
```
**EN:** `main` serves as the main entry point for this script. It mainly works with `args` and relies on `print`, `get_config`, `getattr`, `get_model_params`, `bool`, `ensure_divisibility` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 `args`，并结合 `print`, `get_config`, `getattr`, `get_model_params`, `bool`, `ensure_divisibility` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Entry point (lines 1042-1068)
```python
if __name__ == "__main__":
    parser = FlexibleArgumentParser()
    parser.add_argument(
        "--model", type=str, default="mistralai/Mixtral-8x7B-Instruct-v0.1"
    )
    parser.add_argument(
        "--tp-size", "-tp", "--tensor-parallel-size", type=int, default=2
    )
    parser.add_argument("--enable-expert-parallel", "-enable-ep", action="store_true")
    parser.add_argument(
        "--dtype",
        type=str,
        choices=["auto", "fp8_w8a8", "int8_w8a16", "int4_w4a16"],
        default="auto",
    )
    parser.add_argument("--use-deep-gemm", action="store_true")
    parser.add_argument(
        "--save-dir", type=str, default="./", help="Directory to save tuned results"
    )
    parser.add_argument("--seed", type=int, default=0)
    parser.add_argument("--batch-size", type=int, nargs="+", required=False)
    parser.add_argument("--tune", action="store_true")
    parser.add_argument("--trust-remote-code", action="store_true")
    parser.add_argument("--model-prefix", type=str, required=False)
    args = parser.parse_args()

    main(args)
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `main` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `main` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Distributed execution: the code coordinates work across processes, devices, or Ray workers.
- **CN:** 分布式执行：代码会在多个进程、设备或 Ray worker 之间协调工作。
- **EN:** Serving path benchmarking: the script interacts with HTTP services, servers, or request proxies.
- **CN:** 服务路径基准：脚本会与 HTTP 服务、服务器或请求代理交互。
- **EN:** MoE specialization: the benchmark focuses on expert routing, expert GEMMs, or fused MoE kernels.
- **CN:** MoE 专项：该基准聚焦专家路由、专家 GEMM 或融合 MoE 内核。

## Dependencies / 依赖关系
- **EN:** Standard library: `argparse`, `gc`, `json`, `os`, `time`, `contextlib`.
- **CN:** 标准库依赖：`argparse`, `gc`, `json`, `os`, `time`, `contextlib`。
- **EN:** Third-party packages: `ray`, `torch`, `ray.experimental.tqdm_ray`.
- **CN:** 第三方依赖：`ray`, `torch`, `ray.experimental.tqdm_ray`。
- **EN:** Internal modules: `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.all2all_utils`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.triton_deep_gemm_moe`, `vllm.model_executor.layers.fused_moe.fused_moe`.
- **CN:** 内部模块：`vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.all2all_utils`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.triton_deep_gemm_moe`, `vllm.model_executor.layers.fused_moe.fused_moe`。
