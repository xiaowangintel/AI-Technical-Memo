# 04_mma_tma_2sm_sm100.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/cute/tutorial/blackwell/04_mma_tma_2sm_sm100.cu`

**Purpose / 用途**: Demonstrates a Blackwell SM100 GEMM kernel using 2-SM cooperative `tcgen05.mma` instructions with 2SM multicast TMA loads, extending tutorial 03 by introducing cluster-level CTA pairing (peer + leader) for higher compute throughput on a single MMA tile. / 演示在 NVIDIA Blackwell SM100 上使用双 SM 协同 `tcgen05.mma` 指令与 2SM 多播 TMA 加载的 GEMM 内核；通过引入簇级 CTA 配对（对等 CTA + 领导 CTA）来提升单个 MMA tile 的计算吞吐量，是教程 03 的扩展。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1–30 — License Header / 版权许可头

```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 * ...
 **************************************************************************************************/
```

**EN**: Standard NVIDIA BSD-3-Clause license block. Governs redistribution in both source and binary forms.

**CN**: 标准 NVIDIA BSD-3-Clause 许可证块，规定源码和二进制形式的再发行条件。

---

### Lines 32–45 — Tutorial Series Overview / 教程系列概览

```cpp
// Tutorial series:
// * 01_mma_sm100.cu:                Simple GEMM, tcgen05.mma only
// * 02_mma_tma_sm100.cu:            tcgen05.mma + TMA loads
// * 03_mma_tma_multicast_sm100.cu:  tcgen05.mma + Multicast TMA
// * 04_mma_tma_2sm_sm100.cu:        2SM tcgen05.mma + 2SM Multicast TMA   ← THIS FILE
// * 05_mma_tma_epi_sm100.cu:        2SM + TMA epilogue
```

**EN**: Tutorial 04 is the fourth of five stages. The incremental additions from 03→04 are: (1) 2SM `tcgen05.mma` (two CTAs cooperate on one MMA instruction), and (2) `SM100_TMA_2SM_LOAD_MULTICAST` (a single TMA hardware call loads data for both peer CTAs simultaneously). Tutorial 05 adds a TMA-based epilogue on top of tutorial 04.

**CN**: 教程 04 是五阶段中的第四阶段。相较于教程 03 的增量改进：(1) 2SM `tcgen05.mma`（两个 CTA 协同执行一条 MMA 指令），(2) `SM100_TMA_2SM_LOAD_MULTICAST`（单次 TMA 硬件调用同时为两个对等 CTA 加载数据）。教程 05 在教程 04 基础上增加基于 TMA 的尾声（epilogue）。

---

### Lines 47–68 — Include Directives / 头文件引入

```cpp
#include <iostream>
#include <cstdio>
#include <thrust/host_vector.h>
#include <thrust/device_vector.h>
#include <cutlass/half.h>
#include <cutlass/util/print_error.hpp>
#include <cutlass/arch/barrier.h>
#include <cutlass/cluster_launch.hpp>
#include <cute/tensor.hpp>
#include <cute/arch/cluster_sm90.hpp>
#include <cute/numeric/integral_constant.hpp>
#include <cute/algorithm/cooperative_copy.hpp>
#include <cute/arch/tmem_allocator_sm100.hpp>
#include "example_utils.hpp"
```

**EN**: Includes span four categories:
- **Standard I/O**: `<iostream>`, `<cstdio>` for console output.
- **Thrust**: Host/device vector management, avoiding manual `cudaMalloc`/`cudaFree`.
- **CUTLASS**: `half.h` (F16 type), `barrier.h` (SMEM barriers), `cluster_launch.hpp` (cluster-aware launch API), `print_error.hpp` (error checking).
- **CuTe**: `tensor.hpp` (the full CuTe library), `cluster_sm90.hpp` (cluster query helpers reused on SM100), `integral_constant.hpp` (compile-time integers `_1`, `_256`), `cooperative_copy.hpp` (auto-vectorized copy), `tmem_allocator_sm100.hpp` (SM100 Tensor Memory allocator).
- **Tutorial helper**: `example_utils.hpp` provides `initialize_tensor`, `reference_gemm`, and error-reporting utilities.

**CN**: 头文件分四类：
- **标准 I/O**：`<iostream>`、`<cstdio>` 用于控制台输出。
- **Thrust**：主机/设备向量管理，避免手动 `cudaMalloc`/`cudaFree`。
- **CUTLASS**：`half.h`（F16 类型）、`barrier.h`（SMEM 屏障）、`cluster_launch.hpp`（簇感知启动 API）、`print_error.hpp`（错误检查）。
- **CuTe**：`tensor.hpp`（完整 CuTe 库）、`cluster_sm90.hpp`（SM100 上复用的簇查询辅助函数）、`integral_constant.hpp`（编译期整数 `_1`、`_256`）、`cooperative_copy.hpp`（自动向量化拷贝）、`tmem_allocator_sm100.hpp`（SM100 张量内存分配器）。
- **教程辅助**：`example_utils.hpp` 提供 `initialize_tensor`、`reference_gemm` 和误差报告工具。

---

### Line 70 — Namespace / 命名空间

```cpp
using namespace cute;
```

**EN**: Brings all CuTe symbols into scope (`make_tensor`, `local_tile`, `_1`, `Step`, `tma_partition`, etc.) for concise usage throughout the file.

**CN**: 将所有 CuTe 符号引入当前作用域（`make_tensor`、`local_tile`、`_1`、`Step`、`tma_partition` 等），便于在整个文件中简洁使用。

---

### Lines 78–119 — Algorithm & 2SM Design Description / 算法与 2SM 设计说明

```cpp
// D (f32) = beta * C (F32) + alpha * A (F16) * B (F16)
// A: MxK K-major (row-major),  B: NxK K-major (col-major),  C,D: MxN N-major
//
// Key extensions to tutorial 03:
// 1. 2SM tcgen05.mma instructions
// 2. 2SM TMA instructions
// 3. TMA multicast pattern specialized for 2SM
//
// 2SM MMA semantics:
//   - Only 1 of 2 collaborating CTAs (leader CTA) executes the MMA instruction
//   - Both CTAs contribute to SMEM bandwidth
//   - Accumulator lives in TMEM, shared between the two CTAs
//
// TypeA = half_t, TypeB = half_t, TypeC = float, TypeD = float
// MMA: 256x256x16 F16xF16=F32
```

**EN**: This comment block describes the complete algorithm and the three new concepts introduced in tutorial 04:

**1. 2SM `tcgen05.mma`**: The Blackwell architecture can pair two SMs (CTAs within a cluster) to collaboratively execute a single MMA instruction. The two CTAs are called *peer CTAs*; the one that actually issues the `tcgen05.mma` instruction is the *leader CTA*. This doubles the SMEM read bandwidth available to the MMA unit—both CTAs' SMEM banks are accessible—allowing a larger 256×256 tile versus a single CTA's 128×256.

**2. 2SM TMA**: The `SM100_TMA_2SM_LOAD_MULTICAST` instruction atomically loads data into the SMEM of *both* CTAs in a peer pair in one hardware transaction. Both CTAs must issue the copy call, but the byte-count tracked by the barrier accounts for the combined transfer (2×).

**3. Specialized multicast masks**: Multicast masks must now encode the V-dimension (peer-CTA index) in addition to M and N cluster coordinates, because the arrive signal from the MMA barrier must reach all CTAs in the multicast group, including both elements of each peer pair.

**CN**: 此注释块描述完整算法以及教程 04 引入的三个新概念：

**1. 2SM `tcgen05.mma`**：Blackwell 架构可配对两个 SM（簇内 CTA）协同执行单条 MMA 指令。这两个 CTA 称为*对等 CTA*；实际发起 `tcgen05.mma` 指令的称为*领导 CTA*。这将 MMA 单元可用的 SMEM 读取带宽翻倍——两个 CTA 的 SMEM bank 均可访问——相比单 CTA 的 128×256，可实现更大的 256×256 tile。

**2. 2SM TMA**：`SM100_TMA_2SM_LOAD_MULTICAST` 指令在单次硬件事务中原子性地将数据加载到对等 CTA 对中*两个* CTA 的 SMEM。两个 CTA 都必须发起拷贝调用，但屏障跟踪的字节数要计入合并传输量（2×）。

**3. 专用多播掩码**：多播掩码现在除了 M 和 N 簇坐标外，还需编码 V 维度（对等 CTA 索引），因为 MMA 屏障的到达信号必须传达到多播组中的所有 CTA，包括每个对等对的两个元素。

---

### Line 121 — Architecture Guard / 架构保护宏

```cpp
#if defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)
```

