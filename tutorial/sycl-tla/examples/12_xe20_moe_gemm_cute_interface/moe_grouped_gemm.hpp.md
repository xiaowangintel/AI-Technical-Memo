# moe_grouped_gemm.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `examples/12_xe20_moe_gemm_cute_interface/moe_grouped_gemm.hpp`
- **Purpose / 目的:** Defines reusable declarations and helper logic for the repository's grouped gemm. / 为仓库中的分组 GEMM提供可复用的声明与辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14
````cpp
/***************************************************************************************************
 * Copyright (C) 2025 - 2026 Intel corporation. All rights reserved.
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
**EN:** This opening block carries the license banner and file-level description, framing the grouped gemm example before the executable code begins. It corresponds to block 1 of 20 in the file order.
**CN:** 这一开头代码块给出许可证声明和文件级说明，在真正代码开始前先交代分组 GEMM示例的背景。 它对应本文件顺序中的第 1/20 个代码块。

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
**EN:** This block continues the file's grouped gemm setup or compute path, with `Neither`, `the`, `name`, `of` showing the main symbols being prepared or consumed here. It corresponds to block 2 of 20 in the file order.
**CN:** 这一段继续推进本文件的分组 GEMM初始化或计算流程，其中 `Neither`、`the`、`name`、`of` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 2/20 个代码块。

### Lines 29-32
````cpp
 *POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/
#pragma once
````
**EN:** This block continues the file's grouped gemm setup or compute path, with `POSSIBILITY`, `OF`, `SUCH`, `DAMAGE` showing the main symbols being prepared or consumed here. It corresponds to block 3 of 20 in the file order.
**CN:** 这一段继续推进本文件的分组 GEMM初始化或计算流程，其中 `POSSIBILITY`、`OF`、`SUCH`、`DAMAGE` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 3/20 个代码块。

### Lines 34-43
````cpp
#include "cute/tensor.hpp"
#include "cutlass/cutlass.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/gemm/group_array_problem_shape.hpp"
#include "cutlass/gemm/kernel/tile_scheduler.hpp"
#include "cutlass/kernel_hardware_info.hpp"
#include "cutlass/platform/platform.h"
#include "moe_gemms.hpp"
#include "moe_tile_scheduler.hpp"
#include <cute/util/compat.hpp>
````
**EN:** This block pulls in dependencies required by the file, especially `cute`, so the later grouped gemm code can use the needed APIs and data structures. It corresponds to block 4 of 20 in the file order.
**CN:** 这一段引入本文件所需的依赖，尤其是 `cute`，使后续分组 GEMM代码可以使用相应 API 和数据结构。 它对应本文件顺序中的第 4/20 个代码块。

### Lines 45-46
````cpp
#pragma clang diagnostic ignored "-Wpass-failed"
#pragma clang diagnostic ignored "-Wdeprecated-declarations"
````
**EN:** This block continues the file's grouped gemm setup or compute path, with `pragma`, `clang`, `diagnostic`, `ignored` showing the main symbols being prepared or consumed here. It corresponds to block 5 of 20 in the file order.
**CN:** 这一段继续推进本文件的分组 GEMM初始化或计算流程，其中 `pragma`、`clang`、`diagnostic`、`ignored` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 5/20 个代码块。

### Lines 48-49
````cpp
namespace MoE {
using namespace cute;
````
**EN:** This block continues the file's grouped gemm setup or compute path, with `cute` showing the main symbols being prepared or consumed here. It corresponds to block 6 of 20 in the file order.
**CN:** 这一段继续推进本文件的分组 GEMM初始化或计算流程，其中 `cute` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 6/20 个代码块。

### Lines 51-54
````cpp
using ProblemShapeMNKL = Shape<int, int, int, int>;
using ProblemShape = cutlass::gemm::GroupProblemShape<Shape<int, int, int>>;
using TileScheduler = typename MoE::PersistentTileSchedulerXeMoE<ProblemShape>;
using RasterOrderOptions = typename TileScheduler::RasterOrderOptions;
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `Shape` make the later grouped gemm code easier to assemble and read. It corresponds to block 7 of 20 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `Shape` 这样的符号让后续分组 GEMM代码更容易组装和阅读。 它对应本文件顺序中的第 7/20 个代码块。

### Lines 56-65
````cpp
template <typename T, char LayoutKind>
CUTE_DEVICE auto make_moe_tensor(T *ptr, int r, int c) {
  auto shape = make_shape(r, c);
  if constexpr (LayoutKind == 'C')
    return make_tensor(make_gmem_ptr<T>(ptr),
                       make_layout(shape, make_stride(_1{}, r)));
  else
    return make_tensor(make_gmem_ptr<T>(ptr),
                       make_layout(shape, make_stride(c, _1{})));
}
````
**EN:** This block declares a type-level building block for the file, with `Layout`, `CUTE` indicating the configuration, traits, or storage policy used later. It corresponds to block 8 of 20 in the file order.
**CN:** 这一段声明了文件中的类型级构件，其中 `Layout`、`CUTE` 表明后续会使用的配置、traits 或存储策略。 它对应本文件顺序中的第 8/20 个代码块。

### Lines 67-76
````cpp
template <class GmemTiledCopyA, class GmemTiledCopyB, class GmemTiledCopyD,
          char LayoutKindA, char LayoutKindB, char LayoutKindD, class TiledMMA,
          typename ElementA, typename ElementB, typename ElementS,
          typename ElementD>
CUTE_DEVICE void
MoEGEMM(const ElementA *Activations, const ElementB *Weights,
        const ElementS *Scales, ElementD *Outputs, TiledMMA const &mma,
        const int32_t *M_per_group, const int32_t num_experts, const int32_t N,
        const int32_t K,
        PersistentTileSchedulerSm90GroupParams<ProblemShape> scheduler_params) {
````
**EN:** This block declares a type-level building block for the file, with `Shape`, `Layout`, `CUTE` indicating the configuration, traits, or storage policy used later. It corresponds to block 9 of 20 in the file order.
**CN:** 这一段声明了文件中的类型级构件，其中 `Shape`、`Layout`、`CUTE` 表明后续会使用的配置、traits 或存储策略。 它对应本文件顺序中的第 9/20 个代码块。

### Lines 78-79
````cpp
  TileScheduler scheduler{scheduler_params, const_cast<int32_t *>(M_per_group),
                          N, K, num_experts};
````
**EN:** This block continues the file's grouped gemm setup or compute path, with `TileScheduler`, `scheduler`, `scheduler_params`, `const_cast<int32_t` showing the main symbols being prepared or consumed here. It corresponds to block 10 of 20 in the file order.
**CN:** 这一段继续推进本文件的分组 GEMM初始化或计算流程，其中 `TileScheduler`、`scheduler`、`scheduler_params`、`const_cast<int32_t` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 10/20 个代码块。

### Lines 81-87
````cpp
  auto work_tile_info = scheduler.initial_work_tile_info(Shape<_1, _1, _1>{});
  constexpr char actual_layout_of_B = LayoutKindB ^ ('R' ^ 'C');
  bool did_group_change = true;
  int32_t curr_group = 0;
  int32_t prev_group = 0;
  int32_t cumulative_M = 0;
  int32_t M = 0;
````
**EN:** This block introduces executable logic through a function or method. Here, `Shape`, `Layout` drive a concrete step in the file's grouped gemm flow. It corresponds to block 11 of 20 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `Shape`、`Layout` 推动了本文件分组 GEMM流程中的一个具体步骤。 它对应本文件顺序中的第 11/20 个代码块。

### Lines 89-95
````cpp
  if (work_tile_info.is_valid()) {
    // We don't really need this conditional outside the while loop.
    // It simply helps initialize tensors. If using nullptr would be
    // fine for their initialization, then we can remove this conditional.
    curr_group = work_tile_info.L_idx;
    M = M_per_group[curr_group];
  }
````
**EN:** This block applies conditional control flow. It uses `work_tile_info`, `is_valid`, `We`, `don` to select a path, validate assumptions, or handle special cases in the grouped gemm implementation. It corresponds to block 12 of 20 in the file order.
**CN:** 这一段实现条件控制流。它借助 `work_tile_info`、`is_valid`、`We`、`don` 在分组 GEMM实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 12/20 个代码块。

### Lines 97-101
````cpp
  auto A_tensor = make_moe_tensor<ElementA, LayoutKindA>(
      const_cast<ElementA *>(Activations), M, K);
  auto B_tensor = make_moe_tensor<ElementB, actual_layout_of_B>(
      const_cast<ElementB *>(Weights), N, K);
  auto D_tensor = make_moe_tensor<ElementD, LayoutKindD>(Outputs, M, N);
````
**EN:** This block introduces executable logic through a function or method. Here, `Layout` drive a concrete step in the file's grouped gemm flow. It corresponds to block 13 of 20 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `Layout` 推动了本文件分组 GEMM流程中的一个具体步骤。 它对应本文件顺序中的第 13/20 个代码块。

### Lines 103-106
````cpp
  while (work_tile_info.is_valid()) {
    auto m_coord = work_tile_info.M_idx;
    auto n_coord = work_tile_info.N_idx;
    auto tile_coord = make_coord(m_coord, n_coord, _, 0);
````
**EN:** This block continues the file's grouped gemm setup or compute path, with `work_tile_info`, `is_valid`, `m_coord`, `M_idx` showing the main symbols being prepared or consumed here. It corresponds to block 14 of 20 in the file order.
**CN:** 这一段继续推进本文件的分组 GEMM初始化或计算流程，其中 `work_tile_info`、`is_valid`、`m_coord`、`M_idx` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 14/20 个代码块。

### Lines 108-115
````cpp
    if (did_group_change) {
      curr_group = work_tile_info.L_idx;
      M = M_per_group[curr_group];
      // recompute each time because the groups don't necessarily increment by 1
      for (int i = prev_group; i < curr_group; i++) {
        cumulative_M += M_per_group[i];
      }
      prev_group = curr_group;
````
**EN:** This block iterates over problem instances or data movement steps. The loop around `did_group_change`, `curr_group`, `work_tile_info`, `L_idx` advances the file toward execution, checking, or benchmarking. It corresponds to block 15 of 20 in the file order.
**CN:** 这一段围绕问题规模或数据移动步骤进行迭代。围绕 `did_group_change`、`curr_group`、`work_tile_info`、`L_idx` 的循环把文件推进到执行、校验或基准测试阶段。 它对应本文件顺序中的第 15/20 个代码块。

### Lines 117-121
````cpp
      ElementA *ptr_A_curr_batch =
          const_cast<ElementA *>(Activations) + int64_t(cumulative_M) * K;
      ElementB *ptr_B_curr_batch =
          const_cast<ElementB *>(Weights) + int64_t(curr_group) * K * N;
      ElementD *ptr_D_curr_batch = Outputs + int64_t(cumulative_M) * N;
````
**EN:** This block continues the file's grouped gemm setup or compute path, with `ElementA`, `ptr_A_curr_batch`, `const_cast<ElementA`, `Activations` showing the main symbols being prepared or consumed here. It corresponds to block 16 of 20 in the file order.
**CN:** 这一段继续推进本文件的分组 GEMM初始化或计算流程，其中 `ElementA`、`ptr_A_curr_batch`、`const_cast<ElementA`、`Activations` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 16/20 个代码块。

### Lines 123-128
````cpp
      A_tensor = make_moe_tensor<ElementA, LayoutKindA>(ptr_A_curr_batch, M, K);
      B_tensor =
          make_moe_tensor<ElementB, actual_layout_of_B>(ptr_B_curr_batch, N, K);
      D_tensor = make_moe_tensor<ElementD, LayoutKindD>(ptr_D_curr_batch, M, N);
      did_group_change = false;
    }
````
**EN:** This block continues the file's grouped gemm setup or compute path, with `Layout` showing the main symbols being prepared or consumed here. It corresponds to block 17 of 20 in the file order.
**CN:** 这一段继续推进本文件的分组 GEMM初始化或计算流程，其中 `Layout` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 17/20 个代码块。

### Lines 130-136
````cpp
    // After adding scaledMM mainloops, add something like
    // if constexpr (!cute::is_void_v<ElementS>) {
    //   moe_gemm<GmemTiledCopyA, GmemTiledCopyB, GmemTiledCopyD>(
    //      A_tensor, B_tensor, Scales, D_tensor, tile_coord, mma);
    // } else {
    moe_gemm<GmemTiledCopyA, GmemTiledCopyB, GmemTiledCopyD>(
        A_tensor, B_tensor, D_tensor, tile_coord, mma);
````
**EN:** This block continues the file's grouped gemm setup or compute path, with `mainloop`, `cute` showing the main symbols being prepared or consumed here. It corresponds to block 18 of 20 in the file order.
**CN:** 这一段继续推进本文件的分组 GEMM初始化或计算流程，其中 `mainloop`、`cute` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 18/20 个代码块。

### Lines 138-142
````cpp
    // Get next work tile
    work_tile_info = scheduler.fetch_next_work(work_tile_info);
    did_group_change = curr_group != work_tile_info.L_idx;
  } // end while loop
}
````
**EN:** This block continues the file's grouped gemm setup or compute path, with `Get`, `next`, `work`, `tile` showing the main symbols being prepared or consumed here. It corresponds to block 19 of 20 in the file order.
**CN:** 这一段继续推进本文件的分组 GEMM初始化或计算流程，其中 `Get`、`next`、`work`、`tile` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 19/20 个代码块。

### Lines 144-144
````cpp
} // namespace MoE
````
**EN:** This block continues the file's grouped gemm setup or compute path, with `namespace`, `MoE` showing the main symbols being prepared or consumed here. It corresponds to block 20 of 20 in the file order.
**CN:** 这一段继续推进本文件的分组 GEMM初始化或计算流程，其中 `namespace`、`MoE` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 20/20 个代码块。

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
- **Direct dependencies / 直接依赖:** `cute/tensor.hpp`, `cutlass/cutlass.h`, `cutlass/gemm/gemm.h`, `cutlass/gemm/group_array_problem_shape.hpp`, `cutlass/gemm/kernel/tile_scheduler.hpp`, `cutlass/kernel_hardware_info.hpp`, `cutlass/platform/platform.h`, `moe_gemms.hpp`, `moe_tile_scheduler.hpp`, `cute/util/compat.hpp`
- **Runtime expectations / 运行时依赖:** CUTLASS/CUTE templates / CUTLASS/CUTE 模板
- **Build/context note / 构建说明:** The file relies on surrounding repository infrastructure, compile flags, and example utilities. / 该文件依赖仓库周边基础设施、编译选项以及示例辅助代码。
