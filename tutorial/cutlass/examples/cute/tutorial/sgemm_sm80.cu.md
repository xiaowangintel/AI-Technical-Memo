# sgemm_sm80.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/cute/tutorial/sgemm_sm80.cu`

**Purpose / 用途**: Demonstrates an SM80 (Ampere) HGEMM (float16) using CuTe with `cp.async` for asynchronous gmem→smem loads, explicit multi-stage (circular) shared-memory pipelining, swizzled smem layouts for LDSM, SM80 Tensor Core MMA atoms (`SM80_16x8x16_F16F16F16F16_TN`), and the `SM75_U32x4_LDSM_N` shared-memory load atom. It also includes NT and TN variants using `UniversalFMA` for comparison. / 演示使用 CuTe 的 SM80（Ampere）HGEMM（float16），具备：`cp.async` 异步全局内存→共享内存加载、显式多阶段（循环）共享内存流水线、用于 LDSM 的 swizzled 共享内存布局、SM80 Tensor Core MMA 原子（`SM80_16x8x16_F16F16F16F16_TN`）、以及 `SM75_U32x4_LDSM_N` 共享内存加载原子。同时包含使用 `UniversalFMA` 的 NT 和 TN 变体供对比。

---

## Line-by-Line Analysis / 逐行分析

### License Header (Lines 1–30)

```cpp
/***************************************************************************************************
 * Copyright (c) 2023 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 **************************************************************************************************/
```

**EN**: Standard NVIDIA BSD-3-Clause license. Same as the SM70 tutorial.  
**CN**: 标准 NVIDIA BSD-3 许可证，与 SM70 教程相同。

---

### Standard and Thrust Includes (Lines 31–42)

```cpp
#include <cstdlib>
#include <cstdio>
#include <cassert>
#include <thrust/host_vector.h>
#include <thrust/device_vector.h>
#include <cute/tensor.hpp>
#include "cutlass/util/print_error.hpp"
#include "cutlass/util/GPU_Clock.hpp"
#include "cutlass/util/helper_cuda.hpp"
```

**EN**: Same infrastructure as SM70. Note that `cute/tensor.hpp` now also pulls in SM80-specific copy atoms (`SM80_CP_ASYNC_*`) and MMA atoms (`SM80_16x8x16_*`) through its architecture dispatch mechanism.  
**CN**: 与 SM70 相同的基础设施。注意 `cute/tensor.hpp` 现在还通过其架构分发机制引入 SM80 特定的拷贝原子（`SM80_CP_ASYNC_*`）和 MMA 原子（`SM80_16x8x16_*`）。

---

### `SharedStorage` Struct (Lines 44–52)

```cpp
template <class ElementA, class ElementB,
          class SmemLayoutA, class SmemLayoutB>
struct SharedStorage {
  cute::ArrayEngine<ElementA, cute::cosize_v<SmemLayoutA>> A;
  cute::ArrayEngine<ElementB, cute::cosize_v<SmemLayoutB>> B;
};
```

**EN**: Unlike SM70 which used static `__shared__` arrays, SM80 uses a typed aggregate struct backed by `ArrayEngine`. This enables **dynamic shared memory** (allocated at kernel launch via the `smem_size` argument), which is necessary because SM80 requires large smem sizes (e.g., 3-stage pipeline × 128×64 × FP16 = 49152 bytes) that exceed the default static limit. `cosize_v<SmemLayoutA>` computes the exact number of elements at compile time, including pipeline stages. The struct layout places A and B buffers contiguously, with B immediately following A.  
**CN**: 与 SM70 使用静态 `__shared__` 数组不同，SM80 使用由 `ArrayEngine` 支持的类型化聚合结构体。这启用了**动态共享内存**（在 kernel 启动时通过 `smem_size` 参数分配），这是必要的，因为 SM80 需要大型共享内存（例如，3 阶段流水线 × 128×64 × FP16 = 49152 字节），超过了默认静态限制。`cosize_v<SmemLayoutA>` 在编译期计算精确的元素数量，包括流水线阶段。结构体布局使 A 和 B 缓冲区连续排列，B 紧跟 A 之后。

---

### Kernel Template — Additional S2R Atom Parameters (Lines 54–66)

```cpp
template <class ProblemShape, class CtaTiler,
          class TA, class AStride, class ASmemLayout, class TiledCopyA, class S2RAtomA,
          class TB, class BStride, class BSmemLayout, class TiledCopyB, class S2RAtomB,
          class TC, class CStride, class CSmemLayout, class TiledMma,
          class Alpha, class Beta>
__global__ static
__launch_bounds__(decltype(size(TiledMma{}))::value)
void gemm_device(ProblemShape shape_MNK, CtaTiler cta_tiler,
                 TA const* A, AStride dA, ASmemLayout sA_layout,
                   TiledCopyA copy_a, S2RAtomA s2r_atom_a,
                 TB const* B, BStride dB, BSmemLayout sB_layout,
                   TiledCopyB copy_b, S2RAtomB s2r_atom_b, ...)
```

**EN**: Compared to SM70, the SM80 kernel adds two new template parameters: `S2RAtomA` and `S2RAtomB` — the shared-memory-to-register copy atoms. These are separate from `TiledCopyA`/`TiledCopyB` (which handle gmem→smem) because the smem→rmem copy uses a different instruction (`LDSM` — the matrix-load-from-shared instruction optimized for Tensor Core layouts) rather than regular `LDS`. This decoupling allows independent selection of the global and shared memory access strategies.  
**CN**: 与 SM70 相比，SM80 kernel 新增两个模板参数：`S2RAtomA` 和 `S2RAtomB`——共享内存到寄存器的拷贝原子。这些与 `TiledCopyA`/`TiledCopyB`（处理全局内存→共享内存）分开，因为共享内存→寄存器拷贝使用不同的指令（`LDSM`——针对 Tensor Core 布局优化的矩阵从共享内存加载指令），而非普通的 `LDS`。这种解耦允许独立选择全局和共享内存访问策略。

---

### Static Precondition Checks (Lines 70–90)

```cpp
CUTE_STATIC_ASSERT_V(rank(shape_MNK) == Int<3>{});
CUTE_STATIC_ASSERT_V(size(copy_a) == size(mma));
static_assert(is_static<ASmemLayout>::value);
CUTE_STATIC_ASSERT_V(size<0>(ASmemLayout{}) == size<0>(cta_tiler));  // BLK_M
CUTE_STATIC_ASSERT_V(size<1>(ASmemLayout{}) == size<2>(cta_tiler));  // BLK_K
CUTE_STATIC_ASSERT_V(congruent(select<0,2>(shape_MNK), dA));
```

