# benchmark_2d_silu_mul_fp8_quant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_2d_silu_mul_fp8_quant.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, activation/quantization benchmarks, FP8 kernel experiments, centered around `GROUP_SIZE`, `FLOAT8_T`, `print_timers`, `ImplType`. / 实现与基准测试编排、激活与量化基准、FP8 内核实验相关的逻辑，核心符号包括 `GROUP_SIZE`, `FLOAT8_T`, `print_timers`, `ImplType`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-20)
```python
from dataclasses import dataclass
from enum import Enum
from itertools import product
from typing import Any

import torch
import torch.utils.benchmark as TBenchmark
from torch.utils.benchmark import Measurement as TMeasurement

from vllm.model_executor.layers.quantization.utils.fp8_utils import (
    _per_token_group_quant_fp8_colmajor,
    silu_mul_per_token_group_quant_fp8_colmajor,
)
from vllm.triton_utils import triton
from vllm.utils.deep_gemm import is_deep_gemm_e8m0_used

from .utils import ArgPool, Bench, CudaGraphBenchParams
```
**EN:** This block gathers standard-library helpers such as `dataclasses`, `enum`, `itertools`, `typing`; third-party packages such as `torch`, `torch.utils.benchmark`; project-local modules such as `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.triton_utils`, `vllm.utils.deep_gemm`, `relative:utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `dataclasses`, `enum`, `itertools`, `typing`；第三方依赖，如 `torch`, `torch.utils.benchmark`；项目内部模块，如 `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.triton_utils`, `vllm.utils.deep_gemm`, `relative:utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 22-23)
```python
GROUP_SIZE = 128
FLOAT8_T = torch.float8_e4m3fn
```
**EN:** This top-level block prepares shared state such as `GROUP_SIZE`, `FLOAT8_T`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `GROUP_SIZE`, `FLOAT8_T`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `print_timers` (lines 26-34)
```python
def print_timers(timers: list[TMeasurement], cuda_graph_nops: int):
    print(
        f"Note : The timings reported above is for {cuda_graph_nops} "
        "consecutive invocations of the benchmarking functions. "
        f"Please divide by {cuda_graph_nops} for single invocation "
        "timings."
    )
    compare = TBenchmark.Compare(timers)
    compare.print()
```
**EN:** `print_timers` formats results for display or export. It mainly works with `timers`, `cuda_graph_nops` and relies on `print`, `TBenchmark.Compare`, `compare.print` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `print_timers` 负责格式化结果以便展示或导出。 它主要处理 `timers`, `cuda_graph_nops`，并结合 `print`, `TBenchmark.Compare`, `compare.print` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Class `ImplType` (lines 37-46)
```python
class ImplType(Enum):
    SILU_MUL_PER_TOKEN_GROUP_QUANT_FP8_COLMAJOR = 1
    REFERENCE = 2

    def get_impl(self):
        if self == ImplType.SILU_MUL_PER_TOKEN_GROUP_QUANT_FP8_COLMAJOR:
            return silu_mul_per_token_group_quant_fp8_colmajor
        elif self == ImplType.REFERENCE:
            return reference
        raise ValueError(f"Unrecognized ImplType {self}")
```
**EN:** Class `ImplType` packages shared state and related operations for this benchmark module. It extends `Enum` and exposes methods such as `get_impl`.
**CN:** 类 `ImplType` 用于封装该基准模块的共享状态与相关操作。它继承自 `Enum`，并提供 `get_impl` 等方法。

### Method `ImplType.get_impl` (lines 41-46)
```python
    def get_impl(self):
        if self == ImplType.SILU_MUL_PER_TOKEN_GROUP_QUANT_FP8_COLMAJOR:
            return silu_mul_per_token_group_quant_fp8_colmajor
        elif self == ImplType.REFERENCE:
            return reference
        raise ValueError(f"Unrecognized ImplType {self}")
```
**EN:** `get_impl` implements a helper used by `benchmark_2d_silu_mul_fp8_quant.py`. It mainly works with no explicit parameters and relies on `ValueError` plus branching to move data through this part of the benchmark pipeline.
**CN:** `get_impl` 负责实现 `benchmark_2d_silu_mul_fp8_quant.py` 使用的辅助逻辑。 它主要处理 无显式参数，并结合 `ValueError` 以及 条件分支 来完成这一段基准测试流程。

