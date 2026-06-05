# sdpa.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `benchmarks/transformer/sdpa.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements Python-side logic used for benchmarking, tooling, validation, or repository automation. Backend or dtype coverage is a visible axis of variation in the implementation.
- **Purpose (CN)**: 实现用于基准、工具链、校验或仓库自动化的 Python 侧逻辑。 后端或 dtype 覆盖是该实现中明显的变化维度。

## Line-by-Line Analysis / 逐行分析

### Lines 1-9 / 第 1-9 行

```python
import itertools
from collections import defaultdict
from collections.abc import Callable
from contextlib import nullcontext
from dataclasses import asdict, dataclass

from tabulate import tabulate
from tqdm import tqdm

```

- **EN:** This range mainly pulls in the imports/includes that the later benchmark or integration logic depends on.
- **CN:** 这一段主要引入后续基准或集成逻辑依赖的 import/include。

### Lines 10-20 / 第 10-20 行

```python
import torch
import torch.utils.benchmark as benchmark
from torch._inductor.utils import do_bench_using_profiling
from torch.nn.attention import (
    activate_flash_attention_impl,
    restore_flash_attention_impl,
    sdpa_kernel,
    SDPBackend,
)
from torch.nn.functional import scaled_dot_product_attention

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 21-29 / 第 21-29 行

```python

def benchmark_cuda_function_in_microseconds(func: Callable, *args, **kwargs) -> float:
    """Thin wrapper around do_bench_using_profiling"""

    def no_args():
        func(*args, **kwargs)

    time = do_bench_using_profiling(no_args)
    return time * 1e3
```

- **EN:** Important local symbols in this block include benchmark_cuda_function_in_microseconds, no_args.
- **CN:** 该代码块中的重要局部符号包括 benchmark_cuda_function_in_microseconds、no_args。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 30-40 / 第 30-40 行

```python


def benchmark_torch_function_in_microseconds(func: Callable, *args, **kwargs) -> float:
    # warmup
    for _ in range(5):
        func(*args, **kwargs)
    t0 = benchmark.Timer(
        stmt="func(*args, **kwargs)",
        globals={"args": args, "kwargs": kwargs, "func": func},
    )
    return t0.adaptive_autorange(min_run_time=0.1).median * 1e6
```

- **EN:** Important local symbols in this block include benchmark_torch_function_in_microseconds.
- **CN:** 该代码块中的重要局部符号包括 benchmark_torch_function_in_microseconds。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 41-55 / 第 41-55 行

```python


@dataclass(frozen=True)
class ExperimentConfig:
    batch_size: int
    num_heads: int
    q_seq_len: int
    kv_seq_len: int
    embed_dim: int
    is_causal: bool
    dtype: torch.dtype
    backend: SDPBackend
    flash_impl: str | None = None  # None/"FA2" for default, "FA3", or "FA4"
    device: torch.device = torch.device("cuda")

```

- **EN:** Important local symbols in this block include ExperimentConfig.
- **CN:** 该代码块中的重要局部符号包括 ExperimentConfig。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 56-63 / 第 56-63 行

```python
    @property
    def head_dim(self) -> int:
        return self.embed_dim // self.num_heads

    def asdict(self):
        dict_obj = asdict(self)
        dict_obj["head_dim"] = self.head_dim
        return dict_obj
```

- **EN:** Important local symbols in this block include head_dim, asdict.
- **CN:** 该代码块中的重要局部符号包括 head_dim、asdict。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 64-72 / 第 64-72 行

```python


@dataclass(frozen=True)
class ExperimentResults:
    forward_time: float  # microseconds
    backward_time: float  # microseconds
    forward_tflops: float
    backward_tflops: float

```

- **EN:** Important local symbols in this block include ExperimentResults.
- **CN:** 该代码块中的重要局部符号包括 ExperimentResults。

### Lines 73-81 / 第 73-81 行

```python
    def asdict(self):
        return asdict(self)


@dataclass(frozen=True)
class Experiment:
    config: ExperimentConfig
    results: ExperimentResults

```

- **EN:** Important local symbols in this block include Experiment, asdict.
- **CN:** 该代码块中的重要局部符号包括 Experiment、asdict。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 82-96 / 第 82-96 行

```python
    def asdict(self):
        dict1 = self.config.asdict()
        dict2 = self.results.asdict()
        return {**dict1, **dict2}