**EN**: Compile-time guard. All SM100-specific code (`tcgen05.mma`, TMEM allocator, 2SM TMA) only compiles when targeting SM100 (compute capability 10.0). The `main` function at the end handles the `#else` case with a graceful waiver message.

**CN**: 编译期保护宏。所有 SM100 特定代码（`tcgen05.mma`、TMEM 分配器、2SM TMA）仅在目标架构为 SM100（算力 10.0）时编译。末尾的 `main` 函数在 `#else` 分支中打印友好的跳过提示。

---

### Lines 123–140 — `SharedStorage` Template Struct / `SharedStorage` 模板结构体

```cpp
template <class TypeA, class TypeB, class ASmemLayout, class BSmemLayout>
struct SharedStorage
{
  alignas(128) cute::ArrayEngine<TypeA, cute::cosize_v<ASmemLayout>> A;
  alignas(128) cute::ArrayEngine<TypeB, cute::cosize_v<BSmemLayout>> B;

  alignas(16) cute::uint64_t mma_barrier;   // Tracks MMA completion
  alignas(16) cute::uint64_t tma_barrier;   // Tracks TMA completion
  alignas(16) cute::uint32_t tmem_base_ptr; // TMEM allocation address

  CUTE_DEVICE constexpr auto tensor_sA() { return make_tensor(make_smem_ptr(A.begin()), ASmemLayout{}); }
  CUTE_DEVICE constexpr auto tensor_sB() { return make_tensor(make_smem_ptr(B.begin()), BSmemLayout{}); }
};
```

**EN**: The shared memory layout for a single CTA. Detailed field semantics:

- **`A` / `B`**: Raw element storage whose size is `cosize_v<Layout>` — the total number of storage locations the layout maps to, including any swizzle-induced gaps. `alignas(128)` provides the 128-byte SMEM alignment required by TMA loads.

- **`mma_barrier`** (`uint64_t`): A cluster-scoped arrive/wait barrier. Multiple CTAs `arrive` here; both CTAs in the affected M-row and N-column must arrive before it releases. This prevents the TMA for the next K-tile from overwriting SMEM while the current MMA is still reading it.

- **`tma_barrier`** (`uint64_t`): A transaction-count barrier. The TMA hardware engine acts as a single "thread" that decrements this counter by `tma_transaction_bytes` when the async DMA transfer completes. The leader CTA's `wait_barrier` call blocks until the transfer finishes.

- **`tmem_base_ptr`** (`uint32_t`): The 32-bit hardware address of the TMEM allocation. Stored in SMEM (rather than a register) so that it can be written by warp 0 and read by all other warps after `__syncthreads()`. The peer CTA can also read it via SMEM if needed.

- **`tensor_sA()` / `tensor_sB()`**: Convenience factory functions that bind the raw storage address to the typed swizzled layout, returning a CuTe `Tensor` object that the MMA and TMA subsystems can operate on.

**CN**: 单个 CTA 的共享内存布局。各字段详细语义：

- **`A` / `B`**：原始元素存储，大小为 `cosize_v<Layout>`——布局映射的总存储位置数，包括交织引起的间隙。`alignas(128)` 提供 TMA 加载所需的 128 字节 SMEM 对齐。

- **`mma_barrier`**（`uint64_t`）：簇范围的 arrive/wait 屏障。多个 CTA 在此 `arrive`；受影响的 M 行和 N 列中的所有 CTA 都必须到达后才会释放。这防止下一个 K-tile 的 TMA 在当前 MMA 仍在读取时覆写 SMEM。

- **`tma_barrier`**（`uint64_t`）：事务计数屏障。TMA 硬件引擎作为单一"线程"，在异步 DMA 传输完成时将此计数器减少 `tma_transaction_bytes`。领导 CTA 的 `wait_barrier` 调用阻塞，直到传输完成。

- **`tmem_base_ptr`**（`uint32_t`）：TMEM 分配的 32 位硬件地址。存于 SMEM（而非寄存器），以便 warp 0 写入后所有其他 warp 在 `__syncthreads()` 后可读取。如需要，对等 CTA 也可通过 SMEM 读取。

- **`tensor_sA()` / `tensor_sB()`**：便捷工厂函数，将原始存储地址绑定到类型化交织布局，返回 MMA 和 TMA 子系统可操作的 CuTe `Tensor` 对象。

---

### Lines 142–159 — `gemm_device` Kernel Signature / 设备内核签名

```cpp
template <class SharedStorage,
          class ATensor, class BTensor, class CTensor, class DTensor,
          class MmaTiler_MNK, class TiledMMA, class ClusterShape_MNK,
          class TmaAtomA, class TmaAtomB,
          class Alpha, class Beta>
__global__ static void
gemm_device(ATensor mA, BTensor mB, CTensor mC, DTensor mD,
            MmaTiler_MNK mma_tiler, TiledMMA tiled_mma,
            ClusterShape_MNK cluster_shape,
            CUTE_GRID_CONSTANT TmaAtomA const tma_atom_A,
            CUTE_GRID_CONSTANT TmaAtomB const tma_atom_B,
            Alpha alpha, Beta beta)
```

**EN**: The kernel is fully templated on all algorithm and tensor types, enabling zero-cost abstractions — all shapes, strides, and descriptors are resolved at compile time. `CUTE_GRID_CONSTANT` marks the TMA atoms as *grid-constant* parameters: the CUDA driver broadcasts them from a constant cache rather than re-reading from global memory per thread, reducing memory traffic. Compared to tutorial 03, the signature is unchanged — the 2SM behavior is entirely encoded in the types of `TiledMMA` and `TmaAtomA/B` chosen on the host.

**CN**: 内核对所有算法和张量类型完全模板化，实现零开销抽象——所有形状、步幅和描述符在编译时解析。`CUTE_GRID_CONSTANT` 将 TMA 原子标记为*网格常量*参数：CUDA 驱动从常量缓存广播，而非每线程从全局内存读取，减少内存流量。相较教程 03，签名不变——2SM 行为完全编码在主机端选择的 `TiledMMA` 和 `TmaAtomA/B` 类型中。

---

### Lines 161–171 — Cluster Layout & VMNK Coordinate / 簇布局与 VMNK 坐标

```cpp
// Map cluster shape to (V, M, N, K) CTA layout
Layout cluster_layout_vmnk = tiled_divide(make_layout(cluster_shape),
                                          make_tile(typename TiledMMA::AtomThrID{}));

// Each CTA's position within the cluster
auto mma_coord_vmnk = make_coord(
    blockIdx.x % size<0>(cluster_layout_vmnk), // V: peer CTA index (0=leader, 1=peer)
    blockIdx.x / size<0>(cluster_layout_vmnk), // M: MMA tile M-coordinate
    blockIdx.y,                                // N: MMA tile N-coordinate
    _);                                        // K: all K tiles
```

**EN**: This is the central new concept of tutorial 04. `TiledMMA::AtomThrID` has size 2 for a 2SM MMA atom, so `tiled_divide` introduces a V-mode of size 2 in `cluster_layout_vmnk`. The four-dimensional coordinate `(V, M, N, K)` partitions the cluster's CTAs:

- **V-mode (size 2)**: Which of the two peer CTAs this block is. V=0 is the *leader* (issues MMA, waits on TMA barrier). V=1 is the *peer* (participates in TMA loads but does not issue MMA).
- **M-mode**: Which M-tile of the MMA grid this pair covers.
- **N-mode**: Which N-tile this pair covers (`blockIdx.y`).
- **K-mode**: Underscored `_` means "all K tiles" — the K dimension is iterated in the mainloop, not partitioned across CTAs.

In contrast, single-SM tutorials map `blockIdx.x` directly to the M-tile coordinate.

**CN**: 这是教程 04 的核心新概念。对于 2SM MMA 原子，`TiledMMA::AtomThrID` 大小为 2，因此 `tiled_divide` 在 `cluster_layout_vmnk` 中引入大小为 2 的 V 模式。四维坐标 `(V, M, N, K)` 对簇的 CTA 进行分区：

- **V 模式（大小 2）**：当前块是两个对等 CTA 中的哪一个。V=0 是*领导*（发起 MMA，等待 TMA 屏障）；V=1 是*对等*（参与 TMA 加载但不发起 MMA）。
- **M 模式**：该对覆盖 MMA 网格的哪个 M-tile。
- **N 模式**：该对覆盖哪个 N-tile（`blockIdx.y`）。
- **K 模式**：下划线 `_` 表示"所有 K-tile"——K 维度在主循环中迭代，不跨 CTA 分区。

相比之下，单 SM 教程将 `blockIdx.x` 直接映射到 M-tile 坐标。