### Class `BenchmarkTensors` (lines 50-105)
```python
class BenchmarkTensors:
    input: torch.Tensor
    output: torch.Tensor

    # Reference act output tensor
    ref_act_out: torch.Tensor
    ref_quant_out: torch.Tensor

    @staticmethod
    def make(T: int, N: int) -> "BenchmarkTensors":
        assert T % GROUP_SIZE == 0
        assert N % (GROUP_SIZE * 2) == 0

        input = torch.rand((T, N), dtype=torch.bfloat16, device="cuda")

        # silu_mul_per_token_group_quant_fp8_colmajor output.
        output = torch.rand((T, N // 2), dtype=torch.bfloat16, device="cuda").to(
            FLOAT8_T
        )

        # reference output.
        ref_act_out = torch.empty((T, N // 2), dtype=torch.bfloat16, device="cuda")
        ref_quant_out = torch.empty(
            (T, N // 2), dtype=torch.bfloat16, device="cuda"
        ).to(FLOAT8_T)

        return BenchmarkTensors(
            input=input,
            output=output,
            ref_act_out=ref_act_out,
            ref_quant_out=ref_quant_out,
        )

    @property
    # ... omitted for brevity ...
        elif impl_type == ImplType.REFERENCE:
            return {
                "input": self.input,
                "act_out": self.ref_act_out,
                "quant_out": self.ref_quant_out,
                "use_ue8m0": is_deep_gemm_e8m0_used(),
            }
        raise ValueError(f"Unrecognized impl_type {impl_type}")
```
**EN:** Class `BenchmarkTensors` packages shared state and related operations for this benchmark module. It extends `object` and exposes methods such as `make`, `T`, `N`, `make_impl_kwargs`.
**CN:** 类 `BenchmarkTensors` 用于封装该基准模块的共享状态与相关操作。它继承自 `object`，并提供 `make`, `T`, `N`, `make_impl_kwargs` 等方法。

### Method `BenchmarkTensors.make` (lines 59-81)
```python
    def make(T: int, N: int) -> "BenchmarkTensors":
        assert T % GROUP_SIZE == 0
        assert N % (GROUP_SIZE * 2) == 0

        input = torch.rand((T, N), dtype=torch.bfloat16, device="cuda")

        # silu_mul_per_token_group_quant_fp8_colmajor output.
        output = torch.rand((T, N // 2), dtype=torch.bfloat16, device="cuda").to(
            FLOAT8_T
        )

        # reference output.
        ref_act_out = torch.empty((T, N // 2), dtype=torch.bfloat16, device="cuda")
        ref_quant_out = torch.empty(
            (T, N // 2), dtype=torch.bfloat16, device="cuda"
        ).to(FLOAT8_T)

        return BenchmarkTensors(
            input=input,
            output=output,
            ref_act_out=ref_act_out,
            ref_quant_out=ref_quant_out,
        )
```
**EN:** `make` implements a helper used by `benchmark_2d_silu_mul_fp8_quant.py`. It mainly works with `T`, `N` and relies on `torch.rand`, `torch.rand.to`, `torch.empty`, `torch.empty.to`, `BenchmarkTensors` plus value production to move data through this part of the benchmark pipeline.
**CN:** `make` 负责实现 `benchmark_2d_silu_mul_fp8_quant.py` 使用的辅助逻辑。 它主要处理 `T`, `N`，并结合 `torch.rand`, `torch.rand.to`, `torch.empty`, `torch.empty.to`, `BenchmarkTensors` 以及 结果返回 来完成这一段基准测试流程。

