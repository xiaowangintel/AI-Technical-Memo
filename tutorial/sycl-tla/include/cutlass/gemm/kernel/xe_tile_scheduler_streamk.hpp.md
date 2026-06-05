# xe_tile_scheduler_streamk.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `include/cutlass/gemm/kernel/xe_tile_scheduler_streamk.hpp`
- **Purpose / 用途 (EN):** Implements an Intel Xe-specific GEMM kernel or tile-scheduling component.
- **Purpose / 用途 (CN):** 实现面向 Intel Xe 的 GEMM 内核或 Tile 调度组件。
- **Line count / 行数:** 713

## Line-by-Line Analysis / 逐行分析
> The source is grouped into contiguous significant line ranges for readability. / 为了便于阅读，下面按连续的重要代码区间进行分析。

### Lines 1-30

```cpp
   1 | /***************************************************************************************************
   2 |  * Copyright (c) 2024 - 2024 Codeplay Software Ltd. All rights reserved.
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

### Lines 34-41

```cpp
  34 | #include "cutlass/barrier.h"
  35 | #include "cutlass/block_striped.h"
  36 | #include "cutlass/fast_math.h"
  37 | #include "cutlass/gemm/kernel/sm90_tile_scheduler.hpp"
  38 | #include "cutlass/kernel_hardware_info.hpp"
  39 | #include "cute/layout.hpp"
  40 | #include "cute/tensor.hpp"
  41 | #include "cutlass/gemm/kernel/xe_persistent_tile_scheduler_params_streamk.hpp"
```
**EN:** This include block imports `cutlass/barrier.h`, `cutlass/block_striped.h`, `cutlass/fast_math.h`, `cutlass/gemm/kernel/sm90_tile_scheduler.hpp`, `cutlass/kernel_hardware_info.hpp`, `cute/layout.hpp`, ... (+2 more), providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/barrier.h`, `cutlass/block_striped.h`, `cutlass/fast_math.h`, `cutlass/gemm/kernel/sm90_tile_scheduler.hpp`, `cutlass/kernel_hardware_info.hpp`, `cute/layout.hpp`, ... (+2 more)，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 43-43

```cpp
  43 | namespace cutlass::gemm::kernel::detail {
```
**EN:** These lines open or close the nested CUTLASS namespaces so the declarations stay scoped to the GEMM kernel subsystem.
**CN:** 这些代码行用于打开或关闭 CUTLASS 的嵌套命名空间，使声明被限制在 GEMM 内核子系统内。

### Lines 45-53

```cpp
  45 | // Persistent Thread Block (TB) scheduler leveraging stream-K decomposition
  46 | template <
  47 |   class TileShape,
  48 |   uint32_t ThreadsPerBlock
  49 | >
  50 | class PersistentTileSchedulerXeStreamK {
  51 |   //
  52 |   // Data members
  53 |   //
```
**EN:** This block declares or specializes `TileShape`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `TileShape`，它是该头文件中承载某一层内核策略的核心类。

### Lines 55-56

```cpp
  55 | private:
  56 |   uint64_t current_work_linear_idx_ = 0;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 58-58

```cpp
  58 | public:
```
**EN:** This comment-only block adds explanatory context for the surrounding implementation, such as layout assumptions, scheduling notes, or design rationale.
**CN:** 这一段纯注释内容为周围实现补充了解释性背景，例如布局假设、调度说明或设计动机。

### Lines 60-61

```cpp
  60 |   // Use a dummy barrier manager to simply get the type used to store the barrier
  61 |   using BarrierType = typename NamedBarrierManager<1>::T;
```
**EN:** This alias block derives concise type names `BarrierType` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `BarrierType` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 63-67

```cpp
  63 |   using Params = PersistentTileSchedulerXeStreamKParams;
  64 |   using ReductionMode = Params::ReductionMode;
  65 |   using DecompositionMode = Params::DecompositionMode;
  66 |   using RasterOrder = Params::RasterOrder;
  67 |   using RasterOrderOptions = Params::RasterOrderOptions;
```
**EN:** This alias block derives concise type names `Params`, `ReductionMode`, `DecompositionMode`, `RasterOrder`, `RasterOrderOptions` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Params`, `ReductionMode`, `DecompositionMode`, `RasterOrder`, `RasterOrderOptions` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 69-73

```cpp
  69 |   struct WorkTileInfo {
  70 |     int32_t M_idx = 0;
  71 |     int32_t N_idx = 0;
  72 |     int32_t K_idx = 0;
  73 |     int32_t L_idx = 0;
```
**EN:** This block declares or specializes `WorkTileInfo`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `WorkTileInfo`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 75-77

```cpp
  75 |     // Number of k tiles to compute for this unit of work. For stream-K, this
  76 |     // can indicate the number of K tiles across multiple output tiles.
  77 |     uint32_t k_tile_count = 0;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 79-80

```cpp
  79 |     // Number of k tiles remaining for the work unit as a whole
  80 |     uint32_t k_tile_remaining = 0;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 82-87

