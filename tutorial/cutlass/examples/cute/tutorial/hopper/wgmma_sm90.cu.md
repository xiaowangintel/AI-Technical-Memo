# wgmma_sm90.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/cute/tutorial/hopper/wgmma_sm90.cu`

**Purpose / 用途**: Demonstrates a complete GEMM kernel on NVIDIA Hopper (SM90) GPUs using CuTe's WGMMA (Warpgroup Matrix Multiply-Accumulate) abstraction with `cp.async`-based staging, software-pipelined shared-memory double/triple-buffering, and swizzled SMEM layouts — without TMA. / 在 NVIDIA Hopper（SM90）GPU 上展示一个完整的 GEMM 内核，使用 CuTe 的 WGMMA（Warpgroup 矩阵乘加）抽象、基于 `cp.async` 的分阶段传输、软件流水线化的共享内存双/三缓冲以及 swizzle 化 SMEM 布局，不使用 TMA。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1–30 — License Header / 许可证头

```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 * ...
 **************************************************************************************************/
```

**EN**: Standard NVIDIA/CUTLASS BSD-3-Clause license block. Required legal header for all CUTLASS source files.  
**CN**: 标准 NVIDIA/CUTLASS BSD-3-Clause 许可证块，是所有 CUTLASS 源文件的必要法律头部。

---

### Lines 31–44 — Includes / 头文件包含

```cpp
#include <cstdlib>
#include <cstdio>
#include <cassert>

#include <thrust/host_vector.h>
#include <thrust/device_vector.h>

#include <cute/tensor.hpp>

#include "cutlass/cluster_launch.hpp"

#include "cutlass/util/print_error.hpp"
#include "cutlass/util/GPU_Clock.hpp"
#include "cutlass/util/helper_cuda.hpp"
```

**EN**: Standard C++ headers (`cstdlib`, `cstdio`, `cassert`) provide basic I/O and assertions. Thrust vectors manage host/device memory automatically. `cute/tensor.hpp` is the entry point for the entire CuTe tensor/layout/copy/MMA algebra. `cluster_launch.hpp` exposes `cutlass::launch_kernel_on_cluster` which wraps CUDA cluster launch APIs introduced in CUDA 12. Utility headers add error checking, timing, and helper macros. Notably, this file does **not** include `cutlass/arch/barrier.h` or `sm90_pipeline.hpp` — those are needed only for the TMA version.  
**CN**: 标准 C++ 头文件提供基本 I/O 和断言；Thrust 向量自动管理主机/设备内存；`cute/tensor.hpp` 是整个 CuTe 张量/布局/拷贝/MMA 代数的入口；`cluster_launch.hpp` 暴露 `cutlass::launch_kernel_on_cluster`，封装了 CUDA 12 引入的集群启动 API；工具头文件添加错误检查、计时和辅助宏。注意：此文件**不**包含 `cutlass/arch/barrier.h` 或 `sm90_pipeline.hpp`，这些仅在 TMA 版本中需要。

---

### Line 46 — Namespace Import / 命名空间导入

```cpp
using namespace cute;
```

**EN**: Brings all CuTe symbols (`make_tensor`, `make_shape`, `make_stride`, `copy`, `gemm`, layouts, etc.) into the global namespace for brevity.  
**CN**: 将所有 CuTe 符号（`make_tensor`、`make_shape`、`make_stride`、`copy`、`gemm`、布局等）引入全局命名空间以简化代码。

---

### Lines 48–56 — `SharedStorage` Struct / 共享内存结构体

```cpp
template <class ElementA,
          class ElementB,
          class SmemLayoutA,  // (M,K,P)
          class SmemLayoutB>  // (N,K,P)
struct SharedStorage
{
  alignas(128) cute::ArrayEngine<ElementA, cosize_v<SmemLayoutA>> A;
  alignas(128) cute::ArrayEngine<ElementB, cosize_v<SmemLayoutB>> B;
};
```

**EN**: Defines the shared-memory layout for one CTA. `cosize_v<Layout>` computes the total number of elements required by the layout (accounting for swizzle stride padding). `alignas(128)` enforces 128-byte alignment required for Hopper's swizzled SMEM banks and for the `cp.async` 128-bit wide loads. The third dimension `P` in the layout shapes is the **pipeline depth** (number of buffered k-tiles). Note that this struct has **no** barrier arrays — unlike the TMA version, synchronization here relies on `__syncthreads()` and `cp_async_wait`.  
**CN**: 定义一个 CTA 的共享内存布局。`cosize_v<Layout>` 计算该布局所需的总元素数（考虑 swizzle 步长填充）。`alignas(128)` 强制 128 字节对齐，这是 Hopper swizzle SMEM bank 和 `cp.async` 128 位宽加载的必要条件。布局形状中的第三维 `P` 是**流水线深度**（缓冲的 k 块数）。注意该结构体**没有** barrier 数组——不同于 TMA 版本，这里的同步依赖 `__syncthreads()` 和 `cp_async_wait`。

---

### Lines 58–70 — Kernel Template Signature / 内核模板签名

