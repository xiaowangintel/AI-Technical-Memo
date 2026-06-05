# 08-grouped-gemm.py — Code Analysis / 代码分析

## Source / 来源

- **EN:** Source file: `/root/xw/triton/python/tutorials/08-grouped-gemm.py`
- **CN:** 源文件：`/root/xw/triton/python/tutorials/08-grouped-gemm.py`
- **EN:** Analysis output: `/root/xw/triton/tutorial/python/tutorials/08-grouped-gemm.py.md`
- **CN:** 分析输出：`/root/xw/triton/tutorial/python/tutorials/08-grouped-gemm.py.md`

## Line-by-Line Analysis / 逐行分析

### Block 1 — Module header, imports, and runtime helpers / 模块头、导入与运行时辅助函数

```python
"""
Group GEMM
============================
This group gemm kernel launches a fixed number of CTA to compute a group
of gemms. The scheduling is static and we do it on device.
"""

# Copyright (c) 2023 - 2025 NVIDIA Corporation & Affiliates. All rights reserved.
#
# Permission is hereby granted, free of charge, to any person obtaining
# a copy of this software and associated documentation files
# (the "Software"), to deal in the Software without restriction,
# including without limitation the rights to use, copy, modify, merge,
# publish, distribute, sublicense, and/or sell copies of the Software,
# and to permit persons to whom the Software is furnished to do so,
# subject to the following conditions:
#
# The above copyright notice and this permission notice shall be
# included in all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
# EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
# MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
# IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
# CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
# TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
# SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

from typing import Optional
import torch

import triton
import triton.language as tl

DEVICE = triton.runtime.driver.active.get_active_torch_device()


def is_cuda():
    return triton.runtime.driver.active.get_current_target().backend == "cuda"


def supports_tma():
    return is_cuda() and torch.cuda.get_device_capability()[0] >= 9


def num_sms():
    if is_cuda():
        return torch.cuda.get_device_properties("cuda").multi_processor_count
    return 148
```

**EN:** The file starts by defining its goal: grouped GEMM with a fixed number of CTAs and device-side static scheduling. It imports PyTorch plus Triton, selects the active device, and provides three helpers: `is_cuda()` gates CUDA-only paths, `supports_tma()` enables the Tensor Memory Accelerator path only on Hopper-class GPUs or newer, and `num_sms()` determines how many SMs to use for virtual grid sizing.

**CN:** 文件开头先说明目标：用固定数量的 CTA 执行 grouped GEMM，并在设备端进行静态调度。随后导入 PyTorch 与 Triton，选择当前活动设备，并定义三个辅助函数：`is_cuda()` 用于判断是否为 CUDA 后端，`supports_tma()` 仅在 Hopper 及以上架构时启用 TMA 路径，`num_sms()` 用于获取 SM 数量，以便决定虚拟网格规模。

### Block 2 — Autotuned base kernel declaration / 自动调优的基础内核声明

```python
@triton.autotune(
    configs=[
        triton.Config({
            'BLOCK_SIZE_M': 128,
            'BLOCK_SIZE_N': 128,
            'BLOCK_SIZE_K': 32,
            'NUM_SM': 84,
        }),
        triton.Config({
            'BLOCK_SIZE_M': 128,
            'BLOCK_SIZE_N': 128,
            'BLOCK_SIZE_K': 32,
            'NUM_SM': 128,
        }),
        triton.Config({
            'BLOCK_SIZE_M': 64,
            'BLOCK_SIZE_N': 64,
            'BLOCK_SIZE_K': 32,
            'NUM_SM': 84,
        }),
        triton.Config({
            'BLOCK_SIZE_M': 64,
            'BLOCK_SIZE_N': 64,
            'BLOCK_SIZE_K': 32,
            'NUM_SM': 128,
        }),
        triton.Config({
            'BLOCK_SIZE_M': 128,
            'BLOCK_SIZE_N': 128,
            'BLOCK_SIZE_K': 64,
            'NUM_SM': num_sms(),
        }),
        triton.Config({
            'BLOCK_SIZE_M': 64,
            'BLOCK_SIZE_N': 128,
            'BLOCK_SIZE_K': 64,
            'NUM_SM': num_sms(),
        }),
    ],
    key=['group_size'],
)
@triton.jit
def grouped_matmul_kernel(
    # device tensor of matrices pointers
    group_a_ptrs,
    group_b_ptrs,
    group_c_ptrs,
    # device tensor of gemm sizes. its shape is [group_size, 3]
    # dim 0 is group_size, dim 1 is the values of <M, N, K> of each gemm
    group_gemm_sizes,
    # device tensor of leading dimension sizes. its shape is [group_size, 3]
    # dim 0 is group_size, dim 1 is the values of <lda, ldb, ldc> of each gemm
    g_lds,
    # number of gemms
    group_size,
    # number of virtual SM
    NUM_SM: tl.constexpr,
    # tile sizes
    BLOCK_SIZE_M: tl.constexpr,
    BLOCK_SIZE_N: tl.constexpr,
    BLOCK_SIZE_K: tl.constexpr,
):
```