**EN**: Same category of compile-time guards as SM70. Verifying that all spatial dimensions, thread counts, and stride congruence conditions are met before the kernel body executes. The smem layout for SM80 has 3 modes `(BLK_M, BLK_K, PIPE)`, so `size<0>` and `size<1>` are still BLK_M and BLK_K respectively, with the pipeline dimension being the third.  
**CN**: 与 SM70 相同类别的编译期防护。在 kernel 主体执行前验证所有空间维度、线程数量和 stride 一致性条件是否满足。SM80 的共享内存布局有 3 个模式 `(BLK_M, BLK_K, PIPE)`，因此 `size<0>` 和 `size<1>` 仍分别是 BLK_M 和 BLK_K，流水线维度是第三个。

---

### Global Tensor Views (Lines 97–99)

```cpp
Tensor mA = make_tensor(make_gmem_ptr(A), select<0,2>(shape_MNK), dA); // (M,K)
Tensor mB = make_tensor(make_gmem_ptr(B), select<1,2>(shape_MNK), dB); // (N,K)
Tensor mC = make_tensor(make_gmem_ptr(C), select<0,1>(shape_MNK), dC); // (M,N)
```

**EN**: Identical to SM70. Full global tensors for A (M×K), B (N×K), C (M×N) are constructed as logical CuTe views over raw device pointers.  
**CN**: 与 SM70 相同。A (M×K)、B (N×K)、C (M×N) 的完整全局张量构造为原始设备指针上的逻辑 CuTe 视图。

---

### CTA Tiling (Lines 102–105)

```cpp
auto cta_coord = make_coord(blockIdx.x, blockIdx.y, _);
Tensor gA = local_tile(mA, cta_tiler, cta_coord, Step<_1, X,_1>{}); // (BLK_M,BLK_K,k)
Tensor gB = local_tile(mB, cta_tiler, cta_coord, Step< X,_1,_1>{}); // (BLK_N,BLK_K,k)
Tensor gC = local_tile(mC, cta_tiler, cta_coord, Step<_1,_1, X>{}); // (BLK_M,BLK_N)
```

**EN**: Same `local_tile` pattern as SM70. Each CTA handles one (BLK_M, BLK_N) output tile, iterating over K via the retained `k` dimension in `gA` and `gB`.  
**CN**: 与 SM70 相同的 `local_tile` 模式。每个 CTA 处理一个 (BLK_M, BLK_N) 输出 tile，通过 `gA` 和 `gB` 中保留的 `k` 维度迭代 K。

---

### Dynamic Shared Memory Setup (Lines 108–112)

```cpp
extern __shared__ char shared_memory[];
using SharedStorage = SharedStorage<TA, TB, ASmemLayout, BSmemLayout>;
SharedStorage& smem = *reinterpret_cast<SharedStorage*>(shared_memory);
Tensor sA = make_tensor(make_smem_ptr(smem.A.begin()), sA_layout); // (BLK_M,BLK_K,PIPE)
Tensor sB = make_tensor(make_smem_ptr(smem.B.begin()), sB_layout); // (BLK_N,BLK_K,PIPE)
```

**EN**: SM80 uses `extern __shared__ char shared_memory[]` — dynamic shared memory declared as a raw byte array. The `SharedStorage` struct is overlaid via `reinterpret_cast`. This lets the host compute the exact size needed (`sizeof(SharedStorage)`) and pass it to the kernel launch. The resulting `sA` and `sB` tensors now have a **third dimension for the pipeline stage** (PIPE = 3 in the TN HGEMM case), making them `(BLK_M, BLK_K, PIPE)` and `(BLK_N, BLK_K, PIPE)`.  
**CN**: SM80 使用 `extern __shared__ char shared_memory[]`——动态共享内存声明为原始字节数组。通过 `reinterpret_cast` 叠加 `SharedStorage` 结构体。这使主机能够计算所需的精确大小（`sizeof(SharedStorage)`）并在 kernel 启动时传入。得到的 `sA` 和 `sB` 张量现在有**第三个维度用于流水线阶段**（TN HGEMM 情况下 PIPE = 3），形状为 `(BLK_M, BLK_K, PIPE)` 和 `(BLK_N, BLK_K, PIPE)`。

---

### Copy Partitioning with Pipeline Dimension (Lines 118–129)

```cpp
ThrCopy thr_copy_a = copy_a.get_slice(threadIdx.x);
Tensor tAgA = thr_copy_a.partition_S(gA);   // (CPY,CPY_M,CPY_K,k)
Tensor tAsA = thr_copy_a.partition_D(sA);   // (CPY,CPY_M,CPY_K,PIPE)
```

**EN**: The key difference from SM70: `tAsA` has an extra trailing `PIPE` dimension because `sA` now has `(BLK_M, BLK_K, PIPE)` shape. This means each thread has a per-pipeline-stage view of its portion of smem. The `PIPE` dimension is indexed explicitly during prefetch and main loop to cycle through stages. There is no intermediate register buffer `tArA` because `cp.async` writes directly from global memory to shared memory without routing through registers.  
**CN**: 与 SM70 的关键区别：`tAsA` 有额外的尾部 `PIPE` 维度，因为 `sA` 现在有 `(BLK_M, BLK_K, PIPE)` 形状。这意味着每个线程有其共享内存部分每流水线阶段的视图。`PIPE` 维度在预取和主循环期间被显式索引以循环经过各阶段。没有中间寄存器缓冲区 `tArA`，因为 `cp.async` 直接从全局内存写入共享内存，无需经过寄存器。

---

### Multi-Stage Prefetch Prologue (Lines 135–150)

```cpp
auto K_PIPE_MAX = size<3>(tAsA);   // = PIPE = 3
int k_tile_count = size<3>(tAgA);  // total k-tiles
int k_tile_next = 0;

CUTE_UNROLL
for (int k_pipe = 0; k_pipe < K_PIPE_MAX-1; ++k_pipe) {
  copy(copy_a, tAgA(_,_,_,k_tile_next), tAsA(_,_,_,k_pipe));
  copy(copy_b, tBgB(_,_,_,k_tile_next), tBsB(_,_,_,k_pipe));
  cp_async_fence();
  --k_tile_count;
  if (k_tile_count > 0) { ++k_tile_next; }
}
```

