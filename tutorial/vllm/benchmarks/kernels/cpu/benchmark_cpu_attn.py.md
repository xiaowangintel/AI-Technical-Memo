# benchmark_cpu_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/cpu/benchmark_cpu_attn.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, CPU execution benchmarks, centered around `get_attn_isa`, `tensor_cache`, `main`, `generate_seq_lens`. / 实现与基准测试编排、CPU 执行基准相关的逻辑，核心符号包括 `get_attn_isa`, `tensor_cache`, `main`, `generate_seq_lens`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-17)
```python
import functools
import time

import numpy as np
import torch

from vllm._custom_ops import (
    cpu_attention_with_kv_cache,
    cpu_attn_get_scheduler_metadata,
    cpu_attn_reshape_and_cache,
)
from vllm.utils.argparse_utils import FlexibleArgumentParser
from vllm.utils.torch_utils import STR_DTYPE_TO_TORCH_DTYPE, set_random_seed
from vllm.v1.attention.backends.cpu_attn import CPUAttentionBackend, _get_attn_isa
```
**EN:** This block gathers standard-library helpers such as `functools`, `time`; third-party packages such as `numpy`, `torch`; project-local modules such as `vllm._custom_ops`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`, `vllm.v1.attention.backends.cpu_attn`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `functools`, `time`；第三方依赖，如 `numpy`, `torch`；项目内部模块，如 `vllm._custom_ops`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`, `vllm.v1.attention.backends.cpu_attn`。这些依赖构成了后续基准测试逻辑的基础。

### Function `get_attn_isa` (lines 20-31)
```python
def get_attn_isa(
    block_size: int | None = None,
    dtype: torch.dtype | None = None,
):
    # Delegate to _get_attn_isa so the fallback path applies the same arch
    # gating (e.g. RISC-V RVV is only chosen when the build's hardcoded
    # VLEN=128 kernel is actually present; on VLEN=256 / scalar hosts it
    # correctly falls through to vec/vec16).
    return _get_attn_isa(
        dtype if dtype is not None else torch.bfloat16,
        block_size if block_size else 32,
    )
```
**EN:** `get_attn_isa` implements a helper used by `benchmark_cpu_attn.py`. It mainly works with `block_size`, `dtype` and relies on `_get_attn_isa` plus value production to move data through this part of the benchmark pipeline.
**CN:** `get_attn_isa` 负责实现 `benchmark_cpu_attn.py` 使用的辅助逻辑。 它主要处理 `block_size`, `dtype`，并结合 `_get_attn_isa` 以及 结果返回 来完成这一段基准测试流程。

### Function `tensor_cache` (lines 36-41)
```python
def tensor_cache(
    elem_num: int,
    dtype: torch.dtype,
) -> torch.Tensor:
    tensor = torch.randn(elem_num, dtype=dtype)
    return tensor
```
**EN:** `tensor_cache` implements cache-related experiments. It mainly works with `elem_num`, `dtype` and relies on `torch.randn`, `functools.lru_cache` plus value production to move data through this part of the benchmark pipeline.
**CN:** `tensor_cache` 负责实现缓存相关实验。 它主要处理 `elem_num`, `dtype`，并结合 `torch.randn`, `functools.lru_cache` 以及 结果返回 来完成这一段基准测试流程。

