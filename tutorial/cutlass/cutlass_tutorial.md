# CUTLASS Tutorial / CUTLASS 双语教程

> Based on the repository snapshot in `/root/xw/cutlass` (README indicates CUTLASS 4.5.0).  
> 基于 `/root/xw/cutlass` 中的仓库快照编写（README 显示版本为 CUTLASS 4.5.0）。

This tutorial is meant for readers who are new to CUTLASS but want enough depth to understand how modern CUTLASS kernels are assembled, tuned, and extended.  
本教程面向刚接触 CUTLASS 的读者，同时也尽量覆盖足够的深度，帮助你理解现代 CUTLASS 内核如何被组装、调优与扩展。

A practical note: the repository today spans multiple eras of the library. You will see:
- **CUTLASS 2.x** style APIs (`cutlass::gemm::device::Gemm`, threadblock/warp/thread concepts)
- **CUTLASS 3.x** style APIs (CuTe-based **Device → Kernel → Collective** composition)
- **CUTLASS 4.x** additions, especially **Python DSL / CuTeDSL**

一个实践上的说明：当前仓库同时覆盖了多个时代的接口风格。你会看到：
- **CUTLASS 2.x** 风格 API（`cutlass::gemm::device::Gemm`，以及 threadblock/warp/thread 分层）
- **CUTLASS 3.x** 风格 API（基于 CuTe 的 **Device → Kernel → Collective** 组合方式）
- **CUTLASS 4.x** 新增能力，尤其是 **Python DSL / CuTeDSL**

---

## Table of Contents / 目录