**EN**: This prologue fills `K_PIPE_MAX - 1 = 2` pipeline stages with async copies before computation begins. Each `copy(copy_a, ...)` issues one or more `cp.async` instructions — hardware instructions that bypass the register file and write directly from L1/L2 cache (global memory source) to shared memory (destination), without stalling the issuing thread. `cp_async_fence()` inserts a `cp.async.commit_group` — it groups all previously issued `cp.async` ops into a "commit group" that can be collectively waited on later. The loop is unrolled because `K_PIPE_MAX - 1 = 2` is a small constant. After this prologue, stages 0 and 1 of smem are being filled asynchronously.  
**CN**: 此序言在计算开始前用异步拷贝填充 `K_PIPE_MAX - 1 = 2` 个流水线阶段。每次 `copy(copy_a, ...)` 发出一个或多个 `cp.async` 指令——这些硬件指令绕过寄存器文件，直接从 L1/L2 缓存（全局内存源）写入共享内存（目的地），不会阻塞发出线程。`cp_async_fence()` 插入 `cp.async.commit_group`——将之前发出的所有 `cp.async` 操作分组为一个"提交组"，稍后可以集体等待。该循环被展开，因为 `K_PIPE_MAX - 1 = 2` 是一个小常数。在此序言之后，共享内存的阶段 0 和 1 正在被异步填充。

---

### MMA Register Allocation (Lines 156–170)

```cpp
ThrMMA thr_mma = mma.get_slice(threadIdx.x);
Tensor tCgC = thr_mma.partition_C(gC);              // (MMA,MMA_M,MMA_N)
Tensor tCrA = thr_mma.partition_fragment_A(sA(_,_,0)); // (MMA,MMA_M,MMA_K)
Tensor tCrB = thr_mma.partition_fragment_B(sB(_,_,0)); // (MMA,MMA_N,MMA_K)
Tensor tCrC = thr_mma.make_fragment_C(tCgC);           // (MMA,MMA_M,MMA_N)
clear(tCrC);
```

**EN**: Note the SM80-specific API: `partition_fragment_A(sA(_,_,0))` takes a 2D slice of the smem tensor (stage 0) as the shape reference for allocating MMA register fragments. This is preferred over `make_fragment_A(tCsA)` when the smem tensor has a pipeline stage dimension, to avoid accidentally including the stage dimension in the register fragment shape. For `SM80_16x8x16_F16F16F16F16_TN`:
- Each MMA instruction is 16×8×16 (M×N×K) over half_t
- With a 2×2 atom tiling and 32×32×16 total tile: `MMA_M = 2`, `MMA_N = 2`, `MMA_K = 1` at the tile level
- Each thread owns 8 floats (4 for C, 2 for A, 2 for B) per MMA atom  

**CN**: 注意 SM80 特定 API：`partition_fragment_A(sA(_,_,0))` 以共享内存张量的 2D 切片（阶段 0）作为分配 MMA 寄存器片段的形状参考。当共享内存张量有流水线阶段维度时，这优于 `make_fragment_A(tCsA)`，避免意外将阶段维度包含在寄存器片段形状中。对于 `SM80_16x8x16_F16F16F16F16_TN`：
- 每条 MMA 指令对 half_t 是 16×8×16（M×N×K）
- 在 2×2 原子分块和 32×32×16 总 tile 下：`MMA_M = 2`、`MMA_N = 2`、`MMA_K = 1`（tile 级别）
- 每个线程每个 MMA 原子拥有 8 个 float（C 4 个，A 2 个，B 2 个）

---

### S2R Copy Atom Retiling (Lines 176–184)

```cpp
TiledCopy s2r_copy_a = make_tiled_copy_A(s2r_atom_a, mma);
ThrCopy   s2r_thr_copy_a = s2r_copy_a.get_slice(threadIdx.x);
Tensor tXsA = s2r_thr_copy_a.partition_S(sA);  // (CPY,MMA_M,MMA_K,PIPE)
Tensor tXrA = s2r_thr_copy_a.retile_D(tCrA);   // (CPY,MMA_M,MMA_K)
```

**EN**: This is the critical SM80 addition — the shared-memory-to-register (S2R) copy path using `LDSM`. 

- `make_tiled_copy_A(s2r_atom_a, mma)` derives a TiledCopy that is compatible with the MMA atom's A operand thread/value layout. It "retiles" the smem view so that the thread→smem element mapping exactly matches what `LDSM` expects.
- `partition_S(sA)` partitions the smem tensor according to this S2R TiledCopy — each thread gets a view of which smem locations it should load.
- `retile_D(tCrA)` retiles the existing MMA register fragment `tCrA` to match the CPY dimension of the S2R atom — this avoids allocating a new register buffer and instead reinterprets the existing one.
- The `SM75_U32x4_LDSM_N` atom issues `ldmatrix.sync.aligned.m8n8.x4` — a single instruction that loads 4 8×8 half-precision matrices using a cooperative warp-level load, with each lane providing a shared-memory pointer and the hardware distributing the loaded values optimally for Tensor Core input.  

**CN**: 这是 SM80 的关键新增——使用 `LDSM` 的共享内存到寄存器（S2R）拷贝路径。

- `make_tiled_copy_A(s2r_atom_a, mma)` 派生出与 MMA 原子 A 操作数线程/值布局兼容的 TiledCopy。它"重新分块"共享内存视图，使线程→共享内存元素映射精确匹配 `LDSM` 期望的格式。
- `partition_S(sA)` 根据此 S2R TiledCopy 分区共享内存张量——每个线程获得其应加载的共享内存位置的视图。
- `retile_D(tCrA)` 重新分块现有 MMA 寄存器片段 `tCrA` 以匹配 S2R 原子的 CPY 维度——避免分配新寄存器缓冲区，而是重新解释现有缓冲区。
- `SM75_U32x4_LDSM_N` 原子发出 `ldmatrix.sync.aligned.m8n8.x4`——一条指令，使用协作 warp 级加载加载 4 个 8×8 半精度矩阵，每个 lane 提供共享内存指针，硬件将加载的值以最优方式分配给 Tensor Core 输入。

---

### Main Loop State Variables (Lines 225–234)

```cpp
int smem_pipe_read  = 0;
int smem_pipe_write = K_PIPE_MAX-1;

Tensor tXsA_p = tXsA(_,_,_,smem_pipe_read);
Tensor tXsB_p = tXsB(_,_,_,smem_pipe_read);

auto K_BLOCK_MAX = size<2>(tCrA);
```

**EN**: Two integer indices implement the circular buffer over the PIPE smem stages:
- `smem_pipe_read`: the stage currently being consumed for MMA (starts at 0)
- `smem_pipe_write`: the stage being filled by the next `cp.async` (starts at PIPE-1 = 2, i.e., the stage that was NOT prefetched in the prologue)

