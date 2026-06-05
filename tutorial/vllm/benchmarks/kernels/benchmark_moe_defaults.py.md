# benchmark_moe_defaults.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_moe_defaults.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, mixture-of-experts benchmarks, centered around `FP8_DTYPE`, `old_default_config`, `benchmark_config`, `MODELS`. / 实现与基准测试编排、混合专家基准相关的逻辑，核心符号包括 `FP8_DTYPE`, `old_default_config`, `benchmark_config`, `MODELS`。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 4-16)
```python
"""
Benchmark comparing old vs new default fused MoE configs.

Runs the triton fused_moe kernel with three configurations for each scenario:
  1. Tuned config (from JSON file, if available) — the target to match
  2. Old default (the hardcoded defaults before this change)
  3. New default (the improved defaults)

Usage:
    python benchmarks/kernels/benchmark_moe_defaults.py

Produces a table showing kernel time (us) and speedup of new vs old defaults.
"""
```
**EN:** The module docstring introduces Benchmark comparing old vs new default fused MoE configs. It gives readers the context needed before the implementation details start.
**CN:** 模块文档字符串先说明了 Benchmark comparing old vs new default fused MoE configs 的背景与用途，在进入实现细节前先建立阅读上下文。

### Imports (lines 18-29)
```python
import torch

from vllm.model_executor.layers.fused_moe import fused_topk, override_config
from vllm.model_executor.layers.fused_moe.config import FusedMoEQuantConfig
from vllm.model_executor.layers.fused_moe.fused_moe import (
    fused_experts,
    get_default_config,
    get_moe_configs,
)
from vllm.platforms import current_platform
from vllm.triton_utils import triton
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block gathers third-party packages such as `torch`; project-local modules such as `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.fused_moe`, `vllm.platforms`, `vllm.triton_utils`, `vllm.utils.torch_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了第三方依赖，如 `torch`；项目内部模块，如 `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.fused_moe`, `vllm.platforms`, `vllm.triton_utils`, `vllm.utils.torch_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 31-31)
```python
FP8_DTYPE = current_platform.fp8_dtype()
```
**EN:** This top-level block prepares shared state such as `FP8_DTYPE`. It uses `current_platform.fp8_dtype` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `FP8_DTYPE`。它借助 `current_platform.fp8_dtype` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `old_default_config` (lines 34-62)
```python
def old_default_config(M, E, N, K, topk, dtype=None, block_shape=None):
    """The original defaults before https://github.com/vllm-project/vllm/pull/34846,
    for comparison."""
    if dtype == "fp8_w8a8" and block_shape is not None:
        return {
            "BLOCK_SIZE_M": 64,
            "BLOCK_SIZE_N": block_shape[0],
            "BLOCK_SIZE_K": block_shape[1],
            "GROUP_SIZE_M": 32,
            "SPLIT_K": 1,
            "num_warps": 4,
            "num_stages": 3 if not current_platform.is_rocm() else 2,
        }
    elif M <= E:
        return {
            "BLOCK_SIZE_M": 16,
            "BLOCK_SIZE_N": 32,
            "BLOCK_SIZE_K": 64,
            "GROUP_SIZE_M": 1,
            "SPLIT_K": 1,
        }
    else:
        return {
            "BLOCK_SIZE_M": 64,
            "BLOCK_SIZE_N": 64,
            "BLOCK_SIZE_K": 32,
            "GROUP_SIZE_M": 8,
            "SPLIT_K": 1,
        }
