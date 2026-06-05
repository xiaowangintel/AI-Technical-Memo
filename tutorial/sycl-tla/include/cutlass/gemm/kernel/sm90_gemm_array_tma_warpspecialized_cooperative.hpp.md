# sm90_gemm_array_tma_warpspecialized_cooperative.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `include/cutlass/gemm/kernel/sm90_gemm_array_tma_warpspecialized_cooperative.hpp`
- **Purpose / 用途 (EN):** Implements an SM90-specific GEMM kernel path and its launch-time wiring.
- **Purpose / 用途 (CN):** 实现面向 SM90 的 GEMM 内核路径及其启动期装配逻辑。
- **Line count / 行数:** 1039

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

### Lines 33-50

```cpp
  33 | #include "cutlass/cutlass.h"
  34 | #include "cutlass/workspace.h"
  35 | #include "cutlass/fast_math.h"
  36 | #include "cutlass/kernel_hardware_info.hpp"
  37 | #include "cute/arch/cluster_sm90.hpp"
  38 | #include "cutlass/arch/reg_reconfig.h"
  39 | #include "cutlass/arch/mma_sm90.h"
  40 | #include "cutlass/epilogue/collective/detail.hpp"
  41 | #include "cutlass/gemm/gemm.h"
  42 | #include "cutlass/gemm/dispatch_policy.hpp"
  43 | #include "cutlass/gemm/kernel/gemm_universal_decl.h"
  44 | #include "cutlass/gemm/kernel/tile_scheduler.hpp"
  45 | #include "cutlass/gemm/group_array_problem_shape.hpp"
  46 | #include "cutlass/pipeline/pipeline.hpp"
  47 | #include "cute/tensor.hpp"
  48 | #include "cutlass/trace.h"
  49 | #include "cutlass/gemm/kernel/sm90_tile_scheduler.hpp"
  50 | #include "cutlass/gemm/kernel/sm90_tile_scheduler_group.hpp"
```
**EN:** This include block imports `cutlass/cutlass.h`, `cutlass/workspace.h`, `cutlass/fast_math.h`, `cutlass/kernel_hardware_info.hpp`, `cute/arch/cluster_sm90.hpp`, `cutlass/arch/reg_reconfig.h`, ... (+12 more), providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/cutlass.h`, `cutlass/workspace.h`, `cutlass/fast_math.h`, `cutlass/kernel_hardware_info.hpp`, `cute/arch/cluster_sm90.hpp`, `cutlass/arch/reg_reconfig.h`, ... (+12 more)，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 54-54

```cpp
  54 | namespace cutlass::gemm::kernel {
```
**EN:** These lines open or close the nested CUTLASS namespaces so the declarations stay scoped to the GEMM kernel subsystem.
**CN:** 这些代码行用于打开或关闭 CUTLASS 的嵌套命名空间，使声明被限制在 GEMM 内核子系统内。

### Lines 58-76

```cpp
  58 | template <
  59 |   class ProblemShape_,
  60 |   class CollectiveMainloop_,
  61 |   class CollectiveEpilogue_,
  62 |   class TileScheduler_
  63 | >
  64 | class GemmUniversal<
  65 |   ProblemShape_,
  66 |   CollectiveMainloop_,
  67 |   CollectiveEpilogue_,
  68 |   TileScheduler_,
  69 |   cute::enable_if_t<cute::is_base_of_v<KernelPtrArrayTmaWarpSpecializedCooperative, typename CollectiveMainloop_::DispatchPolicy::Schedule>>
  70 | >
  71 | {
  72 |   // Get the type of the scheduler response.
  73 |   template<typename TileScheduler, typename = void>
  74 |   struct TileSchedulerResponseGetter {
  75 |     using Type = typename TileScheduler::CLCResponse;
  76 |   };
```
**EN:** This alias block derives concise type names `Type` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Type` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 78-81

```cpp
  78 |   template<typename TileScheduler>
  79 |   struct TileSchedulerResponseGetter<TileScheduler, void_t<typename TileScheduler::SchedulerResponse>> {
  80 |     using Type = typename TileScheduler::SchedulerResponse;
  81 |   };
```
**EN:** This alias block derives concise type names `Type` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Type` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 83-89

```cpp
  83 | public:
  84 |   //
  85 |   // Type Aliases
  86 |   //
  87 |   using ProblemShape = ProblemShape_;
  88 |   static_assert(rank(typename ProblemShape::UnderlyingProblemShape{}) == 3 or rank(typename ProblemShape::UnderlyingProblemShape{}) == 4,
  89 |     "ProblemShape{} should be <M,N,K> or <M,N,K,L>");
```
**EN:** These compile-time checks enforce invariants such as problem-rank assumptions, supported scheduler tags, or required tensor-stride shapes.
**CN:** 这些编译期检查用于保证问题维度假设、可支持的调度器标签以及张量步长形状等不变式成立。

### Lines 91-91

```cpp
  91 |   static_assert(cute::is_base_of_v<KernelPtrArrayTmaWarpSpecializedCooperative, typename CollectiveMainloop_::DispatchPolicy::Schedule>);
```
**EN:** These compile-time checks enforce invariants such as problem-rank assumptions, supported scheduler tags, or required tensor-stride shapes.
**CN:** 这些编译期检查用于保证问题维度假设、可支持的调度器标签以及张量步长形状等不变式成立。

### Lines 93-93

```cpp
  93 |   static constexpr bool IsGdcEnabled = false;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 95-111

```cpp
  95 |   // Mainloop derived types
  96 |   using CollectiveMainloop = CollectiveMainloop_;
  97 |   using TileShape = typename CollectiveMainloop::TileShape;
  98 |   using TiledMma  = typename CollectiveMainloop::TiledMma;
  99 |   using ArchTag   = typename CollectiveMainloop::ArchTag;
 100 |   using ElementA  = typename CollectiveMainloop::ElementA;
 101 |   using StrideA   = typename CollectiveMainloop::StrideA;
 102 |   using InternalStrideA = typename CollectiveMainloop::InternalStrideA;
 103 |   using ElementB  = typename CollectiveMainloop::ElementB;
 104 |   using InternalStrideB = typename CollectiveMainloop::InternalStrideB;
 105 |   using StrideB   = typename CollectiveMainloop::StrideB;
 106 |   using DispatchPolicy = typename CollectiveMainloop::DispatchPolicy;
 107 |   using Schedule = typename DispatchPolicy::Schedule;
 108 |   using ElementAccumulator = typename CollectiveMainloop::ElementAccumulator;
 109 |   using ClusterShape = typename DispatchPolicy::ClusterShape;
 110 |   using MainloopArguments = typename CollectiveMainloop::Arguments;
 111 |   using MainloopParams = typename CollectiveMainloop::Params;
```
**EN:** This alias block derives concise type names `CollectiveMainloop`, `TileShape`, `TiledMma`, `ArchTag`, `ElementA` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `CollectiveMainloop`, `TileShape`, `TiledMma`, `ArchTag`, `ElementA` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 113-122

```cpp
 113 |   // Epilogue derived types
 114 |   using CollectiveEpilogue = CollectiveEpilogue_;
 115 |   using ElementC = typename CollectiveEpilogue::ElementC;
 116 |   using StrideC  = typename CollectiveEpilogue::StrideC;
 117 |   using InternalStrideC = typename CollectiveEpilogue::InternalStrideC;
 118 |   using ElementD = typename CollectiveEpilogue::ElementD;
 119 |   using StrideD  = typename CollectiveEpilogue::StrideD;
 120 |   using InternalStrideD = typename CollectiveEpilogue::InternalStrideD;
 121 |   using EpilogueArguments = typename CollectiveEpilogue::Arguments;
 122 |   using EpilogueParams = typename CollectiveEpilogue::Params;
```
**EN:** This alias block derives concise type names `CollectiveEpilogue`, `ElementC`, `StrideC`, `InternalStrideC`, `ElementD` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `CollectiveEpilogue`, `ElementC`, `StrideC`, `InternalStrideC`, `ElementD` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 124-124

```cpp
 124 |   static_assert(ArchTag::kMinComputeCapability >= 90);
```
**EN:** These compile-time checks enforce invariants such as problem-rank assumptions, supported scheduler tags, or required tensor-stride shapes.
**CN:** 这些编译期检查用于保证问题维度假设、可支持的调度器标签以及张量步长形状等不变式成立。

### Lines 126-127

```cpp
 126 |   static constexpr bool IsGroupedGemmKernel = !cute::is_same_v<InternalStrideA, StrideA>;
 127 |   static constexpr uint32_t MinTensorMapWorkspaceAlignment = 64;
```
**EN:** This line assembles the final kernel type by combining the chosen MMA/mainloop object, epilogue object, and threadblock scheduling policy.
**CN:** 这一行通过组合选定的 MMA/主循环对象、epilogue 对象以及线程块调度策略，装配出最终的内核类型。

### Lines 129-135

```cpp
 129 |   static_assert(
 130 |     cute::is_void_v<TileScheduler_>
 131 |     or (
 132 |       IsGroupedGemmKernel
 133 |       and cute::is_any_of_v<TileScheduler_, GroupScheduler>
 134 |     ),
 135 |     "Ptr-Array Cooperative and Grouped Gemm Cooperative kernel only supports the default scheduler.");
```
**EN:** These compile-time checks enforce invariants such as problem-rank assumptions, supported scheduler tags, or required tensor-stride shapes.
**CN:** 这些编译期检查用于保证问题维度假设、可支持的调度器标签以及张量步长形状等不变式成立。

### Lines 137-145

```cpp
 137 |   using SchedulerTag = cute::conditional_t<
 138 |     cute::is_void_v<TileScheduler_>,
 139 |     cute::conditional_t<
 140 |       IsGroupedGemmKernel,
 141 |       GroupScheduler,     // Special grouped gemm scheduler
 142 |       void                // Default scheduler for non-grouped kernels
 143 |     >,
 144 |     TileScheduler_
 145 |   >;
```
**EN:** This alias block derives concise type names `SchedulerTag` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `SchedulerTag` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 147-154

```cpp
 147 |   using TileScheduler = typename detail::TileSchedulerSelector<
 148 |     SchedulerTag,
 149 |     ArchTag,
 150 |     TileShape,
 151 |     ClusterShape,
 152 |     8, // SchedulerPipelineStageCount -- Grouped GEMM scheduler will benefit from a larger number of stages.
 153 |     cute::conditional_t<cute::is_same_v<SchedulerTag, void>, void, ProblemShape> // Use void for default scheduler.
 154 |   >::Scheduler;
```
**EN:** This specialization maps a scheduler tag and architecture tag to the concrete tile-scheduler class that should drive work distribution.
**CN:** 这一段特化把调度器标签和架构标签映射到具体的 Tile 调度器类，从而决定工作分配方式。

### Lines 156-158

```cpp
 156 |   using TileSchedulerArguments = typename TileScheduler::Arguments;
 157 |   using TileSchedulerParams = typename TileScheduler::Params;
 158 |   using TileSchedulerResponse = typename TileSchedulerResponseGetter<TileScheduler>::Type;
```
**EN:** This alias block derives concise type names `TileSchedulerArguments`, `TileSchedulerParams`, `TileSchedulerResponse` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `TileSchedulerArguments`, `TileSchedulerParams`, `TileSchedulerResponse` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 160-160

```cpp
 160 |   static constexpr auto TileSchedulerStages = 8;
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 162-168

```cpp
 162 |   static constexpr uint32_t NumLoadWarpGroups = 1;
 163 |   static constexpr uint32_t NumMmaThreads = size(TiledMma{});
 164 |   static constexpr uint32_t NumMmaWarpGroups = NumMmaThreads / NumThreadsPerWarpGroup;
 165 |   static constexpr uint32_t MaxThreadsPerBlock = NumMmaThreads + (NumLoadWarpGroups * NumThreadsPerWarpGroup);
 166 |   static constexpr uint32_t MinBlocksPerMultiprocessor = 1;
 167 |   static constexpr uint32_t NumProducerThreads = CollectiveMainloop::NumProducerThreadEvents;
 168 |   static constexpr bool     IsMainloopAuxiliaryLoadNeeded = detail::HasAuxiliaryLoad_v<typename CollectiveMainloop::DispatchPolicy>;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 170-172

```cpp
 170 |   /// Register requirement for Load and Math WGs
 171 |   static constexpr uint32_t LoadRegisterRequirement = 40;
 172 |   static constexpr uint32_t MmaRegisterRequirement = 232;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 174-175

```cpp
 174 |   // 1 stage ordered sequence between mainloop and epilogue producer load threads
 175 |   using LoadWarpOrderBarrier = cutlass::OrderedSequenceBarrier<1,2>;
```
**EN:** This alias block derives concise type names `LoadWarpOrderBarrier` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `LoadWarpOrderBarrier` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 177-181

```cpp
 177 |   // Kernel level shared memory storage
 178 |   struct SharedStorage {
 179 |     struct TensorStorage : cute::aligned_struct<128, _1> {
 180 |       using MainloopTensorStorage = typename CollectiveMainloop::TensorStorage;
 181 |       using EpilogueTensorStorage = typename CollectiveEpilogue::TensorStorage;
```
**EN:** This block defines the shared-memory layout used by the kernel, typically combining mainloop buffers, epilogue buffers, or pipeline state into one structured storage object.
**CN:** 这一段定义了内核使用的共享内存布局，通常会把主循环缓冲区、epilogue 缓冲区或流水状态组合到一个结构化存储对象中。

### Lines 183-185

```cpp
 183 |       MainloopTensorStorage mainloop;
 184 |       EpilogueTensorStorage epilogue;
 185 |     } tensors;
```
**EN:** This block defines the shared-memory layout used by the kernel, typically combining mainloop buffers, epilogue buffers, or pipeline state into one structured storage object.
**CN:** 这一段定义了内核使用的共享内存布局，通常会把主循环缓冲区、epilogue 缓冲区或流水状态组合到一个结构化存储对象中。

### Lines 187-190

```cpp
 187 |     struct PipelineStorage : cute::aligned_struct<16, _1> {
 188 |       using TileSchedulerPipelineStorage = typename TileScheduler::PipelineStorage;
 189 |       using MainloopPipelineStorage = typename CollectiveMainloop::PipelineStorage;
 190 |       using EpiLoadPipelineStorage = typename CollectiveEpilogue::PipelineStorage;
```
**EN:** This alias block derives concise type names `TileSchedulerPipelineStorage`, `MainloopPipelineStorage`, `EpiLoadPipelineStorage` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `TileSchedulerPipelineStorage`, `MainloopPipelineStorage`, `EpiLoadPipelineStorage` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 192-196

```cpp
 192 |       alignas(16) TileSchedulerPipelineStorage scheduler;
 193 |       alignas(16) MainloopPipelineStorage mainloop;
 194 |       alignas(16) EpiLoadPipelineStorage epi_load;
 195 |       alignas(16) typename LoadWarpOrderBarrier::SharedStorage load_order;
 196 |     } pipelines;
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 198-198

```cpp
 198 |     alignas(16) TileSchedulerResponse scheduler_response[TileSchedulerStages];
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 200-202

```cpp
 200 |     struct TensorMapStorage : cute::aligned_struct<128, _1> {
 201 |       using MainloopTensorMapStorage = typename CollectiveMainloop::TensorMapStorage;
 202 |       using EpilogueTensorMapStorage = typename CollectiveEpilogue::TensorMapStorage;
```
**EN:** This alias block derives concise type names `MainloopTensorMapStorage`, `EpilogueTensorMapStorage` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `MainloopTensorMapStorage`, `EpilogueTensorMapStorage` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 204-207

```cpp
 204 |       alignas(128) MainloopTensorMapStorage mainloop;
 205 |       alignas(128) EpilogueTensorMapStorage epilogue;
 206 |     } tensormaps;
 207 |   };
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 209-209

```cpp
 209 |   static constexpr int SharedStorageSize = sizeof(SharedStorage);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 211-219

```cpp
 211 |   // Device side arguments
 212 |   struct Arguments {
 213 |     GemmUniversalMode mode{};
 214 |     ProblemShape problem_shape{};
 215 |     MainloopArguments mainloop{};
 216 |     EpilogueArguments epilogue{};
 217 |     KernelHardwareInfo hw_info{};
 218 |     TileSchedulerArguments scheduler{};
 219 |   };
```
**EN:** This `Arguments` structure collects the host-facing runtime inputs: problem shape, pointers, strides, epilogue settings, hardware info, and scheduler data.
**CN:** 这个 `Arguments` 结构体收集面向主机侧的运行时输入，包括问题规模、数据指针、步长、epilogue 配置、硬件信息以及调度器数据。

### Lines 221-234

```cpp
 221 |   // Kernel entry point API
 222 |   struct Params {
 223 |     GemmUniversalMode mode{};
 224 |     ProblemShape problem_shape{};
 225 |     MainloopParams mainloop{};
 226 |     EpilogueParams epilogue{};
 227 |     KernelHardwareInfo hw_info{};
 228 |     TileSchedulerParams scheduler{};
 229 |     void* workspace{nullptr};
 230 |   };
 231 | 
 232 |   //
 233 |   // Methods
 234 |   //
```
**EN:** This `Params` structure stores the packed, device-ready launch parameters that the kernel reads after host-side preprocessing.
**CN:** 这个 `Params` 结构体保存经过主机侧预处理后的、可直接被设备端读取的启动参数。

### Lines 236-240

```cpp
 236 |   // Convert to underlying arguments. In this case, a simple copy for the aliased type.
 237 |   static
 238 |   Params
 239 |   to_underlying_arguments(Arguments const& args, void* workspace) {
 240 |     CUTLASS_TRACE_HOST("to_underlying_arguments():");
```
**EN:** This helper converts the public argument bundle into lower-level mainloop, epilogue, and scheduler parameters expected by the actual kernel implementation.
**CN:** 这个辅助函数把对外的参数包转换成底层内核实现真正需要的 mainloop、epilogue 和调度器参数。

### Lines 242-242

```cpp
 242 |     ProblemShape problem_shapes = args.problem_shape;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 244-251

```cpp
 244 |     // Get SM count if needed, otherwise use user supplied SM count
 245 |     int sm_count = args.hw_info.sm_count;
 246 |     if (sm_count <= 0) {
 247 |       CUTLASS_TRACE_HOST("  WARNING: Arguments do not include a valid SM count.\n"
 248 |           "  For optimal performance, populate the arguments KernelHardwareInfo struct with the SM count.");
 249 |       sm_count = KernelHardwareInfo::query_device_multiprocessor_count(args.hw_info.device_id);
 250 |     }
 251 |     CUTLASS_TRACE_HOST("to_underlying_arguments(): Setting persistent grid SM count to " << sm_count);
```
**EN:** This helper converts the public argument bundle into lower-level mainloop, epilogue, and scheduler parameters expected by the actual kernel implementation.
**CN:** 这个辅助函数把对外的参数包转换成底层内核实现真正需要的 mainloop、epilogue 和调度器参数。

### Lines 253-262

```cpp
 253 |     // Get maximum number of clusters that could co-exist on the target device
 254 |     int max_active_clusters = args.hw_info.max_active_clusters;
 255 |     if (max_active_clusters <= 0) {
 256 |       max_active_clusters = 0;
 257 |       CUTLASS_TRACE_HOST("  WARNING: Arguments do not include a valid max cluster count.\n"
 258 |           "  For optimal performance, populate the arguments KernelHardwareInfo struct with the max_active_clusters.");
 259 |     }
 260 |     else {
 261 |       CUTLASS_TRACE_HOST("to_underlying_arguments(): Setting persistent grid cluster count to " << max_active_clusters);
 262 |     }
```
**EN:** This helper converts the public argument bundle into lower-level mainloop, epilogue, and scheduler parameters expected by the actual kernel implementation.
**CN:** 这个辅助函数把对外的参数包转换成底层内核实现真正需要的 mainloop、epilogue 和调度器参数。

### Lines 264-264

```cpp
 264 |     KernelHardwareInfo hw_info{args.hw_info.device_id, sm_count, max_active_clusters};
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 266-268

```cpp
 266 |     // Calculate workspace pointers
 267 |     uint8_t* workspace_ptr = reinterpret_cast<uint8_t*>(workspace);
 268 |     size_t workspace_offset = 0;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 270-272

```cpp
 270 |     void* epilogue_workspace = workspace_ptr + workspace_offset;
 271 |     workspace_offset += CollectiveEpilogue::get_workspace_size(problem_shapes, args.epilogue, sm_count);
 272 |     workspace_offset = round_nearest(workspace_offset, MinTensorMapWorkspaceAlignment);
```
**EN:** This block reports or initializes any auxiliary workspace needed by the kernel before launch.
**CN:** 这一段用于报告或初始化内核启动前所需的辅助工作区。

### Lines 274-276

```cpp
 274 |     void* mainloop_workspace = workspace_ptr + workspace_offset;
 275 |     workspace_offset += CollectiveMainloop::get_workspace_size(problem_shapes, args.mainloop, sm_count);
 276 |     workspace_offset = round_nearest(workspace_offset, MinTensorMapWorkspaceAlignment);
```
**EN:** This block reports or initializes any auxiliary workspace needed by the kernel before launch.
**CN:** 这一段用于报告或初始化内核启动前所需的辅助工作区。

### Lines 278-281

```cpp
 278 |     void* scheduler_workspace = workspace_ptr + workspace_offset;
 279 |     workspace_offset += TileScheduler::template get_workspace_size<typename ProblemShape::UnderlyingProblemShape, ElementAccumulator>(
 280 |       args.scheduler, typename ProblemShape::UnderlyingProblemShape{}, args.hw_info, NumMmaWarpGroups);
 281 |     workspace_offset = round_nearest(workspace_offset, MinTensorMapWorkspaceAlignment);
```
**EN:** This block reports or initializes any auxiliary workspace needed by the kernel before launch.
**CN:** 这一段用于报告或初始化内核启动前所需的辅助工作区。

### Lines 283-291

```cpp
 283 |     TileSchedulerParams scheduler;
 284 |     if constexpr (IsGroupedGemmKernel) {
 285 |       scheduler = TileScheduler::to_underlying_arguments(
 286 |       problem_shapes, TileShape{}, ClusterShape{}, hw_info, args.scheduler, scheduler_workspace);
 287 |     }
 288 |     else {
 289 |       scheduler = TileScheduler::to_underlying_arguments(
 290 |       problem_shapes.get_host_problem_shape(), TileShape{}, ClusterShape{}, hw_info, args.scheduler, scheduler_workspace);
 291 |     }
```
**EN:** This helper converts the public argument bundle into lower-level mainloop, epilogue, and scheduler parameters expected by the actual kernel implementation.
**CN:** 这个辅助函数把对外的参数包转换成底层内核实现真正需要的 mainloop、epilogue 和调度器参数。

### Lines 293-302

```cpp
 293 |     return {
 294 |       args.mode,
 295 |       problem_shapes,
 296 |       CollectiveMainloop::to_underlying_arguments(problem_shapes, args.mainloop, mainloop_workspace),
 297 |       CollectiveEpilogue::to_underlying_arguments(problem_shapes, args.epilogue, epilogue_workspace),
 298 |       hw_info,
 299 |       scheduler,
 300 |       workspace
 301 |     };
 302 |   }
```
**EN:** This helper converts the public argument bundle into lower-level mainloop, epilogue, and scheduler parameters expected by the actual kernel implementation.
**CN:** 这个辅助函数把对外的参数包转换成底层内核实现真正需要的 mainloop、epilogue 和调度器参数。

### Lines 304-322

```cpp
 304 |   static bool
 305 |   can_implement(Arguments const& args) {
 306 |     bool implementable = true;
 307 |     if constexpr (IsGroupedGemmKernel) {
 308 |       // Group GEMM currently only supports rank-3 problem shapes
 309 |       implementable &= (args.mode == GemmUniversalMode::kGrouped && rank(typename ProblemShape::UnderlyingProblemShape{}) == 3);
 310 |     }
 311 |     else {
 312 |       implementable &= (args.mode == GemmUniversalMode::kArray && rank(typename ProblemShape::UnderlyingProblemShape{}) == 4);
 313 |     }
 314 |     if (!implementable) {
 315 |       CUTLASS_TRACE_HOST("  CAN IMPLEMENT: Arguments or Problem Shape don't meet the requirements for Ptr Array Gemm or Grouped Gemm.\n");
 316 |       return implementable;
 317 |     }
 318 |     implementable &= CollectiveMainloop::can_implement(args.problem_shape, args.mainloop);
 319 |     implementable &= CollectiveEpilogue::can_implement(args.problem_shape, args.epilogue);
 320 |     implementable &= TileScheduler::can_implement(args.scheduler);
 321 |     return implementable;
 322 |   }
```
**EN:** This function checks whether the requested problem, layouts, and runtime options satisfy the constraints of the specialized kernel.
**CN:** 该函数检查请求的问题规模、布局和运行时选项是否满足当前特化内核的约束条件。

### Lines 324-327

```cpp
 324 |   static size_t
 325 |   get_workspace_size(Arguments const& args) {
 326 |     size_t workspace_size = 0;
 327 |     constexpr uint32_t NumEpilogueSubTiles = CollectiveEpilogue::get_store_pipe_increment(TileShape{});
```
**EN:** This block reports or initializes any auxiliary workspace needed by the kernel before launch.
**CN:** 这一段用于报告或初始化内核启动前所需的辅助工作区。

### Lines 329-335

```cpp
 329 |     // Get SM count if needed, otherwise use user supplied SM count
 330 |     int sm_count = args.hw_info.sm_count;
 331 |     if (sm_count <= 0) {
 332 |       CUTLASS_TRACE_HOST("  WARNING: Arguments do not include a valid SM count.\n"
 333 |           "  For optimal performance, populate the arguments KernelHardwareInfo struct with the SM count.");
 334 |       sm_count = KernelHardwareInfo::query_device_multiprocessor_count(args.hw_info.device_id);
 335 |     }
```
**EN:** This block declares or specializes `with`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `with`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 337-338

```cpp
 337 |     workspace_size += CollectiveEpilogue::get_workspace_size(args.problem_shape, args.epilogue, sm_count);
 338 |     workspace_size = round_nearest(workspace_size, MinTensorMapWorkspaceAlignment);
```
**EN:** This block reports or initializes any auxiliary workspace needed by the kernel before launch.
**CN:** 这一段用于报告或初始化内核启动前所需的辅助工作区。

### Lines 340-341

```cpp
 340 |     workspace_size += CollectiveMainloop::get_workspace_size(args.problem_shape, args.mainloop, sm_count);
 341 |     workspace_size = round_nearest(workspace_size, MinTensorMapWorkspaceAlignment);
```
**EN:** This block reports or initializes any auxiliary workspace needed by the kernel before launch.
**CN:** 这一段用于报告或初始化内核启动前所需的辅助工作区。

### Lines 343-345

```cpp
 343 |     workspace_size += TileScheduler::template get_workspace_size<typename ProblemShape::UnderlyingProblemShape, ElementAccumulator>(
 344 |       args.scheduler, typename ProblemShape::UnderlyingProblemShape{}, args.hw_info, NumMmaWarpGroups, NumEpilogueSubTiles);
 345 |     workspace_size = round_nearest(workspace_size, MinTensorMapWorkspaceAlignment);
```
**EN:** This block reports or initializes any auxiliary workspace needed by the kernel before launch.
**CN:** 这一段用于报告或初始化内核启动前所需的辅助工作区。

### Lines 347-348

```cpp
 347 |     return workspace_size;
 348 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 350-357

```cpp
 350 |   static cutlass::Status
 351 |   initialize_workspace(Arguments const& args, void* workspace = nullptr, cudaStream_t stream = nullptr,
 352 |     CudaHostAdapter* cuda_adapter = nullptr) {
 353 |     Status status = Status::kSuccess;
 354 |     uint8_t* workspace_ptr = reinterpret_cast<uint8_t*>(workspace);
 355 |     size_t workspace_offset = 0;
 356 |     constexpr uint32_t NumEpilogueSubTiles = CollectiveEpilogue::get_store_pipe_increment(TileShape{});
 357 |     static constexpr uint32_t NumAccumulatorMtxs = 1;
```
**EN:** This block reports or initializes any auxiliary workspace needed by the kernel before launch.
**CN:** 这一段用于报告或初始化内核启动前所需的辅助工作区。

### Lines 359-364

```cpp
 359 |     status = CollectiveEpilogue::initialize_workspace(args.problem_shape, args.epilogue, workspace_ptr + workspace_offset, stream, cuda_adapter);
 360 |     workspace_offset += CollectiveEpilogue::get_workspace_size(args.problem_shape, args.epilogue, args.hw_info.sm_count);
 361 |     workspace_offset = round_nearest(workspace_offset, MinTensorMapWorkspaceAlignment);
 362 |     if (status != Status::kSuccess) {
 363 |       return status;
 364 |     }
```
**EN:** This block reports or initializes any auxiliary workspace needed by the kernel before launch.
**CN:** 这一段用于报告或初始化内核启动前所需的辅助工作区。

### Lines 366-371

```cpp
 366 |     status = CollectiveMainloop::initialize_workspace(args.problem_shape, args.mainloop, workspace_ptr + workspace_offset, stream, cuda_adapter);
 367 |     workspace_offset += CollectiveMainloop::get_workspace_size(args.problem_shape, args.mainloop, args.hw_info.sm_count);
 368 |     workspace_offset = round_nearest(workspace_offset, MinTensorMapWorkspaceAlignment);
 369 |     if (status != Status::kSuccess) {
 370 |       return status;
 371 |     }
```
**EN:** This block reports or initializes any auxiliary workspace needed by the kernel before launch.
**CN:** 这一段用于报告或初始化内核启动前所需的辅助工作区。

### Lines 373-382

```cpp
 373 |     status = TileScheduler::template initialize_workspace<typename ProblemShape::UnderlyingProblemShape, ElementAccumulator>(
 374 |       args.scheduler, workspace_ptr + workspace_offset, stream, typename ProblemShape::UnderlyingProblemShape{}, args.hw_info, NumMmaWarpGroups, NumEpilogueSubTiles, NumAccumulatorMtxs, cuda_adapter);
 375 |     workspace_offset += TileScheduler::template get_workspace_size<typename ProblemShape::UnderlyingProblemShape, ElementAccumulator>(
 376 |       args.scheduler, typename ProblemShape::UnderlyingProblemShape{}, args.hw_info, NumMmaWarpGroups, NumEpilogueSubTiles);
 377 |     workspace_offset = round_nearest(workspace_offset, MinTensorMapWorkspaceAlignment);
 378 |     if (status != Status::kSuccess) {
 379 |       return status;
 380 |     }
 381 |     return status;
 382 |   }
```
**EN:** This block reports or initializes any auxiliary workspace needed by the kernel before launch.
**CN:** 这一段用于报告或初始化内核启动前所需的辅助工作区。

### Lines 384-401

```cpp
 384 |   // Computes the kernel launch grid shape based on runtime parameters
 385 |   static dim3
 386 |   get_grid_shape(Params const& params) {
 387 |     // Given device SM count, set grid size s.t. we do not launch more thread blocks than we can run concurrently
 388 |     TileSchedulerArguments args{};
 389 |     if constexpr (!std::is_const_v<decltype(args.max_swizzle_size)>) {
 390 |       args.max_swizzle_size = 1 << params.scheduler.log_swizzle_size_;
 391 |     }
 392 |     args.raster_order = params.scheduler.raster_order_ == TileScheduler::RasterOrder::AlongN ? TileScheduler::RasterOrderOptions::AlongN : TileScheduler::RasterOrderOptions::AlongM;
 393 |     dim3 grid_shape;
 394 |     if constexpr (IsGroupedGemmKernel) {
 395 |       grid_shape = TileScheduler::get_grid_shape(params.scheduler, params.problem_shape, TileShape{}, ClusterShape{}, params.hw_info, args);
 396 |     }
 397 |     else {
 398 |       grid_shape = TileScheduler::get_grid_shape(params.scheduler, params.problem_shape.get_host_problem_shape(), TileShape{}, ClusterShape{}, params.hw_info, args);
 399 |     }
 400 |     return grid_shape;
 401 |   }
```
**EN:** These launch helpers compute the CUDA/SYCL execution geometry from the problem shape, tile shape, and scheduling policy.
**CN:** 这些启动辅助函数根据问题规模、Tile 形状和调度策略计算 CUDA/SYCL 的执行网格。

### Lines 403-406

```cpp
 403 |   static dim3
 404 |   get_block_shape() {
 405 |     return dim3(MaxThreadsPerBlock, 1, 1);
 406 |   }
```
**EN:** These launch helpers compute the CUDA/SYCL execution geometry from the problem shape, tile shape, and scheduling policy.
**CN:** 这些启动辅助函数根据问题规模、Tile 形状和调度策略计算 CUDA/SYCL 的执行网格。

### Lines 408-412

```cpp
 408 |   CUTLASS_DEVICE
 409 |   void
 410 |   operator()(Params const& params, char* smem_buf) {
 411 |     using namespace cute;
 412 |     using X = Underscore;
```
**EN:** This is the device-side kernel body. It reconstructs shared storage, computes tile coordinates, runs the matrix-multiply mainloop, and finally applies the epilogue.
**CN:** 这是设备端的内核主体：它重建共享存储、计算 Tile 坐标、执行矩阵乘主循环，并在最后应用 epilogue。

### Lines 414-417

```cpp
 414 | #  if (defined(__CUDA_ARCH_FEAT_SM90_ALL) || defined(__CUDA_ARCH_FEAT_SM120_ALL) || defined(__CUDA_ARCH_FEAT_SM121_ALL) ||\
 415 |       CUDA_ARCH_CONDITIONAL_OR_FAMILY(1200) || CUDA_ARCH_CONDITIONAL_OR_FAMILY(1210))
 416 | #    define ENABLE_SM90_KERNEL_LEVEL 1
 417 | #  endif
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 419-422

```cpp
 419 | // Any Tensor Op MMA Atom in the ISA is arch conditional.
 420 | #if ! defined(ENABLE_SM90_KERNEL_LEVEL)
 421 |     printf("ERROR : Arch conditional MMA instruction used without targeting appropriate compute capability. Aborting.\n");
 422 | #else
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 424-428

```cpp
 424 |     // Preconditions
 425 |     static_assert(size(TiledMma{}) == 256, "Cooperative kernel must have TiledMMA operating using 256 threads.");
 426 |     static_assert(size<0>(TileShape{}) >= 128,
 427 |         "Cooperative kernel requires Tile Size to be greater than or equal to 128 along the M-dimension.");
 428 |     static_assert(NumMmaWarpGroups == 2, "Cooperative kernels currently only support NumMmaWarpGroups == 2");
```
**EN:** These compile-time checks enforce invariants such as problem-rank assumptions, supported scheduler tags, or required tensor-stride shapes.
**CN:** 这些编译期检查用于保证问题维度假设、可支持的调度器标签以及张量步长形状等不变式成立。

### Lines 430-433

```cpp
 430 |     if constexpr (cutlass::epilogue::collective::detail::sm90_is_ptr_array_tma_dispatch_policy_v<typename CollectiveEpilogue::DispatchPolicy>) {
 431 |       static_assert(NumMmaWarpGroups == CollectiveEpilogue::NumEpilogueWarpGroups,
 432 |                     "Tiled MmA does not match expected warp groups performing the epilogue");
 433 |     }
```
**EN:** These compile-time checks enforce invariants such as problem-rank assumptions, supported scheduler tags, or required tensor-stride shapes.
**CN:** 这些编译期检查用于保证问题维度假设、可支持的调度器标签以及张量步长形状等不变式成立。

### Lines 435-438

```cpp
 435 |     static_assert(cute::rank(InternalStrideA{}) == 3, "StrideA must be rank-3: [M, K, L]. If batch mode is not needed, set L stride to Int<0>.");
 436 |     static_assert(cute::rank(InternalStrideB{}) == 3, "StrideB must be rank-3: [N, K, L]. If batch mode is not needed, set L stride to Int<0>.");
 437 |     static_assert(cute::rank(InternalStrideC{}) == 3, "StrideC must be rank-3: [M, N, L]. If batch mode is not needed, set L stride to Int<0>.");
 438 |     static_assert(cute::rank(InternalStrideD{}) == 3, "StrideD must be rank-3: [M, N, L]. If batch mode is not needed, set L stride to Int<0>.");
```
**EN:** These compile-time checks enforce invariants such as problem-rank assumptions, supported scheduler tags, or required tensor-stride shapes.
**CN:** 这些编译期检查用于保证问题维度假设、可支持的调度器标签以及张量步长形状等不变式成立。

### Lines 440-451

```cpp
 440 |     /* In the Cooperative kernel, Consumer0 and Consumer1 collaborate on the same tile */
 441 |     enum class WarpGroupRole {
 442 |       Producer = 0,
 443 |       Consumer0 = 1,
 444 |       Consumer1 = 2
 445 |     };
 446 |     enum class ProducerWarpRole {
 447 |       Mainloop = 0,
 448 |       MainloopAux = 1,
 449 |       Epilogue = 2,
 450 |       Scheduler = 3
 451 |     };
```
**EN:** This block declares or specializes `WarpGroupRole`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `WarpGroupRole`，它是该头文件中承载某一层内核策略的核心类。

### Lines 453-454

```cpp
 453 |     // Kernel level shared memory storage
 454 |     SharedStorage& shared_storage = *reinterpret_cast<SharedStorage*>(smem_buf);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 456-464

```cpp
 456 |     auto scheduler = [&] () {
 457 |       // Group scheduler requires a different constructor that takes a response ptr
 458 |       if constexpr (cute::is_same_v<SchedulerTag, GroupScheduler>) {
 459 |         return TileScheduler{params.scheduler, shared_storage.scheduler_response};
 460 |       }
 461 |       else {
 462 |         return TileScheduler{params.scheduler};
 463 |       }
 464 |     } ();
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 466-468

```cpp
 466 |     // In a warp specialized kernel, collectives expose data movement and compute operations separately
 467 |     CollectiveMainloop collective_mainloop;
 468 |     CollectiveEpilogue collective_epilogue(params.epilogue, shared_storage.tensors.epilogue);
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 470-482

```cpp
 470 |     int thread_idx = int(threadIdx.x);
 471 |     int lane_idx = canonical_lane_idx();
 472 |     int warp_idx = canonical_warp_idx_sync();
 473 |     int warp_idx_in_warp_group = warp_idx % NumWarpsPerWarpGroup;
 474 |     int warp_group_thread_idx = thread_idx % NumThreadsPerWarpGroup;
 475 |     int mma_thread_idx = thread_idx % size(TiledMma{});
 476 |     auto warp_group_idx = canonical_warp_group_idx();
 477 |     auto warp_group_role = WarpGroupRole(warp_group_idx);
 478 |     auto producer_warp_role = ProducerWarpRole(warp_idx_in_warp_group);
 479 |     int lane_predicate = cute::elect_one_sync();
 480 |     uint32_t block_rank_in_cluster = cute::block_rank_in_cluster();
 481 | 
 482 |     // Note: Tma Descriptor Prefetch (from either const or param) is not applicable here
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 484-503

```cpp
 484 |     // TileScheduler pipeline
 485 |     using TileSchedulerPipeline = typename TileScheduler::Pipeline;
 486 |     typename TileSchedulerPipeline::Params tile_scheduler_pipeline_params;
 487 |     if constexpr (cute::is_same_v<SchedulerTag, GroupScheduler>) {
 488 |       if (warp_group_role == WarpGroupRole::Producer
 489 |         && producer_warp_role == ProducerWarpRole::Scheduler) {
 490 |         tile_scheduler_pipeline_params.role = TileSchedulerPipeline::ThreadCategory::Producer;
 491 |       }
 492 |       else {
 493 |         tile_scheduler_pipeline_params.role = TileSchedulerPipeline::ThreadCategory::Consumer;
 494 |       }
 495 |       tile_scheduler_pipeline_params.consumer_arv_count = NumMmaThreads
 496 |                                                         + NumThreadsPerWarp * (
 497 |                                                           1                                                           // Main DMA warp
 498 |                                                           + (collective_epilogue.is_producer_load_needed() ? 1 : 0)   // Epilog DMA warp
 499 |                                                           + (IsMainloopAuxiliaryLoadNeeded ? 1 : 0)                   // Aux DMA warp
 500 |                                                         );
 501 |       tile_scheduler_pipeline_params.producer_arv_count = 1;
 502 |     }
 503 |     TileSchedulerPipeline tile_scheduler_pipeline(shared_storage.pipelines.scheduler, tile_scheduler_pipeline_params);
```
**EN:** This alias block derives concise type names `TileSchedulerPipeline` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `TileSchedulerPipeline` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 504-519

```cpp
 504 |     // Mainloop Load pipeline
 505 |     using MainloopPipeline = typename CollectiveMainloop::MainloopPipeline;
 506 |     typename MainloopPipeline::Params mainloop_pipeline_params;
 507 |     if (warp_group_role == WarpGroupRole::Producer
 508 |       && (producer_warp_role == ProducerWarpRole::Mainloop
 509 |        || producer_warp_role == ProducerWarpRole::MainloopAux)) {
 510 |       mainloop_pipeline_params.role = MainloopPipeline::ThreadCategory::Producer;
 511 |     }
 512 |     if (warp_group_role == WarpGroupRole::Consumer0 || warp_group_role == WarpGroupRole::Consumer1) {
 513 |       mainloop_pipeline_params.role = MainloopPipeline::ThreadCategory::Consumer;
 514 |     }
 515 |     mainloop_pipeline_params.is_leader = warp_group_thread_idx == 0;
 516 |     mainloop_pipeline_params.num_consumers = NumMmaThreads;
 517 |     mainloop_pipeline_params.num_producers = NumProducerThreads;
 518 |     mainloop_pipeline_params.transaction_bytes = params.mainloop.tma_transaction_bytes;
 519 |     MainloopPipeline mainloop_pipeline(shared_storage.pipelines.mainloop, mainloop_pipeline_params, ClusterShape{});
```
**EN:** This alias block derives concise type names `MainloopPipeline` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `MainloopPipeline` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 521-536

```cpp
 521 |     // Epilogue Load pipeline
 522 |     using EpiLoadPipeline = typename CollectiveEpilogue::LoadPipeline;
 523 |     typename EpiLoadPipeline::Params epi_load_pipeline_params;
 524 |     if (warp_group_role == WarpGroupRole::Producer && producer_warp_role == ProducerWarpRole::Epilogue) {
 525 |       epi_load_pipeline_params.role = EpiLoadPipeline::ThreadCategory::Producer;
 526 |     }
 527 |     if (warp_group_role == WarpGroupRole::Consumer0 || warp_group_role == WarpGroupRole::Consumer1) {
 528 |       epi_load_pipeline_params.role = EpiLoadPipeline::ThreadCategory::Consumer;
 529 |     }
 530 |     epi_load_pipeline_params.dst_blockid = cute::block_rank_in_cluster();
 531 |     epi_load_pipeline_params.producer_arv_count = NumThreadsPerWarp;
 532 |     epi_load_pipeline_params.consumer_arv_count = size(TiledMma{});
 533 |     if constexpr (CollectiveEpilogue::RequiresTransactionBytes) {
 534 |       epi_load_pipeline_params.transaction_bytes = params.epilogue.tma_transaction_bytes;
 535 |     }
 536 |     EpiLoadPipeline epi_load_pipeline(shared_storage.pipelines.epi_load, epi_load_pipeline_params);
```
**EN:** This alias block derives concise type names `EpiLoadPipeline` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `EpiLoadPipeline` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 538-542

```cpp
 538 |     // Epilogue Store pipeline
 539 |     using EpiStorePipeline = typename CollectiveEpilogue::StorePipeline;
 540 |     typename EpiStorePipeline::Params epi_store_pipeline_params;
 541 |     epi_store_pipeline_params.always_wait = true;
 542 |     EpiStorePipeline epi_store_pipeline(epi_store_pipeline_params);
```
**EN:** This alias block derives concise type names `EpiStorePipeline` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `EpiStorePipeline` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 544-547

```cpp
 544 |     typename LoadWarpOrderBarrier::Params params_load_order_barrier;
 545 |     params_load_order_barrier.group_id = producer_warp_role == ProducerWarpRole::Mainloop ? 0 : 1;
 546 |     params_load_order_barrier.group_size = NumThreadsPerWarp;
 547 |     LoadWarpOrderBarrier load_order_barrier(shared_storage.pipelines.load_order, params_load_order_barrier);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 549-553

```cpp
 549 |     // Initialize starting pipeline states for the collectives
 550 |     // Epilogue store pipe is producer-only (consumer is TMA unit, waits via scoreboarding)
 551 |     typename TileSchedulerPipeline::PipelineState tile_scheduler_pipe_consumer_state;
 552 |     typename CollectiveMainloop::PipelineState mainloop_pipe_consumer_state;
 553 |     typename CollectiveEpilogue::LoadPipelineState epi_load_pipe_consumer_state;
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 555-560

```cpp
 555 |     // For the DMA Load (producer) we start with an opposite phase
 556 |     // i.e., we skip all waits since we know that the buffer is indeed empty
 557 |     PipelineState tile_scheduler_pipe_producer_state = cutlass::make_producer_start_state<TileSchedulerPipeline>();
 558 |     PipelineState mainloop_pipe_producer_state = cutlass::make_producer_start_state<MainloopPipeline>();
 559 |     PipelineState epi_load_pipe_producer_state = cutlass::make_producer_start_state<EpiLoadPipeline>();
 560 |     PipelineState epi_store_pipe_producer_state = cutlass::make_producer_start_state<EpiStorePipeline>();
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 562-573

```cpp
 562 |     auto cluster_wait_fn = [] () {
 563 |       // We need this to guarantee that the Pipeline init is visible
 564 |       // To all producers and consumer thread blocks in the Cluster
 565 |       if constexpr (size(ClusterShape{}) > 1) {
 566 |         cute::cluster_arrive_relaxed();
 567 |         return [] () { cute::cluster_wait(); };
 568 |       }
 569 |       else {
 570 |         syncthreads();
 571 |         return [] () {}; // do nothing
 572 |       }
 573 |     } ();
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 575-579

```cpp
 575 |     // Get the appropriate blocks for this thread block -- potential for thread block locality
 576 |     TiledMma tiled_mma;
 577 |     const auto blk_shape = TileShape{};                                                                // (BLK_M,BLK_N,BLK_K)
 578 |     const auto c_tile_count = CollectiveEpilogue::get_load_pipe_increment(blk_shape);
 579 |     const auto d_tile_count = CollectiveEpilogue::get_store_pipe_increment(blk_shape);
```
**EN:** This block continues the matrix-multiply engine, epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎、epilogue 输出路径相关逻辑。

### Lines 581-582

```cpp
 581 |     // Wait for all thread blocks in the Cluster
 582 |     cluster_wait_fn();
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 584-584

```cpp
 584 |     auto work_tile_info = scheduler.initial_work_tile_info(ClusterShape{});
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 586-589

```cpp
 586 |     if (not work_tile_info.is_valid()) {
 587 |       // When problem shapes are only on device, the grid launched may be larger than the total number of blocks across groups
 588 |       return;
 589 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 591-592

```cpp
 591 |     // Optionally append 1s until problem shape is rank-4 in case it is only rank-3 (MNK)
 592 |     auto problem_shape_MNKL = append<4>(params.problem_shape.get_problem_shape(work_tile_info.L_idx), 1);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 594-598

```cpp
 594 |     // Prepare and partition the input tensors. Expects a tuple of tensors where:
 595 |     // get<0>(load_inputs) is the tma tensor A after local tiling so that it has shape (BLK_M,BLK_K,m,k,l)
 596 |     // get<1>(load_inputs) is the tma tensor B after local tiling so that it has shape (BLK_N,BLK_K,n,k,l)
 597 |     auto load_inputs = collective_mainloop.load_init(problem_shape_MNKL, params.mainloop);
 598 |     static_assert(cute::tuple_size_v<decltype(load_inputs)> >= 2, "Output of load_init must have at least two elements (A, B)");
```
**EN:** These compile-time checks enforce invariants such as problem-rank assumptions, supported scheduler tags, or required tensor-stride shapes.
**CN:** 这些编译期检查用于保证问题维度假设、可支持的调度器标签以及张量步长形状等不变式成立。

### Lines 600-602

```cpp
 600 |     // Extract out partitioned A and B.
 601 |     Tensor gA_mkl = get<0>(load_inputs);
 602 |     Tensor gB_nkl = get<1>(load_inputs);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 604-605

```cpp
 604 |     // Get pipeline stage increments from tensor shapes
 605 |     auto k_tile_count = size<3>(gA_mkl);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 607-608

```cpp
 607 |     if (warp_group_role == WarpGroupRole::Producer) {
 608 |       cutlass::arch::warpgroup_reg_dealloc<LoadRegisterRequirement>();
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 610-629

```cpp
 610 |       if (producer_warp_role == ProducerWarpRole::Scheduler) {
 611 |         // GroupScheduler requires a producer warp to iterate over the group infos and push
 612 |         // the work tile infos to the downstream pipelines.
 613 |         if constexpr (cute::is_same_v<SchedulerTag, GroupScheduler>) {
 614 |           do {
 615 |             auto [next_work_tile_info, increment_pipe] = scheduler.advance_to_next_work(tile_scheduler_pipeline, tile_scheduler_pipe_producer_state);
 616 |             work_tile_info = next_work_tile_info;
 617 |             if (increment_pipe) {
 618 |               ++tile_scheduler_pipe_producer_state;
 619 |             }
 620 |           } while (work_tile_info.is_valid());
 621 |           tile_scheduler_pipeline.producer_tail(tile_scheduler_pipe_producer_state);
 622 |         }
 623 |       }
 624 |       // Mainloop Producer Warp
 625 |       else if (producer_warp_role == ProducerWarpRole::Mainloop) {
 626 |         int32_t curr_batch = idx2crd(work_tile_info.L_idx, shape<4>(gB_nkl)); // Usually just returns work_tile_info.L_idx;
 627 |         int32_t const mock_l_coord = 0;
 628 |         int32_t const sm_idx = BlockIdxX() + (BlockIdxY() * GridDimX());
 629 |         int32_t const sm_count = params.hw_info.sm_count;
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 631-632

```cpp
 631 |         // Fetch a copy of tensormaps for the CTA
 632 |         auto input_tensormaps = collective_mainloop.tensormaps_init(params.mainloop, shared_storage.tensormaps.mainloop, sm_count, sm_idx);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 634-645

```cpp
 634 |         // Update tensormap for the initial batch for the CTA
 635 |         collective_mainloop.tensormaps_perform_update(
 636 |           shared_storage.tensormaps.mainloop,
 637 |           params.mainloop,
 638 |           input_tensormaps,
 639 |           problem_shape_MNKL,
 640 |           curr_batch
 641 |         );
 642 |         // Ensure warp is converged before issuing tensormap fence release
 643 |         syncwarp();
 644 |         // Entire warp must do this (i.e. it's aligned)
 645 |         collective_mainloop.tensormaps_cp_fence_release(shared_storage.tensormaps.mainloop, input_tensormaps);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 647-658

```cpp
 647 |         bool do_load_order_arrive = true;
 648 |         bool did_batch_change = true;
 649 |         do {
 650 |           if (!TileScheduler::valid_warpgroup_in_work_tile(work_tile_info)) {
 651 |             auto [next_work_tile_info, increment_pipe] = scheduler.fetch_next_work(
 652 |                 work_tile_info, tile_scheduler_pipeline, tile_scheduler_pipe_consumer_state);
 653 |             work_tile_info = next_work_tile_info;
 654 |             if (increment_pipe) {
 655 |               ++tile_scheduler_pipe_consumer_state;
 656 |             }
 657 |             continue;
 658 |           }
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 660-663

```cpp
 660 |           // Compute m_coord, n_coord, l_coord with the post-tiled m-shape and n-shape
 661 |           auto m_coord = idx2crd(work_tile_info.M_idx, shape<2>(gA_mkl));
 662 |           auto n_coord = idx2crd(work_tile_info.N_idx, shape<2>(gB_nkl));
 663 |           auto blk_coord = make_coord(m_coord, n_coord, _, mock_l_coord);
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 665-668

```cpp
 665 |           // Get the number of K tiles to compute for this work as well as the starting K tile offset of the work.
 666 |           auto work_k_tile_count = TileScheduler::get_work_k_tile_count(work_tile_info, problem_shape_MNKL, blk_shape);
 667 |           auto work_k_tile_start = TileScheduler::get_work_k_tile_start(work_tile_info);
 668 |           auto k_tile_iter = cute::make_coord_iterator(idx2crd(work_k_tile_start, shape<3>(gA_mkl)), shape<3>(gA_mkl));
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 670-673

```cpp
 670 |           if (did_batch_change) {
 671 |             load_inputs = collective_mainloop.tensors_perform_update(load_inputs, params.mainloop, problem_shape_MNKL, curr_batch);
 672 |             collective_mainloop.tensormaps_fence_acquire(input_tensormaps);
 673 |           }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 675-688

```cpp
 675 |           collective_mainloop.load(
 676 |             params.mainloop,
 677 |             mainloop_pipeline,
 678 |             mainloop_pipe_producer_state,
 679 |             load_inputs,
 680 |             input_tensormaps,
 681 |             blk_coord,
 682 |             k_tile_iter, work_k_tile_count,
 683 |             lane_idx,
 684 |             block_rank_in_cluster,
 685 |             shared_storage.tensors.mainloop
 686 |           );
 687 |           // Pipeline state is only advanced if there are K tiles to compute
 688 |           mainloop_pipe_producer_state.advance(work_k_tile_count);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 690-694

```cpp
 690 |           // Signal for the epilogue load warp to begin
 691 |           if (do_load_order_arrive) {
 692 |             load_order_barrier.arrive();
 693 |             do_load_order_arrive = false;
 694 |           }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 696-715

```cpp
 696 |           // Get next work tile
 697 |           auto [next_work_tile_info, increment_pipe] = scheduler.fetch_next_work(work_tile_info, tile_scheduler_pipeline, tile_scheduler_pipe_consumer_state);
 698 |           work_tile_info = next_work_tile_info;
 699 |           if (increment_pipe) {
 700 |             ++tile_scheduler_pipe_consumer_state;
 701 |           }
 702 |           auto next_batch = idx2crd(work_tile_info.L_idx, shape<4>(gB_nkl)); // Usually just returns work_tile_info.L_idx
 703 |           did_batch_change = next_batch != curr_batch;
 704 |           if (work_tile_info.is_valid() && did_batch_change) {
 705 |             curr_batch = next_batch;
 706 |             if constexpr (IsGroupedGemmKernel) {
 707 |               problem_shape_MNKL = append<4>(params.problem_shape.get_problem_shape(curr_batch), 1);
 708 |             }
 709 |             collective_mainloop.tensormaps_perform_update(
 710 |               shared_storage.tensormaps.mainloop,
 711 |               params.mainloop,
 712 |               input_tensormaps,
 713 |               problem_shape_MNKL,
 714 |               curr_batch
 715 |             );
```
**EN:** This line assembles the final kernel type by combining the chosen MMA/mainloop object, epilogue object, and threadblock scheduling policy.
**CN:** 这一行通过组合选定的 MMA/主循环对象、epilogue 对象以及线程块调度策略，装配出最终的内核类型。

### Lines 716-721

```cpp
 716 |             // Ensure warp is converged before issuing tensor replace
 717 |             syncwarp();
 718 |             // Entire warp must do this (i.e. it's aligned)
 719 |             collective_mainloop.tensormaps_cp_fence_release(shared_storage.tensormaps.mainloop, input_tensormaps);
 720 |           }
 721 |         } while (work_tile_info.is_valid()); // Scheduler work fetch loop
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 723-729

```cpp
 723 |         // Make sure all Consumer Warp Groups have been waited upon
 724 |         collective_mainloop.load_tail(mainloop_pipeline, mainloop_pipe_producer_state);
 725 |       } // Mainloop Producer Warp End
 726 |       else if (producer_warp_role == ProducerWarpRole::MainloopAux) {
 727 |         if constexpr (IsMainloopAuxiliaryLoadNeeded) {
 728 |           int32_t curr_batch = idx2crd(work_tile_info.L_idx, shape<4>(gB_nkl)); // Usually just returns work_tile_info.L_idx;
 729 |           int32_t const mock_l_coord = 0;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 731-740

```cpp
 731 |           bool did_batch_change = true;
 732 |           do {
 733 |             if (!TileScheduler::valid_warpgroup_in_work_tile(work_tile_info)) {
 734 |               auto [next_work_tile_info, increment_pipe] = scheduler.fetch_next_work(work_tile_info, tile_scheduler_pipeline, tile_scheduler_pipe_consumer_state);
 735 |               work_tile_info = next_work_tile_info;
 736 |               if (increment_pipe) {
 737 |                 ++tile_scheduler_pipe_consumer_state;
 738 |               }
 739 |               continue;
 740 |             }
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 742-745

```cpp
 742 |             // Compute m_coord, n_coord, l_coord with the post-tiled m-shape and n-shape
 743 |             auto m_coord = idx2crd(work_tile_info.M_idx, shape<2>(gA_mkl));
 744 |             auto n_coord = idx2crd(work_tile_info.N_idx, shape<2>(gB_nkl));
 745 |             auto blk_coord = make_coord(m_coord, n_coord, _, mock_l_coord);
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 747-750

```cpp
 747 |             // Get the number of K tiles to compute for this work as well as the starting K tile offset of the work.
 748 |             auto work_k_tile_count = TileScheduler::get_work_k_tile_count(work_tile_info, problem_shape_MNKL, blk_shape);
 749 |             auto work_k_tile_start = TileScheduler::get_work_k_tile_start(work_tile_info);
 750 |             auto k_tile_iter = cute::make_coord_iterator(idx2crd(work_k_tile_start, shape<3>(gA_mkl)), shape<3>(gA_mkl));
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 752-754

```cpp
 752 |             if (did_batch_change) {
 753 |               load_inputs = collective_mainloop.tensors_perform_update(load_inputs, params.mainloop, problem_shape_MNKL, curr_batch);
 754 |             }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 756-766

```cpp
 756 |             collective_mainloop.load_auxiliary(
 757 |               params.mainloop,
 758 |               mainloop_pipeline,
 759 |               mainloop_pipe_producer_state,
 760 |               load_inputs,
 761 |               blk_coord,
 762 |               k_tile_iter, work_k_tile_count,
 763 |               lane_idx,
 764 |               block_rank_in_cluster,
 765 |               shared_storage.tensors.mainloop
 766 |             );
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 768-769

```cpp
 768 |             // Update starting pipeline state for the next tile
 769 |             mainloop_pipe_producer_state.advance(work_k_tile_count);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 771-791

```cpp
 771 |             // Get next work tile
 772 |             auto [next_work_tile_info, increment_pipe] = scheduler.fetch_next_work(work_tile_info, tile_scheduler_pipeline, tile_scheduler_pipe_consumer_state);
 773 |             work_tile_info = next_work_tile_info;
 774 |             if (increment_pipe) {
 775 |               ++tile_scheduler_pipe_consumer_state;
 776 |             }
 777 |             auto next_batch = idx2crd(work_tile_info.L_idx, shape<4>(gB_nkl)); // Usually just returns work_tile_info.L_idx
 778 |             did_batch_change = next_batch != curr_batch;
 779 |             if (work_tile_info.is_valid() && did_batch_change) {
 780 |               curr_batch = next_batch;
 781 |               if constexpr (IsGroupedGemmKernel) {
 782 |                 problem_shape_MNKL = append<4>(params.problem_shape.get_problem_shape(curr_batch), 1);
 783 |               }
 784 |             }
 785 |           } while (work_tile_info.is_valid()); // Scheduler work fetch loop
 786 |         } // End of auxiliary load needed check
 787 |       } // Mainloop Auxiliary Load Producer Warp End
 788 |       // Epilogue Producer Warp
 789 |       else if (producer_warp_role == ProducerWarpRole::Epilogue && collective_epilogue.is_producer_load_needed()) {
 790 |         int32_t const sm_idx = blockIdx.x + (blockIdx.y * gridDim.x);
 791 |         int32_t const sm_count = params.hw_info.sm_count;
```
**EN:** This line assembles the final kernel type by combining the chosen MMA/mainloop object, epilogue object, and threadblock scheduling policy.
**CN:** 这一行通过组合选定的 MMA/主循环对象、epilogue 对象以及线程块调度策略，装配出最终的内核类型。

### Lines 793-793

```cpp
 793 |         auto epi_load_tensormap = get<0>(collective_epilogue.load_init(params.epilogue, shared_storage.tensormaps.epilogue, sm_count, sm_idx));
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 795-796

```cpp
 795 |         bool did_batch_change = true;
 796 |         constexpr bool IsEpiLoad = true;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 798-805

```cpp
 798 |         collective_epilogue.template tensormaps_perform_update<IsEpiLoad>(
 799 |           shared_storage.tensormaps.epilogue,
 800 |           params.epilogue,
 801 |           epi_load_tensormap,
 802 |           problem_shape_MNKL,
 803 |           work_tile_info.L_idx,
 804 |           0
 805 |         );
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 807-809

```cpp
 807 |         // Converge before issuing tensormap fence release since fence is aligned
 808 |         __syncwarp();
 809 |         collective_epilogue.template tensormaps_cp_fence_release<IsEpiLoad>(shared_storage.tensormaps.epilogue, epi_load_tensormap, 0);
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 811-811

```cpp
 811 |         load_order_barrier.wait();
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 813-814

```cpp
 813 |         do {
 814 |           int32_t curr_batch = work_tile_info.L_idx;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 816-817

```cpp
 816 |           // Get next work tile
 817 |           auto [next_work_tile_info, increment_pipe] = scheduler.fetch_next_work(work_tile_info, tile_scheduler_pipeline, tile_scheduler_pipe_consumer_state);
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 819-822

```cpp
 819 |           if (TileScheduler::compute_epilogue(work_tile_info, params.scheduler)) {
 820 |             if constexpr (IsGroupedGemmKernel) {
 821 |               problem_shape_MNKL = append<4>(params.problem_shape.get_problem_shape(work_tile_info.L_idx), 1);
 822 |             }
```
**EN:** This line assembles the final kernel type by combining the chosen MMA/mainloop object, epilogue object, and threadblock scheduling policy.
**CN:** 这一行通过组合选定的 MMA/主循环对象、epilogue 对象以及线程块调度策略，装配出最终的内核类型。

### Lines 824-828

```cpp
 824 |             // Compute m_coord, n_coord, l_coord with the post-tiled m-shape and n-shape
 825 |             auto m_coord = idx2crd(work_tile_info.M_idx, shape<2>(gA_mkl));
 826 |             auto n_coord = idx2crd(work_tile_info.N_idx, shape<2>(gB_nkl));
 827 |             auto l_coord = idx2crd(work_tile_info.L_idx, shape<4>(gB_nkl));
 828 |             auto blk_coord = make_coord(m_coord, n_coord, _, l_coord);
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 830-832

```cpp
 830 |             if (did_batch_change) {
 831 |               collective_epilogue.template tensormaps_fence_acquire<IsEpiLoad>(epi_load_tensormap);
 832 |             }
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 834-846

```cpp
 834 |             epi_load_pipe_producer_state = collective_epilogue.load(
 835 |               epi_load_pipeline,
 836 |               epi_load_pipe_producer_state,
 837 |               problem_shape_MNKL,
 838 |               blk_shape,
 839 |               blk_coord,
 840 |               tiled_mma,
 841 |               lane_idx,
 842 |               shared_storage.tensors.epilogue,
 843 |               epi_load_tensormap,
 844 |               work_tile_info.reduction_subtile_idx()
 845 |             );
 846 |           }
```
**EN:** This block continues the matrix-multiply engine, epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎、epilogue 输出路径相关逻辑。

### Lines 848-852

```cpp
 848 |           work_tile_info = next_work_tile_info;
 849 |           if (increment_pipe) {
 850 |             ++tile_scheduler_pipe_consumer_state;
 851 |           }
 852 |           did_batch_change = curr_batch != work_tile_info.L_idx;
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 854-857

```cpp
 854 |           if (work_tile_info.is_valid() && did_batch_change) {
 855 |             if constexpr (IsGroupedGemmKernel) {
 856 |               problem_shape_MNKL = append<4>(params.problem_shape.get_problem_shape(work_tile_info.L_idx), 1);
 857 |             }
```
**EN:** This line assembles the final kernel type by combining the chosen MMA/mainloop object, epilogue object, and threadblock scheduling policy.
**CN:** 这一行通过组合选定的 MMA/主循环对象、epilogue 对象以及线程块调度策略，装配出最终的内核类型。

### Lines 859-868

```cpp
 859 |             // tensormap update
 860 |             {
 861 |               collective_epilogue.template tensormaps_perform_update<IsEpiLoad>(
 862 |                 shared_storage.tensormaps.epilogue,
 863 |                 params.epilogue,
 864 |                 epi_load_tensormap,
 865 |                 problem_shape_MNKL,
 866 |                 work_tile_info.L_idx,
 867 |                 0
 868 |               );
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 870-874

```cpp
 870 |               // Converge before issuing tensormap fence release since fence is aligned
 871 |               __syncwarp();
 872 |               collective_epilogue.template tensormaps_cp_fence_release<IsEpiLoad>(shared_storage.tensormaps.epilogue, epi_load_tensormap, 0);
 873 |             }
 874 |           }
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 876-876

```cpp
 876 |         } while (work_tile_info.is_valid()); // Scheduler work fetch loop
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 878-881

```cpp
 878 |         // Make sure all Consumer Warp Groups have been waited upon
 879 |         collective_epilogue.load_tail(epi_load_pipeline, epi_load_pipe_producer_state);
 880 |       } // Epilogue Producer Warp End
 881 |     } // Producer Warp Group End
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 883-884

```cpp
 883 |     else if (warp_group_role == WarpGroupRole::Consumer0 || warp_group_role == WarpGroupRole::Consumer1) {
 884 |       cutlass::arch::warpgroup_reg_alloc<MmaRegisterRequirement>();
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 886-887

```cpp
 886 |       // Index of warp group within consumer warp groups
 887 |       int consumer_warp_group_idx = warp_group_role == WarpGroupRole::Consumer0 ? 0 : 1;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 889-894

```cpp
 889 |       int32_t const sm_idx = blockIdx.x + (blockIdx.y * gridDim.x);
 890 |       int32_t const sm_count = params.hw_info.sm_count;
 891 |       // Do we potentially issue tail arrives for TMA stores, if epilogue load is waiting for it
 892 |       bool do_store_tail = false;
 893 |       // Get a copy of tensormaps
 894 |       auto epi_store_tensormap = get<0>(collective_epilogue.store_init(params.epilogue, shared_storage.tensormaps.epilogue, sm_count, sm_idx, consumer_warp_group_idx));
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 896-897

```cpp
 896 |       bool did_batch_change = true;
 897 |       constexpr bool IsEpiLoad = false;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 899-907

```cpp
 899 |       if (warp_idx_in_warp_group == 0) {
 900 |         collective_epilogue.template tensormaps_perform_update<IsEpiLoad>(
 901 |           shared_storage.tensormaps.epilogue,
 902 |           params.epilogue,
 903 |           epi_store_tensormap,
 904 |           problem_shape_MNKL,
 905 |           work_tile_info.L_idx,
 906 |           consumer_warp_group_idx
 907 |         );
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 909-914

```cpp
 909 |         // Converge before issuing tensormap fence release since fence is aligned
 910 |         __syncwarp();
 911 |         collective_epilogue.template tensormaps_cp_fence_release<IsEpiLoad>(shared_storage.tensormaps.epilogue,
 912 |                                                                     epi_store_tensormap,
 913 |                                                                     consumer_warp_group_idx);
 914 |       }
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 916-919

```cpp
 916 |       do {
 917 |         if constexpr (IsGroupedGemmKernel) {
 918 |           problem_shape_MNKL = append<4>(params.problem_shape.get_problem_shape(work_tile_info.L_idx), 1);
 919 |         }
```
**EN:** This line assembles the final kernel type by combining the chosen MMA/mainloop object, epilogue object, and threadblock scheduling policy.
**CN:** 这一行通过组合选定的 MMA/主循环对象、epilogue 对象以及线程块调度策略，装配出最终的内核类型。

### Lines 921-921

```cpp
 921 |         int32_t curr_batch = work_tile_info.L_idx;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 923-928

```cpp
 923 |         // Compute m_coord, n_coord, l_coord with the post-tiled m-shape and n-shape
 924 |         auto m_coord = idx2crd(work_tile_info.M_idx, shape<2>(gA_mkl));
 925 |         auto n_coord = idx2crd(work_tile_info.N_idx, shape<2>(gB_nkl));
 926 |         auto l_coord = idx2crd(work_tile_info.L_idx, shape<4>(gB_nkl));
 927 |         auto blk_coord = make_coord(m_coord, n_coord, _, l_coord);
 928 |         auto work_k_tile_count = TileScheduler::get_work_k_tile_count(work_tile_info, problem_shape_MNKL, blk_shape);
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 930-933

```cpp
 930 |         // Allocate the accumulators for the (M,N) blk_shape
 931 |         //
 932 |         // MSVC CTAD breaks if we say "Tensor" here, so we use "auto" instead.
 933 |         auto accumulators = partition_fragment_C(tiled_mma, take<0,2>(blk_shape));               // (MMA,MMA_M,MMA_N)
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 935-935

```cpp
 935 |         if (TileScheduler::valid_warpgroup_in_work_tile(work_tile_info)) {
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 937-945

```cpp
 937 |           collective_mainloop.mma(
 938 |             mainloop_pipeline,
 939 |             mainloop_pipe_consumer_state,
 940 |             accumulators,
 941 |             work_k_tile_count,
 942 |             mma_thread_idx,
 943 |             shared_storage.tensors.mainloop,
 944 |             params.mainloop
 945 |           );
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 947-952

```cpp
 947 |           // Make sure the math instructions are done and free buffers before entering the epilogue
 948 |           collective_mainloop.mma_tail(
 949 |             mainloop_pipeline,
 950 |             mainloop_pipe_consumer_state,
 951 |             work_k_tile_count
 952 |           );
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 954-956

```cpp
 954 |           // Update starting mainloop pipeline state for the next tile
 955 |           mainloop_pipe_consumer_state.advance(work_k_tile_count);
 956 |         }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 958-960

```cpp
 958 |         // Perform reduction across splits, if needed
 959 |         TileScheduler::fixup(
 960 |           params.scheduler, work_tile_info, accumulators, NumMmaWarpGroups, consumer_warp_group_idx);
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 962-964

```cpp
 962 |         if (did_batch_change) {
 963 |           collective_epilogue.template tensormaps_fence_acquire<IsEpiLoad>(epi_store_tensormap);
 964 |         }
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 966-966

```cpp
 966 |         if (TileScheduler::compute_epilogue(work_tile_info, params.scheduler)) {
```
**EN:** This block continues the epilogue/output path, scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径、调度逻辑相关逻辑。

### Lines 968-984

```cpp
 968 |           // Epilogue and write to gD
 969 |           auto [epi_load_pipe_consumer_state_next, epi_store_pipe_producer_state_next] =
 970 |           collective_epilogue.store(
 971 |             epi_load_pipeline,
 972 |             epi_load_pipe_consumer_state,
 973 |             epi_store_pipeline,
 974 |             epi_store_pipe_producer_state,
 975 |             problem_shape_MNKL,
 976 |             blk_shape,
 977 |             blk_coord,
 978 |             accumulators,
 979 |             tiled_mma,
 980 |             mma_thread_idx,
 981 |             shared_storage.tensors.epilogue,
 982 |             epi_store_tensormap,
 983 |             work_tile_info.reduction_subtile_idx()
 984 |           );
```
**EN:** This block continues the matrix-multiply engine, epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎、epilogue 输出路径相关逻辑。

### Lines 986-989

```cpp
 986 |           epi_load_pipe_consumer_state = epi_load_pipe_consumer_state_next;
 987 |           epi_store_pipe_producer_state = epi_store_pipe_producer_state_next;
 988 |           do_store_tail = true;
 989 |         }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 991-996

```cpp
 991 |         // Get next work tile
 992 |         auto [next_work_tile_info, increment_pipe] = scheduler.fetch_next_work(work_tile_info, tile_scheduler_pipeline, tile_scheduler_pipe_consumer_state);
 993 |         work_tile_info = next_work_tile_info;
 994 |         if (increment_pipe) {
 995 |           ++tile_scheduler_pipe_consumer_state;
 996 |         }
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 998-1011

```cpp
 998 |         did_batch_change = curr_batch != work_tile_info.L_idx;
 999 |         if (work_tile_info.is_valid() && did_batch_change) {
1000 |           if constexpr (IsGroupedGemmKernel) {
1001 |             problem_shape_MNKL = append<4>(params.problem_shape.get_problem_shape(work_tile_info.L_idx), 1);
1002 |           }
1003 |           if (warp_idx_in_warp_group == 0) {
1004 |             collective_epilogue.template tensormaps_perform_update<IsEpiLoad>(
1005 |               shared_storage.tensormaps.epilogue,
1006 |               params.epilogue,
1007 |               epi_store_tensormap,
1008 |               problem_shape_MNKL,
1009 |               work_tile_info.L_idx,
1010 |               consumer_warp_group_idx
1011 |             );
```
**EN:** This line assembles the final kernel type by combining the chosen MMA/mainloop object, epilogue object, and threadblock scheduling policy.
**CN:** 这一行通过组合选定的 MMA/主循环对象、epilogue 对象以及线程块调度策略，装配出最终的内核类型。

### Lines 1013-1019

```cpp
1013 |             // Converge before issuing tensormap fence release since fence is aligned
1014 |             __syncwarp();
1015 |             collective_epilogue.template tensormaps_cp_fence_release<IsEpiLoad>(shared_storage.tensormaps.epilogue,
1016 |                                                                        epi_store_tensormap,
1017 |                                                                        consumer_warp_group_idx);
1018 |           }
1019 |         }
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 1021-1021

```cpp
1021 |       } while (work_tile_info.is_valid()); // Scheduler work fetch loop
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 1023-1035

```cpp
1023 |       // Cooperative only needs TMA to complete at the very end of the kernel
1024 |       if (do_store_tail) {
1025 |         collective_epilogue.store_tail(
1026 |           epi_load_pipeline,
1027 |           epi_load_pipe_consumer_state,
1028 |           epi_store_pipeline,
1029 |           epi_store_pipe_producer_state
1030 |         );
1031 |       }
1032 |     } // Consumer Warp Groups End
1033 | #endif
1034 |   }
1035 | };
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 1039-1039

```cpp
1039 | } // namespace cutlass::gemm::kernel
```
**EN:** These lines open or close the nested CUTLASS namespaces so the declarations stay scoped to the GEMM kernel subsystem.
**CN:** 这些代码行用于打开或关闭 CUTLASS 的嵌套命名空间，使声明被限制在 GEMM 内核子系统内。

## Key Concepts / 关键概念
- Template metaprogramming / 模板元编程
- SFINAE-based specialization / 基于 SFINAE 的特化
- Universal GEMM interface / 通用 GEMM 接口
- Tile scheduling / Tile 调度
- Persistent scheduling / 持久化调度
- Grouped problem handling / 分组问题处理
- Tensor Memory Accelerator / 张量内存加速器
- Warp-specialized execution / Warp 专用化执行

## Dependencies / 依赖关系
- **Included headers / 引入头文件:** `cutlass/cutlass.h`, `cutlass/workspace.h`, `cutlass/fast_math.h`, `cutlass/kernel_hardware_info.hpp`, `cute/arch/cluster_sm90.hpp`, `cutlass/arch/reg_reconfig.h`, `cutlass/arch/mma_sm90.h`, `cutlass/epilogue/collective/detail.hpp`, `cutlass/gemm/gemm.h`, `cutlass/gemm/dispatch_policy.hpp`, `cutlass/gemm/kernel/gemm_universal_decl.h`, `cutlass/gemm/kernel/tile_scheduler.hpp`, ... (+6 more)
- **Subsystems / 子系统:** CuTe tensor utilities / CuTe 张量工具, Architecture tags and intrinsics / 架构标签与内建操作, Epilogue/output pipeline / Epilogue 输出流水, Tile schedulers / Tile 调度器, Collective mainloop / 集体主循环, Collective epilogue / 集体 epilogue
- **Peer kernel headers / 同级内核头文件:** `cutlass/gemm/kernel/gemm_universal_decl.h`, `cutlass/gemm/kernel/tile_scheduler.hpp`, `cutlass/gemm/kernel/sm90_tile_scheduler.hpp`, `cutlass/gemm/kernel/sm90_tile_scheduler_group.hpp`
