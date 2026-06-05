# benchmark_w8a8_block_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_w8a8_block_fp8.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, W8A8 quantization benchmarks, FP8 kernel experiments, centered around `DTYPE_MAP`, `w8a8_block_matmul`, `get_configs_compute_bound`, `get_weight_shapes`. / 实现与基准测试编排、W8A8 量化基准、FP8 内核实验相关的逻辑，核心符号包括 `DTYPE_MAP`, `w8a8_block_matmul`, `get_configs_compute_bound`, `get_weight_shapes`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 5-21)
```python
import argparse
import json
import multiprocessing as mp
import os
import time
from datetime import datetime
from typing import Any

import torch
from tqdm import tqdm

from vllm.model_executor.layers.quantization.utils.fp8_utils import (
    _w8a8_triton_block_scaled_mm,
)
from vllm.platforms import current_platform
from vllm.triton_utils import triton
from vllm.utils.argparse_utils import FlexibleArgumentParser
```
**EN:** This block gathers standard-library helpers such as `argparse`, `json`, `multiprocessing`, `os`, `time`, `datetime`; third-party packages such as `torch`, `tqdm`; project-local modules such as `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.platforms`, `vllm.triton_utils`, `vllm.utils.argparse_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`, `json`, `multiprocessing`, `os`, `time`, `datetime`；第三方依赖，如 `torch`, `tqdm`；项目内部模块，如 `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.platforms`, `vllm.triton_utils`, `vllm.utils.argparse_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 23-34)
```python
mp.set_start_method("spawn", force=True)

assert current_platform.is_cuda() or current_platform.is_rocm(), (
    "Only support tune w8a8 block fp8 kernel on CUDA/ROCm device."
)

DTYPE_MAP = {
    "float32": torch.float32,
    "float16": torch.float16,
    "half": torch.half,
    "bfloat16": torch.bfloat16,
}
```
**EN:** This top-level block prepares shared state such as `DTYPE_MAP`. It uses `mp.set_start_method`, `current_platform.is_cuda`, `current_platform.is_rocm` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `DTYPE_MAP`。它借助 `mp.set_start_method`, `current_platform.is_cuda`, `current_platform.is_rocm` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `w8a8_block_matmul` (lines 37-114)
```python
def w8a8_block_matmul(
    A: torch.Tensor,
    B: torch.Tensor,
    As: torch.Tensor,
    Bs: torch.Tensor,
    block_size: list[int],
    config: dict[str, Any],
    output_dtype: torch.dtype = torch.float16,
) -> torch.Tensor:
    """This function performs matrix multiplication with
    block-wise quantization.

    It takes two input tensors `A` and `B` with scales `As` and `Bs`.
    The output is returned in the specified `output_dtype`.

    Args:
        A: The input tensor, e.g., activation.
        B: The input tensor, e.g., weight.
        As: The per-token-group quantization scale for `A`.
        Bs: The per-block quantization scale for `B`.
        block_size: The block size for per-block quantization.
                    It should be 2-dim, e.g., [128, 128].
        output_dtype: The dtype of the returned tensor.

    Returns:
        torch.Tensor: The result of matmul.
    """
    assert len(block_size) == 2
    block_n, block_k = block_size[0], block_size[1]

    assert A.shape[-1] == B.shape[-1]
    assert A.shape[:-1] == As.shape[:-1] and A.is_contiguous()
    assert triton.cdiv(A.shape[-1], block_k) == As.shape[-1]
    M = A.numel() // A.shape[-1]
    # ... omitted for brevity ...
        As.stride(-2),
        As.stride(-1),
        Bs.stride(1),
        Bs.stride(0),
        **config,
    )

    return C
```
**EN:** `w8a8_block_matmul` This function performs matrix multiplication with block-wise quantization. It mainly works with `A`, `B`, `As`, `Bs`, `block_size`, ... and relies on `len`, `A.is_contiguous`, `triton.cdiv`, `A.numel`, `B.is_contiguous`, `A.new_empty` plus branching to move data through this part of the benchmark pipeline.
**CN:** `w8a8_block_matmul` 的职责是：This function performs matrix multiplication with block-wise quantization。 它主要处理 `A`, `B`, `As`, `Bs`, `block_size`, ...，并结合 `len`, `A.is_contiguous`, `triton.cdiv`, `A.numel`, `B.is_contiguous`, `A.new_empty` 以及 条件分支 来完成这一段基准测试流程。

### Function `get_configs_compute_bound` (lines 117-135)
```python
def get_configs_compute_bound():
    configs = []
    for num_stages in [2, 3, 4, 5]:
        for block_m in [16, 32, 64, 128, 256]:
            for block_k in [64, 128]:
                for block_n in [32, 64, 128, 256]:
                    for num_warps in [4, 8]:
                        for group_size in [1, 16, 32, 64]:
                            configs.append(
                                {
                                    "BLOCK_SIZE_M": block_m,
                                    "BLOCK_SIZE_N": block_n,
                                    "BLOCK_SIZE_K": block_k,
                                    "GROUP_SIZE_M": group_size,
                                    "num_warps": num_warps,
                                    "num_stages": num_stages,
                                }
                            )
    return configs
