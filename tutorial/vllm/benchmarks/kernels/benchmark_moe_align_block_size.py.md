# benchmark_moe_align_block_size.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_moe_align_block_size.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, mixture-of-experts benchmarks, centered around `get_topk_ids`, `benchmark`. / 实现与基准测试编排、混合专家基准相关的逻辑，核心符号包括 `get_topk_ids`, `benchmark`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-12)
```python
import argparse
import itertools

import torch

from vllm.model_executor.layers.fused_moe.moe_align_block_size import (
    moe_align_block_size,
)
from vllm.triton_utils import triton
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block gathers standard-library helpers such as `argparse`, `itertools`; third-party packages such as `torch`; project-local modules such as `vllm.model_executor.layers.fused_moe.moe_align_block_size`, `vllm.triton_utils`, `vllm.utils.torch_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`, `itertools`；第三方依赖，如 `torch`；项目内部模块，如 `vllm.model_executor.layers.fused_moe.moe_align_block_size`, `vllm.triton_utils`, `vllm.utils.torch_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Function `get_topk_ids` (lines 15-21)
```python
def get_topk_ids(num_tokens: int, num_experts: int, topk: int) -> torch.Tensor:
    return torch.stack(
        [
            torch.randperm(num_experts, dtype=torch.int32, device="cuda")[:topk]
            for _ in range(num_tokens)
        ]
    )
```
**EN:** `get_topk_ids` implements sampling/top-k benchmarks. It mainly works with `num_tokens`, `num_experts`, `topk` and relies on `torch.stack`, `torch.randperm`, `range` plus value production to move data through this part of the benchmark pipeline.
**CN:** `get_topk_ids` 负责实现采样与 top-k 基准。 它主要处理 `num_tokens`, `num_experts`, `topk`，并结合 `torch.stack`, `torch.randperm`, `range` 以及 结果返回 来完成这一段基准测试流程。

### Top-level setup (lines 25-31)
```python
num_tokens_range = [1, 16, 256, 4096]
num_experts_range = [16, 64, 224, 256, 280, 512]
topk_range = [1, 2, 8]
ep_size_range = [1, 8]
configs = list(
    itertools.product(num_tokens_range, num_experts_range, topk_range, ep_size_range)
)
```
**EN:** This top-level block prepares shared state such as `num_tokens_range`, `num_experts_range`, `topk_range`, `ep_size_range`, `configs`. It uses `list`, `itertools.product` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `num_tokens_range`, `num_experts_range`, `topk_range`, `ep_size_range`, `configs`。它借助 `list`, `itertools.product` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `benchmark` (lines 45-68)
```python
def benchmark(num_tokens, num_experts, topk, ep_size, provider):
    """Benchmark function for Triton."""
    block_size = 256
    set_random_seed(0)
    topk_ids = get_topk_ids(num_tokens, num_experts, topk)

    e_map = None
    if ep_size != 1:
        local_e = num_experts // ep_size
        e_ids = torch.randperm(num_experts, device="cuda", dtype=torch.int32)[:local_e]
        e_map = torch.full((num_experts,), -1, device="cuda", dtype=torch.int32)
        e_map[e_ids] = torch.arange(local_e, device="cuda", dtype=torch.int32)

    quantiles = [0.5, 0.2, 0.8]

    if provider == "vllm":
        ms, min_ms, max_ms = triton.testing.do_bench(
            lambda: moe_align_block_size(
                topk_ids, block_size, num_experts, e_map, ignore_invalid_experts=True
            ),
            quantiles=quantiles,
        )

    return 1000 * ms, 1000 * max_ms, 1000 * min_ms
```
**EN:** `benchmark` Benchmark function for Triton. It mainly works with `num_tokens`, `num_experts`, `topk`, `ep_size`, `provider` and relies on `set_random_seed`, `get_topk_ids`, `torch.randperm`, `torch.full`, `torch.arange`, `triton.testing.do_bench` plus branching to move data through this part of the benchmark pipeline.
**CN:** `benchmark` 的职责是：Benchmark function for Triton。 它主要处理 `num_tokens`, `num_experts`, `topk`, `ep_size`, `provider`，并结合 `set_random_seed`, `get_topk_ids`, `torch.randperm`, `torch.full`, `torch.arange`, `triton.testing.do_bench` 以及 条件分支 来完成这一段基准测试流程。

### Entry point (lines 71-88)
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--num_experts",
        type=int,
        default=64,
        choices=[8, 16, 32, 64, 128, 256],
    )
    parser.add_argument(
        "--topk",
        type=int,
        default=8,
        choices=[2, 4, 8],
        help="Top-k value for correctness check.",
    )
    args = parser.parse_args()

    benchmark.run(print_data=True, show_plots=True)
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `benchmark.run` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `benchmark.run` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Result presentation: the module aggregates measurements and renders tables or plots.
- **CN:** 结果呈现：该模块会汇总测量结果并生成表格或图形。
- **EN:** MoE specialization: the benchmark focuses on expert routing, expert GEMMs, or fused MoE kernels.
- **CN:** MoE 专项：该基准聚焦专家路由、专家 GEMM 或融合 MoE 内核。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: `argparse`, `itertools`.
- **CN:** 标准库依赖：`argparse`, `itertools`。
- **EN:** Third-party packages: `torch`.
- **CN:** 第三方依赖：`torch`。
- **EN:** Internal modules: `vllm.model_executor.layers.fused_moe.moe_align_block_size`, `vllm.triton_utils`, `vllm.utils.torch_utils`.
- **CN:** 内部模块：`vllm.model_executor.layers.fused_moe.moe_align_block_size`, `vllm.triton_utils`, `vllm.utils.torch_utils`。