```cpp
template <class ProblemShape, class CtaTiler,
          class TA, class AStride, class ASmemLayout, class TiledCopyA,
          class TB, class BStride, class BSmemLayout, class TiledCopyB,
          class TC, class CStride, class TiledMma,
          class Alpha, class Beta>
__global__ static
__launch_bounds__(decltype(size(TiledMma{}))::value)
void
gemm_device(ProblemShape shape_MNK, CtaTiler cta_tiler,
            TA const* A, AStride dA, ASmemLayout sA_layout, TiledCopyA copy_a,
            TB const* B, BStride dB, BSmemLayout sB_layout, TiledCopyB copy_b,
            TC      * C, CStride dC, TiledMma mma,
            Alpha alpha, Beta beta)
```

**EN**: Fully templated CUDA kernel. Every algorithm parameter — layouts, copy atoms, MMA atoms, strides — is a compile-time type, enabling the compiler to specialize and optimize aggressively. `__launch_bounds__` is computed from `size(TiledMma{})`, which equals the number of threads per warpgroup (128 for SM90 WGMMA). Passing `copy_a`, `copy_b`, `mma`, and layout objects **by value** is a CuTe idiom: these are lightweight tag types containing only static information.  
**CN**: 完全模板化的 CUDA 内核。每个算法参数——布局、拷贝原子、MMA 原子、步长——都是编译期类型，使编译器能够积极地特化和优化。`__launch_bounds__` 由 `size(TiledMma{})` 计算得出，对于 SM90 WGMMA 等于每个 warpgroup 的线程数（128）。将 `copy_a`、`copy_b`、`mma` 和布局对象**按值**传递是 CuTe 的惯用法：这些都是只包含静态信息的轻量级标签类型。

---

### Lines 72–89 — Static Precondition Checks / 静态前置条件检查

```cpp
CUTE_STATIC_ASSERT_V(rank(shape_MNK) == Int<3>{});
CUTE_STATIC_ASSERT_V(rank(cta_tiler) == Int<3>{});

CUTE_STATIC_ASSERT_V(size(copy_a) == size(mma));
CUTE_STATIC_ASSERT_V(size(copy_b) == size(mma));

static_assert(is_static<ASmemLayout>::value);
static_assert(is_static<BSmemLayout>::value);

CUTE_STATIC_ASSERT_V(size<0>(ASmemLayout{}) == size<0>(cta_tiler));  // BLK_M
CUTE_STATIC_ASSERT_V(size<0>(BSmemLayout{}) == size<1>(cta_tiler));  // BLK_N
CUTE_STATIC_ASSERT_V(size<1>(ASmemLayout{}) == size<2>(cta_tiler));  // BLK_K
CUTE_STATIC_ASSERT_V(size<1>(BSmemLayout{}) == size<2>(cta_tiler));  // BLK_K

CUTE_STATIC_ASSERT_V(congruent(select<0,2>(shape_MNK), dA));
CUTE_STATIC_ASSERT_V(congruent(select<1,2>(shape_MNK), dB));
CUTE_STATIC_ASSERT_V(congruent(select<0,1>(shape_MNK), dC));
```

**EN**: Compile-time sanity checks that catch mismatches early: problem and tiler must both be rank-3; copy and MMA atoms must agree on thread count; SMEM layouts must be fully static (no runtime shapes); SMEM tile sizes must match CTA tile sizes; stride dimensions must match the corresponding matrix shape (congruence check). These assertions produce readable compiler errors rather than silent runtime bugs.  
**CN**: 编译期合理性检查，用于及早发现不匹配：问题形状和 tiler 都必须是 3 阶；拷贝和 MMA 原子必须在线程数上一致；SMEM 布局必须完全静态（无运行时形状）；SMEM tile 大小必须与 CTA tile 大小匹配；步长维度必须与相应矩阵形状一致（一致性检查）。这些断言会产生可读的编译器错误，而不是无声的运行时错误。

---

### Lines 95–98 — Full Global-Memory Tensors / 完整全局内存张量

```cpp
Tensor mA = make_tensor(make_gmem_ptr(A), select<0,2>(shape_MNK), dA); // (M,K)
Tensor mB = make_tensor(make_gmem_ptr(B), select<1,2>(shape_MNK), dB); // (N,K)
Tensor mC = make_tensor(make_gmem_ptr(C), select<0,1>(shape_MNK), dC); // (M,N)
```

**EN**: Creates "logical" view tensors over the full global arrays. `make_gmem_ptr` wraps the raw pointer with a GMEM address space tag. `select<0,2>(shape_MNK)` extracts dimensions (M,K) from the 3-tuple. No data movement occurs; these are zero-cost abstractions.  
**CN**: 在完整全局数组上创建"逻辑"视图张量。`make_gmem_ptr` 用 GMEM 地址空间标签包装原始指针。`select<0,2>(shape_MNK)` 从三元组中提取 (M,K) 维度。不发生数据移动，这些是零成本的抽象。

---

### Lines 100–104 — CTA-Level Tile Slicing / CTA 级别 Tile 切片

```cpp
auto cta_coord = make_coord(blockIdx.x, blockIdx.y, _);
Tensor gA = local_tile(mA, cta_tiler, cta_coord, Step<_1, X,_1>{});  // (BLK_M,BLK_K,k)
Tensor gB = local_tile(mB, cta_tiler, cta_coord, Step< X,_1,_1>{});  // (BLK_N,BLK_K,k)
Tensor gC = local_tile(mC, cta_tiler, cta_coord, Step<_1,_1, X>{});  // (BLK_M,BLK_N)
```