---

### Lines 173–195 — GMEM Tile Partitioning / 全局内存分片

```cpp
auto mma_coord = select<1,2,3>(mma_coord_vmnk); // Drop V-mode for GMEM slicing
Tensor gA = local_tile(mA, mma_tiler, mma_coord, Step<_1, X,_1>{}); // (MmaTile_M, MmaTile_K, Tiles_K)
Tensor gB = local_tile(mB, mma_tiler, mma_coord, Step< X,_1,_1>{}); // (MmaTile_N, MmaTile_K, Tiles_K)
Tensor gC = local_tile(mC, mma_tiler, mma_coord, Step<_1,_1, X>{});  // (MmaTile_M, MmaTile_N)
Tensor gD = local_tile(mD, mma_tiler, mma_coord, Step<_1,_1, X>{});  // (MmaTile_M, MmaTile_N)
```

**EN**: `local_tile` extracts the subtensor that this CTA *pair* is responsible for. The V-dimension is dropped (`select<1,2,3>`) because global memory slicing is done at the MMA-tile level, not the individual-CTA level. The `Step<>` projection arguments control which modes of the tiler apply:
- `Step<_1,X,_1>` for A: project along M and K, not N (A has no N dependency).
- `Step<X,_1,_1>` for B: project along N and K, not M.
- `Step<_1,_1,X>` for C/D: project along M and N, not K.

The resulting shapes (with example values): `gA: (_128,_64,4)` — 128-row × 64-col tile, 4 K-tiles deep.

**CN**: `local_tile` 提取该 CTA *对*负责的子张量。V 维度被丢弃（`select<1,2,3>`），因为全局内存切分在 MMA-tile 层面进行，而非单个 CTA 层面。`Step<>` 投影参数控制 tiler 的哪些模式适用：
- A 的 `Step<_1,X,_1>`：沿 M 和 K 投影，不投影 N（A 无 N 依赖）。
- B 的 `Step<X,_1,_1>`：沿 N 和 K 投影，不投影 M。
- C/D 的 `Step<_1,_1,X>`：沿 M 和 N 投影，不投影 K。

结果形状（示例值）：`gA: (_128,_64,4)` — 128 行 × 64 列 tile，K 方向 4 个。

---

### Lines 198–206 — SMEM Tensor Binding / SMEM 张量绑定

```cpp
extern __shared__ char shared_memory[];
SharedStorage& shared_storage = *reinterpret_cast<SharedStorage*>(shared_memory);

Tensor tCsA = shared_storage.tensor_sA(); // Swizzled SMEM tensor for A
Tensor tCsB = shared_storage.tensor_sB(); // Swizzled SMEM tensor for B
```

**EN**: Casts the raw dynamic SMEM region to the `SharedStorage` struct and retrieves CuTe tensor views. The layouts include the `Sw<3,4,3>` swizzle function, which remaps SMEM addresses using XOR to eliminate bank conflicts when the MMA hardware (and TMA) access the tiles. These tensors serve as the staging area between GMEM (via TMA) and the MMA units.

**CN**: 将原始动态 SMEM 区域转换为 `SharedStorage` 结构体并获取 CuTe 张量视图。布局包含 `Sw<3,4,3>` 交织函数，通过 XOR 重映射 SMEM 地址，消除 MMA 硬件（和 TMA）访问 tile 时的 bank 冲突。这些张量充当 GMEM（经 TMA）与 MMA 单元之间的暂存区。

---

### Lines 210–217 — MMA Thread Partitioning / MMA 线程分区

```cpp
auto mma_v = get<0>(mma_coord_vmnk);          // V-index: 0 or 1
ThrMMA cta_mma = tiled_mma.get_slice(mma_v);  // CTA's view of the TiledMMA
Tensor tCgA = cta_mma.partition_A(gA);        // (MmaA, NumMma_M, NumMma_K, Tiles_K)
Tensor tCgB = cta_mma.partition_B(gB);        // (MmaB, NumMma_N, NumMma_K, Tiles_K)
Tensor tCgC = cta_mma.partition_C(gC);        // (MmaC, NumMma_M, NumMma_N)
Tensor tCgD = cta_mma.partition_C(gD);        // (MmaC, NumMma_M, NumMma_N)
```

**EN**: `get_slice(mma_v)` selects this CTA's "slice" of the TiledMMA based on its peer index. For a 2SM MMA with `ThrLayoutVMNK = (_2,_1,_1,_1)`, slice 0 and slice 1 return complementary views of the MMA tile. The subsequent `partition_*` calls map the GMEM global tile to the CTA-local subtile according to the MMA layout. In practice, for the default 2SM configuration both CTAs see the same global coordinates (the 2SM instruction's "thread 0" and "thread 1" refer to the two CTAs, not intra-block threads).

**CN**: `get_slice(mma_v)` 根据对等索引选择该 CTA 在 TiledMMA 中的"切片"。对于 `ThrLayoutVMNK = (_2,_1,_1,_1)` 的 2SM MMA，切片 0 和切片 1 返回 MMA tile 的互补视图。后续的 `partition_*` 调用根据 MMA 布局将全局 GMEM tile 映射到 CTA 本地子 tile。在默认 2SM 配置下，两个 CTA 实际上看到相同的全局坐标（2SM 指令的"线程 0"和"线程 1"指的是两个 CTA，而非块内线程）。

---

### Lines 226–238 — Fragment & Accumulator Tensor Creation / 片段与累加器张量创建

```cpp
// SMEM Descriptor Iterators (not register data)
Tensor tCrA = cta_mma.make_fragment_A(tCsA); // (MmaA, NumMma_M, NumMma_K, Tiles_K)
Tensor tCrB = cta_mma.make_fragment_B(tCsB); // (MmaB, NumMma_N, NumMma_K, Tiles_K)

// TMEM accumulator tensor
Tensor tCtAcc = cta_mma.make_fragment_C(tCgC); // (MmaC, NumMma_M, NumMma_N)
```

**EN**: On SM100, the MMA fragments work fundamentally differently from earlier architectures:

- **`tCrA` / `tCrB`** are *SMEM Descriptor Iterators* (`UMMA::DescriptorIterator`). They do not hold register data. Instead, they hold and advance 64-bit SMEM descriptors that the `tcgen05.mma` instruction uses to address A and B tiles directly in SMEM. Each K-step advances the descriptor by a fixed offset (`_2` words). This eliminates the traditional GMEM→RMEM→MMA data path for A and B.

- **`tCtAcc`** is a *TMEM tensor*. On SM100, the MMA accumulator does not live in registers — it lives in Tensor Memory (TMEM), a new dedicated on-chip scratchpad. `make_fragment_C` creates a tensor with TMEM addressing (`tmem_[32b]`) and the appropriate shape `(MmaC, NumMma_M, NumMma_N)`. Its data pointer is initially unset; it is patched after TMEM allocation.

**CN**: 在 SM100 上，MMA 片段的工作方式与早期架构根本不同：

- **`tCrA` / `tCrB`** 是 *SMEM 描述符迭代器*（`UMMA::DescriptorIterator`）。它们不保存寄存器数据，而是保存并递进 64 位 SMEM 描述符，`tcgen05.mma` 指令用这些描述符直接寻址 SMEM 中的 A 和 B tile。每个 K 步进按固定偏移（`_2` 字）递进描述符。这消除了 A 和 B 的传统 GMEM→RMEM→MMA 数据路径。

- **`tCtAcc`** 是 *TMEM 张量*。在 SM100 上，MMA 累加器不驻留在寄存器中，而是驻留在张量内存（TMEM）——一种新的专用片上暂存器——中。`make_fragment_C` 创建具有 TMEM 寻址（`tmem_[32b]`）和适当形状 `(MmaC, NumMma_M, NumMma_N)` 的张量。其数据指针初始未设置，在 TMEM 分配后进行修补。

---

### Lines 240–250 — TMEM Allocation / TMEM 分配

```cpp
uint32_t elect_one_thr  = cute::elect_one_sync();  // Elect one thread per warp
uint32_t elect_one_warp = (threadIdx.x / 32 == 0); // True for warp 0

using TmemAllocator = cute::TMEM::Allocator2Sm;
TmemAllocator tmem_allocator{};

if (elect_one_warp) {
  tmem_allocator.allocate(TmemAllocator::Sm100TmemCapacityColumns,
                          &shared_storage.tmem_base_ptr);
}
__syncthreads();  // Wait for warp 0 to complete allocation
tCtAcc.data() = shared_storage.tmem_base_ptr; // Patch TMEM tensor pointer
```

**EN**: TMEM allocation details:

- **`Allocator2Sm`** (vs `Allocator1Sm`): Because two CTAs share one TMEM region for the 2SM MMA instruction, the 2SM allocator must perform a *paired* reservation. Internally it executes the `tcgen05.alloc.cg` instruction that allocates TMEM for both CTAs in the cluster group atomically.

- **`Sm100TmemCapacityColumns`**: Allocates the full TMEM capacity (512 columns, each column being 32 banks × 32 bits = 128 bytes). The 256×256 F32 accumulator requires `256 × 256 × 4 bytes = 256 KB`, which fits within the 256KB TMEM per SM pair.

- **Warp-0-only allocation**: Only the first warp calls `allocate`. The resulting base pointer is written to `shared_storage.tmem_base_ptr` (in SMEM) so that all warps can read it after the `__syncthreads()` barrier.

- **`tCtAcc.data() = ...`**: Patches the TMEM tensor's data pointer so that `gemm()` and `copy()` calls know where in TMEM to store/load the accumulator.

**CN**: TMEM 分配细节：

- **`Allocator2Sm`**（vs `Allocator1Sm`）：因为两个 CTA 共享 2SM MMA 指令的同一 TMEM 区域，2SM 分配器必须执行*配对*预留。内部执行 `tcgen05.alloc.cg` 指令，原子性地为簇组中的两个 CTA 分配 TMEM。

- **`Sm100TmemCapacityColumns`**：分配完整 TMEM 容量（512 列，每列 32 bank × 32 位 = 128 字节）。256×256 的 F32 累加器需要 `256 × 256 × 4 字节 = 256 KB`，适合每 SM 对的 256KB TMEM。

- **仅 warp 0 分配**：只有第一个 warp 调用 `allocate`。结果基址写入 `shared_storage.tmem_base_ptr`（在 SMEM 中），以便所有 warp 在 `__syncthreads()` 屏障后可读取。

- **`tCtAcc.data() = ...`**：修补 TMEM 张量的数据指针，使 `gemm()` 和 `copy()` 调用知道在 TMEM 的哪个位置存储/加载累加器。

---

### Lines 278–314 — TMA Partitioning & 2SM Multicast Mask Setup / TMA 分区与 2SM 多播掩码设置

```cpp
// Map flat cluster rank to (V,M,N,K) coordinate
auto cta_in_cluster_coord_vmnk = cluster_layout_vmnk.get_flat_coord(
    int(cute::block_rank_in_cluster()));
auto elect_one_cta = get<0>(cta_in_cluster_coord_vmnk) == Int<0>{}; // True for leader CTA

// TMA partition for A: multicast along N-mode of cluster
auto [tAgA, tAsA] = tma_partition(tma_atom_A,
    get<2>(cta_in_cluster_coord_vmnk),          // This CTA's N-coordinate
    make_layout(size<2>(cluster_layout_vmnk)),  // Total CTAs along N
    group_modes<0,3>(tCsA),                    // Flatten (MmaA,NumMma_M,NumMma_K) → single mode
    group_modes<0,3>(tCgA));

// TMA partition for B: multicast along M-mode of cluster
auto [tBgB, tBsB] = tma_partition(tma_atom_B,
    get<1>(cta_in_cluster_coord_vmnk),
    make_layout(size<1>(cluster_layout_vmnk)),
    group_modes<0,3>(tCsB), group_modes<0,3>(tCgB));

// Multicast masks
uint16_t tma_mcast_mask_a = create_tma_multicast_mask<2>(cluster_layout_vmnk, cta_in_cluster_coord_vmnk);
uint16_t tma_mcast_mask_b = create_tma_multicast_mask<1>(cluster_layout_vmnk, cta_in_cluster_coord_vmnk);
uint16_t mma_mcast_mask_c = create_tma_multicast_mask<0,1>(cluster_layout_vmnk, ...) |
                            create_tma_multicast_mask<0,2>(cluster_layout_vmnk, ...);

// Transaction bytes: doubled for 2SM TMA
int tma_transaction_bytes = size<0>(cluster_layout_vmnk) * sizeof(make_tensor_like(tAsA))
                          + size<0>(cluster_layout_vmnk) * sizeof(make_tensor_like(tBsB));
```

**EN**: This section sets up the cluster-aware TMA infrastructure. Each piece is explained:

**`cta_in_cluster_coord_vmnk`**: Converts the flat CTA-in-cluster rank (0..15 for a 4×4 cluster with V=2) into a 4D coordinate (V,M,N,K). V=0 means leader; V=1 means peer.

**`elect_one_cta`**: True only for the leader CTA (V==0 within each peer pair). The leader handles barrier setup and waiting; the peer CTA participates in TMA loads but does not wait on barriers.

**`tma_partition` for A** (multicast in N-direction): All CTAs with the same M-coordinate (different N positions in the cluster) need the same A tile rows. The partition uses this CTA's N-coordinate as its *local index* within the multicast group. `group_modes<0,3>` collapses the first three tensor modes into one, so the partitioner sees a flat 2D tensor `(SMEM_size, K_tiles)`.

**`tma_partition` for B** (multicast in M-direction): Analogously multicasts B columns to CTAs sharing the same N-coordinate.

**Multicast masks**:
- `tma_mcast_mask_a` (`create_tma_multicast_mask<2>`): Projects along mode-2 (N-mode of cluster), producing a bitmask of all CTAs at the same M-V position but varying N — these CTAs all need the same A tile.
- `tma_mcast_mask_b` (`create_tma_multicast_mask<1>`): Projects along mode-1 (M-mode), producing the bitmask for B's multicast recipients.
- `mma_mcast_mask_c`: The MMA completion barrier must notify all CTAs that share this MMA tile so they know SMEM is free. For 2SM this means both V-indices (V=0 and V=1) at all M and N positions within the multicast groups. The OR of `<0,1>` and `<0,2>` projections covers: all M-CTAs at the same V+N position, plus all N-CTAs at the same V+M position, minus double-counting the current CTA.

**`tma_transaction_bytes`**: Multiplied by `size<0>(cluster_layout_vmnk)` = 2. The `SM100_TMA_2SM_LOAD_MULTICAST` instruction atomically loads data for *both* peer CTAs' SMEM in one hardware request. The TMA barrier must count the total bytes written across both CTAs' SMEM banks to know when the transfer is complete.

**CN**: 此段设置簇感知的 TMA 基础设施。逐项说明：

**`cta_in_cluster_coord_vmnk`**：将簇内 CTA 平坦秩（V=2 的 4×4 簇中为 0..15）转换为 4D 坐标（V,M,N,K）。V=0 为领导，V=1 为对等。

**`elect_one_cta`**：仅对每个对等对中的领导 CTA（V==0）为真。领导处理屏障设置和等待；对等 CTA 参与 TMA 加载但不等待屏障。

**A 的 `tma_partition`**（N 方向多播）：具有相同 M 坐标的所有 CTA（簇中不同 N 位置）需要相同的 A tile 行。分区使用该 CTA 的 N 坐标作为多播组内的*本地索引*。`group_modes<0,3>` 将前三个张量模式折叠为一个，使分区器看到平坦的 2D 张量 `(SMEM_size, K_tiles)`。

**B 的 `tma_partition`**（M 方向多播）：类似地将 B 列多播到共享相同 N 坐标的 CTA。

**多播掩码**：
- `tma_mcast_mask_a`（`create_tma_multicast_mask<2>`）：沿模式 2（簇的 N 模式）投影，生成相同 M-V 位置但不同 N 的所有 CTA 的位掩码——这些 CTA 都需要相同的 A tile。
- `tma_mcast_mask_b`（`create_tma_multicast_mask<1>`）：沿模式 1（M 模式）投影，生成 B 多播接收者的位掩码。
- `mma_mcast_mask_c`：MMA 完成屏障必须通知共享此 MMA tile 的所有 CTA，让它们知道 SMEM 已可复用。对于 2SM，这意味着多播组内所有 M 和 N 位置的两个 V 索引（V=0 和 V=1）。`<0,1>` 和 `<0,2>` 投影的 OR 覆盖：相同 V+N 位置的所有 M-CTA，加上相同 V+M 位置的所有 N-CTA，减去对当前 CTA 的重复计数。

**`tma_transaction_bytes`**：乘以 `size<0>(cluster_layout_vmnk)` = 2。`SM100_TMA_2SM_LOAD_MULTICAST` 指令在单次硬件请求中原子性地为*两个*对等 CTA 的 SMEM 加载数据。TMA 屏障必须计算写入两个 CTA SMEM bank 的总字节数，才能知道传输何时完成。

---