**EN:** This decorator stack defines the baseline grouped GEMM kernel. The autotuner explores several tile shapes and virtual-SM counts, keyed by `group_size`, so Triton can pick a configuration appropriate for the grouped workload. The kernel interface is intentionally low-level: instead of tensor batches, it consumes device arrays of raw pointers, problem sizes, and leading dimensions.

**CN:** 这一组装饰器定义了基础版 grouped GEMM 内核。自动调优器会根据 `group_size` 在多种 tile 形状和虚拟 SM 数量之间选择合适配置。内核接口也非常底层：它不直接接收批量张量，而是接收位于设备端的原始指针数组、每个 GEMM 的尺寸以及 leading dimension 信息。

### Block 3 — Base kernel scheduling and tile computation / 基础内核的调度与 tile 计算

```python
    tile_idx = tl.program_id(0)
    last_problem_end = 0
    for g in range(group_size):
        # get the gemm size of the current problem
        gm = tl.load(group_gemm_sizes + g * 3)
        gn = tl.load(group_gemm_sizes + g * 3 + 1)
        gk = tl.load(group_gemm_sizes + g * 3 + 2)
        num_m_tiles = tl.cdiv(gm, BLOCK_SIZE_M)
        num_n_tiles = tl.cdiv(gn, BLOCK_SIZE_N)
        num_tiles = num_m_tiles * num_n_tiles
        # iterate through the tiles in the current gemm problem
        while (tile_idx >= last_problem_end and tile_idx < last_problem_end + num_tiles):
            # pick up a tile from the current gemm problem
            k = gk
            lda = tl.load(g_lds + g * 3)
            ldb = tl.load(g_lds + g * 3 + 1)
            ldc = tl.load(g_lds + g * 3 + 2)
            a_ptr = tl.load(group_a_ptrs + g).to(tl.pointer_type(tl.float16))
            b_ptr = tl.load(group_b_ptrs + g).to(tl.pointer_type(tl.float16))
            c_ptr = tl.load(group_c_ptrs + g).to(tl.pointer_type(tl.float16))
            # figure out tile coordinates
            tile_idx_in_gemm = tile_idx - last_problem_end
            tile_m_idx = tile_idx_in_gemm // num_n_tiles
            tile_n_idx = tile_idx_in_gemm % num_n_tiles

            # do regular gemm here
            offs_am = tile_m_idx * BLOCK_SIZE_M + tl.arange(0, BLOCK_SIZE_M)
            offs_bn = tile_n_idx * BLOCK_SIZE_N + tl.arange(0, BLOCK_SIZE_N)
            offs_k = tl.arange(0, BLOCK_SIZE_K)
            a_ptrs = a_ptr + offs_am[:, None] * lda + offs_k[None, :]
            b_ptrs = b_ptr + offs_k[:, None] * ldb + offs_bn[None, :]
            accumulator = tl.zeros((BLOCK_SIZE_M, BLOCK_SIZE_N), dtype=tl.float32)
            for kk in range(0, tl.cdiv(k, BLOCK_SIZE_K)):
                # hint to Triton compiler to do proper loop pipelining
                tl.multiple_of(a_ptrs, [16, 16])
                tl.multiple_of(b_ptrs, [16, 16])
                # assume full tile for now
                a = tl.load(a_ptrs)
                b = tl.load(b_ptrs)
                accumulator += tl.dot(a, b)
                a_ptrs += BLOCK_SIZE_K
                b_ptrs += BLOCK_SIZE_K * ldb
            c = accumulator.to(tl.float16)

            offs_cm = tile_m_idx * BLOCK_SIZE_M + tl.arange(0, BLOCK_SIZE_M)
            offs_cn = tile_n_idx * BLOCK_SIZE_N + tl.arange(0, BLOCK_SIZE_N)
            c_ptrs = c_ptr + ldc * offs_cm[:, None] + offs_cn[None, :]

            # assumes full tile for now
            tl.store(c_ptrs, c)

            # go to the next tile by advancing NUM_SM
            tile_idx += NUM_SM

        # get ready to go to the next gemm problem
        last_problem_end = last_problem_end + num_tiles
```

**EN:** This is the heart of the tutorial. Each Triton program starts from its own `tile_idx = tl.program_id(0)` and walks the global tile space across all GEMMs. `last_problem_end` turns the group into a flattened tile schedule: each problem occupies a contiguous tile interval, and a CTA keeps jumping by `NUM_SM`, which statically partitions work across the virtual SM set. Inside each tile, the kernel computes standard block GEMM using pointer arithmetic and an FP32 accumulator, then writes FP16 output. The tutorial explicitly assumes full tiles, so it focuses on scheduling rather than edge masking.

