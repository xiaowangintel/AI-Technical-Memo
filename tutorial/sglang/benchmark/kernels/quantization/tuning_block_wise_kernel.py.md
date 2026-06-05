# tuning_block_wise_kernel.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/kernels/quantization/tuning_block_wise_kernel.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on kernels quantization tuning block wise. It primarily builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. / 该 Python 模块聚焦于 kernels quantization tuning block wise 相关流程。它主要用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

## Line-by-Line Analysis / 逐行分析
### Lines 15-50: Imports and setup / 导入与初始化
```python
import argparse
import json
import multiprocessing as mp
import os
import random
import time
from datetime import datetime
from typing import Any, Dict, List

import torch
import triton
from tqdm import tqdm

mp.set_start_method("spawn", force=True)

from sglang.srt.layers.quantization.fp8_kernel import (
    _w8a8_block_fp8_matmul,
    _w8a8_block_fp8_matmul_unrolledx4,
)
from sglang.srt.layers.quantization.int8_kernel import _w8a8_block_int8_matmul
from sglang.srt.utils import (
    get_device,
    get_device_core_count,
    get_device_count,
    get_device_name,
    is_hip,
)

_is_hip = is_hip()

DTYPE_MAP = {
    "float32": torch.float32,
    "float16": torch.float16,
    "half": torch.half,
    "bfloat16": torch.bfloat16,
}
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, prepares tensors and invokes GPU kernels, and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、准备张量并调用 GPU 内核、加载、筛选或序列化基准测试数据集。

### Lines 53-145: Function `w8a8_block_matmul` / 函数 `w8a8_block_matmul`
```python
def w8a8_block_matmul(
    A: torch.Tensor,
    B: torch.Tensor,
    As: torch.Tensor,
    Bs: torch.Tensor,
    block_size: List[int],
    config: Dict[str, Any],
    output_dtype: torch.dtype = torch.float16,
) -> torch.Tensor:
    """This function performs matrix multiplication with block-wise quantization.

    It takes two input tensors `A` and `B` with scales `As` and `Bs`.
    The output is returned in the specified `output_dtype`.

    Args:
        A: The input tensor, e.g., activation.
        B: The input tensor, e.g., weight.
        As: The per-token-group quantization scale for `A`.
        Bs: The per-block quantization scale for `B`.
        block_size: The block size for per-block quantization. It should be 2-dim, e.g., [128, 128].
        output_dytpe: The dtype of the returned tensor.

    Returns:
        torch.Tensor: The result of matmul.
    """
    assert len(block_size) == 2
    block_n, block_k = block_size[0], block_size[1]

    assert A.shape[-1] == B.shape[-1]
    assert A.shape[:-1] == As.shape[:-1] and A.is_contiguous()
    assert triton.cdiv(A.shape[-1], block_k) == As.shape[-1]
    M = A.numel() // A.shape[-1]

    assert B.ndim == 2 and B.is_contiguous() and Bs.ndim == 2
    N, K = B.shape
    assert triton.cdiv(N, block_n) == Bs.shape[0]
    assert triton.cdiv(K, block_k) == Bs.shape[1]

    C_shape = A.shape[:-1] + (N,)
    C = A.new_empty(C_shape, dtype=output_dtype)

    needs_masking = bool(K % config["BLOCK_SIZE_K"] != 0)

    def grid(META):
        return (
            triton.cdiv(M, META["BLOCK_SIZE_M"]) * triton.cdiv(N, META["BLOCK_SIZE_N"]),
        )

    # Use manually unrolledx4 kernel on AMD GPU when the grid size is small.
    # Empirical testing shows the sweet spot lies when it's less than the # of
    # compute units available on the device.
    num_workgroups = triton.cdiv(M, config["BLOCK_SIZE_M"]) * triton.cdiv(
        N, config["BLOCK_SIZE_N"]
    )

    extra_kernel_args = {}
    if A.dtype == torch.float8_e4m3fnuz or A.dtype == torch.float8_e4m3fn:
        kernel = (
            _w8a8_block_fp8_matmul_unrolledx4
            if (_is_hip == True and num_workgroups <= get_device_core_count())
            else _w8a8_block_fp8_matmul
        )
        # set masking flag required by kernel arguments
        extra_kernel_args["needs_masking"] = needs_masking
    else:
        kernel = _w8a8_block_int8_matmul

    kernel[grid](
        A,
        B,
        C,
        As,
        Bs,
        M,
        N,
        K,
        block_n,
        block_k,
        A.stride(-2),
        A.stride(-1),
        B.stride(1),
        B.stride(0),
        C.stride(-2),
        C.stride(-1),
        As.stride(-2),
        As.stride(-1),
        Bs.stride(1),
        Bs.stride(0),
        **config,
        **extra_kernel_args,
    )

    return C
