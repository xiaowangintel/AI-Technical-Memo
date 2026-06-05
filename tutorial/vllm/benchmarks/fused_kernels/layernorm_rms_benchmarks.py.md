# layernorm_rms_benchmarks.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/fused_kernels/layernorm_rms_benchmarks.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements normalization kernel benchmarks, benchmark orchestration, centered around `bench_params_t`, `get_bench_params`, `unfused_int8_impl`, `unfused_fp8_impl`. / 实现与归一化内核基准、基准测试编排相关的逻辑，核心符号包括 `bench_params_t`, `get_bench_params`, `unfused_int8_impl`, `unfused_fp8_impl`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-20)
```python
import pickle as pkl
import time
from collections.abc import Callable, Iterable
from dataclasses import dataclass
from itertools import product

import torch
import torch.utils.benchmark as TBenchmark
from torch.utils.benchmark import Measurement as TMeasurement
from tqdm import tqdm

import vllm._custom_ops as ops
from vllm.benchmarks.lib.utils import default_vllm_config
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.model_executor.layers.quantization.utils.fp8_utils import (
    per_token_group_quant_fp8,
)
```
**EN:** This block gathers standard-library helpers such as `pickle`, `time`, `collections.abc`, `dataclasses`, `itertools`; third-party packages such as `torch`, `torch.utils.benchmark`, `tqdm`; project-local modules such as `vllm._custom_ops`, `vllm.benchmarks.lib.utils`, `vllm.model_executor.layers.layernorm`, `vllm.model_executor.layers.quantization.utils.fp8_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `pickle`, `time`, `collections.abc`, `dataclasses`, `itertools`；第三方依赖，如 `torch`, `torch.utils.benchmark`, `tqdm`；项目内部模块，如 `vllm._custom_ops`, `vllm.benchmarks.lib.utils`, `vllm.model_executor.layers.layernorm`, `vllm.model_executor.layers.quantization.utils.fp8_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Class `bench_params_t` (lines 24-38)
```python
class bench_params_t:
    num_tokens: int
    hidden_size: int
    add_residual: bool
    dtype: torch.dtype
    group_size: list[int]

    def description(self):
        return (
            f"N {self.num_tokens} "
            f"x D {self.hidden_size} "
            f"x R {self.add_residual} "
            f"x DT {self.dtype}"
            f"x GS {self.group_size}"
        )
```
**EN:** Class `bench_params_t` packages shared state and related operations for this benchmark module. It extends `object` and exposes methods such as `description`.
**CN:** 类 `bench_params_t` 用于封装该基准模块的共享状态与相关操作。它继承自 `object`，并提供 `description` 等方法。

### Method `bench_params_t.description` (lines 31-38)
```python
    def description(self):
        return (
            f"N {self.num_tokens} "
            f"x D {self.hidden_size} "
            f"x R {self.add_residual} "
            f"x DT {self.dtype}"
            f"x GS {self.group_size}"
        )
```
**EN:** `description` implements a helper used by `layernorm_rms_benchmarks.py`. It mainly works with no explicit parameters and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `description` 负责实现 `layernorm_rms_benchmarks.py` 使用的辅助逻辑。 它主要处理 无显式参数，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Function `get_bench_params` (lines 41-53)
```python
def get_bench_params() -> list[bench_params_t]:
    ## Test Fixtures
    NUM_TOKENS = [2**x for x in range(11)]
    HIDDEN_SIZES = list(range(1024, 8129, 1024))
    ADD_RESIDUAL = [True, False]
    DTYPES = [torch.bfloat16, torch.float]
    GROUP_SIZES = [[1, 64], [1, 128]]

    combinations = product(NUM_TOKENS, HIDDEN_SIZES, ADD_RESIDUAL, DTYPES, GROUP_SIZES)
    bench_params = list(
        map(lambda x: bench_params_t(x[0], x[1], x[2], x[3], x[4]), combinations)
    )
    return bench_params
```
**EN:** `get_bench_params` implements a helper used by `layernorm_rms_benchmarks.py`. It mainly works with no explicit parameters and relies on `range`, `list`, `product`, `map`, `bench_params_t` plus value production to move data through this part of the benchmark pipeline.
**CN:** `get_bench_params` 负责实现 `layernorm_rms_benchmarks.py` 使用的辅助逻辑。 它主要处理 无显式参数，并结合 `range`, `list`, `product`, `map`, `bench_params_t` 以及 结果返回 来完成这一段基准测试流程。