**EN**: `local_tile` selects this CTA's slice of the global problem. `Step<_1, X, _1>` for `gA` means "tile along M (_1) and K (_1), but not along N (X)"; the result is a rank-3 tensor `(BLK_M, BLK_K, k)` where the last mode iterates over all k-tiles. `gC` uses `Step<_1,_1,X>` — tile M and N, but collapse K — producing a rank-2 tile `(BLK_M, BLK_N)`.  
**CN**: `local_tile` 选择此 CTA 在全局问题中的切片。`gA` 的 `Step<_1, X, _1>` 表示"沿 M(_1) 和 K(_1) 分块，但不沿 N(X)"；结果是形状为 `(BLK_M, BLK_K, k)` 的三阶张量，最后一维遍历所有 k-tile。`gC` 使用 `Step<_1,_1,X>`——分块 M 和 N，但折叠 K——得到形状为 `(BLK_M, BLK_N)` 的二阶 tile。

---

### Lines 106–111 — Shared Memory Tensors / 共享内存张量

```cpp
extern __shared__ char shared_memory[];
using SharedStorage = SharedStorage<TA, TB, ASmemLayout, BSmemLayout>;
SharedStorage& smem = *reinterpret_cast<SharedStorage*>(shared_memory);
Tensor sA = make_tensor(make_smem_ptr(smem.A.begin()), ASmemLayout{}); // (BLK_M,BLK_K,PIPE)
Tensor sB = make_tensor(make_smem_ptr(smem.B.begin()), BSmemLayout{}); // (BLK_N,BLK_K,PIPE)
```

**EN**: Maps the raw `extern __shared__` allocation onto the typed `SharedStorage` struct, then wraps A/B arrays as CuTe tensors with the pipeline-aware layout `(BLK_M, BLK_K, PIPE)`. The pipeline dimension `PIPE` (e.g., 3) means there are 3 independent SMEM buffer slots for A and B, allowing overlap of GMEM→SMEM copies with MMA computation.  
**CN**: 将原始 `extern __shared__` 分配映射到类型化的 `SharedStorage` 结构体，然后用流水线感知布局 `(BLK_M, BLK_K, PIPE)` 将 A/B 数组包装为 CuTe 张量。流水线维度 `PIPE`（如 3）意味着 A 和 B 各有 3 个独立的 SMEM 缓冲槽，允许 GMEM→SMEM 拷贝与 MMA 计算重叠。

---

### Lines 117–130 — Thread-Level Copy Partitioning for A and B / 线程级拷贝分区（A 和 B）

```cpp
ThrCopy thr_copy_a = copy_a.get_slice(threadIdx.x);
Tensor tAgA = thr_copy_a.partition_S(gA);                            // (CPY,CPY_M,CPY_K,k)
Tensor sA_ = as_position_independent_swizzle_tensor(sA);
Tensor tAsA = thr_copy_a.partition_D(sA_);                           // (CPY,CPY_M,CPY_K,PIPE)

ThrCopy thr_copy_b = copy_b.get_slice(threadIdx.x);
Tensor tBgB = thr_copy_b.partition_S(gB);                            // (CPY,CPY_N,CPY_K,k)
Tensor sB_ = as_position_independent_swizzle_tensor(sB);
Tensor tBsB = thr_copy_b.partition_D(sB_);                           // (CPY,CPY_N,CPY_K,PIPE)
```

**EN**: `get_slice(threadIdx.x)` produces a per-thread view of the `TiledCopy` object. `partition_S` slices the global source tensor to show only this thread's elements; `partition_D` does the same for the destination. The call to `as_position_independent_swizzle_tensor` is critical: it marks the SMEM tensor as using a **position-independent swizzle** — meaning the swizzle is computed relative to each thread's write position, not from address 0. This is required for `SM80_CP_ASYNC` when the destination has a swizzle layout, because `cp.async` uses byte-addressed instructions.  
**CN**: `get_slice(threadIdx.x)` 为 `TiledCopy` 对象生成每线程视图。`partition_S` 切分全局源张量，只显示该线程的元素；`partition_D` 对目标做同样操作。调用 `as_position_independent_swizzle_tensor` 至关重要：它将 SMEM 张量标记为使用**位置无关的 swizzle**——即 swizzle 相对于每个线程的写入位置计算，而不是从地址 0 开始。当目标有 swizzle 布局时，`SM80_CP_ASYNC` 需要这样做，因为 `cp.async` 使用字节寻址指令。

---

### Lines 132–151 — Commented-Out Prefetch Block / 注释掉的预取块

```cpp
// auto K_PIPE_MAX = size<3>(tAsA);
// int k_tile_count = size<3>(tAgA);
// int k_tile_next = 0;
// CUTE_UNROLL
// for (int k_pipe = 0; k_pipe < K_PIPE_MAX-1; ++k_pipe) {
//   copy(copy_a, tAgA(_,_,_,k_tile_next), tAsA(_,_,_,k_pipe));
//   copy(copy_b, tBgB(_,_,_,k_tile_next), tBsB(_,_,_,k_pipe));
//   cp_async_fence();
//   --k_tile_count;
//   if (k_tile_count > 0) { ++k_tile_next; }
// }
```

**EN**: An alternative (simpler) prefetch loop left as educational reference. It pre-fills `K_PIPE_MAX - 1` pipeline slots before the main loop starts. The active code below is more complete (handles boundary conditions).  
**CN**: 作为教学参考保留的另一种（更简单）预取循环。它在主循环开始前预填充 `K_PIPE_MAX - 1` 个流水线槽。下面的有效代码更完整（处理边界条件）。