```
**EN:** `w8a8_block_matmul` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. The docstring frames it as: This function performs matrix multiplication with block-wise quantization. It returns `C` to the caller. Notable calls include `A.new_empty`, `bool`, `kernel[grid]`.
**CN:** `w8a8_block_matmul` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `C`。其中较关键的调用包括 `A.new_empty`, `bool`, `kernel[grid]`。

### Lines 148-168: Function `get_rocm_configs_compute_bound` / 函数 `get_rocm_configs_compute_bound`
```python
def get_rocm_configs_compute_bound():
    configs = []
    waves_per_eu_range = 0
    for num_stages in [2]:
        for block_m in [32, 64, 128, 256]:
            for block_k in [32, 64, 128, 256]:
                for block_n in [16, 32, 64, 128, 256]:
                    for num_warps in [4, 8]:
                        for group_size in [1, 4, 8, 16, 32]:
                            configs.append(
                                {
                                    "BLOCK_SIZE_M": block_m,
                                    "BLOCK_SIZE_N": block_n,
                                    "BLOCK_SIZE_K": block_k,
                                    "GROUP_SIZE_M": group_size,
                                    "num_warps": num_warps,
                                    "num_stages": num_stages,
                                    "waves_per_eu": waves_per_eu_range,
                                }
                            )
    return configs
```
**EN:** `get_rocm_configs_compute_bound` is a function that handles audio loading, conversion, or packaging. It returns `configs` to the caller. Notable calls include `configs.append`.
**CN:** `get_rocm_configs_compute_bound` 是一个函数，用于处理音频加载、转换或打包。它会向调用方返回 `configs`。其中较关键的调用包括 `configs.append`。

### Lines 171-192: Function `get_configs_compute_bound` / 函数 `get_configs_compute_bound`
```python
def get_configs_compute_bound():
    configs = []
    if _is_hip:
        configs = get_rocm_configs_compute_bound()
    else:
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
**EN:** `get_configs_compute_bound` is a function that implements the core logic for this scope. It returns `configs` to the caller. Notable calls include `get_rocm_configs_compute_bound`, `configs.append`.
**CN:** `get_configs_compute_bound` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `configs`。其中较关键的调用包括 `get_rocm_configs_compute_bound`, `configs.append`。