def calculate_tflops(
    config: ExperimentConfig,
    time_us: float,
    is_backward: bool = False,
    sparsity: float = 0.0,
) -> float:
    """
    Calculate TFLOPS for scaled dot product attention.

```

- **EN:** Important local symbols in this block include asdict, calculate_tflops.
- **CN:** 该代码块中的重要局部符号包括 asdict、calculate_tflops。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 97-111 / 第 97-111 行

```python
    Parameters:
    - config: The experiment configuration
    - time_us: The execution time in microseconds
    - is_backward: Whether to calculate for backward pass (includes gradient computation)
    - sparsity: Sparsity factor between 0.0 and 1.0, where 0.0 means no sparsity and 1.0 means fully sparse

    Returns:
    - TFLOPS value
    """
    B = config.batch_size
    H = config.num_heads
    M = config.q_seq_len
    N = config.kv_seq_len
    D = config.head_dim

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 112-123 / 第 112-123 行

```python
    # Calculate density factor (1.0 - sparsity)
    density = 1.0 - sparsity

    # Forward pass FLOPs
    qk_flops = (
        M * N * D * 2
    )  # Q*K^T matmul: (M,D) @ (D,N) with 2 FLOPs per multiply-add
    softmax_flops = M * N * 2  # Softmax operations (exp and div)
    av_flops = (
        M * N * D * 2
    )  # Attention @ V: (M,N) @ (N,D) with 2 FLOPs per multiply-add

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 124-132 / 第 124-132 行

```python
    total_flops = B * H * (qk_flops + softmax_flops + av_flops)

    # Apply density factor to account for sparsity
    total_flops *= density

    # For backward pass flash uses 2.5x more flops will use this
    if is_backward:
        total_flops *= 2.5

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 133-148 / 第 133-148 行

```python
    # Convert to TFLOPS: flops / (time_us * 1e-6) / 1e12
    tflops = total_flops / (time_us * 1e-6) / 1e12

    return tflops


def get_input(
    config: ExperimentConfig,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    q = torch.randn(
        (config.batch_size, config.num_heads, config.q_seq_len, config.head_dim),
        dtype=config.dtype,
        device=config.device,
        requires_grad=True,
    )
    k = torch.randn(
```

- **EN:** Important local symbols in this block include get_input.
- **CN:** 该代码块中的重要局部符号包括 get_input。
- **EN:** Tensor construction or buffer wrapping happens here, so input layout and data preparation matter to the surrounding logic.
- **CN:** 这里发生了张量构造或缓冲区封装，因此输入布局与数据准备对周边逻辑很重要。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 149-160 / 第 149-160 行

```python
        (config.batch_size, config.num_heads, config.kv_seq_len, config.head_dim),
        dtype=config.dtype,
        device=config.device,
        requires_grad=True,
    )
    v = torch.randn(
        (config.batch_size, config.num_heads, config.kv_seq_len, config.head_dim),
        dtype=config.dtype,
        device=config.device,
        requires_grad=True,
    )
    return q, k, v
```

- **EN:** Tensor construction or buffer wrapping happens here, so input layout and data preparation matter to the surrounding logic.
- **CN:** 这里发生了张量构造或缓冲区封装，因此输入布局与数据准备对周边逻辑很重要。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 161-169 / 第 161-169 行

```python


def run_single_experiment(config: ExperimentConfig) -> ExperimentResults:
    q, k, v = get_input(config)
    is_causal = config.is_causal
    context = (
        sdpa_kernel(config.backend) if config.backend is not None else nullcontext()
    )

```

- **EN:** Important local symbols in this block include run_single_experiment.
- **CN:** 该代码块中的重要局部符号包括 run_single_experiment。

### Lines 170-182 / 第 170-182 行

```python
    # Activate flash attention implementation if specified
    if config.backend is SDPBackend.FLASH_ATTENTION and config.flash_impl in (
        "FA3",
        "FA4",
    ):
        try:
            activate_flash_attention_impl(config.flash_impl)
        except ImportError as e:
            raise RuntimeError(
                f"Failed to activate {config.flash_impl}: {e}\n"
                f"Please install the required flash attention library or run with default configuration (without --flash_test)."
            ) from e

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 183-198 / 第 183-198 行

```python
    try:
        with context:
            forward_time = benchmark_cuda_function_in_microseconds(
                scaled_dot_product_attention,
                q,
                k,
                v,
                is_causal=is_causal,
                attn_mask=None,
            )
            out_torch = scaled_dot_product_attention(
                q, k, v, is_causal=is_causal, attn_mask=None
            )
            d_out = torch.randn_like(out_torch)
            backward_time = benchmark_cuda_function_in_microseconds(
                out_torch.backward, d_out, retain_graph=True
```

- **EN:** Tensor construction or buffer wrapping happens here, so input layout and data preparation matter to the surrounding logic.
- **CN:** 这里发生了张量构造或缓冲区封装，因此输入布局与数据准备对周边逻辑很重要。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 199-207 / 第 199-207 行

```python
            )
    finally:
        # Restore default FA2 implementation if we activated a different one
        if config.backend is SDPBackend.FLASH_ATTENTION and config.flash_impl in (
            "FA3",
            "FA4",
        ):
            restore_flash_attention_impl()

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 208-215 / 第 208-215 行