**CN:** 这是本教程的核心。每个 Triton program 从自己的 `tile_idx = tl.program_id(0)` 出发，在所有 GEMM 组成的全局 tile 空间中遍历。`last_problem_end` 将整个 group 展平成一维 tile 调度区间：每个问题占据一段连续 tile 区间，而 CTA 通过 `NUM_SM` 步长不断跳跃，从而以静态方式把工作划分到虚拟 SM 集合上。在单个 tile 内部，内核通过指针运算执行标准分块 GEMM，使用 FP32 累加器，再写回 FP16 结果。教程还明确假设所有 tile 都是完整的，因此重点放在调度逻辑，而不是边界 mask 处理。

### Block 4 — Python wrapper for the base kernel / 基础内核的 Python 包装函数

```python
def group_gemm_fn(group_A, group_B):
    assert len(group_A) == len(group_B)
    group_size = len(group_A)

    A_addrs = []
    B_addrs = []
    C_addrs = []
    g_sizes = []
    g_lds = []
    group_C = []
    for i in range(group_size):
        A = group_A[i]
        B = group_B[i]
        assert A.shape[1] == B.shape[0]
        M, K = A.shape
        K, N = B.shape
        C = torch.empty((M, N), device=DEVICE, dtype=A.dtype)
        group_C.append(C)
        A_addrs.append(A.data_ptr())
        B_addrs.append(B.data_ptr())
        C_addrs.append(C.data_ptr())
        g_sizes += [M, N, K]
        g_lds += [A.stride(0), B.stride(0), C.stride(0)]

    # note these are device tensors
    d_a_ptrs = torch.tensor(A_addrs, device=DEVICE)
    d_b_ptrs = torch.tensor(B_addrs, device=DEVICE)
    d_c_ptrs = torch.tensor(C_addrs, device=DEVICE)
    d_g_sizes = torch.tensor(g_sizes, dtype=torch.int32, device=DEVICE)
    d_g_lds = torch.tensor(g_lds, dtype=torch.int32, device=DEVICE)
    # we use a fixed number of CTA, and it's auto-tunable
    grid = lambda META: (META['NUM_SM'], )
    grouped_matmul_kernel[grid](
        d_a_ptrs,
        d_b_ptrs,
        d_c_ptrs,
        d_g_sizes,
        d_g_lds,
        group_size,
    )

    return group_C
```

**EN:** `group_gemm_fn` converts a Python list of matrices into the compact metadata format the kernel expects. It allocates one output tensor per GEMM, gathers data pointers, packs all `(M, N, K)` and leading-dimension triples into flat device tensors, and launches exactly `NUM_SM` programs via the autotuned grid lambda. This wrapper is what makes the low-level kernel usable from normal PyTorch code.

**CN:** `group_gemm_fn` 把 Python 中的矩阵列表转换成内核需要的紧凑元数据格式。它为每个 GEMM 分配输出张量，收集数据指针，把所有 `(M, N, K)` 与 leading dimension 三元组打平成设备张量，并通过自动调优的网格函数启动恰好 `NUM_SM` 个 program。这个包装层把底层内核变成了普通 PyTorch 代码可直接调用的接口。

### Block 5 — TMA autotuning space / TMA 自动调优配置空间

```python
tma_configs = [
    triton.Config({'BLOCK_SIZE_M': BM, 'BLOCK_SIZE_N': BN, 'BLOCK_SIZE_K' : BK}, num_stages=s, num_warps=w) \
    for BM in [128]\
    for BN in [128, 256]\
    for BK in [64, 128]\
    for s in ([3, 4])\
    for w in [4, 8]\
]
```

**EN:** The TMA path uses a richer search space than the baseline kernel. Besides block sizes, it also autotunes pipeline depth (`num_stages`) and warp count (`num_warps`), which matter more when descriptor-driven loads are used.

**CN:** TMA 路径的搜索空间比基础内核更丰富。除了 block 大小外，它还会自动调优流水线深度 `num_stages` 和 warp 数量 `num_warps`，因为在使用描述符驱动加载时，这些参数对性能影响更明显。

### Block 6 — TMA kernel declaration / TMA 内核声明

