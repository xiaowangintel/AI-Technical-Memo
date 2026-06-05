# test_timestep_embedding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/test_timestep_embedding.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import os". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import os”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-41: Imports and module setup
```python
import os
import sys

import numpy as np
import pytest
import torch

try:
    import tabulate
except Exception:
    tabulate = None

from sglang.jit_kernel.timestep_embedding import (
    timestep_embedding as timestep_embedding_cuda,
)
from sglang.jit_kernel.utils import get_ci_test_range
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=16, suite="base-b-kernel-unit-1-gpu-large")
register_cuda_ci(est_time=120, suite="nightly-kernel-1-gpu", nightly=True)

CORRECTNESS_BATCH_SIZES = get_ci_test_range(
    [1, 2, 8, 128, 256, 512, 1536, 2048, 4096, 11008, 16384],
    [1, 128, 2048, 16384],
)
CORRECTNESS_DIMS = get_ci_test_range(
    [32, 128, 256, 512, 1536, 2048, 4096, 8192],
    [32, 512, 8192],
)
DIFFUSERS_BATCH_SIZES = get_ci_test_range(
    [1, 2, 8, 128, 256, 512, 1536, 2048, 16384],
    [1, 512, 16384],
)
DIFFUSERS_DIMS = get_ci_test_range([32, 256, 512, 1536, 8192], [32, 512, 8192])
DTYPES = get_ci_test_range(
    [torch.float16, torch.bfloat16, torch.float32],
    [torch.float16, torch.bfloat16],
)
SCALES = get_ci_test_range([1, 0.01], [1, 0.01])
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 42-77: Function `get_timestep_embedding_reference`
```python
def get_timestep_embedding_reference(
    timesteps: torch.Tensor,
    dim: int,
    *,
    flip_sin_to_cos: bool = False,
    downscale_freq_shift: float = 1,
    scale: float = 1,
    max_period: int = 10000,
):
    assert len(timesteps.shape) == 1, "Timesteps should be a 1d-array"

    timesteps = timesteps.to(torch.float32)
    half_dim = dim // 2
    exponent = -torch.log(
        torch.tensor(max_period, dtype=torch.float32, device=timesteps.device)
    ) * torch.arange(
        start=0, end=half_dim, dtype=torch.float32, device=timesteps.device
    )
    exponent = exponent / (half_dim - downscale_freq_shift)

    emb = torch.exp(exponent)
    emb = timesteps[:, None].float() * emb[None, :]

    emb = scale * emb

    emb = torch.cat([torch.sin(emb), torch.cos(emb)], dim=-1)
    if flip_sin_to_cos:
        emb = torch.cat([emb[:, half_dim:], emb[:, :half_dim]], dim=-1)
    if dim % 2 == 1:
        emb = torch.nn.functional.pad(emb, (0, 1, 0, 0))
    return emb


@pytest.mark.parametrize("batch_size", CORRECTNESS_BATCH_SIZES)
@pytest.mark.parametrize("dim", CORRECTNESS_DIMS)
@pytest.mark.parametrize("dtype", DTYPES)
```
**EN:** This block defines `get_timestep_embedding_reference`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `get_timestep_embedding_reference`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 78-95: Function `test_timestep_embedding_correctness_with_sgld`
```python
def test_timestep_embedding_correctness_with_sgld(batch_size, dim, dtype):
    device = "cuda"
    t = torch.randint(low=0, high=1000, size=(batch_size,), device=device).to(dtype)
    torch_output = get_timestep_embedding_reference(
        t, dim, flip_sin_to_cos=True, downscale_freq_shift=0
    )
    cuda_output = timestep_embedding_cuda(
        t, dim, flip_sin_to_cos=True, downscale_freq_shift=0
    )
    torch.testing.assert_close(torch_output, cuda_output, atol=1e-3, rtol=1e-3)


