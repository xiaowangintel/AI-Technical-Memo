# static_tile_scheduler.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `include/cutlass/gemm/kernel/static_tile_scheduler.hpp`
- **Purpose / 用途 (EN):** Defines tile scheduler tags, parameter objects, or architecture-specific scheduler selection logic.
- **Purpose / 用途 (CN):** 定义 Tile 调度标签、参数对象或与架构相关的调度选择逻辑。
- **Line count / 行数:** 522

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

### Lines 33-41

```cpp
  33 | #include "cutlass/fast_math.h"
  34 | #include "cutlass/gemm_coord.hpp"
  35 | #include "cutlass/kernel_hardware_info.hpp"
  36 | #include "cutlass/gemm/kernel/tile_scheduler_params.h"
  37 | #include "cute/layout.hpp"
  38 | #include "cute/tensor.hpp"
  39 | #include "cute/arch/cluster_sm90.hpp"
  40 | #include "cutlass/pipeline/pipeline.hpp"
  41 | namespace cutlass::gemm::kernel::detail {
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 45-48

```cpp
  45 | // Users are not supposed to use this class directly.
  46 | // This is a CRTP base class for the actual tile schedulers.
  47 | template<class Subclass>
  48 | class StaticPersistentTileScheduler {
```
**EN:** This block declares or specializes `Subclass`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `Subclass`，它是该头文件中承载某一层内核策略的核心类。

### Lines 50-52

```cpp
  50 | private:
  51 |   uint64_t current_work_linear_idx_;
  52 |   uint64_t total_grid_size_;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 54-59

```cpp
  54 | public:
  55 |   struct WorkTileInfo {
  56 |     int32_t M_idx = 0;
  57 |     int32_t N_idx = 0;
  58 |     int32_t L_idx = 0;
  59 |     bool is_valid_tile = false;
```
**EN:** This block declares or specializes `WorkTileInfo`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `WorkTileInfo`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 61-65

```cpp
  61 |     CUTLASS_HOST_DEVICE
  62 |     bool
  63 |     is_valid() const {
  64 |       return is_valid_tile;
  65 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 67-71

```cpp
  67 |     CUTLASS_HOST_DEVICE
  68 |     static WorkTileInfo
  69 |     invalid_work_tile() {
  70 |       return {-1, -1, -1, false};
  71 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 73-77

```cpp
  73 |     CUTLASS_HOST_DEVICE
  74 |     bool
  75 |     is_final_split(uint32_t k_tiles_per_output_tile) const {
  76 |       return true;
  77 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 79-84

```cpp
  79 |     CUTLASS_HOST_DEVICE
  80 |     int32_t
  81 |     reduction_subtile_idx() const {
  82 |       return -1;
  83 |     }
  84 |   };
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 86-89

```cpp
  86 |   using Params = PersistentTileSchedulerSm90Params;
  87 |   using RasterOrder = typename Params::RasterOrder;
  88 |   using RasterOrderOptions = typename Params::RasterOrderOptions;
  89 |   static constexpr bool IsDynamicPersistent = false;
```
**EN:** This alias block derives concise type names `Params`, `RasterOrder`, `RasterOrderOptions` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Params`, `RasterOrder`, `RasterOrderOptions` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 91-95

```cpp
  91 | public:
  92 |   struct Arguments {
  93 |     int max_swizzle_size = 1;
  94 |     RasterOrderOptions raster_order = RasterOrderOptions::Heuristic;
  95 |   };
```
**EN:** This `Arguments` structure collects the host-facing runtime inputs: problem shape, pointers, strides, epilogue settings, hardware info, and scheduler data.
**CN:** 这个 `Arguments` 结构体收集面向主机侧的运行时输入，包括问题规模、数据指针、步长、epilogue 配置、硬件信息以及调度器数据。

### Lines 97-107

```cpp
  97 |   template <class ProblemShapeMNKL, class TileShape, class ClusterShape>
  98 |   static Params
  99 |   to_underlying_arguments(
 100 |       ProblemShapeMNKL problem_shape_mnkl,
 101 |       TileShape tile_shape,
 102 |       ClusterShape cluster_shape,
 103 |       [[maybe_unused]] KernelHardwareInfo const& hw_info,
 104 |       Arguments const& arguments,
 105 |       [[maybe_unused]] void* workspace=nullptr,
 106 |       [[maybe_unused]] const uint32_t epilogue_subtile = 1,
 107 |       [[maybe_unused]] uint32_t ktile_start_alignment_count = 1u) {
```
**EN:** This helper converts the public argument bundle into lower-level mainloop, epilogue, and scheduler parameters expected by the actual kernel implementation.
**CN:** 这个辅助函数把对外的参数包转换成底层内核实现真正需要的 mainloop、epilogue 和调度器参数。

### Lines 109-111

```cpp
 109 |     // We only need the tile and cluster shape during scheduler setup, so let FTAD do the magic
 110 |     static_assert(cute::is_static<TileShape>::value);
 111 |     static_assert(cute::is_static<ClusterShape>::value);
```
**EN:** These compile-time checks enforce invariants such as problem-rank assumptions, supported scheduler tags, or required tensor-stride shapes.
**CN:** 这些编译期检查用于保证问题维度假设、可支持的调度器标签以及张量步长形状等不变式成立。

### Lines 113-113

```cpp
 113 |     dim3 problem_blocks = get_tiled_cta_shape_mnl(problem_shape_mnkl, tile_shape, cluster_shape);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 115-122

```cpp
 115 |     Params params;
 116 |     params.initialize(
 117 |       problem_blocks,
 118 |       to_gemm_coord(cluster_shape),
 119 |       hw_info,
 120 |       arguments.max_swizzle_size,
 121 |       arguments.raster_order
 122 |     );
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 124-125

```cpp
 124 |     return params;
 125 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 127-131

```cpp
 127 |   CUTLASS_HOST_DEVICE
 128 |   static bool
 129 |   can_implement(Arguments const& args) {
 130 |     return args.max_swizzle_size >= 0;
 131 |   }
```
**EN:** This function checks whether the requested problem, layouts, and runtime options satisfy the constraints of the specialized kernel.
**CN:** 该函数检查请求的问题规模、布局和运行时选项是否满足当前特化内核的约束条件。

### Lines 133-134

```cpp
 133 |   CUTLASS_HOST_DEVICE
 134 |   StaticPersistentTileScheduler() { }
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 136-145

```cpp
 136 |   CUTLASS_DEVICE explicit StaticPersistentTileScheduler(Params const& params_) : scheduler_params(params_) {
 137 |     // MSVC requires protecting use of CUDA-specific nonstandard syntax,
 138 |     // like blockIdx and gridDim, with __CUDA_ARCH__.
 139 | #if defined(__CUDA_ARCH__)
 140 |     if (params_.raster_order_ == RasterOrder::AlongN) {
 141 |       current_work_linear_idx_ = uint64_t(blockIdx.x) + uint64_t(blockIdx.y) * uint64_t(gridDim.x);
 142 |     }
 143 |     else {
 144 |       current_work_linear_idx_ = uint64_t(blockIdx.x) * uint64_t(gridDim.y) + uint64_t(blockIdx.y);
 145 |     }
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 147-154

```cpp
 147 |     total_grid_size_ = uint64_t(gridDim.x) * uint64_t(gridDim.y) * uint64_t(gridDim.z);
 148 | #elif defined(__SYCL_DEVICE_ONLY__)
 149 |    if (params_.raster_order_ == RasterOrder::AlongN) {
 150 |      current_work_linear_idx_ = uint64_t(BlockIdxX()) + uint64_t(BlockIdxY()) * uint64_t(GridDimX());
 151 |    }
 152 |    else {
 153 |      current_work_linear_idx_ = uint64_t(BlockIdxX()) * uint64_t(GridDimY()) + uint64_t(BlockIdxY());
 154 |    }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 156-160

```cpp
 156 |    total_grid_size_ = uint64_t(GridDimX()) * uint64_t(GridDimY()) * uint64_t(GridDimZ());
 157 | #else
 158 |     CUTLASS_ASSERT(false && "This line should never be reached");
 159 | #endif
 160 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 162-168

```cpp
 162 |   // Returns the initial work tile info that will be computed over
 163 |   template <class ClusterShape>
 164 |   CUTLASS_DEVICE
 165 |   WorkTileInfo
 166 |   initial_work_tile_info(ClusterShape cluster_shape) {
 167 |     return get_current_work();
 168 |   }
```
**EN:** This block declares or specializes `ClusterShape`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `ClusterShape`，它是该头文件中承载某一层内核策略的核心类。

### Lines 170-174

```cpp
 170 |   CUTLASS_DEVICE
 171 |   WorkTileInfo
 172 |   get_current_work() const {
 173 |     return get_current_work_for_linear_idx(current_work_linear_idx_);
 174 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 176-181

```cpp
 176 |   CUTLASS_DEVICE
 177 |   WorkTileInfo
 178 |   get_current_work_for_linear_idx(uint64_t linear_idx) const {
 179 |     if (linear_idx >= scheduler_params.blocks_per_problem_) {
 180 |       return WorkTileInfo::invalid_work_tile();
 181 |     }
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 183-185

```cpp
 183 |     // Map worker's linear index into the CTA tiled problem shape to the corresponding MNL indices
 184 |     uint64_t work_idx_l, remainder;
 185 |     scheduler_params.divmod_batch_(work_idx_l, remainder, linear_idx);
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 187-187

```cpp
 187 |     uint64_t blk_per_grid_dim = scheduler_params.divmod_cluster_shape_minor_.divide(remainder);
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 189-194

```cpp
 189 |     auto [work_idx_m, work_idx_n] = Subclass::get_work_idx_m_and_n(blk_per_grid_dim,
 190 |                                                          scheduler_params.divmod_cluster_shape_major_,
 191 |                                                          scheduler_params.divmod_cluster_shape_minor_,
 192 |                                                          scheduler_params.divmod_cluster_blk_major_,
 193 |                                                          scheduler_params.log_swizzle_size_,
 194 |                                                          scheduler_params.raster_order_);
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 196-197

```cpp
 196 |     return {work_idx_m, work_idx_n, static_cast<int32_t>(work_idx_l), true};
 197 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 199-203

```cpp
 199 |   CUTLASS_DEVICE
 200 |   void
 201 |   advance_to_next_work(uint32_t advance_count = 1) {
 202 |     current_work_linear_idx_ += total_grid_size_ * uint64_t(advance_count);
 203 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 205-213

```cpp
 205 |   CUTLASS_DEVICE
 206 |   bool is_last_tile(WorkTileInfo& work_tile_info, uint32_t advance_count = 1) const {
 207 |     if (continue_current_work(work_tile_info)) {
 208 |       return false;
 209 |     }
 210 |     return not get_current_work_for_linear_idx(
 211 |         current_work_linear_idx_ + (total_grid_size_ * uint64_t(advance_count))
 212 |     ).is_valid();
 213 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 215-226

```cpp
 215 |   // Computes the linear index within a batch given M and N tile offsets within the batch.
 216 |   // This essentially inverts the mapping performed in get_work_idx_m_and_n
 217 |   static CUTLASS_DEVICE
 218 |   uint64_t
 219 |   get_linear_idx_from_m_and_n(
 220 |     int32_t tile_m,
 221 |     int32_t tile_n,
 222 |     FastDivmodU64Pow2 const& divmod_cluster_shape_major,
 223 |     FastDivmodU64Pow2 const& divmod_cluster_shape_minor,
 224 |     FastDivmodU64 const& divmod_cluster_blk_major,
 225 |     int32_t log_swizzle_size,
 226 |     RasterOrder raster_order) {
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 228-240

```cpp
 228 |     uint64_t minor_work_idx, major_work_idx, cluster_minor_offset;
 229 |     if (raster_order == RasterOrder::AlongN) {
 230 |       minor_work_idx = static_cast<uint64_t>(tile_m);
 231 |       major_work_idx = static_cast<uint64_t>(tile_n);
 232 |       uint64_t cluster_m = divmod_cluster_shape_minor.divide(tile_m) * divmod_cluster_shape_minor.divisor;
 233 |       cluster_minor_offset = tile_m - cluster_m;
 234 |     }
 235 |     else {
 236 |       major_work_idx = static_cast<uint64_t>(tile_m);
 237 |       minor_work_idx = static_cast<uint64_t>(tile_n);
 238 |       uint64_t cluster_n = divmod_cluster_shape_minor.divide(tile_n) * divmod_cluster_shape_minor.divisor;
 239 |       cluster_minor_offset = tile_n - cluster_n;
 240 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 242-244

```cpp
 242 |     uint64_t cluster_idx_minor, cluster_idx_major, cluster_major_offset;
 243 |     cluster_idx_minor = divmod_cluster_shape_minor.divide(minor_work_idx - cluster_minor_offset);
 244 |     divmod_cluster_shape_major(cluster_idx_major, cluster_major_offset, major_work_idx);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 246-247

```cpp
 246 |     uint64_t cluster_idx_minor_div_swizzle = cluster_idx_minor >> log_swizzle_size;
 247 |     uint64_t offset = cluster_idx_minor & ((1 << log_swizzle_size) - 1);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 249-249

```cpp
 249 |     uint64_t extra = cluster_idx_minor_div_swizzle * divmod_cluster_blk_major.divisor + cluster_idx_major;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 251-253

```cpp
 251 |     uint64_t cluster_id = (extra << log_swizzle_size) | offset;
 252 |     return (cluster_id * divmod_cluster_shape_major.divisor + cluster_major_offset) * divmod_cluster_shape_minor.divisor + cluster_minor_offset;
 253 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 255-262

```cpp
 255 |   // Given the inputs, computes the total number of output blocks over which this problem will compute. 
 256 |   // Note that this is only the logical size of our grid, not the physical grid we will actually launch.
 257 |   template<class ProblemShapeMNKL, class BlockShape, class ClusterShape>
 258 |   CUTLASS_HOST_DEVICE static
 259 |   dim3
 260 |   get_tiled_cta_shape_mnl(ProblemShapeMNKL problem_shape_mnkl, BlockShape cta_shape, ClusterShape cluster_shape) {
 261 |     auto cta_m = cute::size(cute::ceil_div(cute::shape<0>(problem_shape_mnkl), cute::shape<0>(cta_shape)));
 262 |     auto cta_n = cute::size(cute::ceil_div(cute::shape<1>(problem_shape_mnkl), cute::shape<1>(cta_shape)));
```
**EN:** This block declares or specializes `ProblemShapeMNKL`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `ProblemShapeMNKL`，它是该头文件中承载某一层内核策略的核心类。

### Lines 264-269

```cpp
 264 |     return Params::get_tiled_cta_shape_mnl(
 265 |       to_gemm_coord(problem_shape_mnkl),
 266 |       to_gemm_coord(cluster_shape),
 267 |       cta_m, cta_n
 268 |     );
 269 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 271-278

```cpp
 271 |   // Reloaded interface that receives WorkTileInfo to deduce next work.
 272 |   // Kernel helper function to get next work tile
 273 |   CUTLASS_DEVICE
 274 |   auto
 275 |   fetch_next_work(WorkTileInfo work_tile_info) {
 276 |     if (continue_current_work(work_tile_info)) {
 277 |       return cute::make_tuple(work_tile_info, true);
 278 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 280-282

```cpp
 280 |     advance_to_next_work();
 281 |     return cute::make_tuple(get_current_work(), true);
 282 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 284-295

```cpp
 284 |   // Given the inputs, computes the total number of output blocks over which this problem will compute.
 285 |   // Note that this is only the logical size of our grid, not the physical grid we will actually launch.
 286 |   template<class ProblemShapeMNKL, class TileShape, class AtomThrShape, class ClusterShape>
 287 |   CUTLASS_HOST_DEVICE static
 288 |   dim3
 289 |   get_tiled_cta_shape_mnl(ProblemShapeMNKL problem_shape_mnkl,
 290 |                           TileShape tile_shape_mnk,
 291 |                           AtomThrShape atom_thr_shape_mnk,
 292 |                           ClusterShape cluster_shape_mnk) {
 293 |     auto [tiles_m, tiles_n, tiles_l] = product_each(ceil_div(select<0,1,3>(problem_shape_mnkl), take<0,2>(tile_shape_mnk)));
 294 |     auto cta_m = round_nearest(tiles_m * size<0>(atom_thr_shape_mnk), size<0>(cluster_shape_mnk));
 295 |     auto cta_n = round_nearest(tiles_n * size<1>(atom_thr_shape_mnk), size<1>(cluster_shape_mnk));
```
**EN:** This block declares or specializes `ProblemShapeMNKL`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `ProblemShapeMNKL`，它是该头文件中承载某一层内核策略的核心类。

### Lines 297-302

```cpp
 297 |     return Params::get_tiled_cta_shape_mnl(
 298 |       to_gemm_coord(problem_shape_mnkl),
 299 |       to_gemm_coord(cluster_shape_mnk),
 300 |       cta_m, cta_n
 301 |     );
 302 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 304-313

```cpp
 304 |   // Kernel helper function to get next work tile
 305 |   template <class TileSchedulerPipeline, class TileSchedulerPipelineState>
 306 |   CUTLASS_DEVICE
 307 |   auto
 308 |   fetch_next_work(
 309 |       WorkTileInfo work_tile_info,
 310 |       TileSchedulerPipeline& scheduler_pipeline,
 311 |       TileSchedulerPipelineState scheduler_pipe_consumer_state) {
 312 |     return fetch_next_work(work_tile_info);
 313 |   }
```
**EN:** This block declares or specializes `TileSchedulerPipeline`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `TileSchedulerPipeline`，它是该头文件中承载某一层内核策略的核心类。

### Lines 315-326

```cpp
 315 |   CUTLASS_DEVICE
 316 |   static auto
 317 |   work_tile_to_cta_coord(WorkTileInfo work_tile_info) {
 318 |     // Get every cta coord in three dimensions of the cluster
 319 |     auto [cta_m_in_cluster, cta_n_in_cluster, cta_l_in_cluster] = cute::block_id_in_cluster();
 320 |     return make_coord(
 321 |       work_tile_info.M_idx + static_cast<int32_t>(cta_m_in_cluster),
 322 |       work_tile_info.N_idx + static_cast<int32_t>(cta_n_in_cluster),
 323 |       _,
 324 |       work_tile_info.L_idx + static_cast<int32_t>(cta_l_in_cluster)
 325 |     );
 326 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 328-339

```cpp
 328 |   CUTLASS_DEVICE
 329 |   static auto
 330 |   work_tile_to_cta_coord(WorkTileInfo work_tile_info, dim3 block_id_in_cluster) {
 331 |     // Get every cta coord in three dimensions of the cluster
 332 |     auto [cta_m_in_cluster, cta_n_in_cluster, cta_l_in_cluster] = block_id_in_cluster;
 333 |     return make_coord(
 334 |       work_tile_info.M_idx + static_cast<int32_t>(cta_m_in_cluster),
 335 |       work_tile_info.N_idx + static_cast<int32_t>(cta_n_in_cluster),
 336 |       _,
 337 |       work_tile_info.L_idx + static_cast<int32_t>(cta_l_in_cluster)
 338 |     );
 339 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 341-352

```cpp
 341 |   // Given the inputs, computes the physical grid we should launch.
 342 |   template<class ProblemShapeMNKL, class BlockShape, class ClusterShape>
 343 |   CUTLASS_HOST_DEVICE static
 344 |   dim3
 345 |   get_grid_shape(
 346 |       [[maybe_unused]] Params const& params,
 347 |       ProblemShapeMNKL problem_shape_mnk,
 348 |       BlockShape cta_shape,
 349 |       ClusterShape cluster_shape,
 350 |       KernelHardwareInfo hw_info,
 351 |       Arguments arguments = Arguments{},
 352 |       bool truncate_by_problem_size=true) {
```
**EN:** These launch helpers compute the CUDA/SYCL execution geometry from the problem shape, tile shape, and scheduling policy.
**CN:** 这些启动辅助函数根据问题规模、Tile 形状和调度策略计算 CUDA/SYCL 的执行网格。

### Lines 354-355

```cpp
 354 |     auto problem_shape_mnkl = cute::append<4>(problem_shape_mnk, cute::Int<1>{});
 355 |     dim3 problem_blocks = get_tiled_cta_shape_mnl(problem_shape_mnkl, cta_shape, cluster_shape);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 357-365

```cpp
 357 |     return Params::get_grid_shape(
 358 |       problem_blocks,
 359 |       to_gemm_coord(cluster_shape),
 360 |       hw_info,
 361 |       arguments.max_swizzle_size,
 362 |       arguments.raster_order,
 363 |       /* truncate_by_problem_size = */true
 364 |     );
 365 |   }
```
**EN:** These launch helpers compute the CUDA/SYCL execution geometry from the problem shape, tile shape, and scheduling policy.
**CN:** 这些启动辅助函数根据问题规模、Tile 形状和调度策略计算 CUDA/SYCL 的执行网格。

### Lines 367-376

```cpp
 367 |   // Given the inputs, computes the physical grid we should launch.
 368 |   template<class ProblemShapeMNKL, class TileShape, class AtomThrShape, class ClusterShape>
 369 |   static dim3
 370 |   get_grid_shape(
 371 |       Params const& params,
 372 |       ProblemShapeMNKL problem_shape_mnkl,
 373 |       TileShape tile_shape_mnk,
 374 |       AtomThrShape atom_thr_shape_mnk,
 375 |       ClusterShape cluster_shape_mnk,
 376 |       KernelHardwareInfo hw_info) {
```
**EN:** These launch helpers compute the CUDA/SYCL execution geometry from the problem shape, tile shape, and scheduling policy.
**CN:** 这些启动辅助函数根据问题规模、Tile 形状和调度策略计算 CUDA/SYCL 的执行网格。

### Lines 378-383

```cpp
 378 |     dim3 problem_blocks = get_tiled_cta_shape_mnl(problem_shape_mnkl, tile_shape_mnk, atom_thr_shape_mnk, cluster_shape_mnk);
 379 |     Arguments args{};
 380 |     if constexpr (!std::is_const_v<decltype(args.max_swizzle_size)>) {
 381 |       args.max_swizzle_size = 1 << params.log_swizzle_size_;
 382 |     }
 383 |     args.raster_order = params.raster_order_ == RasterOrder::AlongN ? RasterOrderOptions::AlongN : RasterOrderOptions::AlongM;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 385-393

```cpp
 385 |     return Params::get_grid_shape(
 386 |       problem_blocks,
 387 |       to_gemm_coord(cluster_shape_mnk),
 388 |       hw_info,
 389 |       args.max_swizzle_size,
 390 |       args.raster_order,
 391 |       /* truncate_by_problem_size = */true
 392 |     );
 393 |   }
```
**EN:** These launch helpers compute the CUDA/SYCL execution geometry from the problem shape, tile shape, and scheduling policy.
**CN:** 这些启动辅助函数根据问题规模、Tile 形状和调度策略计算 CUDA/SYCL 的执行网格。

### Lines 395-407

```cpp
 395 |   // Convert CTA-level work tile info to cluster-level tile coord
 396 |   CUTLASS_DEVICE
 397 |   auto
 398 |   work_tile_to_cluster_coord_mnkl(WorkTileInfo work_tile_info) const {
 399 |     // TileScheduler works at CTA-level, kernel works at cluster-level
 400 |     int m_coord = idx2crd(work_tile_info.M_idx / scheduler_params.cluster_shape_m_,
 401 |                           scheduler_params.problem_tiles_m_);
 402 |     int n_coord = idx2crd(work_tile_info.N_idx / scheduler_params.cluster_shape_n_,
 403 |                           scheduler_params.problem_tiles_n_);
 404 |     int l_coord = idx2crd(work_tile_info.L_idx,
 405 |                           scheduler_params.problem_tiles_l_);
 406 |     return make_coord(m_coord, n_coord, _, l_coord);
 407 |   }
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 409-415

```cpp
 409 |   // Returns whether the block assigned this work should compute the epilogue for the corresponding
 410 |   // output tile. For the basic tile scheduler, this is always true.
 411 |   CUTLASS_HOST_DEVICE
 412 |   static bool
 413 |   compute_epilogue(WorkTileInfo const&, Params const&) {
 414 |     return true;
 415 |   }
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 417-421

```cpp
 417 |   CUTLASS_HOST_DEVICE
 418 |   static bool
 419 |   compute_epilogue(WorkTileInfo const&) {
 420 |     return true;
 421 |   }
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 423-428

```cpp
 423 |   // Performs the reduction across splits for a given output tile. Since this scheduler does
 424 |   // not split output tiles, no reduction is needed.
 425 |   template <class FrgTensorC>
 426 |   CUTLASS_DEVICE
 427 |   static void
 428 |   fixup(Params const&, WorkTileInfo const&, FrgTensorC&, uint32_t, uint32_t) {}
```
**EN:** This block declares or specializes `FrgTensorC`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `FrgTensorC`，它是该头文件中承载某一层内核策略的核心类。

### Lines 430-435

```cpp
 430 |   // Performs the reduction across splits for a given output tile. No fixup is required for
 431 |   // work units returned by this scheduler.
 432 |   template <class FrgTensorC>
 433 |   CUTLASS_DEVICE
 434 |   void
 435 |   fixup(WorkTileInfo const&, FrgTensorC&, uint32_t, uint32_t) const { }
```
**EN:** This block declares or specializes `FrgTensorC`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `FrgTensorC`，它是该头文件中承载某一层内核策略的核心类。

### Lines 437-444

```cpp
 437 |   // Returns whether the current WorkTileInfo passed in should continue to be used. Since
 438 |   // this scheduler only schedules work in units of single, full output tiles, the WorkTileInfo
 439 |   // passed in should not be used after having been processed.
 440 |   CUTLASS_DEVICE
 441 |   static bool
 442 |   continue_current_work(WorkTileInfo&) {
 443 |     return false;
 444 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 446-452

```cpp
 446 |   template <class ProblemShapeMNKL, class TileShape, class Shape>
 447 |   CUTLASS_DEVICE
 448 |   auto
 449 |   get_k_tile_iterator(WorkTileInfo const& work_tile_info, ProblemShapeMNKL problem_shape_MNKL, TileShape tile_shape, Shape) {
 450 |     auto k_tiles = cute::ceil_div(cute::get<2>(problem_shape_MNKL), cute::get<2>(tile_shape));
 451 |     return cute::make_coord_iterator(k_tiles);
 452 |   }
```
**EN:** This block declares or specializes `ProblemShapeMNKL`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `ProblemShapeMNKL`，它是该头文件中承载某一层内核策略的核心类。

### Lines 454-461

```cpp
 454 |   template <class ProblemShape, class TileShape>
 455 |   CUTLASS_HOST_DEVICE
 456 |   static int
 457 |   get_work_k_tile_count(WorkTileInfo const& work_tile_info, ProblemShape problem_shape, TileShape tile_shape) {
 458 |     // All work units returned by this scheduler cover the entire K iteration
 459 |     // space of the output tile assigned to the work unit.
 460 |     return cute::size(cute::ceil_div(cute::get<2>(problem_shape), cute::get<2>(tile_shape)));
 461 |   }
```
**EN:** This block declares or specializes `ProblemShape`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `ProblemShape`，它是该头文件中承载某一层内核策略的核心类。

### Lines 463-468

```cpp
 463 |   CUTLASS_HOST_DEVICE
 464 |   static uint32_t
 465 |   get_work_k_tile_start(WorkTileInfo const&) {
 466 |     // All work units returned by this scheduler start from K tile 0
 467 |     return 0u;
 468 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 470-474

```cpp
 470 |   CUTLASS_DEVICE
 471 |   static bool
 472 |   need_separate_reduction(Params const& params) {
 473 |     return false;
 474 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 476-480

```cpp
 476 |   CUTLASS_DEVICE
 477 |   bool
 478 |   is_work_tile_for_reduction(WorkTileInfo const& work_tile_info, Params const& params) {
 479 |     return false;
 480 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 482-491

```cpp
 482 |   template <class FrgTensorC>
 483 |   CUTLASS_DEVICE
 484 |   void
 485 |   separate_reduction(
 486 |     Params const& params,
 487 |     WorkTileInfo const& work_tile_info,
 488 |     FrgTensorC& accumulators,
 489 |     uint32_t num_barriers,
 490 |     uint32_t barrier_idx) {
 491 |   }
```
**EN:** This block declares or specializes `FrgTensorC`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `FrgTensorC`，它是该头文件中承载某一层内核策略的核心类。

### Lines 493-503

```cpp
 493 |   // Shares the accumulator set with peers in the global workspace
 494 |   template <class FrgTensorC>
 495 |   CUTLASS_DEVICE
 496 |   static void
 497 |   share(
 498 |     Params const& params,
 499 |     WorkTileInfo const& work_tile_info,
 500 |     FrgTensorC& accumulators,
 501 |     uint32_t num_barriers,
 502 |     uint32_t barrier_idx) {
 503 |   }
```
**EN:** This block declares or specializes `FrgTensorC`, the main class that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `FrgTensorC`，它是该头文件中承载某一层内核策略的核心类。

### Lines 505-509

```cpp
 505 |   CUTLASS_DEVICE
 506 |   static bool
 507 |   valid_warpgroup_in_work_tile(WorkTileInfo const& work_tile_info) {
 508 |     return true;
 509 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 511-515

```cpp
 511 |   CUTLASS_DEVICE
 512 |   static bool
 513 |   requires_separate_reduction(Params const& params) {
 514 |     return false;
 515 |   }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 517-520

```cpp
 517 | public:
 518 |   // Sink scheduler params as a member
 519 |   Params scheduler_params;
 520 | };
```
**EN:** This block continues the scheduler logic defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的调度逻辑相关逻辑。

### Lines 522-522

```cpp
 522 | } // namespace cutlass::gemm::kernel::detail
```
**EN:** These lines open or close the nested CUTLASS namespaces so the declarations stay scoped to the GEMM kernel subsystem.
**CN:** 这些代码行用于打开或关闭 CUTLASS 的嵌套命名空间，使声明被限制在 GEMM 内核子系统内。

## Key Concepts / 关键概念
- Template metaprogramming / 模板元编程
- Tile scheduling / Tile 调度
- Persistent scheduling / 持久化调度
- SM90 architecture tuning / SM90 架构调优
- Runtime argument packing / 运行时参数打包
- Packed launch parameters / 打包后的启动参数

## Dependencies / 依赖关系
- **Included headers / 引入头文件:** `cutlass/fast_math.h`, `cutlass/gemm_coord.hpp`, `cutlass/kernel_hardware_info.hpp`, `cutlass/gemm/kernel/tile_scheduler_params.h`, `cute/layout.hpp`, `cute/tensor.hpp`, `cute/arch/cluster_sm90.hpp`, `cutlass/pipeline/pipeline.hpp`
- **Subsystems / 子系统:** CuTe tensor utilities / CuTe 张量工具, Epilogue/output pipeline / Epilogue 输出流水, Tile schedulers / Tile 调度器
- **Peer kernel headers / 同级内核头文件:** `cutlass/gemm/kernel/tile_scheduler_params.h`
