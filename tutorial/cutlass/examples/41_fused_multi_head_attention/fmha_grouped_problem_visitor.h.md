# fmha_grouped_problem_visitor.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/41_fused_multi_head_attention/fmha_grouped_problem_visitor.h`
**Purpose / 用途**: FMHA-specific specialization of CUTLASS grouped-problem scheduling. It adapts the generic `GroupedProblemVisitor` to the fact that FMHA tiles only partition along the query (`M`) dimension and track two problem-size streams. / 该文件是 CUTLASS grouped 问题调度器的 FMHA 特化版本，它把通用 `GroupedProblemVisitor` 适配到 FMHA 的特点：tile 只沿 query（`M`）维切分，并且需要同时维护两组 problem size。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行
```cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice, this
 * list of conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice,
 * this list of conditions and the following disclaimer in the documentation
 * and/or other materials provided with the distribution.
 *
 * 3. Neither the name of the copyright holder nor the names of its
 * contributors may be used to endorse or promote products derived from
 * this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/

/*! \file
    \brief Scheduler for grouped FMHA
*/

#pragma once

#include "cutlass/cutlass.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/matrix_coord.h"
#include "cutlass/gemm/kernel/grouped_problem_visitor.h"

/////////////////////////////////////////////////////////////////////////////////////////////////

namespace cutlass {
namespace gemm {
namespace kernel {

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: License, includes, namespace setup, and file-level motivation.
**CN**: 许可证、依赖头文件、命名空间准备以及文件动机说明。

### Lines 51-72 / 第 51-72 行
```cpp
namespace detail {
// Helper for correctly representing problem sizes in grouped kernels
template <typename ThreadblockShape>
struct FMHAGroupedProblemSizeHelper {

  CUTLASS_HOST_DEVICE
  static cutlass::gemm::GemmCoord grid_shape(const cutlass::gemm::GemmCoord& problem) {
    // FMHA only partitions tiles across the M dimension.
    return cutlass::gemm::GemmCoord(
      ((problem.m() - 1 + ThreadblockShape::kM) / ThreadblockShape::kM), 1, 1);
  }

  CUTLASS_HOST_DEVICE
  static void possibly_transpose_problem(cutlass::gemm::GemmCoord& problem) {}

  CUTLASS_HOST_DEVICE
  static int32_t tile_count(const cutlass::gemm::GemmCoord& grid) {
    return grid.m() * grid.n();
  }
};

} // namespace detail
```
**EN**: `FMHAGroupedProblemSizeHelper` defining FMHA tile geometry and tile counting.
**CN**: `FMHAGroupedProblemSizeHelper`，定义 FMHA 的 tile 几何与 tile 计数方式。

### Lines 73-138 / 第 73-138 行
```cpp

/// Visitor class to abstract away the algorithm for iterating over tiles
template <typename ThreadblockShape,
          GroupScheduleMode GroupScheduleMode_,
          int PrefetchTileCount,
          int ThreadCount,
          bool Transposed = false>
