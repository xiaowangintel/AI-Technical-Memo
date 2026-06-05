# benchmark_reshape_and_cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_reshape_and_cache.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, cache reshape benchmarks, shape enumeration utilities, centered around `run_benchmark`, `main`. / 实现与基准测试编排、缓存重排基准、形状枚举工具相关的逻辑，核心符号包括 `run_benchmark`, `main`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-16)
```python
import random
import time

import torch
from tabulate import tabulate

from vllm import _custom_ops as ops
from vllm.logger import init_logger
from vllm.utils.argparse_utils import FlexibleArgumentParser
from vllm.utils.torch_utils import (
    STR_DTYPE_TO_TORCH_DTYPE,
    create_kv_caches_with_random,
    set_random_seed,
)
```
**EN:** This block gathers standard-library helpers such as `random`, `time`; third-party packages such as `torch`, `tabulate`; project-local modules such as `vllm`, `vllm.logger`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `random`, `time`；第三方依赖，如 `torch`, `tabulate`；项目内部模块，如 `vllm`, `vllm.logger`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 18-18)
```python
logger = init_logger(__name__)
```
**EN:** This top-level block prepares shared state such as `logger`. It uses `init_logger` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `logger`。它借助 `init_logger` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `run_benchmark` (lines 22-109)
```python
def run_benchmark(
    num_tokens: int,
    num_heads: int,
    head_size: int,
    block_size: int,
    num_blocks: int,
    dtype: torch.dtype,
    kv_cache_dtype: str,
    num_iters: int,
    benchmark_mode: str,
    device: str = "cuda",
) -> float:
    """Return latency (seconds) for given num_tokens."""

    if kv_cache_dtype == "fp8" and head_size % 16:
        raise ValueError("fp8 kv-cache requires head_size to be a multiple of 16.")

    set_random_seed(42)
    torch.set_default_device(device)

    # create random key / value tensors [T, H, D].
    key = torch.randn(num_tokens, num_heads, head_size, dtype=dtype, device=device)
    value = torch.randn_like(key)

    # prepare the slot mapping.
    # each token is assigned a unique slot in the KV-cache.
    num_slots = block_size * num_blocks
    if num_tokens > num_slots:
        raise ValueError("num_tokens cannot exceed the total number of cache slots")
    slot_mapping_lst = random.sample(range(num_slots), num_tokens)
    slot_mapping = torch.tensor(slot_mapping_lst, dtype=torch.long, device=device)

    key_caches, value_caches = create_kv_caches_with_random(
        num_blocks,
    # ... omitted for brevity ...

    lat = run_cuda_benchmark(num_iters)

    # free tensors to mitigate OOM when sweeping
    del key, value, key_cache, value_cache, slot_mapping
    torch.accelerator.empty_cache()

    return lat
```
**EN:** `run_benchmark` Return latency (seconds) for given num_tokens. It mainly works with `num_tokens`, `num_heads`, `head_size`, `block_size`, `num_blocks`, ... and relies on `ValueError`, `set_random_seed`, `torch.set_default_device`, `torch.randn`, `torch.randn_like`, `random.sample` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `run_benchmark` 的职责是：Return latency (seconds) for given num_tokens。 它主要处理 `num_tokens`, `num_heads`, `head_size`, `block_size`, `num_blocks`, ...，并结合 `ValueError`, `set_random_seed`, `torch.set_default_device`, `torch.randn`, `torch.randn_like`, `random.sample` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Function `main` (lines 112-131)
```python
def main(args):
    rows = []
    for exp in range(1, 17):
        n_tok = 2**exp
        lat = run_benchmark(
            num_tokens=n_tok,
            num_heads=args.num_heads,
            head_size=args.head_size,
            block_size=args.block_size,
            num_blocks=args.num_blocks,
            dtype=STR_DTYPE_TO_TORCH_DTYPE[args.dtype],
            kv_cache_dtype=args.kv_cache_dtype,
            num_iters=args.iters,
            benchmark_mode=args.mode,
            device="cuda",
        )
        rows.append([n_tok, lat * 1e6])  # convert to microseconds

    print(f"Benchmark results for implementation cuda (measuring with {args.mode}):")
    print(tabulate(rows, headers=["num_tokens", "latency (µs)"], floatfmt=".3f"))
```
**EN:** `main` serves as the main entry point for this script. It mainly works with `args` and relies on `range`, `run_benchmark`, `rows.append`, `print`, `tabulate` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 `args`，并结合 `range`, `run_benchmark`, `rows.append`, `print`, `tabulate` 以及 循环迭代 来完成这一段基准测试流程。

### Entry point (lines 134-172)
```python
if __name__ == "__main__":
    parser = FlexibleArgumentParser()

    parser.add_argument("--num-heads", type=int, default=128)
    parser.add_argument(
        "--head-size",
        type=int,
        choices=[64, 80, 96, 112, 120, 128, 192, 256],
        default=128,
    )
    parser.add_argument("--block-size", type=int, choices=[16, 32], default=16)
    parser.add_argument("--num-blocks", type=int, default=128 * 128)

    parser.add_argument(
        "--dtype",
        type=str,
        choices=["half", "bfloat16", "float"],
        default="bfloat16",
    )

    parser.add_argument(
        "--kv-cache-dtype",
        type=str,
        choices=["auto", "fp8"],
        default="auto",
    )

    parser.add_argument("--iters", type=int, default=200)

    parser.add_argument(
        "--mode",
        type=str,
        choices=["cudagraph", "no_graph"],
        default="cudagraph",
    )

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
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。
- **EN:** Performance metrics: the script emphasizes latency/throughput measurement rather than model quality.
- **CN:** 性能指标：该脚本更强调延迟/吞吐量测量，而不是模型质量。

## Dependencies / 依赖关系
- **EN:** Standard library: `random`, `time`.
- **CN:** 标准库依赖：`random`, `time`。
- **EN:** Third-party packages: `torch`, `tabulate`.
- **CN:** 第三方依赖：`torch`, `tabulate`。
- **EN:** Internal modules: `vllm`, `vllm.logger`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`.
- **CN:** 内部模块：`vllm`, `vllm.logger`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`。