```python
    # Calculate TFLOPS for forward and backward passes
    sparsity = 0.5 if is_causal else 0.0
    forward_tflops = calculate_tflops(config, forward_time, sparsity=sparsity)
    backward_tflops = calculate_tflops(
        config, backward_time, is_backward=True, sparsity=sparsity
    )

    return ExperimentResults(
```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 216-223 / 第 216-223 行

```python
        forward_time=forward_time,
        backward_time=backward_time,
        forward_tflops=forward_tflops,
        backward_tflops=backward_tflops,
    )


def print_results(experiments: list[Experiment]):
```

- **EN:** Important local symbols in this block include print_results.
- **CN:** 该代码块中的重要局部符号包括 print_results。

### Lines 224-231 / 第 224-231 行

```python
    table_data = defaultdict(list)
    for experiment in experiments:
        for key, value in experiment.asdict().items():
            table_data[key].append(value)
    del table_data["device"]
    if table_data["backend"][0] is None:
        del table_data["backend"]
    if table_data["flash_impl"][0] is None:
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 232-246 / 第 232-246 行

```python
        del table_data["flash_impl"]
    print(tabulate(table_data, headers="keys", tablefmt="pretty", floatfmt=".3f"))


def write_results_to_csv(
    experiments: list[Experiment], output_dir: str = "benchmark_results"
):
    """
    Write experiment results to a CSV file in the specified directory.
    The filename includes a timestamp for uniqueness.
    """
    import csv
    import os
    from datetime import datetime

```

- **EN:** Important local symbols in this block include write_results_to_csv.
- **CN:** 该代码块中的重要局部符号包括 write_results_to_csv。

### Lines 247-255 / 第 247-255 行

```python
    # Create output directory if it doesn't exist
    os.makedirs(output_dir, exist_ok=True)

    # Generate filename with timestamp
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    filename = os.path.join(output_dir, f"benchmark_results_{timestamp}.csv")

    # Get all fields from the first experiment
    if not experiments:
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 256-266 / 第 256-266 行

```python
        return

    fieldnames = list(experiments[0].asdict().keys())
    if "device" in fieldnames:
        fieldnames.remove("device")  # Remove device field as it's always cuda

    # Write results to CSV
    with open(filename, "w", newline="") as csvfile:
        writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
        writer.writeheader()
        for experiment in experiments:
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 267-274 / 第 267-274 行

```python
            row = experiment.asdict()
            if "device" in row:
                del row["device"]  # Remove device field
            writer.writerow(row)

    print(f"Results written to: {filename}")


