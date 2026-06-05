# multimodal.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/multimodal.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements multimodal feature handling for the SGLang SRT runtime. It exposes symbols such as `_rotl32`, `_fmix32`, `hash_tiles32_kernel_blocked`, and `add_tree_reduce_u64_kernel` and connects them to backend-specific paths such as `CUDA` and `Triton`. / 该模块为 SGLang 的 SRT 运行时实现了多模态特征处理。它提供了 `_rotl32`、`_fmix32`、`hash_tiles32_kernel_blocked` 以及 `add_tree_reduce_u64_kernel` 等符号，并把这些符号连接到 `CUDA` 和 `Triton` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: File header and module overview
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
```
**EN:** This opening block contains comments, licensing text, or other context that frames the rest of the file before executable code begins.
**CN:** 这一开头部分包含注释、许可证文本或其他上下文信息，用来为后续可执行代码建立背景。

### Lines 14-25: Imports, constants, and runtime setup
```python
"""Logits processing."""

import torch
import triton
import triton.language as tl

FMIX32_C1 = 0x85EBCA6B
FMIX32_C2 = 0xC2B2AE35
POS_C1 = 0x27D4EB2D
POS_C2 = 0x165667B1
```
**EN:** This section prepares the module namespace. It imports `torch`, `triton`, and `triton.language`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `FMIX32_C1`, `FMIX32_C2`, `POS_C1`, and `POS_C2` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `torch`、`triton` 以及 `triton.language`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `FMIX32_C1`、`FMIX32_C2`、`POS_C1` 以及 `POS_C2` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 26-30: Internal helper `_rotl32`
```python
@triton.jit
def _rotl32(x, r: tl.constexpr):
    return (x << r) | (x >> (32 - r))
```
**EN:** This block defines `_rotl32` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_rotl32`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 31-42: Internal helper `_fmix32`
```python
@triton.jit
def _fmix32(x, C1: tl.constexpr, C2: tl.constexpr):
    c1 = tl.full((), C1, tl.uint32)
    c2 = tl.full((), C2, tl.uint32)
    x ^= x >> 16
    x = x * c1
    x ^= x >> 13
    x = x * c2
    x ^= x >> 16
    return x
```
**EN:** This block defines `_fmix32` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.full`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `c1`, `c2`, and `x` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_fmix32`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.full`，说明该流程会编排底层辅助函数或计算内核。 像 `c1`、`c2` 以及 `x` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 43-106: Function `hash_tiles32_kernel_blocked` and its core logic
```python
@triton.jit
def hash_tiles32_kernel_blocked(
    in_ptr,
    out_ptr,
    n_u32,
    seed1,
    seed2,
    FM_C1: tl.constexpr,
    FM_C2: tl.constexpr,
    POS_A: tl.constexpr,
    POS_B: tl.constexpr,
    TILE: tl.constexpr,
    BLOCK: tl.constexpr,
    USE_CG: tl.constexpr,
):
    pid = tl.program_id(axis=0)
    base = pid * TILE

    s1 = tl.full((), seed1, tl.uint32)
    s2 = tl.full((), seed2, tl.uint32)
    posA = tl.full((), POS_A, tl.uint32)
    posB = tl.full((), POS_B, tl.uint32)

    h1 = tl.zeros((), dtype=tl.uint32)
    h2 = tl.zeros((), dtype=tl.uint32)

    for off in tl.static_range(0, TILE, BLOCK):
        idx = base + off + tl.arange(0, BLOCK)
        m = idx < n_u32

        if USE_CG:
            v = tl.load(in_ptr + idx, mask=m, other=0, cache_modifier=".cg")
        else:
            v = tl.load(in_ptr + idx, mask=m, other=0)
        v = v.to(tl.uint32)

        iu = idx.to(tl.uint32)
        p1 = (iu * posA + s1) ^ _rotl32(iu, 15)
        p2 = (iu * posB + s2) ^ _rotl32(iu, 13)

        k1 = _fmix32(v ^ p1, C1=FM_C1, C2=FM_C2)
        k2 = _fmix32(v ^ p2, C1=FM_C1, C2=FM_C2)

        zero32 = tl.zeros_like(k1)
        k1 = tl.where(m, k1, zero32)
        k2 = tl.where(m, k2, zero32)

        h1 += tl.sum(k1, axis=0).to(tl.uint32)
        h2 += tl.sum(k2, axis=0).to(tl.uint32)

    nbytes = tl.full((), n_u32 * 4, tl.uint32)
    h1 ^= nbytes
    h2 ^= nbytes
    h1 = _fmix32(h1, C1=FM_C1, C2=FM_C2)
    h2 = (
        _fmix32(h2, C1=FMIX32_C1, C2=FMIX32_C2)
        if False
        else _fmix32(h2, C1=FM_C1, C2=FM_C2)
    )

    out = (h1.to(tl.uint64) << 32) | h2.to(tl.uint64)
    tl.store(out_ptr + pid, out)