---

### Lines 157–173 — MMA Partitioning and Fragment Allocation / MMA 分区与片段分配

```cpp
ThrMMA thr_mma = mma.get_slice(threadIdx.x);
Tensor tCsA = thr_mma.partition_A(sA);    // (MMA,MMA_M,MMA_K,PIPE)
Tensor tCsB = thr_mma.partition_B(sB);    // (MMA,MMA_N,MMA_K,PIPE)
Tensor tCgC = thr_mma.partition_C(gC);    // (MMA,MMA_M,MMA_N)

Tensor tCrA = thr_mma.make_fragment_A(tCsA);  // (MMA,MMA_M,MMA_K,PIPE)
Tensor tCrB = thr_mma.make_fragment_B(tCsB);  // (MMA,MMA_N,MMA_K,PIPE)
Tensor tCrC = thr_mma.make_fragment_C(tCgC);  // (MMA,MMA_M,MMA_N)

clear(tCrC);
```

**EN**: `get_slice` produces a thread-specific view of the `TiledMMA`. `partition_A/B/C` compute which elements each thread owns. `make_fragment_A/B` allocate **register tensors** that mirror the SMEM layout but live in registers — in the non-TMA WGMMA path these fragments are actually read from registers into the WGMMA pipeline. `make_fragment_C` allocates the accumulator registers. `clear` zero-initializes accumulators. The `tCrA`/`tCrB` tensors retain the PIPE dimension so the pipeline loop can select the correct slot.  
**CN**: `get_slice` 为 `TiledMMA` 生成线程特定视图。`partition_A/B/C` 计算每个线程拥有哪些元素。`make_fragment_A/B` 分配**寄存器张量**，它们镜像 SMEM 布局但存储在寄存器中——在非 TMA 的 WGMMA 路径中，这些片段实际上从寄存器读入 WGMMA 流水线。`make_fragment_C` 分配累加器寄存器。`clear` 将累加器零初始化。`tCrA`/`tCrB` 张量保留 PIPE 维度，以便流水线循环选择正确的槽。

---

### Lines 175–206 — Debug Print Blocks (Disabled) / 调试打印块（已禁用）

```cpp
#if 0
  if(thread0()) {
    print("  mA : "); print(  mA); print("\n");
    ...
  }
#endif
```

**EN**: Three conditional debug print blocks controlled by `#if 0` (disabled at compile time). Each prints the shape, stride, and memory-pointer details of the key tensors. To enable, change `#if 0` to `#if 1`. Useful during development to verify tensor shapes before running the full GEMM.  
**CN**: 三个由 `#if 0` 控制的条件调试打印块（编译期禁用）。每个块打印关键张量的形状、步长和内存指针详情。要启用，将 `#if 0` 改为 `#if 1`。在开发期间用于在运行完整 GEMM 之前验证张量形状。

---

### Lines 208–226 — Prefetch Loop / 预取循环

```cpp
#if 1

auto K_TILE_MAX  = size<3>(tAgA);
auto K_PIPE_MAX  = size<3>(tAsA);

// Prefetch all but the last
CUTE_UNROLL
for (int k = 0; k < K_PIPE_MAX-1; ++k)
{
  copy(copy_a, tAgA(_,_,_,k), tAsA(_,_,_,k));
  copy(copy_b, tBgB(_,_,_,k), tBsB(_,_,_,k));
  cp_async_fence();
}

clear(tCrC);
__syncthreads();
```

**EN**: This is the active code path (`#if 1`). `K_TILE_MAX` is the total number of k-tiles to process; `K_PIPE_MAX` is the pipeline depth (e.g., 3 for triple-buffering). The prefetch loop fills `K_PIPE_MAX - 1` pipeline slots (e.g., 2 out of 3) with `cp.async` loads. Each `cp_async_fence()` creates a fence group so `cp_async_wait` can later wait for a specific group. `__syncthreads()` ensures all threads have reached this point before the main loop accesses SMEM.  
**CN**: 这是有效的代码路径（`#if 1`）。`K_TILE_MAX` 是要处理的 k-tile 总数；`K_PIPE_MAX` 是流水线深度（三缓冲时为 3）。预取循环用 `cp.async` 加载填充 `K_PIPE_MAX - 1` 个流水线槽（如 3 个中的 2 个）。每个 `cp_async_fence()` 创建一个 fence 组，使 `cp_async_wait` 后续可以等待特定组。`__syncthreads()` 确保所有线程在主循环访问 SMEM 之前都已到达此点。

---

### Lines 236–280 — Pipelined Main Loop / 流水线化主循环