### Lines 195-225: Function `get_weight_shapes` / 函数 `get_weight_shapes`
```python
def get_weight_shapes(tp_size):
    # NOTE(HandH1998): The weight shapes only works for DeepSeek-V3. Modify them, if you tune for another different model.
    # cannot TP
    total = [
        (512 + 64, 7168),
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
**EN:** `get_weight_shapes` is a function that implements the core logic for this scope. It returns `weight_shapes` to the caller. Notable calls include `weight_shapes.append`.
**CN:** `get_weight_shapes` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `weight_shapes`。其中较关键的调用包括 `weight_shapes.append`。

### Lines 228-252: Function `benchmark_config` / 函数 `benchmark_config`
```python
def benchmark_config(
    A, B, As, Bs, block_size, config, out_dtype=torch.float16, num_iters=10
):
    def run():
        w8a8_block_matmul(A, B, As, Bs, block_size, config, out_dtype)

    torch.get_device_module().synchronize()
    # JIT complication & warmup
    for _ in range(5):
        run()
    torch.get_device_module().synchronize()

    start_event = torch.get_device_module().Event(enable_timing=True)
    end_event = torch.get_device_module().Event(enable_timing=True)

    latencies: List[float] = []
    for _ in range(num_iters):
        torch.get_device_module().synchronize()
        start_event.record()
        run()
        end_event.record()
        end_event.synchronize()
        latencies.append(start_event.elapsed_time(end_event))
    avg = sum(latencies) / (num_iters * 10) * 1000  # us
    return avg
```
**EN:** `benchmark_config` is a function that measures runtime latency, throughput, or other benchmark metrics and prepares tensors and invokes GPU kernels. It returns `avg` to the caller. Notable calls include `torch.get_device_module().synchronize`, `range`, `torch.get_device_module().Event`.
**CN:** `benchmark_config` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。它会向调用方返回 `avg`。其中较关键的调用包括 `torch.get_device_module().synchronize`, `range`, `torch.get_device_module().Event`。

### Lines 255-326: Function `tune` / 函数 `tune`
```python
def tune(M, N, K, block_size, out_dtype, search_space, input_type):
    factor_for_scale = 1e-2
    device = get_device()

    if input_type == "fp8":
        fp8_info = torch.finfo(
            torch.float8_e4m3fnuz if _is_hip else torch.float8_e4m3fn
        )
        fp8_max, fp8_min = fp8_info.max, fp8_info.min

        A_fp32 = (
            (torch.rand(M, K, dtype=torch.float32, device=device) - 0.5) * 2 * fp8_max
        )
        A = A_fp32.clamp(min=fp8_min, max=fp8_max).to(
            torch.float8_e4m3fnuz if _is_hip else torch.float8_e4m3fn
        )

        B_fp32 = (
            (torch.rand(N, K, dtype=torch.float32, device=device) - 0.5) * 2 * fp8_max
        )
        B = B_fp32.clamp(min=fp8_min, max=fp8_max).to(
            torch.float8_e4m3fnuz if _is_hip else torch.float8_e4m3fn
        )
    else:
        int8_info = torch.iinfo(torch.int8)
        int8_max, int8_min = int8_info.max, int8_info.min

        A_fp32 = (
            (torch.rand(M, K, dtype=torch.float32, device=device) - 0.5) * 2 * int8_max
        )
        A = A_fp32.clamp(min=int8_min, max=int8_max).to(torch.int8)

        B_fp32 = (
            (torch.rand(N, K, dtype=torch.float32, device=device) - 0.5) * 2 * int8_max
        )
        B = B_fp32.clamp(min=int8_min, max=int8_max).to(torch.int8)

    block_n, block_k = block_size[0], block_size[1]
    n_tiles = (N + block_n - 1) // block_n
    k_tiles = (K + block_k - 1) // block_k

    As = torch.rand(M, k_tiles, dtype=torch.float32, device=device) * factor_for_scale
    Bs = (
        torch.rand(n_tiles, k_tiles, dtype=torch.float32, device=device)
        * factor_for_scale
    )

    best_config = None
    best_time = float("inf")
    for config in tqdm(search_space):
        try:
            kernel_time = benchmark_config(
                A,
                B,
                As,
                Bs,
                block_size,
                config,
                out_dtype,
                num_iters=10,
            )
        except triton.runtime.autotuner.OutOfResources:
            # Some configurations may be invalid and fail to compile.
            continue

        if kernel_time < best_time:
            best_time = kernel_time
            best_config = config
    now = datetime.now()
    print(f"{now.ctime()}] Completed tuning for batch_size={M}")
    assert best_config is not None
    return best_config
