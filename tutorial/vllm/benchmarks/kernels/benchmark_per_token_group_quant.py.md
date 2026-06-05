# benchmark_per_token_group_quant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_per_token_group_quant.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, quantization benchmarks, centered around `_triton_mode`, `_time_cuda`, `_run_single`, `parse_args`. / 实现与基准测试编排、量化基准相关的逻辑，核心符号包括 `_triton_mode`, `_time_cuda`, `_run_single`, `parse_args`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-13)
```python
import argparse
import math
from collections.abc import Callable
from contextlib import contextmanager
from unittest.mock import patch

import torch

from vllm.model_executor.layers.quantization.utils import fp8_utils, int8_utils
from vllm.platforms import current_platform
```
**EN:** This block gathers standard-library helpers such as `argparse`, `math`, `collections.abc`, `contextlib`, `unittest.mock`; third-party packages such as `torch`; project-local modules such as `vllm.model_executor.layers.quantization.utils`, `vllm.platforms`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`, `math`, `collections.abc`, `contextlib`, `unittest.mock`；第三方依赖，如 `torch`；项目内部模块，如 `vllm.model_executor.layers.quantization.utils`, `vllm.platforms`。这些依赖构成了后续基准测试逻辑的基础。

### Function `_triton_mode` (lines 17-20)
```python
def _triton_mode():
    """Temporarily force the Triton fallback path"""
    with patch("vllm.platforms.current_platform.is_cuda", return_value=False):
        yield
```
**EN:** `_triton_mode` Temporarily force the Triton fallback path. It mainly works with no explicit parameters and relies on `patch` plus context management to move data through this part of the benchmark pipeline.
**CN:** `_triton_mode` 的职责是：Temporarily force the Triton fallback path。 它主要处理 无显式参数，并结合 `patch` 以及 上下文管理 来完成这一段基准测试流程。

### Function `_time_cuda` (lines 23-42)
```python
def _time_cuda(
    fn: Callable[[], tuple[torch.Tensor, torch.Tensor]],
    warmup_iters: int,
    bench_iters: int,
) -> float:
    # warmup
    for _ in range(warmup_iters):
        fn()
    torch.accelerator.synchronize()

    start = torch.Event(enable_timing=True)
    end = torch.Event(enable_timing=True)

    start.record()
    for _ in range(bench_iters):
        fn()
    end.record()
    torch.accelerator.synchronize()

    return start.elapsed_time(end) / bench_iters  # ms/iter
```
**EN:** `_time_cuda` implements a helper used by `benchmark_per_token_group_quant.py`. It mainly works with `fn`, `warmup_iters`, `bench_iters` and relies on `range`, `fn`, `torch.accelerator.synchronize`, `torch.Event`, `start.record`, `end.record` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `_time_cuda` 负责实现 `benchmark_per_token_group_quant.py` 使用的辅助逻辑。 它主要处理 `fn`, `warmup_iters`, `bench_iters`，并结合 `range`, `fn`, `torch.accelerator.synchronize`, `torch.Event`, `start.record`, `end.record` 以及 循环迭代 来完成这一段基准测试流程。

### Function `_run_single` (lines 45-102)
```python
def _run_single(
    shape: tuple[int, int],
    group_size: int,
    dtype: str,
    *,
    column_major: bool = False,
    scale_ue8m0: bool = False,
    warmup_iters: int,
    bench_iters: int,
) -> None:
    num_tokens, hidden_dim = shape

    device = torch.device("cuda")
    torch.manual_seed(42)
    x = torch.randn(num_tokens, hidden_dim, device=device, dtype=torch.bfloat16) * 8

    if dtype == "fp8":

        def cuda_impl():
            return fp8_utils.per_token_group_quant_fp8(
                x,
                group_size,
                column_major_scales=column_major,
                use_ue8m0=scale_ue8m0,
            )

        def triton_impl():
            with _triton_mode():
                return fp8_utils.per_token_group_quant_fp8(
                    x,
                    group_size,
                    column_major_scales=column_major,
                    use_ue8m0=scale_ue8m0,
                )
    # ... omitted for brevity ...
    cfg_desc = (
        f"shape={shape}  gs={group_size:<3}  col_major={column_major:<5}  "
        f"ue8m0={scale_ue8m0:<5}  dtype={dtype}"
    )
    print(
        f"{cfg_desc:55} | CUDA {cuda_ms:7.3f} ms  | Triton {triton_ms:7.3f} ms  | "
        f"speed-up ×{speedup:5.2f}"
    )
