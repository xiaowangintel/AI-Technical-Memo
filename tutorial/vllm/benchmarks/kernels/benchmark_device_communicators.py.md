# benchmark_device_communicators.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_device_communicators.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, multimodal processing benchmarks, centered around `DEFAULT_SEQUENCE_LENGTHS`, `HIDDEN_SIZE`, `BENCHMARK_DTYPE`, `CUDA_GRAPH_CAPTURE_CYCLES`. / 实现与基准测试编排、多模态处理基准相关的逻辑，核心符号包括 `DEFAULT_SEQUENCE_LENGTHS`, `HIDDEN_SIZE`, `BENCHMARK_DTYPE`, `CUDA_GRAPH_CAPTURE_CYCLES`。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 5-20)
```python
"""
Benchmark script for device communicators:
CustomAllreduce (oneshot, twoshot), PyNcclCommunicator,
and SymmMemCommunicator (multimem, two-shot).

for NCCL symmetric memory you need to set the environment variables
NCCL_NVLS_ENABLE=1 NCCL_CUMEM_ENABLE=1 VLLM_USE_NCCL_SYMM_MEM=1, otherwise NCCL does
not use fast NVLS implementation for all reduce.

Usage:
    torchrun --nproc_per_node=<N> benchmark_device_communicators.py [options]

Example:
    torchrun --nproc_per_node=2 benchmark_device_communicators.py
    --sequence-lengths 512 1024 2048 --num-warmup 10 --num-trials 100
"""
```
**EN:** The module docstring introduces Benchmark script for device communicators: CustomAllreduce (oneshot, twoshot), PyNcclCommunicator, and SymmMemCommunicator (multimem, two-shot). It gives readers the context needed before the implementation details start.
**CN:** 模块文档字符串先说明了 Benchmark script for device communicators: CustomAllreduce (oneshot, twoshot), PyNcclCommunicator, and SymmMemCommunicator (multimem, two-shot) 的背景与用途，在进入实现细节前先建立阅读上下文。

