# deepseek_v4_rope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/deepseek_v4_rope.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements rotary position embedding logic for the SGLang SRT runtime. It exposes symbols such as `precompute_freqs_cis`, `apply_rotary_emb_triton_kernel`, and `apply_rotary_emb_triton` and connects them to backend-specific paths such as `Triton`. / 该模块为 SGLang 的 SRT 运行时实现了旋转位置编码逻辑。它提供了 `precompute_freqs_cis`、`apply_rotary_emb_triton_kernel` 以及 `apply_rotary_emb_triton` 等符号，并把这些符号连接到 `Triton` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Imports, constants, and runtime setup
```python
import math
from functools import lru_cache
from typing import Optional

import tilelang
import torch
import triton
import triton.language as tl

tilelang.set_log_level("WARNING")

pass_configs = {
    tilelang.PassConfigKey.TL_DISABLE_WARP_SPECIALIZED: True,
    tilelang.PassConfigKey.TL_DISABLE_TMA_LOWER: True,
}

FP8 = "float8_e4m3"
BF16 = "bfloat16"
FP32 = "float32"
INT32 = "int32"
```
**EN:** This section prepares the module namespace. It imports `math`, `functools.lru_cache`, `typing.Optional`, `tilelang`, `torch`, and `triton`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `pass_configs`, `FP8`, `BF16`, `FP32`, and `INT32` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `math`、`functools.lru_cache`、`typing.Optional`、`tilelang`、`torch` 以及 `triton`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `pass_configs`、`FP8`、`BF16`、`FP32` 以及 `INT32` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 23-60: Function `precompute_freqs_cis` and its core logic
```python
@lru_cache(2)
def precompute_freqs_cis(
    dim, seqlen, original_seq_len, base, factor, beta_fast, beta_slow
) -> torch.Tensor:

    def find_correction_dim(num_rotations, dim, base, max_seq_len):
        return (
            dim
            * math.log(max_seq_len / (num_rotations * 2 * math.pi))
            / (2 * math.log(base))
        )

    def find_correction_range(low_rot, high_rot, dim, base, max_seq_len):
        low = math.floor(find_correction_dim(low_rot, dim, base, max_seq_len))
        high = math.ceil(find_correction_dim(high_rot, dim, base, max_seq_len))
        return max(low, 0), min(high, dim - 1)

    def linear_ramp_factor(min, max, dim):
        if min == max:
            max += 0.001
        linear_func = (torch.arange(dim, dtype=torch.float32) - min) / (max - min)
        ramp_func = torch.clamp(linear_func, 0, 1)
        return ramp_func

    freqs = 1.0 / (base ** (torch.arange(0, dim, 2, dtype=torch.float32) / dim))
    if original_seq_len > 0:
        low, high = find_correction_range(
            beta_fast, beta_slow, dim, base, original_seq_len
        )
        smooth = 1 - linear_ramp_factor(low, high, dim // 2)
        freqs = freqs / factor * (1 - smooth) + freqs * smooth

    t = torch.arange(seqlen)
    freqs = torch.outer(t, freqs)
    freqs_cis = torch.polar(torch.ones_like(freqs), freqs)
    return freqs_cis
```
**EN:** This block defines `precompute_freqs_cis` and contains the main logic for this step. Decorators like `lru_cache` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `lru_cache`, `torch.arange`, `torch.outer`, `torch.polar`, and `math.floor`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `freqs`, `t`, `freqs_cis`, `low`, and `high` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `precompute_freqs_cis`，并承载这一阶段的核心逻辑。 像 `lru_cache` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `lru_cache`、`torch.arange`、`torch.outer`、`torch.polar` 以及 `math.floor`，说明该流程会编排底层辅助函数或计算内核。 像 `freqs`、`t`、`freqs_cis`、`low` 以及 `high` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 61-116: `apply_rotary_emb_triton_kernel` apply step for rotary embedding emb Triton kernel
```python
@triton.jit
def apply_rotary_emb_triton_kernel(
    x_ptr,
    freqs_ptr,
    positions_ptr,
    rope_dim,
    stride_x_batch,
    stride_x_head,
    stride_x_dim,
    stride_freq_pos,
    stride_freq_dim,
    USE_POS: tl.constexpr,
    IS_INVERSE: tl.constexpr,
    IS_3D: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
):
    pid_batch = tl.program_id(0)
    pid_head = tl.program_id(1)
    pid_dim = tl.program_id(2)

    if USE_POS:
        position = tl.load(positions_ptr + pid_batch)
    else:
        position = pid_batch

    if IS_3D:
        base_offset = pid_batch * stride_x_batch + pid_head * stride_x_head
    else:
        base_offset = pid_batch * stride_x_batch

    offs_pair = pid_dim * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
    mask = offs_pair < (rope_dim // 2)

    offs_x_real = base_offset + offs_pair * 2 * stride_x_dim
    offs_x_imag = base_offset + (offs_pair * 2 + 1) * stride_x_dim

    x_real = tl.load(x_ptr + offs_x_real, mask=mask, other=0.0).to(tl.float32)
    x_imag = tl.load(x_ptr + offs_x_imag, mask=mask, other=0.0).to(tl.float32)

    offs_freq_real = position * stride_freq_pos + offs_pair * 2 * stride_freq_dim
    offs_freq_imag = position * stride_freq_pos + (offs_pair * 2 + 1) * stride_freq_dim

    freq_real = tl.load(freqs_ptr + offs_freq_real, mask=mask, other=0.0)
    freq_imag = tl.load(freqs_ptr + offs_freq_imag, mask=mask, other=0.0)

    if IS_INVERSE:
        out_real = x_real * freq_real + x_imag * freq_imag
        out_imag = x_imag * freq_real - x_real * freq_imag
    else:
        out_real = x_real * freq_real - x_imag * freq_imag
        out_imag = x_real * freq_imag + x_imag * freq_real

    tl.store(x_ptr + offs_x_real, out_real, mask=mask)
    tl.store(x_ptr + offs_x_imag, out_imag, mask=mask)
```
**EN:** This block defines `apply_rotary_emb_triton_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.load.to`, `tl.load`, `tl.store`, and `tl.arange`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid_batch`, `pid_head`, `pid_dim`, `offs_pair`, and `mask` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `apply_rotary_emb_triton_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.load.to`、`tl.load`、`tl.store` 以及 `tl.arange`，说明该流程会编排底层辅助函数或计算内核。 像 `pid_batch`、`pid_head`、`pid_dim`、`offs_pair` 以及 `mask` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 117-179: `apply_rotary_emb_triton` apply step for rotary embedding emb Triton
```python
def apply_rotary_emb_triton(
    x: torch.Tensor,
    freqs_cis: torch.Tensor,
    positions: Optional[torch.Tensor] = None,
    inverse: bool = False,
) -> torch.Tensor:
    is_3d = x.ndim == 3

    if is_3d:
        batch_size, n_heads, rope_dim = x.shape
    else:
        batch_size, rope_dim = x.shape
        n_heads = 1

    freqs_real = torch.view_as_real(freqs_cis).flatten(-2)

    BLOCK_SIZE = 128

    num_blocks_dim = triton.cdiv(rope_dim // 2, BLOCK_SIZE)
    grid = (batch_size, n_heads if is_3d else 1, num_blocks_dim)

    if positions is not None:
        assert positions.shape == (
            batch_size,
        ), f"positions shape {positions.shape} != ({batch_size},)"

        apply_rotary_emb_triton_kernel[grid](
            x,
            freqs_real,
            positions,
            rope_dim,
            x.stride(0),
            x.stride(1) if is_3d else 0,
            x.stride(-1),
            freqs_real.stride(0),
            freqs_real.stride(1),
            USE_POS=True,
            IS_INVERSE=inverse,
            IS_3D=is_3d,
            BLOCK_SIZE=BLOCK_SIZE,
        )
    else:
        assert (
            freqs_real.shape[0] == batch_size
        ), f"freqs_cis batch size {freqs_real.shape[0]} != x batch size {batch_size}"

        apply_rotary_emb_triton_kernel[grid](
            x,
            freqs_real,
            None,
            rope_dim,
            x.stride(0),
            x.stride(1) if is_3d else 0,
            x.stride(-1),
            freqs_real.stride(0),
            freqs_real.stride(1),
            USE_POS=False,
            IS_INVERSE=inverse,
            IS_3D=is_3d,
            BLOCK_SIZE=BLOCK_SIZE,
        )

    return x
```
**EN:** This block defines `apply_rotary_emb_triton` and contains the main logic for this step. It mainly invokes `torch.view_as_real.flatten`, `triton.cdiv`, `apply_rotary_emb_triton_kernel`, `torch.view_as_real`, and `x.stride`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `is_3d`, `freqs_real`, `BLOCK_SIZE`, `num_blocks_dim`, and `grid` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `apply_rotary_emb_triton`，并承载这一阶段的核心逻辑。 它主要调用 `torch.view_as_real.flatten`、`triton.cdiv`、`apply_rotary_emb_triton_kernel`、`torch.view_as_real` 以及 `x.stride`，说明该流程会编排底层辅助函数或计算内核。 像 `is_3d`、`freqs_real`、`BLOCK_SIZE`、`num_blocks_dim` 以及 `grid` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `precompute_freqs_cis`, `apply_rotary_emb_triton_kernel`, and `apply_rotary_emb_triton`. / **主要符号**：核心入口包括 `precompute_freqs_cis`、`apply_rotary_emb_triton_kernel` 以及 `apply_rotary_emb_triton`。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Position encoding**: Describes how rotary embeddings or related position transforms are applied. / **位置编码**：说明如何应用旋转位置编码或相关位置变换。

## Dependencies / 依赖关系
- **Standard library**: `math`, `functools.lru_cache`, and `typing.Optional` / **标准库**：`math`、`functools.lru_cache` 以及 `typing.Optional`
- **Third-party**: `tilelang`, `torch`, `triton`, and `triton.language` / **第三方依赖**：`tilelang`、`torch`、`triton` 以及 `triton.language`
