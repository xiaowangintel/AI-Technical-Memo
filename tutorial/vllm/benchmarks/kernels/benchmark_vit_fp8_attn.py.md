# benchmark_vit_fp8_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_vit_fp8_attn.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, vision transformer benchmarks, FP8 kernel experiments, centered around `NUM_HEADS`, `HEAD_DIM`, `DEFAULT_SEQ_LENS`, `_setup_fp8_attention`. / 实现与基准测试编排、视觉 Transformer 基准、FP8 内核实验相关的逻辑，核心符号包括 `NUM_HEADS`, `HEAD_DIM`, `DEFAULT_SEQ_LENS`, `_setup_fp8_attention`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 18-24)
```python
from functools import partial

import numpy as np
import torch
from torch.profiler import ProfilerActivity, profile, record_function

from vllm.utils.argparse_utils import FlexibleArgumentParser
```
**EN:** This block gathers standard-library helpers such as `functools`; third-party packages such as `numpy`, `torch`, `torch.profiler`; project-local modules such as `vllm.utils.argparse_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `functools`；第三方依赖，如 `numpy`, `torch`, `torch.profiler`；项目内部模块，如 `vllm.utils.argparse_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 27-29)
```python
NUM_HEADS = 16
HEAD_DIM = 72
DEFAULT_SEQ_LENS = [2304, 4096, 8192, 16384]
```
**EN:** This top-level block prepares shared state such as `NUM_HEADS`, `HEAD_DIM`, `DEFAULT_SEQ_LENS`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `NUM_HEADS`, `HEAD_DIM`, `DEFAULT_SEQ_LENS`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `_setup_fp8_attention` (lines 32-76)
```python
def _setup_fp8_attention(num_heads: int, head_dim: int) -> tuple:
    """Create FP8 and BF16 attention modules + workspace."""
    from types import SimpleNamespace
    from unittest.mock import patch

    from vllm.config import VllmConfig, set_current_vllm_config
    from vllm.config.multimodal import MultiModalConfig
    from vllm.model_executor.layers.attention.mm_encoder_attention import (
        MMEncoderAttention,
        _get_flashinfer_workspace_buffer,
    )
    from vllm.v1.attention.backends.registry import AttentionBackendEnum

    old_dtype = torch.get_default_dtype()
    torch.set_default_dtype(torch.bfloat16)

    backend_patch = patch(
        "vllm.model_executor.layers.attention.mm_encoder_attention"
        ".get_vit_attn_backend",
        return_value=AttentionBackendEnum.FLASHINFER,
    )

    # FP8 attention
    mm_config_fp8 = MultiModalConfig(mm_encoder_attn_dtype="fp8")
    vllm_config_fp8 = VllmConfig()
    vllm_config_fp8.model_config = SimpleNamespace(multimodal_config=mm_config_fp8)
    with set_current_vllm_config(vllm_config_fp8), backend_patch:
        attn_fp8 = MMEncoderAttention(
            num_heads=num_heads,
            head_size=head_dim,
            prefix="visual.blocks.0.attn",
        ).to("cuda")

    # BF16 attention (no FP8)
    with set_current_vllm_config(VllmConfig()), backend_patch:
        attn_bf16 = MMEncoderAttention(
            num_heads=num_heads,
            head_size=head_dim,
            prefix="visual.blocks.0.attn",
        ).to("cuda")

    torch.set_default_dtype(old_dtype)

    workspace = _get_flashinfer_workspace_buffer()
    return attn_fp8, attn_bf16, workspace
```
**EN:** `_setup_fp8_attention` Create FP8 and BF16 attention modules + workspace. It mainly works with `num_heads`, `head_dim` and relies on `torch.get_default_dtype`, `torch.set_default_dtype`, `patch`, `MultiModalConfig`, `VllmConfig`, `SimpleNamespace` plus context management to move data through this part of the benchmark pipeline.
**CN:** `_setup_fp8_attention` 的职责是：Create FP8 and BF16 attention modules + workspace。 它主要处理 `num_heads`, `head_dim`，并结合 `torch.get_default_dtype`, `torch.set_default_dtype`, `patch`, `MultiModalConfig`, `VllmConfig`, `SimpleNamespace` 以及 上下文管理 来完成这一段基准测试流程。