```python
@triton.autotune(
    tma_configs,
    key=['group_size'],
)
@triton.jit
def grouped_matmul_tma_kernel(
    # device tensor of matrices pointers
    group_a_ptrs,
    group_b_ptrs,
    group_c_ptrs,
    # device tensor of gemm sizes. its shape is [group_size, 3]
    # dim 0 is group_size, dim 1 is the values of <M, N, K> of each gemm
    group_gemm_sizes,
    # device tensor of leading dimension sizes. its shape is [group_size, 3]
    # dim 0 is group_size, dim 1 is the values of <lda, ldb, ldc> of each gemm
    g_lds,
    # number of gemms
    group_size,
    # number of virtual SM
    NUM_SM: tl.constexpr,
    # tile sizes
    BLOCK_SIZE_M: tl.constexpr,
    BLOCK_SIZE_N: tl.constexpr,
    BLOCK_SIZE_K: tl.constexpr,
    # is the output FP8 or FP16
    FP8: tl.constexpr,
):
```

**EN:** This second kernel keeps the same grouped scheduling interface but adds an `FP8` compile-time flag for descriptor-based I/O. Structurally, the tutorial presents TMA as an alternative implementation of the same grouped GEMM contract rather than as a different API.

**CN:** 第二个内核保留了相同的 grouped 调度接口，但额外增加了一个编译期常量 `FP8`，用于描述符驱动的输入输出类型控制。从结构上看，教程把 TMA 版本展示为“同一个 grouped GEMM 接口的另一种实现”，而不是完全不同的 API。

### Block 7 — TMA kernel body / TMA 内核主体

```python
    dtype = tl.float8e4nv if FP8 else tl.float16
    tile_idx = tl.program_id(0)
    last_problem_end = 0
    for g in range(group_size):
        # get the gemm size of the current problem
        gm = tl.load(group_gemm_sizes + g * 3)
        gn = tl.load(group_gemm_sizes + g * 3 + 1)
        gk = tl.load(group_gemm_sizes + g * 3 + 2)
        num_m_tiles = tl.cdiv(gm, BLOCK_SIZE_M)
        num_n_tiles = tl.cdiv(gn, BLOCK_SIZE_N)
        num_tiles = num_m_tiles * num_n_tiles
        if tile_idx >= last_problem_end and tile_idx < last_problem_end + num_tiles:
            # pick up a tile from the current gemm problem
            lda = tl.load(g_lds + g * 3)
            ldb = tl.load(g_lds + g * 3 + 1)
            ldc = tl.load(g_lds + g * 3 + 2)

            a_ptr = tl.load(group_a_ptrs + g).to(tl.pointer_type(dtype))
            b_ptr = tl.load(group_b_ptrs + g).to(tl.pointer_type(dtype))
            c_ptr = tl.load(group_c_ptrs + g).to(tl.pointer_type(dtype))

            a_desc = tl.make_tensor_descriptor(
                a_ptr,
                shape=[gm, gk],
                strides=[lda, 1],
                block_shape=[BLOCK_SIZE_M, BLOCK_SIZE_K],
            )

            b_desc = tl.make_tensor_descriptor(
                b_ptr,
                shape=[gn, gk],
                strides=[ldb, 1],
                block_shape=[BLOCK_SIZE_N, BLOCK_SIZE_K],
            )
            c_desc = tl.make_tensor_descriptor(
                c_ptr,
                shape=[gm, gn],
                strides=[ldc, 1],
                block_shape=[BLOCK_SIZE_M, BLOCK_SIZE_N],
            )

            # iterate through the tiles in the current gemm problem
            while (tile_idx >= last_problem_end and tile_idx < last_problem_end + num_tiles):
                k = gk
                # figure out tile coordinates
                tile_idx_in_gemm = tile_idx - last_problem_end
                tile_m_idx = tile_idx_in_gemm // num_n_tiles
                tile_n_idx = tile_idx_in_gemm % num_n_tiles

                # do regular gemm here
                offs_am = tile_m_idx * BLOCK_SIZE_M
                offs_bn = tile_n_idx * BLOCK_SIZE_N

                accumulator = tl.zeros((BLOCK_SIZE_M, BLOCK_SIZE_N), dtype=tl.float32)
                for kk in range(0, tl.cdiv(k, BLOCK_SIZE_K)):
                    a = a_desc.load([offs_am, kk * BLOCK_SIZE_K])
                    b = b_desc.load([offs_bn, kk * BLOCK_SIZE_K])
                    accumulator += tl.dot(a, b.T)

                offs_cm = tile_m_idx * BLOCK_SIZE_M
                offs_cn = tile_n_idx * BLOCK_SIZE_N

                c = accumulator.to(dtype)
                c_desc.store([offs_cm, offs_cn], c)

                # go to the next tile by advancing NUM_SM
                tile_idx += NUM_SM

        # get ready to go to the next gemm problem
        last_problem_end = last_problem_end + num_tiles
```

**EN:** The TMA variant preserves the same flattened scheduling idea but replaces manual pointer-vector construction with tensor descriptors. `tl.make_tensor_descriptor` packages shape, stride, and tile geometry so loads and stores become structured operations. One key detail is that `b_desc` uses shape `[gn, gk]`, and the dot uses `b.T`, meaning the wrapper feeds a transposed-and-contiguous `B` layout to better match descriptor access patterns.