```cpp
int k_pipe_read  = 0;
int k_pipe_write = K_PIPE_MAX-1;

CUTE_NO_UNROLL
for (int k_tile = 0; k_tile < K_TILE_MAX; ++k_tile)
{
  int k_tile_next = k_tile + (K_PIPE_MAX-1);
  k_tile_next = (k_tile_next >= K_TILE_MAX) ? K_TILE_MAX-1 : k_tile_next;

  // Copy gmem to smem for k_tile_write
  copy(copy_a, tAgA(_,_,_,k_tile_next), tAsA(_,_,_,k_pipe_write));
  copy(copy_b, tBgB(_,_,_,k_tile_next), tBsB(_,_,_,k_pipe_write));
  cp_async_fence();

  // Advance k_pipe_write
  ++k_pipe_write;
  k_pipe_write = (k_pipe_write == K_PIPE_MAX) ? 0 : k_pipe_write;

  // Wait on all cp.async
  cp_async_wait<0>();

  warpgroup_fence_operand(tCrC);
  warpgroup_arrive();
  cute::gemm(mma, tCrA(_,_,_,k_pipe_read), tCrB(_,_,_,k_pipe_read), tCrC);
  warpgroup_commit_batch();
  warpgroup_wait<0>();
  warpgroup_fence_operand(tCrC);

  // Advance k_pipe_read
  ++k_pipe_read;
  k_pipe_read = (k_pipe_read == K_PIPE_MAX) ? 0 : k_pipe_read;
}
```

**EN**: The core pipelined loop. Two circular counters, `k_pipe_read` and `k_pipe_write`, index into the pipeline SMEM buffer. Each iteration:
1. **Issues the next copy**: fetches `k_tile + (K_PIPE_MAX-1)` ahead (clamped to the last valid tile to avoid OOB).
2. **Fences the copy**: `cp_async_fence()` tags the issued `cp.async` operations as one group.
3. **Waits for all pending copies**: `cp_async_wait<0>()` blocks until **all** inflight `cp.async` groups complete. This is conservative; production code would use `cp_async_wait<K_PIPE_MAX-1>` to overlap more.
4. **Runs WGMMA**: The five-step warpgroup sequence is mandatory:
   - `warpgroup_fence_operand(tCrC)` — establishes a fence on the accumulator registers to prevent the compiler from reordering stores across the WGMMA boundary.
   - `warpgroup_arrive()` — marks the beginning of a WGMMA batch; issues `wgmma.fence`.
   - `cute::gemm(mma, ...)` — emits `wgmma.mma_async` instructions for the entire k-slice. On SM90, 128 threads in the warpgroup cooperate to execute this.
   - `warpgroup_commit_batch()` — issues `wgmma.commit_group`, committing the batch of WGMMAs to the hardware pipeline.
   - `warpgroup_wait<0>()` — issues `wgmma.wait_group 0`, stalling until **all** committed WGMMAs complete.

`CUTE_NO_UNROLL` prevents the compiler from unrolling the loop, which would blow up register pressure.  
**CN**: 核心流水线循环。两个循环计数器 `k_pipe_read` 和 `k_pipe_write` 索引流水线 SMEM 缓冲区。每次迭代：
1. **发起下一次拷贝**：预取 `k_tile + (K_PIPE_MAX-1)` 之前的 tile（限制在最后一个有效 tile 以避免越界）。
2. **Fence 拷贝**：`cp_async_fence()` 将已发起的 `cp.async` 操作标记为一个组。
3. **等待所有待处理拷贝**：`cp_async_wait<0>()` 阻塞直到**所有**飞行中的 `cp.async` 组完成。这是保守的；生产代码会使用 `cp_async_wait<K_PIPE_MAX-1>` 以重叠更多操作。
4. **运行 WGMMA**：五步 warpgroup 序列是强制性的：
   - `warpgroup_fence_operand(tCrC)` — 在累加器寄存器上建立 fence，防止编译器跨 WGMMA 边界重排存储。
   - `warpgroup_arrive()` — 标记 WGMMA 批次的开始；发出 `wgmma.fence`。
   - `cute::gemm(mma, ...)` — 为整个 k 切片发出 `wgmma.mma_async` 指令；在 SM90 上，warpgroup 中的 128 个线程协作执行。
   - `warpgroup_commit_batch()` — 发出 `wgmma.commit_group`，将 WGMMA 批次提交到硬件流水线。
   - `warpgroup_wait<0>()` — 发出 `wgmma.wait_group 0`，阻塞直到**所有**已提交 WGMMA 完成。

`CUTE_NO_UNROLL` 防止编译器展开循环（展开会急剧增加寄存器压力）。

---

### Lines 283–288 — Epilogue / 尾处理

```cpp
axpby(alpha, tCrC, beta, tCgC);
```

**EN**: `axpby` (α·X + β·Y → Y) writes the accumulated result back to global memory: `C = alpha * tCrC + beta * gC`. `tCrC` lives in registers; `tCgC` is the thread's slice of the global C tile. This handles both the scaling and the write-back in one vectorized pass.  
**CN**: `axpby`（α·X + β·Y → Y）将累积结果写回全局内存：`C = alpha * tCrC + beta * gC`。`tCrC` 存储在寄存器中；`tCgC` 是该线程在全局 C tile 中的切片。一次向量化遍历同时处理缩放和回写。

---

### Lines 290–380 — `gemm_nt` Host Function / `gemm_nt` 主机函数

