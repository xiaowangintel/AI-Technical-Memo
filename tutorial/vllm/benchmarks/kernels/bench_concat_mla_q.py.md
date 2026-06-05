# bench_concat_mla_q.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/bench_concat_mla_q.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements tensor concatenation experiments, MLA attention experiments, benchmark orchestration, centered around `NOPE_DIM`, `ROPE_DIM`, `NUM_HEADS`, `NUM_TOKENS`. / 实现与张量拼接实验、MLA 注意力实验、基准测试编排相关的逻辑，核心符号包括 `NOPE_DIM`, `ROPE_DIM`, `NUM_HEADS`, `NUM_TOKENS`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-9)
```python
import argparse

import torch

from vllm import _custom_ops as ops
from vllm.triton_utils import triton
```
**EN:** This block gathers standard-library helpers such as `argparse`; third-party packages such as `torch`; project-local modules such as `vllm`, `vllm.triton_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`；第三方依赖，如 `torch`；项目内部模块，如 `vllm`, `vllm.triton_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 12-16)
```python
NOPE_DIM = 512
ROPE_DIM = 64
NUM_HEADS = 128

NUM_TOKENS = [8, 16, 32, 64, 128, 256, 512, 1024, 2048, 4096, 8192]
```
**EN:** This top-level block prepares shared state such as `NOPE_DIM`, `ROPE_DIM`, `NUM_HEADS`, `NUM_TOKENS`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `NOPE_DIM`, `ROPE_DIM`, `NUM_HEADS`, `NUM_TOKENS`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `get_configs` (lines 19-20)
```python
def get_configs():
    return NUM_TOKENS
```
**EN:** `get_configs` implements a helper used by `bench_concat_mla_q.py`. It mainly works with no explicit parameters and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `get_configs` 负责实现 `bench_concat_mla_q.py` 使用的辅助逻辑。 它主要处理 无显式参数，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Function `make_inputs` (lines 23-36)
```python
def make_inputs(num_tokens, dtype):
    """Create inputs matching the real code path.

    Args:
        contiguous_nope: If False, simulate the transposed BMM output
                         (non-contiguous nope with stride pattern from
                         [N,B,L].transpose(0,1)).
    """
    # Simulate: bmm output [N, B, L].transpose(0, 1) -> [B, N, L]
    raw = torch.randn(NUM_HEADS, num_tokens, NOPE_DIM, dtype=dtype, device="cuda")
    ql_nope = raw.transpose(0, 1)

    q_pe = torch.randn(num_tokens, NUM_HEADS, ROPE_DIM, dtype=dtype, device="cuda")
    return ql_nope, q_pe
```
**EN:** `make_inputs` Create inputs matching the real code path. It mainly works with `num_tokens`, `dtype` and relies on `torch.randn`, `raw.transpose` plus value production to move data through this part of the benchmark pipeline.
**CN:** `make_inputs` 的职责是：Create inputs matching the real code path。 它主要处理 `num_tokens`, `dtype`，并结合 `torch.randn`, `raw.transpose` 以及 结果返回 来完成这一段基准测试流程。

### Function `bench_transposed` (lines 53-72)
```python
def bench_transposed(num_tokens, provider):
    dtype = torch.bfloat16
    ql_nope, q_pe = make_inputs(num_tokens, dtype)

    q_out = torch.empty(
        num_tokens, NUM_HEADS, NOPE_DIM + ROPE_DIM, dtype=dtype, device="cuda"
    )

    quantiles = [0.5, 0.2, 0.8]

    if provider == "torch_cat":
        ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
            lambda: torch.cat((ql_nope, q_pe), dim=-1), quantiles=quantiles, rep=500
        )
    else:
        ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
            lambda: ops.concat_mla_q(ql_nope, q_pe, q_out), quantiles=quantiles, rep=500
        )

    return ms * 1000, max_ms * 1000, min_ms * 1000  # us
```
**EN:** `bench_transposed` implements a helper used by `bench_concat_mla_q.py`. It mainly works with `num_tokens`, `provider` and relies on `make_inputs`, `torch.empty`, `triton.testing.do_bench_cudagraph`, `torch.cat`, `ops.concat_mla_q`, `triton.testing.perf_report` plus branching to move data through this part of the benchmark pipeline.
**CN:** `bench_transposed` 负责实现 `bench_concat_mla_q.py` 使用的辅助逻辑。 它主要处理 `num_tokens`, `provider`，并结合 `make_inputs`, `torch.empty`, `triton.testing.do_bench_cudagraph`, `torch.cat`, `ops.concat_mla_q`, `triton.testing.perf_report` 以及 条件分支 来完成这一段基准测试流程。

### Entry point (lines 75-98)
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Benchmark concat_mla_q vs torch.cat")
    parser.add_argument(
        "--save-path", type=str, default=None, help="Path to save benchmark results"
    )
    args = parser.parse_args()

    print("\n" + "=" * 70)
    print("CONCAT MLA Q KERNEL BENCHMARKS")
    print("=" * 70)
    print(f"Dimensions: nope={NOPE_DIM}, rope={ROPE_DIM}, heads={NUM_HEADS}")
    print(
        f"Per-head output: {NOPE_DIM + ROPE_DIM} bf16 = "
        f"{(NOPE_DIM + ROPE_DIM) * 2} bytes"
    )
    print(f"num_tokens (decode=batch_size, prefill=chunk_size): {NUM_TOKENS}")
    print("=" * 70)

    print("\n--- Non-contiguous nope inputs (transposed BMM output) ---")
    bench_transposed.run(print_data=True, save_path=args.save_path)

    print("\n" + "=" * 70)
    print("Benchmarking complete!")
    print("=" * 70)
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `print`, `bench_transposed.run` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `print`, `bench_transposed.run` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Result presentation: the module aggregates measurements and renders tables or plots.
- **CN:** 结果呈现：该模块会汇总测量结果并生成表格或图形。
- **EN:** Attention specialization: the code studies attention backends, cache layouts, or MLA variants.
- **CN:** 注意力专项：代码研究注意力后端、缓存布局或 MLA 变体。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: `argparse`.
- **CN:** 标准库依赖：`argparse`。
- **EN:** Third-party packages: `torch`.
- **CN:** 第三方依赖：`torch`。
- **EN:** Internal modules: `vllm`, `vllm.triton_utils`.
- **CN:** 内部模块：`vllm`, `vllm.triton_utils`。
