# score_mod.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `benchmarks/transformer/score_mod.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements Python-side logic used for benchmarking, tooling, validation, or repository automation. Backend or dtype coverage is a visible axis of variation in the implementation.
- **Purpose (CN)**: 实现用于基准、工具链、校验或仓库自动化的 Python 侧逻辑。 后端或 dtype 覆盖是该实现中明显的变化维度。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19 / 第 1-19 行

```python
import argparse
import csv
import gc
import itertools
import json
import random
import sys
from collections import defaultdict
from collections.abc import Callable
from contextlib import nullcontext
from dataclasses import asdict, dataclass
from functools import partial, wraps
from typing import Literal

import numpy as np
from config_utils import heads_input_type, load_config_file, print_default_config
from tabulate import tabulate
from tqdm import tqdm

```

- **EN:** This range mainly pulls in the imports/includes that the later benchmark or integration logic depends on.
- **CN:** 这一段主要引入后续基准或集成逻辑依赖的 import/include。

### Lines 20-35 / 第 20-35 行

```python
import torch
import torch.nn.functional as F
from torch.nn.attention import sdpa_kernel, SDPBackend
from torch.nn.attention.flex_attention import (
    BlockMask,
    create_block_mask,
    create_mask,
    flex_attention,
    noop_mask,
)


torch._dynamo.config.automatic_dynamic_shapes = False
# Needed since changing args to function causes recompiles
torch._dynamo.config.recompile_limit = 1000

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 36-50 / 第 36-50 行

```python

from torch._inductor.runtime.benchmarking import benchmarker


def cleanup_memory():
    """Aggressively free GPU memory"""
    torch.cuda.empty_cache()
    gc.collect()
    if torch.cuda.is_available():
        torch.cuda.synchronize()


def safe_backend(backend_name=None, return_dict=False):
    """Decorator that wraps backend functions with error handling

```

- **EN:** Important local symbols in this block include cleanup_memory, safe_backend.
- **CN:** 该代码块中的重要局部符号包括 cleanup_memory、safe_backend。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 51-69 / 第 51-69 行

```python
    Args:
        backend_name: Name of the backend for error messages
        return_dict: If True, returns dict of results for all backends (for run_single_experiment)
                     If False, returns single ExperimentResults (for individual backend functions)
    """

    def decorator(func):
        @wraps(func)
        def wrapper(config, *args, **kwargs):
            try:
                return func(config, *args, **kwargs)
            except torch.OutOfMemoryError:
                print(
                    f"[SKIP] OOM for {backend_name or func.__name__} with shape {config.shape}"
                )
                cleanup_memory()
            except RuntimeError as e:
                error_msg = str(e)
                if "out of resource" in error_msg or "OutOfMemoryError" in error_msg:
```

- **EN:** Important local symbols in this block include decorator, wrapper.
- **CN:** 该代码块中的重要局部符号包括 decorator、wrapper。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 70-86 / 第 70-86 行

```python
                    print(
                        f"[SKIP] Triton OOM for {backend_name or func.__name__} with shape {config.shape}"
                    )
                    cleanup_memory()
                elif "No valid triton configs" in error_msg:
                    print(
                        f"[SKIP] No valid Triton config for {backend_name or func.__name__} with shape {config.shape}"
                    )
                else:
                    print(
                        f"[SKIP] Runtime error for {backend_name or func.__name__} with shape {config.shape}: {str(e)[:100]}"
                    )
            except Exception as e:
                print(
                    f"[SKIP] Error for {backend_name or func.__name__} with shape {config.shape}: {str(e)[:100]}"
                )

```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 87-103 / 第 87-103 行

```python
            # Return appropriate NaN result based on function type
            if return_dict:
                # For run_single_experiment: return dict with NaN for all backends
                nan_result = ExperimentResults(
                    fwd_time=float("nan"),
                    bwd_time=float("nan") if config.calculate_bwd_time else None,
                )
                results = dict.fromkeys(config.backends, nan_result)
                results["flex"] = ExperimentResults(
                    fwd_time=float("nan"),
                    bwd_time=float("nan") if config.calculate_bwd_time else None,
                    sparsity=None,
                )
                return results
            else:
                # For individual backend functions: return single ExperimentResults
                return ExperimentResults(
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 104-133 / 第 104-133 行

```python
                    fwd_time=float("nan"),
                    bwd_time=float("nan") if config.calculate_bwd_time else None,
                )

        return wrapper

    return decorator


# Type definitions
Backend = Literal["math", "efficient", "cudnn", "fav2", "fav3", "fakv", "og-eager"]
AttentionType = Literal[
    "noop",
    "causal",
    "rel",
    "head_bias",
    "alibi",
    "sliding_window",
    "document_mask",
    "prefix_lm",
    "softcap",
]
DtypeString = Literal["bfloat16", "float16", "float32"]
SpeedupType = Literal["fwd", "bwd"]
# Operator Name mapping
backend_to_operator_name = {
    "math": "math attention kernel",
    "efficient": "efficient attention kernel",
    "cudnn": "cudnn attention kernel",
    "fav2": "flash attention 2 kernel",
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 134-149 / 第 134-149 行

```python
    "fav3": "flash attention 3 kernel",
    "fakv": "flash attention kv cache kernel",
    "og-eager": "eager attention kernel",
    "flex": "flex attention kernel",
}


def benchmark_torch_function_in_microseconds(func: Callable, *args, **kwargs) -> float:
    # warmup
    for _ in range(5):
        func(*args, **kwargs)
    return benchmarker.benchmark_gpu(lambda: func(*args, **kwargs)) * 1e3


@dataclass(frozen=True)
class ExperimentConfig:
```

- **EN:** Important local symbols in this block include ExperimentConfig, benchmark_torch_function_in_microseconds.
- **CN:** 该代码块中的重要局部符号包括 ExperimentConfig、benchmark_torch_function_in_microseconds。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 150-164 / 第 150-164 行

```python
    shape: tuple[int, ...]  # [B, Hq, M, Hkv, N, D]
    attn_type: str
    dtype: torch.dtype
    calculate_bwd_time: bool
    cal_bandwidth: bool
    backends: list[str]
    max_autotune: bool

    def __post_init__(self):
        if len(self.shape) != 6:
            raise AssertionError(
                f"Shape must be of length 6 [B, Hq, M, Hkv, N, D], got {len(self.shape)}"
            )

    def asdict(self):
```

- **EN:** Important local symbols in this block include __post_init__, asdict.
- **CN:** 该代码块中的重要局部符号包括 __post_init__、asdict。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 165-180 / 第 165-180 行

```python
        # Convert the dataclass instance to a dictionary
        d = asdict(self)
        # Remove the 'calculate_bwd_time' and `cal_bandwidth` key
        d.pop("calculate_bwd_time", None)
        d.pop("cal_bandwidth", None)
        d["shape(B,Hq,M,Hkv,N,D)"] = d.pop("shape")
        d.pop("backends", None)
        d.pop("max_autotune", False)
        return d


@dataclass(frozen=True)
class Times:
    eager_time: float
    compiled_time: float

```

- **EN:** Important local symbols in this block include Times.
- **CN:** 该代码块中的重要局部符号包括 Times。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 181-197 / 第 181-197 行

```python

@dataclass(frozen=True)
class ExperimentResults:
    fwd_time: float
    bwd_time: float | None
    sparsity: float | None = None


@dataclass(frozen=True)
class Experiment:
    config: ExperimentConfig
    results: dict[str, ExperimentResults]  # backend -> ExperimentResults

    def asdict(self):
        dict1 = self.config.asdict()
        dict2 = self.results
        return {**dict1, **dict2}
```

- **EN:** Important local symbols in this block include ExperimentResults, Experiment, asdict.
- **CN:** 该代码块中的重要局部符号包括 ExperimentResults、Experiment、asdict。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 198-215 / 第 198-215 行

```python


def generate_inputs(
    batch_size: int,
    q_heads: int,
    q_sequence_length: int,
    kv_heads: int,
    kv_sequence_length: int,
    head_dim: int,
    dtype: torch.dtype,
    device: torch.device,
    requires_grad: bool,
    nested_tensors: bool = False,
):
    torch.manual_seed(0)
    q_shape = (batch_size, q_sequence_length, q_heads * head_dim)
    kv_shape = (batch_size, kv_sequence_length, kv_heads * head_dim)

