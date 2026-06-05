# benchmark_paged_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_paged_attention.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, paged attention benchmarks, attention benchmark flows, centered around `NUM_BLOCKS`, `PARTITION_SIZE`, `PARTITION_SIZE_ROCM`, `main`. / 实现与基准测试编排、分页注意力基准、注意力基准流程相关的逻辑，核心符号包括 `NUM_BLOCKS`, `PARTITION_SIZE`, `PARTITION_SIZE_ROCM`, `main`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-17)
```python
import random
import time

import torch

from vllm import _custom_ops as ops
from vllm.logger import init_logger
from vllm.platforms import current_platform
from vllm.utils.argparse_utils import FlexibleArgumentParser
from vllm.utils.torch_utils import (
    STR_DTYPE_TO_TORCH_DTYPE,
    create_kv_caches_with_random,
    set_random_seed,
)
```
**EN:** This block gathers standard-library helpers such as `random`, `time`; third-party packages such as `torch`; project-local modules such as `vllm`, `vllm.logger`, `vllm.platforms`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `random`, `time`；第三方依赖，如 `torch`；项目内部模块，如 `vllm`, `vllm.logger`, `vllm.platforms`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 19-23)
```python
logger = init_logger(__name__)

NUM_BLOCKS = 128 * 1024
PARTITION_SIZE = 512
PARTITION_SIZE_ROCM = 256
```
**EN:** This top-level block prepares shared state such as `logger`, `NUM_BLOCKS`, `PARTITION_SIZE`, `PARTITION_SIZE_ROCM`. It uses `init_logger` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `logger`, `NUM_BLOCKS`, `PARTITION_SIZE`, `PARTITION_SIZE_ROCM`。它借助 `init_logger` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `main` (lines 27-193)
```python
def main(
    version: str,
    num_seqs: int,
    seq_len: int,
    num_query_heads: int,
    num_kv_heads: int,
    head_size: int,
    use_alibi: bool,
    block_size: int,
    dtype: torch.dtype,
    seed: int,
    do_profile: bool,
    device: str = "cuda",
    kv_cache_dtype: str | None = None,
) -> None:
    set_random_seed(seed)

    scale = float(1.0 / (head_size**0.5))
    query = torch.empty(
        num_seqs, num_query_heads, head_size, dtype=dtype, device=device
    )
    query.uniform_(-scale, scale)

    assert num_query_heads % num_kv_heads == 0
    alibi_slopes = None
    if use_alibi:
        alibi_slopes = torch.randn(num_query_heads, dtype=torch.float, device=device)

    seq_lens = [seq_len for _ in range(num_seqs)]
    max_seq_len = max(seq_lens)
    seq_lens = torch.tensor(seq_lens, dtype=torch.int, device=device)

    # Create the block tables.
    max_num_blocks_per_seq = (max_seq_len + block_size - 1) // block_size
    # ... omitted for brevity ...
    run_benchmark(num_iters=3, profile=False)

    # Benchmark.
    if do_profile:
        latency = run_benchmark(num_iters=1, profile=True)
    else:
        latency = run_benchmark(num_iters=100, profile=False)
    print(f"Kernel running time: {latency * 1000000:.3f} us")
```
**EN:** `main` serves as the main entry point for this script. It mainly works with `version`, `num_seqs`, `seq_len`, `num_query_heads`, `num_kv_heads`, ... and relies on `set_random_seed`, `float`, `torch.empty`, `query.uniform_`, `torch.randn`, `range` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 `version`, `num_seqs`, `seq_len`, `num_query_heads`, `num_kv_heads`, ...，并结合 `set_random_seed`, `float`, `torch.empty`, `query.uniform_`, `torch.randn`, `range` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Entry point (lines 196-251)
```python
if __name__ == "__main__":
    logger.warning(
        "This script benchmarks the paged attention kernel. "
        "By default this is no longer used in vLLM inference."
    )

    parser = FlexibleArgumentParser(description="Benchmark the paged attention kernel.")
    parser.add_argument("--version", type=str, choices=["v1", "v2"], default="v2")
    parser.add_argument("--batch-size", type=int, default=8)
    parser.add_argument("--seq-len", type=int, default=4096)
    parser.add_argument("--num-query-heads", type=int, default=64)
    parser.add_argument("--num-kv-heads", type=int, default=8)
    parser.add_argument(
        "--head-size",
        type=int,
        choices=[64, 80, 96, 112, 120, 128, 192, 256],
        default=128,
    )
    parser.add_argument("--block-size", type=int, choices=[16, 32], default=16)
    parser.add_argument("--use-alibi", action="store_true")
    parser.add_argument(
        "--dtype", type=str, choices=["half", "bfloat16", "float"], default="half"
    )
    parser.add_argument("--seed", type=int, default=0)
    parser.add_argument("--profile", action="store_true")
    parser.add_argument(
        "--kv-cache-dtype",
        type=str,
        choices=["auto", "fp8", "fp8_e5m2", "fp8_e4m3"],
        default="auto",
        help="Data type for kv cache storage. If 'auto', will use model "
        "data type. CUDA 11.8+ supports fp8 (=fp8_e4m3) and fp8_e5m2. "
        "ROCm (AMD GPU) supports fp8 (=fp8_e4m3)",
    )
    # ... omitted for brevity ...
        head_size=args.head_size,
        block_size=args.block_size,
        use_alibi=args.use_alibi,
        dtype=STR_DTYPE_TO_TORCH_DTYPE[args.dtype],
        seed=args.seed,
        do_profile=args.profile,
        kv_cache_dtype=args.kv_cache_dtype,
    )
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `logger.warning`, `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `print`, `ValueError` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `logger.warning`, `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `print`, `ValueError` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Attention specialization: the code studies attention backends, cache layouts, or MLA variants.
- **CN:** 注意力专项：代码研究注意力后端、缓存布局或 MLA 变体。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。
- **EN:** Performance metrics: the script emphasizes latency/throughput measurement rather than model quality.
- **CN:** 性能指标：该脚本更强调延迟/吞吐量测量，而不是模型质量。

## Dependencies / 依赖关系
- **EN:** Standard library: `random`, `time`.
- **CN:** 标准库依赖：`random`, `time`。
- **EN:** Third-party packages: `torch`.
- **CN:** 第三方依赖：`torch`。
- **EN:** Internal modules: `vllm`, `vllm.logger`, `vllm.platforms`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`.
- **CN:** 内部模块：`vllm`, `vllm.logger`, `vllm.platforms`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`。