### Function `_build_meta` (lines 79-110)
```python
def _build_meta(
    seq_len: int,
    num_heads: int,
    head_dim: int,
    fp8: bool,
):
    """Build cu_seqlens, max_seqlen, sequence_lengths."""
    from vllm.model_executor.layers.attention.mm_encoder_attention import (
        MMEncoderAttention,
    )
    from vllm.utils.math_utils import round_up
    from vllm.v1.attention.backends.registry import AttentionBackendEnum

    cu_np = np.array([0, seq_len], dtype=np.int32)
    fp8_padded = num_heads * round_up(head_dim, 16) if fp8 else None

    seq_lengths = MMEncoderAttention.maybe_compute_seq_lens(
        AttentionBackendEnum.FLASHINFER, cu_np, torch.device("cuda")
    )
    max_seqlen = torch.tensor(
        MMEncoderAttention.compute_max_seqlen(AttentionBackendEnum.FLASHINFER, cu_np),
        dtype=torch.int32,
    )
    cu_seqlens = MMEncoderAttention.maybe_recompute_cu_seqlens(
        AttentionBackendEnum.FLASHINFER,
        cu_np,
        num_heads * head_dim,
        1,
        torch.device("cuda"),
        fp8_padded_hidden_size=fp8_padded,
    )
    return cu_seqlens, max_seqlen, seq_lengths
```
**EN:** `_build_meta` Build cu_seqlens, max_seqlen, sequence_lengths. It mainly works with `seq_len`, `num_heads`, `head_dim`, `fp8` and relies on `np.array`, `round_up`, `MMEncoderAttention.maybe_compute_seq_lens`, `torch.device`, `torch.tensor`, `MMEncoderAttention.compute_max_seqlen` plus value production to move data through this part of the benchmark pipeline.
**CN:** `_build_meta` 的职责是：Build cu_seqlens, max_seqlen, sequence_lengths。 它主要处理 `seq_len`, `num_heads`, `head_dim`, `fp8`，并结合 `np.array`, `round_up`, `MMEncoderAttention.maybe_compute_seq_lens`, `torch.device`, `torch.tensor`, `MMEncoderAttention.compute_max_seqlen` 以及 结果返回 来完成这一段基准测试流程。

### Function `run_benchmark` (lines 113-174)
```python
def run_benchmark(
    seq_lens: list[int],
    num_heads: int,
    head_dim: int,
    method: str,
):
    """Benchmark FP8 vs BF16 attention across seq_lens.

    Uses FlashInfer GPU-level timing to measure pure kernel time,
    excluding CPU launch overhead.
    """
    if method == "cupti":
        from flashinfer.testing import bench_gpu_time_with_cupti as bench_fn

        bench_fn = partial(bench_fn, use_cuda_graph=True, cold_l2_cache=False)
    elif method == "cudagraph":
        from flashinfer.testing import (
            bench_gpu_time_with_cudagraph as bench_fn,
        )

        bench_fn = partial(bench_fn, cold_l2_cache=False)
    else:
        raise ValueError(f"Invalid method: {method}")

    attn_fp8, attn_bf16, workspace = _setup_fp8_attention(num_heads, head_dim)

    print(f"Timing method: {method}")
    print(f"{'seq_len':>8} {'BF16 (us)':>12} {'FP8 (us)':>12} {'Speedup':>10}")
    print("-" * 46)

    for seq_len in seq_lens:
        torch.manual_seed(42)

        q = torch.randn(
    # ... omitted for brevity ...
        bf16_times = bench_fn(bf16_fn)
        fp8_times = bench_fn(fp8_fn)

        bf16_us = np.median(bf16_times) * 1e3  # ms -> us
        fp8_us = np.median(fp8_times) * 1e3
        speedup = bf16_us / fp8_us if fp8_us > 0 else float("inf")

        print(f"{seq_len:>8} {bf16_us:>12.1f} {fp8_us:>12.1f} {speedup:>9.2f}x")
```
**EN:** `run_benchmark` Benchmark FP8 vs BF16 attention across seq_lens. It mainly works with `seq_lens`, `num_heads`, `head_dim`, `method` and relies on `partial`, `ValueError`, `_setup_fp8_attention`, `print`, `torch.manual_seed`, `torch.randn` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `run_benchmark` 的职责是：Benchmark FP8 vs BF16 attention across seq_lens。 它主要处理 `seq_lens`, `num_heads`, `head_dim`, `method`，并结合 `partial`, `ValueError`, `_setup_fp8_attention`, `print`, `torch.manual_seed`, `torch.randn` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `_make_trace_handler` (lines 177-187)
```python
def _make_trace_handler(output_dir: str, worker_name: str, label: str):
    """Create a trace handler that saves to TensorBoard and prints summary."""

    def handler(prof):
        torch.profiler.tensorboard_trace_handler(output_dir, worker_name)(prof)
        print(f"\n{'=' * 80}")
        print(label)
        print(f"{'=' * 80}")
        print(prof.key_averages().table(sort_by="cuda_time_total", row_limit=20))

    return handler
```
**EN:** `_make_trace_handler` Create a trace handler that saves to TensorBoard and prints summary. It mainly works with `output_dir`, `worker_name`, `label` and relies on `torch.profiler.tensorboard_trace_handler`, `print`, `prof.key_averages.table`, `prof.key_averages` plus value production to move data through this part of the benchmark pipeline.
**CN:** `_make_trace_handler` 的职责是：Create a trace handler that saves to TensorBoard and prints summary。 它主要处理 `output_dir`, `worker_name`, `label`，并结合 `torch.profiler.tensorboard_trace_handler`, `print`, `prof.key_averages.table`, `prof.key_averages` 以及 结果返回 来完成这一段基准测试流程。