```

- **EN:** Important local symbols in this block include generate_inputs.
- **CN:** 该代码块中的重要局部符号包括 generate_inputs。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 216-245 / 第 216-245 行

```python
    if q_heads % kv_heads != 0:
        raise AssertionError(
            f"q_heads ({q_heads}) must be divisible by kv_heads ({kv_heads})"
        )

    make_q = partial(
        torch.rand, q_shape, device=device, dtype=dtype, requires_grad=requires_grad
    )
    make_kv = partial(
        torch.rand, kv_shape, device=device, dtype=dtype, requires_grad=requires_grad
    )

    if nested_tensors:
        query = (
            make_q()
            .view(1, q_sequence_length * batch_size, q_heads, head_dim)
            .transpose(1, 2)
        )
        key = (
            make_kv()
            .view(1, batch_size * kv_sequence_length, kv_heads, head_dim)
            .transpose(1, 2)
        )
        value = (
            make_kv()
            .view(1, batch_size * kv_sequence_length, kv_heads, head_dim)
            .transpose(1, 2)
        )
    else:
        query = (
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 246-260 / 第 246-260 行

```python
            make_q()
            .view(batch_size, q_sequence_length, q_heads, head_dim)
            .transpose(1, 2)
        )
        key = (
            make_kv()
            .view(batch_size, kv_sequence_length, kv_heads, head_dim)
            .transpose(1, 2)
        )
        value = (
            make_kv()
            .view(batch_size, kv_sequence_length, kv_heads, head_dim)
            .transpose(1, 2)
        )
    return query, key, value
```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 261-276 / 第 261-276 行

```python


def generate_jagged_inputs(
    shape: tuple[int, ...],
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    offsets: torch.Tensor,
):
    B, Hq, M, Hkv, N, D = shape

    def offsets_to_lengths(
        offsets: torch.Tensor, device: str | torch.device
    ) -> torch.tensor:
        """Converts a list of offsets to a list of lengths. Reverse op of attn_gym.masks.document_mask.length_to_offsets

```

- **EN:** Important local symbols in this block include generate_jagged_inputs, offsets_to_lengths.
- **CN:** 该代码块中的重要局部符号包括 generate_jagged_inputs、offsets_to_lengths。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 277-293 / 第 277-293 行

```python
        Args:
            offsets: A 1D tensor of offsets
            device: The device to place the output tensor on
        """
        lengths = offsets[1:] - offsets[:-1]
        return lengths

    flatten_q = query.transpose(1, 2).flatten(start_dim=0, end_dim=1)
    flatten_k = key.transpose(1, 2).flatten(start_dim=0, end_dim=1)
    flatten_v = value.transpose(1, 2).flatten(start_dim=0, end_dim=1)

    q_list = [
        flatten_q[offsets[i] : offsets[i + 1]].clone().detach().to(query.dtype)
        for i in range(len(offsets) - 1)
    ]
    q = torch.nested.as_nested_tensor(q_list, device=query.device)

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 294-308 / 第 294-308 行

```python
    k_list = [
        flatten_k[offsets[i] : offsets[i + 1]].clone().detach().to(key.dtype)
        for i in range(len(offsets) - 1)
    ]
    k = torch.nested.as_nested_tensor(k_list, device=key.device)
    v_list = [
        flatten_v[offsets[i] : offsets[i + 1]].clone().detach().to(value.dtype)
        for i in range(len(offsets) - 1)
    ]
    v = torch.nested.as_nested_tensor(v_list, device=value.device)

    return q, k, v


def query_key_value_clones(
```

- **EN:** Important local symbols in this block include query_key_value_clones.
- **CN:** 该代码块中的重要局部符号包括 query_key_value_clones。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 309-324 / 第 309-324 行

```python
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    dtype: torch.dtype = None,
):
    """Clones the query, key, and value tensors and moves them to the specified dtype."""
    if dtype is None:
        dtype = query.dtype
    query_ref = query.clone().detach().to(dtype).requires_grad_(query.requires_grad)
    key_ref = key.clone().detach().to(dtype).requires_grad_(key.requires_grad)
    value_ref = value.clone().detach().to(dtype).requires_grad_(value.requires_grad)
    return query_ref, key_ref, value_ref


@safe_backend("SDPA")
def run_single_backend_sdpa(
```

- **EN:** Important local symbols in this block include run_single_backend_sdpa.
- **CN:** 该代码块中的重要局部符号包括 run_single_backend_sdpa。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 325-342 / 第 325-342 行

```python
    config: ExperimentConfig,
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    out_compile: torch.Tensor,
    score_mod: Callable | None,
    block_mask: BlockMask | None,
    mask_kwargs,
    backend: str,
) -> ExperimentResults:
    backend_context = get_backend_context(backend)
    with backend_context:
        _device = torch.device("cuda")

        eager_sdpa = generate_eager_sdpa(
            config.attn_type, config.shape, config.dtype, block_mask, score_mod
        )

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 343-360 / 第 343-360 行

```python
        if config.attn_type == "document_mask":
            q_eager, k_eager, v_eager = generate_jagged_inputs(
                config.shape, query, key, value, **mask_kwargs
            )
            q_eager = q_eager.transpose(1, 2).requires_grad_(query.requires_grad)
            k_eager = k_eager.transpose(1, 2).requires_grad_(key.requires_grad)
            v_eager = v_eager.transpose(1, 2).requires_grad_(value.requires_grad)
        else:
            q_eager, k_eager, v_eager = query_key_value_clones(query, key, value)

        if eager_sdpa:
            try:
                out_eager = eager_sdpa(query=q_eager, key=k_eager, value=v_eager)
            except RuntimeError as e:
                print(
                    f"[SKIP] SDPA Backend {backend} for shape {config.shape}. \n\t\t\tError encountered: {e} "
                )
                return ExperimentResults(
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 361-377 / 第 361-377 行

```python
                    fwd_time=float("nan"),
                    bwd_time=float("nan") if config.calculate_bwd_time else None,
                )
            if config.attn_type in ["document_mask"]:
                flatten_o_eager = torch.cat(torch.unbind(out_eager.transpose(1, 2)))
                flatten_o_compile = out_compile.transpose(1, 2).flatten(
                    start_dim=0, end_dim=1
                )
                torch.testing.assert_close(
                    flatten_o_eager, flatten_o_compile, atol=1e-2, rtol=1e-2
                )
            elif not (
                config.attn_type in ["rel", "alibi"]
                and config.dtype in [torch.float16, torch.bfloat16]
            ):  # rel has accuracy issue with 16bit floats
                torch.testing.assert_close(out_eager, out_compile, atol=1e-2, rtol=1e-2)

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 378-394 / 第 378-394 行

```python
        if eager_sdpa:
            forward_eager_time = benchmark_torch_function_in_microseconds(
                eager_sdpa, query=q_eager, key=k_eager, value=v_eager
            )
        else:
            forward_eager_time = float("nan")

        if config.calculate_bwd_time:
            # TODO: debug backward pass for njt
            if eager_sdpa and config.attn_type != "document_mask":
                d_out = torch.randn_like(out_eager.transpose(1, 2)).transpose(1, 2)
                backward_eager_time = benchmark_torch_function_in_microseconds(
                    out_eager.backward, d_out, retain_graph=True
                )
            else:
                backward_eager_time = float("nan")

```

- **EN:** Tensor construction or buffer wrapping happens here, so input layout and data preparation matter to the surrounding logic.
- **CN:** 这里发生了张量构造或缓冲区封装，因此输入布局与数据准备对周边逻辑很重要。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 395-418 / 第 395-418 行

```python
            return ExperimentResults(
                fwd_time=forward_eager_time,
                bwd_time=backward_eager_time,
            )
        else:
            return ExperimentResults(
                fwd_time=forward_eager_time,
                bwd_time=None,
            )


@safe_backend("FlashAttention")
def run_single_backend_FA(
    config: ExperimentConfig,
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    out_compile: torch.Tensor,
    score_mod: Callable | None,
    block_mask: BlockMask | None,
    mask_kwargs,
    backend: str,
) -> ExperimentResults:
    if backend not in ["fav3", "fakv"]:
```