```
**EN:** `old_default_config` The original defaults before https://github.com/vllm-project/vllm/pull/34846, for comparison. It mainly works with `M`, `E`, `N`, `K`, `topk`, ... and relies on `current_platform.is_rocm` plus branching to move data through this part of the benchmark pipeline.
**CN:** `old_default_config` 的职责是：The original defaults before https://github.com/vllm-project/vllm/pull/34846, for comparison。 它主要处理 `M`, `E`, `N`, `K`, `topk`, ...，并结合 `current_platform.is_rocm` 以及 条件分支 来完成这一段基准测试流程。

### Function `benchmark_config` (lines 65-153)
```python
def benchmark_config(
    config,
    M,
    E,
    N,
    K,
    topk,
    dtype,
    use_fp8=False,
    block_shape=None,
    num_iters=100,
):
    """Time a single kernel config. Returns kernel time in microseconds."""
    init_dtype = torch.float16 if use_fp8 else dtype

    a = torch.randn(M, K, device="cuda", dtype=init_dtype) / 10
    w1 = torch.randn(E, 2 * N, K, device="cuda", dtype=init_dtype) / 10
    w2 = torch.randn(E, K, N, device="cuda", dtype=init_dtype) / 10

    w1_scale = None
    w2_scale = None
    a1_scale = None
    a2_scale = None
    if use_fp8:
        if block_shape is not None:
            bsn, bsk = block_shape
            n_tiles_w1 = triton.cdiv(2 * N, bsn)
            k_tiles_w1 = triton.cdiv(K, bsk)
            n_tiles_w2 = triton.cdiv(K, bsn)
            k_tiles_w2 = triton.cdiv(N, bsk)
            w1_scale = torch.rand(
                E, n_tiles_w1, k_tiles_w1, device="cuda", dtype=torch.float32
            )
            w2_scale = torch.rand(
    # ... omitted for brevity ...
                w2,
                topk_weights,
                topk_ids,
                quant_config=quant_config,
            )
    end.record()
    torch.accelerator.synchronize()
    return start.elapsed_time(end) / num_iters * 1000  # ms -> us
```
**EN:** `benchmark_config` Time a single kernel config. It mainly works with `config`, `M`, `E`, `N`, `K`, ... and relies on `torch.randn`, `triton.cdiv`, `torch.rand`, `w1.to`, `w2.to`, `FusedMoEQuantConfig.make` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `benchmark_config` 的职责是：Time a single kernel config。 它主要处理 `config`, `M`, `E`, `N`, `K`, ...，并结合 `torch.randn`, `triton.cdiv`, `torch.rand`, `w1.to`, `w2.to`, `FusedMoEQuantConfig.make` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Top-level setup (lines 158-195)
```python
MODELS = [
    # --- Few experts ---
    ("Mixtral bf16", 8, 7168, 4096, 2, None, False, None),
    ("Mixtral fp8", 8, 7168, 4096, 2, "fp8_w8a8", True, None),
    # --- Many experts: real model shapes at tp=1 ---
    # Qwen2-MoE-57B: E=60, topk=4, N=1408, K=2048
    ("Qwen2-MoE bf16", 60, 1408, 2048, 4, None, False, None),
    # DeepSeek-V2: E=64, topk=6, N=1407, K=4096
    # (use 1408 to avoid odd alignment; real model is 1407)
    ("DeepSeek-V2 bf16", 64, 1408, 4096, 6, None, False, None),
    # OLMoE-7B: E=64, topk=8, N=2048, K=2048
    ("OLMoE bf16", 64, 2048, 2048, 8, None, False, None),
    # GLM-4-100B-A10B: E=128, topk=8, N=1408, K=4096
    ("GLM-4-MoE bf16", 128, 1408, 4096, 8, None, False, None),
    # Qwen3-30B-A3B: E=128, topk=8, N=768, K=2048
    ("Qwen3-MoE bf16", 128, 768, 2048, 8, None, False, None),
    # DeepSeek-V3 / MiMo-V2-Flash: E=256, topk=8, N=2048, K=7168
    ("DeepSeek-V3 bf16", 256, 2048, 7168, 8, None, False, None),
    # Qwen3.5-70B-A22B (Qwen3-Next): E=512, topk=10, N=512, K=2048
    ("Qwen3-Next bf16", 512, 512, 2048, 10, None, False, None),
    # E=128 N=1856 bf16
    ("E128 N1856 bf16", 128, 1856, 4096, 8, None, False, None),
    # E=256 N=512 bf16 (DS-V3 tp=4)
    ("DS-V3 tp4 bf16", 256, 512, 7168, 8, None, False, None),
    # E=512 N=512 bf16 (Qwen3-Next tp=1)
    ("Qwen3-Next bf16", 512, 512, 2048, 10, None, False, None),
    # E=512 N=256 bf16 (Qwen3-Next tp=2)
    ("Qwen3-Next tp2", 512, 256, 2048, 10, None, False, None),
    # --- FP8 block quant (many experts) ---
    # DS-V3 tp=4: E=256, N=512, fp8 block
    ("DS-V3 tp4 fp8blk", 256, 512, 7168, 8, "fp8_w8a8", True, [128, 128]),
    # DS-V3 tp=8: E=256, N=256, fp8 block
    ("DS-V3 tp8 fp8blk", 256, 256, 7168, 8, "fp8_w8a8", True, [128, 128]),
    # Qwen3-Next tp=2 fp8 block
    ("Qwen3-Next tp2 fp8blk", 512, 256, 2048, 10, "fp8_w8a8", True, [128, 128]),
]