### Method `BenchmarkTensors.T` (lines 84-85)
```python
    def T(self):
        return self.input.size(0)
```
**EN:** `T` implements a helper used by `benchmark_2d_silu_mul_fp8_quant.py`. It mainly works with no explicit parameters and relies on `self.input.size` plus value production to move data through this part of the benchmark pipeline.
**CN:** `T` 负责实现 `benchmark_2d_silu_mul_fp8_quant.py` 使用的辅助逻辑。 它主要处理 无显式参数，并结合 `self.input.size` 以及 结果返回 来完成这一段基准测试流程。

### Method `BenchmarkTensors.N` (lines 88-89)
```python
    def N(self):
        return self.input.size(1)
```
**EN:** `N` implements a helper used by `benchmark_2d_silu_mul_fp8_quant.py`. It mainly works with no explicit parameters and relies on `self.input.size` plus value production to move data through this part of the benchmark pipeline.
**CN:** `N` 负责实现 `benchmark_2d_silu_mul_fp8_quant.py` 使用的辅助逻辑。 它主要处理 无显式参数，并结合 `self.input.size` 以及 结果返回 来完成这一段基准测试流程。

### Method `BenchmarkTensors.make_impl_kwargs` (lines 91-105)
```python
    def make_impl_kwargs(self, impl_type: ImplType) -> dict[str, Any]:
        if impl_type == ImplType.SILU_MUL_PER_TOKEN_GROUP_QUANT_FP8_COLMAJOR:
            return {
                "input": self.input,
                "output": self.output,
                "use_ue8m0": is_deep_gemm_e8m0_used(),
            }
        elif impl_type == ImplType.REFERENCE:
            return {
                "input": self.input,
                "act_out": self.ref_act_out,
                "quant_out": self.ref_quant_out,
                "use_ue8m0": is_deep_gemm_e8m0_used(),
            }
        raise ValueError(f"Unrecognized impl_type {impl_type}")
```
**EN:** `make_impl_kwargs` implements a helper used by `benchmark_2d_silu_mul_fp8_quant.py`. It mainly works with `impl_type` and relies on `is_deep_gemm_e8m0_used`, `ValueError` plus branching to move data through this part of the benchmark pipeline.
**CN:** `make_impl_kwargs` 负责实现 `benchmark_2d_silu_mul_fp8_quant.py` 使用的辅助逻辑。 它主要处理 `impl_type`，并结合 `is_deep_gemm_e8m0_used`, `ValueError` 以及 条件分支 来完成这一段基准测试流程。

### Function `reference_quant` (lines 108-146)
```python
def reference_quant(x: torch.Tensor, quant_out: torch.Tensor, use_ue8m0: bool):
    """
    Reference triton quant kernel from,
    vllm.model_executor.layers.quantization.utils.fp8_utils
    """
    assert quant_out.size() == x.size()
    # Allocate the scale tensor column-major format.
    shape = (x.shape[-1] // GROUP_SIZE,) + x.shape[:-1]
    x_q = quant_out
    x_s = torch.empty(shape, device=x.device, dtype=torch.float32).permute(-1, -2)

    M = x.numel() // GROUP_SIZE
    N = GROUP_SIZE
    BLOCK = triton.next_power_of_2(N)
    # heuristics for number of warps
    num_warps = min(max(BLOCK // 256, 1), 8)
    num_stages = 1

    finfo = torch.finfo(FLOAT8_T)
    fp8_min = finfo.min
    fp8_max = finfo.max

    _per_token_group_quant_fp8_colmajor[(M,)](
        x,
        x_q,
        x_s,
        GROUP_SIZE,
        x.shape[1],
        x.stride(0),
        x_s.stride(1),
        eps=1e-10,
        fp8_min=fp8_min,
        fp8_max=fp8_max,
        use_ue8m0=use_ue8m0,
        BLOCK=BLOCK,
        num_warps=num_warps,
        num_stages=num_stages,
    )
    return x_q, x_s
```
**EN:** `reference_quant` Reference triton quant kernel from, vllm.model_executor.layers.quantization.utils.fp8_utils. It mainly works with `x`, `quant_out`, `use_ue8m0` and relies on `quant_out.size`, `x.size`, `torch.empty.permute`, `torch.empty`, `x.numel`, `triton.next_power_of_2` plus value production to move data through this part of the benchmark pipeline.
**CN:** `reference_quant` 的职责是：Reference triton quant kernel from, vllm.model_executor.layers.quantization.utils.fp8_utils。 它主要处理 `x`, `quant_out`, `use_ue8m0`，并结合 `quant_out.size`, `x.size`, `torch.empty.permute`, `torch.empty`, `x.numel`, `triton.next_power_of_2` 以及 结果返回 来完成这一段基准测试流程。