```
**EN:** `tune` is a function that measures runtime latency, throughput, or other benchmark metrics and prepares tensors and invokes GPU kernels. It returns `best_config` to the caller. Notable calls include `get_device`, `float`, `tqdm`.
**CN:** `tune` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。它会向调用方返回 `best_config`。其中较关键的调用包括 `get_device`, `float`, `tqdm`。

### Lines 329-363: Function `save_configs` / 函数 `save_configs`
```python
def save_configs(
    N,
    K,
    block_n,
    block_k,
    configs,
    save_path,
    input_type="fp8",
    lock=None,
) -> None:
    os.makedirs(save_path, exist_ok=True)
    device_name = get_device_name().replace(" ", "_")
    json_file_name = f"N={N},K={K},device_name={device_name},dtype={input_type}_w8a8,block_shape=[{block_n}, {block_k}].json"

    config_file_path = os.path.join(save_path, json_file_name)
    print(f"Writing best config to {config_file_path}...")

    if lock is not None:
        lock.acquire()
    try:
        existing_configs = {}
        if os.path.exists(config_file_path):
            with open(config_file_path, "r") as f:
                existing_configs = json.load(f)
            existing_configs = {int(k): v for k, v in existing_configs.items()}

        existing_configs.update(configs)
        existing_configs = dict(sorted(existing_configs.items()))

        with open(config_file_path, "w") as f:
            json.dump(existing_configs, f, indent=4)
            f.write("\n")
    finally:
        if lock is not None:
            lock.release()
```
**EN:** `save_configs` is a function that loads, filters, or serializes benchmark datasets and reads or writes local files and intermediate artifacts. Notable calls include `os.makedirs`, `get_device_name().replace`, `os.path.join`.
**CN:** `save_configs` 是一个函数，用于加载、筛选或序列化基准测试数据集、读写本地文件及中间产物。其中较关键的调用包括 `os.makedirs`, `get_device_name().replace`, `os.path.join`。

### Lines 366-408: Function `tune_on_gpu` / 函数 `tune_on_gpu`
```python
def tune_on_gpu(args_dict):
    """Run tuning on a specific GPU."""
    gpu_id = args_dict["gpu_id"]
    batch_sizes = args_dict["batch_sizes"]
    weight_shapes = args_dict["weight_shapes"]
    args = args_dict["args"]
    lock = args_dict["lock"]

    torch.get_device_module().set_device(gpu_id)
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

    start = time.perf_counter()
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
        save_configs(N, K, block_n, block_k, best_configs, save_path, input_type, lock)

    end = time.perf_counter()
    print(f"Tuning on GPU {gpu_id} took {end - start:.2f} seconds")
```
**EN:** `tune_on_gpu` is a function that measures runtime latency, throughput, or other benchmark metrics and prepares tensors and invokes GPU kernels. The docstring frames it as: Run tuning on a specific GPU. Notable calls include `torch.get_device_module().set_device`, `print`, `get_configs_compute_bound`.
**CN:** `tune_on_gpu` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `torch.get_device_module().set_device`, `print`, `get_configs_compute_bound`。

### Lines 411-420: Function `distribute_batch_sizes` / 函数 `distribute_batch_sizes`
```python
def distribute_batch_sizes(batch_sizes, num_gpus):
    """Distribute batch sizes across available GPUs."""
    # shuffle to distribute workload more evenly and minimize bottleneck effects
    random.shuffle(batch_sizes)
    batches_per_gpu = []
    for i in range(num_gpus):
        start_idx = i * len(batch_sizes) // num_gpus
        end_idx = (i + 1) * len(batch_sizes) // num_gpus
        batches_per_gpu.append(batch_sizes[start_idx:end_idx])
    return batches_per_gpu