struct FMHAGroupedProblemVisitor : public GroupedProblemVisitor<
                                            detail::FMHAGroupedProblemSizeHelper<ThreadblockShape>,
                                            ThreadblockShape,
                                            GroupScheduleMode_,
                                            PrefetchTileCount,
                                            ThreadCount> {

  using ProblemSizeHelper = detail::FMHAGroupedProblemSizeHelper<ThreadblockShape>;
  using Base = GroupedProblemVisitor<ProblemSizeHelper, ThreadblockShape, GroupScheduleMode_, PrefetchTileCount, ThreadCount>;
  using BaseParams = typename Base::Params;
  using SharedStorage = typename Base::SharedStorage;

  cutlass::gemm::GemmCoord const *problem_sizes0;
  cutlass::gemm::GemmCoord const *problem_sizes1;

  struct Params {
    cutlass::gemm::GemmCoord const *problem_sizes0;
    cutlass::gemm::GemmCoord const *problem_sizes1;
    int32_t                         problem_count;
    void const                     *workspace;
    int32_t                         tile_count;

    //
    // Methods
    //

    /// Ctor
    CUTLASS_HOST_DEVICE
    Params(): problem_sizes0(nullptr), problem_sizes1(nullptr),
              problem_count(0), workspace(nullptr), tile_count(0) { }

    /// Ctor
    CUTLASS_HOST_DEVICE
    Params(
      cutlass::gemm::GemmCoord const *problem_sizes0,
      cutlass::gemm::GemmCoord const *problem_sizes1,
      int32_t                         problem_count,
      void const                     *workspace = nullptr,
      int32_t                         tile_count = 0
    ):
      problem_sizes0(problem_sizes0),
      problem_sizes1(problem_sizes1),
      problem_count(problem_count),
      workspace(workspace),
      tile_count(tile_count)
    {}

    /// Convert the FMHA-specific parameters to those used by the base class
    CUTLASS_HOST_DEVICE
    BaseParams to_base() const {
        return BaseParams(// Set problem_sizes as problem_sizes1 because these determine
                          // shape of the final output of FMHA
                          problem_sizes1,
                          problem_count,
                          workspace,
                          tile_count);
    }

  };
```
**EN**: `FMHAGroupedProblemVisitor` declaration and nested `Params` structure with `to_base()` conversion.
**CN**: `FMHAGroupedProblemVisitor` 声明，以及带 `to_base()` 转换逻辑的内部 `Params` 结构。

### Lines 139-170 / 第 139-170 行
```cpp

  //
  // Methods
  //
  CUTLASS_DEVICE
  FMHAGroupedProblemVisitor(
    Params const &params_,
    SharedStorage &shared_storage_, 
    int32_t block_idx
  ): Base (
        params_.to_base(),
        shared_storage_, block_idx),
     problem_sizes0(params_.problem_sizes0),
     problem_sizes1(params_.problem_sizes1)
  {}

  /// Returns the problem size 0 for the current problem
  CUTLASS_HOST_DEVICE
  cutlass::gemm::GemmCoord problem_size0() const {
    GemmCoord problem = problem_sizes0[this->problem_idx];
    ProblemSizeHelper::possibly_transpose_problem(problem);
    return problem;
  }

  /// Returns the problem size 1 for the current problem
  CUTLASS_HOST_DEVICE
  cutlass::gemm::GemmCoord problem_size1() const {
    GemmCoord problem = problem_sizes1[this->problem_idx];
    ProblemSizeHelper::possibly_transpose_problem(problem);
    return problem;
  }
};
```
**EN**: Constructor plus `problem_size0()` and `problem_size1()` accessors.
**CN**: 构造函数，以及 `problem_size0()` / `problem_size1()` 访问器。

### Lines 171-178 / 第 171-178 行
```cpp

/////////////////////////////////////////////////////////////////////////////////////////////////

} // namespace kernel
} // namespace gemm
} // namespace cutlass

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Namespace closure.
**CN**: 命名空间收尾。

---
## Key Concepts / 关键概念
- **EN:** Unlike a normal GEMM, grouped FMHA does not need a full 2D `(M, N)` tile schedule for the outer kernel. Query tiles are assigned over `M`, while key tiles are handled inside the kernel loop.
  **CN:** 与普通 GEMM 不同，grouped FMHA 的外层内核不需要完整的二维 `(M, N)` tile 调度。query tile 在外层按 `M` 维分配，而 key tile 在内核内部循环处理。
- **EN:** `problem_sizes0` and `problem_sizes1` represent different geometric views of the fused computation, so the visitor must preserve both instead of collapsing everything to one GEMM shape.
  **CN:** `problem_sizes0` 与 `problem_sizes1` 代表融合计算的两种几何视图，因此访问器必须同时保留二者，而不能简单压缩成单一 GEMM 形状。
- **EN:** `to_base()` uses `problem_sizes1` for base scheduling because the final output shape is driven by the second matmul/output tensor geometry.
  **CN:** `to_base()` 选择 `problem_sizes1` 作为基类调度输入，因为最终输出形状由第二段矩阵乘法 / 输出张量几何决定。
- **EN:** `fmha_grouped.h` depends on this visitor to implement persistent grouped execution over ragged attention batches.
  **CN:** `fmha_grouped.h` 依赖这个 visitor 来实现对不规则 attention batch 的持久化 grouped 执行。
- **EN:** This header is the scheduling glue that lets the fused kernel scale from one attention problem to many variable-length problems in the same launch.
  **CN:** 该头文件是调度层的胶水代码，使融合内核能够在一次启动中从单个 attention 问题扩展到多个变长问题。
## Dependencies / 依赖项
- `"cutlass/cutlass.h"` — CUTLASS core definitions and portability macros / CUTLASS 核心定义与可移植性宏
- `"cutlass/gemm/gemm.h"` — CUTLASS GEMM shapes, coordinates, and core GEMM types / CUTLASS GEMM 形状、坐标与核心 GEMM 类型
- `"cutlass/matrix_coord.h"` — matrix coordinate helpers and problem-size utilities / 矩阵坐标辅助与问题尺寸工具
- `"cutlass/gemm/kernel/grouped_problem_visitor.h"` — base grouped-problem scheduler specialized here for FMHA / 在此被 FMHA 特化的 grouped problem 调度基类
