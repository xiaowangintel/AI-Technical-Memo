# moe_tile_scheduler.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `examples/12_xe20_moe_gemm_cute_interface/moe_tile_scheduler.hpp`
- **Purpose / 目的:** Defines reusable declarations and helper logic for the repository's mixture-of-experts gemm flow. / 为仓库中的MoE GEMM 流程提供可复用的声明与辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14
````cpp
/***************************************************************************************************
 * Copyright (C) 2025 - 2026 Intel Corporation. All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice,
 *this list of conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice,
 * this list of conditions and the following disclaimer in the documentation
 * and/or other materials provided with the distribution.
 *
````
**EN:** This opening block carries the license banner and file-level description, framing the mixture-of-experts gemm flow example before the executable code begins. It corresponds to block 1 of 43 in the file order.
**CN:** 这一开头代码块给出许可证声明和文件级说明，在真正代码开始前先交代MoE GEMM 流程示例的背景。 它对应本文件顺序中的第 1/43 个代码块。

### Lines 15-28
````cpp
 * 3. Neither the name of the copyright holder nor the names of its
 * contributors may be used to endorse or promote products derived from
 * this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 *ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 *LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 *CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 *SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 *INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 *CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 *ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `Neither`, `the`, `name`, `of` showing the main symbols being prepared or consumed here. It corresponds to block 2 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `Neither`、`the`、`name`、`of` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 2/43 个代码块。

### Lines 29-32
````cpp
 *POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/
#pragma once
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `POSSIBILITY`, `OF`, `SUCH`, `DAMAGE` showing the main symbols being prepared or consumed here. It corresponds to block 3 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `POSSIBILITY`、`OF`、`SUCH`、`DAMAGE` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 3/43 个代码块。

### Lines 34-39
````cpp
#include "cute/layout.hpp"
#include "cute/tensor.hpp"
#include "cutlass/fast_math.h"
#include "cutlass/gemm/kernel/tile_scheduler_params.h"
#include "cutlass/gemm_coord.hpp"
#include "cutlass/kernel_hardware_info.hpp"
````
**EN:** This block pulls in dependencies required by the file, especially `cute`, so the later mixture-of-experts gemm flow code can use the needed APIs and data structures. It corresponds to block 4 of 43 in the file order.
**CN:** 这一段引入本文件所需的依赖，尤其是 `cute`，使后续MoE GEMM 流程代码可以使用相应 API 和数据结构。 它对应本文件顺序中的第 4/43 个代码块。

### Lines 41-54
````cpp
namespace MoE {
using namespace cutlass::gemm::kernel::detail;
using namespace cutlass;
using namespace cutlass::gemm;
using namespace cute;
///////////////////////////////////////////////////////////////////////////////
// Adapted from xe_tile_scheduler_group.hpp
// Persistent Thread Block (TB) scheduler for MoE GEMM
template <class GroupProblemShape>
class PersistentTileSchedulerXeMoE
    : public PersistentTileSchedulerXeGroup<GroupProblemShape> {
  //
  // Data members
  //
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `Shape`, `cute` showing the main symbols being prepared or consumed here. It corresponds to block 5 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `Shape`、`cute` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 5/43 个代码块。

### Lines 56-62
````cpp
private:
  uint64_t current_work_linear_idx_ = 0;
  uint64_t total_grid_size_ = 0;
  int32_t *num_rows_per_expert_ = nullptr;
  int32_t K_ = 0;
  int32_t N_ = 0;
  int32_t num_experts_ = 0;
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `uint64_t`, `current_work_linear_idx_`, `total_grid_size_`, `int32_t` showing the main symbols being prepared or consumed here. It corresponds to block 6 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `uint64_t`、`current_work_linear_idx_`、`total_grid_size_`、`int32_t` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 6/43 个代码块。

### Lines 64-69
````cpp
  // Tracking current group, its starting linear idx and total tiles
  struct GroupInfo {
    int group_idx = 0;
    uint64_t start_linear_idx = 0;
    uint64_t total_tiles = 0;
  } current_group_info_;
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `Tracking`, `current`, `group`, `its` showing the main symbols being prepared or consumed here. It corresponds to block 7 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `Tracking`、`current`、`group`、`its` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 7/43 个代码块。

### Lines 71-76
````cpp
public:
  struct WorkTileInfo {
    int32_t M_idx = 0;
    int32_t N_idx = 0;
    int32_t L_idx = 0;
    bool is_valid_tile = false;
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `WorkTileInfo`, `int32_t`, `M_idx`, `N_idx` showing the main symbols being prepared or consumed here. It corresponds to block 8 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `WorkTileInfo`、`int32_t`、`M_idx`、`N_idx` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 8/43 个代码块。

### Lines 78-79
````cpp
    CUTLASS_HOST_DEVICE
    bool is_valid() const { return is_valid_tile; }
````
**EN:** This block finalizes a local computation or status path. The use of `CUTLASS_HOST_DEVICE`, `bool`, `is_valid`, `is_valid_tile` helps conclude the current stage cleanly before the next block. It corresponds to block 9 of 43 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `CUTLASS_HOST_DEVICE`、`bool`、`is_valid`、`is_valid_tile`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 9/43 个代码块。

### Lines 81-83
````cpp
    CUTLASS_HOST_DEVICE
    static WorkTileInfo invalid_work_tile() { return {-1, -1, -1, false}; }
  };
````
**EN:** This block finalizes a local computation or status path. The use of `CUTLASS_HOST_DEVICE`, `static`, `WorkTileInfo`, `invalid_work_tile` helps conclude the current stage cleanly before the next block. It corresponds to block 10 of 43 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `CUTLASS_HOST_DEVICE`、`static`、`WorkTileInfo`、`invalid_work_tile`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 10/43 个代码块。

### Lines 85-89
````cpp
  using ProblemShape = typename GroupProblemShape::UnderlyingProblemShape;
  using Params = PersistentTileSchedulerSm90GroupParams<GroupProblemShape>;
  using RasterOrder = typename Params::RasterOrder;
  using RasterOrderOptions = typename Params::RasterOrderOptions;
  using BaseClass = PersistentTileSchedulerXeGroup<GroupProblemShape>;
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `Shape` make the later mixture-of-experts gemm flow code easier to assemble and read. It corresponds to block 11 of 43 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `Shape` 这样的符号让后续MoE GEMM 流程代码更容易组装和阅读。 它对应本文件顺序中的第 11/43 个代码块。

### Lines 91-91
````cpp
  Params scheduler_params;
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `Params`, `scheduler_params` showing the main symbols being prepared or consumed here. It corresponds to block 12 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `Params`、`scheduler_params` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 12/43 个代码块。

### Lines 93-95
````cpp
  //
  // Methods
  //
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `Methods` showing the main symbols being prepared or consumed here. It corresponds to block 13 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `Methods` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 13/43 个代码块。

### Lines 97-106
````cpp
  // Given the inputs, computes the total number of output blocks this problem
  // will compute over Note that this is only the logical size of our grid, not
  // the physical grid we will actually launch.
  template <class ClusterShape>
  CUTLASS_HOST_DEVICE static dim3
  get_tiled_cta_shape_mnl(const KernelHardwareInfo &hw_info,
                          ClusterShape cluster_shape) {
    uint32_t total_ctas = 0;
    uint32_t cta_in_N_dim =
        1; // We linearize the blocks across all the problems here
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `ClusterShape`, `Shape` showing the main symbols being prepared or consumed here. It corresponds to block 14 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `ClusterShape`、`Shape` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 14/43 个代码块。

### Lines 108-108
````cpp
    total_ctas = hw_info.sm_count;
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `total_ctas`, `hw_info`, `sm_count` showing the main symbols being prepared or consumed here. It corresponds to block 15 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `total_ctas`、`hw_info`、`sm_count` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 15/43 个代码块。

### Lines 110-112
````cpp
    return Params::get_tiled_cta_shape_mnl(to_gemm_coord(cluster_shape),
                                           total_ctas, cta_in_N_dim);
  }
````
**EN:** This block finalizes a local computation or status path. The use of `Params::get_tiled_cta_shape_mnl`, `to_gemm_coord`, `cluster_shape`, `total_ctas` helps conclude the current stage cleanly before the next block. It corresponds to block 16 of 43 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `Params::get_tiled_cta_shape_mnl`、`to_gemm_coord`、`cluster_shape`、`total_ctas`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 16/43 个代码块。

### Lines 114-125
````cpp
  template <class TileShape, class ClusterShape>
  static Params to_underlying_arguments(
      GroupProblemShape problem_shapes, TileShape tile_shape,
      ClusterShape cluster_shape, KernelHardwareInfo const &hw_info,
      typename BaseClass::Arguments const &arguments,
      [[maybe_unused]] void *workspace = nullptr,
      [[maybe_unused]] const uint32_t epilogue_subtile = 1,
      [[maybe_unused]] uint32_t ktile_start_alignment_count = 1u) {
    return BaseClass::to_underlying_arguments(problem_shapes, tile_shape,
                                              cluster_shape, hw_info, arguments,
                                              workspace);
  }
````
**EN:** This block declares a type-level building block for the file, with `epilogue`, `TileShape`, `ClusterShape`, `Shape` indicating the configuration, traits, or storage policy used later. It corresponds to block 17 of 43 in the file order.
**CN:** 这一段声明了文件中的类型级构件，其中 `epilogue`、`TileShape`、`ClusterShape`、`Shape` 表明后续会使用的配置、traits 或存储策略。 它对应本文件顺序中的第 17/43 个代码块。

### Lines 127-134
````cpp
  // Given the inputs, computes the physical grid we should launch.
  template <class TileShape, class ClusterShape>
  CUTLASS_HOST_DEVICE static dim3
  get_grid_shape([[maybe_unused]] Params const &params,
                 GroupProblemShape problem_shapes, TileShape tile_shape,
                 ClusterShape cluster_shape, KernelHardwareInfo hw_info,
                 typename BaseClass::Arguments arguments,
                 bool truncate_by_problem_size = true) {
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `TileShape`, `ClusterShape`, `Shape` showing the main symbols being prepared or consumed here. It corresponds to block 18 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `TileShape`、`ClusterShape`、`Shape` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 18/43 个代码块。

### Lines 136-139
````cpp
    return BaseClass::get_grid_shape(params, problem_shapes, tile_shape,
                                     cluster_shape, hw_info, arguments,
                                     truncate_by_problem_size);
  }
````
**EN:** This block finalizes a local computation or status path. The use of `BaseClass::get_grid_shape`, `params`, `problem_shapes`, `tile_shape` helps conclude the current stage cleanly before the next block. It corresponds to block 19 of 43 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `BaseClass::get_grid_shape`、`params`、`problem_shapes`、`tile_shape`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 19/43 个代码块。

### Lines 141-154
````cpp
  CUTLASS_DEVICE explicit PersistentTileSchedulerXeMoE(
      Params const &params_, int32_t *num_rows_per_expert, int32_t N, int32_t K,
      int32_t num_experts)
      : scheduler_params(params_) {
    num_rows_per_expert_ = num_rows_per_expert;
    N_ = N;
    K_ = K;
    num_experts_ = num_experts;
    if (scheduler_params.raster_order_ == RasterOrder::AlongN) {
      current_work_linear_idx_ =
          uint64_t(BlockIdxX()) + uint64_t(BlockIdxY()) * uint64_t(GridDimX());
    } else {
      current_work_linear_idx_ =
          uint64_t(BlockIdxX()) * uint64_t(GridDimY()) + uint64_t(BlockIdxY());
````
**EN:** This block applies conditional control flow. It uses `CUTLASS_DEVICE`, `explicit`, `PersistentTileSchedulerXeMoE`, `Params` to select a path, validate assumptions, or handle special cases in the mixture-of-experts gemm flow implementation. It corresponds to block 20 of 43 in the file order.
**CN:** 这一段实现条件控制流。它借助 `CUTLASS_DEVICE`、`explicit`、`PersistentTileSchedulerXeMoE`、`Params` 在MoE GEMM 流程实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 20/43 个代码块。

### Lines 155-158
````cpp
    }
    total_grid_size_ =
        uint64_t(GridDimX()) * uint64_t(GridDimY()) * uint64_t(GridDimZ());
  }
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `total_grid_size_`, `uint64_t`, `GridDimX`, `GridDimY` showing the main symbols being prepared or consumed here. It corresponds to block 21 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `total_grid_size_`、`uint64_t`、`GridDimX`、`GridDimY` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 21/43 个代码块。

### Lines 160-163
````cpp
  CUTLASS_DEVICE
  WorkTileInfo get_current_work() {
    return get_current_work_for_linear_idx(current_work_linear_idx_);
  }
````
**EN:** This block finalizes a local computation or status path. The use of `CUTLASS_DEVICE`, `WorkTileInfo`, `get_current_work`, `get_current_work_for_linear_idx` helps conclude the current stage cleanly before the next block. It corresponds to block 22 of 43 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `CUTLASS_DEVICE`、`WorkTileInfo`、`get_current_work`、`get_current_work_for_linear_idx`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 22/43 个代码块。

### Lines 165-175
````cpp
  CUTLASS_DEVICE
  WorkTileInfo get_current_work_for_linear_idx(uint64_t linear_idx) {
    return get_work_idx_m_and_n(
        linear_idx, current_group_info_, scheduler_params.problem_shapes_,
        scheduler_params.cta_shape_, scheduler_params.cluster_shape_,
        scheduler_params.divmod_cluster_shape_major_,
        scheduler_params.divmod_cluster_shape_minor_,
        scheduler_params.divmod_cta_shape_m_,
        scheduler_params.divmod_cta_shape_n_,
        scheduler_params.log_swizzle_size_, scheduler_params.raster_order_);
  }
````
**EN:** This block finalizes a local computation or status path. The use of `CUTLASS_DEVICE`, `WorkTileInfo`, `get_current_work_for_linear_idx`, `uint64_t` helps conclude the current stage cleanly before the next block. It corresponds to block 23 of 43 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `CUTLASS_DEVICE`、`WorkTileInfo`、`get_current_work_for_linear_idx`、`uint64_t`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 23/43 个代码块。

### Lines 177-180
````cpp
  CUTLASS_DEVICE
  void advance_to_next_work(uint32_t advance_count = 1) {
    current_work_linear_idx_ += total_grid_size_ * uint64_t(advance_count);
  }
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `CUTLASS_DEVICE`, `advance_to_next_work`, `uint32_t`, `advance_count` showing the main symbols being prepared or consumed here. It corresponds to block 24 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `CUTLASS_DEVICE`、`advance_to_next_work`、`uint32_t`、`advance_count` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 24/43 个代码块。

### Lines 182-192
````cpp
  // get work_idx_m, work_idx_n from linear_idx while applying swizzle
  CUTLASS_DEVICE
  WorkTileInfo
  get_work_idx_m_and_n(uint64_t linear_idx, struct GroupInfo &group_info,
                       GroupProblemShape &problem_shapes, GemmCoord cta_shape,
                       cutlass::gemm::GemmCoord cluster_shape,
                       FastDivmodU64Pow2 const &divmod_cluster_shape_major,
                       FastDivmodU64Pow2 const &divmod_cluster_shape_minor,
                       FastDivmodU64 const &divmod_cta_shape_m,
                       FastDivmodU64 const &divmod_cta_shape_n,
                       int32_t log_swizzle_size, RasterOrder raster_order) {
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `Shape` showing the main symbols being prepared or consumed here. It corresponds to block 25 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `Shape` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 25/43 个代码块。

### Lines 194-204
````cpp
    bool valid_tile = true;
    uint64_t ctas_along_m, ctas_along_n;
    int total_problem_groups = num_experts_;
    ctas_along_m = divmod_cta_shape_m.divide(
        cute::shape<0>(
            ProblemShape(num_rows_per_expert_[group_info.group_idx], N_, K_)) +
        divmod_cta_shape_m.divisor - 1);
    ctas_along_n = divmod_cta_shape_n.divide(
        cute::shape<1>(
            ProblemShape(num_rows_per_expert_[group_info.group_idx], N_, K_)) +
        divmod_cta_shape_n.divisor - 1);
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `Shape`, `cute` showing the main symbols being prepared or consumed here. It corresponds to block 26 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `Shape`、`cute` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 26/43 个代码块。

### Lines 206-210
````cpp
    auto problem_blocks_m =
        round_up(ctas_along_m, (1 << log_swizzle_size) * cluster_shape.m());
    auto problem_blocks_n =
        round_up(ctas_along_n, (1 << log_swizzle_size) * cluster_shape.n());
    group_info.total_tiles = problem_blocks_m * problem_blocks_n;
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `problem_blocks_m`, `round_up`, `ctas_along_m`, `log_swizzle_size` showing the main symbols being prepared or consumed here. It corresponds to block 27 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `problem_blocks_m`、`round_up`、`ctas_along_m`、`log_swizzle_size` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 27/43 个代码块。

### Lines 212-213
````cpp
    while (group_info.start_linear_idx + group_info.total_tiles <= linear_idx) {
      group_info.group_idx++;
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `group_info`, `start_linear_idx`, `total_tiles`, `linear_idx` showing the main symbols being prepared or consumed here. It corresponds to block 28 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `group_info`、`start_linear_idx`、`total_tiles`、`linear_idx` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 28/43 个代码块。

### Lines 215-216
````cpp
      if (group_info.group_idx >= total_problem_groups)
        return WorkTileInfo::invalid_work_tile();
````
**EN:** This block applies conditional control flow. It uses `group_info`, `group_idx`, `total_problem_groups`, `WorkTileInfo::invalid_work_tile` to select a path, validate assumptions, or handle special cases in the mixture-of-experts gemm flow implementation. It corresponds to block 29 of 43 in the file order.
**CN:** 这一段实现条件控制流。它借助 `group_info`、`group_idx`、`total_problem_groups`、`WorkTileInfo::invalid_work_tile` 在MoE GEMM 流程实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 29/43 个代码块。

### Lines 218-226
````cpp
      group_info.start_linear_idx += group_info.total_tiles;
      ctas_along_m = divmod_cta_shape_m.divide(
          cute::shape<0>(ProblemShape(
              num_rows_per_expert_[group_info.group_idx], N_, K_)) +
          divmod_cta_shape_m.divisor - 1);
      ctas_along_n = divmod_cta_shape_n.divide(
          cute::shape<1>(ProblemShape(
              num_rows_per_expert_[group_info.group_idx], N_, K_)) +
          divmod_cta_shape_n.divisor - 1);
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `Shape`, `cute` showing the main symbols being prepared or consumed here. It corresponds to block 30 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `Shape`、`cute` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 30/43 个代码块。

### Lines 228-233
````cpp
      problem_blocks_m =
          round_up(ctas_along_m, (1 << log_swizzle_size) * cluster_shape.m());
      problem_blocks_n =
          round_up(ctas_along_n, (1 << log_swizzle_size) * cluster_shape.n());
      group_info.total_tiles = problem_blocks_m * problem_blocks_n;
    }
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `problem_blocks_m`, `round_up`, `ctas_along_m`, `log_swizzle_size` showing the main symbols being prepared or consumed here. It corresponds to block 31 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `problem_blocks_m`、`round_up`、`ctas_along_m`、`log_swizzle_size` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 31/43 个代码块。

### Lines 235-239
````cpp
    uint64_t cluster_id, cluster_major_offset = 0, cluster_minor_offset = 0;
    uint64_t blk_per_grid_dim = divmod_cluster_shape_minor.divide(
        linear_idx - group_info.start_linear_idx);
    divmod_cluster_shape_major(cluster_id, cluster_major_offset,
                               blk_per_grid_dim);
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `uint64_t`, `cluster_id`, `cluster_major_offset`, `cluster_minor_offset` showing the main symbols being prepared or consumed here. It corresponds to block 32 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `uint64_t`、`cluster_id`、`cluster_major_offset`、`cluster_minor_offset` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 32/43 个代码块。

### Lines 241-251
````cpp
    // With static schedulers, we launch grid such that all cluster are linear
    // (1-D) order, i.e., there can only be one cluster in the minor dimension.
    // get_grid_shape() in scheduler params put cluster_shape.m/n() as the minor
    // dimension based on raster order AlongN/M resp. Therefore, the offset of a
    // CTA (inside a cluster) in the minor dimension can be directly be inferred
    // by the blockIdx along the minor dimension.
    if (raster_order == RasterOrder::AlongN) {
      cluster_minor_offset = BlockIdxX();
    } else {
      cluster_minor_offset = BlockIdxY();
    }
````
**EN:** This block applies conditional control flow. It uses `With`, `static`, `schedulers`, `we` to select a path, validate assumptions, or handle special cases in the mixture-of-experts gemm flow implementation. It corresponds to block 33 of 43 in the file order.
**CN:** 这一段实现条件控制流。它借助 `With`、`static`、`schedulers`、`we` 在MoE GEMM 流程实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 33/43 个代码块。

### Lines 253-253
````cpp
    uint64_t cluster_idx_minor, cluster_idx_major;
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `uint64_t`, `cluster_idx_minor`, `cluster_idx_major` showing the main symbols being prepared or consumed here. It corresponds to block 34 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `uint64_t`、`cluster_idx_minor`、`cluster_idx_major` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 34/43 个代码块。

### Lines 255-255
````cpp
    uint64_t cluster_idx_minor_div_swizzle, extra, offset;
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `uint64_t`, `cluster_idx_minor_div_swizzle`, `extra`, `offset` showing the main symbols being prepared or consumed here. It corresponds to block 35 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `uint64_t`、`cluster_idx_minor_div_swizzle`、`extra`、`offset` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 35/43 个代码块。

### Lines 257-258
````cpp
    offset = cluster_id & ((1 << log_swizzle_size) - 1);
    extra = cluster_id >> log_swizzle_size;
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `offset`, `cluster_id`, `log_swizzle_size`, `extra` showing the main symbols being prepared or consumed here. It corresponds to block 36 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `offset`、`cluster_id`、`log_swizzle_size`、`extra` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 36/43 个代码块。

### Lines 260-269
````cpp
    uint64_t curr_group_cluster_blk_major;
    if (raster_order == RasterOrder::AlongN) {
      curr_group_cluster_blk_major =
          divmod_cluster_shape_major.divide(problem_blocks_n);
    } else {
      curr_group_cluster_blk_major =
          divmod_cluster_shape_major.divide(problem_blocks_m);
    }
    cluster_idx_minor_div_swizzle = extra / curr_group_cluster_blk_major;
    cluster_idx_major = extra % curr_group_cluster_blk_major;
````
**EN:** This block applies conditional control flow. It uses `uint64_t`, `curr_group_cluster_blk_major`, `raster_order`, `RasterOrder::AlongN` to select a path, validate assumptions, or handle special cases in the mixture-of-experts gemm flow implementation. It corresponds to block 37 of 43 in the file order.
**CN:** 这一段实现条件控制流。它借助 `uint64_t`、`curr_group_cluster_blk_major`、`raster_order`、`RasterOrder::AlongN` 在MoE GEMM 流程实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 37/43 个代码块。

### Lines 271-272
````cpp
    cluster_idx_minor =
        cluster_idx_minor_div_swizzle * (1 << log_swizzle_size) + offset;
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `cluster_idx_minor`, `cluster_idx_minor_div_swizzle`, `log_swizzle_size`, `offset` showing the main symbols being prepared or consumed here. It corresponds to block 38 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `cluster_idx_minor`、`cluster_idx_minor_div_swizzle`、`log_swizzle_size`、`offset` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 38/43 个代码块。

### Lines 274-279
````cpp
    auto minor_work_idx = static_cast<int32_t>(
        cluster_idx_minor * divmod_cluster_shape_minor.divisor +
        cluster_minor_offset);
    auto major_work_idx = static_cast<int32_t>(
        cluster_idx_major * divmod_cluster_shape_major.divisor +
        cluster_major_offset);
````
**EN:** This block introduces executable logic through a function or method. Here, `minor_work_idx`, `static_cast<int32_t`, `cluster_idx_minor`, `divmod_cluster_shape_minor` drive a concrete step in the file's mixture-of-experts gemm flow flow. It corresponds to block 39 of 43 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `minor_work_idx`、`static_cast<int32_t`、`cluster_idx_minor`、`divmod_cluster_shape_minor` 推动了本文件MoE GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 39/43 个代码块。

### Lines 281-286
````cpp
    if (raster_order == RasterOrder::AlongN) {
      return {minor_work_idx, major_work_idx, group_info.group_idx, valid_tile};
    } else {
      return {major_work_idx, minor_work_idx, group_info.group_idx, valid_tile};
    }
  }
````
**EN:** This block applies conditional control flow. It uses `raster_order`, `RasterOrder::AlongN`, `minor_work_idx`, `major_work_idx` to select a path, validate assumptions, or handle special cases in the mixture-of-experts gemm flow implementation. It corresponds to block 40 of 43 in the file order.
**CN:** 这一段实现条件控制流。它借助 `raster_order`、`RasterOrder::AlongN`、`minor_work_idx`、`major_work_idx` 在MoE GEMM 流程实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 40/43 个代码块。

### Lines 288-293
````cpp
  // Kernel helper function to get next work tile
  CUTLASS_DEVICE
  auto fetch_next_work(WorkTileInfo work_tile_info) {
    advance_to_next_work();
    return get_current_work();
  }
````
**EN:** This block finalizes a local computation or status path. The use of `Kernel`, `helper`, `function`, `to` helps conclude the current stage cleanly before the next block. It corresponds to block 41 of 43 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `Kernel`、`helper`、`function`、`to`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 41/43 个代码块。

### Lines 295-300
````cpp
  // Returns the initial work tile info that will be computed over
  template <class ClusterShape>
  CUTLASS_DEVICE WorkTileInfo initial_work_tile_info(ClusterShape) {
    return get_current_work();
  }
};
````
**EN:** This block finalizes a local computation or status path. The use of `ClusterShape`, `Shape` helps conclude the current stage cleanly before the next block. It corresponds to block 42 of 43 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `ClusterShape`、`Shape`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 42/43 个代码块。

### Lines 302-302
````cpp
} // namespace MoE
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `namespace`, `MoE` showing the main symbols being prepared or consumed here. It corresponds to block 43 of 43 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `namespace`、`MoE` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 43/43 个代码块。

## Key Concepts / 关键概念
- **EN:** Template-based GEMM composition with CUTLASS collectives, adapters, or visitors.
  **CN:** 使用 CUTLASS collective、adapter 或 visitor 进行模板化 GEMM 组合。
- **EN:** CUTE layout algebra is used to describe shapes, coordinates, and tensor views.
  **CN:** 使用 CUTE 布局代数来描述形状、坐标和张量视图。
- **EN:** Header-only abstractions centralize reusable declarations and launch helpers.
  **CN:** 头文件抽象把可复用声明与启动辅助逻辑集中起来。
- **EN:** The file mixes compile-time configuration with runtime problem sizes or arguments.
  **CN:** 该文件把编译期配置与运行期问题规模或参数结合起来。
- **EN:** Validation, benchmarking, or reporting code is interleaved with kernel setup.
  **CN:** 校验、基准测试或结果报告代码与内核配置过程交织在一起。

## Dependencies / 依赖关系
- **Direct dependencies / 直接依赖:** `cute/layout.hpp`, `cute/tensor.hpp`, `cutlass/fast_math.h`, `cutlass/gemm/kernel/tile_scheduler_params.h`, `cutlass/gemm_coord.hpp`, `cutlass/kernel_hardware_info.hpp`
- **Runtime expectations / 运行时依赖:** CUTLASS/CUTE templates / CUTLASS/CUTE 模板
- **Build/context note / 构建说明:** The file relies on surrounding repository infrastructure, compile flags, and example utilities. / 该文件依赖仓库周边基础设施、编译选项以及示例辅助代码。