```cpp
template <class TA, class TB, class TC, class Alpha, class Beta>
void gemm_nt(int m, int n, int k, Alpha alpha,
             TA const* A, int ldA, TB const* B, int ldB,
             Beta beta, TC* C, int ldC, cudaStream_t stream = 0)
{
  auto dA = make_stride(Int<1>{}, ldA);   // (dM, dK) — column-major A
  auto dB = make_stride(Int<1>{}, ldB);   // (dN, dK) — column-major B
  auto dC = make_stride(Int<1>{}, ldC);   // (dM, dN) — column-major C

  auto bM = Int<128>{};
  auto bN = Int<128>{};
  auto bK = Int< 64>{};
  auto cta_tiler = make_shape(bM, bN, bK);
  auto bP = Int<3>{};   // Pipeline depth: 3 SMEM buffers

  auto sA = tile_to_shape(GMMA::Layout_MN_SW128_Atom<TA>{}, make_shape(bM,bK,bP));
  auto sB = tile_to_shape(GMMA::Layout_MN_SW128_Atom<TB>{}, make_shape(bN,bK,bP));

  TiledCopy copyA = make_tiled_copy(Copy_Atom<SM80_CP_ASYNC_CACHEALWAYS<uint128_t>, TA>{},
                                    Layout<Shape<_16,_8>>{},
                                    Layout<Shape< _8,_1>>{});
  TiledCopy copyB = make_tiled_copy(Copy_Atom<SM80_CP_ASYNC_CACHEALWAYS<uint128_t>, TB>{},
                                    Layout<Shape<_16,_8>>{},
                                    Layout<Shape< _8,_1>>{});

  TiledMMA tiled_mma = make_tiled_mma(SM90_64x64x16_F16F16F16_SS<GMMA::Major::MN,GMMA::Major::MN>{});
  ...
}
```

**EN**: Sets up the NT (column-major A, column-major B) GEMM. Key decisions:
- **Strides**: `make_stride(Int<1>{}, ldA)` — the first dimension (M) has stride 1, so A is column-major with leading dimension `ldA`.
- **Tile sizes**: 128×128×64 is a standard Hopper tile; BLK_K=64 with FP16 yields 128 bytes per row (32 elements × 4 bytes… wait, FP16 is 2 bytes so 64×2=128 bytes), fitting the 128-byte swizzle atom.
- **SMEM layout**: `GMMA::Layout_MN_SW128_Atom<TA>` is the **MN-major swizzle** atom — data is laid out along the M (or N) dimension first, with a 128-byte swizzle pattern. This is correct for column-major input (NT case).
- **Copy atom**: `SM80_CP_ASYNC_CACHEALWAYS<uint128_t>` issues a 128-bit `cp.async` (16 bytes per thread per instruction), bypassing L1.
- **Thread layout for copy**: `Layout<Shape<_16,_8>>` means 16 threads in M, 8 in K, totaling 128 threads per copy tile.
- **Value layout**: `Layout<Shape<_8,_1>>` means each thread copies 8 elements in M, 1 in K per instruction.
- **MMA**: `SM90_64x64x16_F16F16F16_SS` — a 64×64×16 WGMMA with FP16 inputs and FP16 accumulator; `SS` = both operands read from **S**hared memory. `GMMA::Major::MN` indicates MN-major operand layout.
- **Cluster size**: `dimCluster(1,1,1)` — no cluster, single CTA.  
**CN**: 设置 NT（A 列主序，B 列主序）GEMM。关键决策：
- **步长**：`make_stride(Int<1>{}, ldA)` — 第一维（M）步长为 1，因此 A 是列主序，主维度为 `ldA`。
- **Tile 大小**：128×128×64 是标准 Hopper tile；BLK_K=64 配合 FP16 每行 128 字节，适合 128 字节 swizzle 原子。
- **SMEM 布局**：`GMMA::Layout_MN_SW128_Atom<TA>` 是 **MN 主序 swizzle** 原子——数据首先沿 M（或 N）维度排列，使用 128 字节 swizzle 模式。这对于列主序输入（NT 情形）是正确的。
- **拷贝原子**：`SM80_CP_ASYNC_CACHEALWAYS<uint128_t>` 发出 128 位 `cp.async`（每线程每指令 16 字节），绕过 L1。
- **拷贝线程布局**：`Layout<Shape<_16,_8>>` 表示 M 方向 16 个线程，K 方向 8 个，每个拷贝 tile 共 128 个线程。
- **值布局**：`Layout<Shape<_8,_1>>` 表示每个线程每指令在 M 方向拷贝 8 个元素，K 方向 1 个。
- **MMA**：`SM90_64x64x16_F16F16F16_SS` — FP16 输入、FP16 累加器的 64×64×16 WGMMA；`SS` = 两个操作数均从**共享内存**读取；`GMMA::Major::MN` 表示 MN 主序操作数布局。
- **集群大小**：`dimCluster(1,1,1)` — 无集群，单 CTA。

---

### Lines 350–380 — Kernel Launch (NT) / 内核启动（NT）

```cpp
dim3 dimBlock(size(tiled_mma));
dim3 dimCluster(1, 1, 1);
dim3 dimGrid(round_up(size(ceil_div(m, bM)), dimCluster.x),
             round_up(size(ceil_div(n, bN)), dimCluster.y));
int smemBytes = sizeof(SharedStorage<TA, TB, decltype(sA), decltype(sB)>);

auto* kernel_ptr = &gemm_device<...>;
CUTE_CHECK_ERROR(cudaFuncSetAttribute(kernel_ptr,
                                      cudaFuncAttributeMaxDynamicSharedMemorySize,
                                      smemBytes));

cutlass::ClusterLaunchParams params = {dimGrid, dimBlock, dimCluster, smemBytes};
cutlass::Status status = cutlass::launch_kernel_on_cluster(params, (void const*) kernel_ptr,
                                                           prob_shape, cta_tiler,
                                                           A, dA, sA, copyA,
                                                           B, dB, sB, copyB,
                                                           C, dC, tiled_mma,
                                                           alpha, beta);
```

