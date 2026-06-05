# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/cutlass_benchmarks/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements shared helper utilities, benchmark orchestration, centered around `to_fp8`, `to_int8`, `to_bf16`, `to_fp16`. / 实现与共享辅助工具、基准测试编排相关的逻辑，核心符号包括 `to_fp8`, `to_int8`, `to_bf16`, `to_fp16`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 6-6)
```python
import torch
```
**EN:** This block gathers third-party packages such as `torch`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了第三方依赖，如 `torch`。这些依赖构成了后续基准测试逻辑的基础。

### Function `to_fp8` (lines 9-13)
```python
def to_fp8(tensor: torch.Tensor) -> torch.Tensor:
    finfo = torch.finfo(torch.float8_e4m3fn)
    return torch.round(tensor.clamp(min=finfo.min, max=finfo.max)).to(
        dtype=torch.float8_e4m3fn
    )
```
**EN:** `to_fp8` coordinates or measures FP8 kernel experiments. It mainly works with `tensor` and relies on `torch.finfo`, `torch.round.to`, `torch.round`, `tensor.clamp` plus value production to move data through this part of the benchmark pipeline.
**CN:** `to_fp8` 负责协调或测量FP8 内核实验。 它主要处理 `tensor`，并结合 `torch.finfo`, `torch.round.to`, `torch.round`, `tensor.clamp` 以及 结果返回 来完成这一段基准测试流程。

### Function `to_int8` (lines 16-17)
```python
def to_int8(tensor: torch.Tensor) -> torch.Tensor:
    return torch.round(tensor.clamp(min=-128, max=127)).to(dtype=torch.int8)
```
**EN:** `to_int8` implements a helper used by `utils.py`. It mainly works with `tensor` and relies on `torch.round.to`, `torch.round`, `tensor.clamp` plus value production to move data through this part of the benchmark pipeline.
**CN:** `to_int8` 负责实现 `utils.py` 使用的辅助逻辑。 它主要处理 `tensor`，并结合 `torch.round.to`, `torch.round`, `tensor.clamp` 以及 结果返回 来完成这一段基准测试流程。

### Function `to_bf16` (lines 20-21)
```python
def to_bf16(tensor: torch.Tensor) -> torch.Tensor:
    return tensor.to(dtype=torch.bfloat16)
```
**EN:** `to_bf16` implements a helper used by `utils.py`. It mainly works with `tensor` and relies on `tensor.to` plus value production to move data through this part of the benchmark pipeline.
**CN:** `to_bf16` 负责实现 `utils.py` 使用的辅助逻辑。 它主要处理 `tensor`，并结合 `tensor.to` 以及 结果返回 来完成这一段基准测试流程。

### Function `to_fp16` (lines 24-25)
```python
def to_fp16(tensor: torch.Tensor) -> torch.Tensor:
    return tensor.to(dtype=torch.float16)
```
**EN:** `to_fp16` implements a helper used by `utils.py`. It mainly works with `tensor` and relies on `tensor.to` plus value production to move data through this part of the benchmark pipeline.
**CN:** `to_fp16` 负责实现 `utils.py` 使用的辅助逻辑。 它主要处理 `tensor`，并结合 `tensor.to` 以及 结果返回 来完成这一段基准测试流程。

### Function `make_rand_tensors` (lines 28-39)
```python
def make_rand_tensors(
    dtype: torch.dtype, m: int, n: int, k: int
) -> tuple[torch.Tensor, torch.Tensor]:
    a = torch.randn((m, k), device="cuda") * 5
    b = torch.randn((n, k), device="cuda").t() * 5

    if dtype == torch.int8:
        return to_int8(a), to_int8(b)
    if dtype == torch.float8_e4m3fn:
        return to_fp8(a), to_fp8(b)

    raise ValueError("unsupported dtype")
```
**EN:** `make_rand_tensors` implements a helper used by `utils.py`. It mainly works with `dtype`, `m`, `n`, `k` and relies on `torch.randn`, `torch.randn.t`, `to_int8`, `to_fp8`, `ValueError` plus branching to move data through this part of the benchmark pipeline.
**CN:** `make_rand_tensors` 负责实现 `utils.py` 使用的辅助逻辑。 它主要处理 `dtype`, `m`, `n`, `k`，并结合 `torch.randn`, `torch.randn.t`, `to_int8`, `to_fp8`, `ValueError` 以及 条件分支 来完成这一段基准测试流程。

## Key Concepts / 关键概念
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: none.
- **CN:** 标准库依赖：无。
- **EN:** Third-party packages: `torch`.
- **CN:** 第三方依赖：`torch`。
- **EN:** Internal modules: none.
- **CN:** 内部模块：无。