- **EN:** Important local symbols in this block include run_single_backend_FA.
- **CN:** 该代码块中的重要局部符号包括 run_single_backend_FA。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 419-434 / 第 419-434 行

```python
        raise AssertionError(f"backend must be 'fav3' or 'fakv', got {backend}")
    # Generate callable for specific backend.
    if backend in ["fav3"]:
        FA = generate_FA_callable(
            config.attn_type, config.shape, config.dtype, backend, **mask_kwargs
        )
    elif backend == "fakv":
        FA = generate_FD_callable(config.attn_type, config.shape, config.dtype)

    q_FA, k_FA, v_FA = query_key_value_clones(query, key, value)
    q_FA, k_FA, v_FA = q_FA.transpose(1, 2), k_FA.transpose(1, 2), v_FA.transpose(1, 2)
    if config.attn_type == "document_mask":
        q_FA = q_FA.flatten(start_dim=0, end_dim=1)
        k_FA = k_FA.flatten(start_dim=0, end_dim=1)
        v_FA = v_FA.flatten(start_dim=0, end_dim=1)

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 435-449 / 第 435-449 行

```python
    if FA:
        out_FA = FA(q=q_FA, k=k_FA, v=v_FA)
        if config.attn_type in ["document_mask"]:
            out_FA_updated = out_FA[None, :, :, :]
        else:
            out_FA_updated = out_FA

        if not (
            config.attn_type in ["rel", "alibi"]
            and config.dtype in [torch.float16, torch.bfloat16]
        ):
            torch.testing.assert_close(
                out_FA_updated, out_compile.transpose(1, 2), atol=1e-2, rtol=1e-2
            )

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 450-465 / 第 450-465 行

```python
    if FA:
        forward_FA_time = benchmark_torch_function_in_microseconds(
            FA, q=q_FA, k=k_FA, v=v_FA
        )
    else:
        forward_FA_time = float("nan")

    if config.calculate_bwd_time:
        if FA:
            d_out = torch.randn_like(out_FA)
            backward_FA_time = benchmark_torch_function_in_microseconds(
                out_FA.backward, d_out, retain_graph=True
            )
        else:
            backward_FA_time = float("nan")

```

- **EN:** Tensor construction or buffer wrapping happens here, so input layout and data preparation matter to the surrounding logic.
- **CN:** 这里发生了张量构造或缓冲区封装，因此输入布局与数据准备对周边逻辑很重要。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 466-494 / 第 466-494 行

```python
    return ExperimentResults(
        fwd_time=forward_FA_time,
        bwd_time=backward_FA_time if config.calculate_bwd_time else None,
    )


@safe_backend("flex_attention", return_dict=True)
def run_single_experiment(
    config: ExperimentConfig,
    dynamic=False,
) -> dict[str, ExperimentResults]:
    device = torch.device("cuda")
    batch_size, q_heads, q_seq_len, kv_heads, kv_seq_len, head_dim = config.shape
    query, key, value = generate_inputs(
        batch_size,
        q_heads,
        q_seq_len,
        kv_heads,
        kv_seq_len,
        head_dim,
        config.dtype,
        device,
        requires_grad=config.calculate_bwd_time,
        nested_tensors=config.attn_type == "document_mask",
    )
    score_mod = generate_score_mod(config.attn_type, config.shape)
    block_mask, mask_kwargs = generate_block_mask(config.attn_type, config.shape)
    kernel_options = get_kernel_options(config.attn_type, config.shape)

```

- **EN:** Important local symbols in this block include run_single_experiment.
- **CN:** 该代码块中的重要局部符号包括 run_single_experiment。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 495-511 / 第 495-511 行