### Function `unfused_int8_impl` (lines 57-72)
```python
def unfused_int8_impl(
    rms_norm_layer: RMSNorm,
    x: torch.Tensor,
    residual: torch.Tensor | None,
    quant_dtype: torch.dtype,
    group_size: list[int],
):
    # Norm
    torch_out = None
    if residual is None:
        torch_out = rms_norm_layer.forward_cuda(x, residual)
    else:
        torch_out, _ = rms_norm_layer.forward_cuda(x, residual)

    # Quant
    torch_out, _, _ = ops.scaled_int8_quant(torch_out)
```
**EN:** `unfused_int8_impl` implements a helper used by `layernorm_rms_benchmarks.py`. It mainly works with `rms_norm_layer`, `x`, `residual`, `quant_dtype`, `group_size` and relies on `rms_norm_layer.forward_cuda`, `ops.scaled_int8_quant` plus branching to move data through this part of the benchmark pipeline.
**CN:** `unfused_int8_impl` 负责实现 `layernorm_rms_benchmarks.py` 使用的辅助逻辑。 它主要处理 `rms_norm_layer`, `x`, `residual`, `quant_dtype`, `group_size`，并结合 `rms_norm_layer.forward_cuda`, `ops.scaled_int8_quant` 以及 条件分支 来完成这一段基准测试流程。

### Function `unfused_fp8_impl` (lines 75-90)
```python
def unfused_fp8_impl(
    rms_norm_layer: RMSNorm,
    x: torch.Tensor,
    residual: torch.Tensor | None,
    quant_dtype: torch.dtype,
    group_size: list[int],
):
    # Norm
    torch_out = None
    if residual is None:
        torch_out = rms_norm_layer.forward_cuda(x, residual)
    else:
        torch_out, _ = rms_norm_layer.forward_cuda(x, residual)

    # Quant
    torch_out, _ = ops.scaled_fp8_quant(torch_out)
```
**EN:** `unfused_fp8_impl` coordinates or measures FP8 kernel experiments. It mainly works with `rms_norm_layer`, `x`, `residual`, `quant_dtype`, `group_size` and relies on `rms_norm_layer.forward_cuda`, `ops.scaled_fp8_quant` plus branching to move data through this part of the benchmark pipeline.
**CN:** `unfused_fp8_impl` 负责协调或测量FP8 内核实验。 它主要处理 `rms_norm_layer`, `x`, `residual`, `quant_dtype`, `group_size`，并结合 `rms_norm_layer.forward_cuda`, `ops.scaled_fp8_quant` 以及 条件分支 来完成这一段基准测试流程。

### Function `unfused_groupwise_fp8_impl` (lines 93-110)
```python
def unfused_groupwise_fp8_impl(
    rms_norm_layer: RMSNorm,
    x: torch.Tensor,
    residual: torch.Tensor | None,
    quant_dtype: torch.dtype,
    group_size: list[int],
):
    # Norm
    torch_out = None
    if residual is None:
        torch_out = rms_norm_layer.forward_cuda(x, residual)
    else:
        torch_out, _ = rms_norm_layer.forward_cuda(x, residual)

    # Quant
    torch_out, _ = per_token_group_quant_fp8(
        torch_out, group_size=group_size[1], use_ue8m0=False
    )
```
**EN:** `unfused_groupwise_fp8_impl` coordinates or measures FP8 kernel experiments. It mainly works with `rms_norm_layer`, `x`, `residual`, `quant_dtype`, `group_size` and relies on `rms_norm_layer.forward_cuda`, `per_token_group_quant_fp8` plus branching to move data through this part of the benchmark pipeline.
**CN:** `unfused_groupwise_fp8_impl` 负责协调或测量FP8 内核实验。 它主要处理 `rms_norm_layer`, `x`, `residual`, `quant_dtype`, `group_size`，并结合 `rms_norm_layer.forward_cuda`, `per_token_group_quant_fp8` 以及 条件分支 来完成这一段基准测试流程。

