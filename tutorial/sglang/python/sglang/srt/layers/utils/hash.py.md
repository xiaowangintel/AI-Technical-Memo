# hash.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/utils/hash.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements runtime layer support for the SGLang SRT runtime. It exposes symbols such as `rotl32`, `fmix32`, `murmur3_mix`, and `murmur_hash32_kernel` and connects them to backend-specific paths such as `Triton` and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了运行时层支持。它提供了 `rotl32`、`fmix32`、`murmur3_mix` 以及 `murmur_hash32_kernel` 等符号，并把这些符号连接到 `Triton` 和 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: Module imports and dependency wiring
```python
import torch
import triton
import triton.language as tl
```
**EN:** This section prepares the module namespace. It imports `torch`, `triton`, and `triton.language`, so later blocks can reuse runtime, tensor, or backend helpers.
**CN:** 该部分负责准备模块命名空间。 它导入了 `torch`、`triton` 以及 `triton.language`，让后续代码可以复用运行时、张量或后端辅助逻辑。

### Lines 6-15: Function `rotl32` and its core logic
```python
@triton.jit
def rotl32(x, r: tl.constexpr) -> tl.uint32:
    """
    rotate left 32-bit integer x by r bits
    e.g. x = 01110001, r = 2 -> 11000101
    """
    x = x.to(tl.uint64)
    return ((x << r) | (x >> (32 - r))) & 0xFFFFFFFF
```
**EN:** This block defines `rotl32` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `x.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `x` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `rotl32`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `x.to`，说明该流程会编排底层辅助函数或计算内核。 像 `x` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 16-28: Function `fmix32` and its core logic
```python
@triton.jit
def fmix32(h: tl.uint32) -> tl.uint32:
    """
    final mix of 32-bit hash value for MurmurHash
    """
    h ^= h >> 16
    h = (h * 0x85EBCA6B) & 0xFFFFFFFF
    h ^= h >> 13
    h = (h * 0xC2B2AE35) & 0xFFFFFFFF
    h ^= h >> 16
    return h
```
**EN:** This block defines `fmix32` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. Intermediate names such as `h` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fmix32`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 像 `h` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 29-49: Function `murmur3_mix` and its core logic
```python
@triton.jit
def murmur3_mix(h: tl.uint32, k: tl.uint32) -> tl.uint32:
    """
    Mixes a 32-bit key into the hash state.
    """
    c1: tl.uint32 = 0xCC9E2D51
    c2: tl.uint32 = 0x1B873593
    r1: tl.constexpr = 15
    r2: tl.constexpr = 13
    mm: tl.uint32 = 5
    nn: tl.uint32 = 0xE6546B64

    k = (k * c1) & 0xFFFFFFFF
    k = rotl32(k, r1)
    k = (k * c2) & 0xFFFFFFFF
    h ^= k
    h = rotl32(h, r2)
    h = (h * mm + nn) & 0xFFFFFFFF
    return h
```
**EN:** This block defines `murmur3_mix` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `rotl32`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `c1`, `c2`, `r1`, `r2`, and `mm` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `murmur3_mix`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `rotl32`，说明该流程会编排底层辅助函数或计算内核。 像 `c1`、`c2`、`r1`、`r2` 以及 `mm` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 50-103: Function `murmur_hash32_kernel` and its core logic
```python
@triton.jit
def murmur_hash32_kernel(
    seed_ptr,
    positions_ptr,
    col_indices_ptr,
    output_ptr,
    num_rows,
    num_cols,
    BLOCK_SIZE: tl.constexpr,
):
    """
    MurmurHash 32-bit implementation for Triton.
    Reference:
    - https://medium.com/@thealonemusk/murmurhash-the-scrappy-algorithm-that-secretly-powers-half-the-internet-2d3f79b4509b
    - https://en.wikipedia.org/wiki/MurmurHash

    We treat 64-bit seed, 32-bit position, and 32-bit col_index as 4 4-byte blocks, and bit-blend them together.
    """
    pid_row = tl.program_id(0)
    pid_col = tl.program_id(1)

    row_idx = pid_row
    col_offsets = pid_col * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
    mask = col_offsets < num_cols

    # Load inputs
    seed = tl.load(seed_ptr + row_idx).to(tl.uint64)
    pos = tl.load(positions_ptr + row_idx).to(tl.uint32)
    col = tl.load(col_indices_ptr + col_offsets, mask=mask, other=0).to(tl.uint32)

    h: tl.uint32 = 0  # hash accumulator

    # Process seed_low
    k: tl.uint32 = (seed & 0xFFFFFFFF).to(tl.uint32)
    h = murmur3_mix(h, k)

    # Process seed_high
    k = ((seed >> 32) & 0xFFFFFFFF).to(tl.uint32)
    h = murmur3_mix(h, k)

    # Process position block starting from seed32
    h = murmur3_mix(h, pos)

    # Process col block
    h = murmur3_mix(h, col)

    # Finalize (len=16 for seed + pos + col)
    h ^= 16
    h = fmix32(h)

    # Store result as uint32
    tl.store(output_ptr + row_idx * num_cols + col_offsets, h, mask=mask)
```
**EN:** This block defines `murmur_hash32_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.load.to`, `to`, `murmur3_mix`, and `fmix32`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid_row`, `pid_col`, `row_idx`, `col_offsets`, and `mask` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `murmur_hash32_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.load.to`、`to`、`murmur3_mix` 以及 `fmix32`，说明该流程会编排底层辅助函数或计算内核。 像 `pid_row`、`pid_col`、`row_idx`、`col_offsets` 以及 `mask` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 104-121: Function `murmur_hash32` and its core logic
```python
def murmur_hash32(seed, positions, col_indices):
    assert (
        seed.shape == positions.shape
    ), "Seed and positions must have the same shape (n,)"
    assert (
        len(seed.shape) == 1 and len(col_indices.shape) == 1
    ), f"Inputs must be 1D tensors {seed.shape=} {col_indices.shape=}"
    n = seed.shape[0]
    m = col_indices.shape[0]
    device = seed.device
    hashed = torch.empty((n, m), dtype=torch.uint32, device=device)

    BLOCK_SIZE = 1024
    grid = (n, triton.cdiv(m, BLOCK_SIZE))
    murmur_hash32_kernel[grid](
        seed, positions, col_indices, hashed, n, m, BLOCK_SIZE=BLOCK_SIZE
    )
    return hashed
```
**EN:** This block defines `murmur_hash32` and contains the main logic for this step. It mainly invokes `torch.empty`, `murmur_hash32_kernel`, `triton.cdiv`, and `len`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `n`, `m`, `device`, `hashed`, and `BLOCK_SIZE` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `murmur_hash32`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty`、`murmur_hash32_kernel`、`triton.cdiv` 以及 `len`，说明该流程会编排底层辅助函数或计算内核。 像 `n`、`m`、`device`、`hashed` 以及 `BLOCK_SIZE` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `rotl32`, `fmix32`, `murmur3_mix`, `murmur_hash32_kernel`, and `murmur_hash32`. / **主要符号**：核心入口包括 `rotl32`、`fmix32`、`murmur3_mix`、`murmur_hash32_kernel` 以及 `murmur_hash32`。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Acceleration stack**: The module depends on external acceleration libraries and wraps them behind Python entry points. / **加速栈**：该模块依赖外部加速库，并通过 Python 入口对它们进行封装。

## Dependencies / 依赖关系
- **Third-party**: `torch`, `triton`, and `triton.language` / **第三方依赖**：`torch`、`triton` 以及 `triton.language`
