# benchmark_activation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_activation.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, centered around `benchmark_activation`. / 实现与基准测试编排相关的逻辑，核心符号包括 `benchmark_activation`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 5-14)
```python
import itertools

import torch

import vllm.model_executor.layers.activation  # noqa F401
from vllm.benchmarks.lib.utils import default_vllm_config
from vllm.model_executor.custom_op import op_registry
from vllm.triton_utils import triton
from vllm.utils.argparse_utils import FlexibleArgumentParser
from vllm.utils.torch_utils import STR_DTYPE_TO_TORCH_DTYPE, set_random_seed
```
**EN:** This block gathers standard-library helpers such as `itertools`; third-party packages such as `torch`; project-local modules such as `vllm.model_executor.layers.activation`, `vllm.benchmarks.lib.utils`, `vllm.model_executor.custom_op`, `vllm.triton_utils`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `itertools`；第三方依赖，如 `torch`；项目内部模块，如 `vllm.model_executor.layers.activation`, `vllm.benchmarks.lib.utils`, `vllm.model_executor.custom_op`, `vllm.triton_utils`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 16-19)
```python
batch_size_range = [1, 16, 128]
seq_len_range = [1, 16, 64, 1024, 4096]
intermediate_size = [3072, 9728, 12288]
configs = list(itertools.product(batch_size_range, seq_len_range, intermediate_size))
```
**EN:** This top-level block prepares shared state such as `batch_size_range`, `seq_len_range`, `intermediate_size`, `configs`. It uses `list`, `itertools.product` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `batch_size_range`, `seq_len_range`, `intermediate_size`, `configs`。它借助 `list`, `itertools.product` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `benchmark_activation` (lines 23-58)
```python
def benchmark_activation(
    batch_size: int,
    seq_len: int,
    intermediate_size: int,
    provider: str,
    func_name: str,
    dtype: torch.dtype,
):
    device = "cuda"
    num_tokens = batch_size * seq_len
    dim = intermediate_size
    set_random_seed(42)
    torch.set_default_device(device)

    if func_name == "gelu_and_mul":
        layer = op_registry[func_name](approximate="none")
    elif func_name == "gelu_and_mul_tanh":
        layer = op_registry["gelu_and_mul"](approximate="tanh")
    elif func_name == "fatrelu_and_mul":
        threshold = 0.5
        layer = op_registry[func_name](threshold)
    else:
        layer = op_registry[func_name]()

    x = torch.randn(num_tokens, dim, dtype=dtype, device=device)
    compiled_layer = torch.compile(layer.forward_native)

    if provider == "custom":
        fn = lambda: layer(x)
    elif provider == "compiled":
        fn = lambda: compiled_layer(x)

    ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
        fn, quantiles=[0.5, 0.2, 0.8]
    )
    return ms, max_ms, min_ms
```
**EN:** `benchmark_activation` coordinates or measures benchmark orchestration. It mainly works with `batch_size`, `seq_len`, `intermediate_size`, `provider`, `func_name`, ... and relies on `set_random_seed`, `torch.set_default_device`, `op_registry`, `torch.randn`, `torch.compile`, `layer` plus branching to move data through this part of the benchmark pipeline.
**CN:** `benchmark_activation` 负责协调或测量基准测试编排。 它主要处理 `batch_size`, `seq_len`, `intermediate_size`, `provider`, `func_name`, ...，并结合 `set_random_seed`, `torch.set_default_device`, `op_registry`, `torch.randn`, `torch.compile`, `layer` 以及 条件分支 来完成这一段基准测试流程。

### Entry point (lines 61-106)
```python
if __name__ == "__main__":
    parser = FlexibleArgumentParser(description="Benchmark the custom activation op.")
    parser.add_argument(
        "--func-name",
        type=str,
        choices=[
            "mul_and_silu",
            "silu_and_mul",
            "gelu_and_mul",
            "gelu_and_mul_tanh",
            "fatrelu_and_mul",
            "swigluoai_and_mul",
            "gelu_new",
            "gelu_fast",
            "quick_gelu",
        ],
        default="silu_and_mul",
    )
    parser.add_argument(
        "--dtype", type=str, choices=["half", "bfloat16", "float"], default="bfloat16"
    )
    args = parser.parse_args()
    assert args

    func_name = args.func_name
    dtype = STR_DTYPE_TO_TORCH_DTYPE[args.dtype]

    perf_report = triton.testing.perf_report(
        triton.testing.Benchmark(
            x_names=["batch_size", "seq_len", "intermediate_size"],
            x_vals=configs,
            line_arg="provider",
            line_vals=["custom", "compiled"],
            line_names=["Custom OP", "Compiled"],
            styles=[("blue", "-"), ("green", "-")],
            ylabel="ms",
            plot_name=f"{func_name}-op-performance",
            args={},
        )
    )

    perf_report(
        lambda batch_size, seq_len, intermediate_size, provider: benchmark_activation(
            batch_size, seq_len, intermediate_size, provider, func_name, dtype
        )
    ).run(print_data=True)
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `triton.testing.perf_report`, `triton.testing.Benchmark`, `perf_report.run` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `triton.testing.perf_report`, `triton.testing.Benchmark`, `perf_report.run` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Result presentation: the module aggregates measurements and renders tables or plots.
- **CN:** 结果呈现：该模块会汇总测量结果并生成表格或图形。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: `itertools`.
- **CN:** 标准库依赖：`itertools`。
- **EN:** Third-party packages: `torch`.
- **CN:** 第三方依赖：`torch`。
- **EN:** Internal modules: `vllm.model_executor.layers.activation`, `vllm.benchmarks.lib.utils`, `vllm.model_executor.custom_op`, `vllm.triton_utils`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`.
- **CN:** 内部模块：`vllm.model_executor.layers.activation`, `vllm.benchmarks.lib.utils`, `vllm.model_executor.custom_op`, `vllm.triton_utils`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`。