```
**EN:** `distribute_batch_sizes` is a function that implements the core logic for this scope. The docstring frames it as: Distribute batch sizes across available GPUs. It returns `batches_per_gpu` to the caller. Notable calls include `random.shuffle`, `range`, `batches_per_gpu.append`.
**CN:** `distribute_batch_sizes` 是一个函数，用于实现当前作用域的核心逻辑。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `batches_per_gpu`。其中较关键的调用包括 `random.shuffle`, `range`, `batches_per_gpu.append`。

### Lines 423-486: Function `main` / 函数 `main`
```python
def main(args):
    print(args)

    num_gpus = get_device_count()
    if num_gpus == 0:
        raise RuntimeError("No GPU available for tuning")
    print(f"Found {num_gpus} GPUs for parallel tuning")

    torch.get_device_module().init()

    if args.batch_sizes is None:
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
        batch_sizes = args.batch_sizes

    # Support manual N and K specification
    if args.N is not None and args.K is not None:
        weight_shapes = [(args.N, args.K)]
        print(f"Using manually specified weight shape: N={args.N}, K={args.K}")
    else:
        weight_shapes = get_weight_shapes(args.tp_size)
        print(f"Using predefined weight shapes for TP size {args.tp_size}")

    batches_per_gpu = distribute_batch_sizes(batch_sizes, num_gpus)

    ctx = mp.get_context("spawn")
    manager = ctx.Manager()
    lock = manager.Lock()

    process_args = []
    for gpu_id in range(num_gpus):
        process_args.append(
            {
                "gpu_id": gpu_id,
                "batch_sizes": batches_per_gpu[gpu_id],
                "weight_shapes": weight_shapes,  # Each GPU processes all weight shapes
                "args": args,
                "lock": lock,
            }
        )

    with ctx.Pool(num_gpus) as pool:
        pool.map(tune_on_gpu, process_args)

    print("Multi-GPU tuning completed")
```
**EN:** `main` is a function that prepares tensors and invokes GPU kernels. Notable calls include `print`, `get_device_count`, `torch.get_device_module().init`.
**CN:** `main` 是一个函数，用于准备张量并调用 GPU 内核。其中较关键的调用包括 `print`, `get_device_count`, `torch.get_device_module().init`。

### Lines 489-532: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "--tp-size",
        "-tp",
        type=int,
        default=8,
        help="Tensor parallelism size (ignored if --N and --K are specified)",
    )
    parser.add_argument(
        "--N",
        type=int,
        default=None,
        help="Output dimension of weight matrix (number of columns)",
    )
    parser.add_argument(
        "--K",
        type=int,
        default=None,
        help="Input dimension of weight matrix (number of rows)",
    )
    parser.add_argument(
        "--input-type", type=str, choices=["fp8", "int8"], default="fp8"
    )
    parser.add_argument(
        "--out-dtype",
        type=str,
        choices=["float32", "float16", "bfloat16", "half"],
        default="float16",
    )
    parser.add_argument("--block-n", type=int, default=128)
    parser.add_argument("--block-k", type=int, default=128)
    parser.add_argument("--batch-sizes", nargs="+", type=int, required=False)
    parser.add_argument(
        "--save-path", type=str, default="python/sglang/srt/layers/quantization/configs"
    )
    args = parser.parse_args()

    # Validate arguments
    if (args.N is None) != (args.K is None):
        parser.error("--N and --K must be specified together or not at all")

    main(args)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration and prepares tensors and invokes GPU kernels.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、准备张量并调用 GPU 内核。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Audio processing / 音频处理**: Prepares waveform data for speech or streaming benchmarks. / 为语音或流式基准测试准备波形数据。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `json`, `multiprocessing`, `os`, `random`, `time`, `datetime`, `typing`
- **Third-party / 第三方依赖**: `torch`, `triton`, `tqdm`
- **Internal / 项目内部依赖**: `sglang.srt.layers.quantization.fp8_kernel`, `sglang.srt.layers.quantization.int8_kernel`, `sglang.srt.utils`
