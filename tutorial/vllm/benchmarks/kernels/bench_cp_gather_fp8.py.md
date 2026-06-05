# bench_cp_gather_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/bench_cp_gather_fp8.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements FP8 kernel experiments, benchmark orchestration, centered around `NOPE_DIM`, `ROPE_DIM`, `HEAD_DIM`, `ENTRY_BYTES`. / 实现与FP8 内核实验、基准测试编排相关的逻辑，核心符号包括 `NOPE_DIM`, `ROPE_DIM`, `HEAD_DIM`, `ENTRY_BYTES`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-9)
```python
import argparse
import math

import torch

from vllm import _custom_ops as ops
from vllm.triton_utils import triton
```
**EN:** This block gathers standard-library helpers such as `argparse`, `math`; third-party packages such as `torch`; project-local modules such as `vllm`, `vllm.triton_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`, `math`；第三方依赖，如 `torch`；项目内部模块，如 `vllm`, `vllm.triton_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 12-27)
```python
NOPE_DIM = 512
ROPE_DIM = 64
HEAD_DIM = NOPE_DIM + ROPE_DIM  # 576 BF16 output elements per token
ENTRY_BYTES = 656  # 512 FP8 + 16 scales + 128 BF16 RoPE
BLOCK_SIZE = 64  # tokens per physical cache block - get_supported_kernel_block_sizes

# Realistic prefill scenarios:
#   - 1 long prefill: single request, 16K-96K tokens
#   - 4 medium prefills: 4 requests, 4K-24K tokens each
#   - 16 shorter prefills: 16 requests, 1K-6K tokens each
SCENARIOS = [
    # (label, num_reqs, total_tokens_list)
    ("1-req", 1, [8192, 16384, 32768, 65536, 98304]),
    ("4-reqs", 4, [8192, 16384, 32768, 65536, 98304]),
    ("16-reqs", 16, [8192, 16384, 32768, 65536, 98304]),
]
```
**EN:** This top-level block prepares shared state such as `NOPE_DIM`, `ROPE_DIM`, `HEAD_DIM`, `ENTRY_BYTES`, `BLOCK_SIZE`, `SCENARIOS`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `NOPE_DIM`, `ROPE_DIM`, `HEAD_DIM`, `ENTRY_BYTES`, `BLOCK_SIZE`, `SCENARIOS`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `make_inputs` (lines 30-77)
```python
def make_inputs(total_tokens, num_reqs, block_size):
    """Create synthetic FP8 cache, block table, and output buffer.

    Fills the cache with random bytes (we only measure throughput,
    not correctness). Block table maps each request to contiguous
    physical blocks.
    """
    # Divide tokens evenly across requests
    base_len = total_tokens // num_reqs
    remainder = total_tokens % num_reqs
    seq_lens = [base_len + (1 if r < remainder else 0) for r in range(num_reqs)]

    # workspace_starts: cumulative sum of seq_lens
    workspace_starts = [0] * num_reqs
    for r in range(1, num_reqs):
        workspace_starts[r] = workspace_starts[r - 1] + seq_lens[r - 1]

    # Physical blocks needed per request
    blocks_per_req = [math.ceil(s / block_size) for s in seq_lens]
    total_blocks = sum(blocks_per_req)
    max_blocks = max(blocks_per_req)

    # Allocate cache with random data (content doesn't matter for perf)
    cache = torch.randint(
        0,
        256,
        (total_blocks, block_size, ENTRY_BYTES),
        dtype=torch.uint8,
        device="cuda",
    )

    # Block table: contiguous block assignments
    block_table = torch.zeros(num_reqs, max_blocks, dtype=torch.int32, device="cuda")
    block_idx = 0
    for r in range(num_reqs):
        for b in range(blocks_per_req[r]):
            block_table[r, b] = block_idx
            block_idx += 1

    # Output workspace
    dst = torch.zeros(total_tokens, HEAD_DIM, dtype=torch.bfloat16, device="cuda")

    seq_lens_t = torch.tensor(seq_lens, dtype=torch.int32, device="cuda")
    workspace_starts_t = torch.tensor(
        workspace_starts, dtype=torch.int32, device="cuda"
    )

    return cache, dst, block_table, seq_lens_t, workspace_starts_t