@pytest.mark.parametrize("batch_size", DIFFUSERS_BATCH_SIZES)
@pytest.mark.parametrize("dim", DIFFUSERS_DIMS)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("flip_sin_to_cos", [False, True])
@pytest.mark.parametrize("downscale_freq_shift", [0, 1])
@pytest.mark.parametrize("scale", SCALES)
```
**EN:** This block defines `test_timestep_embedding_correctness_with_sgld`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_timestep_embedding_correctness_with_sgld`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 96-119: Function `test_timestep_embedding_correctness_with_diffusers`
```python
def test_timestep_embedding_correctness_with_diffusers(
    batch_size, dim, flip_sin_to_cos, downscale_freq_shift, scale, dtype
):
    device = "cuda"
    t = torch.randint(low=0, high=1000, size=(batch_size,), device=device).to(dtype)
    torch_output = get_timestep_embedding_reference(
        t,
        dim,
        flip_sin_to_cos=flip_sin_to_cos,
        downscale_freq_shift=downscale_freq_shift,
        scale=scale,
        max_period=10000,
    )
    cuda_output = timestep_embedding_cuda(
        t,
        dim,
        flip_sin_to_cos=flip_sin_to_cos,
        downscale_freq_shift=downscale_freq_shift,
        scale=scale,
        max_period=10000,
    )
    torch.testing.assert_close(torch_output, cuda_output, atol=1e-3, rtol=1e-3)
```
**EN:** This block defines `test_timestep_embedding_correctness_with_diffusers`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_timestep_embedding_correctness_with_diffusers`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 120-183: Function `test_timestep_embedding_perf`
```python
def test_timestep_embedding_perf():
    if os.environ.get("SGLANG_RUN_JIT_KERNEL_PERF_TESTS") != "1":
        pytest.skip("Perf test disabled by default")
    if tabulate is None:
        pytest.skip("Optional dependency 'tabulate' is not installed")

    NUM_BATCH = [1, 2, 8, 63, 256, 512, 613, 1024, 1536]
    NUM_DIM = [32, 64, 128, 256, 512, 1024, 2048, 4096]

    def perf_kernel_fn(kernel_fn: callable, *args, **kwargs):
        warmup_times = 4
        repeat_times = 20
        start = torch.cuda.Event(enable_timing=True)
        end = torch.cuda.Event(enable_timing=True)

        for _ in range(warmup_times):
            output_fn = kernel_fn(*args, **kwargs)
        torch.cuda.synchronize()

        start.record()
        for _ in range(repeat_times):
            output_fn = kernel_fn(*args, **kwargs)
        end.record()
        end.synchronize()
        return start.elapsed_time(end) / repeat_times

    device = "cuda"
    results = []

    cuda_speedups = []
    for B in NUM_BATCH:
        for dim in NUM_DIM:
            t = torch.linspace(0, max(100000, B), steps=B, device=device).to(
                torch.float32
            )
            time_torch = perf_kernel_fn(get_timestep_embedding_reference, t, dim)
            time_cuda = perf_kernel_fn(timestep_embedding_cuda, t, dim)
            speedup_cuda = time_torch / time_cuda

            results.append(
                {
                    "Batch Size": B,
                    "Dimension": dim,
                    "Torch Time (ms)": time_torch,
                    "CUDA Time (ms)": time_cuda,
                    "Speedup (CUDA)": speedup_cuda,
                }
            )
            cuda_speedups.append(speedup_cuda)

    print("=== Timestep Embedding Benchmark Results ===")
    print(
        tabulate.tabulate(
            results,
            headers="keys",
            tablefmt="fancy_grid",
            floatfmt=(".0f", ".0f", ".6f", ".6f", ".5f"),
        )
    )
    print(f"Average Speedup(cuda): {np.mean(cuda_speedups):.4f}")


if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v", "-s"]))
```
**EN:** This block defines `test_timestep_embedding_perf`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_timestep_embedding_perf`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `os`
- `sys`
- `numpy as np`
- `pytest`
- `torch`
- `tabulate`
- `sglang.jit_kernel.timestep_embedding -> (`
- `sglang.jit_kernel.utils -> get_ci_test_range`
- `sglang.test.ci.ci_register -> register_cuda_ci`