### Function `reference` (lines 149-156)
```python
def reference(
    input: torch.Tensor,
    act_out: torch.Tensor,
    quant_out: torch.Tensor,
    use_ue8m0: bool,
) -> tuple[torch.Tensor, torch.Tensor]:
    torch.ops._C.silu_and_mul(act_out, input)
    return reference_quant(act_out, quant_out, use_ue8m0)
```
**EN:** `reference` implements a helper used by `benchmark_2d_silu_mul_fp8_quant.py`. It mainly works with `input`, `act_out`, `quant_out`, `use_ue8m0` and relies on `torch.ops._C.silu_and_mul`, `reference_quant` plus value production to move data through this part of the benchmark pipeline.
**CN:** `reference` 负责实现 `benchmark_2d_silu_mul_fp8_quant.py` 使用的辅助逻辑。 它主要处理 `input`, `act_out`, `quant_out`, `use_ue8m0`，并结合 `torch.ops._C.silu_and_mul`, `reference_quant` 以及 结果返回 来完成这一段基准测试流程。

### Function `bench_impl` (lines 159-191)
```python
def bench_impl(
    bench_tensors: list[BenchmarkTensors], impl_type: ImplType
) -> TMeasurement:
    T = bench_tensors[0].T
    N = bench_tensors[0].N

    arg_pool_size = len(bench_tensors)
    kwargs_list = [bt.make_impl_kwargs(impl_type) for bt in bench_tensors]

    # warmup
    for kwargs in kwargs_list:
        impl_type.get_impl()(**kwargs)
    torch.accelerator.synchronize()

    # Merge into a single kwargs and qualify arguments as ArgPool
    kwargs = {k: ArgPool([]) for k in kwargs_list[0]}
    for _kwargs in kwargs_list:
        for k, v in _kwargs.items():
            kwargs[k].values.append(v)

    cuda_graph_params = None
    cuda_graph_params = CudaGraphBenchParams(arg_pool_size)
    timer = None
    with Bench(
        cuda_graph_params,
        "silu-mul-quant",
        f"num_tokens={T}, N={N}",
        impl_type.name,
        impl_type.get_impl(),
        **kwargs,
    ) as bench:
        timer = bench.run()
    return timer
```
**EN:** `bench_impl` implements a helper used by `benchmark_2d_silu_mul_fp8_quant.py`. It mainly works with `bench_tensors`, `impl_type` and relies on `len`, `bt.make_impl_kwargs`, `impl_type.get_impl`, `torch.accelerator.synchronize`, `ArgPool`, `_kwargs.items` plus iteration and context management to move data through this part of the benchmark pipeline.
**CN:** `bench_impl` 负责实现 `benchmark_2d_silu_mul_fp8_quant.py` 使用的辅助逻辑。 它主要处理 `bench_tensors`, `impl_type`，并结合 `len`, `bt.make_impl_kwargs`, `impl_type.get_impl`, `torch.accelerator.synchronize`, `ArgPool`, `_kwargs.items` 以及 循环迭代、上下文管理 来完成这一段基准测试流程。

