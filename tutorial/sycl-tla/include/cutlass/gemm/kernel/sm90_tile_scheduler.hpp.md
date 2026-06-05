# sm90_tile_scheduler.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `include/cutlass/gemm/kernel/sm90_tile_scheduler.hpp`
- **Purpose / 用途 (EN):** Implements an SM90-specific GEMM kernel path and its launch-time wiring.
- **Purpose / 用途 (CN):** 实现面向 SM90 的 GEMM 内核路径及其启动期装配逻辑。
- **Line count / 行数:** 153

## Line-by-Line Analysis / 逐行分析
> The source is grouped into contiguous significant line ranges for readability. / 为了便于阅读，下面按连续的重要代码区间进行分析。

### Lines 1-20

```cpp
   1 | /***************************************************************************************************
   2 |  * Copyright (c) 2023 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
   3 |  * SPDX-License-Identifier: BSD-3-Clause
   4 |  *
   5 |  * Redistribution and use in source and binary forms, with or without
   6 |  * modification, are permitted provided that the following conditions are met:
   7 |  *
   8 |  * 1. Redistributions of source code must retain the above copyright notice, this
   9 |  * list of conditions and the following disclaimer.
  10 |  *
  11 |  * 2. Redistributions in binary form must reproduce the above copyright notice,
  12 |  * this list of conditions and the following disclaimer in the documentation
  13 |  * and/or other materials provided with the distribution.
  14 |  *
  15 |  * 3. Neither the name of the copyright holder nor the names of its
  16 |  * contributors may be used to endorse or promote products derived from
  17 |  * this software without specific prior written permission.
  18 |  *
  19 |  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
  20 |  * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
```
**EN:** This opening comment records the license, copyright ownership, and redistribution conditions that apply to the header.
**CN:** 开头注释说明了该头文件适用的许可证、版权归属以及再分发条件。

### Lines 21-31

```cpp
  21 |  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
  22 |  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
  23 |  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
  24 |  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
  25 |  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
  26 |  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
  27 |  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
  28 |  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
  29 |  *
  30 |  **************************************************************************************************/
  31 | #pragma once
```
**EN:** This opening comment records the license, copyright ownership, and redistribution conditions that apply to the header.
**CN:** 开头注释说明了该头文件适用的许可证、版权归属以及再分发条件。

### Lines 33-33