### Imports (lines 22-45)
```python
import json
import os
import time
from collections.abc import Callable
from contextlib import nullcontext

import torch
import torch.distributed as dist
from torch.distributed import ProcessGroup

from vllm.distributed.device_communicators.custom_all_reduce import CustomAllreduce
from vllm.distributed.device_communicators.flashinfer_all_reduce import (
    FlashInferAllReduce,
)
from vllm.distributed.device_communicators.pynccl import (
    PyNcclCommunicator,
    register_nccl_symmetric_ops,
)
from vllm.distributed.device_communicators.pynccl_allocator import (
    set_graph_pool_id,
)
from vllm.distributed.device_communicators.symm_mem import SymmMemCommunicator
from vllm.logger import init_logger
from vllm.utils.argparse_utils import FlexibleArgumentParser
```
**EN:** This block gathers standard-library helpers such as `json`, `os`, `time`, `collections.abc`, `contextlib`; third-party packages such as `torch`, `torch.distributed`; project-local modules such as `vllm.distributed.device_communicators.custom_all_reduce`, `vllm.distributed.device_communicators.flashinfer_all_reduce`, `vllm.distributed.device_communicators.pynccl`, `vllm.distributed.device_communicators.pynccl_allocator`, `vllm.distributed.device_communicators.symm_mem`, `vllm.logger`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `json`, `os`, `time`, `collections.abc`, `contextlib`；第三方依赖，如 `torch`, `torch.distributed`；项目内部模块，如 `vllm.distributed.device_communicators.custom_all_reduce`, `vllm.distributed.device_communicators.flashinfer_all_reduce`, `vllm.distributed.device_communicators.pynccl`, `vllm.distributed.device_communicators.pynccl_allocator`, `vllm.distributed.device_communicators.symm_mem`, `vllm.logger`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 47-57)
```python
logger = init_logger(__name__)

# Default sequence lengths to benchmark
DEFAULT_SEQUENCE_LENGTHS = [16, 64, 128, 512, 1024, 2048, 4096, 8192]

# Fixed hidden size and dtype for all benchmarks
HIDDEN_SIZE = 8192
BENCHMARK_DTYPE = torch.bfloat16

# CUDA graph settings
CUDA_GRAPH_CAPTURE_CYCLES = 10
```
**EN:** This top-level block prepares shared state such as `logger`, `DEFAULT_SEQUENCE_LENGTHS`, `HIDDEN_SIZE`, `BENCHMARK_DTYPE`, `CUDA_GRAPH_CAPTURE_CYCLES`. It uses `init_logger` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `logger`, `DEFAULT_SEQUENCE_LENGTHS`, `HIDDEN_SIZE`, `BENCHMARK_DTYPE`, `CUDA_GRAPH_CAPTURE_CYCLES`。它借助 `init_logger` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Class `CommunicatorBenchmark` (lines 60-386)
```python
class CommunicatorBenchmark:
    """Benchmark class for testing device communicators."""

    def __init__(
        self,
        rank: int,
        world_size: int,
        device: torch.device,
        cpu_group: ProcessGroup,
        sequence_lengths: list[int],
    ):
        self.rank = rank
        self.world_size = world_size
        self.device = device
        self.cpu_group = cpu_group

        # Calculate max_size_override based on largest sequence length
        max_seq_len = max(sequence_lengths)
        max_tensor_elements = max_seq_len * HIDDEN_SIZE
        self.max_size_override = max_tensor_elements * BENCHMARK_DTYPE.itemsize + 1

        # Initialize communicators
        self.custom_allreduce = None
        self.pynccl_comm = None
        self.symm_mem_comm = None
        self.symm_mem_comm_multimem = None
        self.symm_mem_comm_two_shot = None
        self.fi_ar_comm = None

        self._init_communicators()

    def _init_communicators(self):
        """Initialize all available communicators."""
        try:
    # ... omitted for brevity ...
                (end_time - start_time) / num_trials / CUDA_GRAPH_CAPTURE_CYCLES * 1000
            )

        except Exception as e:
            logger.error("CUDA graph benchmark failed: %s", e)
            raise RuntimeError(
                f"CUDA graph benchmark failed for communicator: {e}"
            ) from e
```
**EN:** Class `CommunicatorBenchmark` is the main object-oriented wrapper for this module. Its docstring says: Benchmark class for testing device communicators. It extends `object` and organizes behavior through `__init__`, `_init_communicators`, `benchmark_allreduce`, `benchmark_allreduce_single`.
**CN:** 类 `CommunicatorBenchmark` 是该模块中的主要面向对象封装。文档字符串指出：Benchmark class for testing device communicators。它继承自 `object`，并通过 `__init__`, `_init_communicators`, `benchmark_allreduce`, `benchmark_allreduce_single` 组织行为。

