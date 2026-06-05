# sm90_tile_scheduler_stream_k.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `include/cutlass/gemm/kernel/sm90_tile_scheduler_stream_k.hpp`
- **Purpose / 用途 (EN):** Implements an SM90-specific GEMM kernel path and its launch-time wiring.
- **Purpose / 用途 (CN):** 实现面向 SM90 的 GEMM 内核路径及其启动期装配逻辑。
- **Line count / 行数:** 1113

## Line-by-Line Analysis / 逐行分析
> The source is grouped into contiguous significant line ranges for readability. / 为了便于阅读，下面按连续的重要代码区间进行分析。

### Lines 1-30

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
```
**EN:** This opening comment records the license, copyright ownership, and redistribution conditions that apply to the header.
**CN:** 开头注释说明了该头文件适用的许可证、版权归属以及再分发条件。

### Lines 32-32

```cpp
  32 | #pragma once
```
**EN:** This directive makes the header idempotent so repeated inclusion does not create duplicate definitions.
**CN:** 该指令让头文件只生效一次，避免重复包含导致的重复定义。

### Lines 34-40

```cpp
  34 | #include "cutlass/barrier.h"
  35 | #include "cutlass/block_striped.h"
  36 | #include "cutlass/fast_math.h"
  37 | #include "cutlass/gemm/kernel/sm90_tile_scheduler.hpp"
  38 | #include "cutlass/kernel_hardware_info.hpp"
  39 | #include "cute/layout.hpp"
  40 | #include "cute/tensor.hpp"
```
**EN:** This include block imports `cutlass/barrier.h`, `cutlass/block_striped.h`, `cutlass/fast_math.h`, `cutlass/gemm/kernel/sm90_tile_scheduler.hpp`, `cutlass/kernel_hardware_info.hpp`, `cute/layout.hpp`, ... (+1 more), providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/barrier.h`, `cutlass/block_striped.h`, `cutlass/fast_math.h`, `cutlass/gemm/kernel/sm90_tile_scheduler.hpp`, `cutlass/kernel_hardware_info.hpp`, `cute/layout.hpp`, ... (+1 more)，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 42-42

```cpp
  42 | namespace cutlass::gemm::kernel::detail {
```
**EN:** These lines open or close the nested CUTLASS namespaces so the declarations stay scoped to the GEMM kernel subsystem.
**CN:** 这些代码行用于打开或关闭 CUTLASS 的嵌套命名空间，使声明被限制在 GEMM 内核子系统内。

### Lines 44-52

```cpp
  44 | // Persistent Thread Block (TB) scheduler leveraging stream-K decomposition
  45 | template <
  46 |   class TileShape,
  47 |   class ClusterShape
  48 | >
  49 | class PersistentTileSchedulerSm90StreamK {
  50 |   //
  51 |   // Data members
  52 |   //
```
**EN:** This block declares or specializes `TileShape`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `TileShape`，它是该头文件中承载某一层内核策略的核心类。

### Lines 54-55

```cpp
  54 | private:
  55 |   using UnderlyingScheduler = PersistentTileSchedulerSm90;
```
**EN:** This alias block derives concise type names `UnderlyingScheduler` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `UnderlyingScheduler` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 57-59

```cpp
  57 | private:
  58 |   using UnderlyingArguments = typename UnderlyingScheduler::Arguments;
  59 |   using UnderlyingParams = typename UnderlyingScheduler::Params;
```
**EN:** This alias block derives concise type names `UnderlyingArguments`, `UnderlyingParams` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `UnderlyingArguments`, `UnderlyingParams` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 61-63

```cpp
  61 |   dim3 block_id_in_cluster_;
  62 |   uint64_t current_work_linear_idx_ = 0;
  63 |   uint32_t unit_iter_start_ = 0;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 65-65

```cpp
  65 | public:
```
**EN:** This comment-only block adds explanatory context for the surrounding implementation, such as layout assumptions, scheduling notes, or design rationale.
**CN:** 这一段纯注释内容为周围实现补充了解释性背景，例如布局假设、调度说明或设计动机。

### Lines 67-69

```cpp
  67 |   using RasterOrder = UnderlyingScheduler::RasterOrder;
  68 |   using RasterOrderOptions = UnderlyingScheduler::RasterOrderOptions;
  69 |   static constexpr bool IsDynamicPersistent = false;
```
**EN:** This alias block derives concise type names `RasterOrder`, `RasterOrderOptions` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `RasterOrder`, `RasterOrderOptions` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 71-75

```cpp
  71 |   using Pipeline = PipelineEmpty;
  72 |   using PipelineStorage = typename Pipeline::SharedStorage;
  73 |   using ThrottlePipeline = PipelineEmpty;
  74 |   using ThrottlePipelineStorage = typename ThrottlePipeline::SharedStorage;
  75 |   struct CLCResponse {};
```
**EN:** This alias block derives concise type names `Pipeline`, `PipelineStorage`, `ThrottlePipeline`, `ThrottlePipelineStorage` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Pipeline`, `PipelineStorage`, `ThrottlePipeline`, `ThrottlePipelineStorage` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 77-82

```cpp
  77 |   class SharedStorage {
  78 |   public:
  79 |     CUTLASS_DEVICE PipelineStorage pipeline() { return PipelineStorage{}; }
  80 |     CUTLASS_DEVICE ThrottlePipelineStorage throttle_pipeline() { return ThrottlePipelineStorage{}; }
  81 |     CUTLASS_DEVICE CLCResponse* data() { return nullptr; }
  82 |   };
```
**EN:** This block defines the shared-memory layout used by the kernel, typically combining mainloop buffers, epilogue buffers, or pipeline state into one structured storage object.
**CN:** 这一段定义了内核使用的共享内存布局，通常会把主循环缓冲区、epilogue 缓冲区或流水状态组合到一个结构化存储对象中。

### Lines 84-85

```cpp
  84 |   // Use a dummy barrier manager to simply get the type used to store the barrier
  85 |   using BarrierType = typename NamedBarrierManager<1>::T;
```
**EN:** This alias block derives concise type names `BarrierType` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `BarrierType` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 87-89