`tXsA_p` is a pointer slice into the current read stage of smem — updated each iteration. `K_BLOCK_MAX` is the number of MMA sub-steps per k-tile (`BLK_K / mma_k_size = 64 / 16 = 4` for the TN HGEMM).  

**CN**: 两个整数索引实现 PIPE 共享内存阶段的循环缓冲区：
- `smem_pipe_read`：当前被 MMA 消耗的阶段（从 0 开始）
- `smem_pipe_write`：被下一个 `cp.async` 填充的阶段（从 PIPE-1 = 2 开始，即序言中未预取的阶段）

`tXsA_p` 是共享内存当前读取阶段的指针切片——每次迭代更新。`K_BLOCK_MAX` 是每个 k-tile 的 MMA 子步骤数（对 TN HGEMM：`BLK_K / mma_k_size = 64 / 16 = 4`）。

---

### Register Pipeline Prefetch (Lines 238–246)

```cpp
if (K_BLOCK_MAX > 1) {
  cp_async_wait<K_PIPE_MAX-2>();
  __syncthreads();
  copy(s2r_atom_a, tXsA_p(_,_,Int<0>{}), tXrA(_,_,Int<0>{}));
  copy(s2r_atom_b, tXsB_p(_,_,Int<0>{}), tXrB(_,_,Int<0>{}));
}
```

**EN**: `cp_async_wait<K_PIPE_MAX-2>()` generates `cp.async.wait_group N` which stalls until at most N commit groups are outstanding. With `K_PIPE_MAX-2 = 1`, this waits until the first prefetched stage (stage 0) is fully committed into smem. The subsequent `__syncthreads()` ensures all threads see the committed data. Then the first k-block's A and B data is loaded from smem into MMA registers via `LDSM`. This seeds the register pipeline exactly as the SM70 case, but now with `LDSM` and an async-smem-backed pipeline.  
**CN**: `cp_async_wait<K_PIPE_MAX-2>()` 生成 `cp.async.wait_group N`，阻塞直到最多有 N 个提交组未完成。`K_PIPE_MAX-2 = 1` 时，等待直到第一个预取阶段（阶段 0）完全提交到共享内存。随后的 `__syncthreads()` 确保所有线程看到已提交的数据。然后通过 `LDSM` 将第一个 k-block 的 A 和 B 数据从共享内存加载到 MMA 寄存器。这与 SM70 情况相同地播种寄存器流水线，但现在使用 `LDSM` 和异步共享内存支持的流水线。

---

### Main Loop Comment (Lines 249–258)

```cpp
// PIPELINED MAIN LOOP
// TUTORIAL: Example of a gemm loop that pipelines shared memory
//           using SM80's cp.async instructions and explicit pipelines
//           in shared memory.
//   Data is read from global(k_tile_next) to shared(smem_pipe_write).
//   Data is read from shared(smem_pipe_read) to registers(k_block_next).
//   Data is computed on registers(k_block).
//
//   This allows all copies and compute to overlap:
//     Copy from gmem->smem can overlap with copies from smem->rmem and compute on rmem.
//     Copy from smem->rmem can overlap with compute on rmem.
```

**EN**: The tutorial comment precisely describes the three-level pipeline enabled by SM80:
1. **gmem → smem** (async, via `cp.async`): Overlaps with stages 2 and 3
2. **smem → rmem** (via `LDSM`): Overlaps with stage 3
3. **rmem → compute** (MMA): The "inner most" stage, always running

This is more powerful than SM70's two-level pipeline because `cp.async` truly decouples the gmem→smem transfer from all register and compute activity.  

**CN**: 教程注释精确描述了 SM80 启用的三级流水线：
1. **全局内存→共享内存**（异步，通过 `cp.async`）：与阶段 2 和 3 重叠
2. **共享内存→寄存器**（通过 `LDSM`）：与阶段 3 重叠
3. **寄存器→计算**（MMA）："最内层"阶段，始终运行

这比 SM70 的两级流水线更强大，因为 `cp.async` 真正将全局内存→共享内存传输与所有寄存器和计算活动解耦。

---

### Three-Level Pipelined Main Loop (Lines 261–301)

```cpp
CUTE_NO_UNROLL
while (k_tile_count > -(K_PIPE_MAX-1))
{
  CUTE_UNROLL
  for (int k_block = 0; k_block < K_BLOCK_MAX; ++k_block)
  {
    if (k_block == K_BLOCK_MAX - 1) {
      tXsA_p = tXsA(_,_,_,smem_pipe_read);
      tXsB_p = tXsB(_,_,_,smem_pipe_read);
      cp_async_wait<K_PIPE_MAX-2>();
      __syncthreads();
    }

    // smem -> rmem: load k_block+1 (prefetch next)
    auto k_block_next = (k_block + Int<1>{}) % K_BLOCK_MAX;
    copy(s2r_atom_a, tXsA_p(_,_,k_block_next), tXrA(_,_,k_block_next));
    copy(s2r_atom_b, tXsB_p(_,_,k_block_next), tXrB(_,_,k_block_next));

    // gmem -> smem: issue async copy for next k-tile
    if (k_block == 0) {
      copy(copy_a, tAgA(_,_,_,k_tile_next), tAsA(_,_,_,smem_pipe_write));
      copy(copy_b, tBgB(_,_,_,k_tile_next), tBsB(_,_,_,smem_pipe_write));
      cp_async_fence();
      --k_tile_count;
      if (k_tile_count > 0) { ++k_tile_next; }
      smem_pipe_write = smem_pipe_read;
      smem_pipe_read = (smem_pipe_read == K_PIPE_MAX-1) ? 0 : smem_pipe_read+1;
    }

    // compute: MMA on current k_block
    gemm(mma, tCrA(_,_,k_block), tCrB(_,_,k_block), tCrC);
  }
}
```

**EN**: Detailed breakdown of the three concurrent pipelines each k_block iteration:

**Event at `k_block == K_BLOCK_MAX - 1` (last sub-step of current k-tile)**:
- Update `tXsA_p`/`tXsB_p` to point to the NEW `smem_pipe_read` stage — this is the next tile just fetched asynchronously
- `cp_async_wait<K_PIPE_MAX-2>()` = wait until ≤1 commit group is in-flight → ensures the next-to-read stage is complete in smem
- `__syncthreads()` — barrier so all threads agree smem is ready

**Prefetch smem → rmem (every iteration)**:
- `copy(s2r_atom_a, tXsA_p(_,_,k_block_next), tXrA(_,_,k_block_next))` — issues `LDSM` for the next k-block's data. This overlaps with the MMA computation below. `k_block_next` is computed as `(k_block + 1) % K_BLOCK_MAX` — it's a **static** modulo because `K_BLOCK_MAX` is a compile-time constant, enabling the compiler to generate all LDSM addresses at compile time.