```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 275-286 / 第 275-286 行

```python
def generate_experiment_configs() -> list[ExperimentConfig]:
    batch_sizes = [1, 8, 16]
    num_heads = [16]
    q_kv_seq_lens = [(128, 128), (256, 256), (512, 512), (1024, 1024), (8192, 8192)]
    embed_dims = [2048]
    backends = [None]  # If set to None, all backends are enabled
    dtypes = [
        torch.bfloat16,
    ]
    is_causal = [True, False]
    all_configs = []
    for (
```

- **EN:** Important local symbols in this block include generate_experiment_configs.
- **CN:** 该代码块中的重要局部符号包括 generate_experiment_configs。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Looping logic iterates over benchmark dimensions, shape metadata, or container elements.
- **CN:** 循环逻辑会遍历 benchmark 维度、形状元数据或容器元素。

### Lines 287-302 / 第 287-302 行

```python
        bsz,
        heads,
        (q_seq_len, kv_seq_len),
        embed_dim,
        causal,
        dtype,
        backend,
    ) in itertools.product(
        batch_sizes, num_heads, q_kv_seq_lens, embed_dims, is_causal, dtypes, backends
    ):
        all_configs.append(
            ExperimentConfig(
                batch_size=bsz,
                num_heads=heads,
                q_seq_len=q_seq_len,
                kv_seq_len=kv_seq_len,
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 303-310 / 第 303-310 行

```python
                embed_dim=embed_dim,
                is_causal=causal,
                dtype=dtype,
                backend=backend,
            )
        )

    return all_configs
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 311-326 / 第 311-326 行

```python


def generate_experiment_configs_with_flash() -> list[ExperimentConfig]:
    batch_sizes = [1, 8, 16]
    num_heads = [16]
    q_kv_seq_lens = [(128, 128), (256, 256), (512, 512), (1024, 1024), (8192, 8192)]
    embed_dims = [2048]
    backends = [
        SDPBackend.FLASH_ATTENTION,
    ]
    dtypes = [
        torch.bfloat16,
    ]
    is_causal = [True, False]
    # FA2 (default), "FA3", "FA4" for the alternative implementations
    flash_impls = ["FA2", "FA3"]
```

- **EN:** Important local symbols in this block include generate_experiment_configs_with_flash.
- **CN:** 该代码块中的重要局部符号包括 generate_experiment_configs_with_flash。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 327-342 / 第 327-342 行

```python
    all_configs = []
    for (
        bsz,
        heads,
        (q_seq_len, kv_seq_len),
        embed_dim,
        causal,
        dtype,
        backend,
        flash_impl,
    ) in itertools.product(
        batch_sizes,
        num_heads,
        q_kv_seq_lens,
        embed_dims,
        is_causal,
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Looping logic iterates over benchmark dimensions, shape metadata, or container elements.
- **CN:** 循环逻辑会遍历 benchmark 维度、形状元数据或容器元素。

### Lines 343-358 / 第 343-358 行

```python
        dtypes,
        backends,
        flash_impls,
    ):
        all_configs.append(
            ExperimentConfig(
                batch_size=bsz,
                num_heads=heads,
                q_seq_len=q_seq_len,
                kv_seq_len=kv_seq_len,
                embed_dim=embed_dim,
                is_causal=causal,
                dtype=dtype,
                backend=backend,
                flash_impl=flash_impl,
            )
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 359-366 / 第 359-366 行

```python
        )

    return all_configs


def main():
    import argparse

```

- **EN:** Important local symbols in this block include main.
- **CN:** 该代码块中的重要局部符号包括 main。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 367-374 / 第 367-374 行

```python
    parser = argparse.ArgumentParser(description="SDPA benchmark runner")
    parser.add_argument(
        "--flash_test",
        action="store_true",
        help="Use flash attention configs (generate_experiment_configs_with_flash)",
    )
    args = parser.parse_args()

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 375-382 / 第 375-382 行

```python
    seed = 123
    torch.manual_seed(seed)
    results = []
    if args.flash_test:
        configs = generate_experiment_configs_with_flash()
    else:
        configs = generate_experiment_configs()
    for config in tqdm(configs):
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 383-390 / 第 383-390 行

```python
        results.append(Experiment(config, run_single_experiment(config)))

    print_results(results)
    write_results_to_csv(results, "../benchmark_results")


if __name__ == "__main__":
    main()
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

## Key Concepts / 关键概念

- **Benchmark infrastructure** — 基准设施
- **PyTorch tensor operations** — 覆盖或配置具体的 PyTorch 张量/算子行为。
- **Data type coverage** — 体现文件如何在不同张量 dtype 间覆盖行为差异。
- **Device coverage** — 突出 CPU/CUDA/移动端等后端相关执行路径。
- **Benchmark workflow** — 记录或实现性能测量场景。
- **Representative symbols: ExperimentConfig, ExperimentResults, Experiment, benchmark_cuda_function_in_microseconds, no_args, benchmark_torch_function_in_microseconds, head_dim, asdict** — 代表性符号：ExperimentConfig、ExperimentResults、Experiment、benchmark_cuda_function_in_microseconds、no_args、benchmark_torch_function_in_microseconds、head_dim、asdict

## Dependencies / 依赖关系

- `itertools`
- `collections`
- `collections.abc`
- `contextlib`
- `dataclasses`
- `tabulate`
- `tqdm`
- `torch`
- `torch.utils.benchmark`
- `torch._inductor.utils`
- `torch.nn.attention`
- `torch.nn.functional`
- `csv`
- `os`
- `datetime`
- `argparse`