```cpp
  87 |   using Params = PersistentTileSchedulerSm90StreamKParams;
  88 |   using ReductionMode = Params::ReductionMode;
  89 |   using DecompositionMode = Params::DecompositionMode;
```
**EN:** This alias block derives concise type names `Params`, `ReductionMode`, `DecompositionMode` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Params`, `ReductionMode`, `DecompositionMode` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 91-95

```cpp
  91 |   struct WorkTileInfo {
  92 |     int32_t M_idx = 0;
  93 |     int32_t N_idx = 0;
  94 |     int32_t K_idx = 0;
  95 |     int32_t L_idx = 0;
```
**EN:** This block declares or specializes `WorkTileInfo`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `WorkTileInfo`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 97-99

```cpp
  97 |     // Number of k tiles to compute for this unit of work. For stream-K, this
  98 |     // can indicate the number of K tiles across multiple output tiles.
  99 |     uint32_t k_tile_count = 0;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 101-102

```cpp
 101 |     // Number of k tiles remaining for the work unit as a whole
 102 |     uint32_t k_tile_remaining = 0;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 104-105

```cpp
 104 |     // Whether this unit of work is the final split for the given tile
 105 |     bool is_separate_reduction = false;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 107-113

```cpp
 107 |     CUTLASS_HOST_DEVICE
 108 |     bool
 109 |     is_valid() const {
 110 |       // A work tile that computes no K tiles is invalid unless it is a separate-reduction work tile
 111 |       // (which only performs reduction and epilogue)
 112 |       return k_tile_count > 0 || is_separate_reduction;
 113 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 115-119

```cpp
 115 |     CUTLASS_HOST_DEVICE
 116 |     bool
 117 |     is_reduction_unit() const {
 118 |       return is_separate_reduction;
 119 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 121-128

```cpp
 121 |     CUTLASS_HOST_DEVICE
 122 |     int32_t
 123 |     reduction_subtile_idx() const {
 124 |       // For separate reduction units, the K_idx of the work tile is unused.
 125 |       // Therefore, we override it to contain the subtile of that the reduction
 126 |       // unit operates on.
 127 |       return is_reduction_unit() ? K_idx : -1;
 128 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 130-135

```cpp
 130 |     CUTLASS_HOST_DEVICE
 131 |     void
 132 |     setup_separate_reduction(int32_t epilogue_subtile_idx) {
 133 |       // Set the epilogue subtile in the K_idx, since this is otherwise unused
 134 |       // by separate reduction units.
 135 |       K_idx = epilogue_subtile_idx;
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 137-141

```cpp
 137 |       is_separate_reduction = true;
 138 |       k_tile_count = 0;
 139 |       // Clean up remaining k tiles
 140 |       k_tile_remaining = 0;
 141 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 143-147

```cpp
 143 |     CUTLASS_HOST_DEVICE
 144 |     static WorkTileInfo
 145 |     invalid_work_tile() {
 146 |       return {-1, -1, -1, -1, 0};
 147 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 149-154

```cpp
 149 |     CUTLASS_HOST_DEVICE
 150 |     bool
 151 |     is_final_split(uint32_t k_tiles_per_output_tile) const {
 152 |       return (K_idx + k_tile_count) == k_tiles_per_output_tile;
 153 |     }
 154 |   };
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 156-156

```cpp
 156 |   struct Arguments {
```
**EN:** This `Arguments` structure collects the host-facing runtime inputs: problem shape, pointers, strides, epilogue settings, hardware info, and scheduler data.
**CN:** 这个 `Arguments` 结构体收集面向主机侧的运行时输入，包括问题规模、数据指针、步长、epilogue 配置、硬件信息以及调度器数据。

### Lines 158-160

```cpp
 158 |     Arguments() = default;
 159 |     Arguments(Arguments const&) = default;
 160 |     Arguments(Arguments&&) = default;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 162-171

```cpp
 162 |     CUTLASS_HOST_DEVICE
 163 |     Arguments&
 164 |     operator=(Arguments const& args) {
 165 |       splits = args.splits;
 166 |       max_swizzle_size = args.max_swizzle_size;
 167 |       raster_order = args.raster_order;
 168 |       reduction_mode = args.reduction_mode;
 169 |       decomposition_mode = args.decomposition_mode;
 170 |       return *this;
 171 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 173-182

```cpp
 173 |     CUTLASS_HOST_DEVICE
 174 |     Arguments&
 175 |     operator=(Arguments&& args) noexcept {
 176 |       splits = args.splits;
 177 |       max_swizzle_size = args.max_swizzle_size;
 178 |       raster_order = args.raster_order;
 179 |       reduction_mode = args.reduction_mode;
 180 |       decomposition_mode = args.decomposition_mode;
 181 |       return *this;
 182 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 184-185

```cpp
 184 |     CUTLASS_HOST_DEVICE
 185 |     Arguments(int splits_) : splits(splits_) {}
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 187-192

```cpp
 187 |     CUTLASS_HOST_DEVICE
 188 |     Arguments(int splits_, int max_swizzle_size_, RasterOrderOptions raster_order_, DecompositionMode decomposition_mode_) :
 189 |       splits(splits_),
 190 |       max_swizzle_size(max_swizzle_size_),
 191 |       raster_order(raster_order_),
 192 |       decomposition_mode(decomposition_mode_) {}
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 194-202

```cpp
 194 |     // The splitting factor to be used in a split-K decomposition of the problem.
 195 |     // If this is set to a value greater than 1, stream-K decomposition logic
 196 |     // is bypassed in favor of a split-K decomposition.
 197 |     int splits = 1;
 198 |     int max_swizzle_size = 1;
 199 |     RasterOrderOptions raster_order = RasterOrderOptions::Heuristic;
 200 |     ReductionMode reduction_mode = ReductionMode::Deterministic;
 201 |     DecompositionMode decomposition_mode = DecompositionMode::Heuristic;
 202 |   };
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 204-209

```cpp
 204 |   // Sink scheduler params as a member
 205 |   Params scheduler_params;
 206 | 
 207 |   //
 208 |   // Methods
 209 |   //
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 211-221

```cpp
 211 |   template <class ProblemShape>
 212 |   static Params
 213 |   to_underlying_arguments(
 214 |       ProblemShape problem_shape,
 215 |       TileShape tile_shape,
 216 |       ClusterShape cluster_shape,
 217 |       KernelHardwareInfo const& hw_info,
 218 |       Arguments const& args,
 219 |       void* workspace,
 220 |       const uint32_t epilogue_subtile = 1,
 221 |       [[maybe_unused]] uint32_t ktile_start_alignment_count = 1u) {
```
**EN:** This helper converts the public argument bundle into lower-level mainloop, epilogue, and scheduler parameters expected by the actual kernel implementation.
**CN:** 这个辅助函数把对外的参数包转换成底层内核实现真正需要的 mainloop、epilogue 和调度器参数。

### Lines 223-224

```cpp
 223 |     static_assert(cute::is_static<TileShape>::value);
 224 |     static_assert(cute::is_static<ClusterShape>::value);
```
**EN:** These compile-time checks enforce invariants such as problem-rank assumptions, supported scheduler tags, or required tensor-stride shapes.
**CN:** 这些编译期检查用于保证问题维度假设、可支持的调度器标签以及张量步长形状等不变式成立。

### Lines 226-228

```cpp
 226 |     auto problem_shape_mnkl = cute::append<4>(problem_shape, cute::Int<1>{});
 227 |     dim3 problem_blocks = get_tiled_cta_shape_mnl(problem_shape_mnkl, tile_shape, cluster_shape);
 228 |     uint32_t k_tile_per_output_tile = cute::size(cute::ceil_div(cute::shape<2>(problem_shape_mnkl), cute::shape<2>(TileShape{})));
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 230-245

```cpp
 230 |     Params params;
 231 |     params.initialize(
 232 |       problem_blocks,
 233 |       k_tile_per_output_tile,
 234 |       to_gemm_coord(cluster_shape),
 235 |       hw_info,
 236 |       args.splits,
 237 |       args.max_swizzle_size,
 238 |       args.raster_order,
 239 |       args.reduction_mode,
 240 |       args.decomposition_mode,
 241 |       workspace,
 242 |       epilogue_subtile
 243 |     );
 244 |     return params;
 245 |   }
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 247-253

```cpp
 247 |   static bool
 248 |   can_implement(Arguments const& args) {
 249 |     // Split count > 1 is only valid for heuristic and split-K decomposition modes
 250 |     return (args.splits == 1 ||
 251 |             args.decomposition_mode == DecompositionMode::Heuristic ||
 252 |             args.decomposition_mode == DecompositionMode::SplitK);
 253 |   }
```
**EN:** This function checks whether the requested problem, layouts, and runtime options satisfy the constraints of the specialized kernel.
**CN:** 该函数检查请求的问题规模、布局和运行时选项是否满足当前特化内核的约束条件。

### Lines 255-256

```cpp
 255 |   CUTLASS_HOST_DEVICE
 256 |   PersistentTileSchedulerSm90StreamK() { };
```
**EN:** This block continues the scheduler logic, Stream-K flow defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑、Stream-K 流程相关逻辑。

### Lines 258-267

```cpp
 258 |   CUTLASS_DEVICE
 259 |   PersistentTileSchedulerSm90StreamK(Params const& params_) : scheduler_params(params_), block_id_in_cluster_(cute::block_id_in_cluster()) {
 260 |     if (params_.raster_order_ == RasterOrder::AlongN) {
 261 |       current_work_linear_idx_ = uint64_t(BlockIdxX()) + uint64_t(BlockIdxY()) * uint64_t(GridDimX());
 262 |     }
 263 |     else {
 264 |       current_work_linear_idx_ = uint64_t(BlockIdxX()) * uint64_t(GridDimY()) + uint64_t(BlockIdxY());
 265 |     }
 266 | 
 267 |   }
```
**EN:** This block continues the scheduler logic, Stream-K flow defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑、Stream-K 流程相关逻辑。

### Lines 269-273

```cpp
 269 |   CUTLASS_DEVICE
 270 |   WorkTileInfo
 271 |   get_current_work() {
 272 |     return get_current_work_for_linear_idx(unit_iter_start_, current_work_linear_idx_, block_id_in_cluster_, scheduler_params);
 273 |   }
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 275-287

```cpp
 275 |   CUTLASS_DEVICE
 276 |   static WorkTileInfo
 277 |   get_current_work_for_linear_idx(uint32_t &unit_iter_start, uint64_t linear_idx, dim3 block_id_in_cluster, Params const& params) {
 278 |     // The maximum number of work units is units_per_problem_ * splits_.
 279 |     // The multiplication by splits_ is used for handling split-K, in which
 280 |     // units_per_problem_ is equal to the total number of output tiles. To account
 281 |     // for the fact that we have splits_ peers per output tile, we multiply this
 282 |     // value by splits_. For stream-K, this multiplication ends up being a no-op
 283 |     // because splits_ is set to 1 for stream-K.
 284 |     if(linear_idx >= (params.units_per_problem_ * params.divmod_splits_.divisor + params.separate_reduction_units_)) {
 285 |       // Invalid work. Return an empty result.
 286 |       return WorkTileInfo::invalid_work_tile();
 287 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 289-292

```cpp
 289 |     WorkTileInfo work_tile_info;
 290 |     assign_work(params, linear_idx, block_id_in_cluster, work_tile_info, unit_iter_start);
 291 |     return work_tile_info;
 292 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 294-303

```cpp
 294 |   // Returns whether the current work_tile_info passed in should continue to be used. This
 295 |   // occurs only in the stream-K decomposition with stream-K work units, which encompass
 296 |   // work over multiple output tiles. If the current work_tile_info should continue to be
 297 |   // used, it is updated to advance to the next output tile it should cover.
 298 |   CUTLASS_DEVICE
 299 |   bool
 300 |   continue_current_work(WorkTileInfo& work_tile_info) const {
 301 |     return continue_current_work_for_linear_idx(
 302 |       current_work_linear_idx_, unit_iter_start_, block_id_in_cluster_, work_tile_info, scheduler_params);
 303 |   }
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 305-312

```cpp
 305 |   CUTLASS_DEVICE
 306 |   static bool
 307 |   continue_current_work_for_linear_idx(
 308 |     uint64_t linear_idx,
 309 |     uint32_t unit_iter_start,
 310 |     dim3 block_id_in_cluster,
 311 |     WorkTileInfo& work_tile_info,
 312 |     Params const& params) {
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 314-314

```cpp
 314 |     work_tile_info.k_tile_remaining -= work_tile_info.k_tile_count;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 316-321

```cpp
 316 |     if (work_tile_info.k_tile_remaining == 0) {
 317 |       return false;
 318 |     }
 319 |     fast_assign_work(unit_iter_start, params, linear_idx, block_id_in_cluster, work_tile_info);
 320 |     return work_tile_info.is_valid();
 321 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 323-327

```cpp
 323 |   CUTLASS_DEVICE
 324 |   void
 325 |   advance_to_next_work(uint32_t advance_count = 1) {
 326 |     current_work_linear_idx_ += uint64_t(GridDimX()) * uint64_t(GridDimY()) * uint64_t(GridDimZ()) * uint64_t(advance_count);
 327 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 329-344

```cpp
 329 |   CUTLASS_DEVICE
 330 |   bool is_last_tile(WorkTileInfo work_tile_info, uint32_t advance_count = 1) const {
 331 |      // Never pass this by reference; it needs a copy,
 332 |     // because continue_current_work will modify it.
 333 |     if (continue_current_work(work_tile_info)) {
 334 |       return false;
 335 |     }
 336 |     return not get_current_work_for_linear_idx(
 337 |         unit_iter_start_,
 338 |         current_work_linear_idx_ + (
 339 |           uint64_t(GridDimX()) * uint64_t(GridDimY()) * uint64_t(GridDimZ()) * uint64_t(advance_count)
 340 |           ),
 341 |         block_id_in_cluster_,
 342 |         scheduler_params
 343 |     ).is_valid();
 344 |   }
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 346-353

```cpp
 346 |   // Given the inputs, computes the total number of output blocks this problem will compute over
 347 |   // Note that this is only the logical size of our grid, not the physical grid we will actually launch.
 348 |   template <class ProblemShape>
 349 |   CUTLASS_HOST_DEVICE static
 350 |   dim3
 351 |   get_tiled_cta_shape_mnl(ProblemShape problem_shape_mnkl, TileShape cta_shape, ClusterShape cluster_shape) {
 352 |     return UnderlyingScheduler::get_tiled_cta_shape_mnl(problem_shape_mnkl, cta_shape, cluster_shape);
 353 |   }
```
**EN:** This block declares or specializes `ProblemShape`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `ProblemShape`，它是该头文件中承载某一层内核策略的核心类。

### Lines 355-365

```cpp
 355 |   // Given the cluster shape, computes the physical grid we should launch.
 356 |   template <class ProblemShape>
 357 |   CUTLASS_HOST_DEVICE static
 358 |   dim3
 359 |   get_grid_shape(
 360 |     [[maybe_unused]] Params const& params,
 361 |     ProblemShape problem_shape,
 362 |     TileShape tile_shape,
 363 |     ClusterShape cluster_shape,
 364 |     KernelHardwareInfo hw_info,
 365 |     Arguments arguments) {
```
**EN:** These launch helpers compute the CUDA/SYCL execution geometry from the problem shape, tile shape, and scheduling policy.
**CN:** 这些启动辅助函数根据问题规模、Tile 形状和调度策略计算 CUDA/SYCL 的执行网格。

### Lines 367-368

```cpp
 367 |     auto problem_shape_mnkl = cute::append<4>(problem_shape, cute::Int<1>{});
 368 |     dim3 problem_blocks = get_tiled_cta_shape_mnl(problem_shape_mnkl, tile_shape, cluster_shape);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 370-377

```cpp
 370 |     return Params::get_grid_shape(
 371 |       problem_blocks,
 372 |       to_gemm_coord(cluster_shape),
 373 |       hw_info,
 374 |       arguments.max_swizzle_size,
 375 |       arguments.raster_order
 376 |     );
 377 |   }
```
**EN:** These launch helpers compute the CUDA/SYCL execution geometry from the problem shape, tile shape, and scheduling policy.
**CN:** 这些启动辅助函数根据问题规模、Tile 形状和调度策略计算 CUDA/SYCL 的执行网格。

### Lines 379-385

```cpp
 379 |   // Returns whether fixup is needed for `work_tile_info`.
 380 |   CUTLASS_HOST_DEVICE
 381 |   static bool
 382 |   requires_fixup(Params const& params, WorkTileInfo const& work_tile_info) {
 383 |     // Fixup is not needed for invalid or data-parallel tiles
 384 |     return work_tile_info.is_valid() && work_tile_info.k_tile_count != params.divmod_tiles_per_output_tile_.divisor;
 385 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 387-391

```cpp
 387 |   CUTLASS_HOST_DEVICE
 388 |   static bool
 389 |   requires_separate_reduction(Params const& params) {
 390 |     return params.requires_separate_reduction();
 391 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 393-399

```cpp
 393 |   // When the work tile is not special for reduction, it's valid. Otherwise need to skip
 394 |   // global loading that producer warpgroup do, also math computation that consumer warpgroup do.
 395 |   CUTLASS_DEVICE
 396 |   static bool
 397 |   valid_warpgroup_in_work_tile(WorkTileInfo const& work_tile_info) {
 398 |     return !work_tile_info.is_reduction_unit();
 399 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 401-416

```cpp
 401 |   // Performs the reduction across splits for a given output tile.
 402 |   template <class FrgTensorC>
 403 |   CUTLASS_DEVICE
 404 |   static void
 405 |   fixup(
 406 |     Params const& params,
 407 |     WorkTileInfo const& work_tile_info,
 408 |     FrgTensorC& accumulators,
 409 |     uint32_t num_barriers,
 410 |     uint32_t barrier_idx) {
 411 |     static constexpr uint32_t Offset = static_cast<int>(cutlass::arch::ReservedNamedBarriers::StreamkBarrier0);
 412 |     static constexpr uint32_t MaxNumNamedBarriers = 2;
 413 |     using BarrierManager = NamedBarrierManager<NumThreadsPerWarpGroup, Offset, MaxNumNamedBarriers>;
 414 |     return fixup_helper<FrgTensorC, BarrierManager>(
 415 |       params, work_tile_info, accumulators, num_barriers, barrier_idx);
 416 |   }
```
**EN:** This alias block derives concise type names `BarrierManager` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `BarrierManager` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 418-429

```cpp
 418 |   // Helper for performing the reduction across splits for a given output tile.
 419 |   template <class FrgTensorC, class BarrierManager>
 420 |   CUTLASS_DEVICE
 421 |   static void
 422 |   fixup_helper(
 423 |     Params const& params,
 424 |     WorkTileInfo const& work_tile_info,
 425 |     FrgTensorC& accumulators,
 426 |     uint32_t num_barriers,
 427 |     uint32_t barrier_idx,
 428 |     uint32_t num_accumulator_mtxs = 1,
 429 |     uint32_t idx_accumulator_mtxs = 0) {
```
**EN:** This block declares or specializes `FrgTensorC`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `FrgTensorC`，它是该头文件中承载某一层内核策略的核心类。

### Lines 431-431

```cpp
 431 |     using ElementAccumulator = typename FrgTensorC::value_type;
```
**EN:** This alias block derives concise type names `ElementAccumulator` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `ElementAccumulator` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 433-436

```cpp
 433 |     if (!requires_fixup(params, work_tile_info)) {
 434 |       return;
 435 |     }
 436 |     uint64_t tile_idx = output_tile_index(params, work_tile_info);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 438-439

```cpp
 438 |     // Index of the lock on which to wait
 439 |     uint64_t lock_idx = (tile_idx * num_barriers) + barrier_idx;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 441-455

```cpp
 441 |     uint64_t reduction_tile_idx = tile_idx;
 442 |     uint64_t num_peers = 0;
 443 |     uint64_t reduction_peer_offset = 0;
 444 |     if (
 445 |       params.requires_separate_reduction()
 446 |       ) {
 447 |       // If separate reduction is to be performed, each stream-K unit writes its partials
 448 |       // to a separate portion of the workspace. There are as many of these portions as there
 449 |       // are peers for a given output tile, so we multiply the tile index by the maximum peer count.
 450 |       auto [first_peer_id, my_peer_id, last_peer_id] = tile_peer_range(params, tile_idx, work_tile_info);
 451 |       auto peer_id_in_output_tile = my_peer_id - first_peer_id;
 452 |       num_peers = last_peer_id - first_peer_id + 1;
 453 |       reduction_tile_idx = tile_idx * Params::max_peers_per_tile(params.sk_units_, params.sk_tiles_);
 454 |       reduction_peer_offset = peer_id_in_output_tile * cute::size<0>(TileShape{}) * cute::size<1>(TileShape{}) * num_accumulator_mtxs;
 455 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 457-462

```cpp
 457 |     // Reductions use BlockStripedReduce with a width of BarrierManager::ThreadCount under the hood.
 458 |     // Thus, the start of the reduction space is the same across all threads in a warp group.
 459 |     uint64_t reduction_offset_base = (static_cast<uint64_t>(cute::size<0>(TileShape{})) * static_cast<uint64_t>(cute::size<1>(TileShape{})) * reduction_tile_idx * num_accumulator_mtxs) +
 460 |       (static_cast<uint64_t>(size(accumulators)) * barrier_idx * BarrierManager::ThreadCount * num_accumulator_mtxs)
 461 |       + static_cast<uint64_t>(size(accumulators)) * BarrierManager::ThreadCount * idx_accumulator_mtxs;
 462 |     uint64_t reduction_offset = reduction_offset_base + reduction_peer_offset;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 464-464

```cpp
 464 |     ElementAccumulator* group_reduction_workspace = reinterpret_cast<ElementAccumulator*>(params.reduction_workspace_) + reduction_offset;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 466-467

```cpp
 466 |     using AccumulatorArrayT = Array<typename FrgTensorC::value_type, size(FrgTensorC{})>;
 467 |     using BlockStripedReduceT = BlockStripedReduce<BarrierManager::ThreadCount, AccumulatorArrayT>;
```
**EN:** This alias block derives concise type names `AccumulatorArrayT`, `BlockStripedReduceT` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `AccumulatorArrayT`, `BlockStripedReduceT` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 469-470

```cpp
 469 |     AccumulatorArrayT* reduction_workspace_array = reinterpret_cast<AccumulatorArrayT*>(group_reduction_workspace);
 470 |     AccumulatorArrayT* accumulator_array = reinterpret_cast<AccumulatorArrayT*>(accumulators.data());
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 472-472

```cpp
 472 |     uint32_t barrier_group_thread_idx = ThreadIdxX() % BarrierManager::ThreadCount;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 474-491

```cpp
 474 |     // The number of tiles for which reduction is required is either:
 475 |     //   (a) the total number of output tiles (in the case of split-K)
 476 |     //   (b) the number of stream-K tiles (potentially multiplied by peer count if using separate reduction)
 477 |     // To calculate the total number of output tiles in the split-K case, we
 478 |     // note that, in the split-K case, the units_per_problem_ member of Params will be
 479 |     // the total number of output tiles.
 480 |     uint32_t reduction_tiles = 0;
 481 |     if (params.divmod_splits_.divisor > 1) {
 482 |       reduction_tiles = params.units_per_problem_;
 483 |     }
 484 |     else if (
 485 |       params.requires_separate_reduction()
 486 |       ) {
 487 |       reduction_tiles = params.sk_tiles_ * Params::max_peers_per_tile(params.sk_units_, params.sk_tiles_);
 488 |     }
 489 |     else {
 490 |       reduction_tiles = params.sk_tiles_;
 491 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 493-496

```cpp
 493 |     uint64_t reduction_workspace_size = Params::get_reduction_workspace_size(
 494 |       reduction_tiles, to_gemm_coord(TileShape{}), sizeof_bits<ElementAccumulator>::value, num_accumulator_mtxs);
 495 |     BarrierType* lock_workspace = reinterpret_cast<BarrierType*>(
 496 |       reinterpret_cast<uint8_t*>(params.reduction_workspace_) + reduction_workspace_size);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 498-501

```cpp
 498 |     if (work_tile_info.is_reduction_unit()) {
 499 |       // Wait until the peers collaborating on this output tile have all written
 500 |       // their accumulators to workspace.
 501 |       BarrierManager::wait_eq(barrier_idx, lock_workspace, barrier_group_thread_idx, lock_idx, num_peers);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 503-526

```cpp
 503 |       separate_reduction<FrgTensorC, BarrierManager>(accumulators, num_barriers, group_reduction_workspace, barrier_group_thread_idx, num_peers, num_accumulator_mtxs);
 504 |     }
 505 |     else if (!compute_epilogue(work_tile_info, params)) {
 506 |       if (
 507 |         params.requires_separate_reduction()
 508 |         || work_tile_info.K_idx == 0
 509 |         ) {
 510 |         // The first peer initializes the workspace partials in the non-separate-reduction case,
 511 |         // and all peers write to their own location in workspace when using separate reduction
 512 |         BlockStripedReduceT::store(reduction_workspace_array, *accumulator_array, barrier_group_thread_idx);
 513 |       }
 514 |       else {
 515 |         if (params.reduction_mode_ == ReductionMode::Deterministic) {
 516 |           // Wait until the preceding split added its accumulators
 517 |           BarrierManager::wait_eq(barrier_idx, lock_workspace, barrier_group_thread_idx, lock_idx, work_tile_info.K_idx);
 518 |         }
 519 |         else {
 520 |           // Wait until the first split has stored its accumulators. Note that the first split will have
 521 |           // accumulated a value into the lock potentially greater than one (since the locked value is
 522 |           // incremented by work_tile_info.k_tile_count below for both the deterministic and non-deterministic)
 523 |           // cases. For non-deterministic reductions, all that non-first or last splits care about is whether
 524 |           // the first split has been written, so we only wait while the locked value is less than 1.
 525 |           BarrierManager::wait_lt(barrier_idx, lock_workspace, barrier_group_thread_idx, lock_idx, 1);
 526 |         }
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 528-530

```cpp
 528 |         // Perform reduction in workspace
 529 |         BlockStripedReduceT::reduce(reduction_workspace_array, *accumulator_array, barrier_group_thread_idx);
 530 |       }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 532-534

```cpp
 532 |       // If separate reduction is being performed, each participating stream-K unit increments the barrier
 533 |       // by only 1. Otherwise, increment by the K tile count that this unit has processed.
 534 |       uint32_t increment = params.requires_separate_reduction() ? 1 : work_tile_info.k_tile_count;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 536-543

```cpp
 536 |       // Signal our arrival
 537 |       if (idx_accumulator_mtxs == (num_accumulator_mtxs - 1)) {
 538 |         BarrierManager::arrive_inc(barrier_idx, lock_workspace, barrier_group_thread_idx, lock_idx, increment);
 539 |       }
 540 |     }
 541 |     else {
 542 |       // Wait until the preceding split added its accumulators
 543 |       BarrierManager::wait_eq(barrier_idx, lock_workspace, barrier_group_thread_idx, lock_idx, work_tile_info.K_idx);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 545-549

```cpp
 545 |       // The block computing the final split for the tile adds previously-reduced partials
 546 |       // to its accumulators and computes the epilogue.
 547 |       BlockStripedReduceT::load_add(*accumulator_array, reduction_workspace_array, barrier_group_thread_idx);
 548 |     }
 549 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 551-562

```cpp
 551 |   template <class FrgTensorC, class BarrierManager>
 552 |   CUTLASS_DEVICE
 553 |   static void
 554 |   separate_reduction(
 555 |       FrgTensorC& accumulators,
 556 |       uint32_t num_barriers,
 557 |       typename FrgTensorC::value_type* reduction_workspace,
 558 |       uint32_t thread_idx,
 559 |       uint64_t num_peers,
 560 |       uint32_t num_accumulator_mtxs) {
 561 |     using AccumulatorArrayT = Array<typename FrgTensorC::value_type, size(FrgTensorC{})>;
 562 |     using BlockStripedReduceT = BlockStripedReduce<BarrierManager::ThreadCount, AccumulatorArrayT>;
```
**EN:** This alias block derives concise type names `AccumulatorArrayT`, `BlockStripedReduceT` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `AccumulatorArrayT`, `BlockStripedReduceT` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 564-564

```cpp
 564 |     AccumulatorArrayT* accumulator_array = reinterpret_cast<AccumulatorArrayT*>(accumulators.data());
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 566-567

```cpp
 566 |     plus<AccumulatorArrayT> add_fragments;
 567 |     uint64_t peer_offset = cute::size<0>(TileShape{}) * cute::size<1>(TileShape{}) * num_accumulator_mtxs;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 569-572

```cpp
 569 |     for (uint64_t i = 0; i < num_peers; ++i) {
 570 |       // Load peer fragment
 571 |       AccumulatorArrayT addend_fragment;
 572 |       auto peer_reduction_workspace = reinterpret_cast<AccumulatorArrayT*>(reduction_workspace + (i * peer_offset));
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 574-576

```cpp
 574 |       BlockStripedReduceT::load_add(*accumulator_array, peer_reduction_workspace, thread_idx);
 575 |     }
 576 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 578-590

```cpp
 578 |   // Returns whether the block assigned this work should compute the epilogue for the corresponding
 579 |   // output tile. For the case of stream-K, this should only occur if the work is marked as the final split.
 580 |   CUTLASS_HOST_DEVICE
 581 |   static bool
 582 |   compute_epilogue(WorkTileInfo const& work_tile_info, Params const& params) {
 583 |     // `is_final_split` will be set to `true` for the following scenarios, all of which must compute the epilogue:
 584 |     //  1. The tile is computed in data-parallel mode
 585 |     //  2. The tile is computed in split-/stream-K mode and this work unit represents the final split of the tile
 586 |     //  3. The tile is computed in split-/stream-K mode and separate reduction is used, and this is a separate reduction unit
 587 |     return work_tile_info.is_valid() &&
 588 |             (work_tile_info.is_final_split(params.divmod_tiles_per_output_tile_.divisor) &&
 589 |              !params.requires_separate_reduction()) || work_tile_info.is_separate_reduction;
 590 |   }
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 592-603

```cpp
 592 |   // Returns the linearized index of the output tile corresponding to the tile with offset [L, M, K]
 593 |   CUTLASS_DEVICE
 594 |   static uint64_t
 595 |   output_tile_index(Params const& params, WorkTileInfo const& work_tile_info) {
 596 |     uint64_t linear_idx_in_batch = UnderlyingScheduler::get_linear_idx_from_m_and_n(
 597 |       work_tile_info.M_idx, work_tile_info.N_idx,
 598 |       params.divmod_cluster_shape_major_,
 599 |       params.divmod_cluster_shape_minor_,
 600 |       params.divmod_cluster_blk_major_,
 601 |       params.log_swizzle_size_,
 602 |       params.raster_order_
 603 |     );
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 605-607

```cpp
 605 |     uint64_t tiles_mn = params.divmod_batch_.divisor;
 606 |     return tiles_mn * work_tile_info.L_idx + linear_idx_in_batch;
 607 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 609-617

```cpp
 609 |   template <class ProblemShape, class ElementAccumulator>
 610 |   static size_t
 611 |   get_workspace_size(
 612 |     Arguments const& args,
 613 |     ProblemShape problem_shape,
 614 |     KernelHardwareInfo const& hw_info,
 615 |     uint32_t mma_warp_groups,
 616 |     const uint32_t epilogue_subtile = 1,
 617 |     [[maybe_unused]] uint32_t num_accumulator_mtxs = 1) {
```
**EN:** This block reports or initializes any auxiliary workspace needed by the kernel before launch.
**CN:** 这一段用于报告或初始化内核启动前所需的辅助工作区。

### Lines 619-619

```cpp
 619 |     auto problem_shape_mnkl = cute::append<4>(problem_shape, 1);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 621-622

```cpp
 621 |     ClusterShape cluster_shape;
 622 |     TileShape tile_shape;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 624-625

```cpp
 624 |     dim3 problem_blocks = get_tiled_cta_shape_mnl(problem_shape_mnkl, tile_shape, cluster_shape);
 625 |     uint32_t k_tile_per_output_tile = cute::size(cute::ceil_div(cute::shape<2>(problem_shape_mnkl), cute::shape<2>(TileShape{})));
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 627-643

```cpp
 627 |     return Params::get_workspace_size(
 628 |       problem_blocks,
 629 |       k_tile_per_output_tile,
 630 |       to_gemm_coord(tile_shape),
 631 |       to_gemm_coord(cluster_shape),
 632 |       hw_info,
 633 |       args.splits,
 634 |       args.max_swizzle_size,
 635 |       args.raster_order,
 636 |       args.decomposition_mode,
 637 |       args.reduction_mode,
 638 |       mma_warp_groups,
 639 |       sizeof_bits<BarrierType>::value,
 640 |       sizeof_bits<ElementAccumulator>::value,
 641 |       epilogue_subtile
 642 |     );
 643 |   }
```
**EN:** This block reports or initializes any auxiliary workspace needed by the kernel before launch.
**CN:** 这一段用于报告或初始化内核启动前所需的辅助工作区。

### Lines 645-656

```cpp
 645 |   template <class ProblemShape, class ElementAccumulator>
 646 |   static cutlass::Status
 647 |   initialize_workspace(
 648 |     Arguments const& args,
 649 |     void* workspace,
 650 |     cudaStream_t stream,
 651 |     ProblemShape const& problem_shape,
 652 |     KernelHardwareInfo const& hw_info,
 653 |     uint32_t mma_warp_groups,
 654 |     const uint32_t epilogue_subtile = 1,
 655 |     [[maybe_unused]] uint32_t num_accumulator_mtxs = 1,
 656 |     CudaHostAdapter* cuda_adapter = nullptr) {
```
**EN:** This block reports or initializes any auxiliary workspace needed by the kernel before launch.
**CN:** 这一段用于报告或初始化内核启动前所需的辅助工作区。

### Lines 658-658

```cpp
 658 |     auto problem_shape_mnkl = cute::append<4>(problem_shape, 1);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 660-661

```cpp
 660 |     ClusterShape cluster_shape;
 661 |     TileShape tile_shape;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 663-664

```cpp
 663 |     dim3 problem_blocks = get_tiled_cta_shape_mnl(problem_shape_mnkl, tile_shape, cluster_shape);
 664 |     uint32_t k_tile_per_output_tile = cute::size(cute::ceil_div(cute::shape<2>(problem_shape_mnkl), cute::shape<2>(TileShape{})));
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 666-686

```cpp
 666 |     return Params::initialize_workspace(
 667 |       workspace,
 668 |       stream,
 669 |       problem_blocks,
 670 |       k_tile_per_output_tile,
 671 |       to_gemm_coord(tile_shape),
 672 |       to_gemm_coord(cluster_shape),
 673 |       hw_info,
 674 |       args.splits,
 675 |       args.max_swizzle_size,
 676 |       args.raster_order,
 677 |       args.decomposition_mode,
 678 |       args.reduction_mode,
 679 |       mma_warp_groups,
 680 |       sizeof_bits<BarrierType>::value,
 681 |       sizeof_bits<ElementAccumulator>::value,
 682 |       epilogue_subtile,
 683 |       1,
 684 |       cuda_adapter
 685 |     );
 686 |   }
```
**EN:** This block reports or initializes any auxiliary workspace needed by the kernel before launch.
**CN:** 这一段用于报告或初始化内核启动前所需的辅助工作区。

### Lines 688-693

```cpp
 688 |   template <class ProblemShape>
 689 |   CUTLASS_HOST_DEVICE
 690 |   static uint32_t
 691 |   get_work_k_tile_count(WorkTileInfo const& work_tile_info, ProblemShape, TileShape) {
 692 |     return work_tile_info.k_tile_count;
 693 |   }
```
**EN:** This block declares or specializes `ProblemShape`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `ProblemShape`，它是该头文件中承载某一层内核策略的核心类。

### Lines 695-699

```cpp
 695 |   CUTLASS_HOST_DEVICE
 696 |   static uint32_t
 697 |   get_work_k_tile_start(WorkTileInfo const& work_tile_info) {
 698 |     return work_tile_info.K_idx;
 699 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 701-707

```cpp
 701 |   // Kernel helper function to get next work tile
 702 |   CUTLASS_DEVICE
 703 |   auto
 704 |   fetch_next_work(WorkTileInfo work_tile_info) {
 705 |     if (continue_current_work(work_tile_info)) {
 706 |       return cute::make_tuple(work_tile_info, true);
 707 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 709-711

```cpp
 709 |     advance_to_next_work();
 710 |     return cute::make_tuple(get_current_work(), true);
 711 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 713-722

```cpp
 713 |   // Kernel helper function to get next work tile
 714 |   template <class TileSchedulerPipeline, class TileSchedulerPipelineState>
 715 |   CUTLASS_DEVICE
 716 |   auto
 717 |   fetch_next_work(
 718 |       WorkTileInfo work_tile_info,
 719 |       TileSchedulerPipeline& scheduler_pipeline,
 720 |       TileSchedulerPipelineState scheduler_pipe_consumer_state) {
 721 |     return fetch_next_work(work_tile_info);
 722 |   }
```
**EN:** This block declares or specializes `TileSchedulerPipeline`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `TileSchedulerPipeline`，它是该头文件中承载某一层内核策略的核心类。

### Lines 724-729

```cpp
 724 |   // Returns the initial work tile info that will be computed over
 725 |   CUTLASS_DEVICE
 726 |   WorkTileInfo
 727 |   initial_work_tile_info(ClusterShape) {
 728 |     return get_current_work();
 729 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 731-740

```cpp
 731 |   // Given raster order and current work tile linear index, reset cta m and n index in the cluster.
 732 |   CUTLASS_DEVICE
 733 |   static dim3
 734 |   get_current_work_cta_m_n_in_cluster(
 735 |     Params const& params,
 736 |     uint64_t linear_idx,
 737 |     dim3 block_id_in_cluster) {
 738 |     auto [cta_m_in_cluster_, cta_n_in_cluster_, _] = block_id_in_cluster;
 739 |     uint64_t cta_m_in_cluster = static_cast<uint64_t>(cta_m_in_cluster_);
 740 |     uint64_t cta_n_in_cluster = static_cast<uint64_t>(cta_n_in_cluster_);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 742-745

```cpp
 742 |     // Determine the CTA's M and N offsets within the preferred cluster
 743 |     // This simply finds the linear offset of the CTA within the cluster, and takes a divmod
 744 |     // on it depending on the rasterization order used by the scheduler.
 745 |     uint64_t cluster_linear_work_idx_tmp = params.div_cluster_size(linear_idx) * params.get_cluster_size();
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 747-752

```cpp
 747 |     if (params.raster_order_ == RasterOrder::AlongN) {
 748 |       params.divmod_cluster_shape_minor_(cta_n_in_cluster, cta_m_in_cluster, linear_idx - cluster_linear_work_idx_tmp);
 749 |     }
 750 |     else {
 751 |       params.divmod_cluster_shape_minor_(cta_m_in_cluster, cta_n_in_cluster, linear_idx - cluster_linear_work_idx_tmp);
 752 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 754-755

```cpp
 754 |     return {static_cast<uint32_t>(cta_m_in_cluster), static_cast<uint32_t>(cta_n_in_cluster), _};
 755 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 757-757

```cpp
 757 | private:
```
**EN:** This comment-only block adds explanatory context for the surrounding implementation, such as layout assumptions, scheduling notes, or design rationale.
**CN:** 这一段纯注释内容为周围实现补充了解释性背景，例如布局假设、调度说明或设计动机。

### Lines 759-780

```cpp
 759 |   CUTLASS_DEVICE
 760 |   static uint32_t
 761 |   get_current_work_iter_start_possible_update_work_tile_k_remaining(
 762 |     Params const& params,
 763 |     uint64_t linear_idx,
 764 |     WorkTileInfo& work_tile_info) {
 765 |     // In the CUTLASS 2.x implementation of stream K, stream-K work is assigned to each stream-K
 766 |     // threadblock individually. For the most part, the set of K iterations corresponding to stream-K
 767 |     // work was divided amongst stream-K threadblocks, and a threadblock determined which tile
 768 |     // it would compute a (potentially-partial) output tile for based on the space of k iterations
 769 |     // assigned to it. This often results in stream-K threadblocks processing tiles with different
 770 |     // offsets in the K dimension from one another. This can reduce locality, but is lmitied to the
 771 |     // (generally few) waves of threadblocks assigned to compute stream-K work.
 772 |     //
 773 |     // With the introduction of threadblock clusters, there is additional benefit to maintaining
 774 |     // locality in the K dimension: shared portions of operands can be multicasted to threadblocks
 775 |     // within a cluster. Thus, we would like to ensure that the assignment of stream-K work to
 776 |     // threadblocks respects the ability to perform multicasting.
 777 |     //
 778 |     // To do so, we divide up the linearized stream-K units into clusters and share the same K
 779 |     // offsets for work within clusters.
 780 |     uint64_t cluster_linear_work_idx = params.div_cluster_size(linear_idx);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 782-783

```cpp
 782 |     uint64_t group_idx;
 783 |     params.divmod_sk_groups_(cluster_linear_work_idx, group_idx, cluster_linear_work_idx);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 785-791

```cpp
 785 |     // Determine whether we are in a "big group" that will process an additional
 786 |     // stream-K cluster tile.
 787 |     uint64_t sk_cluster_tiles = params.div_cluster_size(params.sk_tiles_);
 788 |     uint64_t sk_cluster_tiles_in_group = params.divmod_sk_groups_.divide(sk_cluster_tiles);
 789 |     if (group_idx < params.big_groups_) {
 790 |       ++sk_cluster_tiles_in_group;
 791 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 793-799

```cpp
 793 |     // Determine whether we are in a "big unit" within the group, that will process
 794 |     // an additional K chunk in the group.
 795 |     uint64_t sk_tiles_in_group = sk_cluster_tiles_in_group * params.get_cluster_size();
 796 |     uint64_t k_tiles_in_group = sk_tiles_in_group * params.divmod_tiles_per_output_tile_.divisor;
 797 |     uint64_t k_tiles_per_unit_in_group = params.divmod_sk_units_per_group_.divide(k_tiles_in_group);
 798 |     uint64_t big_units_in_group = params.div_cluster_size(
 799 |       k_tiles_in_group - (k_tiles_per_unit_in_group * params.divmod_sk_units_per_group_.divisor));
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 801-802

```cpp
 801 |     uint64_t split;
 802 |     params.divmod_clusters_mnl_(split, cluster_linear_work_idx, cluster_linear_work_idx);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 804-808

```cpp
 804 |     bool is_split_k = params.divmod_splits_.divisor > 1;
 805 |     uint64_t big_unit_cmp_lhs = is_split_k ? split : cluster_linear_work_idx;
 806 |     uint64_t big_unit_cmp_rhs = is_split_k ? params.big_units_ : big_units_in_group;
 807 |     uint64_t linear_idx_mult = is_split_k ? params.divmod_tiles_per_output_tile_.divisor : k_tiles_per_unit_in_group;
 808 |     uint64_t k_tiles_per_split = is_split_k ? params.divmod_k_tiles_per_sk_unit_.divisor : k_tiles_per_unit_in_group;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 810-812

```cpp
 810 |     // Determine the starting k iteration computed by this stream-K work unit
 811 |     uint32_t unit_iter_start = (linear_idx_mult * cluster_linear_work_idx) +
 812 |                                (k_tiles_per_split * split);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 814-833

```cpp
 814 |     // Adjust the starting position and number of k iterations for "big units," which
 815 |     // compute one extra iteration. If there are any big units, they will be the first
 816 |     // in the linearized ID space.
 817 |     auto k_tiles_in_my_split = k_tiles_per_split;
 818 |     if (big_unit_cmp_lhs < big_unit_cmp_rhs) {
 819 |       // Since the "big units" are the first units in the linearized ID space, each
 820 |       // of the units preceding this big unit computed one extra iteration. Thus,
 821 |       // we must offset our start iteration by the number of units that precede
 822 |       // the current unit in the linearized ID space.
 823 |       unit_iter_start += big_unit_cmp_lhs;
 824 |       ++k_tiles_in_my_split;
 825 |     }
 826 |     else {
 827 |       // Increment by one for each of the big clusters (since all big units precede this unit)
 828 |       unit_iter_start += big_unit_cmp_rhs;
 829 |     }
 830 |     if (!is_split_k) {
 831 |       // Adjust the unit starting position and number of tiles to avoid
 832 |       // computing splits of size less than min_iters_per_sk_unit_
 833 |       int unused, start_tile_k_tile;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 834-853

```cpp
 834 |       params.divmod_tiles_per_output_tile_(unused, start_tile_k_tile, unit_iter_start);
 835 |       if (start_tile_k_tile < Params::min_iters_per_sk_unit_) {
 836 |         // Starting K tile is in range [0, Params::min_iters_per_sk_unit_), which means that another
 837 |         // stream-K unit will be computing a split with fewer than Params::min_iters_per_sk_unit_ K tiles.
 838 |         // Adjust our work to take over these K tiles.
 839 |         unit_iter_start -= start_tile_k_tile;
 840 |         k_tiles_in_my_split += start_tile_k_tile;
 841 |       }
 842 |       else if (start_tile_k_tile > (params.divmod_tiles_per_output_tile_.divisor - Params::min_iters_per_sk_unit_)) {
 843 |         // Starting K tile is within the final Params::min_iters_per_sk_unit_ K tiles of some output tile,
 844 |         // which means that this unit will compute a split with fewer than Params::min_iters_per_sk_unit_ K tiles.
 845 |         // Adjust our work to shed these K tiles to a neighboring stream-K unit that will compute more consecutive K tiles.
 846 |         auto adjustment_tiles = (params.divmod_tiles_per_output_tile_.divisor - start_tile_k_tile);
 847 |         unit_iter_start += adjustment_tiles;
 848 |         k_tiles_in_my_split -= adjustment_tiles;
 849 |       }
 850 |       else if (params.ktile_start_alignment_count_ == 2 && start_tile_k_tile % 2 != 0) {
 851 |         // ktile for each SM start from even number
 852 |         // If start from odd number ktile within the output tile
 853 |         //    now start at the ktile one before my initial ktile start (take one ktile from prev sm)
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 854-861

```cpp
 854 |         // if end on odd number ktile within the output tile
 855 |         //    now end at ktile that one before my ktile end (give one ktile to next sm)
 856 |         unit_iter_start -= 1;
 857 |         k_tiles_in_my_split += 1;
 858 |       }
 859 |     }
 860 |     if (work_tile_info.k_tile_count == 0) {
 861 |       // This is a new unit
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 863-867

```cpp
 863 |       if (!is_split_k) {
 864 |         //
 865 |         // Adjust the unit ending position and number of tiles to avoid
 866 |         // computing splits of size less than min_iters_per_sk_unit_
 867 |         //
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 869-870

```cpp
 869 |         // Begin by assuming that no adjustment is needed
 870 |         auto initial_unit_iter_end = unit_iter_start + k_tiles_in_my_split;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 872-873

```cpp
 872 |         int unused, end_tile_k_tile;
 873 |         params.divmod_tiles_per_output_tile_(unused, end_tile_k_tile, initial_unit_iter_end);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 875-895

```cpp
 875 |         if (end_tile_k_tile < Params::min_iters_per_sk_unit_) {
 876 |           // Ending K tile is within the first Params::min_iters_per_sk_unit_ K tiles of some output tile,
 877 |           // which means that this unit will compute a split with fewer than Params::min_iters_per_sk_unit_ K tiles.
 878 |           // Adjust our work to shed these K tiles to a neighboring stream-K unit that will compute more consecutive K tiles.
 879 |           k_tiles_in_my_split -= end_tile_k_tile;
 880 |         }
 881 |         else if (end_tile_k_tile > (params.divmod_tiles_per_output_tile_.divisor - Params::min_iters_per_sk_unit_)) {
 882 |           // Ending K tile is within the final Params::min_iters_per_sk_unit_ K tiles of some output tile,
 883 |           // which means that some other unit will compute a split with fewer than Params::min_iters_per_sk_unit_ K tiles.
 884 |           // Adjust our work to take on these K tiles.
 885 |           k_tiles_in_my_split += (params.divmod_tiles_per_output_tile_.divisor - end_tile_k_tile);
 886 |         }
 887 |         else if (params.ktile_start_alignment_count_ == 2 && end_tile_k_tile % 2 != 0) {
 888 |           // ktile for each SM start from even number
 889 |           // If start from odd number ktile within the output tile
 890 |           //    now start at the ktile one before my initial ktile start (take one ktile from prev sm)
 891 |           // If end on odd number ktile within the output tile,
 892 |           //    now end at ktile that one before my ktile end (give one ktile to next sm)
 893 |           k_tiles_in_my_split -= 1;
 894 |         }
 895 |       }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 897-900

```cpp
 897 |       work_tile_info.k_tile_remaining = k_tiles_in_my_split;
 898 |     }
 899 |     return unit_iter_start;
 900 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 902-913

```cpp
 902 |   // Update output tile index given existing remaining k tiles of current work tile.
 903 |   CUTLASS_DEVICE
 904 |   static uint64_t update_output_tile_id_and_work_tile_k(
 905 |     Params const& params,
 906 |     WorkTileInfo& work_tile_info,
 907 |     uint64_t linear_idx,
 908 |     uint32_t unit_iter_start,
 909 |     uint64_t cta_m_in_cluster,
 910 |     uint64_t cta_n_in_cluster) {
 911 |     // we divide up the linearized stream-K units into clusters and share the same K
 912 |     // offsets for work within clusters.
 913 |     uint64_t cluster_linear_work_idx = params.div_cluster_size(linear_idx);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 915-916

```cpp
 915 |     uint64_t unused, group_idx;
 916 |     params.divmod_sk_groups_(unused, group_idx, cluster_linear_work_idx);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 918-928

```cpp
 918 |     uint32_t unit_iter_end = unit_iter_start + work_tile_info.k_tile_remaining - 1;
 919 | 
 920 |     // Find the output tile corresponding to the final k tile covered by this
 921 |     // work unit. Stream-K work units will work backwards in terms of the tiles they
 922 |     // are responsible computing. This is beneficial because the final (partial)
 923 |     // tile computed by a stream-K block is typically the beginning of the output
 924 |     // tile, while the beginning (partial) tile is typically the ending of another
 925 |     // output tile. Since ending portions of an output tile must reduce across
 926 |     // other work units computing portions of that output tile, it is preferable
 927 |     // for them to be computed later, so as to reduce the likelihood of blocking
 928 |     // on other work.
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 930-932

```cpp
 930 |     auto output_tile_id_in_group = params.divmod_tiles_per_output_tile_.divide(unit_iter_end);
 931 |     uint32_t output_tile_iter_start = output_tile_id_in_group * params.divmod_tiles_per_output_tile_.divisor;
 932 |     uint32_t output_tile_iter_end = output_tile_iter_start + params.divmod_tiles_per_output_tile_.divisor;
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 934-936

```cpp
 934 |     // Convert the output tile from the linearized space within each group to the
 935 |     // overall linearized space.
 936 |     uint64_t output_tile_id = (output_tile_id_in_group * params.divmod_sk_groups_.divisor) + group_idx;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 938-939

```cpp
 938 |     // Bring the linearized tile ID back into the space of tiles, rather than clusters
 939 |     output_tile_id *= params.get_cluster_size();
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 941-951

```cpp
 941 |     // The final linearized tile ID is in units of the cluster dimension over which we rasterize.
 942 |     if (params.raster_order_ == RasterOrder::AlongN) {
 943 |       output_tile_id += cta_n_in_cluster * params.divmod_cluster_shape_minor_.divisor;
 944 |     }
 945 |     else {
 946 |       output_tile_id += cta_m_in_cluster * params.divmod_cluster_shape_minor_.divisor;
 947 |     }
 948 |     // The unit's starting k iteration in the current tile is either the starting
 949 |     // iteration for the tile as a whole, or the starting k iteration for the unit
 950 |     // as a whole (if the latter is greater than the former).
 951 |     uint32_t tile_iter_start = max(output_tile_iter_start, unit_iter_start);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 953-956

```cpp
 953 |     // Similarly, the unit's ending k iteration (exclusive) is either the end of
 954 |     // the current tile it is assigned, or the ending iteration of the unit as a whole
 955 |     // (if the latter is less than the former).
 956 |     uint32_t tile_iter_end = min(output_tile_iter_end, unit_iter_end + 1);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 958-960

```cpp
 958 |     // Set the k offset to be the starting k tile for this output tile
 959 |     work_tile_info.K_idx = static_cast<int32_t>(tile_iter_start - output_tile_iter_start);
 960 |     work_tile_info.k_tile_count = tile_iter_end - tile_iter_start;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 962-972

```cpp
 962 |     return output_tile_id;
 963 |   }
 964 |   // Given output tile index, update M, N, L index of current work tile info.
 965 |   CUTLASS_DEVICE
 966 |   static void
 967 |   update_work_tile_m_n_l(
 968 |     Params const& params,
 969 |     uint32_t output_tile_id,
 970 |     WorkTileInfo& work_tile_info,
 971 |     uint64_t cta_m_in_cluster,
 972 |     uint64_t cta_n_in_cluster) {
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 974-975

```cpp
 974 |     uint64_t work_idx_l, remainder;
 975 |     params.divmod_batch_(work_idx_l, remainder, output_tile_id);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 977-977

```cpp
 977 |     uint64_t cta_per_grid_dim = params.divmod_cluster_shape_minor_.divide(remainder);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 979-988

```cpp
 979 |     auto [work_idx_m, work_idx_n] = UnderlyingScheduler::get_work_idx_m_and_n(
 980 |                                           cta_per_grid_dim,
 981 |                                           params.divmod_cluster_shape_major_,
 982 |                                           params.divmod_cluster_shape_minor_,
 983 |                                           params.divmod_cluster_blk_major_,
 984 |                                           params.log_swizzle_size_,
 985 |                                           params.raster_order_
 986 |                                           , cta_m_in_cluster  
 987 |                                           , cta_n_in_cluster  
 988 |                                         );
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 990-994

```cpp
 990 |     // Set the M, N, and L block offsets
 991 |     work_tile_info.M_idx = work_idx_m;
 992 |     work_tile_info.N_idx = work_idx_n;
 993 |     work_tile_info.L_idx = static_cast<int32_t>(work_idx_l);
 994 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 996-1006

```cpp
 996 |   // Sets the current stream-K work to compute within work_tile_info. If new_unit is true, work_tile_info
 997 |   // is populated as a new unit of work. Otherwise, state existing in work_tile_info (e.g., remaining
 998 |   // iterations) is used to find the next tile in the current work unit.
 999 |   CUTLASS_DEVICE
1000 |   static void
1001 |   assign_work(
1002 |     Params const& params,
1003 |     uint64_t linear_idx,
1004 |     dim3 block_id_in_cluster,
1005 |     WorkTileInfo& work_tile_info,
1006 |     uint32_t &unit_iter_start) {
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 1008-1009

```cpp
1008 |     auto [cta_m_in_cluster, cta_n_in_cluster, _] =
1009 |       get_current_work_cta_m_n_in_cluster(params, linear_idx, block_id_in_cluster);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 1011-1020

```cpp
1011 |     uint64_t output_tile_id = linear_idx;
1012 |     if (linear_idx >= params.units_per_problem_ * params.divmod_splits_.divisor) {
1013 |       // Separate-reduction work
1014 |       auto cluster_size = params.get_cluster_size();
1015 |       // Divide up the linearized separate reduction units into clusters
1016 |       uint64_t cluster_linear_reduction_unit_idx = params.div_cluster_size((linear_idx - params.units_per_problem_));
1017 |       uint64_t cluster_tile_idx, epi_subtile_idx;
1018 |       params.divmod_epilogue_subtile_(cluster_tile_idx, epi_subtile_idx, cluster_linear_reduction_unit_idx);
1019 |       // Bring the linearized tile ID back into the space of tiles, rather than clusters
1020 |       output_tile_id = cluster_tile_idx * cluster_size;
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 1022-1037

```cpp
1022 |       work_tile_info.setup_separate_reduction(epi_subtile_idx);
1023 |     }
1024 |     else if (linear_idx >= params.sk_units_ && params.divmod_splits_.divisor == 1) {
1025 |       // Data-parallel work
1026 |       output_tile_id = linear_idx - params.sk_units_ + params.sk_tiles_;
1027 |       work_tile_info.K_idx = 0;
1028 |       work_tile_info.k_tile_count = params.divmod_tiles_per_output_tile_.divisor;
1029 |       work_tile_info.k_tile_remaining = params.divmod_tiles_per_output_tile_.divisor;
1030 |     }
1031 |     else {
1032 |       unit_iter_start = get_current_work_iter_start_possible_update_work_tile_k_remaining(params, linear_idx, work_tile_info);
1033 |       output_tile_id = update_output_tile_id_and_work_tile_k(params, work_tile_info,
1034 |         linear_idx, unit_iter_start, cta_m_in_cluster, cta_n_in_cluster);
1035 |     }
1036 |     update_work_tile_m_n_l(params, output_tile_id, work_tile_info, cta_m_in_cluster, cta_n_in_cluster);
1037 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 1039-1049

```cpp
1039 |   // The fast path to get current output tile index then update fields of work tile info
1040 |   // when continuing current work tile is needed, since k tile starting index has precomputed
1041 |   // in the first time fetching current work tile.
1042 |   CUTLASS_DEVICE
1043 |   static void
1044 |   fast_assign_work(
1045 |     uint32_t unit_iter_start,
1046 |     Params const& params,
1047 |     uint64_t linear_idx,
1048 |     dim3 block_id_in_cluster,
1049 |     WorkTileInfo& work_tile_info) {
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 1051-1052

```cpp
1051 |     auto [cta_m_in_cluster, cta_n_in_cluster, _] =
1052 |       get_current_work_cta_m_n_in_cluster(params, linear_idx, block_id_in_cluster);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 1054-1055

```cpp
1054 |     uint64_t output_tile_id = update_output_tile_id_and_work_tile_k(params, work_tile_info,
1055 |       linear_idx, unit_iter_start, cta_m_in_cluster, cta_n_in_cluster);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 1057-1058

```cpp
1057 |     update_work_tile_m_n_l(params, output_tile_id, work_tile_info, cta_m_in_cluster, cta_n_in_cluster);
1058 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 1060-1068

```cpp
1060 |   // Returns the starting and ending peer ID of this tile
1061 |   CUTLASS_HOST_DEVICE
1062 |   static auto
1063 |   tile_peer_range(Params const& params, uint32_t tile_idx, WorkTileInfo const& work_tile_info) {
1064 |     uint32_t cur_k_tile = static_cast<uint32_t>(work_tile_info.K_idx);
1065 |     uint32_t tile_idx_in_cluster_path = params.div_cluster_size(tile_idx);
1066 |     uint32_t start_k_tile = params.divmod_tiles_per_output_tile_.divisor * tile_idx_in_cluster_path;
1067 |     uint32_t end_k_tile = start_k_tile + params.divmod_tiles_per_output_tile_.divisor - 1;
1068 |     uint32_t big_unit_k_tiles = params.big_units_ * (params.divmod_k_tiles_per_sk_unit_.divisor + 1);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 1070-1077

```cpp
1070 |     auto adjust_unit = [&](uint32_t k_tile, uint32_t unit_idx, uint32_t unit_k_start, uint32_t unit_k_end) {
1071 |       if (k_tile - start_k_tile < Params::min_iters_per_sk_unit_ &&
1072 |           unit_k_end - start_k_tile < Params::min_iters_per_sk_unit_) {
1073 |         // k_tile is within the first min_iters_per_sk_unit_ K tiles of this output tile,
1074 |         // and the stream-K unit computes fewer than min_iters_per_sk_unit_ K tiles for this
1075 |         // output tile. This work will thus be subsumed by the next stream-K unit.
1076 |         ++unit_idx;
1077 |       }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 1079-1087

```cpp
1079 |       if (end_k_tile + 1 - k_tile < Params::min_iters_per_sk_unit_ &&
1080 |           end_k_tile + 1 - unit_k_start < Params::min_iters_per_sk_unit_) {
1081 |         // k_tile is within the last min_iters_per_sk_unit_ K tiles of this output tile,
1082 |         // and the stream-K unit computes fewer than min_iters_per_sk_unit_ K tiles for this
1083 |         // output tile. This work will thus be subsumed by the previous stream-K unit.
1084 |         --unit_idx;
1085 |       }
1086 |       return unit_idx;
1087 |     };
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 1089-1107

```cpp
1089 |     // Lambda to find the ID of the stream-K unit that computes this K tile
1090 |     auto find_unit = [&](uint32_t k_tile) {
1091 |       if (k_tile < big_unit_k_tiles) {
1092 |         // The tile is within the "big unit range"
1093 |         uint32_t unit_idx = params.divmod_k_tiles_per_sk_big_unit_.divide(k_tile);
1094 |         uint32_t unit_k_start = unit_idx * params.divmod_k_tiles_per_sk_big_unit_.divisor;
1095 |         uint32_t unit_k_end = unit_k_start + params.divmod_k_tiles_per_sk_big_unit_.divisor;
1096 |         return static_cast<uint64_t>(adjust_unit(k_tile, unit_idx, unit_k_start, unit_k_end));
1097 |       }
1098 |       else {
1099 |         // The tile is after the "big unit range." Account for this by finding the "normal unit"
1100 |         // that it belongs to, and then offsetting by the number of big units
1101 |         uint32_t unit_idx_after_big_units = params.divmod_k_tiles_per_sk_unit_.divide(k_tile - big_unit_k_tiles);
1102 |         uint32_t unit_k_start = unit_idx_after_big_units * params.divmod_k_tiles_per_sk_unit_.divisor + (params.big_units_ * params.divmod_k_tiles_per_sk_big_unit_.divisor);
1103 |         uint32_t unit_k_end = unit_k_start + params.divmod_k_tiles_per_sk_unit_.divisor;
1104 |         uint32_t unit_idx = unit_idx_after_big_units + params.big_units_;
1105 |         return static_cast<uint64_t>(adjust_unit(k_tile, unit_idx, unit_k_start, unit_k_end));
1106 |       }
1107 |     };
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 1109-1111

```cpp
1109 |     return cute::make_tuple(find_unit(start_k_tile), find_unit(start_k_tile + cur_k_tile), find_unit(end_k_tile));
1110 |   }
1111 | };
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 1113-1113

```cpp
1113 | } // namespace cutlass::gemm::kernel::detail
```
**EN:** These lines open or close the nested CUTLASS namespaces so the declarations stay scoped to the GEMM kernel subsystem.
**CN:** 这些代码行用于打开或关闭 CUTLASS 的嵌套命名空间，使声明被限制在 GEMM 内核子系统内。

## Key Concepts / 关键概念
- Template metaprogramming / 模板元编程
- Tile scheduling / Tile 调度
- Persistent scheduling / 持久化调度
- Stream-K scheduling / Stream-K 调度
- SM90 architecture tuning / SM90 架构调优
- Runtime argument packing / 运行时参数打包
- Packed launch parameters / 打包后的启动参数
- Shared memory layout / 共享内存布局

## Dependencies / 依赖关系
- **Included headers / 引入头文件:** `cutlass/barrier.h`, `cutlass/block_striped.h`, `cutlass/fast_math.h`, `cutlass/gemm/kernel/sm90_tile_scheduler.hpp`, `cutlass/kernel_hardware_info.hpp`, `cute/layout.hpp`, `cute/tensor.hpp`
- **Subsystems / 子系统:** CuTe tensor utilities / CuTe 张量工具, Architecture tags and intrinsics / 架构标签与内建操作, Epilogue/output pipeline / Epilogue 输出流水, Threadblock MMA building blocks / 线程块级 MMA 构件, Tile schedulers / Tile 调度器
- **Peer kernel headers / 同级内核头文件:** `cutlass/gemm/kernel/sm90_tile_scheduler.hpp`