```
**EN:** `get_configs_compute_bound` implements a helper used by `benchmark_w8a8_block_fp8.py`. It mainly works with no explicit parameters and relies on `configs.append` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `get_configs_compute_bound` 负责实现 `benchmark_w8a8_block_fp8.py` 使用的辅助逻辑。 它主要处理 无显式参数，并结合 `configs.append` 以及 循环迭代 来完成这一段基准测试流程。

### Function `get_weight_shapes` (lines 138-171)
```python
def get_weight_shapes(tp_size):
    # NOTE(HandH1998): The weight shapes only works for DeepSeek-V3.
    # Modify them, if you tune for another different model.
    # cannot TP
    total = [
        (512 + 64, 7168),
        (2112, 7168),
        ((128 + 64) * 128, 7168),
        (128 * (128 + 128), 512),
        (7168, 16384),
        (7168, 18432),
    ]
    # N can TP
    n_tp = [
        (18432 * 2, 7168),
        ((128 + 64) * 128, 7168),
        (128 * (128 + 128), 512),
        (24576, 1536),
        (12288, 7168),
        (4096, 7168),
    ]
    # K can TP
    k_tp = [(7168, 18432), (7168, 16384), (7168, 2048)]

    weight_shapes = []
    for t in total:
        weight_shapes.append(t)
    for n_t in n_tp:
        new_t = (n_t[0] // tp_size, n_t[1])
        weight_shapes.append(new_t)
    for k_t in k_tp:
        new_t = (k_t[0], k_t[1] // tp_size)
        weight_shapes.append(new_t)
    return weight_shapes
```
**EN:** `get_weight_shapes` implements shape enumeration utilities. It mainly works with `tp_size` and relies on `weight_shapes.append` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `get_weight_shapes` 负责实现形状枚举工具。 它主要处理 `tp_size`，并结合 `weight_shapes.append` 以及 循环迭代 来完成这一段基准测试流程。

### Function `benchmark_config` (lines 174-198)
```python
def benchmark_config(
    A, B, As, Bs, block_size, config, out_dtype=torch.float16, num_iters=10
):
    def run():
        w8a8_block_matmul(A, B, As, Bs, block_size, config, out_dtype)

    torch.accelerator.synchronize()
    # JIT complication & warmup
    for _ in range(5):
        run()
    torch.accelerator.synchronize()

    start_event = torch.Event(enable_timing=True)
    end_event = torch.Event(enable_timing=True)

    latencies: list[float] = []
    for i in range(num_iters):
        torch.accelerator.synchronize()
        start_event.record()
        run()
        end_event.record()
        end_event.synchronize()
        latencies.append(start_event.elapsed_time(end_event))
    avg = sum(latencies) / (num_iters * 10) * 1000  # us
    return avg
```
**EN:** `benchmark_config` coordinates or measures benchmark orchestration. It mainly works with `A`, `B`, `As`, `Bs`, `block_size`, ... and relies on `w8a8_block_matmul`, `torch.accelerator.synchronize`, `range`, `run`, `torch.Event`, `start_event.record` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `benchmark_config` 负责协调或测量基准测试编排。 它主要处理 `A`, `B`, `As`, `Bs`, `block_size`, ...，并结合 `w8a8_block_matmul`, `torch.accelerator.synchronize`, `range`, `run`, `torch.Event`, `start_event.record` 以及 循环迭代 来完成这一段基准测试流程。

### Function `tune` (lines 201-254)
```python
def tune(M, N, K, block_size, out_dtype, search_space, input_type):
    factor_for_scale = 1e-2

    if input_type == "fp8":
        fp8_info = torch.finfo(torch.float8_e4m3fn)
        fp8_max, fp8_min = fp8_info.max, fp8_info.min

        A_fp32 = (
            (torch.rand(M, K, dtype=torch.float32, device="cuda") - 0.5) * 2 * fp8_max
        )
        A = A_fp32.clamp(min=fp8_min, max=fp8_max).to(torch.float8_e4m3fn)

        B_fp32 = (
            (torch.rand(N, K, dtype=torch.float32, device="cuda") - 0.5) * 2 * fp8_max
        )
        B = B_fp32.clamp(min=fp8_min, max=fp8_max).to(torch.float8_e4m3fn)
    else:
        raise RuntimeError("Currently, only support tune w8a8 block fp8 kernel.")

    block_n, block_k = block_size[0], block_size[1]
    n_tiles = (N + block_n - 1) // block_n
    k_tiles = (K + block_k - 1) // block_k

    As = torch.rand(M, k_tiles, dtype=torch.float32, device="cuda") * factor_for_scale
    Bs = (
        torch.rand(n_tiles, k_tiles, dtype=torch.float32, device="cuda")
        * factor_for_scale
    )

    best_config = None
    best_time = float("inf")
    for config in tqdm(search_space):
        try:
            kernel_time = benchmark_config(
    # ... omitted for brevity ...

        if kernel_time < best_time:
            best_time = kernel_time
            best_config = config
    now = datetime.now()
    print(f"{now.ctime()}] Completed tuning for batch_size={M}")
    assert best_config is not None
    return best_config
```
**EN:** `tune` implements a helper used by `benchmark_w8a8_block_fp8.py`. It mainly works with `M`, `N`, `K`, `block_size`, `out_dtype`, ... and relies on `torch.finfo`, `torch.rand`, `A_fp32.clamp.to`, `A_fp32.clamp`, `B_fp32.clamp.to`, `B_fp32.clamp` plus iteration, branching and error handling to move data through this part of the benchmark pipeline.
**CN:** `tune` 负责实现 `benchmark_w8a8_block_fp8.py` 使用的辅助逻辑。 它主要处理 `M`, `N`, `K`, `block_size`, `out_dtype`, ...，并结合 `torch.finfo`, `torch.rand`, `A_fp32.clamp.to`, `A_fp32.clamp`, `B_fp32.clamp.to`, `B_fp32.clamp` 以及 循环迭代、条件分支、异常处理 来完成这一段基准测试流程。

### Function `save_configs` (lines 257-278)
```python
def save_configs(
    N,
    K,
    block_n,
    block_k,
    configs,
    save_path,
    input_type="fp8",
) -> None:
    os.makedirs(save_path, exist_ok=True)
    device_name = current_platform.get_device_name().replace(" ", "_")
    json_file_name = (
        f"N={N},K={K},device_name={device_name},dtype={input_type}_w8a8,"
        f"block_shape=[{block_n},{block_k}].json"
    )

    config_file_path = os.path.join(save_path, json_file_name)
    print(f"Writing best config to {config_file_path}...")

    with open(config_file_path, "w") as f:
        json.dump(configs, f, indent=4)
        f.write("\n")
```
**EN:** `save_configs` writes benchmark outputs to persistent storage. It mainly works with `N`, `K`, `block_n`, `block_k`, `configs`, ... and relies on `os.makedirs`, `current_platform.get_device_name.replace`, `current_platform.get_device_name`, `os.path.join`, `print`, `open` plus context management to move data through this part of the benchmark pipeline.
**CN:** `save_configs` 负责将基准结果写入持久化存储。 它主要处理 `N`, `K`, `block_n`, `block_k`, `configs`, ...，并结合 `os.makedirs`, `current_platform.get_device_name.replace`, `current_platform.get_device_name`, `os.path.join`, `print`, `open` 以及 上下文管理 来完成这一段基准测试流程。

### Function `tune_on_gpu` (lines 281-322)
```python
def tune_on_gpu(args_dict):
    """Run tuning on a specific GPU."""
    gpu_id = args_dict["gpu_id"]
    batch_sizes = args_dict["batch_sizes"]
    weight_shapes = args_dict["weight_shapes"]
    args = args_dict["args"]

    torch.accelerator.set_device_index(gpu_id)
    print(f"Starting tuning on GPU {gpu_id} with batch sizes {batch_sizes}")

    block_n = args.block_n
    block_k = args.block_k
    out_dtype = DTYPE_MAP[args.out_dtype]
    save_path = args.save_path
    input_type = args.input_type

    search_space = get_configs_compute_bound()
    search_space = [
        config for config in search_space if block_k % config["BLOCK_SIZE_K"] == 0
    ]

    start = time.time()
    for shape in tqdm(weight_shapes, desc=f"GPU {gpu_id} - Shapes"):
        N, K = shape[0], shape[1]
        print(f"[GPU {gpu_id}] Tune for weight shape of `N: {N}, K: {K}`")
        benchmark_results = [
            tune(
                batch_size,
                N,
                K,
                [block_n, block_k],
                out_dtype,
                search_space,
                input_type,
            )
            for batch_size in tqdm(batch_sizes, desc=f"GPU {gpu_id} - Batch sizes")
        ]
        best_configs = {M: config for M, config in zip(batch_sizes, benchmark_results)}
        save_configs(N, K, block_n, block_k, best_configs, save_path, input_type)

    end = time.time()
    print(f"Tuning on GPU {gpu_id} took {end - start:.2f} seconds")
```
**EN:** `tune_on_gpu` Run tuning on a specific GPU. It mainly works with `args_dict` and relies on `torch.accelerator.set_device_index`, `print`, `get_configs_compute_bound`, `time.time`, `tqdm`, `tune` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `tune_on_gpu` 的职责是：Run tuning on a specific GPU。 它主要处理 `args_dict`，并结合 `torch.accelerator.set_device_index`, `print`, `get_configs_compute_bound`, `time.time`, `tqdm`, `tune` 以及 循环迭代 来完成这一段基准测试流程。

### Function `distribute_batch_sizes` (lines 325-332)
```python
def distribute_batch_sizes(batch_sizes, num_gpus):
    """Distribute batch sizes across available GPUs."""
    batches_per_gpu = []
    for i in range(num_gpus):
        start_idx = i * len(batch_sizes) // num_gpus
        end_idx = (i + 1) * len(batch_sizes) // num_gpus
        batches_per_gpu.append(batch_sizes[start_idx:end_idx])
    return batches_per_gpu
```
**EN:** `distribute_batch_sizes` Distribute batch sizes across available GPUs. It mainly works with `batch_sizes`, `num_gpus` and relies on `range`, `len`, `batches_per_gpu.append` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `distribute_batch_sizes` 的职责是：Distribute batch sizes across available GPUs。 它主要处理 `batch_sizes`, `num_gpus`，并结合 `range`, `len`, `batches_per_gpu.append` 以及 循环迭代 来完成这一段基准测试流程。

### Function `main` (lines 335-388)
```python
def main(args):
    print(args)
    num_gpus = torch.accelerator.device_count()
    if num_gpus == 0:
        raise RuntimeError("No GPU available for tuning")
    print(f"Found {num_gpus} GPUs for parallel tuning")

    torch.cuda.init()

    if args.batch_size is None:
        batch_sizes = [
            1,
            2,
            4,
            8,
            16,
            24,
            32,
            48,
            64,
            96,
            128,
            256,
            512,
            1024,
            1536,
            2048,
            3072,
            4096,
        ]
    else:
        batch_sizes = [args.batch_size]
        num_gpus = 1  # If only one batch size, use only one GPU

    # ... omitted for brevity ...
            }
        )

    ctx = mp.get_context("spawn")
    with ctx.Pool(num_gpus) as pool:
        pool.map(tune_on_gpu, process_args)

    print("Multi-GPU tuning completed")
```
**EN:** `main` serves as the main entry point for this script. It mainly works with `args` and relies on `print`, `torch.accelerator.device_count`, `RuntimeError`, `torch.cuda.init`, `get_weight_shapes`, `distribute_batch_sizes` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 `args`，并结合 `print`, `torch.accelerator.device_count`, `RuntimeError`, `torch.cuda.init`, `get_weight_shapes`, `distribute_batch_sizes` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Entry point (lines 391-415)
```python
if __name__ == "__main__":
    parser = FlexibleArgumentParser(
        description="""
Tune triton w8a8 block fp8 for DeepSeek-V3/DeepSeek-R1:
    python3 benchmark_w8a8_block_fp8.py --tp-size 8 --input-type fp8
Then copy to model_executor/layers/quantization/utils/configs
        """,
        formatter_class=argparse.RawTextHelpFormatter,
    )

    parser.add_argument("--tp-size", "-tp", type=int, default=8)
    parser.add_argument("--input-type", type=str, choices=["fp8"], default="fp8")
    parser.add_argument(
        "--out-dtype",
        type=str,
        choices=["float32", "float16", "bfloat16", "half"],
        default="float16",
    )
    parser.add_argument("--block-n", type=int, default=128)
    parser.add_argument("--block-k", type=int, default=128)
    parser.add_argument("--batch-size", type=int, required=False)
    parser.add_argument("--save-path", type=str, default="./")
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
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: `argparse`, `json`, `multiprocessing`, `os`, `time`, `datetime`.
- **CN:** 标准库依赖：`argparse`, `json`, `multiprocessing`, `os`, `time`, `datetime`。
- **EN:** Third-party packages: `torch`, `tqdm`.
- **CN:** 第三方依赖：`torch`, `tqdm`。
- **EN:** Internal modules: `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.platforms`, `vllm.triton_utils`, `vllm.utils.argparse_utils`.
- **CN:** 内部模块：`vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.platforms`, `vllm.triton_utils`, `vllm.utils.argparse_utils`。