**Issue gmem → smem copy (only at `k_block == 0`)**:
- Writes the next k-tile into `smem_pipe_write` stage using `cp.async`
- `cp_async_fence()` commits the group — this is the async load for a tile that will be read 2 tiles later
- Advances the circular buffer: `smem_pipe_write = smem_pipe_read`, `smem_pipe_read = (smem_pipe_read + 1) % K_PIPE_MAX`

**MMA computation**:
- `gemm(mma, tCrA(_,_,k_block), tCrB(_,_,k_block), tCrC)` — executes `mma.sync.aligned` (Tensor Core instruction) on currently-resident register data

**Loop termination**: `k_tile_count` starts at the total number of k-tiles and is decremented at `k_block==0`. The condition `k_tile_count > -(K_PIPE_MAX-1)` ensures the loop runs for extra "drain" iterations to flush the async pipeline after all gmem tiles have been issued.  

**CN**: 每次 k_block 迭代中三个并发流水线的详细分解：

**`k_block == K_BLOCK_MAX - 1` 时的事件（当前 k-tile 的最后子步骤）**：
- 更新 `tXsA_p`/`tXsB_p` 指向新的 `smem_pipe_read` 阶段——这是刚刚异步获取的下一个 tile
- `cp_async_wait<K_PIPE_MAX-2>()` = 等待直到 ≤1 个提交组在途→确保下一个要读取的阶段在共享内存中完成
- `__syncthreads()`——屏障，使所有线程同意共享内存已准备好

**预取共享内存→寄存器（每次迭代）**：
- `copy(s2r_atom_a, tXsA_p(_,_,k_block_next), tXrA(_,_,k_block_next))` — 为下一个 k-block 的数据发出 `LDSM`。这与下方 MMA 计算重叠。`k_block_next` 计算为 `(k_block + 1) % K_BLOCK_MAX`——这是**静态**取模，因为 `K_BLOCK_MAX` 是编译期常数，使编译器能在编译时生成所有 LDSM 地址。

**发出全局内存→共享内存拷贝（仅在 `k_block == 0` 时）**：
- 使用 `cp.async` 将下一个 k-tile 写入 `smem_pipe_write` 阶段
- `cp_async_fence()` 提交该组——这是将在 2 个 tile 后读取的 tile 的异步加载
- 推进循环缓冲区：`smem_pipe_write = smem_pipe_read`，`smem_pipe_read = (smem_pipe_read + 1) % K_PIPE_MAX`

**MMA 计算**：
- `gemm(mma, tCrA(_,_,k_block), tCrB(_,_,k_block), tCrC)` — 对当前驻留寄存器数据执行 `mma.sync.aligned`（Tensor Core 指令）

**循环终止**：`k_tile_count` 从总 k-tile 数开始，在 `k_block==0` 时递减。条件 `k_tile_count > -(K_PIPE_MAX-1)` 确保循环额外运行"排空"迭代，在所有全局内存 tile 发出后刷新异步流水线。

---

### Epilogue (Line 309)

```cpp
axpby(alpha, tCrC, beta, tCgC);
```

**EN**: Same as SM70: scales accumulators by `alpha`, adds `beta * C`, writes to global memory. For the TN HGEMM, all values are `half_t`.  
**CN**: 与 SM70 相同：将累加器乘以 `alpha`，加上 `beta * C`，写回全局内存。对 TN HGEMM，所有值均为 `half_t`。

---

### `gemm_nt` (half_t) — Not Implemented (Lines 312–323)

```cpp
template <class Alpha, class Beta>
void gemm_nt(int m, int n, int k, ..., cute::half_t const* A, ...) {
  assert(false && "Not implemented");
}
```

**EN**: The NT (non-transposed A) half-precision GEMM is declared but not implemented. This is because the Tensor Core MMA instruction `SM80_16x8x16_F16F16F16F16_TN` requires A to be in row-major format (transposed), making NT configuration non-trivial to set up with the same atom. Marking it as not-implemented prevents silent incorrect results.  
**CN**: NT（A 非转置）半精度 GEMM 已声明但未实现。这是因为 Tensor Core MMA 指令 `SM80_16x8x16_F16F16F16F16_TN` 要求 A 为行主序（转置），使得用同一原子设置 NT 配置非常复杂。标记为未实现可防止静默错误结果。

---

### `gemm_tn` (half_t) — TN HGEMM with Tensor Cores (Lines 326–430)

```cpp
void gemm_tn(int m, int n, int k, ...,
             cute::half_t const* A, ..., cute::half_t const* B, ...)
{
  auto bM = Int<128>{};
  auto bN = Int<128>{};
  auto bK = Int< 64>{};
  auto bP = Int<  3>{};  // Pipeline stages

  // Swizzled smem layout for LDSM + 128-bit k-major loads
  auto swizzle_atom = composition(
    Swizzle<3,3,3>{},
    Layout<Shape<_8, Shape<_8,_8>>,
           Stride<_8, Stride<_1,_64>>>{});

  auto sA = tile_to_shape(swizzle_atom, make_shape(bM,bK,bP));
  auto sB = tile_to_shape(swizzle_atom, make_shape(bN,bK,bP));

  TiledCopy copyA = make_tiled_copy(
    Copy_Atom<SM80_CP_ASYNC_CACHEALWAYS<uint128_t>, cute::half_t>{},
    Layout<Shape<_16,_8>,Stride<_8,_1>>{},  // 16x8 thr layout, k-major
    Layout<Shape< _1,_8>>{});               // 1x8 val layout, k-major

  TiledMMA mmaC = make_tiled_mma(
    SM80_16x8x16_F16F16F16F16_TN{},
    Layout<Shape<_2,_2>>{},   // 2x2 MMA atoms
    Tile<_32,_32,_16>{});     // 32x32x16 tiled MMA

  Copy_Atom<SM75_U32x4_LDSM_N, half_t> s2r_atom_A;
  Copy_Atom<SM75_U32x4_LDSM_N, half_t> s2r_atom_B;
```

**EN**: This is the fully optimized SM80 HGEMM configuration. Key choices explained:

**BLK_K = 64**: Much larger than SM70's 8. SM80 Tensor Cores consume 16 elements along K per MMA call, and with `K_BLOCK_MAX = BLK_K / 16 = 4` sub-steps, we amortize the smem pipeline overhead over more computation.