**CN:** TMA 版本保留了相同的扁平化调度思想，但把手工构造指针向量替换为张量描述符。`tl.make_tensor_descriptor` 将形状、步长和 tile 几何信息打包起来，使加载和存储变成结构化操作。一个关键细节是 `b_desc` 的形状是 `[gn, gk]`，并且计算时使用 `b.T`，这意味着包装函数会传入转置且连续化后的 `B`，以更好匹配描述符访问模式。

### Block 8 — TMA wrapper / TMA 包装函数

```python
def group_gemm_tma_fn(group_A, group_B):

    assert supports_tma()

    assert len(group_A) == len(group_B)
    group_size = len(group_A)

    A_addrs = []
    B_addrs = []
    C_addrs = []
    g_sizes = []
    g_lds = []
    group_C = []
    for i in range(group_size):
        A = group_A[i]
        B = group_B[i]
        assert A.shape[1] == B.shape[1]
        M, K = A.shape
        N, K = B.shape
        C = torch.empty((M, N), device=DEVICE, dtype=A.dtype)
        group_C.append(C)
        A_addrs.append(A.data_ptr())
        B_addrs.append(B.data_ptr())
        C_addrs.append(C.data_ptr())
        g_sizes += [M, N, K]
        g_lds += [A.stride(0), B.stride(0), C.stride(0)]
    # note these are device tensors
    d_a_ptrs = torch.tensor(A_addrs, device=DEVICE)
    d_b_ptrs = torch.tensor(B_addrs, device=DEVICE)
    d_c_ptrs = torch.tensor(C_addrs, device=DEVICE)
    d_g_sizes = torch.tensor(g_sizes, dtype=torch.int32, device=DEVICE)
    d_g_lds = torch.tensor(g_lds, dtype=torch.int32, device=DEVICE)

    # we use a fixed number of CTA, and it's auto-tunable

    # TMA descriptors require a global memory allocation
    def alloc_fn(size: int, alignment: int, stream: Optional[int]):
        return torch.empty(size, device="cuda", dtype=torch.int8)

    triton.set_allocator(alloc_fn)

    grid = lambda META: (META['NUM_SM'], )
    grouped_matmul_tma_kernel[grid](d_a_ptrs, d_b_ptrs, d_c_ptrs, d_g_sizes, d_g_lds, group_size,
                                    FP8=torch.float8_e4m3fn == group_A[0].dtype, NUM_SM=num_sms())
    return group_C
```

**EN:** The TMA wrapper mirrors the baseline wrapper but imposes a different shape contract: `group_B` is expected as `(N, K)` because the kernel reads descriptor tiles and transposes them in the dot product. It also installs a custom allocator with `triton.set_allocator`, because TMA descriptors require global-memory-backed allocations.

**CN:** TMA 包装函数与基础包装函数结构相似，但对输入形状有不同约束：`group_B` 需要是 `(N, K)`，因为内核以描述符方式读取 tile，并在点积时做转置。它还通过 `triton.set_allocator` 安装了自定义分配器，因为 TMA 描述符要求使用位于全局内存中的分配。

### Block 9 — Correctness test setup and validation / 正确性测试构造与校验

```python
group_m = [1024, 512, 256, 128]
group_n = [1024, 512, 256, 128]
group_k = [1024, 512, 256, 128]
group_A = []
group_B = []
group_B_T = []
assert len(group_m) == len(group_n)
assert len(group_n) == len(group_k)
group_size = len(group_m)
for i in range(group_size):
    M = group_m[i]
    N = group_n[i]
    K = group_k[i]
    A = torch.rand((M, K), device=DEVICE, dtype=torch.float16)
    B = torch.rand((K, N), device=DEVICE, dtype=torch.float16)
    B_T = B.T.contiguous()
    group_A.append(A)
    group_B.append(B)
    group_B_T.append(B_T)

tri_out = group_gemm_fn(group_A, group_B)
ref_out = [torch.matmul(a, b) for a, b in zip(group_A, group_B)]
for i in range(group_size):
    assert torch.allclose(ref_out[i], tri_out[i], atol=1e-2, rtol=1e-2)

if supports_tma():
    tri_tma_out = group_gemm_tma_fn(group_A, group_B_T)
    for i in range(group_size):
        assert torch.allclose(ref_out[i], tri_tma_out[i], atol=1e-2, rtol=1e-2)
```

**EN:** This section acts as the tutorial’s built-in test. It creates four GEMMs of descending sizes, runs the Triton grouped kernel, compares against `torch.matmul`, and then conditionally repeats the check for the TMA path using pre-transposed `B_T`. The tolerance is loose enough for FP16 arithmetic but strict enough to catch obvious indexing or layout bugs.