BATCH_SIZES = [1, 4, 8, 16, 32, 64, 128, 256, 512, 1024, 2048, 4096]
```
**EN:** This top-level block prepares shared state such as `MODELS`, `BATCH_SIZES`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `MODELS`, `BATCH_SIZES`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `main` (lines 198-274)
```python
def main():
    set_random_seed(0)
    torch.set_default_device("cuda")
    dtype = torch.bfloat16

    for name, E, N, K, topk, dtype_str, use_fp8, block_shape in MODELS:
        print(f"\n{'=' * 90}")
        print(f"  {name}  (E={E}, N={N}, K={K}, topk={topk})")
        print(f"{'=' * 90}")

        # Try to load tuned config
        block_n = block_shape[0] if block_shape else None
        block_k = block_shape[1] if block_shape else None
        tuned = get_moe_configs(E, N, dtype_str, block_n, block_k)
        has_tuned = tuned is not None
        print(f"  Tuned config available: {has_tuned}")

        hdr = (
            f"{'Batch':>6} | {'Tuned (us)':>11} | {'Old (us)':>11} | "
            f"{'New (us)':>11} | {'New/Old':>8} | {'New/Tuned':>10}"
        )
        print(f"  {hdr}")
        print(f"  {'-' * len(hdr)}")

        for M in BATCH_SIZES:
            old_cfg = old_default_config(M, E, N, K, topk, dtype_str, block_shape)
            new_cfg = get_default_config(M, E, N, K, topk, dtype_str, block_shape)

            if has_tuned:
                tuned_cfg = tuned[min(tuned.keys(), key=lambda x: abs(x - M))]
                t_tuned = benchmark_config(
                    tuned_cfg,
                    M,
                    E,
    # ... omitted for brevity ...
            ratio_tuned = f"{t_new / t_tuned:10.2f}x" if t_tuned else f"{'N/A':>10}"
            # flag regressions where new default is >5% slower than old
            marker = " <--" if ratio_new_old > 1.05 else ""

            print(
                f"  {M:>6} | {tuned_str} | {t_old:11.2f} | {t_new:11.2f} "
                f"| {ratio_new_old:7.2f}x | {ratio_tuned}{marker}"
            )
```
**EN:** `main` serves as the main entry point for this script. It mainly works with no explicit parameters and relies on `set_random_seed`, `torch.set_default_device`, `print`, `get_moe_configs`, `len`, `old_default_config` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 无显式参数，并结合 `set_random_seed`, `torch.set_default_device`, `print`, `get_moe_configs`, `len`, `old_default_config` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Entry point (lines 277-278)
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
- **EN:** Serving path benchmarking: the script interacts with HTTP services, servers, or request proxies.
- **CN:** 服务路径基准：脚本会与 HTTP 服务、服务器或请求代理交互。
- **EN:** MoE specialization: the benchmark focuses on expert routing, expert GEMMs, or fused MoE kernels.
- **CN:** MoE 专项：该基准聚焦专家路由、专家 GEMM 或融合 MoE 内核。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: none.
- **CN:** 标准库依赖：无。
- **EN:** Third-party packages: `torch`.
- **CN:** 第三方依赖：`torch`。
- **EN:** Internal modules: `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.fused_moe`, `vllm.platforms`, `vllm.triton_utils`, `vllm.utils.torch_utils`.
- **CN:** 内部模块：`vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.fused_moe`, `vllm.platforms`, `vllm.triton_utils`, `vllm.utils.torch_utils`。