### Function `fused_impl` (lines 113-122)
```python
def fused_impl(
    rms_norm_layer: RMSNorm,  # this stores the weights
    x: torch.Tensor,
    residual: torch.Tensor | None,
    quant_dtype: torch.dtype,
    group_size: list[int],
):
    out, _ = ops.rms_norm_dynamic_per_token_quant(
        x, rms_norm_layer.weight, 1e-6, quant_dtype, residual=residual
    )
```
**EN:** `fused_impl` implements a helper used by `layernorm_rms_benchmarks.py`. It mainly works with `rms_norm_layer`, `x`, `residual`, `quant_dtype`, `group_size` and relies on `ops.rms_norm_dynamic_per_token_quant` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `fused_impl` 负责实现 `layernorm_rms_benchmarks.py` 使用的辅助逻辑。 它主要处理 `rms_norm_layer`, `x`, `residual`, `quant_dtype`, `group_size`，并结合 `ops.rms_norm_dynamic_per_token_quant` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Function `fused_groupwise_impl` (lines 125-140)
```python
def fused_groupwise_impl(
    rms_norm_layer: RMSNorm,  # this stores the weights
    x: torch.Tensor,
    residual: torch.Tensor | None,
    quant_dtype: torch.dtype,
    group_size: list[int],
):
    out, _ = ops.rms_norm_per_block_quant(
        x,
        rms_norm_layer.weight,
        1e-6,
        quant_dtype,
        group_size,
        residual=residual,
        is_scale_transposed=True,
    )
```
**EN:** `fused_groupwise_impl` implements a helper used by `layernorm_rms_benchmarks.py`. It mainly works with `rms_norm_layer`, `x`, `residual`, `quant_dtype`, `group_size` and relies on `ops.rms_norm_per_block_quant` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `fused_groupwise_impl` 负责实现 `layernorm_rms_benchmarks.py` 使用的辅助逻辑。 它主要处理 `rms_norm_layer`, `x`, `residual`, `quant_dtype`, `group_size`，并结合 `ops.rms_norm_per_block_quant` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Function `bench_fn` (lines 144-171)
```python
def bench_fn(
    rms_norm_layer: RMSNorm,
    x: torch.Tensor,
    residual: torch.Tensor,
    quant_dtype: torch.dtype,
    group_size: list[int],
    label: str,
    sub_label: str,
    fn: Callable,
    description: str,
) -> TMeasurement:
    min_run_time = 1

    globals = {
        "rms_norm_layer": rms_norm_layer,
        "x": x,
        "residual": residual,
        "quant_dtype": quant_dtype,
        "group_size": group_size,
        "fn": fn,
    }
    return TBenchmark.Timer(
        stmt="fn(rms_norm_layer, x, residual, quant_dtype, group_size)",
        globals=globals,
        label=label,
        sub_label=sub_label,
        description=description,
    ).blocked_autorange(min_run_time=min_run_time)
```
**EN:** `bench_fn` implements a helper used by `layernorm_rms_benchmarks.py`. It mainly works with `rms_norm_layer`, `x`, `residual`, `quant_dtype`, `group_size`, ... and relies on `TBenchmark.Timer.blocked_autorange`, `TBenchmark.Timer` plus value production to move data through this part of the benchmark pipeline.
**CN:** `bench_fn` 负责实现 `layernorm_rms_benchmarks.py` 使用的辅助逻辑。 它主要处理 `rms_norm_layer`, `x`, `residual`, `quant_dtype`, `group_size`, ...，并结合 `TBenchmark.Timer.blocked_autorange`, `TBenchmark.Timer` 以及 结果返回 来完成这一段基准测试流程。