**CN:** 这一段就是教程内置的正确性测试。它构造四个尺寸逐渐减小的 GEMM，运行 Triton grouped 内核，并与 `torch.matmul` 的结果比较；如果支持 TMA，则再用预先转置好的 `B_T` 重复校验。这里的容差对 FP16 计算来说足够宽松，但依然能发现明显的索引或布局错误。

### Block 10 — Benchmark launch helpers / 性能基准的启动辅助函数

```python
# only launch the kernel, no tensor preparation here to remove all overhead
def triton_perf_fn(a_ptrs, b_ptrs, c_ptrs, sizes, lds, group_size):
    grid = lambda META: (META['NUM_SM'], )
    grouped_matmul_kernel[grid](
        a_ptrs,
        b_ptrs,
        c_ptrs,
        sizes,
        lds,
        group_size,
    )


def triton_tma_perf_fn(a_ptrs, b_ptrs, c_ptrs, sizes, lds, group_size, dtype):
    grid = lambda META: (META['NUM_SM'], )
    grouped_matmul_tma_kernel[grid](a_ptrs, b_ptrs, c_ptrs, sizes, lds, group_size, FP8=torch.float8_e4m3fn == dtype,
                                    NUM_SM=num_sms())


def torch_perf_fn(group_A, group_B):
    for a, b in zip(group_A, group_B):
        torch.matmul(a, b)
```

**EN:** These helpers isolate kernel execution from setup overhead. The Triton helpers launch already-prepared pointer metadata, while `torch_perf_fn` measures a simple loop over PyTorch matmuls. This keeps the benchmark focused on execution cost rather than Python-side packing work.

**CN:** 这些辅助函数将真正的内核执行与准备阶段开销分离开来。Triton 辅助函数直接启动已经准备好的指针与元数据，而 `torch_perf_fn` 则测量一组普通的 PyTorch `matmul`。这样基准测试更聚焦于执行成本，而不是 Python 端的数据打包开销。

### Block 11 — Square-matrix benchmark / 方阵基准测试

```python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        # argument names to use as an x-axis for the plot
        x_names=['N'],
        x_vals=[2**i for i in range(7, 11)],  # different possible values for `x_name`
        line_arg='provider',
        # argument name whose value corresponds to a different line in the plot
        # possible values for `line_arg``
        line_vals=['cublas', 'triton'] + (['triton-tma'] if supports_tma() else []),
        # label name for the lines
        line_names=["cuBLAS", "Triton"] + (['Triton + TMA'] if supports_tma() else []),
        # line styles
        styles=[('green', '-'), ('blue', '-')] + ([('red', '-')] if supports_tma() else []),
        ylabel="runtime(ms)",  # label name for the y-axis
        plot_name="group-gemm-performance",
        # name for the plot. Used also as a file name for saving the plot.
        args={},
    ))
def benchmark_square_matrices(N, provider):
    group_size = 4
    group_A = []
    group_B = []
    group_B_T = []
    A_addrs = []
    B_addrs = []
    B_T_addrs = []
    C_addrs = []
    g_sizes = []
    g_lds = []
    group_C = []
    for i in range(group_size):
        A = torch.rand((N, N), device=DEVICE, dtype=torch.float16)
        B = torch.rand((N, N), device=DEVICE, dtype=torch.float16)
        C = torch.empty((N, N), device=DEVICE, dtype=torch.float16)
        B_T = B.T.contiguous()
        group_A.append(A)
        group_B.append(B)
        group_B_T.append(B_T)
        group_C.append(C)
        A_addrs.append(A.data_ptr())
        B_addrs.append(B.data_ptr())
        B_T_addrs.append(B_T.data_ptr())
        C_addrs.append(C.data_ptr())
        g_sizes += [N, N, N]
        g_lds += [N, N, N]

    d_a_ptrs = torch.tensor(A_addrs, device=DEVICE)
    d_b_ptrs = torch.tensor(B_addrs, device=DEVICE)
    d_b_t_ptrs = torch.tensor(B_T_addrs, device=DEVICE)
    d_c_ptrs = torch.tensor(C_addrs, device=DEVICE)
    d_g_sizes = torch.tensor(g_sizes, dtype=torch.int32, device=DEVICE)
    d_g_lds = torch.tensor(g_lds, dtype=torch.int32, device=DEVICE)

    quantiles = [0.5, 0.2, 0.8]
    if provider == 'cublas':
        ms, min_ms, max_ms = triton.testing.do_bench(lambda: torch_perf_fn(group_A, group_B), quantiles=quantiles)
    if provider == 'triton':
        ms, min_ms, max_ms = triton.testing.do_bench(
            lambda: triton_perf_fn(d_a_ptrs, d_b_ptrs, d_c_ptrs, d_g_sizes, d_g_lds, group_size), quantiles=quantiles)
    if provider == 'triton-tma':
        ms, min_ms, max_ms = triton.testing.do_bench(
            lambda: triton_tma_perf_fn(d_a_ptrs, d_b_t_ptrs, d_c_ptrs, d_g_sizes, d_g_lds, group_size, dtype=torch.
                                       float16), quantiles=quantiles)
    return ms, min_ms, max_ms
