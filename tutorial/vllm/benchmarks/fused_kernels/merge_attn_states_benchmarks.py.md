# merge_attn_states_benchmarks.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/fused_kernels/merge_attn_states_benchmarks.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, centered around `NUM_TOKENS_LIST`, `HEAD_CONFIGS`, `TP_SIZES`, `INPUT_DTYPES`. / 实现与基准测试编排相关的逻辑，核心符号包括 `NUM_TOKENS_LIST`, `HEAD_CONFIGS`, `TP_SIZES`, `INPUT_DTYPES`。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 3-17)
```python
"""
Benchmark: Fused FP8 output quantization in merge_attn_states

Compares fused vs unfused approaches for producing FP8-quantized merged
attention output:
  1. Fused CUDA     -- single CUDA kernel (merge + FP8 quant)
  2. Fused Triton   -- single Triton kernel (merge + FP8 quant)
  3. Unfused CUDA   -- CUDA merge + torch.compiled FP8 quant
  4. Unfused Triton  -- Triton merge + torch.compiled FP8 quant

Usage:
    python benchmarks/fused_kernels/merge_attn_states_benchmarks.py
    python benchmarks/fused_kernels/merge_attn_states_benchmarks.py --tp 1 4 8
    python benchmarks/fused_kernels/merge_attn_states_benchmarks.py --dtype bfloat16
"""
```
**EN:** The module docstring introduces Benchmark: Fused FP8 output quantization in merge_attn_states Compares fused vs unfused approaches for producing FP8-quantized merged attention output: 1. It gives readers the context needed before the implementation details start.
**CN:** 模块文档字符串先说明了 Benchmark: Fused FP8 output quantization in merge_attn_states Compares fused vs unfused approaches for producing FP8-quantized merged attention output: 1 的背景与用途，在进入实现细节前先建立阅读上下文。

### Imports (lines 19-32)
```python
import argparse
import itertools

import torch

from vllm._custom_ops import merge_attn_states as merge_attn_states_cuda
from vllm.benchmarks.lib.utils import default_vllm_config
from vllm.model_executor.layers.quantization.input_quant_fp8 import QuantFP8
from vllm.model_executor.layers.quantization.utils.quant_utils import GroupShape
from vllm.platforms import current_platform
from vllm.triton_utils import triton
from vllm.v1.attention.ops.triton_merge_attn_states import (
    merge_attn_states as merge_attn_states_triton,
)
```
**EN:** This block gathers standard-library helpers such as `argparse`, `itertools`; third-party packages such as `torch`; project-local modules such as `vllm._custom_ops`, `vllm.benchmarks.lib.utils`, `vllm.model_executor.layers.quantization.input_quant_fp8`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.triton_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`, `itertools`；第三方依赖，如 `torch`；项目内部模块，如 `vllm._custom_ops`, `vllm.benchmarks.lib.utils`, `vllm.model_executor.layers.quantization.input_quant_fp8`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.triton_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 38-51)
```python
NUM_TOKENS_LIST = [1, 16, 64, 256, 1024, 4096]

# (label, num_heads, head_size) — num_heads is for TP=1
HEAD_CONFIGS = [
    ("DeepSeek-V3 MLA", 128, 128),
    ("Llama-70B", 64, 128),
    ("Llama-8B", 32, 128),
]

TP_SIZES = [1, 2, 4, 8]

INPUT_DTYPES = [torch.float32, torch.float16, torch.bfloat16]

QUANTILES = [0.5, 0.2, 0.8]
```
**EN:** This top-level block prepares shared state such as `NUM_TOKENS_LIST`, `HEAD_CONFIGS`, `TP_SIZES`, `INPUT_DTYPES`, `QUANTILES`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `NUM_TOKENS_LIST`, `HEAD_CONFIGS`, `TP_SIZES`, `INPUT_DTYPES`, `QUANTILES`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `short_dtype` (lines 59-60)
```python
def short_dtype(dtype: torch.dtype) -> str:
    return str(dtype).removeprefix("torch.")