**bP = 3 pipeline stages**: 3-stage pipeline provides enough buffering to hide both the gmem→smem latency (hundreds of cycles) and the smem→rmem LDSM latency, maximizing Tensor Core utilization.

**Swizzle layout**: `Swizzle<3,3,3>{}` with the given base layout creates a permuted smem indexing that:
- Ensures 128-bit aligned accesses from global memory (for `cp.async` issuing `LDG.128`)
- Avoids shared memory bank conflicts for LDSM loads, which access 8 elements per lane in a specific pattern
- The `Stride<_8, Stride<_1,_64>>` in the base layout creates a "column-major within 8×8 blocks, tiles in K" structure that the swizzle further permutes

**`SM80_CP_ASYNC_CACHEALWAYS<uint128_t>`**: Issues `cp.async.ca.shared.global` loading 16 bytes (128 bits = 8 half_t) per thread per call. `CACHEALWAYS` fills the L2 cache (vs `CACHEGLOBAL` which bypasses L2).

**Thread layout `<_16,_8>` with `Stride<_8,_1>`**: 128 threads total (128 = 16×8). Stride 8 in M and 1 in K means consecutive threads are adjacent in K — perfect for coalesced global loads from row-major A.

**Val layout `<_1,_8>`**: Each thread loads 1×8 = 8 half_t values per call, packed in the K direction — this is the 128-bit vectorized load.

**`SM80_16x8x16_F16F16F16F16_TN`**: The SM80 Tensor Core `mma.sync.aligned.m16n8k16.row.col.f16.f16.f16.f16` instruction. "TN" means A is transposed (row-major) and B is non-transposed (column-major in this naming convention). Dimensions: M=16, N=8, K=16, all half_t.

**`Tile<_32,_32,_16>`**: The tiled MMA covers a 32×32×16 region, using a 2×2 arrangement of 16×8×16 atoms (filling 32×16 in MN, then doubled in N via the 2×2 grid to reach 32×32×16). 128 threads participate.

**`SM75_U32x4_LDSM_N`**: `ldmatrix.sync.aligned.m8n8.x4.b16` — loads 4 matrices of 8×8 half_t from shared memory cooperatively across a warp. Each of 32 lanes provides an address; the hardware loads 128 bytes total and distributes them in exactly the register layout required by the MMA instruction's A or B operand.  

**CN**: 这是完全优化的 SM80 HGEMM 配置。关键选择说明：

**BLK_K = 64**：比 SM70 的 8 大得多。SM80 Tensor Core 每次 MMA 调用消耗 K 方向 16 个元素，`K_BLOCK_MAX = BLK_K / 16 = 4` 个子步骤将共享内存流水线开销分摊到更多计算上。

**bP = 3 流水线阶段**：3 阶段流水线提供足够的缓冲以隐藏全局内存→共享内存延迟（数百个周期）和共享内存→寄存器 LDSM 延迟，最大化 Tensor Core 利用率。

**Swizzle 布局**：`Swizzle<3,3,3>{}` 与给定的基础布局创建一个排列的共享内存索引，它：
- 确保全局内存的 128 位对齐访问（用于 `cp.async` 发出 `LDG.128`）
- 避免 LDSM 加载的共享内存 bank 冲突，LDSM 按特定模式每 lane 访问 8 个元素
- 基础布局中的 `Stride<_8, Stride<_1,_64>>` 创建"8×8 块内列主序，K 方向排列"结构，swizzle 进一步对其进行排列

**`SM80_CP_ASYNC_CACHEALWAYS<uint128_t>`**：发出 `cp.async.ca.shared.global`，每线程每次调用加载 16 字节（128 位 = 8 个 half_t）。`CACHEALWAYS` 填充 L2 缓存（而 `CACHEGLOBAL` 绕过 L2）。

**线程布局 `<_16,_8>` 带 `Stride<_8,_1>`**：共 128 个线程（128 = 16×8）。M 方向步长 8、K 方向步长 1 意味着连续线程在 K 方向相邻——完美适合从行主序 A 的合并全局加载。

**值布局 `<_1,_8>`**：每个线程每次调用加载 1×8 = 8 个 half_t 值，在 K 方向打包——这是 128 位向量化加载。

**`SM80_16x8x16_F16F16F16F16_TN`**：SM80 Tensor Core `mma.sync.aligned.m16n8k16.row.col.f16.f16.f16.f16` 指令。"TN"表示 A 转置（行主序），B 非转置（在此命名约定中为列主序）。维度：M=16，N=8，K=16，均为 half_t。

**`Tile<_32,_32,_16>`**：分块 MMA 覆盖 32×32×16 区域，使用 2×2 排列的 16×8×16 原子（在 MN 中填充 32×16，然后在 N 方向通过 2×2 网格加倍以达到 32×32×16）。128 个线程参与。

**`SM75_U32x4_LDSM_N`**：`ldmatrix.sync.aligned.m8n8.x4.b16`——跨 warp 协作从共享内存加载 4 个 8×8 half_t 矩阵。32 个 lane 各提供一个地址；硬件总共加载 128 字节，并以 MMA 指令 A 或 B 操作数所需的确切寄存器布局分配它们。

---

### Kernel Launch Configuration (Lines 403–429)

```cpp
int smem_size = int(sizeof(SharedStorage<cute::half_t, cute::half_t,
                                        decltype(sA), decltype(sB)>));
dim3 dimBlock(size(mmaC));      // 128 threads
dim3 dimGrid(ceil_div(M,bM), ceil_div(N,bN));

cudaFuncSetAttribute(kernel_fptr,
  cudaFuncAttributeMaxDynamicSharedMemorySize, smem_size);
cudaFuncSetAttribute(kernel_fptr,
  cudaFuncAttributePreferredSharedMemoryCarveout, 100);

kernel_fptr<<<dimGrid, dimBlock, smem_size, stream>>>(...);
```

**EN**: Three important SM80-specific launch details:

1. **Dynamic smem size**: `smem_size = sizeof(SharedStorage)` gives the exact bytes needed for 3 pipeline stages × (128×64 + 128×64) × 2 bytes = 98304 bytes.

2. **`cudaFuncAttributeMaxDynamicSharedMemorySize`**: SM80 supports up to 164 KB of shared memory per SM with opt-in, far beyond the default 48 KB. This attribute must be set before launch to allow the larger allocation. The `kernel_fptr` (explicit function pointer typed with all template arguments) is required because CUDA's attribute API needs a concrete function pointer.

3. **`cudaFuncAttributePreferredSharedMemoryCarveout = 100`**: Requests 100% of the L1/smem unified cache be allocated as shared memory (0% as L1 data cache). On SM80, the L1 and smem share a 192 KB pool. This maximizes smem for the large pipeline buffers.  