### Lines 317–327 — Barrier Initialization & Cluster Sync / 屏障初始化与簇同步

```cpp
if (elect_one_warp && elect_one_thr) {
  int num_mcast_participants = size<1>(cluster_layout_vmnk) + size<2>(cluster_layout_vmnk) - 1;
  cute::initialize_barrier(shared_storage.mma_barrier, num_mcast_participants);
  cute::initialize_barrier(shared_storage.tma_barrier, /* num_threads */ 1);
}
int mma_barrier_phase_bit = 0;
int tma_barrier_phase_bit = 0;
cute::cluster_sync(); // All CTAs in cluster sync here
```

**EN**: Barrier initialization details:

- **`num_mcast_participants`** for `mma_barrier`: The formula `size<1> + size<2> - 1` counts the number of CTAs that must call `arrive` on this barrier before it releases. For a cluster of shape `(V=2, M=2, N=4)`, `size<1>=2` (M CTAs) and `size<2>=4` (N CTAs) gives 5. This includes: all N=4 CTAs at the same M position (they share B's A-load) UNION all M=2 CTAs at the same N position (they share A's load) — but the CTA at the intersection is only counted once.

- **`tma_barrier`** initialized with 1 "thread": The TMA hardware engine acts as one logical thread that decrements the transaction-count barrier when the DMA completes. No CTA thread calls `arrive` on this barrier explicitly.

- **Phase bits**: Each barrier has a phase bit (`mma_barrier_phase_bit`, `tma_barrier_phase_bit`) initialized to 0. After each `wait_barrier(barrier, phase_bit)` call, the phase bit is toggled (`^= 1`). This implements *reusable barriers* — the same barrier storage is used for every K-tile iteration without reinitialization.

- **`cute::cluster_sync()`**: A hardware cluster-level barrier (not `__syncthreads()`). All CTAs in the cluster must reach this call before any can proceed. This ensures that: (a) barrier initialization by one CTA is visible to all other CTAs, and (b) the TMEM allocation (stored in SMEM) is visible to the peer CTA before the mainloop begins.

**CN**: 屏障初始化细节：

- **`mma_barrier` 的 `num_mcast_participants`**：公式 `size<1> + size<2> - 1` 计算屏障释放前必须调用 `arrive` 的 CTA 数量。对于形状 `(V=2, M=2, N=4)` 的簇，`size<1>=2`（M 个 CTA）和 `size<2>=4`（N 个 CTA）得 5。这包括：相同 M 位置的所有 N=4 个 CTA（它们共享 B 的 A 加载）并集相同 N 位置的所有 M=2 个 CTA（它们共享 A 的加载）——但交叉点的 CTA 只计一次。

- **`tma_barrier`** 以 1 个"线程"初始化：TMA 硬件引擎作为一个逻辑线程，在 DMA 完成时递减事务计数屏障。没有 CTA 线程显式在此屏障上调用 `arrive`。

- **相位位**：每个屏障有一个相位位（`mma_barrier_phase_bit`、`tma_barrier_phase_bit`），初始化为 0。每次 `wait_barrier(barrier, phase_bit)` 调用后，相位位取反（`^= 1`）。这实现了*可复用屏障*——同一屏障存储在每个 K-tile 迭代中无需重新初始化即可复用。

- **`cute::cluster_sync()`**：硬件簇级屏障（非 `__syncthreads()`）。簇中所有 CTA 必须到达此调用后才能继续。这确保：(a) 一个 CTA 的屏障初始化对所有其他 CTA 可见；(b) TMEM 分配（存于 SMEM）在主循环开始前对对等 CTA 可见。

---

### Lines 329–377 — The Mainloop / 主循环

```cpp
tiled_mma.accumulate_ = UMMA::ScaleOut::Zero; // First MMA clears accumulator

for (int k_tile = 0; k_tile < size<3>(tCgA); ++k_tile) {

  // ---- 2a: TMA Load (both CTAs participate) ----
  if (elect_one_warp && elect_one_thr) {
    if (elect_one_cta) { // Leader only sets barrier byte count
      cute::set_barrier_transaction_bytes(shared_storage.tma_barrier, tma_transaction_bytes);
    }
    // Both CTAs issue TMA loads with multicast masks
    copy(tma_atom_A.with(shared_storage.tma_barrier, tma_mcast_mask_a), tAgA(_,k_tile), tAsA);
    copy(tma_atom_B.with(shared_storage.tma_barrier, tma_mcast_mask_b), tBgB(_,k_tile), tBsB);
  }

  // ---- 2b: MMA (leader CTA only) ----
  if (elect_one_cta) {
    cute::wait_barrier(shared_storage.tma_barrier, tma_barrier_phase_bit);
    tma_barrier_phase_bit ^= 1;
    if (elect_one_warp) {
      for (int k_block = 0; k_block < size<2>(tCrA); ++k_block) {
        gemm(tiled_mma, tCrA(_,_,k_block), tCrB(_,_,k_block), tCtAcc);
        tiled_mma.accumulate_ = UMMA::ScaleOut::One; // Accumulate after 1st block
      }
      cutlass::arch::umma_arrive_multicast_2x1SM(&shared_storage.mma_barrier, mma_mcast_mask_c);
    }
  }
  // Both CTAs wait for MMA to complete before reusing SMEM
  cute::wait_barrier(shared_storage.mma_barrier, mma_barrier_phase_bit);
  mma_barrier_phase_bit ^= 1;
}
```

**EN**: The mainloop is the heart of the 2SM execution model. Each K-tile iteration has two synchronization phases:

**Phase 2a — Asynchronous TMA Load**:
- Only one thread per warp (warp 0 of any warp) issues the TMA. Both CTAs (peer and leader) must call `copy()` because `SM100_TMA_2SM_LOAD_MULTICAST` requires both CTAs to participate — the hardware routes the DMA to fill both CTAs' SMEM simultaneously.
- Only the leader CTA (`elect_one_cta`) registers the expected byte count on `tma_barrier`. If both CTAs registered, the barrier count would be doubled and would never complete properly.
- `tma_atom_A.with(barrier, mask)` attaches the TMA atom to the barrier and multicast mask. When the hardware DMA completes, it automatically decrements the barrier by the number of bytes transferred.