```cpp
  33 | #include "cutlass/gemm/kernel/static_tile_scheduler.hpp"
```
**EN:** This include block imports `cutlass/gemm/kernel/static_tile_scheduler.hpp`, providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/gemm/kernel/static_tile_scheduler.hpp`，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 35-35

```cpp
  35 | namespace cutlass::gemm::kernel::detail {
```
**EN:** These lines open or close the nested CUTLASS namespaces so the declarations stay scoped to the GEMM kernel subsystem.
**CN:** 这些代码行用于打开或关闭 CUTLASS 的嵌套命名空间，使声明被限制在 GEMM 内核子系统内。

### Lines 39-41

```cpp
  39 | // Persistent Thread Block (TB) scheduler
  40 | class PersistentTileSchedulerSm90:
  41 | public StaticPersistentTileScheduler<PersistentTileSchedulerSm90> {
```
**EN:** This block declares or specializes `PersistentTileSchedulerSm90`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `PersistentTileSchedulerSm90`，它是该头文件中承载某一层内核策略的核心类。

### Lines 43-49

```cpp
  43 |   using BaseScheduler = StaticPersistentTileScheduler<PersistentTileSchedulerSm90>;
  44 | public:
  45 |   using StaticPersistentTileScheduler::StaticPersistentTileScheduler;
  46 |   using Params = PersistentTileSchedulerSm90Params;
  47 |   using RasterOrder = typename Params::RasterOrder;
  48 |   using RasterOrderOptions = typename Params::RasterOrderOptions;
  49 |   using Arguments = BaseScheduler::Arguments;
```
**EN:** This alias block derives concise type names `BaseScheduler`, `StaticPersistentTileScheduler`, `Params`, `RasterOrder`, `RasterOrderOptions` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `BaseScheduler`, `StaticPersistentTileScheduler`, `Params`, `RasterOrder`, `RasterOrderOptions` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 51-51

```cpp
  51 |   static constexpr bool IsDynamicPersistent = false;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 53-56

```cpp
  53 |   using Pipeline = PipelineEmpty;
  54 |   using PipelineStorage = typename Pipeline::SharedStorage;
  55 |   using ThrottlePipeline = PipelineEmpty;
  56 |   using ThrottlePipelineStorage = typename ThrottlePipeline::SharedStorage;
```
**EN:** This alias block derives concise type names `Pipeline`, `PipelineStorage`, `ThrottlePipeline`, `ThrottlePipelineStorage` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Pipeline`, `PipelineStorage`, `ThrottlePipeline`, `ThrottlePipelineStorage` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 58-58

```cpp
  58 |   struct CLCResponse {};
```
**EN:** This block declares or specializes `CLCResponse`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `CLCResponse`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 60-65

```cpp
  60 |   class SharedStorage {
  61 |   public:
  62 |     CUTLASS_DEVICE PipelineStorage pipeline() { return PipelineStorage{}; }
  63 |     CUTLASS_DEVICE ThrottlePipelineStorage throttle_pipeline() { return ThrottlePipelineStorage{}; }
  64 |     CUTLASS_DEVICE CLCResponse* data() { return nullptr; }
  65 |   };
```
**EN:** This block defines the shared-memory layout used by the kernel, typically combining mainloop buffers, epilogue buffers, or pipeline state into one structured storage object.
**CN:** 这一段定义了内核使用的共享内存布局，通常会把主循环缓冲区、epilogue 缓冲区或流水状态组合到一个结构化存储对象中。

### Lines 67-88

```cpp
  67 |   // get work_idx_m, work_idx_n from blk_per_grid_dim while applying swizzle
  68 |   static CUTLASS_DEVICE
  69 |   cute::tuple<int32_t, int32_t>
  70 |   get_work_idx_m_and_n(
  71 |       uint64_t blk_per_grid_dim,
  72 |       FastDivmodU64Pow2 const& divmod_cluster_shape_major,
  73 |       FastDivmodU64Pow2 const& divmod_cluster_shape_minor,
  74 |       FastDivmodU64 const& divmod_cluster_blk_major,
  75 |       int32_t log_swizzle_size,
  76 |       RasterOrder raster_order) {
  77 |     auto [cta_m_in_cluster, cta_n_in_cluster, _] = cute::block_id_in_cluster();
  78 |     return get_work_idx_m_and_n(
  79 |       blk_per_grid_dim,
  80 |       divmod_cluster_shape_major,
  81 |       divmod_cluster_shape_minor,
  82 |       divmod_cluster_blk_major,
  83 |       log_swizzle_size,
  84 |       raster_order,
  85 |       cta_m_in_cluster,
  86 |       cta_n_in_cluster
  87 |     );
  88 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 90-100

```cpp
  90 |   static CUTLASS_DEVICE
  91 |   cute::tuple<int32_t, int32_t>
  92 |   get_work_idx_m_and_n(
  93 |       uint64_t blk_per_grid_dim,
  94 |       FastDivmodU64Pow2 const& divmod_cluster_shape_major,
  95 |       FastDivmodU64Pow2 const& divmod_cluster_shape_minor,
  96 |       FastDivmodU64 const& divmod_cluster_blk_major,
  97 |       int32_t log_swizzle_size,
  98 |       RasterOrder raster_order,
  99 |       uint64_t cta_m_in_cluster,
 100 |       uint64_t cta_n_in_cluster) {
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 102-103

```cpp
 102 |     uint64_t cluster_id, cluster_major_offset = 0, cluster_minor_offset = 0;
 103 |     divmod_cluster_shape_major(cluster_id, cluster_major_offset, blk_per_grid_dim);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 105-110

```cpp
 105 |     if (raster_order == RasterOrder::AlongN) {
 106 |       cluster_minor_offset = cta_m_in_cluster;
 107 |     }
 108 |     else {
 109 |       cluster_minor_offset = cta_n_in_cluster;
 110 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 112-112

```cpp
 112 |     uint64_t cluster_idx_minor, cluster_idx_major;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 114-114

```cpp
 114 |     uint64_t cluster_idx_minor_div_swizzle, extra, offset;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 116-117

```cpp
 116 |     offset = cluster_id & ((1 << log_swizzle_size) - 1);
 117 |     extra = cluster_id >> log_swizzle_size;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 119-119

```cpp
 119 |     divmod_cluster_blk_major(cluster_idx_minor_div_swizzle, cluster_idx_major, extra);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 121-121

```cpp
 121 |     cluster_idx_minor = cluster_idx_minor_div_swizzle * (1 << log_swizzle_size) + offset;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 123-126

```cpp
 123 |     auto minor_work_idx = static_cast<int32_t>(cluster_idx_minor * divmod_cluster_shape_minor.divisor +
 124 |                                                cluster_minor_offset);
 125 |     auto major_work_idx = static_cast<int32_t>(cluster_idx_major * divmod_cluster_shape_major.divisor +
 126 |                                                cluster_major_offset);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 128-135

```cpp
 128 |     if (raster_order == RasterOrder::AlongN) {
 129 |       return {minor_work_idx, major_work_idx};
 130 |     }
 131 |     else {
 132 |       return {major_work_idx, minor_work_idx};
 133 |     }
 134 | 
 135 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 137-142

```cpp
 137 |   // The basic tile scheduler does not require any additional workspace
 138 |   template <class ProblemShape, class ElementAccumulator>
 139 |   static size_t
 140 |   get_workspace_size(Arguments const&, ProblemShape, KernelHardwareInfo const&, uint32_t, const uint32_t = 1, uint32_t = 1) {
 141 |     return 0;
 142 |   }
```
**EN:** This block reports or initializes any auxiliary workspace needed by the kernel before launch.
**CN:** 这一段用于报告或初始化内核启动前所需的辅助工作区。

### Lines 144-153

```cpp
 144 |   template <class ProblemShape, class ElementAccumulator>
 145 |   static cutlass::Status
 146 |   initialize_workspace(Arguments const&, void*, cudaStream_t, ProblemShape, KernelHardwareInfo const&,
 147 |     uint32_t, const uint32_t = 1, uint32_t = 1, CudaHostAdapter* cuda_adapter = nullptr) {
 148 |     return Status::kSuccess;
 149 |   }
 150 | 
 151 | };
 152 | 
 153 | }
```
**EN:** This block reports or initializes any auxiliary workspace needed by the kernel before launch.
**CN:** 这一段用于报告或初始化内核启动前所需的辅助工作区。

## Key Concepts / 关键概念
- Template metaprogramming / 模板元编程
- Tile scheduling / Tile 调度
- Persistent scheduling / 持久化调度
- SM90 architecture tuning / SM90 架构调优
- Runtime argument packing / 运行时参数打包
- Packed launch parameters / 打包后的启动参数
- Shared memory layout / 共享内存布局

## Dependencies / 依赖关系
- **Included headers / 引入头文件:** `cutlass/gemm/kernel/static_tile_scheduler.hpp`
- **Subsystems / 子系统:** CuTe tensor utilities / CuTe 张量工具, Tile schedulers / Tile 调度器
- **Peer kernel headers / 同级内核头文件:** `cutlass/gemm/kernel/static_tile_scheduler.hpp`