```
**EN:** `make_inputs` Create synthetic FP8 cache, block table, and output buffer. It mainly works with `total_tokens`, `num_reqs`, `block_size` and relies on `range`, `math.ceil`, `sum`, `max`, `torch.randint`, `torch.zeros` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `make_inputs` 的职责是：Create synthetic FP8 cache, block table, and output buffer。 它主要处理 `total_tokens`, `num_reqs`, `block_size`，并结合 `range`, `math.ceil`, `sum`, `max`, `torch.randint`, `torch.zeros` 以及 循环迭代 来完成这一段基准测试流程。

### Function `bench_scenario` (lines 80-119)
```python
def bench_scenario(label, num_reqs, total_tokens_list, save_path):
    """Run benchmark for a specific (num_reqs, total_tokens) scenario."""

    @triton.testing.perf_report(
        triton.testing.Benchmark(
            x_names=["total_tokens"],
            x_vals=total_tokens_list,
            line_arg="provider",
            line_vals=["cuda_kernel"],
            line_names=["cp_gather_fp8 (CUDA)"],
            styles=[("green", "-")],
            ylabel="Latency (us)",
            plot_name=f"cp_gather_fp8-{label}-bs{BLOCK_SIZE}",
            args={"num_reqs": num_reqs},
        )
    )
    def bench_fn(total_tokens, provider, num_reqs):
        cache, dst, block_table, seq_lens_t, ws_starts = make_inputs(
            total_tokens, num_reqs, BLOCK_SIZE
        )

        quantiles = [0.5, 0.2, 0.8]

        ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
            lambda: ops.cp_gather_and_upconvert_fp8_kv_cache(
                cache, dst, block_table, seq_lens_t, ws_starts, num_reqs
            ),
            quantiles=quantiles,
            rep=500,
        )

        return ms * 1000, max_ms * 1000, min_ms * 1000  # us

    seq_len_per_req = total_tokens_list[0] // num_reqs
    seq_len_per_req_max = total_tokens_list[-1] // num_reqs
    print(
        f"\n--- {label}: {num_reqs} request(s), "
        f"~{seq_len_per_req}-{seq_len_per_req_max} tokens/req ---"
    )
    bench_fn.run(print_data=True, save_path=save_path)
```
**EN:** `bench_scenario` Run benchmark for a specific (num_reqs, total_tokens) scenario. It mainly works with `label`, `num_reqs`, `total_tokens_list`, `save_path` and relies on `make_inputs`, `triton.testing.do_bench_cudagraph`, `ops.cp_gather_and_upconvert_fp8_kv_cache`, `triton.testing.perf_report`, `triton.testing.Benchmark`, `print` plus value production to move data through this part of the benchmark pipeline.
**CN:** `bench_scenario` 的职责是：Run benchmark for a specific (num_reqs, total_tokens) scenario。 它主要处理 `label`, `num_reqs`, `total_tokens_list`, `save_path`，并结合 `make_inputs`, `triton.testing.do_bench_cudagraph`, `ops.cp_gather_and_upconvert_fp8_kv_cache`, `triton.testing.perf_report`, `triton.testing.Benchmark`, `print` 以及 结果返回 来完成这一段基准测试流程。

### Entry point (lines 122-153)
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="Benchmark cp_gather_and_upconvert_fp8_kv_cache"
    )
    parser.add_argument(
        "--save-path",
        type=str,
        default=None,
        help="Path to save benchmark results as CSV",
    )
    args = parser.parse_args()

    # Print data volume info for bandwidth analysis
    read_per_token = ENTRY_BYTES  # 656 bytes from cache
    write_per_token = HEAD_DIM * 2  # 576 * 2 = 1152 bytes to workspace
    total_per_token = read_per_token + write_per_token  # 1808 bytes

    print("\n" + "=" * 70)
    print("CP_GATHER_AND_UPCONVERT_FP8_KV_CACHE BENCHMARKS")
    print("=" * 70)
    print(f"Cache entry: {ENTRY_BYTES} bytes (512 FP8 + 16 scales + 128 RoPE)")
    print(f"Output row:  {HEAD_DIM} BF16 = {HEAD_DIM * 2} bytes")
    print(f"Per token:   {total_per_token} bytes (read + write)")
    print(f"Block size:  {BLOCK_SIZE} tokens/block")
    print("=" * 70)

    for label, num_reqs, total_tokens_list in SCENARIOS:
        bench_scenario(label, num_reqs, total_tokens_list, args.save_path)

    print("\n" + "=" * 70)
    print("Benchmarking complete!")
    print("=" * 70)
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `print`, `bench_scenario` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `print`, `bench_scenario` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Serving path benchmarking: the script interacts with HTTP services, servers, or request proxies.
- **CN:** 服务路径基准：脚本会与 HTTP 服务、服务器或请求代理交互。
- **EN:** Result presentation: the module aggregates measurements and renders tables or plots.
- **CN:** 结果呈现：该模块会汇总测量结果并生成表格或图形。
- **EN:** Attention specialization: the code studies attention backends, cache layouts, or MLA variants.
- **CN:** 注意力专项：代码研究注意力后端、缓存布局或 MLA 变体。

## Dependencies / 依赖关系
- **EN:** Standard library: `argparse`, `math`.
- **CN:** 标准库依赖：`argparse`, `math`。
- **EN:** Third-party packages: `torch`.
- **CN:** 第三方依赖：`torch`。
- **EN:** Internal modules: `vllm`, `vllm.triton_utils`.
- **CN:** 内部模块：`vllm`, `vllm.triton_utils`。