### Function `main` (lines 45-181)
```python
def main(
    seq_lens: list[tuple[int, int]],
    num_heads: tuple[int, int],
    head_size: int,
    sliding_window: int = None,
    dtype: torch.dtype = torch.bfloat16,
    block_size: int = 128,
    num_blocks: int = 4096,
    use_sink: bool = False,
    enable_kv_split: bool = False,
    isa: str | None = None,
    seed: int = 0,
    iters: int = 20,
) -> None:
    set_random_seed(seed)
    num_seqs = len(seq_lens)
    query_lens = [x[0] for x in seq_lens]
    kv_lens = [x[1] for x in seq_lens]
    num_query_heads = num_heads[0]
    num_kv_heads = num_heads[1]
    assert num_query_heads % num_kv_heads == 0
    max_kv_len = max(kv_lens)
    window_size = (sliding_window - 1, 0) if sliding_window is not None else (-1, -1)
    scale = head_size**-0.5
    token_num = sum(query_lens)

    if isa is None:
        isa = get_attn_isa(block_size, dtype)

    s_aux = (
        15 * torch.rand((num_query_heads,), dtype=torch.bfloat16) if use_sink else None
    )

    query = tensor_cache(
    # ... omitted for brevity ...
    time_mean = np.mean(times)
    time_std = np.std(times)

    print("\tmin (ms) = ", time_min)
    print("\tmax (ms) = ", time_max)
    print("\tmean (ms) = ", time_mean)
    print("\tstd = ", time_std)
    print("\tmedian (ms) = ", np.median(times))
```
**EN:** `main` serves as the main entry point for this script. It mainly works with `seq_lens`, `num_heads`, `head_size`, `sliding_window`, `dtype`, ... and relies on `set_random_seed`, `len`, `max`, `sum`, `get_attn_isa`, `torch.rand` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 `seq_lens`, `num_heads`, `head_size`, `sliding_window`, `dtype`, ...，并结合 `set_random_seed`, `len`, `max`, `sum`, `get_attn_isa`, `torch.rand` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `generate_seq_lens` (lines 184-208)
```python
def generate_seq_lens(
    batch_size: int,
    q_len_min: int,
    q_len_max: int,
    kv_len_min: int,
    kv_len_max: int,
    seed: int = 0,
) -> list[tuple[int, int]]:
    assert 1 <= q_len_min <= q_len_max
    assert 1 <= kv_len_min <= kv_len_max
    assert kv_len_max >= q_len_min

    g = torch.Generator(device="cpu").manual_seed(seed)

    def rint(lo: int, hi: int) -> int:
        return torch.randint(lo, hi + 1, (1,), generator=g).item()

    seq_lens: list[tuple[int, int]] = []
    for _ in range(batch_size):
        # ensure q <= kv
        kv = rint(max(kv_len_min, q_len_min), kv_len_max)
        q = rint(q_len_min, min(q_len_max, kv))
        seq_lens.append((q, kv))

    return seq_lens
```
**EN:** `generate_seq_lens` implements a helper used by `benchmark_cpu_attn.py`. It mainly works with `batch_size`, `q_len_min`, `q_len_max`, `kv_len_min`, `kv_len_max`, ... and relies on `torch.Generator.manual_seed`, `torch.Generator`, `torch.randint.item`, `torch.randint`, `range`, `rint` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `generate_seq_lens` 负责实现 `benchmark_cpu_attn.py` 使用的辅助逻辑。 它主要处理 `batch_size`, `q_len_min`, `q_len_max`, `kv_len_min`, `kv_len_max`, ...，并结合 `torch.Generator.manual_seed`, `torch.Generator`, `torch.randint.item`, `torch.randint`, `range`, `rint` 以及 循环迭代 来完成这一段基准测试流程。

### Entry point (lines 211-270)
```python
if __name__ == "__main__":
    parser = FlexibleArgumentParser(description="Benchmark the paged attention kernel.")
    parser.add_argument("--batch-size", type=int, default=64)
    parser.add_argument("--q-len-min", type=int, default=512)
    parser.add_argument("--q-len-max", type=int, default=512)
    parser.add_argument("--kv-len-min", type=int, default=512)
    parser.add_argument("--kv-len-max", type=int, default=512)
    parser.add_argument("--num-blocks", type=int, default=4096)

    parser.add_argument("--sliding-window", type=int, default=None)
    parser.add_argument("--num-query-heads", type=int, default=32)
    parser.add_argument("--num-kv-heads", type=int, default=8)
    parser.add_argument(
        "--head-size",
        type=int,
        choices=CPUAttentionBackend.get_supported_head_sizes(),
        default=128,
    )
    parser.add_argument("--enable-kv-split", action="store_true")
    parser.add_argument("--block-size", type=int, choices=[32, 64, 128], default=128)
    parser.add_argument(
        "--dtype", type=str, choices=["half", "bfloat16", "float"], default="bfloat16"
    )
    parser.add_argument("--use-sink", action="store_true")
    parser.add_argument(
        "--isa", type=str, choices=["vec", "neon", "amx", "vec16", "rvv"], default=None
    )
    parser.add_argument("--seed", type=int, default=0)
    parser.add_argument("--iters", type=int, default=20)

    args = parser.parse_args()
    print(args)

    seq_lens = generate_seq_lens(
    # ... omitted for brevity ...
        use_sink=args.use_sink,
        enable_kv_split=args.enable_kv_split,
        isa=args.isa
        if args.isa is not None
        else get_attn_isa(args.block_size, STR_DTYPE_TO_TORCH_DTYPE[args.dtype]),
        seed=args.seed,
        iters=args.iters,
    )
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `FlexibleArgumentParser`, `parser.add_argument`, `CPUAttentionBackend.get_supported_head_sizes`, `parser.parse_args`, `print`, `generate_seq_lens` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `FlexibleArgumentParser`, `parser.add_argument`, `CPUAttentionBackend.get_supported_head_sizes`, `parser.parse_args`, `print`, `generate_seq_lens` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Attention specialization: the code studies attention backends, cache layouts, or MLA variants.
- **CN:** 注意力专项：代码研究注意力后端、缓存布局或 MLA 变体。

## Dependencies / 依赖关系
- **EN:** Standard library: `functools`, `time`.
- **CN:** 标准库依赖：`functools`, `time`。
- **EN:** Third-party packages: `numpy`, `torch`.
- **CN:** 第三方依赖：`numpy`, `torch`。
- **EN:** Internal modules: `vllm._custom_ops`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`, `vllm.v1.attention.backends.cpu_attn`.
- **CN:** 内部模块：`vllm._custom_ops`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`, `vllm.v1.attention.backends.cpu_attn`。