### Method `CommunicatorBenchmark.__init__` (lines 63-89)
```python
    def __init__(
        self,
        rank: int,
        world_size: int,
        device: torch.device,
        cpu_group: ProcessGroup,
        sequence_lengths: list[int],
    ):
        self.rank = rank
        self.world_size = world_size
        self.device = device
        self.cpu_group = cpu_group

        # Calculate max_size_override based on largest sequence length
        max_seq_len = max(sequence_lengths)
        max_tensor_elements = max_seq_len * HIDDEN_SIZE
        self.max_size_override = max_tensor_elements * BENCHMARK_DTYPE.itemsize + 1

        # Initialize communicators
        self.custom_allreduce = None
        self.pynccl_comm = None
        self.symm_mem_comm = None
        self.symm_mem_comm_multimem = None
        self.symm_mem_comm_two_shot = None
        self.fi_ar_comm = None

        self._init_communicators()
```
**EN:** `__init__` implements a helper used by `benchmark_device_communicators.py`. It mainly works with `rank`, `world_size`, `device`, `cpu_group`, `sequence_lengths` and relies on `max`, `self._init_communicators` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `__init__` 负责实现 `benchmark_device_communicators.py` 使用的辅助逻辑。 它主要处理 `rank`, `world_size`, `device`, `cpu_group`, `sequence_lengths`，并结合 `max`, `self._init_communicators` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Method `CommunicatorBenchmark.benchmark_allreduce` (lines 184-325)
```python
    def benchmark_allreduce(
        self, sequence_length: int, num_warmup: int, num_trials: int
    ) -> dict[str, float]:
        """Benchmark allreduce operations for all available communicators."""

        results = {}

        # Define communicators with their benchmark functions
        communicators = []

        if self.custom_allreduce is not None:
            comm = self.custom_allreduce
            # CustomAllreduce one-shot
            communicators.append(
                (
                    "ca_1stage",
                    lambda t, c=comm: c.custom_all_reduce(t),
                    lambda t, c=comm: c.should_custom_ar(t),
                    comm.capture(),
                    {"VLLM_CUSTOM_ALLREDUCE_ALGO": "1stage"},
                    None,  # no destroy function
                )
            )
            # CustomAllreduce two-shot
            communicators.append(
                (
                    "ca_2stage",
                    lambda t, c=comm: c.custom_all_reduce(t),
                    lambda t, c=comm: c.should_custom_ar(t),
                    comm.capture(),
                    {"VLLM_CUSTOM_ALLREDUCE_ALGO": "2stage"},
                    None,  # no destroy function
                )
            )
        # ... omitted for brevity ...
                # Restore environment variables to their original state
                for key, original_value in saved_env.items():
                    if original_value is None:
                        os.environ.pop(key, None)
                    else:
                        os.environ[key] = original_value

        return results
```
**EN:** `benchmark_allreduce` Benchmark allreduce operations for all available communicators. It mainly works with `sequence_length`, `num_warmup`, `num_trials` and relies on `communicators.append`, `c.custom_all_reduce`, `c.should_custom_ar`, `comm.capture`, `c.all_reduce`, `nullcontext` plus iteration, branching and error handling to move data through this part of the benchmark pipeline.
**CN:** `benchmark_allreduce` 的职责是：Benchmark allreduce operations for all available communicators。 它主要处理 `sequence_length`, `num_warmup`, `num_trials`，并结合 `communicators.append`, `c.custom_all_reduce`, `c.should_custom_ar`, `comm.capture`, `c.all_reduce`, `nullcontext` 以及 循环迭代、条件分支、异常处理 来完成这一段基准测试流程。