```
**EN:** This block defines `hash_tiles32_kernel_blocked` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.full`, `tl.zeros`, `tl.static_range`, and `_fmix32`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid`, `base`, `s1`, `s2`, and `posA` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `hash_tiles32_kernel_blocked`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.full`、`tl.zeros`、`tl.static_range` 以及 `_fmix32`，说明该流程会编排底层辅助函数或计算内核。 像 `pid`、`base`、`s1`、`s2` 以及 `posA` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 107-119: Function `add_tree_reduce_u64_kernel` and its core logic
```python
@triton.jit
def add_tree_reduce_u64_kernel(in_ptr, out_ptr, n_elems, CHUNK: tl.constexpr):
    pid = tl.program_id(axis=0)
    start = pid * CHUNK
    h = tl.zeros((), dtype=tl.uint64)
    for i in tl.static_range(0, CHUNK):
        idx = start + i
        m = idx < n_elems
        v = tl.load(in_ptr + idx, mask=m, other=0).to(tl.uint64)
        h += v
    tl.store(out_ptr + pid, h)
```
**EN:** This block defines `add_tree_reduce_u64_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.zeros`, `tl.static_range`, `tl.store`, and `tl.load.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid`, `start`, `h`, `idx`, and `m` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `add_tree_reduce_u64_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.zeros`、`tl.static_range`、`tl.store` 以及 `tl.load.to`，说明该流程会编排底层辅助函数或计算内核。 像 `pid`、`start`、`h`、`idx` 以及 `m` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 120-132: Internal helper `_as_uint32_words`
```python
def _as_uint32_words(t: torch.Tensor) -> torch.Tensor:
    assert t.is_cuda, "Use .cuda() first"
    tb = t.contiguous().view(torch.uint8)
    nbytes = tb.numel()
    pad = (4 - (nbytes & 3)) & 3
    if pad:
        tb_p = torch.empty(nbytes + pad, dtype=torch.uint8, device=tb.device)
        tb_p[:nbytes].copy_(tb)
        tb_p[nbytes:].zero_()
        tb = tb_p
    return tb.view(torch.uint32)
```
**EN:** This block defines `_as_uint32_words` and contains the main logic for this step. It mainly invokes `t.contiguous.view`, `tb.numel`, `tb.view`, `torch.empty`, and `tb_p.copy_`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `tb`, `nbytes`, `pad`, and `tb_p` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_as_uint32_words`，并承载这一阶段的核心逻辑。 它主要调用 `t.contiguous.view`、`tb.numel`、`tb.view`、`torch.empty` 以及 `tb_p.copy_`，说明该流程会编排底层辅助函数或计算内核。 像 `tb`、`nbytes`、`pad` 以及 `tb_p` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 133-143: Internal helper `_final_splitmix64`
```python
def _final_splitmix64(x: int) -> int:
    mask = (1 << 64) - 1
    x &= mask
    x ^= x >> 30
    x = (x * 0xBF58476D1CE4E5B9) & mask
    x ^= x >> 27
    x = (x * 0x94D049BB133111EB) & mask
    x ^= x >> 31
    return x
```
**EN:** This block defines `_final_splitmix64` and contains the main logic for this step. Intermediate names such as `mask` and `x` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_final_splitmix64`，并承载这一阶段的核心逻辑。 像 `mask` 和 `x` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 144-189: Function `gpu_tensor_hash` and its core logic
```python
@torch.inference_mode()
def gpu_tensor_hash(
    tensor: torch.Tensor,
    *,
    seed: int = 0x243F6A88,
    tile_words: int = 8192,
    block_words: int = 256,
    reduce_chunk: int = 1024,
    num_warps: int = 4,
    num_stages: int = 4,
    use_cg: bool = True,
) -> int:
    assert tensor.is_cuda, "Use .cuda() first"
    u32 = _as_uint32_words(tensor)
    n = u32.numel()
    if n == 0:
        return 0

    grid1 = (triton.cdiv(n, tile_words),)
    partials = torch.empty(grid1[0], dtype=torch.uint64, device=u32.device)
    hash_tiles32_kernel_blocked[grid1](
        u32,
        partials,
        n,
        seed1=seed & 0xFFFFFFFF,
        seed2=((seed * 0x9E3779B1) ^ 0xDEADBEEF) & 0xFFFFFFFF,
        FM_C1=FMIX32_C1,
        FM_C2=FMIX32_C2,
        POS_A=POS_C1,
        POS_B=POS_C2,
        TILE=tile_words,
        BLOCK=block_words,
        USE_CG=use_cg,
        num_warps=num_warps,
        num_stages=num_stages,
    )

    cur = partials
    while cur.numel() > 1:
        n_elems = cur.numel()
        grid2 = (triton.cdiv(n_elems, reduce_chunk),)
        nxt = torch.empty(grid2[0], dtype=torch.uint64, device=cur.device)
        add_tree_reduce_u64_kernel[grid2](cur, nxt, n_elems, CHUNK=reduce_chunk)
        cur = nxt

    return _final_splitmix64(int(cur.item()))
```
**EN:** This block defines `gpu_tensor_hash` and contains the main logic for this step. Decorators like `torch.inference_mode` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `torch.inference_mode`, `_as_uint32_words`, `u32.numel`, `torch.empty`, and `hash_tiles32_kernel_blocked`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `u32`, `n`, `grid1`, `partials`, and `cur` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `gpu_tensor_hash`，并承载这一阶段的核心逻辑。 像 `torch.inference_mode` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `torch.inference_mode`、`_as_uint32_words`、`u32.numel`、`torch.empty` 以及 `hash_tiles32_kernel_blocked`，说明该流程会编排底层辅助函数或计算内核。 像 `u32`、`n`、`grid1`、`partials` 以及 `cur` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `_rotl32`, `_fmix32`, `hash_tiles32_kernel_blocked`, `add_tree_reduce_u64_kernel`, and `_as_uint32_words`. / **主要符号**：核心入口包括 `_rotl32`、`_fmix32`、`hash_tiles32_kernel_blocked`、`add_tree_reduce_u64_kernel` 以及 `_as_uint32_words`。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Sampling policy**: Covers post-processing of logits and token selection strategies. / **采样策略**：涵盖 logits 后处理与 token 选择策略。

## Dependencies / 依赖关系
- **Third-party**: `torch`, `triton`, and `triton.language` / **第三方依赖**：`torch`、`triton` 以及 `triton.language`