**CN**: 三个重要的 SM80 特定启动细节：

1. **动态共享内存大小**：`smem_size = sizeof(SharedStorage)` 给出所需的精确字节数：3 流水线阶段 × (128×64 + 128×64) × 2 字节 = 98304 字节。

2. **`cudaFuncAttributeMaxDynamicSharedMemorySize`**：SM80 支持每个 SM 最多 164 KB 的共享内存（需选择加入），远超默认的 48 KB。必须在启动前设置此属性以允许更大的分配。需要 `kernel_fptr`（带所有模板参数的显式函数指针），因为 CUDA 的属性 API 需要具体的函数指针。

3. **`cudaFuncAttributePreferredSharedMemoryCarveout = 100`**：请求将 100% 的 L1/共享内存统一缓存分配为共享内存（0% 作为 L1 数据缓存）。在 SM80 上，L1 和共享内存共享 192 KB 池。这为大型流水线缓冲区最大化共享内存。

---

### `gemm_nt` (float) and `gemm_tn` (float) with `UniversalFMA` (Lines 432–580)

```cpp
// gemm_nt — NT float GEMM
TiledCopy copyA = make_tiled_copy(
  Copy_Atom<SM80_CP_ASYNC_CACHEALWAYS<uint128_t>, TA>{},
  Layout<Shape<_32,_8>>{},   // m-major
  Layout<Shape< _4,_1>>{});

TiledMMA mmaC = make_tiled_mma(UniversalFMA<TC,TA,TB>{},
                               Layout<Shape<_16,_16,_1>>{});

// s2r atom: AutoVectorizingCopy (plain LDS)
gemm_device<<<...>>>(... Copy_Atom<AutoVectorizingCopy, TA>{} ...);
```

**EN**: These float variants use `SM80_CP_ASYNC_CACHEALWAYS` for the gmem→smem copy (gaining async pipelining even with `UniversalFMA`) but use `AutoVectorizingCopy` (plain `LDS` or register-to-register) for the smem→rmem stage. The S2R atom is passed as an inline object rather than a named variable. The smem layout has `bP=3` pipeline stages but no swizzle — straightforward row/column-major with optional padding in the TN case. `BLK_K = 8` (same as SM70) because `UniversalFMA` operates on scalar elements, so there is no benefit to larger K tiles. This configuration demonstrates that the SM80 `cp.async` pipeline framework is usable even without Tensor Cores.  
**CN**: 这些 float 变体使用 `SM80_CP_ASYNC_CACHEALWAYS` 进行全局内存→共享内存拷贝（即使使用 `UniversalFMA` 也能获得异步流水线），但使用 `AutoVectorizingCopy`（普通 `LDS` 或寄存器到寄存器）进行共享内存→寄存器阶段。S2R 原子作为内联对象而非命名变量传入。共享内存布局有 `bP=3` 流水线阶段但无 swizzle——TN 情况下采用直接的行/列主序，可选填充。`BLK_K = 8`（与 SM70 相同），因为 `UniversalFMA` 在标量元素上操作，较大 K tile 没有好处。此配置演示了即使没有 Tensor Core，SM80 `cp.async` 流水线框架也可使用。

---

### TN float — Padded + Tiled Smem Layout (Lines 538–543)

```cpp
auto sA_atom = make_layout(
    make_shape(bM, bK),
    make_stride(Int<1>{}, bM+Int<1>{}));  // padded m-major

auto sA = tile_to_shape(sA_atom, make_shape(bM, bK, bP));
auto sB = tile_to_shape(sA_atom, make_shape(bN, bK, bP));
```

**EN**: `tile_to_shape` takes a "base atom" layout (a 2D BLK_M×BLK_K tile with bank-conflict-free padding) and tiles it to fill a larger shape that includes the pipeline dimension (bP). This is the correct way to extend a per-tile layout to a multi-stage smem layout in CuTe — it preserves the stride/padding structure of the atom within each stage. Note that `sB` uses `sA_atom` (not `sB_atom`) for its tiling — the same m-major padded layout is reused, which works because both A and B slabs have the same inner structure in the TN case.  
**CN**: `tile_to_shape` 接受"基础原子"布局（具有无 bank 冲突填充的 2D BLK_M×BLK_K tile），将其平铺以填充包括流水线维度（bP）的更大形状。这是在 CuTe 中将每 tile 布局扩展到多阶段共享内存布局的正确方式——它在每个阶段内保留原子的 stride/填充结构。注意 `sB` 使用 `sA_atom`（而非 `sB_atom`）进行平铺——重用相同的 m-major 填充布局，这在 TN 情况下有效，因为 A 和 B 板块具有相同的内部结构。

---

### `gemm` Dispatcher (Lines 582–600)

```cpp
template <class TA, class TB, class TC, class Alpha, class Beta>
void gemm(char transA, char transB, int m, int n, int k, ...) {
  if (transA == 'N' && transB == 'T') return gemm_nt(...);
  if (transA == 'T' && transB == 'N') return gemm_tn(...);
  assert(false && "Not implemented");
}
```

**EN**: Runtime dispatch based on transpose flags. Template deduction selects the correct typed overload — for `half_t` arguments, the `half_t`-specialized `gemm_nt`/`gemm_tn` overloads are chosen; for `float`, the generic template is used.  
**CN**: 基于转置标志的运行时分发。模板推导选择正确的类型化重载——对 `half_t` 参数，选择 `half_t` 特化的 `gemm_nt`/`gemm_tn` 重载；对 `float`，使用通用模板。

---

### `main` Function (Lines 603–717)

```cpp
if (props.major < 8) { ... return 0; }  // SM80 requirement
using TA = cute::half_t; using TB = cute::half_t;
using TC = cute::half_t; using TI = cute::half_t;
// Default: m=5120, n=5120, k=4096, transA='N', transB='T'
// Thrust alloc, random [-1,1] init, warmup + 100 timed iters
printf("CUTE_GEMM: [%6.1f]GFlop/s  (%6.4f)ms\n", ...);
```