### Method `CommunicatorBenchmark.benchmark_allreduce_single` (lines 327-386)
```python
    def benchmark_allreduce_single(
        self,
        sequence_length: int,
        allreduce_fn: Callable[[torch.Tensor], torch.Tensor | None],
        should_use_fn: Callable[[torch.Tensor], bool],
        context,
        num_warmup: int,
        num_trials: int,
    ) -> float | None:
        """Benchmark method with CUDA graph optimization."""
        try:
            # Create test tensor (2D: sequence_length x hidden_size)
            tensor = torch.randn(
                sequence_length, HIDDEN_SIZE, dtype=BENCHMARK_DTYPE, device=self.device
            )
            if not should_use_fn(tensor):
                return None

            torch.accelerator.synchronize()
            stream = torch.cuda.Stream()
            with torch.cuda.stream(stream):
                graph_input = tensor.clone()

                # Warmup before capture
                for _ in range(3):
                    allreduce_fn(graph_input)

                # Capture the graph using context manager
                with context:
                    graph = torch.cuda.CUDAGraph()
                    graph_pool = torch.cuda.graph_pool_handle()
                    set_graph_pool_id(graph_pool)
                    with torch.cuda.graph(graph, pool=graph_pool, stream=stream):
                        for _ in range(CUDA_GRAPH_CAPTURE_CYCLES):
        # ... omitted for brevity ...
                (end_time - start_time) / num_trials / CUDA_GRAPH_CAPTURE_CYCLES * 1000
            )

        except Exception as e:
            logger.error("CUDA graph benchmark failed: %s", e)
            raise RuntimeError(
                f"CUDA graph benchmark failed for communicator: {e}"
            ) from e
```
**EN:** `benchmark_allreduce_single` Benchmark method with CUDA graph optimization. It mainly works with `sequence_length`, `allreduce_fn`, `should_use_fn`, `context`, `num_warmup`, ... and relies on `torch.randn`, `should_use_fn`, `torch.accelerator.synchronize`, `torch.cuda.Stream`, `torch.cuda.stream`, `tensor.clone` plus iteration, branching, error handling and context management to move data through this part of the benchmark pipeline.
**CN:** `benchmark_allreduce_single` 的职责是：Benchmark method with CUDA graph optimization。 它主要处理 `sequence_length`, `allreduce_fn`, `should_use_fn`, `context`, `num_warmup`, ...，并结合 `torch.randn`, `should_use_fn`, `torch.accelerator.synchronize`, `torch.cuda.Stream`, `torch.cuda.stream`, `tensor.clone` 以及 循环迭代、条件分支、异常处理、上下文管理 来完成这一段基准测试流程。

### Function `_calculate_speedup_info` (lines 389-404)
```python
def _calculate_speedup_info(comm_results: dict[str, float]) -> str:
    """Calculate speedup information for a single tensor size."""
    if not comm_results:
        return "N/A"

    # Find the fastest communicator
    fastest_comm = min(comm_results.keys(), key=lambda k: comm_results[k])
    fastest_time = comm_results[fastest_comm]

    # Calculate speedup vs PyNccl if available
    if "pynccl" in comm_results:
        pynccl_time = comm_results["pynccl"]
        speedup = pynccl_time / fastest_time
        return f"{fastest_comm} ({speedup:.2f}x)"
    else:
        return f"{fastest_comm} (N/A)"
```
**EN:** `_calculate_speedup_info` Calculate speedup information for a single tensor size. It mainly works with `comm_results` and relies on `min`, `comm_results.keys` plus branching to move data through this part of the benchmark pipeline.
**CN:** `_calculate_speedup_info` 的职责是：Calculate speedup information for a single tensor size。 它主要处理 `comm_results`，并结合 `min`, `comm_results.keys` 以及 条件分支 来完成这一段基准测试流程。