### Function `test_correctness` (lines 194-211)
```python
def test_correctness(T: int, N: int):
    print(f"Testing num_tokens={T}, N={N} ...")

    bench_tensor = BenchmarkTensors.make(T, N)

    def output_from_impl(impl: ImplType) -> tuple[torch.Tensor, torch.Tensor]:
        return impl.get_impl()(**bench_tensor.make_impl_kwargs(impl))

    # reference output
    ref_out_q, ref_out_s = output_from_impl(ImplType.REFERENCE)

    # test output
    out_q, out_s = output_from_impl(
        ImplType.SILU_MUL_PER_TOKEN_GROUP_QUANT_FP8_COLMAJOR
    )

    torch.testing.assert_close(ref_out_q.to(torch.float32), out_q.to(torch.float32))
    torch.testing.assert_close(ref_out_s, out_s)
```
**EN:** `test_correctness` implements a helper used by `benchmark_2d_silu_mul_fp8_quant.py`. It mainly works with `T`, `N` and relies on `print`, `BenchmarkTensors.make`, `impl.get_impl`, `bench_tensor.make_impl_kwargs`, `output_from_impl`, `torch.testing.assert_close` plus value production to move data through this part of the benchmark pipeline.
**CN:** `test_correctness` 负责实现 `benchmark_2d_silu_mul_fp8_quant.py` 使用的辅助逻辑。 它主要处理 `T`, `N`，并结合 `print`, `BenchmarkTensors.make`, `impl.get_impl`, `bench_tensor.make_impl_kwargs`, `output_from_impl`, `torch.testing.assert_close` 以及 结果返回 来完成这一段基准测试流程。

### Function `run` (lines 214-236)
```python
def run(Ts: list[int], Ns: list[int], arg_pool_size: int) -> list[TMeasurement]:
    timers = []
    for N, T in product(Ns, Ts):
        test_correctness(T, N)

        bench_tensors: list[BenchmarkTensors] = [
            BenchmarkTensors.make(T, N) for _ in range(arg_pool_size)
        ]

        silu_mul_quant_timer = bench_impl(
            bench_tensors, ImplType.SILU_MUL_PER_TOKEN_GROUP_QUANT_FP8_COLMAJOR
        )
        timers.append(silu_mul_quant_timer)
        reference_timer = bench_impl(bench_tensors, ImplType.REFERENCE)
        timers.append(reference_timer)

        print_timers(
            [silu_mul_quant_timer, reference_timer], cuda_graph_nops=arg_pool_size
        )

    print_timers(timers, cuda_graph_nops=arg_pool_size)

    return timers
```
**EN:** `run` implements a helper used by `benchmark_2d_silu_mul_fp8_quant.py`. It mainly works with `Ts`, `Ns`, `arg_pool_size` and relies on `product`, `test_correctness`, `BenchmarkTensors.make`, `range`, `bench_impl`, `timers.append` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `run` 负责实现 `benchmark_2d_silu_mul_fp8_quant.py` 使用的辅助逻辑。 它主要处理 `Ts`, `Ns`, `arg_pool_size`，并结合 `product`, `test_correctness`, `BenchmarkTensors.make`, `range`, `bench_impl`, `timers.append` 以及 循环迭代 来完成这一段基准测试流程。

### Entry point (lines 239-244)
```python
if __name__ == "__main__":
    T = [128 * i for i in range(1, 16)] + [2048 * i for i in range(1, 65)]
    N = [2048, 4096, 8192]

    print(f"T = {T}, N = {N}")
    run(T, N, arg_pool_size=8)
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `range`, `print`, `run` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `range`, `print`, `run` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: `dataclasses`, `enum`, `itertools`, `typing`.
- **CN:** 标准库依赖：`dataclasses`, `enum`, `itertools`, `typing`。
- **EN:** Third-party packages: `torch`, `torch.utils.benchmark`.
- **CN:** 第三方依赖：`torch`, `torch.utils.benchmark`。
- **EN:** Internal modules: `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.triton_utils`, `vllm.utils.deep_gemm`, `relative:utils`.
- **CN:** 内部模块：`vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.triton_utils`, `vllm.utils.deep_gemm`, `relative:utils`。