```
**EN:** `short_dtype` implements a helper used by `merge_attn_states_benchmarks.py`. It mainly works with `dtype` and relies on `str.removeprefix`, `str` plus value production to move data through this part of the benchmark pipeline.
**CN:** `short_dtype` 负责实现 `merge_attn_states_benchmarks.py` 使用的辅助逻辑。 它主要处理 `dtype`，并结合 `str.removeprefix`, `str` 以及 结果返回 来完成这一段基准测试流程。

### Function `make_inputs` (lines 63-83)
```python
def make_inputs(
    num_tokens: int,
    num_heads: int,
    head_size: int,
    dtype: torch.dtype,
):
    """Create random prefix/suffix outputs and LSEs."""
    prefix_output = torch.randn(
        (num_tokens, num_heads, head_size), dtype=dtype, device="cuda"
    )
    suffix_output = torch.randn(
        (num_tokens, num_heads, head_size), dtype=dtype, device="cuda"
    )
    prefix_lse = torch.randn(num_heads, num_tokens, dtype=torch.float32, device="cuda")
    suffix_lse = torch.randn(num_heads, num_tokens, dtype=torch.float32, device="cuda")
    # Sprinkle some inf values to exercise edge-case paths
    mask = torch.rand(num_heads, num_tokens, device="cuda") < 0.05
    prefix_lse[mask] = float("inf")
    mask2 = torch.rand(num_heads, num_tokens, device="cuda") < 0.05
    suffix_lse[mask2] = float("inf")
    return prefix_output, suffix_output, prefix_lse, suffix_lse
```
**EN:** `make_inputs` Create random prefix/suffix outputs and LSEs. It mainly works with `num_tokens`, `num_heads`, `head_size`, `dtype` and relies on `torch.randn`, `torch.rand`, `float` plus value production to move data through this part of the benchmark pipeline.
**CN:** `make_inputs` 的职责是：Create random prefix/suffix outputs and LSEs。 它主要处理 `num_tokens`, `num_heads`, `head_size`, `dtype`，并结合 `torch.randn`, `torch.rand`, `float` 以及 结果返回 来完成这一段基准测试流程。

### Function `build_configs` (lines 86-96)
```python
def build_configs(head_configs, num_tokens_list, input_dtypes, tp_sizes):
    """Build (num_tokens, num_heads, head_size, dtype_str) config tuples,
    applying TP division to num_heads and skipping invalid combos."""
    configs = []
    for (_, nh, hs), nt, dtype, tp in itertools.product(
        head_configs, num_tokens_list, input_dtypes, tp_sizes
    ):
        nh_tp = nh // tp
        if nh_tp >= 1:
            configs.append((nt, nh_tp, hs, short_dtype(dtype)))
    return configs
```
**EN:** `build_configs` Build (num_tokens, num_heads, head_size, dtype_str) config tuples, applying TP division to num_heads and skipping invalid combos. It mainly works with `head_configs`, `num_tokens_list`, `input_dtypes`, `tp_sizes` and relies on `itertools.product`, `configs.append`, `short_dtype` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `build_configs` 的职责是：Build (num_tokens, num_heads, head_size, dtype_str) config tuples, applying TP division to num_heads and skipping invalid combos。 它主要处理 `head_configs`, `num_tokens_list`, `input_dtypes`, `tp_sizes`，并结合 `itertools.product`, `configs.append`, `short_dtype` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `parse_args` (lines 99-125)
```python
def parse_args():
    parser = argparse.ArgumentParser(
        description="Benchmark merge_attn_states fused FP8 quantization"
    )
    parser.add_argument(
        "--num-tokens",
        type=int,
        nargs="+",
        default=None,
        help=f"Override token counts (default: {NUM_TOKENS_LIST})",
    )
    parser.add_argument(
        "--tp",
        type=int,
        nargs="+",
        default=None,
        help=f"TP sizes to simulate (divides num_heads) (default: {TP_SIZES})",
    )
    parser.add_argument(
        "--dtype",
        type=str,
        nargs="+",
        default=None,
        help="Input dtypes (e.g. bfloat16 float16 float32). "
        f"Default: {[short_dtype(d) for d in INPUT_DTYPES]}",
    )
    return parser.parse_args()
```
**EN:** `parse_args` parses external input into structured benchmark settings. It mainly works with no explicit parameters and relies on `argparse.ArgumentParser`, `parser.add_argument`, `short_dtype`, `parser.parse_args` plus value production to move data through this part of the benchmark pipeline.
**CN:** `parse_args` 负责把外部输入解析为结构化的基准配置。 它主要处理 无显式参数，并结合 `argparse.ArgumentParser`, `parser.add_argument`, `short_dtype`, `parser.parse_args` 以及 结果返回 来完成这一段基准测试流程。

### Top-level setup (lines 132-146)
```python
args = parse_args()

num_tokens_list = args.num_tokens if args.num_tokens else NUM_TOKENS_LIST
tp_sizes = args.tp if args.tp else TP_SIZES

if args.dtype:
    from vllm.utils.torch_utils import STR_DTYPE_TO_TORCH_DTYPE

    input_dtypes = [STR_DTYPE_TO_TORCH_DTYPE[d] for d in args.dtype]