### Function `run_profile` (lines 190-256)
```python
def run_profile(
    seq_len: int,
    num_heads: int,
    head_dim: int,
    warmup: int,
    output_dir: str,
):
    """Profile FP8 vs BF16 attention with PyTorch profiler."""
    attn_fp8, attn_bf16, workspace = _setup_fp8_attention(num_heads, head_dim)

    torch.manual_seed(42)
    q = torch.randn(
        seq_len,
        num_heads,
        head_dim,
        device="cuda",
        dtype=torch.bfloat16,
    )
    k = torch.randn_like(q)
    v = torch.randn_like(q)

    cu_fp8, max_s, seq_l = _build_meta(seq_len, num_heads, head_dim, fp8=True)
    # we can reuse cu_fp8 for cu_bf16 since q, k, and v are contiguous
    cu_bf16 = cu_fp8.clone()

    sched = torch.profiler.schedule(wait=0, warmup=warmup, active=1)

    # Profile BF16 (warmup handled by profiler schedule)
    with profile(
        activities=[ProfilerActivity.CPU, ProfilerActivity.CUDA],
        schedule=sched,
        on_trace_ready=_make_trace_handler(
            output_dir,
            f"bf16_h{head_dim}_s{seq_len}",
    # ... omitted for brevity ...
                attn_fp8._forward_flashinfer(
                    q.clone(), k.clone(), v.clone(), cu_fp8, max_s, seq_l
                )
                torch.accelerator.synchronize()
            prof_fp8.step()

    print(f"\nTensorBoard traces saved to: {output_dir}")
    print(f"View with: tensorboard --logdir={output_dir}")
```
**EN:** `run_profile` Profile FP8 vs BF16 attention with PyTorch profiler. It mainly works with `seq_len`, `num_heads`, `head_dim`, `warmup`, `output_dir` and relies on `_setup_fp8_attention`, `torch.manual_seed`, `torch.randn`, `torch.randn_like`, `_build_meta`, `cu_fp8.clone` plus iteration and context management to move data through this part of the benchmark pipeline.
**CN:** `run_profile` 的职责是：Profile FP8 vs BF16 attention with PyTorch profiler。 它主要处理 `seq_len`, `num_heads`, `head_dim`, `warmup`, `output_dir`，并结合 `_setup_fp8_attention`, `torch.manual_seed`, `torch.randn`, `torch.randn_like`, `_build_meta`, `cu_fp8.clone` 以及 循环迭代、上下文管理 来完成这一段基准测试流程。

### Entry point (lines 259-324)
```python
if __name__ == "__main__":
    parser = FlexibleArgumentParser(description="Benchmark FP8 vs BF16 ViT attention.")
    parser.add_argument(
        "--seq-lens",
        type=int,
        nargs="+",
        default=DEFAULT_SEQ_LENS,
        help="Sequence lengths to benchmark",
    )
    parser.add_argument(
        "--num-heads",
        type=int,
        default=NUM_HEADS,
    )
    parser.add_argument(
        "--head-dim",
        type=int,
        default=HEAD_DIM,
    )
    parser.add_argument(
        "--method",
        choices=["cupti", "cudagraph"],
        default="cudagraph",
        help="GPU timing method: cupti (CUPTI kernel timing) or "
        "cudagraph (CUDA graph capture/replay). Default: cudagraph",
    )
    parser.add_argument(
        "--warmup",
        type=int,
        default=10,
        help="Warmup iterations (profile mode only)",
    )
    parser.add_argument(
        "--profile",
    # ... omitted for brevity ...
        )
    else:
        run_benchmark(
            args.seq_lens,
            args.num_heads,
            args.head_dim,
            args.method,
        )
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `run_profile`, `run_benchmark` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `run_profile`, `run_benchmark` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Attention specialization: the code studies attention backends, cache layouts, or MLA variants.
- **CN:** 注意力专项：代码研究注意力后端、缓存布局或 MLA 变体。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: `functools`.
- **CN:** 标准库依赖：`functools`。
- **EN:** Third-party packages: `numpy`, `torch`, `torch.profiler`.
- **CN:** 第三方依赖：`numpy`, `torch`, `torch.profiler`。
- **EN:** Internal modules: `vllm.utils.argparse_utils`.
- **CN:** 内部模块：`vllm.utils.argparse_utils`。