### Function `print_results` (lines 407-464)
```python
def print_results(
    results: dict[str, dict[str, float]], sequence_lengths: list[int], world_size: int
):
    """Print benchmark results in a formatted table."""

    print(f"\n{'=' * 130}")
    print("Device Communicator Benchmark Results")
    print(
        f"World Size: {world_size}, Data Type: {BENCHMARK_DTYPE}, "
        f"Hidden Size: {HIDDEN_SIZE}"
    )
    print(f"{'=' * 130}")

    # Get all communicator names
    all_comms = set()
    for size_results in results.values():
        all_comms.update(size_results.keys())

    all_comms = sorted(list(all_comms))

    # Print header
    header = f"{'Tensor Shape':<20}{'Tensor Size':<15}"
    for comm in all_comms:
        header += f"{comm:<20}"
    header += f"{'Best (Speedup vs PyNccl)':<30}"
    print(header)
    print("-" * len(header))

    # Print results for each sequence length
    for seq_len in sequence_lengths:
        if seq_len in results:
            # Calculate tensor size in elements and bytes
            tensor_elements = seq_len * HIDDEN_SIZE
            tensor_bytes = tensor_elements * BENCHMARK_DTYPE.itemsize
    # ... omitted for brevity ...
            speedup_info = _calculate_speedup_info(results[seq_len])
            row += f"{speedup_info:<30}"

            print(row)

    print(f"{'=' * 130}")
    print("All times are in milliseconds (ms) per allreduce operation")
    print("Speedup column shows: fastest_algorithm (speedup_vs_pynccl)")
```
**EN:** `print_results` Print benchmark results in a formatted table. It mainly works with `results`, `sequence_lengths`, `world_size` and relies on `print`, `set`, `results.values`, `all_comms.update`, `size_results.keys`, `sorted` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `print_results` 的职责是：Print benchmark results in a formatted table。 它主要处理 `results`, `sequence_lengths`, `world_size`，并结合 `print`, `set`, `results.values`, `all_comms.update`, `size_results.keys`, `sorted` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `main` (lines 467-567)
```python
def main():
    parser = FlexibleArgumentParser(description="Benchmark device communicators")

    parser.add_argument(
        "--sequence-lengths",
        type=int,
        nargs="+",
        default=DEFAULT_SEQUENCE_LENGTHS,
        help="Sequence lengths to benchmark (tensor shape: seq_len x hidden_size)",
    )

    parser.add_argument(
        "--num-warmup", type=int, default=5, help="Number of warmup iterations"
    )

    parser.add_argument(
        "--num-trials", type=int, default=50, help="Number of benchmark trials"
    )

    parser.add_argument("--output-json", type=str, help="Output results to JSON file")

    args = parser.parse_args()

    # Initialize distributed
    if not dist.is_initialized():
        dist.init_process_group(backend="gloo")
    rank = dist.get_rank()
    world_size = dist.get_world_size()

    # Set device
    device = torch.device(f"cuda:{rank}")
    torch.accelerator.set_device_index(device)

    # Get CPU process group
    # ... omitted for brevity ...
            with open(args.output_json, "w") as f:
                json.dump(output_data, f, indent=2)

            logger.info("Results saved to %s", args.output_json)

    # Cleanup
    if cpu_group != dist.group.WORLD:
        dist.destroy_process_group(cpu_group)
```
**EN:** `main` serves as the main entry point for this script. It mainly works with no explicit parameters and relies on `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `dist.is_initialized`, `dist.init_process_group`, `dist.get_rank` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 无显式参数，并结合 `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `dist.is_initialized`, `dist.init_process_group`, `dist.get_rank` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Entry point (lines 570-571)
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
- **EN:** Distributed execution: the code coordinates work across processes, devices, or Ray workers.
- **CN:** 分布式执行：代码会在多个进程、设备或 Ray worker 之间协调工作。
- **EN:** Performance metrics: the script emphasizes latency/throughput measurement rather than model quality.
- **CN:** 性能指标：该脚本更强调延迟/吞吐量测量，而不是模型质量。

## Dependencies / 依赖关系
- **EN:** Standard library: `json`, `os`, `time`, `collections.abc`, `contextlib`.
- **CN:** 标准库依赖：`json`, `os`, `time`, `collections.abc`, `contextlib`。
- **EN:** Third-party packages: `torch`, `torch.distributed`.
- **CN:** 第三方依赖：`torch`, `torch.distributed`。
- **EN:** Internal modules: `vllm.distributed.device_communicators.custom_all_reduce`, `vllm.distributed.device_communicators.flashinfer_all_reduce`, `vllm.distributed.device_communicators.pynccl`, `vllm.distributed.device_communicators.pynccl_allocator`, `vllm.distributed.device_communicators.symm_mem`, `vllm.logger`.
- **CN:** 内部模块：`vllm.distributed.device_communicators.custom_all_reduce`, `vllm.distributed.device_communicators.flashinfer_all_reduce`, `vllm.distributed.device_communicators.pynccl`, `vllm.distributed.device_communicators.pynccl_allocator`, `vllm.distributed.device_communicators.symm_mem`, `vllm.logger`。