**EN**: The main function checks for SM80+, uses `cute::half_t` (CuTe's FP16 wrapper, compatible with `__half`) for all matrix types, defaults to 5120×5120×4096 which on an A100 with `half_t` should achieve close to peak Tensor Core throughput. Prints device name and SM version. 100 iterations for stable timing. No reference comparison — tutorial focus is on CuTe API structure.  
**CN**: main 函数检查 SM80+，对所有矩阵类型使用 `cute::half_t`（CuTe 的 FP16 封装，与 `__half` 兼容），默认 5120×5120×4096，在 A100 上使用 `half_t` 应接近峰值 Tensor Core 吞吐量。打印设备名称和 SM 版本。100 次迭代以获得稳定计时。无参考比较——教程重点是 CuTe API 结构。

---

## Key Concepts / 关键概念

- **`cp.async` / 异步全局内存加载**: SM80 instruction (`cp.async.ca.shared.global`) that copies from global memory to shared memory without routing through registers, freeing the register file and allowing compute to proceed concurrently. Managed via `cp_async_fence()` and `cp_async_wait<N>()`. / SM80 指令，从全局内存直接复制到共享内存，无需经过寄存器，释放寄存器文件并允许计算并发进行。通过 `cp_async_fence()` 和 `cp_async_wait<N>()` 管理。

- **Multi-stage smem pipeline / 多阶段共享内存流水线**: A circular buffer of `PIPE` (=3) stages in shared memory, where writes (via `cp.async`) and reads (for MMA) proceed at different pointers, separated by `PIPE-1` stages of buffering to fully hide global memory latency. / 共享内存中 `PIPE`（=3）阶段的循环缓冲区，写入（通过 `cp.async`）和读取（用于 MMA）在不同指针处进行，通过 `PIPE-1` 阶段的缓冲完全隐藏全局内存延迟。

- **`LDSM` / `ldmatrix` instruction**: `ldmatrix.sync.aligned.m8n8.x4.b16` — a warp-cooperative load that reads 4×(8×8) FP16 matrices from shared memory directly into the register layout required by the Tensor Core instruction, avoiding manual register permutation. / warp 协作加载指令，从共享内存将 4×(8×8) FP16 矩阵直接读入 Tensor Core 指令所需的寄存器布局，避免手动寄存器排列。

- **`SM80_16x8x16_F16F16F16F16_TN` MMA Atom**: Wraps the `mma.sync.aligned.m16n8k16` PTX instruction. Computes D = A×B + C for 16×8×16 tiles of FP16, with A in row-major (T) and B in column-major (N) format in the hardware's internal representation. 32 threads cooperate, each holding 2 elements of A, 2 of B, and 4 of C/D in registers. / 封装 `mma.sync.aligned.m16n8k16` PTX 指令，计算 16×8×16 FP16 tile 的 D = A×B + C，A 为行主序（T），B 为列主序（N）。32 个线程协作，每个线程在寄存器中持有 A 的 2 个元素、B 的 2 个元素和 C/D 的 4 个元素。

- **Swizzle Layout / Swizzle 布局**: A bit-permutation of the smem address based on row and column indices, preventing all threads in a warp from accessing the same smem bank. `Swizzle<3,3,3>` XOR-mixes 3 bits of the row index into the column address — sufficient to eliminate bank conflicts for the 128-bit wide loads used here. / 基于行列索引的共享内存地址位排列，防止 warp 中所有线程访问同一共享内存 bank。`Swizzle<3,3,3>` 将行索引的 3 位 XOR 混合到列地址中——足以消除此处使用的 128 位宽加载的 bank 冲突。

- **`make_tiled_copy_A` / `retile_D`**: Creates a TiledCopy compatible with the MMA atom's operand layout (for S2R), and reinterprets an existing register fragment tensor to match the CPY granularity, avoiding redundant register allocation. / 创建与 MMA 原子操作数布局兼容的 TiledCopy（用于 S2R），并重新解释现有寄存器片段张量以匹配 CPY 粒度，避免冗余寄存器分配。

- **Dynamic smem + `cudaFuncSetAttribute`**: SM80 smem can be up to 164 KB with opt-in via `cudaFuncAttributeMaxDynamicSharedMemorySize`. Combined with `PreferredSharedMemoryCarveout=100`, this maximizes the smem pool at the cost of L1 caching. / SM80 共享内存通过 `cudaFuncAttributeMaxDynamicSharedMemorySize` 选择加入可达 164 KB。结合 `PreferredSharedMemoryCarveout=100`，以牺牲 L1 缓存为代价最大化共享内存池。

- **SM70 → SM80 Progression / SM70→SM80 演进**: SM70 uses `UniversalFMA` (scalar FMA, no Tensor Core) with a two-level gmem→rmem→smem→rmem→compute pipeline. SM80 adds: (1) `cp.async` for true async gmem→smem eliminating register traffic; (2) `LDSM`/`ldmatrix` for optimal smem→rmem in Tensor Core layouts; (3) `mma.sync` Tensor Core instructions for 16× throughput vs scalar FMA; (4) swizzled smem to avoid bank conflicts at 128-bit access width. / SM70 使用 `UniversalFMA`（标量 FMA，无 Tensor Core），具有两级全局内存→寄存器→共享内存→寄存器→计算流水线。SM80 增加：(1) `cp.async` 实现真正的异步全局内存→共享内存，消除寄存器流量；(2) `LDSM`/`ldmatrix` 用于 Tensor Core 布局的最优共享内存→寄存器；(3) `mma.sync` Tensor Core 指令，吞吐量是标量 FMA 的 16 倍；(4) swizzled 共享内存以避免 128 位访问宽度下的 bank 冲突。

---

## Dependencies / 依赖项

- `cute/tensor.hpp` — CuTe core including SM80-specific atoms: `SM80_CP_ASYNC_CACHEALWAYS`, `SM80_16x8x16_F16F16F16F16_TN`, `SM75_U32x4_LDSM_N` / CuTe 核心，包含 SM80 特定原子
- `thrust/host_vector.h`, `thrust/device_vector.h` — CPU/GPU memory management / CPU/GPU 内存管理
- `cutlass/util/print_error.hpp` — `CUTE_CHECK_LAST()` CUDA error checking / CUDA 错误检查宏
- `cutlass/util/GPU_Clock.hpp` — Lightweight GPU wall-clock timer / 轻量级 GPU 计时器
- `cutlass/util/helper_cuda.hpp` — Device properties helpers / 设备属性辅助函数
- CUDA ≥ SM80 (Ampere) — Required for `cp.async`, `ldmatrix`, and SM80 MMA instructions / 需要 SM80（Ampere）或更新架构，用于 `cp.async`、`ldmatrix` 和 SM80 MMA 指令
- `cute::half_t` — CuTe's FP16 scalar type (wraps `__half`) / CuTe 的 FP16 标量类型（封装 `__half`）
- PTX ISA 7.0+ — For `cp.async.ca.shared.global`, `ldmatrix.sync`, `mma.sync.aligned.m16n8k16` / 用于 `cp.async`、`ldmatrix.sync`、`mma.sync.aligned.m16n8k16`