### Function `bench` (lines 174-285)
```python
def bench(params: bench_params_t, label: str, sub_label: str) -> Iterable[TMeasurement]:
    # Make inputs
    layer = RMSNorm(params.hidden_size, 1e-6).to(dtype=params.dtype)
    # Make weights
    layer.weight.data.normal_(mean=1.0, std=0.1)
    # Make inputs
    scale = 1 / params.hidden_size
    x = (
        torch.randn(
            params.num_tokens, params.hidden_size, dtype=params.dtype, device="cuda"
        )
        * scale
    )
    residual = (
        (torch.randn_like(x) * scale).to(device="cuda") if params.add_residual else None
    )

    timers = []

    # unfused int8 impl.
    timers.append(
        bench_fn(
            layer,
            x,
            residual,
            torch.int8,
            params.group_size,
            label,
            sub_label,
            unfused_int8_impl,
            "unfused_int8_impl",
        )
    )

    # ... omitted for brevity ...
            fused_groupwise_impl,
            "fused_groupwise_fp8_impl",
        )
    )

    print_timers(timers)

    return timers
```
**EN:** `bench` implements a helper used by `layernorm_rms_benchmarks.py`. It mainly works with `params`, `label`, `sub_label` and relies on `RMSNorm.to`, `RMSNorm`, `layer.weight.data.normal_`, `torch.randn`, `to`, `torch.randn_like` plus value production to move data through this part of the benchmark pipeline.
**CN:** `bench` 负责实现 `layernorm_rms_benchmarks.py` 使用的辅助逻辑。 它主要处理 `params`, `label`, `sub_label`，并结合 `RMSNorm.to`, `RMSNorm`, `layer.weight.data.normal_`, `torch.randn`, `to`, `torch.randn_like` 以及 结果返回 来完成这一段基准测试流程。

### Function `print_timers` (lines 290-292)
```python
def print_timers(timers: Iterable[TMeasurement]):
    compare = TBenchmark.Compare(timers)
    compare.print()
```
**EN:** `print_timers` formats results for display or export. It mainly works with `timers` and relies on `TBenchmark.Compare`, `compare.print` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `print_timers` 负责格式化结果以便展示或导出。 它主要处理 `timers`，并结合 `TBenchmark.Compare`, `compare.print` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Function `main` (lines 296-308)
```python
def main():
    torch.set_default_device("cuda")
    bench_params = get_bench_params()

    timers = []
    for bp in tqdm(bench_params):
        timers.extend(bench(bp, "rms-norm-dynamic-per-token-quant", bp.description()))
    print_timers(timers)

    # pickle all the results
    timestamp = int(time.time())
    with open(f"rms_norm_dpt_quant-{timestamp}.pkl", "wb") as f:
        pkl.dump(timers, f)
```
**EN:** `main` serves as the main entry point for this script. It mainly works with no explicit parameters and relies on `torch.set_default_device`, `get_bench_params`, `tqdm`, `timers.extend`, `bench`, `bp.description` plus iteration and context management to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 无显式参数，并结合 `torch.set_default_device`, `get_bench_params`, `tqdm`, `timers.extend`, `bench`, `bp.description` 以及 循环迭代、上下文管理 来完成这一段基准测试流程。

### Entry point (lines 311-312)
```python
if __name__ == "__main__":
    main()
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `main` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `main` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: `pickle`, `time`, `collections.abc`, `dataclasses`, `itertools`.
- **CN:** 标准库依赖：`pickle`, `time`, `collections.abc`, `dataclasses`, `itertools`。
- **EN:** Third-party packages: `torch`, `torch.utils.benchmark`, `tqdm`.
- **CN:** 第三方依赖：`torch`, `torch.utils.benchmark`, `tqdm`。
- **EN:** Internal modules: `vllm._custom_ops`, `vllm.benchmarks.lib.utils`, `vllm.model_executor.layers.layernorm`, `vllm.model_executor.layers.quantization.utils.fp8_utils`.
- **CN:** 内部模块：`vllm._custom_ops`, `vllm.benchmarks.lib.utils`, `vllm.model_executor.layers.layernorm`, `vllm.model_executor.layers.quantization.utils.fp8_utils`。