```python
    if config.max_autotune:
        compiled_sdpa = torch.compile(
            flex_attention, dynamic=dynamic, mode="max-autotune-no-cudagraphs"
        )
    else:
        compiled_sdpa = torch.compile(flex_attention, dynamic=dynamic)

    out_compile = compiled_sdpa(
        query=query,
        key=key,
        value=value,
        score_mod=score_mod,
        block_mask=block_mask,
        enable_gqa=True,
        kernel_options=kernel_options,
    )

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 512-541 / 第 512-541 行

```python
    forward_compiled_time = benchmark_torch_function_in_microseconds(
        compiled_sdpa,
        query,
        key,
        value,
        score_mod=score_mod,
        block_mask=block_mask,
        enable_gqa=True,
        kernel_options=kernel_options,
    )

    results = {}
    for backend in config.backends:
        if backend in ["fav3", "fakv"]:
            results[backend] = run_single_backend_FA(
                config,
                query,
                key,
                value,
                out_compile,
                score_mod,
                block_mask,
                mask_kwargs,
                backend,
            )
        else:  # sdpa (also supports fav2)
            results[backend] = run_single_backend_sdpa(
                config,
                query,
                key,
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 542-557 / 第 542-557 行

```python
                value,
                out_compile,
                score_mod,
                block_mask,
                mask_kwargs,
                backend,
            )

    if config.calculate_bwd_time:
        d_out = torch.randn_like(out_compile)
        backward_compile_time = benchmark_torch_function_in_microseconds(
            out_compile.backward, d_out, retain_graph=True
        )
    sparsity = block_mask.sparsity() / 100.0 if block_mask is not None else 0.0
    sparsity = sparsity if config.attn_type != "document_mask" else 0.5

```

- **EN:** Tensor construction or buffer wrapping happens here, so input layout and data preparation matter to the surrounding logic.
- **CN:** 这里发生了张量构造或缓冲区封装，因此输入布局与数据准备对周边逻辑很重要。

### Lines 558-573 / 第 558-573 行

```python
    results["flex"] = ExperimentResults(
        fwd_time=forward_compiled_time,
        bwd_time=backward_compile_time if config.calculate_bwd_time else None,
        sparsity=sparsity,
    )

    return results


def calculate_speedup(
    results: ExperimentResults, baseline_results: ExperimentResults, type: str
) -> float:
    if type == "fwd":
        return baseline_results.fwd_time / results.fwd_time
    elif type == "bwd":
        if results.bwd_time is None:
```

- **EN:** Important local symbols in this block include calculate_speedup.
- **CN:** 该代码块中的重要局部符号包括 calculate_speedup。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 574-603 / 第 574-603 行

```python
            raise AssertionError("results.bwd_time must not be None for bwd speedup")
        return baseline_results.bwd_time / results.bwd_time
    else:
        raise ValueError(f"Invalid type {type}")


def calculate_bandwidth(
    config: ExperimentConfig, results: ExperimentResults, type: str
) -> float:
    B, Hq, M, Hkv, N, D = config.shape
    sparsity = results.sparsity if M == 1 else 0.0
    if type == "fwd":
        batch_size, q_heads, q_seq_len, kv_heads, kv_seq_len, head_dim = config.shape
        query_size = (
            batch_size
            * q_heads
            * q_seq_len
            * head_dim
            * torch.finfo(config.dtype).bits
            / 8
        )
        kv_size = (
            batch_size
            * kv_heads
            * kv_seq_len
            * head_dim
            * torch.finfo(config.dtype).bits
            / 8
            * 2
        )
```

- **EN:** Important local symbols in this block include calculate_bandwidth.
- **CN:** 该代码块中的重要局部符号包括 calculate_bandwidth。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 604-622 / 第 604-622 行

```python
        output_size = query_size
        total_size = (
            query_size + kv_size * (1 - sparsity) + output_size
        ) / 1e9  # In GB
        time_in_seconds = results.fwd_time / 1e6
        return total_size / time_in_seconds / 1e3
    else:
        raise ValueError(f"Invalid type {type}")


def calculate_tflops(config: ExperimentConfig, results: ExperimentResults) -> float:
    (B, Hq, M, Hkv, N, D) = config.shape
    qk_flops = M * N * D * 2
    softmax_flops = M * N * 2  # Not counting online softmax overhead
    o_flops = M * D * N * 2
    # Not counting split k overhead
    sparsity = results.sparsity if results.sparsity is not None else 0.0
    total_flops = B * Hq * (qk_flops + softmax_flops + o_flops) * (1 - sparsity)
    return total_flops / results.fwd_time / 1e6  # in TFLOPs/
```

- **EN:** Important local symbols in this block include calculate_tflops.
- **CN:** 该代码块中的重要局部符号包括 calculate_tflops。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 623-638 / 第 623-638 行

```python


def get_average_speedups(results: list[Experiment], type: str, backend: str):
    # Calculate speedups
    speedups = [
        calculate_speedup(r.results["flex"], r.results[backend], type) for r in results
    ]

    # Find indices of max and min speedups
    max_speedup_index = np.nanargmax(speedups)
    min_speedup_index = np.nanargmin(speedups)

    # Get the config dictionaries
    max_config_dict = results[max_speedup_index].config.asdict()
    min_config_dict = results[min_speedup_index].config.asdict()

```

- **EN:** Important local symbols in this block include get_average_speedups.
- **CN:** 该代码块中的重要局部符号包括 get_average_speedups。

### Lines 639-653 / 第 639-653 行

```python
    # Create table data
    table_data = [
        {
            "Type": "Average",
            "Speedup": np.nanmean(speedups),
            **dict.fromkeys(max_config_dict),
        },
        {"Type": "Max", "Speedup": speedups[max_speedup_index], **max_config_dict},
        {"Type": "Min", "Speedup": speedups[min_speedup_index], **min_config_dict},
    ]

    return table_data


def print_results(results: list[Experiment], save_path: str | None = None):
```

- **EN:** Important local symbols in this block include print_results.
- **CN:** 该代码块中的重要局部符号包括 print_results。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 654-670 / 第 654-670 行

```python
    table_data = defaultdict(list)
    for experiment in results:
        backends = experiment.config.backends + ["flex"]
        for key, value in experiment.asdict().items():
            if key in backends:
                if value.fwd_time:
                    table_data[f"fwd_{key}"].append(float(value.fwd_time))
                if value.bwd_time:
                    table_data[f"bwd_{key}"].append(float(value.bwd_time))
            else:
                table_data[key].append(value)

    # Calculate speedups
    for backend in results[0].config.backends:
        fwd_speedups = [
            calculate_speedup(r.results["flex"], r.results[backend], type="fwd")
            for r in results
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 671-686 / 第 671-686 行

```python
        ]
        table_data[f"fwd_speedup_flex_over_{backend}"] = fwd_speedups

    if results[0].config.calculate_bwd_time:
        for backend in results[0].config.backends:
            bwd_speedups = [
                calculate_speedup(r.results["flex"], r.results[backend], type="bwd")
                for r in results
            ]
            table_data[f"bwd_speedup_flex_over_{backend}"] = bwd_speedups

    # Calculate mem + computational throughput
    if results[0].config.cal_bandwidth:
        fwd_bandwidth = [
            calculate_bandwidth(r.config, r.results["flex"], type="fwd")
            for r in results
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 687-702 / 第 687-702 行

```python
        ]
        table_data["fwd_mem_bw (TB/s)"] = fwd_bandwidth
        fwd_tflops = [calculate_tflops(r.config, r.results["flex"]) for r in results]
        table_data["TFlops/s"] = fwd_tflops

    print(tabulate(table_data, headers="keys", tablefmt="github", floatfmt=".3f"))

    for backend in results[0].config.backends:
        if np.isnan(table_data[f"fwd_speedup_flex_over_{backend}"]).all():
            continue
        print("\n")
        print(f"FWD Speedup of Flex over {backend}".center(125, "="))
        print("\n")
        average_data = get_average_speedups(results, type="fwd", backend=backend)
        print(tabulate(average_data, headers="keys", tablefmt="github", floatfmt=".3f"))

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 703-718 / 第 703-718 行

```python
        if results[0].config.calculate_bwd_time:
            print("\n")
            print(f"BWD Speedup of Flex over {backend}".center(125, "="))
            print("\n")
            average_data = get_average_speedups(results, type="bwd", backend=backend)
            print(
                tabulate(
                    average_data, headers="keys", tablefmt="github", floatfmt=".3f"
                )
            )

    if save_path is not None:
        with open(save_path, "w", newline="") as csvfile:
            writer = csv.DictWriter(csvfile, fieldnames=table_data.keys())
            writer.writeheader()
            for i in range(len(next(iter(table_data.values())))):
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 719-733 / 第 719-733 行

```python
                row = {k: v[i] for k, v in table_data.items()}
                writer.writerow(row)
        print(f"\nResults saved to {save_path}")


# Generate score_mods and BlockMasks
softcap_value = 50
dropout_p = 0.0


def generate_score_mod(attn_type: str, shape: tuple[int, ...]) -> Callable | None:
    B, Hq, M, Hkv, N, D = shape
    is_decoding = M == 1
    from attn_gym.mods import generate_alibi_bias, generate_tanh_softcap

```

- **EN:** Important local symbols in this block include generate_score_mod.
- **CN:** 该代码块中的重要局部符号包括 generate_score_mod。

### Lines 734-751 / 第 734-751 行

```python
    def relative_bias(score, b, h, m, n):
        return score + (m - n)

    def head_bias(score, b, h, m, n):
        return score + 2 * h

    function_dict = {
        "noop": None,
        "causal": None,
        "rel": relative_bias,
        "head_bias": head_bias,
        "alibi": generate_alibi_bias(Hq),
        "sliding_window": None,
        "document_mask": None,
        "prefix_lm": None,
        "softcap": generate_tanh_softcap(softcap_value, approx=True),
    }

```

- **EN:** Important local symbols in this block include relative_bias, head_bias.
- **CN:** 该代码块中的重要局部符号包括 relative_bias、head_bias。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 752-766 / 第 752-766 行

```python
    score_mod = function_dict[attn_type]
    is_decoding = M == 1
    if is_decoding and score_mod:
        offset = torch.tensor(N // 2).to("cuda")

        def score_mod_w_offset(score, b, h, m, n):
            return score_mod(score, b, h, m + offset, n)

        new_score_mod = score_mod_w_offset
    else:
        new_score_mod = score_mod

    return new_score_mod


```

- **EN:** Important local symbols in this block include score_mod_w_offset.
- **CN:** 该代码块中的重要局部符号包括 score_mod_w_offset。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 767-781 / 第 767-781 行

```python
sliding_window_size = 512
prefix_length = 512


def generate_block_mask(attn_type: str, shape: tuple[int, ...]):
    B, Hq, M, Hkv, N, D = shape
    is_decoding = M == 1

    def causal(b, h, m, n):
        return m >= n

    def gen_offset(off):
        def offset(b, h, m, n):
            return m + off >= n

```

- **EN:** Important local symbols in this block include generate_block_mask, causal, gen_offset, offset.
- **CN:** 该代码块中的重要局部符号包括 generate_block_mask、causal、gen_offset、offset。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 782-797 / 第 782-797 行

```python
        return offset

    from attn_gym.masks import (
        generate_doc_mask_mod,
        generate_prefix_lm_mask,
        generate_sliding_window,
    )
    from attn_gym.masks.document_mask import length_to_offsets

    def generate_random_lengths(total_length, num_documents):
        # Initialize all lengths to 1 to ensure each document has at least one token
        lengths = [1] * num_documents
        remaining_length = total_length - num_documents

        # Randomly distribute the remaining length
        for _ in range(remaining_length):
```

- **EN:** Important local symbols in this block include generate_random_lengths.
- **CN:** 该代码块中的重要局部符号包括 generate_random_lengths。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 798-812 / 第 798-812 行

```python
            index = random.randint(0, num_documents - 1)
            lengths[index] += 1
        return lengths

    mask_mod_kwargs = {}

    if attn_type == "document_mask" and is_decoding:
        raise AssertionError(
            "document_mask attention type is not supported in decoding mode"
        )
    if attn_type == "document_mask":
        random.seed(0)
        lengths = generate_random_lengths(N * B, B)
        mask_mod_kwargs = dict(offsets=length_to_offsets(lengths, "cuda"))

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 813-827 / 第 813-827 行

```python
    mask_mod_dict = {
        "noop": None,
        "causal": causal,
        "rel": None,
        "head_bias": None,
        "alibi": causal,
        "sliding_window": generate_sliding_window(sliding_window_size),
        "document_mask": partial(generate_doc_mask_mod, mask_mod=causal),
        "prefix_lm": generate_prefix_lm_mask(prefix_length),
        "softcap": causal,
    }

    mask_mod = mask_mod_dict[attn_type]

    if mask_mod_kwargs:
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 828-842 / 第 828-842 行

```python
        mask_mod = mask_mod(**mask_mod_kwargs)

    if is_decoding and mask_mod:
        cached_seq_len = torch.tensor(N // 2).to("cuda")

        def decoding_w_cached_seq_len(b, h, m, n):
            return mask_mod(b, h, m + cached_seq_len, n)

        new_mask_mod = decoding_w_cached_seq_len
    else:
        new_mask_mod = mask_mod

    mask_shape = (1, 1, M, N) if attn_type != "document_mask" else (1, 1, M * B, N * B)
    compiled_block_mask = torch.compile(create_block_mask)
    if new_mask_mod:
```

- **EN:** Important local symbols in this block include decoding_w_cached_seq_len.
- **CN:** 该代码块中的重要局部符号包括 decoding_w_cached_seq_len。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 843-869 / 第 843-869 行

```python
        block_mask = compiled_block_mask(new_mask_mod, *mask_shape, "cuda")
    else:
        block_mask = compiled_block_mask(noop_mask, *mask_shape, "cuda")
    return block_mask, mask_mod_kwargs


def get_kernel_options(attn_type: str, shape: tuple[int, ...]):
    B, Hq, M, Hkv, N, D = shape
    is_decoding = M == 1
    kernel_opt_training_dict = {
        "noop": None,
        "causal": None,
        "rel": None,
        "head_bias": None,
        "alibi": None,
        "sliding_window": None,
        "document_mask": {
            "BLOCK_N": 32,
            "BLOCK_M": 128,
            "fwd_num_warps": 8,
            "fwd_num_stages": 4,
            "BLOCK_M1": 64,
            "BLOCK_N1": 64,
            "BLOCK_M2": 64,
            "BLOCK_N2": 64,
        }
        if torch.cuda.get_device_capability() >= (8, 0) and D <= 128
```

- **EN:** Important local symbols in this block include get_kernel_options.
- **CN:** 该代码块中的重要局部符号包括 get_kernel_options。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 870-895 / 第 870-895 行

```python
        else None,
        "prefix_lm": None,
        "softcap": None,
    }

    def get_default_split_k(B: int, H: int, Mk: int) -> int:
        num_SM = torch.cuda.get_device_properties("cuda").multi_processor_count
        """Heuristic for the number of splits from xformer"""
        bh = max(B * H, 1)  # NOTE: Handle B*h=0 case
        split_k = num_SM // bh * 2  # Each SM should at least get one block.
        split_k = max(split_k, 1)

        return split_k

    kernel_opt_decoding_dict = {
        "noop": None,
        "causal": {"SPLIT_KV": get_default_split_k(B, Hkv, N) * 2},
        "rel": None,
        "head_bias": None,
        "alibi": {"SPLIT_KV": get_default_split_k(B, Hkv, N) * 2},
        "sliding_window": None,
        "document_mask": None,
        "prefix_lm": None,
        "softcap": {"SPLIT_KV": get_default_split_k(B, Hkv, N) * 2},
    }

```

- **EN:** Important local symbols in this block include get_default_split_k.
- **CN:** 该代码块中的重要局部符号包括 get_default_split_k。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 896-925 / 第 896-925 行

```python
    return (
        kernel_opt_decoding_dict[attn_type]
        if is_decoding
        else kernel_opt_training_dict[attn_type]
    )


# Setup Backend


def get_backend_context(backend: str):
    """
    Returns a context manager for the specified backend.
    Args:
        backend (str): The name of the backend to use.
                       Valid options are 'math', 'efficient', 'cudnn', 'fav2', 'fav3', 'fakv', 'og-eager'.
    Returns:
        A context manager for the specified backend.
    Raises:
        ValueError: If an invalid backend is specified.
    """
    backends = {
        "fav2": sdpa_kernel(SDPBackend.FLASH_ATTENTION),
        "cudnn": sdpa_kernel(SDPBackend.CUDNN_ATTENTION),
        "math": sdpa_kernel(SDPBackend.MATH),
        "efficient": sdpa_kernel(SDPBackend.EFFICIENT_ATTENTION),
        "fav3": nullcontext(),
        "fakv": nullcontext(),
        "og-eager": nullcontext(),
    }
```

- **EN:** Important local symbols in this block include get_backend_context.
- **CN:** 该代码块中的重要局部符号包括 get_backend_context。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 926-940 / 第 926-940 行

```python

    if backend not in backends:
        raise ValueError(
            f"Unknown backend: {backend}. Valid options are: {', '.join(backends.keys())}"
        )

    return backends[backend]


def generate_FA_callable(
    attn_type: str, shape: tuple[int, ...], dtype: torch.dtype, backend: str, **kwargs
) -> Callable | None:
    if dtype not in [torch.float16, torch.bfloat16]:
        return None
    if backend == "fav3":
```

- **EN:** Important local symbols in this block include generate_FA_callable.
- **CN:** 该代码块中的重要局部符号包括 generate_FA_callable。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 941-956 / 第 941-956 行

```python
        try:
            from flash_attn.flash_attn_interface import (
                flash_attn_func,
                flash_attn_varlen_func,
            )
        except ImportError:
            print(
                "Flash attention 3 is not installed. Please install it to run fav3 backend. "
            )
            raise
    else:
        print("Unknown backend " + backend)
        return None

    B, Hq, M, Hkv, N, D = shape

```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 957-971 / 第 957-971 行

```python
    FA_kwargs = {}
    if attn_type == "alibi":
        h = torch.arange(Hq, dtype=torch.float32, device="cuda")
        alibi_slopes = torch.exp2(-((h + 1) * 8.0 / Hq))
        FA_kwargs = dict(alibi_slopes=alibi_slopes)
    elif attn_type == "document_mask":
        FA_kwargs["cu_seqlens_q"] = kwargs["offsets"].to(torch.int32)
        FA_kwargs["cu_seqlens_k"] = kwargs["offsets"].to(torch.int32)

        def offsets_to_lengths(
            offsets: torch.Tensor, device: str | torch.device
        ) -> torch.tensor:
            lengths = offsets[1:] - offsets[:-1]
            return lengths

```

- **EN:** Important local symbols in this block include offsets_to_lengths.
- **CN:** 该代码块中的重要局部符号包括 offsets_to_lengths。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 972-990 / 第 972-990 行

```python
        lengths = offsets_to_lengths(kwargs["offsets"], "cpu")
        max_length = torch.max(lengths)
        FA_kwargs["max_seqlen_q"] = max_length
        FA_kwargs["max_seqlen_k"] = max_length

    FA_dict = {
        "noop": partial(flash_attn_func, causal=False),
        "causal": partial(flash_attn_func, causal=True),
        "rel": None,
        "head_bias": None,
        "alibi": partial(flash_attn_func, causal=True, **FA_kwargs),
        "sliding_window": partial(
            flash_attn_func, window_size=(sliding_window_size, 0), causal=True
        ),
        "document_mask": partial(flash_attn_varlen_func, causal=True, **FA_kwargs),
        "prefix_lm": None,
        "softcap": partial(flash_attn_func, softcap=softcap_value, causal=True),
    }

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 991-1006 / 第 991-1006 行

```python
    return FA_dict[attn_type]


def generate_FD_callable(
    attn_type: str, shape: tuple[int, ...], dtype: torch.dtype
) -> Callable | None:
    if dtype not in [torch.float16, torch.bfloat16]:
        return None
    try:
        from flash_attn import flash_attn_with_kvcache
    except ImportError:
        print(
            "Flash attention 2 is not installed. Please install it to run fakv backend. "
        )
        raise

```

- **EN:** Important local symbols in this block include generate_FD_callable.
- **CN:** 该代码块中的重要局部符号包括 generate_FD_callable。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 1007-1036 / 第 1007-1036 行

```python
    B, Hq, M, Hkv, N, D = shape

    if M != 1:
        raise AssertionError(f"M must be 1 for FD callable, got {M}")

    def flash_attn_with_kvcache_renamed(q, k, v, **kwargs):
        return flash_attn_with_kvcache(q, k_cache=k, v_cache=v, **kwargs)

    FA_kwargs = {}
    if attn_type == "alibi":
        h = torch.arange(Hq, dtype=torch.float32, device="cuda")
        alibi_slopes = torch.exp2(-((h + 1) * 8.0 / Hq))
        FA_kwargs = dict(alibi_slopes=alibi_slopes)

    FD_dict = {
        "noop": partial(flash_attn_with_kvcache_renamed, causal=False),
        "causal": partial(flash_attn_with_kvcache_renamed, cache_seqlens=N // 2),
        "rel": None,
        "head_bias": None,
        "alibi": partial(
            flash_attn_with_kvcache_renamed, cache_seqlens=N // 2, **FA_kwargs
        ),
        "sliding_window": partial(
            flash_attn_with_kvcache_renamed,
            cache_seqlens=N // 2,
            window_size=(sliding_window_size, 0),
        ),
        "document_mask": None,
        "prefix_lm": None,
        "softcap": partial(flash_attn_with_kvcache_renamed, softcap=softcap_value),
```

- **EN:** Important local symbols in this block include flash_attn_with_kvcache_renamed.
- **CN:** 该代码块中的重要局部符号包括 flash_attn_with_kvcache_renamed。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 1037-1055 / 第 1037-1055 行

```python
    }

    return FD_dict[attn_type]


def generate_attn_mask_linear_score_mod(
    shape: tuple[int, ...],
    block_mask: BlockMask,
    score_mod: Callable,
    dtype: torch.dtype,
):
    B, Hq, M, N = shape
    if block_mask is None and score_mod is None:
        return None
    b = torch.arange(B, dtype=int, device="cuda")
    h = torch.arange(Hq, dtype=int, device="cuda")
    m = torch.arange(M, dtype=int, device="cuda")
    n = torch.arange(N, dtype=int, device="cuda")

```

- **EN:** Important local symbols in this block include generate_attn_mask_linear_score_mod.
- **CN:** 该代码块中的重要局部符号包括 generate_attn_mask_linear_score_mod。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 1056-1079 / 第 1056-1079 行

```python
    score = torch.zeros(B, Hq, M, N, dtype=dtype, device="cuda")
    bias = score_mod(
        score,
        b[:, None, None, None],
        h[None, :, None, None],
        m[None, None, :, None],
        n[None, None, None, :],
    )
    bool_mask = create_mask(block_mask.mask_mod, B, Hq, M, N, device="cuda")
    attn_mask = bias.masked_fill(bool_mask.logical_not(), float("-inf"))
    return attn_mask.to(dtype)


def generate_eager_sdpa(
    attn_type: str,
    shape: tuple[int, ...],
    dtype: torch.dtype,
    block_mask: BlockMask,
    score_mod: Callable | None = None,
    **kwargs,
) -> Callable | None:
    B, Hq, M, Hkv, N, D = shape
    is_decoding = M == 1
    if attn_type == "sliding_window" or attn_type == "prefix_lm":
```

- **EN:** Important local symbols in this block include generate_eager_sdpa.
- **CN:** 该代码块中的重要局部符号包括 generate_eager_sdpa。
- **EN:** Tensor construction or buffer wrapping happens here, so input layout and data preparation matter to the surrounding logic.
- **CN:** 这里发生了张量构造或缓冲区封装，因此输入布局与数据准备对周边逻辑很重要。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 1080-1094 / 第 1080-1094 行

```python
        attn_mask = create_mask(block_mask.mask_mod, 1, 1, M, N, device="cuda")
    elif attn_type == "rel":
        attn_mask = generate_attn_mask_linear_score_mod(
            [1, 1, M, N], block_mask, score_mod, dtype
        )
    elif attn_type == "head_bias":
        h = torch.arange(Hq, dtype=int, device="cuda")
        attn_mask = (2 * h[None, :, None, None]).broadcast_to(1, Hq, M, N).to(dtype)
    elif attn_type == "alibi":
        attn_mask = generate_attn_mask_linear_score_mod(
            [1, Hq, M, N], block_mask, score_mod, dtype
        )
    else:
        attn_mask = None

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 1095-1117 / 第 1095-1117 行

```python
    sdpa_dict = {
        "noop": partial(
            F.scaled_dot_product_attention, is_causal=False, enable_gqa=(Hq != Hkv)
        ),
        "causal": partial(
            F.scaled_dot_product_attention, is_causal=True, enable_gqa=(Hq != Hkv)
        ),
        "rel": partial(
            F.scaled_dot_product_attention, is_causal=False, enable_gqa=(Hq != Hkv)
        ),
        "head_bias": partial(
            F.scaled_dot_product_attention, is_causal=False, enable_gqa=(Hq != Hkv)
        ),
        "alibi": partial(
            F.scaled_dot_product_attention, is_causal=False, enable_gqa=(Hq != Hkv)
        ),
        "sliding_window": partial(
            F.scaled_dot_product_attention, is_causal=False, enable_gqa=(Hq != Hkv)
        ),
        "document_mask": partial(
            F.scaled_dot_product_attention, is_causal=True, enable_gqa=(Hq != Hkv)
        )
        if Hq == Hkv
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 1118-1133 / 第 1118-1133 行

```python
        else None,
        "prefix_lm": partial(
            F.scaled_dot_product_attention, is_causal=False, enable_gqa=(Hq != Hkv)
        ),
        "softcap": None,
    }

    if is_decoding and attn_type == "causal":
        attn_mask = create_mask(block_mask.mask_mod, 1, 1, M, N, device="cuda")
        sdpa_dict["causal"] = partial(
            F.scaled_dot_product_attention, is_causal=False, enable_gqa=(Hq != Hkv)
        )

    return (
        partial(sdpa_dict[attn_type], attn_mask=attn_mask)
        if sdpa_dict[attn_type]
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 1134-1152 / 第 1134-1152 行

```python
        else None
    )


def generate_experiment_configs(
    calculate_bwd: bool,
    dtype: torch.dtype,
    batch_sizes: list[int],
    num_heads: list[tuple[int, int]],
    seq_lens: list[int],
    head_dims: list[int],
    score_mods_str: list[str],
    decoding: bool,
    kv_cache_size: list[int],
    cal_bandwidth: bool,
    backends: list[str],
    max_autotune: bool,
) -> list[ExperimentConfig]:
    if calculate_bwd and decoding:
```

- **EN:** Important local symbols in this block include generate_experiment_configs.
- **CN:** 该代码块中的重要局部符号包括 generate_experiment_configs。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 1153-1177 / 第 1153-1177 行

```python
        raise AssertionError("Decoding does not support backward")

    if decoding:
        q_kv_seq_lens = [(1, i) for i in seq_lens]  # only testing query length == 1
    else:
        q_kv_seq_lens = [(i, i) for i in seq_lens]  # only testing q_len == kv_len
    dtypes = [dtype]

    all_configs = []
    for (
        bsz,
        (q_heads, kv_heads),
        (q_seq_len, kv_seq_len),
        head_dim,
        attn_type,
        dtype,
    ) in itertools.product(
        kv_cache_size if kv_cache_size else batch_sizes,
        num_heads,
        q_kv_seq_lens,
        head_dims,
        score_mods_str,
        dtypes,
    ):
        if kv_cache_size:
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Looping logic iterates over benchmark dimensions, shape metadata, or container elements.
- **CN:** 循环逻辑会遍历 benchmark 维度、形状元数据或容器元素。

### Lines 1178-1201 / 第 1178-1201 行

```python
            head_size_bytes = torch.finfo(dtype).bits / 8 * head_dim
            bsz = int(
                (bsz * 1024 * 1024) // (kv_heads * kv_seq_len * head_size_bytes * 2)
            )
            if bsz <= 0:
                continue

        if q_heads % kv_heads != 0:
            raise AssertionError(
                f"q_heads ({q_heads}) must be divisible by kv_heads ({kv_heads})"
            )

        all_configs.append(
            ExperimentConfig(
                shape=(bsz, q_heads, q_seq_len, kv_heads, kv_seq_len, head_dim),
                attn_type=attn_type,
                dtype=dtype,
                calculate_bwd_time=calculate_bwd,
                cal_bandwidth=cal_bandwidth,
                backends=backends,
                max_autotune=max_autotune,
            )
        )

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 1202-1220 / 第 1202-1220 行

```python
    return all_configs


def _output_json_for_dashboard(
    experiments,
    output_file,
    benchmark_name="PyTorch operator microbenchmark",
):
    """
    Write the result into JSON format for PyTorch OSS dashboard.
    The JSON format is defined at
    https://github.com/pytorch/pytorch/wiki/How-to-integrate-with-PyTorch-OSS-benchmark-database

    Args:
        experiments: List of experiment results
        output_file: Path to output JSON file
        benchmark_name: Name of the benchmark
    """
    if not experiments:
```

- **EN:** Important local symbols in this block include _output_json_for_dashboard.
- **CN:** 该代码块中的重要局部符号包括 _output_json_for_dashboard。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 1221-1235 / 第 1221-1235 行

```python
        return

    import math
    import platform
    from dataclasses import asdict, dataclass
    from typing import Any

    # Prepare headers and records for JSON output
    records = []
    for experiment in experiments:
        config = experiment.config
        results_dict = (
            experiment.results
        )  # This is a dict: backend -> ExperimentResults

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 1236-1252 / 第 1236-1252 行

```python
        # Process each backend result
        for backend, results in results_dict.items():
            # Skip backends that were not run (NaN results)
            if math.isnan(results.fwd_time):
                continue

            # Extract data from experiment
            test_name = f"{backend}_{config.attn_type}_"
            input_config = f"shape: {config.shape}, dtype: {config.dtype}"

            # Determine mode based on backward pass
            mode = "training" if config.calculate_bwd_time else "inference"

            # Extract dtype
            dtype = (
                str(config.dtype).split(".")[1]
                if "." in str(config.dtype)
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 1253-1267 / 第 1253-1267 行

```python
                else str(config.dtype)
            )

            # Determine device
            device = "cuda"

            # Get device architecture
            device_arch = (
                torch.cuda.get_device_name(0)
                if device == "cuda"
                else platform.processor()
                if device == "cpu"
                else "unknown"
            )

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 1268-1282 / 第 1268-1282 行

```python
            # Create dataclasses for JSON structure
            @dataclass
            class BenchmarkInfo:
                name: str
                mode: str | None
                dtype: str
                extra_info: dict[str, Any]

            @dataclass
            class ModelInfo:
                name: str
                type: str
                origins: list[str]
                extra_info: dict[str, Any]

```

- **EN:** Important local symbols in this block include BenchmarkInfo, ModelInfo.
- **CN:** 该代码块中的重要局部符号包括 BenchmarkInfo、ModelInfo。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 1283-1297 / 第 1283-1297 行

```python
            @dataclass
            class MetricInfo:
                name: str
                unit: str
                benchmark_values: list[float]
                target_value: float | None

            @dataclass
            class BenchmarkRecord:
                benchmark: BenchmarkInfo
                model: ModelInfo
                metric: MetricInfo

            operator_name = backend_to_operator_name.get(backend, backend)

```

- **EN:** Important local symbols in this block include MetricInfo, BenchmarkRecord.
- **CN:** 该代码块中的重要局部符号包括 MetricInfo、BenchmarkRecord。

### Lines 1298-1327 / 第 1298-1327 行

```python
            # Benchmark extra info
            benchmark_extra_info = {
                "input_config": input_config,
                "device": device,
                "arch": device_arch,
                "operator_name": operator_name,
                "attn_type": config.attn_type,
                "shape": str(config.shape),
                "max_autotune": config.max_autotune,
            }
            # Add record for forward latency
            record_fwd_latency = BenchmarkRecord(
                benchmark=BenchmarkInfo(
                    name=benchmark_name,
                    mode=mode,
                    dtype=dtype,
                    extra_info=benchmark_extra_info,
                ),
                model=ModelInfo(
                    name=test_name + str(config.shape),
                    type="attention-benchmark",
                    origins=["pytorch"],
                    extra_info={
                        "operator_name": operator_name,
                        "attn_type": config.attn_type,
                    },
                ),
                metric=MetricInfo(
                    name="forward latency",
                    unit="us",
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 1328-1357 / 第 1328-1357 行

```python
                    benchmark_values=[results.fwd_time],
                    target_value=None,
                ),
            )
            records.append(asdict(record_fwd_latency))

            # Add record for forward memory bandwidth (if available)
            if config.cal_bandwidth:
                record_fwd_bandwidth = BenchmarkRecord(
                    benchmark=BenchmarkInfo(
                        name=benchmark_name,
                        mode=mode,
                        dtype=dtype,
                        extra_info=benchmark_extra_info,
                    ),
                    model=ModelInfo(
                        name=test_name + str(config.shape),
                        type="attention-benchmark",
                        origins=["pytorch"],
                        extra_info={
                            "operator_name": operator_name,
                        },
                    ),
                    metric=MetricInfo(
                        name="memory bandwidth",
                        unit="TB/s",
                        benchmark_values=[calculate_bandwidth(config, results, "fwd")],
                        target_value=None,
                    ),
                )
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 1358-1385 / 第 1358-1385 行

```python
                records.append(asdict(record_fwd_bandwidth))

            # Add record for forward TFLOPS (if available)
            if config.cal_bandwidth:
                record_fwd_tflops = BenchmarkRecord(
                    benchmark=BenchmarkInfo(
                        name=benchmark_name,
                        mode=mode,
                        dtype=dtype,
                        extra_info=benchmark_extra_info,
                    ),
                    model=ModelInfo(
                        name=test_name + str(config.shape),
                        type="attention-benchmark",
                        origins=["pytorch"],
                        extra_info={
                            "operator_name": operator_name,
                        },
                    ),
                    metric=MetricInfo(
                        name="tflops",
                        unit="TFLOPS/s",
                        benchmark_values=[calculate_tflops(config, results)],
                        target_value=None,
                    ),
                )
                records.append(asdict(record_fwd_tflops))

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 1386-1415 / 第 1386-1415 行

```python
            # Add record for backward latency (if available and not NaN)
            if (
                config.calculate_bwd_time
                and results.bwd_time is not None
                and not math.isnan(results.bwd_time)
            ):
                record_bwd_latency = BenchmarkRecord(
                    benchmark=BenchmarkInfo(
                        name=benchmark_name,
                        mode=mode,
                        dtype=dtype,
                        extra_info=benchmark_extra_info,
                    ),
                    model=ModelInfo(
                        name=test_name + str(config.shape),
                        type="attention-benchmark",
                        origins=["pytorch"],
                        extra_info={
                            "operator_name": operator_name,
                        },
                    ),
                    metric=MetricInfo(
                        name="backward latency",
                        unit="us",
                        benchmark_values=[results.bwd_time],
                        target_value=None,
                    ),
                )
                records.append(asdict(record_bwd_latency))

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 1416-1440 / 第 1416-1440 行

```python
    # Write all records to the output file
    with open(output_file, "w", encoding="utf-8") as f:
        json.dump(records, f, indent=2)


def main(
    dynamic: bool = False,
    calculate_bwd: bool = False,
    dtype: DtypeString = "bfloat16",
    b: list[int] | None = None,
    nh: list[str] | None = None,
    s: list[int] | None = None,
    d: list[int] | None = None,
    mods: list[AttentionType] | None = None,
    backend: list[Backend] | None = None,
    max_autotune: bool = False,
    decoding: bool = False,
    kv_size: list[int] | None = None,
    throughput: bool = True,
    save_path: str | None = None,
    output_json_for_dashboard: str | None = None,
    benchmark_name: str = "PyTorch operator microbenchmark",
) -> None:
    """Run sweep over sizes and score mods for flex attention.

```

- **EN:** Important local symbols in this block include main.
- **CN:** 该代码块中的重要局部符号包括 main。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 1441-1457 / 第 1441-1457 行

```python
    Usage Examples:
        # Use a yml config file
        python score_mod.py --config basic_config.yaml

        # Use a json config file
        python score_mod.py --config my_config.json

        # Generate a config template
        python score_mod.py --print-config json > my_config.json # For a json config
        python score_mod.py --print-config yaml > my_config.yaml # For a yaml config

        # Override config with CLI args
        python score_mod.py --config my_config.json -dtype float16 --max-autotune

        # Pure CLI usage
        python score_mod.py -b 4 8 -s 1024 2048 -mods causal alibi --backend efficient

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 1458-1478 / 第 1458-1478 行

```python
    Args:
        dynamic: Runs a dynamic shapes version of compiled flex attention
        calculate_bwd: Calculate backward pass times
        dtype: Data type for tensors (bfloat16, float16, float32)
        b: Batch sizes to benchmark
        nh: Number of query and key/value heads in format "Hq,Hkv"
        s: Sequence lengths to benchmark
        d: Head dimensions to benchmark
        mods: Score modifications: noop, causal, rel, head_bias, alibi, sliding_window, document_mask, prefix_lm, softcap
        backend: Backends for attention computation: math, efficient, cudnn, fav2, fav3, fakv, og-eager
        max_autotune: Turn on max-autotune optimization
        decoding: Benchmark decoding mode (query sequence length = 1)
        kv_size: Key/value cache size in MiB (ignores batch size if specified)
        throughput: Calculate kernel memory bandwidth & computational throughput (always True)
        save_path: Path to save the results CSV file
        output_json_for_dashboard: Path to save results in JSON format for PyTorch OSS dashboard
        benchmark_name: Name of the benchmark for dashboard output
    """
    # Convert dtype string to torch dtype (if not already converted)
    import torch

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 1479-1508 / 第 1479-1508 行

```python
    if isinstance(dtype, str):
        dtype = getattr(torch, dtype)

    # Always calculate throughput
    throughput = True
    print("Backend: ", backend)
    seed = 123
    np.random.seed(seed)
    torch.manual_seed(seed)
    results = []
    for experiment_count, config in enumerate(
        tqdm(
            generate_experiment_configs(
                calculate_bwd,
                dtype,
                b,
                nh,
                s,
                d,
                mods,
                decoding,
                kv_size,
                throughput,
                backend,
                max_autotune,
            )
        ),
        start=1,
    ):
        results.append(
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 1509-1523 / 第 1509-1523 行

```python
            Experiment(
                config,
                run_single_experiment(
                    config,
                    dynamic=dynamic,
                ),
            )
        )

        # Periodic memory cleanup every 50 experiments
        if experiment_count % 50 == 0:
            cleanup_memory()

    print_results(results, save_path)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 1524-1548 / 第 1524-1548 行

```python
    # Output JSON for dashboard if requested
    if output_json_for_dashboard:
        _output_json_for_dashboard(results, output_json_for_dashboard, benchmark_name)


if __name__ == "__main__":
    # Set up the argument parser
    parser = argparse.ArgumentParser(
        description="Run sweep over sizes and score mods for flex attention"
    )
    parser.add_argument(
        "--config",
        type=str,
        help="Path to JSON config file. CLI args override config file values.",
        default=None,
    )
    parser.add_argument(
        "--dynamic",
        action="store_true",
        help="Runs a dynamic shapes version of compiled flex attention.",
    )
    parser.add_argument(
        "--calculate-bwd", action="store_true", help="Calculate backward pass times"
    )

```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 1549-1578 / 第 1549-1578 行

```python
    parser.add_argument("-dtype", type=str, help="dtype", default="bfloat16")

    parser.add_argument(
        "-b", type=int, nargs="+", help="batch sizes", default=[2, 8, 16]
    )
    parser.add_argument(
        "-nh",
        type=heads_input_type,
        nargs="+",
        help="# of q-heads,kv-heads",
        default=[(16, 16), (16, 2)],
    )
    parser.add_argument(
        "-s", type=int, nargs="+", help="sequence lengths", default=[512, 1024, 4096]
    )
    parser.add_argument("-d", type=int, nargs="+", help="head dims", default=[64, 128])
    parser.add_argument(
        "-mods",
        type=str,
        nargs="+",
        help="score mods: noop, causal, rel, head_bias, alibi, sliding_window, document_mask, prefix_lm, softcap",
        default=["noop", "causal", "alibi", "sliding_window"],
    )
    parser.add_argument(
        "--max-autotune", action="store_true", help="Turn on max-autotune"
    )
    parser.add_argument(
        "--decoding",
        action="store_true",
        help="Benchmark Decoding (query sequence length = 1)",
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 1579-1608 / 第 1579-1608 行

```python
    )
    parser.add_argument(
        "--kv-size",
        type=int,
        nargs="+",
        required=False,
        help="""
key/value size in MiB.
Ignores -b batch size and calculate batch size from kv size instead when specified.
""",
    )
    parser.add_argument(
        "--throughput",
        action="store_true",
        help="Calculate kernel memory bandwidth & computational throughput. ",
    )
    parser.add_argument(
        "--save-path",
        type=str,
        help="Path to save the results JSON file (optional)",
        default=None,
    )
    parser.add_argument(
        "--backend",
        type=str,
        nargs="+",
        choices=["math", "efficient", "cudnn", "fav2", "fav3", "fakv"],
        default=["efficient"],
        help="Backend to use for attention computation",
    )
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 1609-1630 / 第 1609-1630 行

```python
    parser.add_argument(
        "--output-json-for-dashboard",
        type=str,
        help="Path to save results in JSON format for PyTorch OSS dashboard",
        default=None,
    )
    parser.add_argument(
        "--benchmark-name",
        type=str,
        help="Name of the benchmark for dashboard output",
        default="PyTorch operator microbenchmark",
    )
    parser.add_argument(
        "--print-config",
        type=str,
        choices=["json", "yaml"],
        help="Print a default config template in JSON or YAML format and exit",
        default=None,
    )
    # Parse arguments
    args = parser.parse_args()

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 1631-1646 / 第 1631-1646 行

```python
    # Handle --print-config
    if args.print_config:
        print_default_config(args.print_config)
        sys.exit(0)

    # Load and merge config if provided
    if args.config:
        config = load_config_file(args.config)

        # Merge config with CLI args (CLI args take precedence)
        json_args = argparse.Namespace()
        json_args.__dict__ = config
        args = parser.parse_args(namespace=json_args)

    # Convert dtype string to torch dtype (only if it's still a string)
    if isinstance(args.dtype, str):
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 1647-1654 / 第 1647-1654 行

```python
        args.dtype = getattr(torch, args.dtype)

    # Remove config and print_config from args before passing to main
    args_dict = vars(args)
    args_dict.pop("config", None)
    args_dict.pop("print_config", None)

    main(**args_dict)
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

## Key Concepts / 关键概念

- **Benchmark infrastructure** — 基准设施
- **PyTorch tensor operations** — 覆盖或配置具体的 PyTorch 张量/算子行为。
- **Data type coverage** — 体现文件如何在不同张量 dtype 间覆盖行为差异。
- **Device coverage** — 突出 CPU/CUDA/移动端等后端相关执行路径。
- **Benchmark workflow** — 记录或实现性能测量场景。
- **Declarative configuration** — 以声明式格式表示构建或工作流设置。
- **Structured metadata** — 存放机器可读的设置或清单式元数据。
- **Representative symbols: ExperimentConfig, Times, ExperimentResults, Experiment, BenchmarkInfo, ModelInfo, MetricInfo, BenchmarkRecord** — 代表性符号：ExperimentConfig、Times、ExperimentResults、Experiment、BenchmarkInfo、ModelInfo、MetricInfo、BenchmarkRecord

## Dependencies / 依赖关系

- `argparse`
- `csv`
- `gc`
- `itertools`
- `json`
- `random`
- `sys`
- `collections`
- `collections.abc`
- `contextlib`
- `dataclasses`
- `functools`
- `typing`
- `numpy`
- `config_utils`
- `tabulate`
- `tqdm`
- `torch`
- `torch.nn.functional`
- `torch.nn.attention`
- `torch.nn.attention.flex_attention`
- `torch._inductor.runtime.benchmarking`
- `attn_gym.mods`
- `attn_gym.masks`