1. [Introduction to CUTLASS / CUTLASS 简介](#1-introduction-to-cutlass--cutlass-简介)
2. [Architecture Overview / 架构概览](#2-architecture-overview--架构概览)
3. [Core Concepts / 核心概念](#3-core-concepts--核心概念)
4. [CuTe (CUDA Templates) / CuTe 模板库](#4-cute-cuda-templates--cute-模板库)
5. [GPU Architecture Background / GPU 架构背景](#5-gpu-architecture-background--gpu-架构背景)
6. [CUTLASS 2.x vs 3.x / 版本对比](#6-cutlass-2x-vs-3x--版本对比)
7. [Basic GEMM Tutorial / 基础 GEMM 教程](#7-basic-gemm-tutorial--基础-gemm-教程)
8. [Convolution / 卷积](#8-convolution--卷积)
9. [Advanced Topics / 高级主题](#9-advanced-topics--高级主题)
10. [Hopper Architecture Features / Hopper 架构特性](#10-hopper-architecture-features--hopper-架构特性)
11. [Blackwell Architecture Features / Blackwell 架构特性](#11-blackwell-architecture-features--blackwell-架构特性)
12. [Python Interface (CuTeDSL) / Python 接口](#12-python-interface-cutedsl--python-接口)
13. [Building & Running / 构建与运行](#13-building--running--构建与运行)
14. [References & Resources / 参考资源](#14-references--resources--参考资源)

---

## 1. Introduction to CUTLASS / CUTLASS 简介

### English

**CUTLASS** stands for **CUDA Templates for Linear Algebra Subroutines and Solvers**. In practice, it is a C++ template library for building **high-performance CUDA linear algebra kernels**: GEMM, convolution, attention-related kernels, grouped GEMM, sparse GEMM, block-scaled GEMM, and many fused kernels.

The core idea is simple:

- GPU math kernels are fast when computation, data movement, and synchronization are carefully tiled.
- Those tiling patterns are hard to write by hand for every architecture.
- CUTLASS provides reusable building blocks so you can assemble kernels without rewriting every low-level detail.

The README describes CUTLASS as a collection of abstractions for implementing high-performance GEMM and related computations “at all levels and scales within CUDA,” with support for mixed precision, async copy, Tensor Core instructions, and many data types across Volta, Turing, Ampere, Ada, Hopper, and Blackwell.

CUTLASS is **not** the same thing as cuBLAS or cuDNN:

- **cuBLAS** and **cuDNN** are production libraries with highly optimized, prepackaged kernels and heuristics.
- **CUTLASS** is a kernel construction toolkit and template library.
- If you need a standard GEMM or convolution and want the library to choose for you, cuBLAS/cuDNN are often the easiest choice.
- If you need a custom kernel, a fused epilogue, a nonstandard layout, a research prototype, or architecture-specific tuning, CUTLASS is often the right tool.

A good mental model is:

- **cuBLAS/cuDNN = ready-made library calls**
- **CUTLASS = the library author’s toolbox**

### 中文

**CUTLASS** 的全称是 **CUDA Templates for Linear Algebra Subroutines and Solvers**。实际使用中，它是一个基于 C++ 模板的高性能 CUDA 线性代数内核库，用来构建 **GEMM、卷积、注意力、Grouped GEMM、Sparse GEMM、Block-scaled GEMM** 以及各种融合内核。

它的核心思想很直接：

- GPU 数学内核之所以快，是因为计算、数据搬运和同步都经过了精细的分块（tiling）设计。
- 这些分块策略如果完全手写，会非常复杂，而且不同 GPU 架构往往需要不同实现。
- CUTLASS 把这些模式拆成可复用模块，让你不必为每一代架构都从零写底层内核。

README 中把 CUTLASS 描述为：用于在 CUDA 中“各个层级与尺度”实现高性能 GEMM 及相关计算的一组抽象，支持混合精度、异步拷贝、Tensor Core 指令，以及跨 Volta、Turing、Ampere、Ada、Hopper、Blackwell 等架构的大量数据类型。

CUTLASS **并不等同于** cuBLAS 或 cuDNN：

- **cuBLAS**、**cuDNN** 是面向最终用户的生产级库，内置大量高性能内核和启发式选择机制。
- **CUTLASS** 更像是构造这些内核的模板工具箱。
- 如果你只是想直接调用标准 GEMM/卷积，并让库帮你选实现，cuBLAS/cuDNN 往往更方便。
- 如果你需要自定义 kernel、融合 epilogue、非标准 layout、研究型原型、或者针对新架构深度调优，那么 CUTLASS 更合适。

一个非常有用的理解方式是：

- **cuBLAS/cuDNN = 开箱即用的库调用**
- **CUTLASS = 写高性能库的人所使用的工具箱**

---

## 2. Architecture Overview / 架构概览

### English

CUTLASS 3.x reframes the GEMM stack into a clean hierarchy. The main documentation (`media/docs/cpp/gemm_api_3x.md`) lists these levels:

1. **Device**
2. **Kernel**
3. **Collective**
4. **Tiled MMA and Copy**
5. **Atom**

For learners, it is often easier to read this as:

**Device → Kernel → Collective → Tiled/Atom → thread-level execution**

| Layer | Role | Representative types |
|---|---|---|
| Device | Host-side entry point: argument packing, initialization, launch | `cutlass::gemm::device::GemmUniversalAdapter` |
| Kernel | Grid-level scheduling over tiles or persistent work | `cutlass::gemm::kernel::GemmUniversal` |
| Collective | Cooperative mainloop + epilogue for a tile/cluster | `CollectiveBuilder`, `CollectiveMma`, `DefaultEpilogue` |
| Tiled | Warpgroup/warp/thread decomposition of copy and MMA | `cute::TiledMma`, `cute::TiledCopy` |
| Atom | Smallest hardware operation wrapper | `cute::Mma_Atom`, `cute::Copy_Atom` |

In CUTLASS 2.x, people often learned the hierarchy as **device → threadblock → warp → thread**. That mental model still helps, but CUTLASS 3.x generalizes it so the abstraction better survives hardware evolution (for example, Hopper warp-group instructions and Blackwell features).

#### Template-based design philosophy

CUTLASS is heavily template-driven. That is not accidental; it is the core design philosophy.

- **Types encode structure**: data type, layout, tile shape, instruction shape, schedule.
- **Compile-time specialization matters**: many loops are fully unrolled and many layout checks happen at compile time.
- **Policies are extension points**: schedules, epilogues, tile shapes, and architecture tags are selected by types.
- **Performance is expressed structurally**: you “describe” how computation is tiled and mapped, rather than writing one giant monolithic kernel.

### 中文

CUTLASS 3.x 把 GEMM 栈重新整理成一个更清晰的层次结构。官方文档 `media/docs/cpp/gemm_api_3x.md` 给出的层级是：

1. **Device**
2. **Kernel**
3. **Collective**
4. **Tiled MMA and Copy**
5. **Atom**

对学习者来说，更容易理解成：

**Device → Kernel → Collective → Tiled/Atom → thread 级执行**

| 层级 | 作用 | 代表类型 |
|---|---|---|
| Device | 主机端入口：参数打包、初始化、发射 kernel | `cutlass::gemm::device::GemmUniversalAdapter` |
| Kernel | 网格级别的 tile 调度或 persistent 调度 | `cutlass::gemm::kernel::GemmUniversal` |
| Collective | 针对一个 tile/cluster 的协作式 mainloop 与 epilogue | `CollectiveBuilder`, `CollectiveMma`, `DefaultEpilogue` |
| Tiled | 对 copy 与 MMA 做 warpgroup/warp/thread 层面的分解 | `cute::TiledMma`, `cute::TiledCopy` |
| Atom | 最底层的硬件操作封装 | `cute::Mma_Atom`, `cute::Copy_Atom` |

在 CUTLASS 2.x 里，大家更常见的理解方式是 **device → threadblock → warp → thread**。这个理解今天仍然有效，但 CUTLASS 3.x 把它做了更一般化的抽象，使其能更自然地适配新硬件特性，例如 Hopper 的 warp-group 指令和 Blackwell 的新能力。

#### 基于模板的设计哲学

CUTLASS 大量使用模板，这不是“实现细节”，而是设计核心。

- **类型本身就编码结构**：数据类型、layout、tile shape、instruction shape、schedule 都体现在类型里。
- **编译期特化非常重要**：很多循环会被完全展开，很多 layout 检查会在编译期完成。
- **策略类（policy）就是扩展点**：schedule、epilogue、tile shape、架构标签通常通过类型来选择。
- **性能通过结构表达**：你不是写一个巨大而难维护的 kernel，而是在描述“计算如何被分块、如何映射到硬件上”。

### A concrete 3.x composition example / 一个具体的 3.x 组合示例

**Source / 源码:** `media/docs/cpp/gemm_api_3x.md`

```c++
using CollectiveMainloop = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    ElementA, LayoutA, AlignmentA,
    ElementB, LayoutB, AlignmentB,
    ElementAccumulator,
    TilesShape, ClusterShape,
    cutlass::gemm::collective::StageCountAuto,
    cutlass::gemm::collective::KernelScheduleAuto
  >::CollectiveOp;

using CollectiveEpilogue = cutlass::epilogue::collective::DefaultEpilogue<
    ElementC,
    cutlass::gemm::TagToStrideC_t<LayoutC>,
    cutlass::gemm::TagToStrideC_t<LayoutC>,
    cutlass::epilogue::thread::LinearCombination<ElementC, 1, ElementAccumulator, ElementAccumulator>>;

using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
    cute::Shape<int,int,int,int>,
    CollectiveMainloop,
    CollectiveEpilogue
>;

using GemmHandle = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;
```

**English:** This is the essence of CUTLASS 3.x: build a mainloop, build an epilogue, compose them into a kernel, and wrap the kernel in a device-level launcher.  
**中文：** 这段代码体现了 CUTLASS 3.x 的核心思想：先构造 mainloop，再构造 epilogue，把两者组合成 kernel，最后再包装成 host-side 的 device launcher。

---

## 3. Core Concepts / 核心概念

## 3.1 Layouts / 布局

### English

A **layout** maps logical coordinates to physical memory offsets. In older CUTLASS APIs you often see tags like:

- `cutlass::layout::RowMajor`
- `cutlass::layout::ColumnMajor`
- `cutlass::layout::TensorNHWC`
- `cutlass::layout::NCxHWx`

In CuTe/CUTLASS 3.x, the unifying abstraction is a single vocabulary type:

- `cute::Layout<Shape, Stride>`

This is one of the most important conceptual changes in the library.

**Source / 源码:** `media/docs/cpp/layout.md`

```c++
layout::ColumnMajor layout(lda);
int offset = layout({row, column});     // row + lda * column

layout::RowMajor layout_rm(lda);
int offset_rm = layout_rm({row, column}); // lda * row + column
```

**Source / 源码:** `media/docs/cpp/cute/01_layout.md`

```c++
Layout s2xd4_row = make_layout(make_shape(Int<2>{},4),
                               LayoutRight{});
```

Layouts are not just “row-major or column-major.” In CuTe, layouts can be hierarchical, composed, tiled, divided, and transformed algebraically.

### 中文

**Layout（布局）** 的作用是把逻辑坐标映射成实际内存偏移。老版本 CUTLASS API 中常见的布局标签包括：

- `cutlass::layout::RowMajor`
- `cutlass::layout::ColumnMajor`
- `cutlass::layout::TensorNHWC`
- `cutlass::layout::NCxHWx`

在 CuTe / CUTLASS 3.x 里，这些概念被统一到一个核心抽象：

- `cute::Layout<Shape, Stride>`

这是 CUTLASS 3.x 最重要的概念变化之一。

Layout 不只是“行主序/列主序”这么简单。在 CuTe 中，layout 可以是层次化的（hierarchical）、可组合的（composition）、可分块的（tiling/divide），并且可以做严格的代数变换。

## 3.2 Tiles and tiling strategies / Tile 与分块策略

### English

**Tiling** is the central performance idea of CUTLASS.

A GEMM is not executed as one flat triple loop. It is split into:

- **CTA / threadblock tiles**
- **warp or warpgroup tiles**
- **instruction tiles**
- **thread-level fragments**

CUTLASS documents this hierarchy explicitly.

**Source / 源码:** adapted from `media/docs/cpp/efficient_gemm.md`

```c++
for (int cta_k = 0; cta_k < GemmK; cta_k += CtaTileK) {
  for (int warp_n = 0; warp_n < CtaTileN; warp_n += WarpTileN) {
    for (int warp_m = 0; warp_m < CtaTileM; warp_m += WarpTileM) {
      for (int mma_k = 0; mma_k < WarpTileK; mma_k += MmaK) {
        mma_instruction(d, a, b, c);
      }
    }
  }
}
```

Important tiling-related ideas in CUTLASS include:

- **CTA tile shape**: balances reuse vs occupancy
- **warp tile shape**: balances instruction throughput vs register pressure
- **instruction shape**: matches Tensor Core or SIMT instructions
- **cluster shape**: especially important for Hopper/Blackwell kernels
- **Split-K / Sliced-K / Stream-K**: alternative decompositions for better load balance

### 中文

**分块（tiling）** 是 CUTLASS 性能设计的核心。

GEMM 并不是一个平铺的三重循环，而是被拆分成多个层次：

- **CTA / threadblock tile**
- **warp 或 warpgroup tile**
- **instruction tile**
- **thread 级 fragment**

CUTLASS 的文档非常明确地把这种层次写了出来。

在 CUTLASS 中，和 tiling 直接相关的重要概念包括：

- **CTA tile shape**：在数据复用与 occupancy 之间平衡
- **warp tile shape**：在指令吞吐与寄存器压力之间平衡
- **instruction shape**：匹配 Tensor Core 或 SIMT 指令粒度
- **cluster shape**：对 Hopper / Blackwell 尤其重要
- **Split-K / Sliced-K / Stream-K**：用于改善负载均衡的不同分解策略

## 3.3 Epilogues / Epilogue

### English

The **epilogue** is the phase after accumulation. It writes the output tile back efficiently and often performs fused elementwise work such as:

- `alpha * acc + beta * C`
- bias addition
- clamp / activation
- ReLU / GELU / top-k / softmax-related fusion
- auxiliary outputs such as `amax`

A classic epilogue operator looks like this.

**Source / 源码:** `examples/47_ampere_gemm_universal_streamk/ampere_gemm_universal_streamk.cu`

```c++
using EpilogueOp = cutlass::epilogue::thread::LinearCombination<
    ElementC,
    AlignmentC,
    ElementAccumulator,
    ElementAccumulator>;
```

A more advanced fused epilogue from Hopper FP8 blockwise scaling looks like this.

**Source / 源码:** `examples/67_hopper_fp8_warp_specialized_gemm_with_blockwise_scaling/67_hopper_fp8_warp_specialized_gemm_with_blockwise_scaling.cu`

```c++
using FusionOperation = cutlass::epilogue::fusion::ScaledLinCombPerRowBiasEltActAmaxAux<
    LayoutAux, cutlass::epilogue::thread::ReLU,
    ElementD, ElementCompute, ElementAux,
    ElementAmax, ElementBias, ElementC>;
```

### 中文

**Epilogue** 是累计完成之后的阶段。它不仅负责把结果高效写回内存，还常常顺带完成融合的逐元素计算，例如：

- `alpha * acc + beta * C`
- bias 加法
- clamp / activation
- ReLU / GELU / top-k / softmax 相关融合
- `amax` 之类的辅助输出

最经典的 epilogue 是 `LinearCombination`。而在更高级的 Hopper FP8 示例里，epilogue 甚至会融合 bias、激活函数、辅助输出等多种逻辑。

## 3.4 Data types / 数据类型

### English

CUTLASS supports a wide range of numeric types. The `fundamental_types.md` documentation lists many of them, including:

- `half_t` (FP16)
- `bfloat16_t` (BF16)
- `tfloat32_t` (TF32)
- `float_e4m3_t`, `float_e5m2_t` (FP8)
- `float_e3m2_t`, `float_e2m3_t` (FP6)
- `float_e2m1_t` (FP4)
- `int4_t`, `uint4_t`, `int8_t`
- block-scaled types like `mx_float8_t`, `mx_float6_t`, `mx_float4_t`, `nv_float4_t`

A practical rule of thumb:

- **FP16/BF16**: common training/inference workhorses
- **TF32**: Ampere+ compromise for FP32-like workflows with Tensor Cores
- **FP8 / block-scaled formats**: modern high-throughput ML kernels
- **INT8 / INT4**: inference and quantized workloads
- **Sparse and block-scaled types**: architecture-specific acceleration paths

### 中文

CUTLASS 支持非常广泛的数据类型。`fundamental_types.md` 中列出了很多，包括：

- `half_t`（FP16）
- `bfloat16_t`（BF16）
- `tfloat32_t`（TF32）
- `float_e4m3_t`、`float_e5m2_t`（FP8）
- `float_e3m2_t`、`float_e2m3_t`（FP6）
- `float_e2m1_t`（FP4）
- `int4_t`、`uint4_t`、`int8_t`
- `mx_float8_t`、`mx_float6_t`、`mx_float4_t`、`nv_float4_t` 等 block-scaled 类型

一个实用的理解方式是：

- **FP16/BF16**：训练和推理中最常见的高性能类型
- **TF32**：Ampere 之后用于兼顾 FP32 工作流与 Tensor Core 吞吐的折中方案
- **FP8 / block-scaled**：现代高吞吐机器学习内核的重要格式
- **INT8 / INT4**：量化推理常用
- **Sparse / block-scaled**：强依赖具体 GPU 架构的新型加速路径

---

## 4. CuTe (CUDA Templates) / CuTe 模板库

### English

**CuTe** is the core abstraction library introduced with CUTLASS 3.x. The README and `media/docs/cpp/cute/00_quickstart.md` describe it as a library for defining and operating on **hierarchically multidimensional layouts of threads and data**.

CuTe gives CUTLASS a compact vocabulary:

- **Layout**: how coordinates map to indices
- **Tensor**: data pointer/iterator + layout
- **TiledCopy**: how data moves cooperatively
- **TiledMma**: how MMA instructions are tiled across threads
- **Atoms**: hardware-level copy/MMA units

### 中文

**CuTe** 是 CUTLASS 3.x 引入的核心抽象库。README 和 `media/docs/cpp/cute/00_quickstart.md` 都把它描述为：用于定义和操作 **线程与数据的层次化多维布局** 的 C++ CUDA 模板库。

CuTe 为 CUTLASS 提供了一套非常紧凑的“词汇表”：

- **Layout**：坐标如何映射到索引
- **Tensor**：数据指针/迭代器 + layout
- **TiledCopy**：数据如何协作搬运
- **TiledMma**：MMA 指令如何在多个线程间铺开
- **Atom**：最底层的硬件 copy/MMA 单元

### Layout algebra / 布局代数

**Source / 源码:** `media/docs/cpp/cute/02_layout_algebra.md`

CuTe supports operations such as:

- `composition`
- `coalesce`
- `logical_divide`
- `tiled_divide`
- `flat_divide`

These matter because GPU kernels constantly need to answer questions like:

- How do I map a tile of data to a tile of threads?
- How do I split a tensor into CTA tiles, warp tiles, and MMA tiles?
- How do I express that transformation generically, not just for one hardcoded shape?

这些操作之所以重要，是因为 GPU kernel 一直在回答下面这类问题：

- 如何把一个数据 tile 映射给一组线程？
- 如何把 tensor 拆成 CTA tile、warp tile、MMA tile？
- 如何把这种映射写成通用表达，而不是只支持一个写死的 shape？

### Tensor abstraction / Tensor 抽象

**Source / 源码:** `media/docs/cpp/cute/03_tensor.md`

```c++
Tensor gmem_8sx16d = make_tensor(make_gmem_ptr(A), make_shape(Int<8>{},16));
Layout smem_layout = make_layout(make_shape(Int<4>{},Int<8>{}));
Tensor smem_4x8_col = make_tensor(make_smem_ptr(smem), smem_layout);
```

**English:** A `Tensor` is not just “an array.” It is a view that carries enough structural information so that CuTe algorithms can choose optimized implementations.  
**中文：** `Tensor` 并不只是“数组”。它是带有足够结构信息的视图，因此 CuTe 的算法能够据此选择更优的实现路径。

### Generic tensor algorithms / 通用 Tensor 算法

**Source / 源码:** `media/docs/cpp/cute/04_algorithms.md`

CuTe provides generic algorithms like:

- `copy`
- `copy_if`
- `gemm`
- `fill`
- `clear`
- `axpby`

These algorithms dispatch based on the tensor types and memory spaces. For example, a copy from global memory to shared memory might be lowered to `cp.async` or TMA, depending on architecture and tensor descriptors.

CuTe 提供了 `copy`、`copy_if`、`gemm`、`fill`、`clear`、`axpby` 等通用算法。这些算法会根据 tensor 的类型与内存空间自动做分派。例如，从 global memory 到 shared memory 的 copy，在不同架构上可能被降低成 `cp.async` 或 TMA。

### TMA tensors / TMA Tensor

The CuTe TMA tutorial (`0z_tma_tensors.md`) explains an advanced but important idea: TMA instructions use **descriptors and coordinates**, not ordinary global-memory pointers. CuTe therefore models TMA access with specialized tensor machinery so that TMA coordinates can still be tiled, partitioned, and sliced just like ordinary tensors.

CuTe 的 TMA 教程还解释了一个非常关键的高级概念：TMA 指令使用的是 **descriptor + 坐标**，而不是普通的全局内存指针。因此 CuTe 需要用特殊的 tensor 表达方式来描述 TMA 访问，这样这些坐标仍然可以像普通 tensor 一样被切片、分块和重排。

---

## 5. GPU Architecture Background / GPU 架构背景

### English

To understand CUTLASS well, you need a lightweight mental model of NVIDIA GPUs.

### 5.1 Execution hierarchy / 执行层级

A useful hierarchy is:

- **Grid**
- **Cluster** (important for Hopper/Blackwell)
- **CTA / thread block**
- **Warpgroup** (important for Hopper WGMMA and some modern kernels)
- **Warp** (32 threads)
- **Thread**

CUTLASS maps algorithmic tiles onto this hierarchy.

CUTLASS 会把算法中的 tile 映射到这个硬件层级上。

### 5.2 SMs, warps, Tensor Cores / SM、warp 与 Tensor Core

- An **SM** (Streaming Multiprocessor) executes thread blocks.
- A **warp** is the basic scheduling unit of 32 threads.
- Modern Tensor Core instructions operate at warp or warpgroup granularity, not as isolated scalar operations.

- **SM（Streaming Multiprocessor）** 是执行 thread block 的核心硬件单元。
- **warp** 是由 32 个线程组成的基本调度单元。
- 现代 Tensor Core 指令通常以 warp 或 warpgroup 为粒度工作，而不是标量级操作。

### 5.3 Tensor Core evolution across generations / Tensor Core 的代际演化

| Generation | CUTLASS-relevant highlights | CUTLASS examples |
|---|---|---|
| Volta | First Tensor Cores; early WMMA / tensor-op GEMM | `examples/07_volta_tensorop_gemm` |
| Turing | Expanded integer tensor ops, better inference-oriented support | `examples/08_turing_tensorop_gemm`, `09_turing_tensorop_conv2dfprop` |
| Ampere | TF32, BF16, `cp.async`, sparse tensor ops, wider mixed precision | `examples/14_ampere_tf32_tensorop_gemm`, `15_ampere_sparse_tensorop_gemm` |
| Hopper | TMA, GMMA/WGMMA, warp specialization, FP8, clusters | `examples/48_hopper_warp_specialized_gemm`, `54_hopper_fp8_warp_specialized_gemm` |
| Blackwell | `tcgen05` MMA, TMEM, CLC scheduler, 2-SM collaboration, FP4/NVFP4/MX formats | `examples/70_blackwell_gemm`, `72_blackwell_narrow_precision_gemm`, `92_blackwell_moe_gemm` |

**English:** The exact instruction set changes, but the CUTLASS pattern stays remarkably stable: copy tiles efficiently, stage them through the memory hierarchy, issue tiled MMA instructions, then fuse output work in the epilogue.  
**中文：** 虽然每一代硬件指令都在变化，但 CUTLASS 的核心模式相当稳定：高效搬运 tile，经由不同层级内存做 staging，发射分块 MMA 指令，最后在 epilogue 中融合输出处理。

### 5.4 Memory hierarchy / 内存层级

CUTLASS performance is fundamentally about matching computation to the memory hierarchy:

- **Registers**: fastest, smallest, private to each thread
- **Shared memory**: on-chip scratchpad shared by a CTA (and coordinated carefully)
- **L2 cache**: chip-wide cache that can help tile reuse across CTAs
- **Global memory**: large but high latency
- **TMEM** (Blackwell): a new tensor-memory space used by Blackwell MMA paths

CUTLASS 的性能本质上是在让计算模式适配 GPU 的内存层级：

- **寄存器（registers）**：最快、最小、线程私有
- **共享内存（shared memory）**：片上缓存区，由 CTA 协作使用
- **L2 cache**：芯片级缓存，可提升 CTA 之间的 tile 复用
- **全局内存（global memory）**：容量大但延迟高
- **TMEM（Blackwell）**：Blackwell 引入的新型 tensor memory，用于部分 MMA 路径

### 5.5 TMA (Tensor Memory Accelerator) / TMA

The CuTe TMA documentation explains TMA very clearly:

- TMA was introduced on **Hopper**.
- One TMA instruction can transfer a multidimensional tile between global and shared memory.
- TMA uses a **descriptor** created on the host.
- Inside the kernel, threads issue the TMA operation using descriptor + coordinates.

This is a big deal because TMA reduces the per-element address generation overhead and makes large multidimensional copies more efficient.

CuTe 的 TMA 文档解释得非常清楚：

- TMA 首先出现在 **Hopper**。
- 一条 TMA 指令就能在 global memory 和 shared memory 之间搬运一个多维 tile。
- TMA 依赖主机端预先创建的 **descriptor**。
- kernel 中真正发起 TMA 时，用的是 descriptor 加坐标，而不是普通指针。

这很重要，因为它显著减少了逐元素地址生成的开销，让大规模多维搬运更高效。

---

## 6. CUTLASS 2.x vs 3.x / 版本对比

### English

CUTLASS 3.x is not “just a newer syntax.” It is a structural redesign.

| Topic | CUTLASS 2.x | CUTLASS 3.x |
|---|---|---|
| Mental model | device → threadblock → warp → thread | device → kernel → collective → tiled/atom |
| Core abstraction | many named iterators / maps | CuTe `Layout` + `Tensor` |
| Kernel composition | more architecture-specific named types | builder-style composition via collectives |
| Layout vocabulary | row/column tags dominate | generalized layouts via CuTe |
| Extension points | threadblock/warp internals | collectives, tiled MMA/copy, dispatch policies |
| Hardware evolution | harder to fit new concepts | better fit for Hopper/Blackwell features |

The `cutlass_3x_design.md` document gives the high-level reason for CuTe: CUTLASS 2.x had many bespoke iterators and implicit thread-to-data mappings. CUTLASS 3.x replaces those with a single formal layout/tensor vocabulary, making kernels more composable and more robust to new hardware.

The compatibility story is important too:

- `cutlass::gemm::device::GemmUniversalAdapter` is the common device entry point.
- `cutlass::gemm::kernel::GemmUniversal` can bridge 2.x and 3.x styles.
- You can migrate gradually instead of rewriting everything at once.

### 中文

CUTLASS 3.x 不只是“换了一个新语法”，而是一次架构层面的重构。

| 主题 | CUTLASS 2.x | CUTLASS 3.x |
|---|---|---|
| 心智模型 | device → threadblock → warp → thread | device → kernel → collective → tiled/atom |
| 核心抽象 | 大量命名 iterator / thread map | CuTe `Layout` + `Tensor` |
| kernel 组合方式 | 更依赖架构特定的命名类型 | 通过 collective builder 进行组合 |
| layout 词汇 | 主要靠 row/column 标签 | 通过 CuTe 的通用 layout 表达 |
| 扩展点 | threadblock/warp 内部细节 | collectives、tiled MMA/copy、dispatch policies |
| 适应新硬件 | 新概念较难塞入原有层级 | 更适合 Hopper/Blackwell 这类新特性 |

`cutlass_3x_design.md` 给出的根本原因是：CUTLASS 2.x 中存在很多专用 iterator，并且线程到数据的映射往往隐含在实现细节里；CUTLASS 3.x 用 CuTe 的统一 layout/tensor 词汇替代这些概念，使 kernel 更容易组合，也更能适应新硬件。

同时，兼容性也考虑得很好：

- `cutlass::gemm::device::GemmUniversalAdapter` 是统一的 device 层入口。
- `cutlass::gemm::kernel::GemmUniversal` 可以兼容 2.x 与 3.x 风格。
- 因此迁移通常可以渐进进行，而不需要一次性重写。

### Migration path / 迁移路径建议

### English

A practical migration path from 2.x to 3.x is:

1. Keep using a familiar **device-level GEMM handle**.
2. Learn the 3.x **CollectiveBuilder** pattern.
3. Replace bespoke threadblock internals with **collective mainloop + collective epilogue**.
4. Learn CuTe layouts/tensors only as deeply as needed for your customization level.
5. Move architecture-specific tuning into tile shapes, schedules, and epilogue policies.

### 中文

从 2.x 迁移到 3.x，一个比较务实的路径是：

1. 先保留熟悉的 **device-level GEMM handle** 用法。
2. 学会 3.x 的 **CollectiveBuilder** 模式。
3. 把原来定制的 threadblock 内部实现，替换为 **collective mainloop + collective epilogue**。
4. 根据需要逐步学习 CuTe 的 layout/tensor，而不是一开始就把全部细节学完。
5. 把架构相关调优收敛到 tile shape、schedule 和 epilogue policy 上。

---

## 7. Basic GEMM Tutorial / 基础 GEMM 教程

This section walks through a simple GEMM first in a beginner-friendly 2.x style, then connects it to the 3.x mental model.  
本节先用适合初学者的 2.x 风格讲一个简单 GEMM，再把它连接到 3.x 的思维方式上。

## 7.1 The simplest host-side GEMM / 最简单的 host 端 GEMM

**Source / 源码:** `examples/00_basic_gemm/basic_gemm.cu`

```c++
using ColumnMajor = cutlass::layout::ColumnMajor;

using CutlassGemm = cutlass::gemm::device::Gemm<
    float,        // A element type
    ColumnMajor,  // A layout
    float,        // B element type
    ColumnMajor,  // B layout
    float,        // C/D element type
    ColumnMajor   // C/D layout
>;

CutlassGemm gemm_operator;

CutlassGemm::Arguments args(
    {M, N, K},
    {A, lda},
    {B, ldb},
    {C, ldc},
    {C, ldc},
    {alpha, beta});

cutlass::Status status = gemm_operator(args);
```

### English explanation

This is the beginner entry point:

- `Gemm<...>` picks a default implementation for the chosen data types and layouts.
- `Arguments` packages the problem size, matrix references, and epilogue scalars.
- `gemm_operator(args)` launches the kernel.

The meaning of the runtime parameters is the standard BLAS GEMM meaning:

- `M`: rows of output `D`
- `N`: columns of output `D`
- `K`: reduction dimension
- `lda`, `ldb`, `ldc`: leading dimensions / strides
- `alpha`, `beta`: epilogue scalars in `D = alpha * A*B + beta * C`

### 中文解释

这就是最适合入门的 CUTLASS GEMM 入口：

- `Gemm<...>` 会根据你选择的数据类型和 layout，匹配一个默认实现。
- `Arguments` 把问题尺寸、矩阵引用和 epilogue 标量封装起来。
- `gemm_operator(args)` 负责真正发射 kernel。

这些运行时参数的含义与标准 BLAS GEMM 一致：

- `M`：输出 `D` 的行数
- `N`：输出 `D` 的列数
- `K`：规约维度
- `lda`、`ldb`、`ldc`：leading dimension / stride
- `alpha`、`beta`：公式 `D = alpha * A*B + beta * C` 中的标量

## 7.2 What template parameters mean / 模板参数含义

### English

In the simple `device::Gemm` form above, many low-level choices are hidden behind defaults. When you want explicit control, you move to a more expanded form.

**Source / 源码:** adapted from `examples/47_ampere_gemm_universal_streamk/ampere_gemm_universal_streamk.cu`

```c++
using DeviceGemmBasic = cutlass::gemm::device::GemmUniversal<
    ElementA, LayoutA,
    ElementB, LayoutB,
    ElementC, LayoutC,
    ElementAccumulator,
    OperatorClass,
    ArchTag,
    ThreadblockShape,
    WarpShape,
    InstructionShape,
    EpilogueOp,
    cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<>,
    NumStages,
    AlignmentA,
    AlignmentB>;
```

Key parameters:

| Parameter | Meaning |
|---|---|
| `ElementA/B/C` | element types of input/output tensors |
| `LayoutA/B/C` | memory layouts |
| `ElementAccumulator` | internal accumulation type |
| `OperatorClass` | SIMT vs TensorOp / block-scaled TensorOp |
| `ArchTag` | target GPU architecture (e.g. `Sm80`, `Sm90`, `Sm100`) |
| `ThreadblockShape` | CTA tile size |
| `WarpShape` | warp tile size |
| `InstructionShape` | Tensor Core/SIMT instruction tile |
| `EpilogueOp` | output transformation |
| `ThreadblockSwizzle` | grid/tile mapping policy |
| `NumStages` | pipeline stage count |
| `AlignmentA/B` | vectorized memory access alignment |

### 中文

在上面那个简单的 `device::Gemm` 版本里，很多底层选择都被默认值隐藏掉了。想要显式控制时，就需要进入更完整的模板形式。

关键参数可以这样理解：

| 参数 | 含义 |
|---|---|
| `ElementA/B/C` | 输入输出张量的数据类型 |
| `LayoutA/B/C` | 内存布局 |
| `ElementAccumulator` | 内部累加类型 |
| `OperatorClass` | 使用 SIMT 还是 TensorOp / block-scaled TensorOp |
| `ArchTag` | 目标 GPU 架构（如 `Sm80`、`Sm90`、`Sm100`） |
| `ThreadblockShape` | CTA tile 大小 |
| `WarpShape` | warp tile 大小 |
| `InstructionShape` | Tensor Core/SIMT 指令的 tile 大小 |
| `EpilogueOp` | 输出变换方式 |
| `ThreadblockSwizzle` | 网格与 tile 的映射策略 |
| `NumStages` | pipeline 的 stage 数 |
| `AlignmentA/B` | 向量化访存对齐要求 |

## 7.3 A 3.x view of the same GEMM / 用 3.x 视角看同一个 GEMM

### English

In CUTLASS 3.x, you should think in three steps:

1. Build the **collective mainloop**
2. Build the **collective epilogue**
3. Compose them into **`kernel::GemmUniversal`**, then wrap with **`GemmUniversalAdapter`**

If you understand those three steps, you already understand most of modern CUTLASS GEMM assembly.

### 中文

在 CUTLASS 3.x 里，建议把 GEMM 分成三步看：

1. 构造 **collective mainloop**
2. 构造 **collective epilogue**
3. 把两者组合成 **`kernel::GemmUniversal`**，再用 **`GemmUniversalAdapter`** 包装

如果你真正理解了这三步，其实就已经掌握了现代 CUTLASS GEMM 组装的大部分核心。

## 7.4 A Hopper 3.x GEMM example / Hopper 3.x GEMM 示例

**Source / 源码:** `examples/48_hopper_warp_specialized_gemm/48_hopper_warp_specialized_gemm.cu`

```c++
using TileShape    = Shape<_128,_128,_32>;
using ClusterShape = Shape<_4,_2,_1>;

using CollectiveMainloop = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    ElementA, LayoutA, AlignmentA,
    ElementB, LayoutB, AlignmentB,
    ElementAccumulator,
    TileShape, ClusterShape,
    cutlass::gemm::collective::StageCountAutoCarveout<
      static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))>,
    cutlass::gemm::collective::KernelScheduleAuto
  >::CollectiveOp;

using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
    Shape<int,int,int>,
    CollectiveMainloop,
    CollectiveEpilogue
>;
```

**English:** Notice how the “shape of work” and the “schedule” are explicit, but low-level implementation details remain factored into builders and policies.  
**中文：** 注意这里“工作块的形状”和“调度策略”是显式的，但更底层的实现细节被封装在 builder 与 policy 中，这正是 3.x 风格的优势。

## 7.5 One useful advanced variation: Stream-K / 一个实用变体：Stream-K

**Source / 源码:** `examples/47_ampere_gemm_universal_streamk/ampere_gemm_universal_streamk.cu`

```c++
using DeviceGemmStreamK = cutlass::gemm::device::GemmUniversal<
    ElementA, LayoutA,
    ElementB, LayoutB,
    ElementC, LayoutC,
    ElementAccumulator,
    OperatorClass,
    ArchTag,
    ThreadblockShape,
    WarpShape,
    InstructionShape,
    EpilogueOp,
    cutlass::gemm::threadblock::ThreadblockSwizzleStreamK,
    NumStages,
    AlignmentA,
    AlignmentB>;
```

**English:** Here the key difference is the swizzle/scheduler choice. Same mathematical GEMM, different work decomposition.  
**中文：** 这里最关键的变化其实只是 swizzle / scheduler 的选择：数学上还是同一个 GEMM，但工作分解方式不同。

---

## 8. Convolution / 卷积

### English

CUTLASS implements convolution primarily through **implicit GEMM**. Instead of explicitly materializing an `im2col` matrix, it forms the needed GEMM tiles on the fly while loading data.

The convolution documentation (`media/docs/cpp/implicit_gemm_convolution.md`) explains the mapping:

- activations `x` are often viewed as NHWC
- filters `w` are often viewed as KRSC
- output `y` is often NPQK
- convolution becomes a GEMM where:
  - `A` is `NHW x RSC`
  - `B` is `RSC x K`
  - `C` is `NPQ x K`

This avoids the memory blow-up of explicitly constructing the full `im2col` matrix.

### 中文

CUTLASS 中卷积最核心的实现方式是 **implicit GEMM（隐式 GEMM）**。它不会显式地构造一个完整的 `im2col` 矩阵，而是在加载数据时动态形成 GEMM 所需的 tile。

卷积文档 `media/docs/cpp/implicit_gemm_convolution.md` 给出了映射关系：

- 输入激活张量 `x` 常见为 NHWC
- 卷积核 `w` 常见为 KRSC
- 输出 `y` 常见为 NPQK
- 卷积可改写为一个 GEMM：
  - `A` 是 `NHW x RSC`
  - `B` 是 `RSC x K`
  - `C` 是 `NPQ x K`

这样做的优点是避免显式 `im2col` 带来的巨大中间存储与带宽浪费。

### The core mapping equations / 核心映射公式

**Source / 源码:** `media/docs/cpp/implicit_gemm_convolution.md`

```text
y[n, p, q, k] = sum_c(sum_r(sum_s( x[n, f(p, r), g(q, s), c] * w[k, r, s, c] )))

i = q + Q * (p + P * n)
j = k
gemm_k = s + S * (r + R * c)
```

### A CUTLASS convolution kernel definition / CUTLASS 卷积 kernel 定义示例

**Source / 源码:** `media/docs/cpp/implicit_gemm_convolution.md`

```c++
using Conv2dFpropKernel = typename cutlass::conv::kernel::DefaultConv2dFprop<
  ElementInputA,
  LayoutInputA,
  ElementInputB,
  LayoutInputB,
  ElementC,
  LayoutC,
  ElementAccumulator,
  MMAOp,
  SmArch,
  ThreadblockShape,
  WarpShape,
  InstructionShape,
  EpilogueOp,
  SwizzleThreadBlock,
  NumStages,
  cutlass::arch::OpMultiplyAddSaturate,
  cutlass::conv::IteratorAlgorithm::kOptimized
>::Kernel;
```

### Practical notes / 实践要点

### English

- NHWC is especially common for high-performance convolution paths.
- Alignment matters: the docs explicitly recommend 128-bit aligned NHWC tensors for best performance.
- Fprop, dgrad, and wgrad share the same big idea, but differ in iterator math and problem mapping.
- CUTLASS also supports convolution fused with bias / activation / scale.

### 中文

- NHWC 是高性能卷积路径里非常常见的布局。
- 对齐很重要：文档明确建议尽量使用 **128-bit 对齐** 的 NHWC tensor。
- fprop、dgrad、wgrad 都建立在同样的隐式 GEMM 思想上，只是 iterator 数学和问题映射不同。
- CUTLASS 还支持与 bias、激活函数、scale 等操作融合的卷积 kernel。

---

## 9. Advanced Topics / 高级主题

## 9.1 Fused multi-head attention / 融合多头注意力

### English

CUTLASS includes fused attention-style kernels because attention is bandwidth-sensitive: writing intermediate matrices like `QK^T`, softmax results, and `P*V` back to global memory can dominate runtime.

Repository examples include:

- `examples/41_fused_multi_head_attention`
- `examples/77_blackwell_fmha`
- `examples/88_hopper_fmha`

The big idea is to fuse:

1. score computation
2. scaling / masking
3. softmax
4. value accumulation

into fewer kernels and fewer global-memory round trips.

### 中文

CUTLASS 提供 fused attention 风格的 kernel，因为 attention 非常容易受内存带宽限制：如果把 `QK^T`、softmax 中间结果、`P*V` 这些中间矩阵都写回 global memory，再读回来，代价往往非常高。

仓库中的相关示例包括：

- `examples/41_fused_multi_head_attention`
- `examples/77_blackwell_fmha`
- `examples/88_hopper_fmha`

核心思想就是把：

1. score 计算
2. scale / mask
3. softmax
4. value 累积

尽量融合进更少的 kernel、减少中间结果回写。

## 9.2 Mixed precision / 混合精度

### English

Mixed precision is everywhere in CUTLASS:

- FP16/BF16 inputs with FP32 accumulation
- TF32 paths for FP32-like GEMMs on Ampere+
- FP8 and mixed FP8 combinations (`e4m3`, `e5m2`)
- INT8/INT4 inference kernels
- dequantization / scaling fusion in epilogues or mainloops

Representative examples:

- `examples/54_hopper_fp8_warp_specialized_gemm`
- `examples/55_hopper_mixed_dtype_gemm`
- `examples/58_ada_fp8_gemm`
- `examples/67_hopper_fp8_warp_specialized_gemm_with_blockwise_scaling`
- `examples/72_blackwell_narrow_precision_gemm`

### 中文

混合精度几乎贯穿整个 CUTLASS：

- FP16/BF16 输入 + FP32 累加
- Ampere 及以后用 TF32 支撑类 FP32 GEMM
- FP8 与混合 FP8（`e4m3`、`e5m2`）
- INT8 / INT4 推理 kernel
- 在 mainloop 或 epilogue 里融合 dequantization / scaling

相关示例包括：

- `examples/54_hopper_fp8_warp_specialized_gemm`
- `examples/55_hopper_mixed_dtype_gemm`
- `examples/58_ada_fp8_gemm`
- `examples/67_hopper_fp8_warp_specialized_gemm_with_blockwise_scaling`
- `examples/72_blackwell_narrow_precision_gemm`

## 9.3 Grouped GEMM / Grouped GEMM

### English

Grouped GEMM means: launch one persistent kernel over **many GEMM problems with different sizes**.

The grouped scheduler documentation explains that the kernel launches fewer thread blocks than total tiles, and each thread block repeatedly asks the scheduler for the next tile/problem to process.

Representative files:

- `examples/24_gemm_grouped`
- `examples/57_hopper_grouped_gemm`
- `media/docs/cpp/grouped_scheduler.md`

This is especially useful for workloads like MoE, batched serving, and irregular inference.

### 中文

Grouped GEMM 的含义是：在一个 persistent kernel 中，同时处理 **很多尺寸不同的 GEMM 问题**。

官方 grouped scheduler 文档说明了它的工作方式：kernel 并不会为所有 tile 启动同样数量的 thread block，而是启动较少的 worker，然后让每个 worker 不断向 scheduler 请求下一个 tile / 问题来处理。

典型文件包括：

- `examples/24_gemm_grouped`
- `examples/57_hopper_grouped_gemm`
- `media/docs/cpp/grouped_scheduler.md`

这对 MoE、批量推理、不规则小矩阵任务特别有用。

## 9.4 StreamK partitioning / StreamK 分解

### English

The Stream-K example describes Stream-K as a contrast to classic data-parallel and Split-K GEMM decompositions. It is a work-centric decomposition designed to improve utilization and load balancing, especially when standard tile assignment leaves the GPU underutilized.

Representative file:

- `examples/47_ampere_gemm_universal_streamk/ampere_gemm_universal_streamk.cu`

### 中文

Stream-K 示例把它和经典的数据并行分解以及 Split-K 做了对比。它是一种以工作负载为中心（work-centric）的分解方式，目标是在标准 tile 分配无法充分利用 GPU 时，改善利用率和负载均衡。

代表文件：

- `examples/47_ampere_gemm_universal_streamk/ampere_gemm_universal_streamk.cu`

## 9.5 Distributed GEMM / 分布式 GEMM

### English

CUTLASS has an experimental **Distributed GEMM** API for tensor-parallel style GEMMs across multiple GPUs. The Hopper example notes an any-to-any NVLink topology requirement and uses wrappers like:

- `DistributedGemmKernelWrapper`
- `DistributedGemmUniversalAdapter`

Representative file:

- `examples/65_distributed_gemm/65_distributed_gemm.cu`

### 中文

CUTLASS 提供了实验性的 **Distributed GEMM** API，用于多 GPU 的 tensor parallel 风格 GEMM。Hopper 示例明确要求设备之间具备 any-to-any 的 NVLink 拓扑，并通过以下封装来构建分布式 kernel：

- `DistributedGemmKernelWrapper`
- `DistributedGemmUniversalAdapter`

代表文件：

- `examples/65_distributed_gemm/65_distributed_gemm.cu`

## 9.6 Sparse GEMM / 稀疏 GEMM

### English

Ampere introduced structured sparse Tensor Core paths. In CUTLASS sparse GEMM, you often have an extra metadata tensor `E` describing sparsity structure.

**Source / 源码:** `examples/15_ampere_sparse_tensorop_gemm/ampere_sparse_tensorop_gemm.cu`

```c++
using Gemm = cutlass::gemm::device::SparseGemm<
    ElementInputA,
    LayoutInputA,
    ElementInputB,
    LayoutInputB,
    ElementOutput,
    LayoutOutput,
    ElementAccumulator,
    MMAOp,
    SmArch,
    ShapeMMAThreadBlock,
    ShapeMMAWarp,
    ShapeMMAOp,
    EpilogueOp,
    SwizzleThreadBlock,
    NumStages>;
```

That example also shows that metadata must often be **reordered** for efficient loading by sparse Tensor Core instructions.

### 中文

Ampere 引入了结构化稀疏 Tensor Core 路径。在 CUTLASS 的 sparse GEMM 中，通常会多出一个元数据张量 `E`，用来描述稀疏结构。

该示例还展示了一个关键点：为了让稀疏 Tensor Core 指令高效加载，metadata 往往需要先做 **reorder**。

## 9.7 Block-scaled GEMM / Block-scaled GEMM

### English

Block-scaled GEMM is one of the most important recent additions in CUTLASS. Instead of applying one scale to an entire tensor, scale factors are associated with blocks (for example every 16 or 32 elements along K, depending on format/architecture).

Representative files:

- `examples/67_hopper_fp8_warp_specialized_gemm_with_blockwise_scaling`
- `examples/72_blackwell_narrow_precision_gemm`
- `examples/81_blackwell_gemm_blockwise`
- `examples/89_sm103_fp4_ultra_gemm`
- `media/docs/cpp/blackwell_functionality.md`

### 中文

Block-scaled GEMM 是 CUTLASS 近几个版本中最重要的增强之一。它不是给整个 tensor 只配一个 scale，而是把 scale factor 绑定到更小的 block 上（例如沿 K 维每 16 或 32 个元素一个 scale，具体取决于格式和架构）。

相关文件包括：

- `examples/67_hopper_fp8_warp_specialized_gemm_with_blockwise_scaling`
- `examples/72_blackwell_narrow_precision_gemm`
- `examples/81_blackwell_gemm_blockwise`
- `examples/89_sm103_fp4_ultra_gemm`
- `media/docs/cpp/blackwell_functionality.md`

---

## 10. Hopper Architecture Features / Hopper 架构特性

### English

Hopper is where modern CUTLASS 3.x ideas really become visible.

The repository’s Hopper GEMM examples and `efficient_gemm.md` highlight several defining features:

### 10.1 TMA

Hopper introduces **Tensor Memory Accelerator (TMA)** for large multidimensional copies between global memory and shared memory.

### 10.2 Warp-specialized kernels

The `efficient_gemm.md` documentation explains Hopper warp specialization as splitting a thread block into:

- **producer warp groups**: move data (often with TMA)
- **consumer warp groups**: issue MMA instructions

This lets copy and compute overlap more cleanly.

### 10.3 WGMMA / GMMA

Hopper introduces newer Tensor Core instructions, commonly discussed in CUTLASS docs/examples as **GMMA/WGMMA**. These operate at warpgroup granularity and are one reason CUTLASS 3.x needed a more general abstraction than the old warp-only mental model.

### 10.4 Asynchronous pipeline

Hopper kernels use asynchronous pipelines and barriers so that:

- one stage is being computed,
- another stage is being loaded,
- and a future stage may already be in flight.

This is essential because high-performance GEMM is almost always limited by the need to hide memory latency while maintaining enough data reuse.

### 中文

Hopper 是 CUTLASS 3.x 设计思想真正“显形”的一代架构。

从仓库中的 Hopper 示例以及 `efficient_gemm.md` 可以看到几个标志性特征：

### 10.1 TMA

Hopper 引入了 **Tensor Memory Accelerator (TMA)**，用于在 global memory 与 shared memory 之间高效搬运大规模多维 tile。

### 10.2 Warp-specialized kernel

`efficient_gemm.md` 解释了 Hopper 的 warp specialization：一个 thread block 会被分成两类角色：

- **producer warp group**：负责搬运数据（通常借助 TMA）
- **consumer warp group**：负责发射 MMA 指令做计算

这样 copy 和 compute 可以更自然地重叠。

### 10.3 WGMMA / GMMA

Hopper 引入了更新的 Tensor Core 指令，在 CUTLASS 文档和示例中通常会以 **GMMA/WGMMA** 的形式出现。它们以 warpgroup 为粒度工作，这也是 CUTLASS 3.x 需要比旧版 warp-only 心智模型更通用抽象的重要原因之一。

### 10.4 异步流水线

Hopper kernel 大量依赖异步 pipeline 与 barrier，使得：

- 当前 stage 在计算，
- 下一 stage 在加载，
- 更远的一 stage 可能已经在传输途中。

这对高性能 GEMM 非常关键，因为 GEMM 的高性能几乎总是建立在“隐藏内存延迟 + 保持足够数据复用”这两个目标同时实现的基础上。

### A Hopper example / Hopper 示例

**Source / 源码:** `examples/48_hopper_warp_specialized_gemm/48_hopper_warp_specialized_gemm.cu`

```c++
using TileShape    = Shape<_128,_128,_32>;
using ClusterShape = Shape<_4,_2,_1>;
using ArchTag      = cutlass::arch::Sm90;
using OperatorClass = cutlass::arch::OpClassTensorOp;
```

**English:** Hopper kernels make cluster shape and schedule first-class design choices.  
**中文：** 在 Hopper kernel 中，cluster shape 和 schedule 已经成为一等公民式的设计选择，而不只是附属参数。

---

## 11. Blackwell Architecture Features / Blackwell 架构特性

### English

Blackwell extends the CUTLASS 3.x model again. The repository’s Blackwell docs and examples emphasize several new ideas.

### 11.1 SM100 `tcgen05` MMA

The Blackwell functionality doc states that Blackwell SM100 introduces **`tcgen05.mma`** instructions supporting legacy types as well as new narrow-precision and block-scaled formats.

### 11.2 TMEM (Tensor Memory)

The `70_blackwell_fp16_gemm.cu` example explicitly notes that Blackwell MMA paths store accumulation results in **TMEM** instead of the ordinary register-file-only view used previously.

### 11.3 Extended warp specialization

Blackwell continues the warp-specialized approach and extends it, using TMEM and newer scheduling strategies to decouple MMA and epilogue work even further.

### 11.4 Cluster Launch Control (CLC)

The `blackwell_cluster_launch_control.md` document explains Blackwell’s **dynamic persistent scheduling** via cluster launch control. Instead of relying only on static persistent scheduling, workers can dynamically fetch work for better load balance.

### 11.5 2-SM collaborative MMA / 2-SM 协同 MMA

Newer Blackwell examples and README notes mention **2SM MMA** kernels. This means a tile can be computed cooperatively across two SMs / CTAs depending on the kernel schedule and cluster configuration.

### 11.6 Multicast TMA / 多播 TMA

The README notes that some SM100 mixed TMA + `cp.async` kernels use **TMA multicast** for matrix A tiles when the cluster shape is nontrivial along N.

### 11.7 FP4 and narrow-precision support

Blackwell adds strong support for:

- FP8 / FP6 / FP4-like narrow formats
- block-scaled formats such as `mx_float*` and `nv_float4_t`
- high-throughput block-scaled MMA paths

### 中文

Blackwell 在 CUTLASS 3.x 模型基础上又向前推进了一大步。仓库中的 Blackwell 文档和示例强调了以下几个新方向。

### 11.1 SM100 `tcgen05` MMA

Blackwell 功能文档明确说明：SM100 引入了 **`tcgen05.mma`** 指令族，既支持传统类型，也支持新的窄精度与 block-scaled 数据格式。

### 11.2 TMEM（Tensor Memory）

`70_blackwell_fp16_gemm.cu` 示例明确指出，Blackwell 的部分 MMA 路径会把累加结果存放到 **TMEM**，这和此前主要基于寄存器理解的模型相比是一个重要变化。

### 11.3 扩展版 warp specialization

Blackwell 延续并强化了 warp-specialized 设计，借助 TMEM 和新的调度策略，使 MMA 和 epilogue 的解耦程度更高。

### 11.4 Cluster Launch Control（CLC）

`blackwell_cluster_launch_control.md` 解释了 Blackwell 的 **动态 persistent 调度**。与纯静态 persistent 调度相比，worker 可以动态获取新工作，从而改善负载均衡。

### 11.5 2-SM 协同 MMA

README 和新一代 Blackwell 示例都提到了 **2SM MMA**。它意味着一个 tile 可以在两个 SM / CTA 间协同计算，具体取决于 kernel schedule 和 cluster 配置。

### 11.6 多播 TMA

README 指出，在某些 SM100 的 mixed TMA + `cp.async` kernel 中，当 cluster shape 在 N 方向非平凡时，会对 A tile 使用 **TMA multicast**。

### 11.7 FP4 与窄精度支持

Blackwell 强化了对以下格式的支持：

- FP8 / FP6 / FP4 等窄精度格式
- `mx_float*`、`nv_float4_t` 这类 block-scaled 格式
- 更高吞吐的 block-scaled MMA 路径

### Blackwell code examples / Blackwell 代码示例

**Source / 源码:** `examples/70_blackwell_gemm/70_blackwell_fp16_gemm.cu`

```c++
using ArchTag          = cutlass::arch::Sm100;
using MmaTileShape_MNK = Shape<_256,_128,_64>;
using ClusterShape_MNK = Shape<_2,_2,_1>;
```

**Source / 源码:** `examples/72_blackwell_narrow_precision_gemm/72a_blackwell_nvfp4_bf16_gemm.cu`

```c++
using ElementA      = cutlass::nv_float4_t<cutlass::float_e2m1_t>;
using ElementB      = cutlass::nv_float4_t<cutlass::float_e2m1_t>;
using OperatorClass = cutlass::arch::OpClassBlockScaledTensorOp;
```

**Source / 源码:** `examples/89_sm103_fp4_ultra_gemm/89_sm103_fp4_ultra_gemm.cu`

```c++
using MmaTileShape1Sm = cute::Shape<cute::_128, cute::_256, Int<768>>;
using MmaTileShape2Sm = cute::Shape<cute::_256, cute::_256, Int<768>>;
```

**English:** These snippets show the three major Blackwell themes: new architecture tag, new narrow/block-scaled datatypes, and explicit 1SM vs 2SM kernel shapes.  
**中文：** 这几段代码很好地体现了 Blackwell 的三个关键词：新的架构标签、新的窄精度/块缩放数据类型，以及显式区分 1SM 和 2SM 的 kernel 形状。

---

## 12. Python Interface (CuTeDSL) / Python 接口

### English

There are really **two Python stories** in this repository.

### 12.1 High-level CUTLASS Python interface

The `python/README.md` documents a high-level Python interface that can declare, compile, and run CUTLASS operators from Python.

**Source / 源码:** `python/README.md`

```python
import cutlass
import numpy as np

plan = cutlass.op.Gemm(element=np.float16, layout=cutlass.LayoutType.RowMajor)
A, B, C, D = [np.ones((1024, 1024), dtype=np.float16) for i in range(4)]
plan.run(A, B, C, D)
```

This interface emphasizes ease of use. The README explicitly says it does **not** try to be a full autotuning library like cuBLAS and does not necessarily pick the globally optimal kernel.

### 12.2 CuTeDSL / CUTLASS DSL

CUTLASS 4.x introduces Python-native DSLs, especially **CuTe DSL**.

According to the Python DSL docs:

- it is a Python-based DSL for high-performance GPU kernels
- it stays conceptually aligned with CuTe C++
- it supports JIT compilation
- it integrates with DLPack / frameworks like PyTorch and JAX
- its main decorators are `@jit` and `@kernel`

### 12.3 Jupyter notebooks

The Python DSL documentation lists educational notebooks covering topics such as:

- hello world
- printing
- data types
- tensors
- TensorSSA
- layout algebra
- elementwise add
- CUDA graphs

The older Python interface also includes example notebooks such as:

- `python/docs/externals/00_basic_gemm.ipynb`
- `python/docs/externals/01_epilogue.ipynb`
- `python/docs/externals/02_pytorch_extension_grouped_gemm.ipynb`

### 中文

这个仓库里的 Python 其实有 **两条路线**。

### 12.1 高层 CUTLASS Python 接口

`python/README.md` 介绍了一个高层 Python 接口，可以直接从 Python 里声明、编译、运行 CUTLASS 算子。

上面的 `plan = cutlass.op.Gemm(...)` 就是最典型的例子。

这个接口强调的是“易用性”。README 也明确说明：它 **不是** 像 cuBLAS 那样的完整自动调优库，也不保证总能选到全局最优 kernel。

### 12.2 CuTeDSL / CUTLASS DSL

CUTLASS 4.x 又引入了 Python 原生的 DSL，最核心的是 **CuTe DSL**。

根据 Python DSL 文档：

- 它是面向高性能 GPU kernel 的 Python DSL
- 概念上与 CuTe C++ 对齐
- 支持 JIT 编译
- 能和 DLPack、PyTorch、JAX 等框架协作
- 核心装饰器是 `@jit` 和 `@kernel`

### 12.3 Jupyter notebooks

Python DSL 文档中还列出了大量教学 notebook，例如：

- hello world
- printing
- data types
- tensors
- TensorSSA
- layout algebra
- elementwise add
- CUDA graphs

而旧的 Python interface 也提供了 notebook，例如：

- `python/docs/externals/00_basic_gemm.ipynb`
- `python/docs/externals/01_epilogue.ipynb`
- `python/docs/externals/02_pytorch_extension_grouped_gemm.ipynb`

### Installation notes / 安装说明

### English

For the older Python interface:

```bash
pip install nvidia-cutlass
```

For CuTe DSL, the docs recommend using the repository’s setup script:

```bash
./cutlass/python/CuTeDSL/setup.sh --cu12
# or
./cutlass/python/CuTeDSL/setup.sh --cu13
```

### 中文

对于较早的 Python 接口，可以直接：

```bash
pip install nvidia-cutlass
```

对于 CuTe DSL，文档建议使用仓库中的安装脚本：

```bash
./cutlass/python/CuTeDSL/setup.sh --cu12
# 或
./cutlass/python/CuTeDSL/setup.sh --cu13
```

---

## 13. Building & Running / 构建与运行

### English

The CUTLASS quickstart (`media/docs/cpp/quickstart.md`) gives the essential build flow.

### 13.1 Requirements / 依赖

Minimum practical requirements from the repo docs:

- CUDA Toolkit **11.4+**
- C++17 host compiler
- CMake **3.18+**
- Volta (SM70) or newer for modern CUTLASS-3-era features

For modern Hopper/Blackwell examples, use recent CUDA 12.x/12.8+ or the toolkit version explicitly called out by the example/docs.

### 13.2 Basic CMake setup / 基本 CMake 配置

**Source / 源码:** `media/docs/cpp/quickstart.md`

```bash
export CUDACXX=${CUDA_INSTALL_PATH}/bin/nvcc
mkdir build && cd build
cmake .. -DCUTLASS_NVCC_ARCHS=90a    # Hopper
# or
cmake .. -DCUTLASS_NVCC_ARCHS=100a   # Blackwell SM100
```

Important architecture notes:

- `90a` is needed for Hopper architecture-accelerated features
- `100a` is needed for Blackwell architecture-accelerated features
- building only `90` or `100` may compile some PTX, but kernels that rely on `a`-suffixed features can fail at runtime

### 13.3 Useful CMake flags / 常用 CMake 参数

```bash
cmake .. -DCUTLASS_NVCC_ARCHS=90a -DCUTLASS_ENABLE_TESTS=OFF -DCUTLASS_UNITY_BUILD_ENABLED=ON
cmake .. -DCUTLASS_NVCC_ARCHS=90a -DCUTLASS_LIBRARY_OPERATIONS=conv2d
cmake .. -DCUTLASS_NVCC_ARCHS=90a -DCUTLASS_LIBRARY_KERNELS=cutlass3x*
```

Optional dependency flags:

```bash
-DCUTLASS_ENABLE_CUBLAS=OFF
-DCUTLASS_ENABLE_CUDNN=OFF
```

### 13.4 Build and run the profiler / 构建并运行 profiler

```bash
make cutlass_profiler -j12
./tools/profiler/cutlass_profiler --kernels=sgemm --m=4352 --n=4096 --k=4096
./tools/profiler/cutlass_profiler --operation=conv2d --n=8 --h=224 --w=224 --c=128 --k=128 --r=3 --s=3
```

### 13.5 Build and run tests / 构建并运行测试

```bash
make test_unit -j
```

You can also build narrower test groups like warp-level GEMM tests.

你也可以只构建更小范围的测试目标，例如 warp-level GEMM 测试。

### 13.6 Build and run specific examples / 构建并运行具体示例

A common workflow is:

```bash
mkdir -p build && cd build
cmake .. -DCUTLASS_NVCC_ARCHS=90a
make 48_hopper_warp_specialized_gemm -j
./examples/48_hopper_warp_specialized_gemm/48_hopper_warp_specialized_gemm --m=2048 --n=2048 --k=2048
```

For distributed GEMM, the example recommends an extra flag:

```bash
cmake .. -DCUTLASS_NVCC_ARCHS="90a" -DCUTLASS_ENABLE_GDC_FOR_SM90=1
```

### 中文

CUTLASS 的 quickstart 文档 `media/docs/cpp/quickstart.md` 已经给出了最关键的构建路径。

#### 依赖

仓库文档给出的最低实用要求包括：

- CUDA Toolkit **11.4+**
- 支持 C++17 的 host 编译器
- CMake **3.18+**
- 至少 Volta（SM70）以上架构用于现代 CUTLASS-3 时代的能力

如果你要跑 Hopper / Blackwell 示例，建议使用更新的 CUDA 12.x / 12.8+，或者直接按示例/文档中标出的 toolkit 版本来。

#### 基本配置

最常见的命令就是：

```bash
export CUDACXX=${CUDA_INSTALL_PATH}/bin/nvcc
mkdir build && cd build
cmake .. -DCUTLASS_NVCC_ARCHS=90a    # Hopper
# 或
cmake .. -DCUTLASS_NVCC_ARCHS=100a   # Blackwell SM100
```

这里 `90a` / `100a` 很关键，因为 Hopper / Blackwell 的一些架构加速特性只在带 `a` 后缀的目标架构上才可正确生成与运行。

#### 常用参数

- `CUTLASS_ENABLE_TESTS=OFF`：减少构建量
- `CUTLASS_UNITY_BUILD_ENABLED=ON`：减少编译时间
- `CUTLASS_LIBRARY_OPERATIONS=conv2d`：只编译某类算子
- `CUTLASS_LIBRARY_KERNELS=cutlass3x*`：按名字过滤 kernel

#### profiler 与测试

- `make cutlass_profiler -j12`
- `make test_unit -j`

这是最常见的验证方式。

---

## 14. References & Resources / 参考资源

### Official repository docs / 官方仓库文档

- `README.md`
- `media/docs/cpp/quickstart.md`
- `media/docs/cpp/functionality.md`
- `media/docs/cpp/cutlass_3x_design.md`
- `media/docs/cpp/gemm_api_3x.md`
- `media/docs/cpp/efficient_gemm.md`
- `media/docs/cpp/implicit_gemm_convolution.md`
- `media/docs/cpp/grouped_scheduler.md`
- `media/docs/cpp/cute/00_quickstart.md`
- `media/docs/cpp/cute/01_layout.md`
- `media/docs/cpp/cute/02_layout_algebra.md`
- `media/docs/cpp/cute/03_tensor.md`
- `media/docs/cpp/cute/04_algorithms.md`
- `media/docs/cpp/cute/0z_tma_tensors.md`
- `media/docs/cpp/blackwell_functionality.md`
- `media/docs/cpp/blackwell_cluster_launch_control.md`
- `media/docs/pythonDSL/overview.rst`
- `media/docs/pythonDSL/quick_start.rst`

### Representative examples / 代表性示例

- Basic GEMM: `examples/00_basic_gemm`
- Turing convolution: `examples/09_turing_tensorop_conv2dfprop`
- Ampere TF32 / sparse / StreamK: `examples/14_ampere_tf32_tensorop_gemm`, `15_ampere_sparse_tensorop_gemm`, `47_ampere_gemm_universal_streamk`
- Hopper GEMM / FP8 / Grouped GEMM: `examples/48_hopper_warp_specialized_gemm`, `54_hopper_fp8_warp_specialized_gemm`, `57_hopper_grouped_gemm`
- Distributed GEMM: `examples/65_distributed_gemm`
- Hopper blockwise FP8: `examples/67_hopper_fp8_warp_specialized_gemm_with_blockwise_scaling`
- Blackwell GEMM / narrow precision / MoE / FMHA: `examples/70_blackwell_gemm`, `72_blackwell_narrow_precision_gemm`, `77_blackwell_fmha`, `92_blackwell_moe_gemm`

### External references / 外部参考资料

- CUTLASS official docs: <https://docs.nvidia.com/cutlass/latest/>
- CUTLASS blog: <https://devblogs.nvidia.com/cutlass-linear-algebra-cuda/>
- GTC talk on CUTLASS hierarchy: <https://on-demand-gtc.gputechconf.com/gtcnew/sessionview.php?sessionName=s8854-cutlass%3a+software+primitives+for+dense+linear+algebra+at+all+levels+and+scales+within+cuda>
- PTX docs for MMA / WGMMA / tcgen05 / TMA: <https://docs.nvidia.com/cuda/parallel-thread-execution/>
- CUDA Programming Guide: <https://docs.nvidia.com/cuda/cuda-c-programming-guide/>
- Stream-K paper: <https://arxiv.org/abs/2301.03598>
- Hopper architecture in-depth blog: <https://developer.nvidia.com/blog/nvidia-hopper-architecture-in-depth/>
- OCP microscaling formats (MX): <https://www.opencompute.org/documents/ocp-microscaling-formats-mx-v1-0-spec-final-pdf>

---

## Suggested learning path / 建议学习路径

### English

If you are learning CUTLASS from scratch, a very effective order is:

1. `examples/00_basic_gemm`
2. `media/docs/cpp/efficient_gemm.md`
3. `media/docs/cpp/cutlass_3x_design.md`
4. `media/docs/cpp/gemm_api_3x.md`
5. CuTe docs: `00_quickstart.md`, `01_layout.md`, `03_tensor.md`
6. Hopper example `48_hopper_warp_specialized_gemm`
7. One advanced topic relevant to your workload (grouped, sparse, FP8, convolution, attention, distributed)

### 中文

如果你是从零开始学习 CUTLASS，一个很有效的学习顺序是：

1. `examples/00_basic_gemm`
2. `media/docs/cpp/efficient_gemm.md`
3. `media/docs/cpp/cutlass_3x_design.md`
4. `media/docs/cpp/gemm_api_3x.md`
5. CuTe 文档：`00_quickstart.md`、`01_layout.md`、`03_tensor.md`
6. Hopper 示例 `48_hopper_warp_specialized_gemm`
7. 再选一个与你工作负载最相关的高级主题（grouped、sparse、FP8、卷积、attention、distributed）

### Final takeaway / 最后总结

**English:** CUTLASS is best understood not as “a giant GEMM library,” but as a vocabulary for expressing high-performance GPU linear algebra across architectures. CuTe is the language of layouts and tensors; collectives are the language of cooperative work; and modern architectures like Hopper and Blackwell are the reason these abstractions matter.  
**中文：** CUTLASS 最好的理解方式，不是把它看成“一个很大的 GEMM 库”，而是把它看成一种用来表达高性能 GPU 线性代数实现的方法论与词汇体系。CuTe 负责表达 layout 与 tensor，collective 负责表达协作计算，而 Hopper 与 Blackwell 这样的新架构正是这些抽象存在价值的最好证明。