```

**EN:** The first benchmark sweeps square matrix size `N` while keeping the group size fixed at 4. It prepares both normal `B` and transposed `B_T`, then measures cuBLAS-via-PyTorch, the baseline Triton grouped kernel, and the TMA variant when supported. Using `perf_report` means Triton can both print timing tables and emit plots.

**CN:** 第一个基准测试扫描方阵尺寸 `N`，同时将 group 大小固定为 4。它同时准备普通布局的 `B` 和转置连续化后的 `B_T`，然后分别测量经由 PyTorch 调用的 cuBLAS、基础 Triton grouped 内核，以及在支持时启用的 TMA 版本。借助 `perf_report`，Triton 可以同时输出时间表格和性能图。

### Block 12 — Batch-shape benchmark and execution / 批次形状基准与最终执行

```python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        # argument names to use as an x-axis for the plot
        x_names=['M'],
        x_vals=[2**i for i in range(7, 11)],  # different possible values for `x_name`
        line_arg='provider',
        # argument name whose value corresponds to a different line in the plot
        # possible values for `line_arg``
        line_vals=['cublas', 'triton'] + (['triton-tma'] if supports_tma() else []),
        # label name for the lines
        line_names=["cuBLAS", "Triton"] + (['Triton + TMA'] if supports_tma() else []),
        # line styles
        styles=[('green', '-'), ('blue', '-')] + ([('red', '-')] if supports_tma() else []),
        ylabel="runtime(ms)",  # label name for the y-axis
        plot_name="group-gemm-performance-m-8192-k-8192",
        # name for the plot. Used also as a file name for saving the plot.
        args={},
    ))
def benchmark_batches(M, provider):
    N = 8192
    K = 8192
    group_size = 4
    group_A = []
    group_B = []
    group_B_T = []
    A_addrs = []
    B_addrs = []
    B_T_addrs = []
    C_addrs = []
    g_sizes = []
    g_lds = []
    g_T_lds = []
    group_C = []
    for i in range(group_size):
        A = torch.rand((M, K), device=DEVICE, dtype=torch.float16)
        B = torch.rand((K, N), device=DEVICE, dtype=torch.float16)
        C = torch.empty((M, N), device=DEVICE, dtype=torch.float16)
        B_T = B.T.contiguous()
        group_A.append(A)
        group_B.append(B)
        group_B_T.append(B_T)
        group_C.append(C)
        A_addrs.append(A.data_ptr())
        B_addrs.append(B.data_ptr())
        B_T_addrs.append(B_T.data_ptr())
        C_addrs.append(C.data_ptr())
        g_sizes += [M, N, K]
        g_lds += [A.stride(0), B.stride(0), C.stride(0)]
        g_T_lds += [A.stride(0), B_T.stride(0), C.stride(0)]

    d_a_ptrs = torch.tensor(A_addrs, device=DEVICE)
    d_b_ptrs = torch.tensor(B_addrs, device=DEVICE)
    d_b_t_ptrs = torch.tensor(B_T_addrs, device=DEVICE)
    d_c_ptrs = torch.tensor(C_addrs, device=DEVICE)
    d_g_sizes = torch.tensor(g_sizes, dtype=torch.int32, device=DEVICE)
    d_g_lds = torch.tensor(g_lds, dtype=torch.int32, device=DEVICE)
    d_g_t_lds = torch.tensor(g_T_lds, dtype=torch.int32, device=DEVICE)

    quantiles = [0.5, 0.2, 0.8]
    if provider == 'cublas':
        ms, min_ms, max_ms = triton.testing.do_bench(lambda: torch_perf_fn(group_A, group_B), quantiles=quantiles)
    if provider == 'triton':
        ms, min_ms, max_ms = triton.testing.do_bench(
            lambda: triton_perf_fn(d_a_ptrs, d_b_ptrs, d_c_ptrs, d_g_sizes, d_g_lds, group_size), quantiles=quantiles)
    if provider == 'triton-tma':
        ms, min_ms, max_ms = triton.testing.do_bench(
            lambda: triton_tma_perf_fn(d_a_ptrs, d_b_t_ptrs, d_c_ptrs, d_g_sizes, d_g_t_lds, group_size, dtype=torch.
                                       float16), quantiles=quantiles)
    return ms, min_ms, max_ms