```cpp
  82 |     CUTLASS_HOST_DEVICE
  83 |     bool
  84 |     is_valid() const {
  85 |       // A work tile that computes no K tiles is invalid
  86 |       return k_tile_count > 0;
  87 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 89-93

```cpp
  89 |     CUTLASS_HOST_DEVICE
  90 |     static WorkTileInfo
  91 |     invalid_work_tile() {
  92 |       return {-1, -1, -1, -1, 0};
  93 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 95-100

```cpp
  95 |     CUTLASS_HOST_DEVICE
  96 |     bool
  97 |     is_final_split(uint32_t k_tiles_per_output_tile) const {
  98 |       return (K_idx + k_tile_count) == k_tiles_per_output_tile;
  99 |     }
 100 |   };
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 102-102

```cpp
 102 |   struct Arguments {
```
**EN:** This `Arguments` structure collects the host-facing runtime inputs: problem shape, pointers, strides, epilogue settings, hardware info, and scheduler data.
**CN:** 这个 `Arguments` 结构体收集面向主机侧的运行时输入，包括问题规模、数据指针、步长、epilogue 配置、硬件信息以及调度器数据。

### Lines 104-106

```cpp
 104 |     Arguments() = default;
 105 |     Arguments(Arguments const&) = default;
 106 |     Arguments(Arguments&&) = default;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 108-115

```cpp
 108 |     CUTLASS_HOST_DEVICE
 109 |     Arguments&
 110 |     operator=(Arguments const& args) {
 111 |       splits = args.splits;
 112 |       reduction_mode = args.reduction_mode;
 113 |       decomposition_mode = args.decomposition_mode;
 114 |       return *this;
 115 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 117-124

```cpp
 117 |     CUTLASS_HOST_DEVICE
 118 |     Arguments&
 119 |     operator=(Arguments&& args) noexcept {
 120 |       splits = args.splits;
 121 |       reduction_mode = args.reduction_mode;
 122 |       decomposition_mode = args.decomposition_mode;
 123 |       return *this;
 124 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 126-127

```cpp
 126 |     CUTLASS_HOST_DEVICE
 127 |     Arguments(int splits_) : splits(splits_) {}
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 129-132

```cpp
 129 |     CUTLASS_HOST_DEVICE
 130 |     Arguments(int splits_, DecompositionMode decomposition_mode_) :
 131 |       splits(splits_),
 132 |       decomposition_mode(decomposition_mode_) {}
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 134-142

```cpp
 134 |     // The splitting factor to be used in a split-K decomposition of the problem.
 135 |     // If this is set to a value greater than 1, stream-K decomposition logic
 136 |     // is bypassed in favor of a split-K decomposition.
 137 |     int splits = 1;
 138 |     RasterOrderOptions raster_order = RasterOrderOptions::Heuristic;
 139 |     ReductionMode reduction_mode = ReductionMode::Deterministic;
 140 |     DecompositionMode decomposition_mode = DecompositionMode::Heuristic;
 141 |     int max_swizzle_size = 1;
 142 |   };
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 144-149

```cpp
 144 |   // Sink scheduler params as a member
 145 |   Params scheduler_params;
 146 | 
 147 |   //
 148 |   // Methods
 149 |   //
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 151-161

```cpp
 151 |   template <class ProblemShape, class ClusterShape>
 152 |   static Params
 153 |   to_underlying_arguments(
 154 |     ProblemShape problem_shape,
 155 |     TileShape tile_shape,
 156 |     [[maybe_unused]] ClusterShape cluster_shape,
 157 |     KernelHardwareInfo const& hw_info,
 158 |     Arguments const& args,
 159 |     void* workspace,
 160 |     [[maybe_unused]] const uint32_t epilogue_subtile = 1,
 161 |     [[maybe_unused]] uint32_t ktile_start_alignment_count = 1u) {
```
**EN:** This helper converts the public argument bundle into lower-level mainloop, epilogue, and scheduler parameters expected by the actual kernel implementation.
**CN:** 这个辅助函数把对外的参数包转换成底层内核实现真正需要的 mainloop、epilogue 和调度器参数。

### Lines 163-163

```cpp
 163 |     static_assert(cute::is_static<TileShape>::value);
```
**EN:** These compile-time checks enforce invariants such as problem-rank assumptions, supported scheduler tags, or required tensor-stride shapes.
**CN:** 这些编译期检查用于保证问题维度假设、可支持的调度器标签以及张量步长形状等不变式成立。

### Lines 165-167

```cpp
 165 |     auto problem_shape_mnkl = cute::append<4>(problem_shape, cute::Int<1>{});
 166 |     dim3 problem_blocks = get_tiled_wg_shape_mnl(problem_shape_mnkl, tile_shape);
 167 |     uint32_t k_tile_per_output_tile = cute::size(cute::ceil_div(cute::shape<2>(problem_shape_mnkl), cute::shape<2>(TileShape{})));
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 169-180

```cpp
 169 |     Params params;
 170 |     params.initialize(
 171 |       problem_blocks,
 172 |       k_tile_per_output_tile,
 173 |       hw_info,
 174 |       args.splits,
 175 |       args.reduction_mode,
 176 |       args.decomposition_mode,
 177 |       workspace
 178 |     );
 179 |     return params;
 180 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 182-188

```cpp
 182 |   static bool
 183 |   can_implement(Arguments const& args) {
 184 |     // Split count > 1 is only valid for heuristic and split-K decomposition modes
 185 |     return (args.splits == 1 ||
 186 |             args.decomposition_mode == DecompositionMode::Heuristic ||
 187 |             args.decomposition_mode == DecompositionMode::SplitK);
 188 |   }
```
**EN:** This function checks whether the requested problem, layouts, and runtime options satisfy the constraints of the specialized kernel.
**CN:** 该函数检查请求的问题规模、布局和运行时选项是否满足当前特化内核的约束条件。

### Lines 190-191

```cpp
 190 |   CUTLASS_HOST_DEVICE
 191 |   PersistentTileSchedulerXeStreamK() { };
```
**EN:** This block continues the scheduler logic, Stream-K flow defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑、Stream-K 流程相关逻辑。

### Lines 193-196

```cpp
 193 |   CUTLASS_HOST_DEVICE
 194 |   PersistentTileSchedulerXeStreamK(Params const& params_) : scheduler_params(params_) {
 195 |     current_work_linear_idx_ = uint64_t(BlockIdxX());
 196 |   }
```
**EN:** This block continues the scheduler logic, Stream-K flow defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑、Stream-K 流程相关逻辑。

### Lines 198-202

```cpp
 198 |   CUTLASS_DEVICE
 199 |   WorkTileInfo
 200 |   get_current_work() const {
 201 |     return get_current_work_for_linear_idx(current_work_linear_idx_, scheduler_params);
 202 |   }
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 204-216

```cpp
 204 |   CUTLASS_DEVICE
 205 |   static WorkTileInfo
 206 |   get_current_work_for_linear_idx(uint64_t linear_idx, Params const& params) {
 207 |     // The maximum number of work units is units_per_problem_ * splits_.
 208 |     // The multiplication by splits_ is used for handling split-K, in which
 209 |     // units_per_problem_ is equal to the total number of output tiles. To account
 210 |     // for the fact that we have splits_ peers per output tile, we multiply this
 211 |     // value by splits_. For stream-K, this multiplication ends up being a no-op
 212 |     // because splits_ is set to 1 for stream-K.
 213 |     if(linear_idx >= (params.units_per_problem_ * params.divmod_splits_.divisor)) {
 214 |       // Invalid work. Return an empty result.
 215 |       return WorkTileInfo::invalid_work_tile();
 216 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 218-221

```cpp
 218 |     WorkTileInfo work_tile_info;
 219 |     assign_work(params, linear_idx, work_tile_info);
 220 |     return work_tile_info;
 221 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 223-232

```cpp
 223 |   // Returns whether the current work_tile_info passed in should continue to be used. This
 224 |   // occurs only in the stream-K decomposition with stream-K work units, which encompass
 225 |   // work over multiple output tiles. If the current work_tile_info should continue to be
 226 |   // used, it is updated to advance to the next output tile it should cover.
 227 |   CUTLASS_DEVICE
 228 |   bool
 229 |   continue_current_work(WorkTileInfo& work_tile_info) const {
 230 |     return continue_current_work_for_linear_idx(
 231 |       current_work_linear_idx_, work_tile_info, scheduler_params);
 232 |   }
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 234-239

```cpp
 234 |   CUTLASS_DEVICE
 235 |   static bool
 236 |   continue_current_work_for_linear_idx(
 237 |     uint64_t linear_idx,
 238 |     WorkTileInfo& work_tile_info,
 239 |     Params const& params) {
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 241-241

```cpp
 241 |     work_tile_info.k_tile_remaining -= work_tile_info.k_tile_count;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 243-248

```cpp
 243 |     if (work_tile_info.k_tile_remaining == 0) {
 244 |       return false;
 245 |     }
 246 |     assign_work(params, linear_idx, work_tile_info);
 247 |     return work_tile_info.is_valid();
 248 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 250-254

```cpp
 250 |   CUTLASS_DEVICE
 251 |   void
 252 |   advance_to_next_work(uint32_t advance_count = 1) {
 253 |     current_work_linear_idx_ += uint64_t(GridDimX()) * uint64_t(GridDimY()) * uint64_t(GridDimZ()) * uint64_t(advance_count);
 254 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 256-262

```cpp
 256 |   // Given the inputs, computes the total number of output work-groups this problem will compute over.
 257 |   template <class ProblemShape>
 258 |   CUTLASS_HOST_DEVICE static
 259 |   dim3
 260 |   get_tiled_wg_shape_mnl(ProblemShape problem_shape_mnkl, TileShape cta_shape) {
 261 |     return Params::get_tiled_wg_shape_mnl(to_gemm_coord(problem_shape_mnkl), to_gemm_coord(cta_shape));
 262 |   }
```
**EN:** This block declares or specializes `ProblemShape`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `ProblemShape`，它是该头文件中承载某一层内核策略的核心类。

### Lines 264-275

```cpp
 264 |   // Computes the physical grid we should launch.
 265 |   template <class ProblemShape, class ClusterShape>
 266 |   CUTLASS_HOST_DEVICE static
 267 |   dim3
 268 |   get_grid_shape(
 269 |     [[maybe_unused]] Params const& params,
 270 |     ProblemShape problem_shape,
 271 |     TileShape tile_shape,
 272 |     [[maybe_unused]] ClusterShape cluster_shape,
 273 |     KernelHardwareInfo hw_info,
 274 |     [[maybe_unused]] Arguments arguments = Arguments{},
 275 |     [[maybe_unused]] bool truncate_by_problem_size=true) {
```
**EN:** These launch helpers compute the CUDA/SYCL execution geometry from the problem shape, tile shape, and scheduling policy.
**CN:** 这些启动辅助函数根据问题规模、Tile 形状和调度策略计算 CUDA/SYCL 的执行网格。

### Lines 277-278

```cpp
 277 |     auto problem_shape_mnkl = cute::append<4>(problem_shape, cute::Int<1>{});
 278 |     dim3 problem_blocks = get_tiled_wg_shape_mnl(problem_shape_mnkl, tile_shape);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 280-284

```cpp
 280 |     return Params::get_grid_shape(
 281 |       problem_blocks,
 282 |       hw_info
 283 |     );
 284 |   }
```
**EN:** These launch helpers compute the CUDA/SYCL execution geometry from the problem shape, tile shape, and scheduling policy.
**CN:** 这些启动辅助函数根据问题规模、Tile 形状和调度策略计算 CUDA/SYCL 的执行网格。

### Lines 286-292

```cpp
 286 |   // Returns whether fixup is needed for `work_tile_info`.
 287 |   CUTLASS_HOST_DEVICE
 288 |   static bool
 289 |   requires_fixup(Params const& params, WorkTileInfo const& work_tile_info) {
 290 |     // Fixup is not needed for invalid or data-parallel tiles
 291 |     return work_tile_info.is_valid() && work_tile_info.k_tile_count != params.divmod_tiles_per_output_tile_.divisor;
 292 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 294-309

```cpp
 294 |   // Performs the reduction across splits for a given output tile.
 295 | template <class FrgTensorC>
 296 |   CUTLASS_DEVICE
 297 |   static void
 298 |   fixup(
 299 |     Params const& params,
 300 |     WorkTileInfo const& work_tile_info,
 301 |     FrgTensorC& accumulators,
 302 |     uint32_t num_barriers = 1,
 303 |     uint32_t barrier_idx = 0) {
 304 |     static constexpr uint32_t Offset = static_cast<int>(cutlass::arch::ReservedNamedBarriers::StreamkBarrier0);
 305 |     static constexpr uint32_t MaxNumNamedBarriers = 1;
 306 |     using BarrierManager = NamedBarrierManager<ThreadsPerBlock, Offset, MaxNumNamedBarriers>;
 307 |     return fixup_helper<FrgTensorC, BarrierManager>(
 308 |       params, work_tile_info, accumulators, num_barriers, barrier_idx);
 309 |   }
```
**EN:** This alias block derives concise type names `BarrierManager` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `BarrierManager` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 311-321

```cpp
 311 |   // Helper for performing the reduction across splits for a given output tile.
 312 |   template <class FrgTensorC, class BarrierManager>
 313 |   CUTLASS_DEVICE
 314 |   static void
 315 |   fixup_helper(
 316 |     Params const& params,
 317 |     WorkTileInfo const& work_tile_info,
 318 |     FrgTensorC& accumulators,
 319 |     uint32_t num_barriers,
 320 |     uint32_t barrier_idx,
 321 |     uint32_t num_accumulator_mtxs = 1) {
```
**EN:** This block declares or specializes `FrgTensorC`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `FrgTensorC`，它是该头文件中承载某一层内核策略的核心类。

### Lines 323-323

```cpp
 323 |     using ElementAccumulator = typename FrgTensorC::value_type;
```
**EN:** This alias block derives concise type names `ElementAccumulator` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `ElementAccumulator` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 325-328

```cpp
 325 |     if (!requires_fixup(params, work_tile_info)) {
 326 |       return;
 327 |     }
 328 |     auto tile_idx = output_tile_index(params, work_tile_info);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 330-331

```cpp
 330 |     // Index of the lock on which to wait
 331 |     auto lock_idx = (tile_idx * num_barriers) + barrier_idx;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 333-335

```cpp
 333 |     auto reduction_tile_idx = tile_idx;
 334 |     auto reduction_peer_offset = 0;
 335 |     int barrier_group_thread_idx = ThreadIdxX();
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 337-341

```cpp
 337 |     // Reductions use BlockStripedReduce with a width of BarrierManager::ThreadCount under the hood.
 338 |     // Thus, the start of the reduction space is the same across all threads in a work group.
 339 |     int reduction_offset =
 340 |       (cute::size<0>(TileShape{}) * cute::size<1>(TileShape{}) * reduction_tile_idx * num_accumulator_mtxs) +
 341 |       reduction_peer_offset;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 343-343

```cpp
 343 |     ElementAccumulator* group_reduction_workspace = reinterpret_cast<ElementAccumulator*>(params.reduction_workspace_) + reduction_offset;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 345-346

```cpp
 345 |     using AccumulatorArrayT = Array<typename FrgTensorC::value_type, size(FrgTensorC{})>;
 346 |     using BlockStripedReduceT = BlockStripedReduce<ThreadsPerBlock, AccumulatorArrayT>;
```
**EN:** This alias block derives concise type names `AccumulatorArrayT`, `BlockStripedReduceT` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `AccumulatorArrayT`, `BlockStripedReduceT` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 348-349

```cpp
 348 |     AccumulatorArrayT* reduction_workspace_array = reinterpret_cast<AccumulatorArrayT*>(group_reduction_workspace);
 349 |     AccumulatorArrayT* accumulator_array = reinterpret_cast<AccumulatorArrayT*>(accumulators.data());
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 351-363

```cpp
 351 |     // The number of tiles for which reduction is required is either:
 352 |     //   (a) the total number of output tiles (in the case of split-K)
 353 |     //   (b) the number of stream-K tiles
 354 |     // To calculate the total number of output tiles in the split-K case, we
 355 |     // note that, in the split-K case, the units_per_problem_ member of Params will be
 356 |     // the total number of output tiles.
 357 |     uint32_t reduction_tiles = 0;
 358 |     if (params.divmod_splits_.divisor > 1) {
 359 |       reduction_tiles = params.units_per_problem_;
 360 |     }
 361 |     else {
 362 |       reduction_tiles = params.sk_tiles_;
 363 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 365-368

```cpp
 365 |     auto reduction_workspace_size = Params::get_reduction_workspace_size(
 366 |       reduction_tiles, to_gemm_coord(TileShape{}), sizeof_bits<ElementAccumulator>::value, num_accumulator_mtxs);
 367 |     BarrierType* lock_workspace = reinterpret_cast<BarrierType*>(
 368 |       reinterpret_cast<uint8_t*>(params.reduction_workspace_) + reduction_workspace_size);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 370-377

```cpp
 370 |     if (!compute_epilogue(work_tile_info, params)) {
 371 |       if (work_tile_info.K_idx == 0) {
 372 |         // The first peer initializes the workspace partials
 373 |         BlockStripedReduceT::store(reduction_workspace_array, *accumulator_array, barrier_group_thread_idx);
 374 |       }
 375 |       else {
 376 |         // Wait until the preceding split added its accumulators
 377 |         BarrierManager::wait_eq(barrier_idx, lock_workspace, barrier_group_thread_idx, lock_idx, work_tile_info.K_idx);
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 379-381

```cpp
 379 |         // Perform reduction in workspace
 380 |         BlockStripedReduceT::reduce(reduction_workspace_array, *accumulator_array, barrier_group_thread_idx);
 381 |       }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 383-385

```cpp
 383 |       // Each participating stream-K unit increments the barrier by the K tile count that this unit has
 384 |       // processed.
 385 |       int32_t increment = work_tile_info.k_tile_count;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 387-398

```cpp
 387 |       // Signal our arrival
 388 |       BarrierManager::arrive_inc(barrier_idx, lock_workspace, barrier_group_thread_idx, lock_idx, increment);
 389 |     }
 390 |     else {
 391 |       if (params.reduction_mode_ == ReductionMode::Deterministic) {
 392 |         // Wait until the preceding split added its accumulators
 393 |         BarrierManager::wait_eq(barrier_idx, lock_workspace, barrier_group_thread_idx, lock_idx, work_tile_info.K_idx);
 394 |       }
 395 |       else {
 396 |         // Wait until the first split has stored its accumulators
 397 |         BarrierManager::wait_lt(barrier_idx, lock_workspace, barrier_group_thread_idx, lock_idx, 1);
 398 |       }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 400-404

```cpp
 400 |       // The block computing the final split for the tile adds previously-reduced partials
 401 |       // to its accumulators and computes the epilogue.
 402 |       BlockStripedReduceT::load_add(*accumulator_array, reduction_workspace_array, barrier_group_thread_idx);
 403 |     }
 404 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 406-416

```cpp
 406 |   // Returns whether the block assigned this work should compute the epilogue for the corresponding
 407 |   // output tile. For the case of stream-K, this should only occur if the work is marked as the final split.
 408 |   CUTLASS_HOST_DEVICE
 409 |   static bool
 410 |   compute_epilogue(WorkTileInfo const& work_tile_info, Params const& params) {
 411 |     // `is_final_split` will be set to `true` for the following scenarios, all of which must compute the epilogue:
 412 |     //  1. The tile is computed in data-parallel mode
 413 |     //  2. The tile is computed in split-/stream-K mode and this work unit represents the final split of the tile
 414 |     return work_tile_info.is_valid() &&
 415 |             work_tile_info.is_final_split(params.divmod_tiles_per_output_tile_.divisor);
 416 |   }
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 418-425

```cpp
 418 |   // Returns the linearized index of the output tile corresponding to the tile with offset [L, M, K]
 419 |   CUTLASS_DEVICE
 420 |   static int
 421 |   output_tile_index(Params const& params, WorkTileInfo const& work_tile_info) {
 422 |     uint64_t linear_idx_in_batch = Params::get_linear_idx_from_m_and_n(
 423 |       work_tile_info.M_idx, work_tile_info.N_idx,
 424 |       params.divmod_blk_major_
 425 |     );
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 427-429

```cpp
 427 |     uint64_t tiles_mn = params.divmod_batch_.divisor;
 428 |     return tiles_mn * work_tile_info.L_idx + linear_idx_in_batch;
 429 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 431-439

```cpp
 431 |   template <class ProblemShape, class ElementAccumulator>
 432 |   static size_t
 433 |   get_workspace_size(
 434 |     Arguments const& args,
 435 |     ProblemShape problem_shape,
 436 |     KernelHardwareInfo const& hw_info,
 437 |     [[maybe_unused]] uint32_t mma_warp_groups = 1,
 438 |     [[maybe_unused]] const uint32_t epilogue_subtile = 1,
 439 |     [[maybe_unused]] uint32_t num_accumulator_mtxs = 1) {
```
**EN:** This block reports or initializes any auxiliary workspace needed by the kernel before launch.
**CN:** 这一段用于报告或初始化内核启动前所需的辅助工作区。

### Lines 441-441

```cpp
 441 |     auto problem_shape_mnkl = cute::append<4>(problem_shape, 1);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 443-443

```cpp
 443 |     TileShape tile_shape;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 445-446

```cpp
 445 |     dim3 problem_blocks = get_tiled_wg_shape_mnl(problem_shape_mnkl, tile_shape);
 446 |     uint32_t k_tile_per_output_tile = cute::size(cute::ceil_div(cute::shape<2>(problem_shape_mnkl), cute::shape<2>(TileShape{})));
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 448-458

```cpp
 448 |     return Params::get_workspace_size(
 449 |       problem_blocks,
 450 |       k_tile_per_output_tile,
 451 |       to_gemm_coord(tile_shape),
 452 |       hw_info,
 453 |       args.splits,
 454 |       args.decomposition_mode,
 455 |       sizeof_bits<BarrierType>::value,
 456 |       sizeof_bits<ElementAccumulator>::value
 457 |     );
 458 |   }
```
**EN:** This block reports or initializes any auxiliary workspace needed by the kernel before launch.
**CN:** 这一段用于报告或初始化内核启动前所需的辅助工作区。

### Lines 460-471

```cpp
 460 |   template <class ProblemShape, class ElementAccumulator>
 461 |   static cutlass::Status
 462 |   initialize_workspace(
 463 |     Arguments const& args,
 464 |     void* workspace,
 465 |     cudaStream_t stream,
 466 |     ProblemShape const& problem_shape,
 467 |     KernelHardwareInfo const& hw_info,
 468 |     [[maybe_unused]] uint32_t mma_warp_groups = 1,
 469 |     [[maybe_unused]] const uint32_t epilogue_subtile = 1,
 470 |     [[maybe_unused]] uint32_t num_accumulator_mtxs = 1,
 471 |     [[maybe_unused]] CudaHostAdapter* cuda_adapter = nullptr) {
```
**EN:** This block reports or initializes any auxiliary workspace needed by the kernel before launch.
**CN:** 这一段用于报告或初始化内核启动前所需的辅助工作区。

### Lines 473-473

```cpp
 473 |     auto problem_shape_mnkl = cute::append<4>(problem_shape, 1);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 475-475

```cpp
 475 |     TileShape tile_shape;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 477-478

```cpp
 477 |     dim3 problem_blocks = get_tiled_wg_shape_mnl(problem_shape_mnkl, tile_shape);
 478 |     uint32_t k_tile_per_output_tile = cute::size(cute::ceil_div(cute::shape<2>(problem_shape_mnkl), cute::shape<2>(TileShape{})));
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 480-492

```cpp
 480 |     return Params::initialize_workspace(
 481 |       workspace,
 482 |       problem_blocks,
 483 |       k_tile_per_output_tile,
 484 |       to_gemm_coord(tile_shape),
 485 |       hw_info,
 486 |       args.splits,
 487 |       args.decomposition_mode,
 488 |       sizeof_bits<BarrierType>::value,
 489 |       sizeof_bits<ElementAccumulator>::value,
 490 |       stream
 491 |     );
 492 |   }
```
**EN:** This block reports or initializes any auxiliary workspace needed by the kernel before launch.
**CN:** 这一段用于报告或初始化内核启动前所需的辅助工作区。

### Lines 494-499

```cpp
 494 |   template <class ProblemShape>
 495 |   CUTLASS_HOST_DEVICE
 496 |   static int
 497 |   get_work_k_tile_count(WorkTileInfo const& work_tile_info, ProblemShape, TileShape) {
 498 |     return work_tile_info.k_tile_count;
 499 |   }
```
**EN:** This block declares or specializes `ProblemShape`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `ProblemShape`，它是该头文件中承载某一层内核策略的核心类。

### Lines 501-505

```cpp
 501 |   CUTLASS_HOST_DEVICE
 502 |   static uint32_t
 503 |   get_work_k_tile_start(WorkTileInfo const& work_tile_info) {
 504 |     return work_tile_info.K_idx;
 505 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 507-513

```cpp
 507 |   // Kernel helper function to get next work tile
 508 |   CUTLASS_DEVICE
 509 |   auto
 510 |   fetch_next_work(WorkTileInfo work_tile_info) {
 511 |     if (continue_current_work(work_tile_info)) {
 512 |       return cute::make_tuple(work_tile_info, true);
 513 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 515-517

```cpp
 515 |     advance_to_next_work();
 516 |     return cute::make_tuple(get_current_work(), true);
 517 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 519-525

```cpp
 519 |   // Returns the initial work tile info that will be computed over
 520 |   template <class ClusterShape>
 521 |   CUTLASS_DEVICE
 522 |   WorkTileInfo
 523 |   initial_work_tile_info(ClusterShape cluster_shape) {
 524 |     return get_current_work();
 525 |   }
```
**EN:** This block declares or specializes `ClusterShape`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `ClusterShape`，它是该头文件中承载某一层内核策略的核心类。

### Lines 527-536

```cpp
 527 | private:
 528 |   // Sets the current stream-K work to compute within work_tile_info. If new_unit is true, work_tile_info
 529 |   // is populated as a new unit of work. Otherwise, state existing in work_tile_info (e.g., remaining
 530 |   // iterations) is used to find the next tile in the current work unit.
 531 | CUTLASS_DEVICE
 532 |   static void
 533 |   assign_work(
 534 |     Params const& params,
 535 |     uint64_t linear_idx,
 536 |     WorkTileInfo& work_tile_info) {
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 538-546

```cpp
 538 |     uint64_t output_tile_id = linear_idx;
 539 |     if (linear_idx >= params.sk_units_ && params.divmod_splits_.divisor == 1) {
 540 |       // Data-parallel work
 541 |       output_tile_id = linear_idx - params.sk_units_ + params.sk_tiles_;
 542 |       work_tile_info.K_idx = 0;
 543 |       work_tile_info.k_tile_count = params.divmod_tiles_per_output_tile_.divisor;
 544 |       work_tile_info.k_tile_remaining = params.divmod_tiles_per_output_tile_.divisor;
 545 |     }
 546 |     else {
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 548-553

```cpp
 548 |       // Determine whether we are in a "big unit" within the group, that will process
 549 |       // an additional K chunk in the group.
 550 |       auto sk_tiles_in_group = params.sk_tiles_;
 551 |       auto k_tiles_in_group = sk_tiles_in_group * params.divmod_tiles_per_output_tile_.divisor;
 552 |       auto k_tiles_per_unit_in_group = params.divmod_sk_units_per_group_.divide(k_tiles_in_group);
 553 |       auto big_units_in_group = k_tiles_in_group - (k_tiles_per_unit_in_group * params.divmod_sk_units_per_group_.divisor);
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 555-556

```cpp
 555 |       uint64_t split;
 556 |       params.divmod_sk_units_per_group_(split, output_tile_id, output_tile_id);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 558-562

```cpp
 558 |       bool is_split_k = params.divmod_splits_.divisor > 1;
 559 |       auto big_unit_cmp_lhs = is_split_k ? split : output_tile_id;
 560 |       auto big_unit_cmp_rhs = is_split_k ? params.big_units_ : big_units_in_group;
 561 |       auto linear_idx_mult = is_split_k ? params.divmod_tiles_per_output_tile_.divisor : k_tiles_per_unit_in_group;
 562 |       auto k_tiles_per_split = is_split_k ? params.divmod_k_tiles_per_sk_unit_.divisor : k_tiles_per_unit_in_group;
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 564-565

```cpp
 564 |       // Determine the starting k iteration computed by this stream-K work unit
 565 |       uint32_t unit_iter_start = (linear_idx_mult * linear_idx) + (k_tiles_per_split * split);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 567-582

```cpp
 567 |       // Adjust the starting position and number of k iterations for "big units," which
 568 |       // compute one extra iteration. If there are any big units, they will be the first
 569 |       // in the linearized ID space.
 570 |       auto k_tiles_in_my_split = k_tiles_per_split;
 571 |       if (big_unit_cmp_lhs < big_unit_cmp_rhs) {
 572 |         // Since the "big units" are the first units in the linearized ID space, each
 573 |         // of the units preceding this big unit computed one extra iteration. Thus,
 574 |         // we must offset our start iteration by the number of units that precede
 575 |         // the current unit in the linearized ID space.
 576 |         unit_iter_start += big_unit_cmp_lhs;
 577 |         ++k_tiles_in_my_split;
 578 |       }
 579 |       else {
 580 |         // Increment by one for each of the big clusters (since all big units precede this unit)
 581 |         unit_iter_start += big_unit_cmp_rhs;
 582 |       }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 584-603

```cpp
 584 |       if (!is_split_k) {
 585 |         // Adjust the unit starting position and number of tiles to avoid
 586 |         // computing splits of size less than min_iters_per_sk_unit_
 587 |         int unused, start_tile_k_tile;
 588 |         params.divmod_tiles_per_output_tile_(unused, start_tile_k_tile, unit_iter_start);
 589 |         if (start_tile_k_tile < Params::min_iters_per_sk_unit_) {
 590 |           // Starting K tile is in range [0, Params::min_iters_per_sk_unit_), which means that another
 591 |           // stream-K unit will be computing a split with fewer than Params::min_iters_per_sk_unit_ K tiles.
 592 |           // Adjust our work to take over these K tiles.
 593 |           unit_iter_start -= start_tile_k_tile;
 594 |           k_tiles_in_my_split += start_tile_k_tile;
 595 |         }
 596 |         else if (start_tile_k_tile > (params.divmod_tiles_per_output_tile_.divisor - Params::min_iters_per_sk_unit_)) {
 597 |           // Starting K tile is within the final Params::min_iters_per_sk_unit_ K tiles of some output tile,
 598 |           // which means that this unit will compute a split with fewer than Params::min_iters_per_sk_unit_ K tiles.
 599 |           // Adjust our work to shed these K tiles to a neighboring stream-K unit that will compute more consecutive K tiles.
 600 |           auto adjustment_tiles = (params.divmod_tiles_per_output_tile_.divisor - start_tile_k_tile);
 601 |           unit_iter_start += adjustment_tiles;
 602 |           k_tiles_in_my_split -= adjustment_tiles;
 603 |         }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 604-613

```cpp
 604 |         else if (params.ktile_start_alignment_count == 2 && start_tile_k_tile % 2 != 0) {
 605 |           // ktile for each SM start from even number
 606 |           // If start from odd number ktile within the output tile
 607 |           //    now start at the ktile one before my initial ktile start (take one ktile from prev sm)
 608 |           // if end on odd number ktile within the output tile
 609 |           //    now end at ktile that one before my ktile end (give one ktile to next sm)
 610 |           unit_iter_start -= 1;
 611 |           k_tiles_in_my_split += 1;
 612 |         }
 613 |       }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 615-616

```cpp
 615 |       if (work_tile_info.k_tile_count == 0) {
 616 |         // This is a new unit
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 618-622

```cpp
 618 |         if (!is_split_k) {
 619 |           //
 620 |           // Adjust the unit ending position and number of tiles to avoid
 621 |           // computing splits of size less than min_iters_per_sk_unit_
 622 |           //
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 624-625

```cpp
 624 |           // Begin by assuming that no adjustment is needed
 625 |           auto initial_unit_iter_end = unit_iter_start + k_tiles_in_my_split;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 627-628

```cpp
 627 |           int unused, end_tile_k_tile;
 628 |           params.divmod_tiles_per_output_tile_(unused, end_tile_k_tile, initial_unit_iter_end);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 630-650

```cpp
 630 |           if (end_tile_k_tile < Params::min_iters_per_sk_unit_) {
 631 |             // Ending K tile is within the first Params::min_iters_per_sk_unit_ K tiles of some output tile,
 632 |             // which means that this unit will compute a split with fewer than Params::min_iters_per_sk_unit_ K tiles.
 633 |             // Adjust our work to shed these K tiles to a neighboring stream-K unit that will compute more consecutive K tiles.
 634 |             k_tiles_in_my_split -= end_tile_k_tile;
 635 |           }
 636 |           else if (end_tile_k_tile > (params.divmod_tiles_per_output_tile_.divisor - Params::min_iters_per_sk_unit_)) {
 637 |             // Ending K tile is within the final Params::min_iters_per_sk_unit_ K tiles of some output tile,
 638 |             // which means that some other unit will compute a split with fewer than Params::min_iters_per_sk_unit_ K tiles.
 639 |             // Adjust our work to take on these K tiles.
 640 |             k_tiles_in_my_split += (params.divmod_tiles_per_output_tile_.divisor - end_tile_k_tile);
 641 |           }
 642 |           else if (params.ktile_start_alignment_count == 2 && end_tile_k_tile % 2 != 0) {
 643 |             // ktile for each SM start from even number
 644 |             // If start from odd number ktile within the output tile
 645 |             //    now start at the ktile one before my initial ktile start (take one ktile from prev sm)
 646 |             // If end on odd number ktile within the output tile,
 647 |             //    now end at ktile that one before my ktile end (give one ktile to next sm)
 648 |             k_tiles_in_my_split -= 1;
 649 |           }
 650 |         }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 652-653

```cpp
 652 |         work_tile_info.k_tile_remaining = k_tiles_in_my_split;
 653 |       }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 655-665

```cpp
 655 |       uint32_t unit_iter_end = unit_iter_start + work_tile_info.k_tile_remaining - 1;
 656 | 
 657 |       // Find the output tile corresponding to the final k tile covered by this
 658 |       // work unit. Stream-K work units will work backwards in terms of the tiles they
 659 |       // are responsible computing. This is beneficial because the final (partial)
 660 |       // tile computed by a stream-K block is typically the beginning of the output
 661 |       // tile, while the beginning (partial) tile is typically the ending of another
 662 |       // output tile. Since ending portions of an output tile must reduce across
 663 |       // other work units computing portions of that output tile, it is preferable
 664 |       // for them to be computed later, so as to reduce the likelihood of blocking
 665 |       // on other work.
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 667-669

```cpp
 667 |       auto output_tile_id_in_group = params.divmod_tiles_per_output_tile_.divide(unit_iter_end);
 668 |       uint32_t output_tile_iter_start = output_tile_id_in_group * params.divmod_tiles_per_output_tile_.divisor;
 669 |       uint32_t output_tile_iter_end = output_tile_iter_start + params.divmod_tiles_per_output_tile_.divisor;
```
**EN:** This template parameter list exposes element types, layouts, alignments, tile shapes, operators, and policy tags so the next definition can be specialized for many kernel variants.
**CN:** 这一段模板参数列表暴露了元素类型、布局、对齐方式、Tile 形状、运算符和策略标签，使后续定义能够适配多种内核变体。

### Lines 671-673

```cpp
 671 |       // Convert the output tile from the linearized space within each group to the
 672 |       // overall linearized space.
 673 |       output_tile_id = output_tile_id_in_group * params.divmod_sk_groups_.divisor;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 675-678

```cpp
 675 |       // The unit's starting k iteration in the current tile is either the starting
 676 |       // iteration for the tile as a whole, or the starting k iteration for the unit
 677 |       // as a whole (if the latter is greater than the former).
 678 |       uint32_t tile_iter_start = max(output_tile_iter_start, unit_iter_start);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 680-683

```cpp
 680 |       // Similarly, the unit's ending k iteration (exclusive) is either the end of
 681 |       // the current tile it is assigned, or the ending iteration of the unit as a whole
 682 |       // (if the latter is less than the former).
 683 |       uint32_t tile_iter_end = min(output_tile_iter_end, unit_iter_end + 1);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 685-688

```cpp
 685 |       // Set the k offset to be the starting k tile for this output tile
 686 |       work_tile_info.K_idx = static_cast<int32_t>(tile_iter_start - output_tile_iter_start);
 687 |       work_tile_info.k_tile_count = tile_iter_end - tile_iter_start;
 688 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 690-690

```cpp
 690 |     uint64_t work_idx_l, remainder;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 692-694

```cpp
 692 |     if(params.divmod_splits_.divisor > 1) {
 693 |       output_tile_id %= params.units_per_problem_;
 694 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 696-696

```cpp
 696 |     params.divmod_batch_(work_idx_l, remainder, output_tile_id);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 698-698

```cpp
 698 |     uint64_t cta_per_grid_dim = remainder;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 700-703

```cpp
 700 |     auto [work_idx_m, work_idx_n] = Params::get_work_idx_m_and_n(
 701 |                                           cta_per_grid_dim,
 702 |                                           params.divmod_blk_major_
 703 |                                         );
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 705-711

```cpp
 705 |     // Set the M, N, and L block offsets
 706 |     work_tile_info.M_idx = work_idx_m;
 707 |     work_tile_info.N_idx = work_idx_n;
 708 |     work_tile_info.L_idx = work_idx_l;
 709 |   }
 710 | 
 711 | };
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 713-713

```cpp
 713 | } // namespace cutlass::gemm::kernel::detail
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

## Dependencies / 依赖关系
- **Included headers / 引入头文件:** `cutlass/barrier.h`, `cutlass/block_striped.h`, `cutlass/fast_math.h`, `cutlass/gemm/kernel/sm90_tile_scheduler.hpp`, `cutlass/kernel_hardware_info.hpp`, `cute/layout.hpp`, `cute/tensor.hpp`, `cutlass/gemm/kernel/xe_persistent_tile_scheduler_params_streamk.hpp`
- **Subsystems / 子系统:** CuTe tensor utilities / CuTe 张量工具, Architecture tags and intrinsics / 架构标签与内建操作, Epilogue/output pipeline / Epilogue 输出流水, Tile schedulers / Tile 调度器
- **Peer kernel headers / 同级内核头文件:** `cutlass/gemm/kernel/sm90_tile_scheduler.hpp`, `cutlass/gemm/kernel/xe_persistent_tile_scheduler_params_streamk.hpp`