else:
    input_dtypes = INPUT_DTYPES

configs = build_configs(HEAD_CONFIGS, num_tokens_list, input_dtypes, tp_sizes)

torch._dynamo.config.recompile_limit = 8888
```
**EN:** This top-level block prepares shared state such as `args`, `num_tokens_list`, `tp_sizes`, `configs`, `torch`. It uses `parse_args`, `build_configs` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `args`, `num_tokens_list`, `tp_sizes`, `configs`, `torch`。它借助 `parse_args`, `build_configs` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `benchmark` (lines 168-244)
```python
def benchmark(num_tokens, num_heads, head_size, dtype_str, provider):
    input_dtype = getattr(torch, dtype_str)
    fp8_dtype = current_platform.fp8_dtype()
    prefix_out, suffix_out, prefix_lse, suffix_lse = make_inputs(
        num_tokens, num_heads, head_size, input_dtype
    )
    output_scale = torch.tensor([0.1], dtype=torch.float32, device="cuda")

    if provider == "fused_cuda":
        output = torch.empty(
            (num_tokens, num_heads, head_size), dtype=fp8_dtype, device="cuda"
        )
        fn = lambda: merge_attn_states_cuda(
            output,
            prefix_out,
            prefix_lse,
            suffix_out,
            suffix_lse,
            output_scale=output_scale,
        )
    elif provider == "fused_triton":
        output = torch.empty(
            (num_tokens, num_heads, head_size), dtype=fp8_dtype, device="cuda"
        )
        fn = lambda: merge_attn_states_triton(
            output,
            prefix_out,
            prefix_lse,
            suffix_out,
            suffix_lse,
            output_scale=output_scale,
        )
    elif provider == "unfused_cuda":
        merge_buf = torch.empty(
    # ... omitted for brevity ...
                merge_buf, prefix_out, prefix_lse, suffix_out, suffix_lse
            )
            compiled_quant(quant_input, output_scale)

        fn = unfused_fn

    ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(fn, quantiles=QUANTILES)
    return 1000 * ms, 1000 * max_ms, 1000 * min_ms  # us
```
**EN:** `benchmark` coordinates or measures benchmark orchestration. It mainly works with `num_tokens`, `num_heads`, `head_size`, `dtype_str`, `provider` and relies on `getattr`, `current_platform.fp8_dtype`, `make_inputs`, `torch.tensor`, `torch.empty`, `merge_attn_states_cuda` plus branching to move data through this part of the benchmark pipeline.
**CN:** `benchmark` 负责协调或测量基准测试编排。 它主要处理 `num_tokens`, `num_heads`, `head_size`, `dtype_str`, `provider`，并结合 `getattr`, `current_platform.fp8_dtype`, `make_inputs`, `torch.tensor`, `torch.empty`, `merge_attn_states_cuda` 以及 条件分支 来完成这一段基准测试流程。

### Function `main` (lines 252-259)
```python
def main():
    device_name = current_platform.get_device_name()
    print(f"Device: {device_name}")
    print(f"Token counts: {num_tokens_list}")
    print(f"TP sizes: {tp_sizes}")
    print(f"Input dtypes: {[short_dtype(d) for d in input_dtypes]}")
    print(f"Head configs: {[(c[0], c[1], c[2]) for c in HEAD_CONFIGS]}")
    benchmark.run(print_data=True)
```
**EN:** `main` serves as the main entry point for this script. It mainly works with no explicit parameters and relies on `current_platform.get_device_name`, `print`, `short_dtype`, `benchmark.run` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 无显式参数，并结合 `current_platform.get_device_name`, `print`, `short_dtype`, `benchmark.run` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Entry point (lines 262-264)
```python
if __name__ == "__main__":
    with torch.inference_mode():
        main()
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `torch.inference_mode`, `main` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `torch.inference_mode`, `main` 等调用串起来，使模块能够从命令行直接启动。

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
- **EN:** Standard library: `argparse`, `itertools`.
- **CN:** 标准库依赖：`argparse`, `itertools`。
- **EN:** Third-party packages: `torch`.
- **CN:** 第三方依赖：`torch`。
- **EN:** Internal modules: `vllm._custom_ops`, `vllm.benchmarks.lib.utils`, `vllm.model_executor.layers.quantization.input_quant_fp8`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.triton_utils`.
- **CN:** 内部模块：`vllm._custom_ops`, `vllm.benchmarks.lib.utils`, `vllm.model_executor.layers.quantization.input_quant_fp8`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.triton_utils`。