benchmark_square_matrices.run(show_plots=True, print_data=True)
benchmark_batches.run(show_plots=True, print_data=True)
```

**EN:** The second benchmark fixes `N = K = 8192` and sweeps `M`, highlighting how grouped GEMM behaves when only one dimension changes. Unlike the square benchmark, it keeps separate leading-dimension metadata for normal `B` and transposed `B_T`, because the TMA path consumes the transposed layout directly. The final two lines execute both benchmark suites immediately when the script runs.

**CN:** 第二个基准测试固定 `N = K = 8192`，只扫描 `M`，从而观察 grouped GEMM 在单一维度变化时的表现。与方阵基准不同，这里为普通布局的 `B` 和转置后的 `B_T` 分别维护 leading dimension 元数据，因为 TMA 路径会直接消费转置布局。最后两行会在脚本运行时立即执行两个基准测试套件。

## Key Concepts / 关键概念

- **EN:** **Static grouped scheduling** — all GEMM tiles from the whole group are flattened into one logical tile space, and each program advances by `NUM_SM`.
  
  **CN:** **静态 grouped 调度** —— 整个 group 的所有 GEMM tile 被展平成一个逻辑 tile 空间，每个 program 以 `NUM_SM` 为步长前进。

- **EN:** **Fixed CTA count** — the launch grid is not proportional to total tile count; it is capped to a virtual-SM count and relies on in-kernel looping.
  
  **CN:** **固定 CTA 数量** —— 启动网格并不随 tile 总数线性增长，而是限制为虚拟 SM 数量，并依赖内核内部循环完成剩余工作。

- **EN:** **Pointer-array interface** — grouped inputs are passed as device tensors of raw addresses plus compact size/stride metadata.
  
  **CN:** **指针数组接口** —— grouped 输入通过设备端原始地址数组以及紧凑的尺寸/步长元数据传入。

- **EN:** **Baseline vs. TMA path** — the baseline kernel uses explicit pointer arithmetic, while the TMA kernel uses tensor descriptors and a transposed `B` layout.
  
  **CN:** **基础版与 TMA 路径对比** —— 基础内核使用显式指针运算，而 TMA 内核使用张量描述符以及转置后的 `B` 布局。

- **EN:** **Autotuning** — different tile sizes, warp counts, stage counts, and virtual-SM values are explored automatically.
  
  **CN:** **自动调优** —— 系统会自动探索不同的 tile 大小、warp 数、stage 数以及虚拟 SM 数量。

- **EN:** **Validation and benchmarking** — the tutorial includes both correctness checks against `torch.matmul` and performance comparisons against cuBLAS/PyTorch.
  
  **CN:** **验证与基准测试** —— 教程同时包含与 `torch.matmul` 的正确性对比，以及与 cuBLAS/PyTorch 的性能对比。

## Dependencies / 依赖关系

- **EN:** `is_cuda()`, `supports_tma()`, and `num_sms()` feed environment decisions into autotuning, TMA gating, and launch sizing.
  
  **CN:** `is_cuda()`、`supports_tma()` 和 `num_sms()` 为自动调优、TMA 启用判断以及启动规模提供环境信息。

- **EN:** `group_gemm_fn()` prepares metadata and launches `grouped_matmul_kernel()`.
  
  **CN:** `group_gemm_fn()` 负责准备元数据并启动 `grouped_matmul_kernel()`。

- **EN:** `group_gemm_tma_fn()` prepares descriptor-friendly metadata, sets the allocator, and launches `grouped_matmul_tma_kernel()`.
  
  **CN:** `group_gemm_tma_fn()` 负责准备适合描述符的元数据、设置分配器，并启动 `grouped_matmul_tma_kernel()`。

- **EN:** The correctness test depends on `group_gemm_fn()`, optionally `group_gemm_tma_fn()`, and PyTorch `torch.matmul` as the reference implementation.
  
  **CN:** 正确性测试依赖 `group_gemm_fn()`、可选的 `group_gemm_tma_fn()`，以及作为参考实现的 PyTorch `torch.matmul`。

- **EN:** `triton_perf_fn()` and `triton_tma_perf_fn()` are lightweight launch helpers used by both benchmark functions.
  
  **CN:** `triton_perf_fn()` 和 `triton_tma_perf_fn()` 是两个轻量级启动辅助函数，供两个基准函数共同使用。

- **EN:** `benchmark_square_matrices()` and `benchmark_batches()` depend on Triton’s `perf_report`/`do_bench` framework and compare providers: cuBLAS via PyTorch, Triton baseline, and optional Triton TMA.
  
  **CN:** `benchmark_square_matrices()` 和 `benchmark_batches()` 依赖 Triton 的 `perf_report`/`do_bench` 框架，并比较三类提供者：经由 PyTorch 调用的 cuBLAS、Triton 基础版以及可选的 Triton TMA。