**Phase 2b — Synchronous MMA Execution**:
- Only the leader CTA (`elect_one_cta`) waits on `tma_barrier`. This is sufficient because only the leader CTA will issue the MMA instruction.
- `wait_barrier` blocks until `tma_transaction_bytes` have been decremented (i.e., both CTAs' SMEM has been filled).
- Only warp 0 of the leader CTA calls `gemm()`: the `tcgen05.mma` instruction is single-thread-per-warp internally managed by CuTe.
- The inner K-block loop (`size<2>(tCrA)` = 4 iterations) fires 4 MMA instructions, each processing a 256×256×16 sub-problem. The descriptor iterators `tCrA`, `tCrB` advance automatically.
- `umma_arrive_multicast_2x1SM`: After all K-blocks, warp 0 of the leader signals the MMA barrier with `mma_mcast_mask_c`. This notifies both V-CTAs at all participating M and N positions that the MMA is complete and SMEM can be overwritten.
- Both CTAs then `wait_barrier(mma_barrier, ...)`: The peer CTA (which never issued the MMA) also waits here. Both must confirm MMA completion before the next TMA load can overwrite the A/B SMEM tiles.

**CN**: 主循环是 2SM 执行模型的核心。每个 K-tile 迭代有两个同步阶段：

**阶段 2a — 异步 TMA 加载**：
- 每个 warp 只有一个线程（任意 warp 的 warp 0）发起 TMA。两个 CTA（对等和领导）都必须调用 `copy()`，因为 `SM100_TMA_2SM_LOAD_MULTICAST` 需要两个 CTA 共同参与——硬件将 DMA 路由至同时填充两个 CTA 的 SMEM。
- 只有领导 CTA（`elect_one_cta`）在 `tma_barrier` 上注册预期字节数。如果两个 CTA 都注册，屏障计数将加倍，永远无法正确完成。
- `tma_atom_A.with(barrier, mask)` 将 TMA 原子附加到屏障和多播掩码。硬件 DMA 完成时，自动将屏障减少传输字节数。

**阶段 2b — 同步 MMA 执行**：
- 只有领导 CTA（`elect_one_cta`）等待 `tma_barrier`。这已足够，因为只有领导 CTA 会发起 MMA 指令。
- `wait_barrier` 阻塞，直到 `tma_transaction_bytes` 被递减完（即两个 CTA 的 SMEM 都已填充）。
- 只有领导 CTA 的 warp 0 调用 `gemm()`：`tcgen05.mma` 指令在内部由 CuTe 管理为每 warp 单线程。
- 内层 K-block 循环（`size<2>(tCrA)` = 4 次迭代）触发 4 条 MMA 指令，每条处理 256×256×16 子问题。描述符迭代器 `tCrA`、`tCrB` 自动递进。
- `umma_arrive_multicast_2x1SM`：所有 K-block 处理完后，领导 CTA 的 warp 0 以 `mma_mcast_mask_c` 向 MMA 屏障发出信号，通知所有参与 M 和 N 位置的两个 V-CTA MMA 已完成，SMEM 可被覆写。
- 两个 CTA 都调用 `wait_barrier(mma_barrier, ...)`：从未发起 MMA 的对等 CTA 也在此等待。两者都必须确认 MMA 完成，下一个 TMA 加载才能覆写 A/B SMEM tile。

---

### Lines 379–410 — The Epilogue: TMEM → RMEM → GMEM / 尾声：TMEM → RMEM → GMEM

```cpp
// Create TMEM→RMEM tiled copy
TiledCopy tiled_t2r_copy = make_tmem_copy(SM100_TMEM_LOAD_32dp32b1x{}, tCtAcc);
ThrCopy   thr_t2r_copy   = tiled_t2r_copy.get_slice(threadIdx.x);

// Load C from GMEM → RMEM
Tensor tDgC = thr_t2r_copy.partition_D(tCgC);
Tensor tDrC = make_fragment_like(tDgC);
copy(tDgC, tDrC);

// Load accumulator: TMEM → RMEM
Tensor tDtAcc = thr_t2r_copy.partition_S(tCtAcc);
Tensor tDgD   = thr_t2r_copy.partition_D(tCgD);
Tensor tDrAcc = make_tensor<AccType>(shape(tDgD));
copy(tiled_t2r_copy, tDtAcc, tDrAcc);  // tcgen05.ld

// D = alpha * Acc + beta * C (in registers)
axpby(alpha, tDrAcc, beta, tDrC);

// Store D: RMEM → GMEM
copy(tDrC, tDgD);
```

**EN**: Tutorial 04's epilogue is register-centric:

1. **`make_tmem_copy(SM100_TMEM_LOAD_32dp32b1x{}, ...)`**: Creates a tiled-copy atom for the `tcgen05.ld` instruction. `32dp32b1x` = 32 data-paths (threads), 32-bit elements, ×1 element per thread per call. CuTe handles thread-level partitioning of the 256×256 F32 accumulator across 128 threads.

2. **C load GMEM→RMEM**: A simple element-wise copy from global memory to registers. This is a *scalar* load, not TMA — no staging through SMEM. For large tiles this can be bandwidth-limited.

3. **Accumulator load TMEM→RMEM**: `copy(tiled_t2r_copy, tDtAcc, tDrAcc)` translates to one or more `tcgen05.ld` instructions, unloading the TMEM accumulator into registers. Each thread loads its portion as determined by the tiled-copy partitioning.

4. **`axpby(alpha, tDrAcc, beta, tDrC)`**: Computes `tDrC = alpha * tDrAcc + beta * tDrC` in registers. The result is stored back in `tDrC`.

5. **D store RMEM→GMEM**: A scalar store from registers to global memory. Tutorial 05 replaces steps 2 and 5 with TMA SMEM-staged loads/stores for higher bandwidth.

**CN**: 教程 04 的尾声以寄存器为中心：

1. **`make_tmem_copy(SM100_TMEM_LOAD_32dp32b1x{}, ...)`**：为 `tcgen05.ld` 指令创建 tiled-copy 原子。`32dp32b1x` = 32 数据路径（线程）、32 位元素、每线程每次调用 ×1 个元素。CuTe 处理 256×256 F32 累加器在 128 个线程间的线程级分区。

2. **C 加载 GMEM→RMEM**：从全局内存到寄存器的简单逐元素拷贝，为*标量*加载，不经 TMA——无需经 SMEM 暂存。对大 tile 可能受限于带宽。

3. **累加器加载 TMEM→RMEM**：`copy(tiled_t2r_copy, tDtAcc, tDrAcc)` 翻译为一或多条 `tcgen05.ld` 指令，将 TMEM 累加器卸载到寄存器。每个线程按 tiled-copy 分区加载其部分。

4. **`axpby(alpha, tDrAcc, beta, tDrC)`**：在寄存器中计算 `tDrC = alpha * tDrAcc + beta * tDrC`，结果存回 `tDrC`。

5. **D 存储 RMEM→GMEM**：从寄存器到全局内存的标量存储。教程 05 用 TMA SMEM 暂存的加载/存储替换步骤 2 和 5，以获得更高带宽。

---

### Lines 404–410 — TMEM Release / TMEM 释放

```cpp
__syncthreads();
if (elect_one_warp) {
  tmem_allocator.release_allocation_lock();
  tmem_allocator.free(shared_storage.tmem_base_ptr, TmemAllocator::Sm100TmemCapacityColumns);
}
```

**EN**: TMEM must be explicitly released in two steps:
1. **`release_allocation_lock()`**: Releases the hardware scheduling lock so that the next wave of CTAs can be dispatched and begin their own TMEM allocation. This is done *before* `free` to maximize occupancy — the next CTA can start its prologue while this CTA's deallocation is still in progress.
2. **`free(...)`**: Returns the TMEM columns to the allocator pool. The `__syncthreads()` before this block ensures all threads have finished reading TMEM (epilogue complete) before freeing.

**CN**: TMEM 必须分两步显式释放：
1. **`release_allocation_lock()`**：释放硬件调度锁，使下一波 CTA 可被调度并开始自己的 TMEM 分配。此操作在 `free` *之前*执行以最大化占用率——下一个 CTA 可在当前 CTA 的释放仍在进行时开始其序言（prologue）。
2. **`free(...)`**：将 TMEM 列归还给分配器池。此块之前的 `__syncthreads()` 确保所有线程在释放前已完成读取 TMEM（尾声完成）。

---

### Lines 412–606 — `gemm_host_f16xf16_f32_f32_tnt` (Host GEMM Launcher) / 主机端 GEMM 启动器

#### Lines 429–439 — Global Memory Tensor Creation / 全局内存张量创建

```cpp
Tensor mA = make_tensor(make_gmem_ptr(device_ptr_A), layout_A); // (Gemm_M, Gemm_K)
Tensor mB = make_tensor(make_gmem_ptr(device_ptr_B), layout_B); // (Gemm_N, Gemm_K)
Tensor mC = make_tensor(make_gmem_ptr(device_ptr_C), layout_C); // (Gemm_M, Gemm_N)
Tensor mD = make_tensor(make_gmem_ptr(device_ptr_D), layout_D); // (Gemm_M, Gemm_N)
```

**EN**: Wraps raw device pointers in CuTe tensors using the specified layouts. `make_gmem_ptr` creates a global-memory-tagged pointer that CuTe uses to generate correct memory access patterns. These tensors represent the *full* problem matrices in global memory.

**CN**: 使用指定布局将原始设备指针包装为 CuTe 张量。`make_gmem_ptr` 创建全局内存标记指针，CuTe 用其生成正确的内存访问模式。这些张量代表全局内存中的*完整*问题矩阵。

---

#### Lines 450–470 — 2SM TiledMMA Creation / 2SM TiledMMA 创建

```cpp
TiledMMA tiled_mma = make_tiled_mma(
    SM100_MMA_F16BF16_2x1SM_SS<TypeA, TypeB, TypeC, 256, 256,
                                UMMA::Major::K, UMMA::Major::K>{});
// Printed output:
//   ThrLayoutVMNK:  (_2,_1,_1,_1):(_1,_0,_0,_0)  <- V=2 means 2SM
//   Shape_MNK:  (_256,_256,_16)
//   LayoutA_TV: (_2,(_128,_16)):(_128,(_1,_256))
//   LayoutC_TV: (_2,(_128,_256)):(_128,(_1,_256))

auto bM = tile_size<0>(tiled_mma);             // 256
auto bN = tile_size<1>(tiled_mma);             // 256
auto bK = tile_size<2>(tiled_mma) * Int<4>{};  // 16 * 4 = 64
auto mma_tiler = make_shape(bM, bN, bK);       // (256, 256, 64)
```

**EN**: The instruction `SM100_MMA_F16BF16_2x1SM_SS` encodes:
- `2x1SM`: Two SMs cooperate on one MMA. The `ThrLayoutVMNK = (_2,_1,_1,_1)` confirms the V-mode (peer-CTA dimension) is 2.
- `F16BF16`: A is F16, B is BF16-compatible (used here with F16).
- `_SS`: Both operands from SMEM (Shared-Shared).
- `256, 256`: MMA M and N dimensions.
- `Major::K, Major::K`: A and B are K-major in SMEM.

The tile K is 4× the hardware K dimension (16) to amortize TMA overhead — four `tcgen05.mma` instructions are issued per K-tile load.

**CN**: 指令 `SM100_MMA_F16BF16_2x1SM_SS` 编码为：
- `2x1SM`：两个 SM 协同执行一个 MMA。`ThrLayoutVMNK = (_2,_1,_1,_1)` 确认 V 模式（对等 CTA 维度）为 2。
- `F16BF16`：A 为 F16，B 兼容 BF16（此处使用 F16）。
- `_SS`：两个操作数均来自 SMEM（共享-共享）。
- `256, 256`：MMA 的 M 和 N 维度。
- `Major::K, Major::K`：A 和 B 在 SMEM 中为 K 主序。

tile K 是硬件 K 维度（16）的 4 倍，以分摊 TMA 开销——每个 K-tile 加载发起四条 `tcgen05.mma` 指令。

---

#### Lines 499–516 — SMEM Layout Construction with Swizzle / 带交织的 SMEM 布局构造

```cpp
auto mma_shape_A = partition_shape_A(tiled_mma, make_shape(size<0>(mma_tiler), size<2>(mma_tiler)));
// Result: ((_128,_16),_1,_4)  -- (MmaA, NumMma_M, NumMma_K)

auto sA_layout = UMMA::tile_to_mma_shape(UMMA::Layout_K_SW128_Atom<TypeA>{}, mma_shape_A);
// Result: Sw<3,4,3> o smem_ptr[16b](unset) o ((_128,_16),_1,_4):((_64,_1),_0,_16)
```

**EN**: 
- `partition_shape_A` converts the pre-partitioned tile shape `(MmaTile_M, MmaTile_K)` = `(256, 64)` into the post-partitioned MMA fragment shape `(MmaA, NumMma_M, NumMma_K)` = `((_128,_16), 1, 4)`. This reflects how the MMA instruction sees its A operand: 128×16 per hardware sub-operation, tiled 4× in K.
- `tile_to_mma_shape` applies the SM100 K-SW128 swizzle: `Sw<3,4,3>` is a 3-level XOR swizzle with base shift 4 and step 3, creating a 128-byte-wide swizzle that eliminates SMEM bank conflicts for the MMA hardware's access pattern.
- The final stride `((_64,_1), _0, _16)` shows: within a sub-tile, row stride is 64 (elements), column stride is 1; K-tiles are stride-16 apart.

**CN**：
- `partition_shape_A` 将预分区的 tile 形状 `(MmaTile_M, MmaTile_K)` = `(256, 64)` 转换为后分区的 MMA 片段形状 `(MmaA, NumMma_M, NumMma_K)` = `((_128,_16), 1, 4)`。这反映了 MMA 指令如何看待其 A 操作数：每次硬件子操作 128×16，K 方向重复 4 次。
- `tile_to_mma_shape` 应用 SM100 K-SW128 交织：`Sw<3,4,3>` 是 3 级 XOR 交织，基础移位 4，步长 3，创建 128 字节宽的交织，消除 MMA 硬件访问模式的 SMEM bank 冲突。
- 最终步幅 `((_64,_1), _0, _16)` 显示：子 tile 内行步幅为 64（元素），列步幅为 1；K-tile 之间步幅为 16。

---

#### Lines 525–566 — 2SM TMA Atom Creation (Host) / 2SM TMA 原子创建（主机端）

```cpp
auto cluster_shape = make_shape(Int<4>{}, Int<4>{}, Int<1>{}); // 4x4x1 cluster = 16 CTAs
Layout cluster_layout_vmnk = tiled_divide(make_layout(cluster_shape),
    make_tile(typename decltype(tiled_mma)::AtomThrID{}));
// cluster_layout_vmnk shape: (2, 2, 4, 1) -- V=2, M=2, N=4, K=1

Copy_Atom tma_atom_A = make_tma_atom_A_sm100(
    SM100_TMA_2SM_LOAD_MULTICAST{}, mA, sA_layout,
    mma_tiler, tiled_mma, cluster_layout_vmnk);
// tma_atom_A ThrID: _2:_1  <- 2 CTAs per TMA call

Copy_Atom tma_atom_B = make_tma_atom_B_sm100(
    SM100_TMA_2SM_LOAD_MULTICAST{}, mB, sB_layout,
    mma_tiler, tiled_mma, cluster_layout_vmnk);
```

**EN**: The SM100-specific TMA creation functions differ from SM90's `make_tma_atom` in three ways:
1. They require the `TiledMMA` argument (for CTA-level partitioning of the SMEM descriptor).
2. They require `cluster_layout_vmnk` (the full VMNK layout, not just the multicast dimension) so the function can determine: which cluster dimension to multicast along, how many CTAs participate, and how to partition SMEM per CTA.
3. `make_tma_atom_A_sm100` automatically infers that A should be multicast along the N-mode (all N-CTAs at a given M position need the same A rows), and `make_tma_atom_B_sm100` infers B along M-mode.

The `ThrID: _2:_1` output confirms that two CTAs participate in each TMA call.

**CN**: SM100 专用的 TMA 创建函数与 SM90 的 `make_tma_atom` 有三点不同：
1. 需要 `TiledMMA` 参数（用于 SMEM 描述符的 CTA 级分区）。
2. 需要 `cluster_layout_vmnk`（完整 VMNK 布局，而非仅多播维度），使函数可确定：沿哪个簇维度多播，有多少 CTA 参与，以及如何按 CTA 分区 SMEM。
3. `make_tma_atom_A_sm100` 自动推断 A 应沿 N 模式多播（给定 M 位置的所有 N-CTA 需要相同的 A 行），`make_tma_atom_B_sm100` 推断 B 沿 M 模式多播。

`ThrID: _2:_1` 输出确认每次 TMA 调用有两个 CTA 参与。

---

#### Lines 568–606 — Grid Computation & Kernel Launch / 网格计算与内核启动

```cpp
dim3 dimBlock(128);  // 4 warps per CTA
dim3 dimCluster(4, 4, 1);  // 16 CTAs per cluster
dim3 dimGrid(
    size(ceil_div(Gemm_M, bM * size<1>(cluster_layout_vmnk))) * dimCluster.x,
    size(ceil_div(Gemm_N, bN * size<2>(cluster_layout_vmnk))) * dimCluster.y);
// For 512x1024 problem: dimGrid = (4, 4)  (each cluster covers 512 M × 1024 N)

cutlass::ClusterLaunchParams params = {dimGrid, dimBlock, dimCluster, smemBytes};
cutlass::launch_kernel_on_cluster(params, kernel_ptr,
    mA_tma, mB_tma, mC, mD,
    mma_tiler, tiled_mma, cluster_shape,
    tma_atom_A, tma_atom_B, alpha, beta);
```

**EN**: Grid computation:
- `bM * size<1>(cluster_layout_vmnk)` = 256 × 2 = 512: each cluster covers 512 M-rows.
- `bN * size<2>(cluster_layout_vmnk)` = 256 × 4 = 1024: each cluster covers 1024 N-cols.
- For a 512×1024 problem, one cluster covers the entire problem, giving `dimGrid = (4, 4)` (one cluster = 4×4 CTAs, one grid = one cluster).

`launch_kernel_on_cluster` uses CUDA's cooperative cluster launch to ensure all 16 CTAs in each cluster start together and can use cluster-level synchronization. Note that `mC` is passed as a plain GMEM tensor (not TMA-wrapped) since epilogue 04 loads C directly to registers.

**CN**: 网格计算：
- `bM * size<1>(cluster_layout_vmnk)` = 256 × 2 = 512：每个簇覆盖 512 行 M。
- `bN * size<2>(cluster_layout_vmnk)` = 256 × 4 = 1024：每个簇覆盖 1024 列 N。
- 对于 512×1024 问题，一个簇覆盖整个问题，给出 `dimGrid = (4, 4)`（一个簇 = 4×4 个 CTA，一个网格 = 一个簇）。

`launch_kernel_on_cluster` 使用 CUDA 协作簇启动，确保每个簇中的所有 16 个 CTA 一起启动并可使用簇级同步。注意 `mC` 作为普通 GMEM 张量传递（未 TMA 包装），因为教程 04 的尾声将 C 直接加载到寄存器。

---

### Lines 610–738 — `main` Function / main 函数

```cpp
int main(int argc, char** argv) {
  // Verify SM100 GPU
  if ((props.major != 10) || (props.major == 10 && props.minor > 1)) { return -1; }

  // Parse M=512, N=1024, K=256 (with optional CLI overrides)
  int Gemm_M = 512, Gemm_N = 1024, Gemm_K = 256;

  // Allocate and initialize host A (F16), B (F16), C (F32)
  thrust::host_vector<TypeA> host_A(Gemm_M * Gemm_K);
  initialize_tensor(host_tensor_A); // Fill with random values
  thrust::device_vector<TypeA> device_A = host_A;

  // Run GPU GEMM
  gemm_host_f16xf16_f32_f32_tnt(... alpha=1.0, beta=0.0 ...);
  thrust::host_vector<TypeD> host_D = device_D;

  // Run CPU reference GEMM (scalar, for correctness check)
  reference_gemm<TypeAccumulator>(host_tensor_A, host_tensor_B, host_tensor_C,
                                  host_reference_tensor_D, alpha, beta);

  // Compare GPU vs CPU results
  auto relative_error = print_matrix_multiply_mollified_relative_error(...);
  bool success = relative_error <= 0.0;
}
```

**EN**: The `main` function orchestrates the full test:
1. **Device check**: Confirms compute capability 10.x (Blackwell/SM100). Note `props.minor > 1` rejects future SM10x variants.
2. **Data initialization**: Random F16 values for A and B; random F32 for C; alpha=1, beta=0 (pure GEMM, no C scaling).
3. **GPU kernel**: Calls `gemm_host_f16xf16_f32_f32_tnt` which builds descriptors and launches the cluster kernel.
4. **CPU reference**: `reference_gemm` runs a scalar F32 GEMM on the host for correctness comparison.
5. **Error check**: `print_matrix_multiply_mollified_relative_error` computes a mollified relative error (accounting for floating-point rounding) and reports pass/fail.

**CN**: `main` 函数统筹完整测试：
1. **设备检查**：确认算力 10.x（Blackwell/SM100）。注意 `props.minor > 1` 排除未来的 SM10x 变体。
2. **数据初始化**：A 和 B 随机 F16 值；C 随机 F32；alpha=1，beta=0（纯 GEMM，无 C 缩放）。
3. **GPU 内核**：调用 `gemm_host_f16xf16_f32_f32_tnt` 构建描述符并启动簇内核。
4. **CPU 参考**：`reference_gemm` 在主机上运行标量 F32 GEMM 用于正确性比较。
5. **误差检查**：`print_matrix_multiply_mollified_relative_error` 计算柔化相对误差（计入浮点舍入），报告通过/失败。

---

## Key Concepts / 关键概念

- **2SM Cooperative MMA (`_2x1SM`) / 双 SM 协同 MMA**: Two CTAs (peer + leader) collaborate on one `tcgen05.mma` 256×256×16 instruction. Only the leader CTA issues `gemm()`; both contribute to SMEM read bandwidth. The V-dimension in `ThrLayoutVMNK` encodes this pairing. / 两个 CTA（对等 + 领导）协同执行一条 `tcgen05.mma` 256×256×16 指令。只有领导 CTA 发起 `gemm()`；两者共同提供 SMEM 读取带宽。`ThrLayoutVMNK` 中的 V 维度编码此配对。

- **2SM Multicast TMA (`SM100_TMA_2SM_LOAD_MULTICAST`) / 双 SM 多播 TMA**: A single TMA hardware call atomically fills the SMEM of both peer CTAs. Both CTAs call `copy()`, but only the leader registers expected bytes on the barrier. Transaction bytes are doubled relative to 1SM. / 单次 TMA 硬件调用原子性地填充两个对等 CTA 的 SMEM。两个 CTA 都调用 `copy()`，但只有领导在屏障上注册预期字节数。事务字节数相对于 1SM 翻倍。

- **TMEM (Tensor Memory) / 张量内存**: SM100-exclusive on-chip accumulator memory. Allocated via `TMEM::Allocator2Sm` for the paired reservation required by 2SM MMA. Must follow the precise allocate → use → release_lock → free sequence. / SM100 专用片上累加器内存。通过 `TMEM::Allocator2Sm` 进行 2SM MMA 所需的配对预留。必须遵循精确的 allocate → use → release_lock → free 顺序。

- **Phase-bit Reusable Barriers / 相位位可复用屏障**: Barriers are reused across K-tile iterations by toggling a phase bit (`^= 1`), avoiding per-iteration barrier reinitialization overhead. / 通过切换相位位（`^= 1`）在 K-tile 迭代间复用屏障，避免每次迭代的屏障重新初始化开销。

- **Cluster-wide Sync (`cute::cluster_sync()`) / 簇范围同步**: A hardware barrier spanning all CTAs in a cluster. Used once after barrier init and TMEM allocation to ensure all peer CTAs see consistent state before the mainloop. / 跨越簇内所有 CTA 的硬件屏障。在屏障初始化和 TMEM 分配后使用一次，确保所有对等 CTA 在主循环开始前看到一致状态。

- **SMEM Descriptor Fragments / SMEM 描述符片段**: `tCrA`/`tCrB` are `UMMA::DescriptorIterator` objects (not register data). They advance a 64-bit hardware SMEM descriptor through the K-dimension, eliminating the GMEM→RMEM staging for A and B. / `tCrA`/`tCrB` 是 `UMMA::DescriptorIterator` 对象（非寄存器数据）。它们在 K 维度上递进 64 位硬件 SMEM 描述符，消除 A 和 B 的 GMEM→RMEM 暂存。

- **VMNK Coordinate System / VMNK 坐标系**: SM100 introduces a 4D cluster coordinate (V, M, N, K) where V is the peer-CTA index within a 2SM pair. This replaces the 3D (M, N, K) system of SM90. / SM100 引入 4D 簇坐标（V, M, N, K），其中 V 是 2SM 对内的对等 CTA 索引，替代 SM90 的 3D（M, N, K）系统。

- **Multicast Mask Arithmetic / 多播掩码运算**: `create_tma_multicast_mask<modes>()` generates bitmasks over cluster CTA indices projected along specified modes. The MMA barrier mask (`mma_mcast_mask_c`) ORs two projections to cover all CTA pairs sharing the MMA result. / `create_tma_multicast_mask<modes>()` 生成沿指定模式投影的簇 CTA 索引位掩码。MMA 屏障掩码（`mma_mcast_mask_c`）对两个投影做 OR，覆盖共享 MMA 结果的所有 CTA 对。

---

## Dependencies / 依赖项

- `<cute/tensor.hpp>` — Full CuTe tensor, layout, copy, and gemm library / 完整 CuTe 张量、布局、拷贝和 gemm 库
- `<cute/arch/tmem_allocator_sm100.hpp>` — `TMEM::Allocator2Sm`, `Allocator1Sm` for SM100 Tensor Memory / SM100 张量内存的 `TMEM::Allocator2Sm`、`Allocator1Sm`
- `<cute/arch/cluster_sm90.hpp>` — `block_rank_in_cluster()`, cluster query helpers (reused on SM100) / `block_rank_in_cluster()` 和簇查询辅助函数（SM100 上复用）
- `<cute/numeric/integral_constant.hpp>` — Compile-time integers `_1`, `_2`, `_4`, `_128`, `_256`, etc. / 编译期整数 `_1`、`_2`、`_4`、`_128`、`_256` 等
- `<cute/algorithm/cooperative_copy.hpp>` — Auto-vectorized cooperative copy between memory spaces / 内存空间间的自动向量化协同拷贝
- `<cutlass/arch/barrier.h>` — `initialize_barrier()`, `wait_barrier()`, `set_barrier_transaction_bytes()` / `initialize_barrier()`、`wait_barrier()`、`set_barrier_transaction_bytes()`
- `<cutlass/cluster_launch.hpp>` — `ClusterLaunchParams`, `launch_kernel_on_cluster()` for cooperative cluster launch / 协作簇启动的 `ClusterLaunchParams`、`launch_kernel_on_cluster()`
- `<cutlass/half.h>` — `cutlass::half_t` (F16 type used for A and B) / `cutlass::half_t`（A 和 B 使用的 F16 类型）
- `<thrust/host_vector.h>`, `<thrust/device_vector.h>` — Host/device memory allocation and H2D/D2H transfers / 主机/设备内存分配及 H2D/D2H 传输
- `"example_utils.hpp"` — `initialize_tensor()`, `reference_gemm()`, `print_matrix_multiply_mollified_relative_error()`, `create_tma_multicast_mask()` / `initialize_tensor()`、`reference_gemm()`、`print_matrix_multiply_mollified_relative_error()`、`create_tma_multicast_mask()`