**EN**: `size(tiled_mma)` gives the total thread count (128 for a single warpgroup). Grid is rounded up to cluster multiples. Because the SMEM usage for triple-buffered tiles exceeds the default 48 KB limit, `cudaFuncSetAttribute` with `cudaFuncAttributeMaxDynamicSharedMemorySize` raises the limit to the requested size (Hopper allows up to 228 KB). `launch_kernel_on_cluster` uses CUDA's cooperative launch API to correctly initialize cluster-level shared memory addressing.  
**CN**: `size(tiled_mma)` 给出总线程数（单 warpgroup 为 128）。Grid 向上舍入到集群倍数。由于三缓冲 tile 的 SMEM 使用超过默认 48 KB 限制，`cudaFuncSetAttribute` 配合 `cudaFuncAttributeMaxDynamicSharedMemorySize` 将限制提高到所需大小（Hopper 最多允许 228 KB）。`launch_kernel_on_cluster` 使用 CUDA 的协作启动 API 正确初始化集群级共享内存寻址。

---

### Lines 382–474 — `gemm_tn` Host Function / `gemm_tn` 主机函数

```cpp
auto dA = make_stride(ldA, Int<1>{});   // (dM, dK) — row-major A
auto dB = make_stride(ldB, Int<1>{});   // (dN, dK) — row-major B

auto sA = tile_to_shape(GMMA::Layout_K_SW128_Atom<TA>{}, make_shape(bM,bK,bP));
auto sB = tile_to_shape(GMMA::Layout_K_SW128_Atom<TB>{}, make_shape(bN,bK,bP));

TiledCopy copyA = make_tiled_copy(Copy_Atom<SM80_CP_ASYNC_CACHEALWAYS<uint128_t>, TA>{},
                                  Layout<Shape<_16,_8>,Stride<_8,_1>>{}, // k-major
                                  Layout<Shape< _1,_8>>{});              // 1 in M, 8 in K

TiledMMA tiled_mma = make_tiled_mma(SM90_64x64x16_F16F16F16_SS<GMMA::Major::K,GMMA::Major::K>{});
```

**EN**: The TN variant (transposed A, normal B — so A is row-major). Key differences from NT:
- **Strides** are swapped: stride 1 is now in K, so A rows are contiguous.
- **SMEM layout**: `GMMA::Layout_K_SW128_Atom` is the **K-major swizzle** atom, suitable for row-major input.
- **Copy thread layout**: `Layout<Shape<_16,_8>,Stride<_8,_1>>` is k-major (stride-1 in K direction), matching row-major access.
- **Value layout**: `Layout<Shape<_1,_8>>` — 1 element in M, 8 in K — loads a short row.
- **MMA**: `GMMA::Major::K` indicates K-major operands.  
**CN**: TN 变体（A 转置，B 正常——所以 A 是行主序）。与 NT 的关键区别：
- **步长**互换：步长 1 现在在 K 方向，所以 A 的行是连续的。
- **SMEM 布局**：`GMMA::Layout_K_SW128_Atom` 是 **K 主序 swizzle** 原子，适合行主序输入。
- **拷贝线程布局**：`Layout<Shape<_16,_8>,Stride<_8,_1>>` 是 K 主序（K 方向步长为 1），与行主序访问匹配。
- **值布局**：`Layout<Shape<_1,_8>>` — M 方向 1 个元素，K 方向 8 个——加载一小段行。
- **MMA**：`GMMA::Major::K` 表示 K 主序操作数。

---

### Lines 476–494 — `gemm` Dispatch Function / `gemm` 调度函数

```cpp
template <class TA, class TB, class TC, class Alpha, class Beta>
void gemm(char transA, char transB, int m, int n, int k, ...)
{
  if (transA == 'N' && transB == 'T') {
    return gemm_nt(m, n, k, alpha, A, ldA, B, ldB, beta, C, ldC, stream);
  } else
  if (transA == 'T' && transB == 'N') {
    return gemm_tn(m, n, k, alpha, A, ldA, B, ldB, beta, C, ldC, stream);
  }
  assert(false && "Not implemented");
}
```

**EN**: A BLAS-style dispatch that routes NT (A not transposed, B transposed) and TN (A transposed, B not transposed) cases to their respective optimized functions. Other combinations are not implemented.  
**CN**: BLAS 风格的调度，将 NT（A 不转置，B 转置）和 TN（A 转置，B 不转置）情形路由到各自的优化函数。其他组合未实现。

---

### Lines 497–611 — `main` Function / `main` 函数

```cpp
int main(int argc, char** argv)
{
  // Device capability check
  if (props.major != 9) { return 0; }

#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
  // Default: m=5120, n=5120, k=4096
  using TA = cute::half_t;
  using TB = cute::half_t;
  using TC = cute::half_t;

  // Initialize host vectors with ±1 values
  for (int j = 0; j < m*k; ++j) h_A[j] = TA(int((rand() % 2) ? 1 : -1));

  // Upload to device, run once for correctness, then 100 iterations for timing
  timer.start();
  for (int i = 0; i < timing_iterations; ++i) { gemm(...); }
  double cute_time = timer.seconds() / timing_iterations;
  printf("CUTE_GEMM:     [%6.1f]GFlop/s  (%6.4f)ms\n", gflops / cute_time, cute_time*1000);
#endif
  return 0;
}
```