```
**EN:** `_run_single` implements a helper used by `benchmark_per_token_group_quant.py`. It mainly works with `shape`, `group_size`, `dtype`, `column_major`, `scale_ue8m0`, ... and relies on `torch.device`, `torch.manual_seed`, `torch.randn`, `fp8_utils.per_token_group_quant_fp8`, `_triton_mode`, `int8_utils.per_token_group_quant_int8` plus branching and context management to move data through this part of the benchmark pipeline.
**CN:** `_run_single` 负责实现 `benchmark_per_token_group_quant.py` 使用的辅助逻辑。 它主要处理 `shape`, `group_size`, `dtype`, `column_major`, `scale_ue8m0`, ...，并结合 `torch.device`, `torch.manual_seed`, `torch.randn`, `fp8_utils.per_token_group_quant_fp8`, `_triton_mode`, `int8_utils.per_token_group_quant_int8` 以及 条件分支、上下文管理 来完成这一段基准测试流程。

### Function `parse_args` (lines 105-110)
```python
def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument("--warmup-iters", type=int, default=10)
    parser.add_argument("--bench-iters", type=int, default=100)
    parser.add_argument("--dtype", choices=["fp8", "int8", "both"], default="both")
    return parser.parse_args()
```
**EN:** `parse_args` parses external input into structured benchmark settings. It mainly works with no explicit parameters and relies on `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args` plus value production to move data through this part of the benchmark pipeline.
**CN:** `parse_args` 负责把外部输入解析为结构化的基准配置。 它主要处理 无显式参数，并结合 `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args` 以及 结果返回 来完成这一段基准测试流程。

### Entry point (lines 113-159)
```python
if __name__ == "__main__":
    if not current_platform.is_cuda():
        raise RuntimeError("CUDA device is required to run this benchmark.")

    args = parse_args()
    warmup_iters, bench_iters = args.warmup_iters, args.bench_iters

    shapes = [(32, 128), (64, 256), (16, 512)]
    group_sizes = [64, 128]

    dtypes = ["fp8", "int8"] if args.dtype == "both" else [args.dtype]

    header = (
        "Configuration".ljust(55)
        + " | "
        + "CUDA (ms)".center(12)
        + " | "
        + "Triton (ms)".center(13)
        + " | "
        + "Speed-up"
    )
    print(header)
    print("-" * len(header))

    for dtype in dtypes:
        for shape in shapes:
            for gs in group_sizes:
                if dtype == "fp8":
                    for col_major in (False, True):
                        for ue8m0 in (False, True):
                            _run_single(
                                shape,
                                gs,
                                dtype,
                                column_major=col_major,
                                scale_ue8m0=ue8m0,
                                warmup_iters=warmup_iters,
                                bench_iters=bench_iters,
                            )
                else:  # INT8 has no col-major / ue8m0 switches
                    _run_single(
                        shape,
                        gs,
                        dtype,
                        warmup_iters=warmup_iters,
                        bench_iters=bench_iters,
                    )
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `current_platform.is_cuda`, `RuntimeError`, `parse_args`, `ljust`, `center`, `print` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `current_platform.is_cuda`, `RuntimeError`, `parse_args`, `ljust`, `center`, `print` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: `argparse`, `math`, `collections.abc`, `contextlib`, `unittest.mock`.
- **CN:** 标准库依赖：`argparse`, `math`, `collections.abc`, `contextlib`, `unittest.mock`。
- **EN:** Third-party packages: `torch`.
- **CN:** 第三方依赖：`torch`。
- **EN:** Internal modules: `vllm.model_executor.layers.quantization.utils`, `vllm.platforms`.
- **CN:** 内部模块：`vllm.model_executor.layers.quantization.utils`, `vllm.platforms`。
