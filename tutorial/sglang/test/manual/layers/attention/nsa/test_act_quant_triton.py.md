# test_act_quant_triton.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/layers/attention/nsa/test_act_quant_triton.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `act quant triton` scenario in `test/manual/layers/attention/nsa`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual/layers/attention/nsa` 中的 `act quant triton` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Imports and shared helpers / 导入与共享辅助项
```python
"""
Unit tests comparing TileLang and Triton implementations of activation quantization.
Tests both accuracy and performance.
"""

import time
from typing import Tuple

import pytest
import torch

from sglang.srt.layers.attention.nsa.tilelang_kernel import act_quant
from sglang.srt.layers.attention.nsa.triton_kernel import act_quant as act_quant_triton
```
**EN:** This range imports `time`, `typing`, `pytest` and `torch`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。

### Lines 16-40: Helper routines around benchmark_kernel / 辅助例程
```python
def benchmark_kernel(
    fn,
    x: torch.Tensor,
    block_size: int,
    scale_fmt,
    warmup: int = 10,
    repeat: int = 100,
    use_cuda_graph: bool = True,
) -> Tuple[float, torch.Tensor, torch.Tensor]:
    """
    Benchmark a kernel function.

    Args:
        fn: Function to benchmark
        x: Input tensor
        block_size: Block size for quantization
        scale_fmt: Scale format
        warmup: Number of warmup iterations
        repeat: Number of repeat iterations
        use_cuda_graph: Whether to use CUDA graphs for more accurate timing

    Returns:
        Tuple of (avg_time_ms, quantized_output, scales)
    """
    # Warmup
```
**EN:** This range implements helper routine(s) `benchmark_kernel` so setup, transformation, or validation logic can be reused cleanly. The code also ties this block to evaluation or benchmark execution. Representative call sites include `of`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 41-65: Scenario logic / 场景逻辑
```python
    for _ in range(warmup):
        y, s = fn(x, block_size=block_size, scale_fmt=scale_fmt)

    if not x.is_cuda or not use_cuda_graph:
        # Fallback to regular timing
        if x.is_cuda:
            torch.cuda.synchronize()

        start = time.perf_counter()
        for _ in range(repeat):
            y, s = fn(x, block_size=block_size, scale_fmt=scale_fmt)

        if x.is_cuda:
            torch.cuda.synchronize()

        end = time.perf_counter()
        avg_time_ms = (end - start) / repeat * 1000

        return avg_time_ms, y, s

    # Use CUDA graph for more accurate timing
    torch.cuda.synchronize()

    # Allocate output buffers
    N = x.size(-1)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `fn`, `synchronize`, `perf_counter` and `size`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 66-90: Scenario logic / 场景逻辑
```python
    y = torch.empty_like(x, dtype=torch.float8_e4m3fn)
    s = x.new_empty(*x.size()[:-1], N // block_size, dtype=torch.float32)

    # Capture CUDA graph
    graph = torch.cuda.CUDAGraph()
    with torch.cuda.graph(graph):
        y_cap, s_cap = fn(x, block_size=block_size, scale_fmt=scale_fmt)

    # Warmup with graph
    for _ in range(warmup):
        graph.replay()

    torch.cuda.synchronize()

    # Timing with CUDA graph
    start_event = torch.cuda.Event(enable_timing=True)
    end_event = torch.cuda.Event(enable_timing=True)

    start_event.record()
    for _ in range(repeat):
        graph.replay()
    end_event.record()

    torch.cuda.synchronize()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `empty_like`, `new_empty`, `size` and `CUDAGraph`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 91-93: Scenario logic / 场景逻辑
```python
    avg_time_ms = start_event.elapsed_time(end_event) / repeat

    return avg_time_ms, y_cap, s_cap
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `elapsed_time`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 96-120: Helper routines around check_accuracy / 辅助例程
```python
def check_accuracy(
    y_ref: torch.Tensor,
    s_ref: torch.Tensor,
    y_test: torch.Tensor,
    s_test: torch.Tensor,
    rtol: float = 1e-2,
    atol: float = 1e-2,
) -> Tuple[bool, dict]:
    """
    Check accuracy between reference and test outputs.

    Args:
        y_ref: Reference quantized output
        s_ref: Reference scales
        y_test: Test quantized output
        s_test: Test scales
        rtol: Relative tolerance
        atol: Absolute tolerance

    Returns:
        Tuple of (passed, metrics_dict)
    """
    # Convert FP8 to float for comparison
    y_ref_float = y_ref.float()
    y_test_float = y_test.float()
```
**EN:** This range implements helper routine(s) `check_accuracy` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `of`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 121-145: Scenario logic / 场景逻辑
```python

    # Compute differences
    y_diff = torch.abs(y_ref_float - y_test_float)
    s_diff = torch.abs(s_ref - s_test)

    # Compute metrics
    y_max_diff = y_diff.max().item()
    y_mean_diff = y_diff.mean().item()
    s_max_diff = s_diff.max().item()
    s_mean_diff = s_diff.mean().item()

    # Check relative and absolute tolerance
    y_close = torch.allclose(y_ref_float, y_test_float, rtol=rtol, atol=atol)
    s_close = torch.allclose(s_ref, s_test, rtol=rtol, atol=atol)

    # Compute percentage of matching elements
    y_match_pct = (y_ref_float == y_test_float).float().mean().item() * 100

    metrics = {
        "y_max_diff": y_max_diff,
        "y_mean_diff": y_mean_diff,
        "y_match_pct": y_match_pct,
        "s_max_diff": s_max_diff,
        "s_mean_diff": s_mean_diff,
        "y_close": y_close,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `abs`, `item`, `mean` and `allclose`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 146-151: Scenario logic / 场景逻辑
```python
        "s_close": s_close,
    }

    passed = y_close and s_close

    return passed, metrics
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 152-154: Scenario logic / 场景逻辑
```python


@pytest.mark.skipif(not torch.cuda.is_available(), reason="CUDA not available")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skipif` and `is_available`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 155-179: Test routines around test_act_quant_comprehensive_benchmark / 测试例程
```python
def test_act_quant_comprehensive_benchmark(scale_fmt=None):
    """Comprehensive benchmark across multiple sizes with CUDA graphs."""
    device = torch.device("cuda")
    dtype = torch.bfloat16
    block_size = 128

    shapes = [
        (128, 512),
        (256, 1024),
        (512, 2048),
        (1024, 4096),
        (2048, 8192),
        (4096, 16384),
    ]

    print("\n" + "=" * 100)
    print("Comprehensive Performance Benchmark with CUDA Graphs")
    print("=" * 100)
    print(
        f"{'Shape':<20} {'TileLang (ms)':<15} {'Triton (ms)':<15} {'Speedup':<10} {'Status'}"
    )
    print("-" * 100)

    for shape in shapes:
        torch.manual_seed(42)
```
**EN:** This range defines concrete test routine(s) `test_act_quant_comprehensive_benchmark`. The logic drives the target scenario and encodes the expected acceptance criteria. The code also ties this block to evaluation or benchmark execution. Representative call sites include `device`, `TileLang`, `Triton` and `manual_seed`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 180-204: Scenario logic / 场景逻辑
```python
        x = torch.randn(shape, dtype=dtype, device=device)

        try:
            # Benchmark both with CUDA graphs
            time_tilelang, y_ref, s_ref = benchmark_kernel(
                act_quant,
                x,
                block_size,
                scale_fmt,
                warmup=5,
                repeat=50,
                use_cuda_graph=True,
            )
            time_triton, y_triton, s_triton = benchmark_kernel(
                act_quant_triton,
                x,
                block_size,
                scale_fmt,
                warmup=5,
                repeat=50,
                use_cuda_graph=True,
            )

            # Check accuracy
            passed, _ = check_accuracy(y_ref, s_ref, y_triton, s_triton)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. Representative call sites include `randn`, `benchmark_kernel` and `check_accuracy`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 205-229: Scenario logic / 场景逻辑
```python

            speedup = time_tilelang / time_triton if time_triton > 0 else 0
            status = "✓ PASS" if passed else "✗ FAIL"

            print(
                f"{str(shape):<20} {time_tilelang:<15.4f} {time_triton:<15.4f} "
                f"{speedup:<10.2f} {status}"
            )
        except Exception as e:
            print(f"{str(shape):<20} ERROR: {str(e)}")

    print("=" * 100)

    # Also run without CUDA graphs for comparison
    print("\n" + "=" * 100)
    print("Performance Benchmark WITHOUT CUDA Graphs (for comparison)")
    print("=" * 100)
    print(
        f"{'Shape':<20} {'TileLang (ms)':<15} {'Triton (ms)':<15} {'Speedup':<10} {'Status'}"
    )
    print("-" * 100)

    for shape in shapes:
        torch.manual_seed(42)
        x = torch.randn(shape, dtype=dtype, device=device)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. Representative call sites include `Graphs`, `TileLang`, `Triton` and `manual_seed`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 230-254: Scenario logic / 场景逻辑
```python

        try:
            # Benchmark both without CUDA graphs
            time_tilelang, y_ref, s_ref = benchmark_kernel(
                act_quant,
                x,
                block_size,
                scale_fmt,
                warmup=5,
                repeat=50,
                use_cuda_graph=False,
            )
            time_triton, y_triton, s_triton = benchmark_kernel(
                act_quant_triton,
                x,
                block_size,
                scale_fmt,
                warmup=5,
                repeat=50,
                use_cuda_graph=False,
            )

            # Check accuracy
            passed, _ = check_accuracy(y_ref, s_ref, y_triton, s_triton)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. Representative call sites include `benchmark_kernel` and `check_accuracy`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 255-265: Scenario logic / 场景逻辑
```python
            speedup = time_tilelang / time_triton if time_triton > 0 else 0
            status = "✓ PASS" if passed else "✗ FAIL"

            print(
                f"{str(shape):<20} {time_tilelang:<15.4f} {time_triton:<15.4f} "
                f"{speedup:<10.2f} {status}"
            )
        except Exception as e:
            print(f"{str(shape):<20} ERROR: {str(e)}")

    print("=" * 100)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 266-281: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    # Run comprehensive benchmark
    if torch.cuda.is_available():
        print("\n" + "=" * 80)
        print("Running Comprehensive Benchmark with scale_fmt=None")
        print("=" * 80)
        test_act_quant_comprehensive_benchmark(scale_fmt=None)

        print("\n" + "=" * 80)
        print("Running Comprehensive Benchmark with scale_fmt!=None")
        print("=" * 80)
        test_act_quant_comprehensive_benchmark(scale_fmt="any")
    else:
        print("CUDA not available. Skipping tests.")
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. The code also ties this block to evaluation or benchmark execution. Representative call sites include `is_available` and `test_act_quant_comprehensive_benchmark`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Pytest markers / Pytest 标记
- Accuracy evaluation / 精度评测
- Performance benchmarking / 性能基准测试
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `time`, `typing`
- **Third-party / 第三方库**: `pytest`, `torch`
- **Project Modules / 项目模块**: `sglang.srt.layers.attention.nsa.tilelang_kernel`, `sglang.srt.layers.attention.nsa.triton_kernel`