**EN**: Guards execution behind a compute-capability check (SM90 only). Default problem size is 5120×5120×4096 in FP16. Matrices are filled with random ±1 values. One warm-up pass ensures kernel compilation and device-side allocation are complete; 100 timed iterations yield a stable throughput estimate. Performance is printed in GFlop/s.  
**CN**: 在计算能力检查（仅 SM90）后执行。默认问题大小为 5120×5120×4096 FP16。矩阵用随机 ±1 值填充。一次预热通道确保内核编译和设备端分配完成；100 次计时迭代产生稳定的吞吐量估计。性能以 GFlop/s 打印。

---

## Key Concepts / 关键概念

- **WGMMA (Warpgroup MMA) / WGMMA（Warpgroup 矩阵乘加）**: SM90-specific asynchronous MMA executed by all 128 threads of a warpgroup together. Uses `wgmma.mma_async` PTX instructions. Both operands (`SS` variant) are read from shared memory descriptors. / SM90 专用异步 MMA，由 warpgroup 的全部 128 个线程共同执行，使用 `wgmma.mma_async` PTX 指令；两个操作数（SS 变体）均通过共享内存描述符读取。

- **Warpgroup fence sequence / Warpgroup fence 序列**: The mandatory `fence_operand → arrive → gemm → commit_batch → wait` sequence synchronizes register state and hardware WGMMA pipelines correctly. / 强制性的 `fence_operand → arrive → gemm → commit_batch → wait` 序列正确同步寄存器状态和硬件 WGMMA 流水线。

- **`cp.async` staging / `cp.async` 分阶段**: `SM80_CP_ASYNC_CACHEALWAYS` issues non-blocking 128-bit loads from global memory to shared memory, allowing overlap with computation. / `SM80_CP_ASYNC_CACHEALWAYS` 发出从全局内存到共享内存的非阻塞 128 位加载，允许与计算重叠。

- **Software pipeline / 软件流水线**: Triple-buffered (PIPE=3) pipeline with circular `k_pipe_read`/`k_pipe_write` counters overlaps GMEM→SMEM copies with WGMMA computation. / 三缓冲（PIPE=3）流水线，通过循环 `k_pipe_read`/`k_pipe_write` 计数器使 GMEM→SMEM 拷贝与 WGMMA 计算重叠。

- **Swizzled SMEM layout / Swizzle SMEM 布局**: `GMMA::Layout_MN_SW128_Atom` and `Layout_K_SW128_Atom` apply a 128-byte XOR swizzle to avoid bank conflicts and produce the descriptor-compatible layout required by WGMMA. / `GMMA::Layout_MN_SW128_Atom` 和 `Layout_K_SW128_Atom` 应用 128 字节 XOR swizzle，避免 bank 冲突并生成 WGMMA 所需的描述符兼容布局。

- **`tile_to_shape` / `tile_to_shape`**: Repeats a base layout atom over a larger shape, creating a fully tiled layout including the pipeline dimension. / 将基础布局原子在更大的形状上重复，创建包含流水线维度的完整分块布局。

- **`as_position_independent_swizzle_tensor` / 位置无关 swizzle 张量**: Required wrapper when using `cp.async` to a swizzled SMEM destination, ensuring correct address computation per thread. / 使用 `cp.async` 向 swizzle SMEM 目标写入时的必要包装，确保每个线程正确计算地址。

- **`CuTe GMMA::Major` / CuTe GMMA::Major 枚举**: Distinguishes MN-major (column-major input) from K-major (row-major input) operand layouts, aligning copy and MMA descriptors. / 区分 MN 主序（列主序输入）和 K 主序（行主序输入）操作数布局，对齐拷贝和 MMA 描述符。

---

## Dependencies / 依赖项

- `cute/tensor.hpp` — entire CuTe tensor/layout/copy/MMA algebra / 整个 CuTe 张量/布局/拷贝/MMA 代数
- `cutlass/cluster_launch.hpp` — `launch_kernel_on_cluster`, `ClusterLaunchParams` for CUDA cluster API / 集群启动参数和 API 封装
- `cutlass/util/print_error.hpp` — `CUTE_CHECK_ERROR`, `CUTE_CHECK_LAST` error macros / 错误检查宏
- `cutlass/util/GPU_Clock.hpp` — `GPU_Clock` high-resolution GPU timer / 高精度 GPU 计时器
- `cutlass/util/helper_cuda.hpp` — CUDA helper utilities / CUDA 辅助工具
- `thrust/host_vector.h`, `thrust/device_vector.h` — host/device memory management / 主机/设备内存管理
- `SM90_64x64x16_F16F16F16_SS` — Hopper 64×64×16 WGMMA MMA atom (both operands from SMEM) / Hopper 64×64×16 WGMMA MMA 原子（两操作数来自 SMEM）
- `SM80_CP_ASYNC_CACHEALWAYS` — 128-bit non-temporal async copy atom / 128 位非时序异步拷贝原子
- `GMMA::Layout_MN_SW128_Atom` / `GMMA::Layout_K_SW128_Atom` — swizzled SMEM layout atoms for WGMMA / 用于 WGMMA 的 swizzle SMEM 布局原子
