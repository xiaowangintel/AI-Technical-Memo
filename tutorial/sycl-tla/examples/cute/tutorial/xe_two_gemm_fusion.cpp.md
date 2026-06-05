# xe_two_gemm_fusion.cpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `examples/cute/tutorial/xe_two_gemm_fusion.cpp`
- **Purpose / 目的:** Demonstrates and validates the repository's gemm pipeline implementation. / 演示并验证仓库中的GEMM 流程实现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14
````cpp
/***************************************************************************************************
* Copyright (C) 2026 Intel Corporation, All rights reserved.
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
````
**EN:** This opening block carries the license banner and file-level description, framing the gemm pipeline example before the executable code begins. It corresponds to block 1 of 93 in the file order.
**CN:** 这一开头代码块给出许可证声明和文件级说明，在真正代码开始前先交代GEMM 流程示例的背景。 它对应本文件顺序中的第 1/93 个代码块。

### Lines 15-28
````cpp
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
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `Neither`, `the`, `name`, `of` showing the main symbols being prepared or consumed here. It corresponds to block 2 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `Neither`、`the`、`name`、`of` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 2/93 个代码块。

### Lines 29-30
````cpp
*
**************************************************************************************************/
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with the surrounding symbols showing the main symbols being prepared or consumed here. It corresponds to block 3 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 周围的符号 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 3/93 个代码块。

### Lines 32-40
````cpp
//
// Two-stage GEMM: D = (A * B) * C
//   Stage 1: AB = A * B   (M,K) x (N1,K)  -> accumulator (M,N1), stored to SLM as half_t
//   Stage 2: D  = AB * C  (M,N1) via SLM  x (N2,N1) -> (M,N2), written to global memory
//
// In this example N1 == N2 == N and K1 == K for simplicity. The intermediate
// result lives entirely in shared local memory (SLM) so that no extra global
// memory traffic is needed between the two stages.
//
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `Two`, `stage`, `GEMM`, `D` showing the main symbols being prepared or consumed here. It corresponds to block 4 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `Two`、`stage`、`GEMM`、`D` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 4/93 个代码块。

### Lines 42-44
````cpp
#include <sycl/sycl.hpp>
#include <cute/util/compat.hpp>
#include <sycl/ext/intel/experimental/grf_size_properties.hpp>
````
**EN:** This block pulls in dependencies required by the file, especially `sycl`, `cute`, so the later gemm pipeline code can use the needed APIs and data structures. It corresponds to block 5 of 93 in the file order.
**CN:** 这一段引入本文件所需的依赖，尤其是 `sycl`、`cute`，使后续GEMM 流程代码可以使用相应 API 和数据结构。 它对应本文件顺序中的第 5/93 个代码块。

### Lines 46-46
````cpp
#include <cute/tensor.hpp>
````
**EN:** This block pulls in dependencies required by the file, especially `cute`, so the later gemm pipeline code can use the needed APIs and data structures. It corresponds to block 6 of 93 in the file order.
**CN:** 这一段引入本文件所需的依赖，尤其是 `cute`，使后续GEMM 流程代码可以使用相应 API 和数据结构。 它对应本文件顺序中的第 6/93 个代码块。

### Lines 48-55
````cpp
#include "cutlass/kernel_hardware_info.h"
#include "cutlass/platform/platform.h"
#include "cutlass/tensor_ref.h"
#include "cutlass/util/sycl_event_manager.hpp"
#include "cutlass/util/GPU_Clock.hpp"
#include "cutlass/util/reference/device/gemm_complex.h"
#include "cutlass/util/reference/device/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_fill.h"
````
**EN:** This block pulls in dependencies required by the file, especially `sycl`, `reference`, so the later gemm pipeline code can use the needed APIs and data structures. It corresponds to block 7 of 93 in the file order.
**CN:** 这一段引入本文件所需的依赖，尤其是 `sycl`、`reference`，使后续GEMM 流程代码可以使用相应 API 和数据结构。 它对应本文件顺序中的第 7/93 个代码块。

### Lines 57-58
````cpp
#include "../../common/sycl_cute_common.hpp"
#include "cutlass/util/command_line.h"
````
**EN:** This block pulls in dependencies required by the file, especially `sycl`, `cute`, so the later gemm pipeline code can use the needed APIs and data structures. It corresponds to block 8 of 93 in the file order.
**CN:** 这一段引入本文件所需的依赖，尤其是 `sycl`、`cute`，使后续GEMM 流程代码可以使用相应 API 和数据结构。 它对应本文件顺序中的第 8/93 个代码块。

### Lines 60-65
````cpp
#if defined(__clang__)
  #pragma clang diagnostic ignored "-Wpass-failed"
  #pragma clang diagnostic ignored "-Wdeprecated-declarations"
#elif defined(__GNUC__)
  #pragma GCC diagnostic ignored "-Wdeprecated-declarations"
#endif
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `defined`, `__clang__`, `pragma`, `clang` showing the main symbols being prepared or consumed here. It corresponds to block 9 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `defined`、`__clang__`、`pragma`、`clang` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 9/93 个代码块。

### Lines 67-69
````cpp
using namespace cute;
// Command line options parsing
struct Options {
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `cute` make the later gemm pipeline code easier to assemble and read. It corresponds to block 10 of 93 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `cute` 这样的符号让后续GEMM 流程代码更容易组装和阅读。 它对应本文件顺序中的第 10/93 个代码块。

### Lines 71-72
````cpp
  bool help;
  bool error;
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `bool`, `help`, `error` showing the main symbols being prepared or consumed here. It corresponds to block 11 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `bool`、`help`、`error` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 11/93 个代码块。

### Lines 74-74
````cpp
  int m, n, k, iterations, verify;
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `verify` showing the main symbols being prepared or consumed here. It corresponds to block 12 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `verify` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 12/93 个代码块。

### Lines 76-80
````cpp
  Options():
    help(false),
    error(false),
    m(128), n(128), k(128), iterations(100), verify(1)
  { }
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `verify` showing the main symbols being prepared or consumed here. It corresponds to block 13 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `verify` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 13/93 个代码块。

### Lines 82-84
````cpp
  // Parses the command line
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `Parses`, `the`, `command`, `line` showing the main symbols being prepared or consumed here. It corresponds to block 14 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `Parses`、`the`、`command`、`line` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 14/93 个代码块。

### Lines 86-89
````cpp
    if (cmd.check_cmd_line_flag("help")) {
      help = true;
      return;
    }
````
**EN:** This block applies conditional control flow. It uses `cmd`, `check_cmd_line_flag`, `help`, `true` to select a path, validate assumptions, or handle special cases in the gemm pipeline implementation. It corresponds to block 15 of 93 in the file order.
**CN:** 这一段实现条件控制流。它借助 `cmd`、`check_cmd_line_flag`、`help`、`true` 在GEMM 流程实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 15/93 个代码块。

### Lines 91-96
````cpp
    cmd.get_cmd_line_argument("m", m, 128);
    cmd.get_cmd_line_argument("n", n, 128);
    cmd.get_cmd_line_argument("k", k, 128);
    cmd.get_cmd_line_argument("iterations", iterations, 100);
    cmd.get_cmd_line_argument("verify", verify, 1);
  }
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `verify` showing the main symbols being prepared or consumed here. It corresponds to block 16 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `verify` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 16/93 个代码块。

### Lines 98-99
````cpp
  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `Prints`, `the`, `usage`, `statement` showing the main symbols being prepared or consumed here. It corresponds to block 17 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `Prints`、`the`、`usage`、`statement` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 17/93 个代码块。

### Lines 101-111
````cpp
    out << "GEMM Example\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement\n\n"
      << "  --m=<int>                   Sets the M extent of the GEMM\n"
      << "  --n=<int>                   Sets the N extent of the GEMM\n"
      << "  --k=<int>                   Sets the K extent of the GEMM\n"
      << "  --iterations=<int>          Iterations\n"
      << "  --verify=<int>              Specify whether to verify.\n\n";
    return out;
  }
};
````
**EN:** This block finalizes a local computation or status path. The use of `verify` helps conclude the current stage cleanly before the next block. It corresponds to block 18 of 93 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `verify`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 18/93 个代码块。

### Lines 113-126
````cpp
// ---------------------------------------------------------------------------
// Device kernel: two-stage GEMM  D = (A * B) * C
// ---------------------------------------------------------------------------
template <class ATensor, class BTensor, class CTensor, class DTensor,
          class TiledMMA>
void
gemm_two_stage_device(ATensor   const& A,     // (M, K)
                      BTensor   const& B,     // (N, K)
                      CTensor   const& C,     // (N2, N1)
                      DTensor        & D,     // (M, N2)
                      TiledMMA  const& mma)
{
  // =========================================================================
  // Full two-stage path
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `Tensor` showing the main symbols being prepared or consumed here. It corresponds to block 19 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `Tensor` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 19/93 个代码块。

### Lines 127-131
````cpp
  // =========================================================================
  auto item     = sycl::ext::oneapi::this_work_item::get_nd_item<2>();
  auto wg_m     = int(item.get_group(1));
  auto wg_n     = int(item.get_group(0));
  auto local_id = int(item.get_local_id(0));
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `sycl` showing the main symbols being prepared or consumed here. It corresponds to block 20 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `sycl` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 20/93 个代码块。

### Lines 133-136
````cpp
  Tensor cA = make_identity_tensor(A.shape());
  Tensor cB = make_identity_tensor(B.shape());
  Tensor cC = make_identity_tensor(C.shape());
  Tensor cD = make_identity_tensor(D.shape());
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `Tensor` showing the main symbols being prepared or consumed here. It corresponds to block 21 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `Tensor` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 21/93 个代码块。

### Lines 138-139
````cpp
  auto wg_tile  = mma.tile_mnk();
  auto wg_coord = make_coord(wg_m, wg_n, 0);
````
**EN:** This block introduces executable logic through a function or method. Here, `wg_tile`, `mma`, `tile_mnk`, `wg_coord` drive a concrete step in the file's gemm pipeline flow. It corresponds to block 22 of 93 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `wg_tile`、`mma`、`tile_mnk`、`wg_coord` 推动了本文件GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 22/93 个代码块。

### Lines 141-142
````cpp
  auto BLK_M = size<0>(wg_tile);
  auto BLK_N = size<1>(wg_tile);
````
**EN:** This block introduces executable logic through a function or method. Here, `BLK_M`, `size<0`, `wg_tile`, `BLK_N` drive a concrete step in the file's gemm pipeline flow. It corresponds to block 23 of 93 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `BLK_M`、`size<0`、`wg_tile`、`BLK_N` 推动了本文件GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 23/93 个代码块。

### Lines 144-150
````cpp
  auto smem = compat::local_mem<half_t[size(select<0,1>(wg_tile))]>();
  Tensor STensor   = make_tensor(make_smem_ptr(smem),
                        make_layout(make_shape(BLK_M, BLK_N),
                                    make_stride(Int<decltype(BLK_N)::value>{}, _1{})));
  Tensor SInTensor = make_tensor(make_smem_ptr(smem),
                        make_layout(make_shape(BLK_N, BLK_M),
                                    make_stride(_1{}, Int<decltype(BLK_N)::value>{})));
````
**EN:** This block introduces executable logic through a function or method. Here, `Tensor` drive a concrete step in the file's gemm pipeline flow. It corresponds to block 24 of 93 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `Tensor` 推动了本文件GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 24/93 个代码块。

### Lines 152-155
````cpp
  Tensor gA_1  = local_tile(cA, select<0,2>(wg_tile), make_coord(wg_m, _));
  Tensor gB_1  = local_tile(cB, select<1,2>(wg_tile), make_coord(wg_n, _));
  Tensor gC_2  = local_tile(cC, select<1,2>(wg_tile), make_coord(wg_n, _));
  Tensor gD_2  = local_tile(cD, wg_tile, wg_coord, Step<_1,_1, X>{});
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `Tensor` showing the main symbols being prepared or consumed here. It corresponds to block 25 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `Tensor` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 25/93 个代码块。

### Lines 157-160
````cpp
  auto copy_a  = make_block_2d_copy_A(mma, A);
  auto copy_b  = make_block_2d_copy_B(mma, B);
  auto copy_c2 = make_block_2d_copy_B(mma, C);
  auto copy_d  = make_block_2d_copy_D(mma, D);
````
**EN:** This block introduces executable logic through a function or method. Here, `copy_a`, `make_block_2d_copy_A`, `mma`, `A` drive a concrete step in the file's gemm pipeline flow. It corresponds to block 26 of 93 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `copy_a`、`make_block_2d_copy_A`、`mma`、`A` 推动了本文件GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 26/93 个代码块。

### Lines 162-165
````cpp
  auto copy_X  = make_block_2d_copy_D(mma,
                    make_tensor(make_gmem_ptr(static_cast<half_t*>(nullptr)), D.layout()));
  auto copy_Y  = make_block_2d_copy_A(mma,
                    make_tensor(A.data(), make_layout(shape(A))));
````
**EN:** This block introduces executable logic through a function or method. Here, `copy_X`, `make_block_2d_copy_D`, `mma`, `make_tensor` drive a concrete step in the file's gemm pipeline flow. It corresponds to block 27 of 93 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `copy_X`、`make_block_2d_copy_D`、`mma`、`make_tensor` 推动了本文件GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 27/93 个代码块。

### Lines 167-171
````cpp
  using StoreAtom  = Copy_Atom<UniversalCopy<half_t>, half_t>;
  using StoreTiler = typename decltype(copy_X)::Tiler_MN;
  using StoreTV    = typename decltype(copy_X)::TiledLayout_TV;
  auto  slm_store  = TiledCopy<StoreAtom, StoreTV, StoreTiler>{};
  auto  thr_slm_st = slm_store.get_slice(local_id);
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `Layout` make the later gemm pipeline code easier to assemble and read. It corresponds to block 28 of 93 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `Layout` 这样的符号让后续GEMM 流程代码更容易组装和阅读。 它对应本文件顺序中的第 28/93 个代码块。

### Lines 173-177
````cpp
  using LoadAtom   = Copy_Atom<UniversalCopy<half_t>, half_t>;
  using LoadTV     = typename decltype(copy_Y)::TiledLayout_TV;
  using LoadTiler  = typename decltype(copy_Y)::Tiler_MN;
  auto  slm_load   = TiledCopy<LoadAtom, LoadTV, LoadTiler>{};
  auto  thr_slm_ld = slm_load.get_slice(local_id);
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `Layout` make the later gemm pipeline code easier to assemble and read. It corresponds to block 29 of 93 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `Layout` 这样的符号让后续GEMM 流程代码更容易组装和阅读。 它对应本文件顺序中的第 29/93 个代码块。

### Lines 179-183
````cpp
  auto thr_mma    = mma.get_slice(local_id);
  auto thr_copy_a = copy_a.get_slice(local_id);
  auto thr_copy_b = copy_b.get_slice(local_id);
  auto thr_copy_c = copy_c2.get_slice(local_id);
  auto thr_copy_Y = copy_Y.get_slice(local_id);
````
**EN:** This block introduces executable logic through a function or method. Here, `thr_mma`, `mma`, `get_slice`, `local_id` drive a concrete step in the file's gemm pipeline flow. It corresponds to block 30 of 93 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `thr_mma`、`mma`、`get_slice`、`local_id` 推动了本文件GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 30/93 个代码块。

### Lines 185-188
````cpp
  auto tCrA_1 = thr_mma.partition_sg_fragment_A(gA_1(_,_,0));
  auto tCrB_1 = thr_mma.partition_sg_fragment_B(gB_1(_,_,0));
  auto tArA_1 = thr_copy_a.partition_sg_fragment_D(gA_1(_,_,0));
  auto tBrB_1 = thr_copy_b.partition_sg_fragment_D(gB_1(_,_,0));
````
**EN:** This block introduces executable logic through a function or method. Here, `tCrA_1`, `thr_mma`, `partition_sg_fragment_A`, `gA_1` drive a concrete step in the file's gemm pipeline flow. It corresponds to block 31 of 93 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `tCrA_1`、`thr_mma`、`partition_sg_fragment_A`、`gA_1` 推动了本文件GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 31/93 个代码块。

### Lines 190-193
````cpp
  auto tCrA_2  = thr_mma.partition_sg_fragment_A(gA_1(_,_,0));
  auto tCrB_2  = thr_mma.partition_sg_fragment_B(gC_2(_,_,0));
  auto tBrB_2  = thr_copy_c.partition_sg_fragment_D(gC_2(_,_,0));
  auto tArA_2  = thr_copy_Y.partition_sg_fragment_D(gA_1(_,_,0));
````
**EN:** This block introduces executable logic through a function or method. Here, `tCrA_2`, `thr_mma`, `partition_sg_fragment_A`, `gA_1` drive a concrete step in the file's gemm pipeline flow. It corresponds to block 32 of 93 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `tCrA_2`、`thr_mma`、`partition_sg_fragment_A`、`gA_1` 推动了本文件GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 32/93 个代码块。

### Lines 195-197
````cpp
  Tensor tAgA_1 = thr_copy_a.partition_S(gA_1);
  Tensor tBgB_1 = thr_copy_b.partition_S(gB_1);
  Tensor tBgC_2 = thr_copy_c.partition_S(gC_2);
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `Tensor` showing the main symbols being prepared or consumed here. It corresponds to block 33 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `Tensor` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 33/93 个代码块。

### Lines 199-199
````cpp
  auto tCrAcc = thr_mma.partition_sg_fragment_C(make_identity_tensor(select<0,1>(wg_tile)));
````
**EN:** This block introduces executable logic through a function or method. Here, `tCrAcc`, `thr_mma`, `partition_sg_fragment_C`, `make_identity_tensor` drive a concrete step in the file's gemm pipeline flow. It corresponds to block 34 of 93 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `tCrAcc`、`thr_mma`、`partition_sg_fragment_C`、`make_identity_tensor` 推动了本文件GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 34/93 个代码块。

### Lines 201-203
````cpp
  auto thr_copy_X = copy_X.get_slice(local_id);
  auto r16 = thr_copy_X.partition_sg_fragment_S(
                 local_tile(cD, wg_tile, wg_coord, Step<_1,_1,X>{}));
````
**EN:** This block introduces executable logic through a function or method. Here, `thr_copy_X`, `copy_X`, `get_slice`, `local_id` drive a concrete step in the file's gemm pipeline flow. It corresponds to block 35 of 93 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `thr_copy_X`、`copy_X`、`get_slice`、`local_id` 推动了本文件GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 35/93 个代码块。

### Lines 205-208
````cpp
  Tensor tOrO = thr_slm_st.retile_S(r16);
  Tensor tOsO = thr_slm_st.partition_D(STensor);
  Tensor tIrI = thr_slm_ld.retile_D(tArA_2);
  Tensor tIsI = thr_slm_ld.partition_S(SInTensor);
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `Tensor` showing the main symbols being prepared or consumed here. It corresponds to block 36 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `Tensor` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 36/93 个代码块。

### Lines 210-210
````cpp
  Tensor tCgD = thr_mma.partition_C(gD_2);
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `Tensor` showing the main symbols being prepared or consumed here. It corresponds to block 37 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `Tensor` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 37/93 个代码块。

### Lines 212-217
````cpp
  auto prefetch_a = make_block_2d_prefetch(copy_a);
  auto prefetch_b = make_block_2d_prefetch(copy_b);
  auto thr_pf_A   = prefetch_a.get_slice(local_id);
  auto thr_pf_B   = prefetch_b.get_slice(local_id);
  auto pAgA       = thr_pf_A.partition_S(gA_1);
  auto pBgB       = thr_pf_B.partition_S(gB_1);
````
**EN:** This block introduces executable logic through a function or method. Here, `prefetch_a`, `make_block_2d_prefetch`, `copy_a`, `prefetch_b` drive a concrete step in the file's gemm pipeline flow. It corresponds to block 38 of 93 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `prefetch_a`、`make_block_2d_prefetch`、`copy_a`、`prefetch_b` 推动了本文件GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 38/93 个代码块。

### Lines 219-222
````cpp
  const int prefetch_dist = 3;
  constexpr int barrier_scope = 2;
  int k_tile_count   = ceil_div(shape<1>(A), get<2>(wg_tile));
  int k_tile_prefetch = 0;
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `prefetch_dist`, `constexpr`, `barrier_scope`, `k_tile_count` showing the main symbols being prepared or consumed here. It corresponds to block 39 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `prefetch_dist`、`constexpr`、`barrier_scope`、`k_tile_count` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 39/93 个代码块。

### Lines 224-224
````cpp
  clear(tCrAcc);
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `clear`, `tCrAcc` showing the main symbols being prepared or consumed here. It corresponds to block 40 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `clear`、`tCrAcc` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 40/93 个代码块。

### Lines 226-230
````cpp
  CUTE_UNROLL
  for (; k_tile_prefetch < prefetch_dist; k_tile_prefetch++) {
    prefetch(prefetch_a, pAgA(_,_,_,k_tile_prefetch));
    prefetch(prefetch_b, pBgB(_,_,_,k_tile_prefetch));
  }
````
**EN:** This block iterates over problem instances or data movement steps. The loop around `CUTE` advances the file toward execution, checking, or benchmarking. It corresponds to block 41 of 93 in the file order.
**CN:** 这一段围绕问题规模或数据移动步骤进行迭代。围绕 `CUTE` 的循环把文件推进到执行、校验或基准测试阶段。 它对应本文件顺序中的第 41/93 个代码块。

### Lines 232-242
````cpp
  for (int k_tile = 0; k_tile < k_tile_count; k_tile++, k_tile_prefetch++) {
    barrier_arrive(barrier_scope);
    copy(copy_a, tAgA_1(_,_,_,k_tile), tArA_1);
    copy(copy_b, tBgB_1(_,_,_,k_tile), tBrB_1);
    prefetch(prefetch_a, pAgA(_,_,_,k_tile_prefetch));
    prefetch(prefetch_b, pBgB(_,_,_,k_tile_prefetch));
    reorder(tArA_1, tCrA_1);
    reorder(tBrB_1, tCrB_1);
    gemm(mma, tCrA_1, tCrB_1, tCrAcc);
    barrier_wait(barrier_scope);
  }
````
**EN:** This block iterates over problem instances or data movement steps. The loop around `k_tile`, `k_tile_count`, `k_tile_prefetch`, `barrier_arrive` advances the file toward execution, checking, or benchmarking. It corresponds to block 42 of 93 in the file order.
**CN:** 这一段围绕问题规模或数据移动步骤进行迭代。围绕 `k_tile`、`k_tile_count`、`k_tile_prefetch`、`barrier_arrive` 的循环把文件推进到执行、校验或基准测试阶段。 它对应本文件顺序中的第 42/93 个代码块。

### Lines 244-245
````cpp
  reorder(tCrAcc, r16);
  copy(slm_store, tOrO, tOsO);
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `reorder`, `tCrAcc`, `r16`, `copy` showing the main symbols being prepared or consumed here. It corresponds to block 43 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `reorder`、`tCrAcc`、`r16`、`copy` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 43/93 个代码块。

### Lines 247-250
````cpp
  barrier_arrive(SPIRVScope::ScopeWorkgroup,
                 SPIRVMemorySemantics::SemanticsRelease | SPIRVMemorySemantics::SemanticsWGMemory);
  barrier_wait  (SPIRVScope::ScopeWorkgroup,
                 SPIRVMemorySemantics::SemanticsAcquire | SPIRVMemorySemantics::SemanticsWGMemory);
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `barrier_arrive`, `SPIRVScope::ScopeWorkgroup`, `SPIRVMemorySemantics::SemanticsRelease`, `SPIRVMemorySemantics::SemanticsWGMemory` showing the main symbols being prepared or consumed here. It corresponds to block 44 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `barrier_arrive`、`SPIRVScope::ScopeWorkgroup`、`SPIRVMemorySemantics::SemanticsRelease`、`SPIRVMemorySemantics::SemanticsWGMemory` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 44/93 个代码块。

### Lines 252-254
````cpp
  // Stage 2
  int k2_tile_count = ceil_div(shape<1>(C), get<2>(wg_tile));
  clear(tCrAcc);
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `Stage`, `k2_tile_count`, `ceil_div`, `shape<1` showing the main symbols being prepared or consumed here. It corresponds to block 45 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `Stage`、`k2_tile_count`、`ceil_div`、`shape<1` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 45/93 个代码块。

### Lines 256-258
````cpp
  for (int k_tile = 0; k_tile < k2_tile_count; k_tile++) {
    copy(slm_load, tIsI(_,_,k_tile), tIrI(_,_,0));
    copy(copy_c2, tBgC_2(_,_,_,k_tile), tBrB_2);
````
**EN:** This block iterates over problem instances or data movement steps. The loop around `k_tile`, `k2_tile_count`, `copy`, `slm_load` advances the file toward execution, checking, or benchmarking. It corresponds to block 46 of 93 in the file order.
**CN:** 这一段围绕问题规模或数据移动步骤进行迭代。围绕 `k_tile`、`k2_tile_count`、`copy`、`slm_load` 的循环把文件推进到执行、校验或基准测试阶段。 它对应本文件顺序中的第 46/93 个代码块。

### Lines 260-263
````cpp
    barrier_arrive(SPIRVScope::ScopeWorkgroup,
                   SPIRVMemorySemantics::SemanticsRelease | SPIRVMemorySemantics::SemanticsWGMemory);
    barrier_wait  (SPIRVScope::ScopeWorkgroup,
                   SPIRVMemorySemantics::SemanticsAcquire | SPIRVMemorySemantics::SemanticsWGMemory);
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `barrier_arrive`, `SPIRVScope::ScopeWorkgroup`, `SPIRVMemorySemantics::SemanticsRelease`, `SPIRVMemorySemantics::SemanticsWGMemory` showing the main symbols being prepared or consumed here. It corresponds to block 47 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `barrier_arrive`、`SPIRVScope::ScopeWorkgroup`、`SPIRVMemorySemantics::SemanticsRelease`、`SPIRVMemorySemantics::SemanticsWGMemory` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 47/93 个代码块。

### Lines 265-268
````cpp
    reorder(tArA_2, tCrA_2);
    reorder(tBrB_2, tCrB_2);
    gemm(mma, tCrA_2, tCrB_2, tCrAcc);
  }
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `reorder`, `tArA_2`, `tCrA_2`, `tBrB_2` showing the main symbols being prepared or consumed here. It corresponds to block 48 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `reorder`、`tArA_2`、`tCrA_2`、`tBrB_2` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 48/93 个代码块。

### Lines 270-271
````cpp
  copy(copy_d, tCrAcc, tCgD);
}
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `copy`, `copy_d`, `tCrAcc`, `tCgD` showing the main symbols being prepared or consumed here. It corresponds to block 49 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `copy`、`copy_d`、`tCrAcc`、`tCgD` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 49/93 个代码块。

### Lines 273-285
````cpp
// ---------------------------------------------------------------------------
// Host helpers (choose MMA, launch kernel, verify, benchmark)
// ---------------------------------------------------------------------------
template <typename TA, typename TB, typename TC>
auto choose_mma_op()
{
  if constexpr (is_complete_v<XE_DPAS_TT<8, TC, TA, TB>>)
    return XE_DPAS_TT<8, TC, TA, TB>{};
  else if constexpr (is_same_v<TA, cute::bfloat16_t>)
    return XE_DPAS_TT<8, float, cute::bfloat16_t>{};
  else
    return XE_DPAS_TT<8, float, cute::half_t>{};
}
````
**EN:** This block applies conditional control flow. It uses `cute`, `benchmark`, `verify` to select a path, validate assumptions, or handle special cases in the gemm pipeline implementation. It corresponds to block 50 of 93 in the file order.
**CN:** 这一段实现条件控制流。它借助 `cute`、`benchmark`、`verify` 在GEMM 流程实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 50/93 个代码块。

### Lines 287-292
````cpp
template <class ATensor, class BTensor, class CTensor>
auto choose_tiled_mma(ATensor const& A, BTensor const& B, CTensor const&)
{
  using TA = typename ATensor::element_type;
  using TB = typename BTensor::element_type;
  using TC = typename CTensor::element_type;
````
**EN:** This block declares a type-level building block for the file, with `Tensor` indicating the configuration, traits, or storage policy used later. It corresponds to block 51 of 93 in the file order.
**CN:** 这一段声明了文件中的类型级构件，其中 `Tensor` 表明后续会使用的配置、traits 或存储策略。 它对应本文件顺序中的第 51/93 个代码块。

### Lines 294-294
````cpp
  auto op = choose_mma_op<TA, TB, TC>();
````
**EN:** This block introduces executable logic through a function or method. Here, `op`, `choose_mma_op<TA`, `TB`, `TC` drive a concrete step in the file's gemm pipeline flow. It corresponds to block 52 of 93 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `op`、`choose_mma_op<TA`、`TB`、`TC` 推动了本文件GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 52/93 个代码块。

### Lines 296-298
````cpp
  constexpr bool byte = (cute::max(sizeof_bits_v<TA>, sizeof_bits_v<TB>) <= 8);
  constexpr bool a_t  = is_constant_v<1, decltype(stride<0>(A))>;
  constexpr bool b_n  = is_constant_v<1, decltype(stride<0>(B))>;
````
**EN:** This block introduces executable logic through a function or method. Here, `cute` drive a concrete step in the file's gemm pipeline flow. It corresponds to block 53 of 93 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `cute` 推动了本文件GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 53/93 个代码块。

### Lines 300-303
````cpp
  constexpr bool use_1x_dpas_per_k = a_t || (byte && b_n);
  constexpr bool use_4x8_sg = ((sizeof_bits_v<TB> < sizeof_bits_v<TA>)
                                  && !(is_same_v<TB, cute::float_e5m2_t>))
                           || (b_n && sizeof_bits_v<TB> < 8);
````
**EN:** This block introduces executable logic through a function or method. Here, `cute` drive a concrete step in the file's gemm pipeline flow. It corresponds to block 54 of 93 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `cute` 推动了本文件GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 54/93 个代码块。

### Lines 305-305
````cpp
  using _K = conditional_t<use_1x_dpas_per_k, C<op.K>, C<op.K*2>>;
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `_K`, `conditional_t<use_1x_dpas_per_k`, `C<op`, `K` make the later gemm pipeline code easier to assemble and read. It corresponds to block 55 of 93 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `_K`、`conditional_t<use_1x_dpas_per_k`、`C<op`、`K` 这样的符号让后续GEMM 流程代码更容易组装和阅读。 它对应本文件顺序中的第 55/93 个代码块。

### Lines 307-310
````cpp
  using WGTile    = Shape<_128, _128, _K>;
  using SGLayout8x4 = Layout<Shape<_4, _4, _1>, Stride<_4, _1, _0>>;
  using SGLayout4x8 = Layout<Shape<_4, _4, _1>, Stride<_4, _1, _0>>;
  using SGLayout    = conditional_t<use_4x8_sg, SGLayout4x8, SGLayout8x4>;
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `Shape`, `Layout` make the later gemm pipeline code easier to assemble and read. It corresponds to block 56 of 93 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `Shape`、`Layout` 这样的符号让后续GEMM 流程代码更容易组装和阅读。 它对应本文件顺序中的第 56/93 个代码块。

### Lines 312-314
````cpp
  using MMA = typename TiledMMAHelper<MMA_Atom<decltype(op)>, Layout<WGTile>, SGLayout>::TiledMMA;
  return MMA{};
}
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `Layout` make the later gemm pipeline code easier to assemble and read. It corresponds to block 57 of 93 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `Layout` 这样的符号让后续GEMM 流程代码更容易组装和阅读。 它对应本文件顺序中的第 57/93 个代码块。

### Lines 316-317
````cpp
// Kernel name helper
template <class, class, char, char> class TwoStageGemmName;
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `Kernel`, `name`, `helper`, `char` showing the main symbols being prepared or consumed here. It corresponds to block 58 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `Kernel`、`name`、`helper`、`char` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 58/93 个代码块。

### Lines 319-328
````cpp
template <class ATensor, class BTensor, class CTensor, class DTensor,
          typename TA, typename TB, char layoutA, char layoutB>
void
gemm_two_stage(sycl::queue &Q,
               ATensor const& A,
               BTensor const& B,
               CTensor const& C,
               DTensor      & D)
{
  auto mma = choose_tiled_mma(A, B, D);
````
**EN:** This block declares a type-level building block for the file, with `Tensor`, `sycl` indicating the configuration, traits, or storage policy used later. It corresponds to block 59 of 93 in the file order.
**CN:** 这一段声明了文件中的类型级构件，其中 `Tensor`、`sycl` 表明后续会使用的配置、traits 或存储策略。 它对应本文件顺序中的第 59/93 个代码块。

### Lines 330-332
````cpp
  sycl::range<2> local  = {size(mma), 1};
  sycl::range<2> global = {local[0] * ceil_div(shape<0>(B), get<1>(mma.tile_mnk())),
                           local[1] * ceil_div(shape<0>(A), get<0>(mma.tile_mnk()))};
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `sycl` showing the main symbols being prepared or consumed here. It corresponds to block 60 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `sycl` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 60/93 个代码块。

### Lines 334-335
````cpp
  namespace syclex  = sycl::ext::oneapi::experimental;
  namespace intelex = sycl::ext::intel::experimental;
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `sycl` showing the main symbols being prepared or consumed here. It corresponds to block 61 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `sycl` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 61/93 个代码块。

### Lines 337-340
````cpp
  syclex::properties kernel_props {
    syclex::sub_group_size<16>,
    intelex::grf_size<256>
  };
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `sycl` showing the main symbols being prepared or consumed here. It corresponds to block 62 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `sycl` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 62/93 个代码块。

### Lines 342-347
````cpp
  auto event = Q.parallel_for<TwoStageGemmName<TA, TB, layoutA, layoutB>>(
    sycl::nd_range<2>(global, local), kernel_props,
    [=](auto) {
      gemm_two_stage_device(A, B, C, D, mma);
    }
  );
````
**EN:** This block introduces executable logic through a function or method. Here, `sycl` drive a concrete step in the file's gemm pipeline flow. It corresponds to block 63 of 93 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `sycl` 推动了本文件GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 63/93 个代码块。

### Lines 349-350
````cpp
  EventManager::getInstance().addEvent(event);
}
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `EventManager::getInstance`, `addEvent`, `event` showing the main symbols being prepared or consumed here. It corresponds to block 64 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `EventManager::getInstance`、`addEvent`、`event` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 64/93 个代码块。

### Lines 352-353
````cpp
// Verification: D_ref = (A * B) * C  computed element-wise on device
template <class...> class TwoStageVerifyName;
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `Verification`, `D_ref`, `A`, `B` showing the main symbols being prepared or consumed here. It corresponds to block 65 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `Verification`、`D_ref`、`A`、`B` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 65/93 个代码块。

### Lines 355-365
````cpp
template <class ATensor, class BTensor, class CTensor, class DTensor>
bool
verify_two_stage(sycl::queue &Q,
                 ATensor const& A,   // (M, K)
                 BTensor const& B,   // (N, K)
                 CTensor const& C,   // (N, N)  -- inner dim matches N
                 DTensor const& D)   // (M, N)
{
  int m  = size<0>(A);
  int n  = size<0>(B);      // == size<0>(C) == size<1>(D)
  int k  = size<1>(A);
````
**EN:** This block declares a type-level building block for the file, with `Tensor`, `sycl`, `verify` indicating the configuration, traits, or storage policy used later. It corresponds to block 66 of 93 in the file order.
**CN:** 这一段声明了文件中的类型级构件，其中 `Tensor`、`sycl`、`verify` 表明后续会使用的配置、traits 或存储策略。 它对应本文件顺序中的第 66/93 个代码块。

### Lines 367-368
````cpp
  auto ok = sycl::malloc_shared<bool>(1, Q);
  *ok = true;
````
**EN:** This block introduces executable logic through a function or method. Here, `sycl` drive a concrete step in the file's gemm pipeline flow. It corresponds to block 67 of 93 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `sycl` 推动了本文件GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 67/93 个代码块。

### Lines 370-371
````cpp
  using AccType = typename DTensor::element_type;
  using SignedAccType = ensure_signed_t<AccType>;
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `Tensor` make the later gemm pipeline code easier to assemble and read. It corresponds to block 68 of 93 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `Tensor` 这样的符号让后续GEMM 流程代码更容易组装和阅读。 它对应本文件顺序中的第 68/93 个代码块。

### Lines 373-375
````cpp
  Q.parallel_for<TwoStageVerifyName<ATensor, BTensor, CTensor, DTensor>>(
    sycl::range<2>(m, n), [=](sycl::item<2> id) {
      int i = id[0], j = id[1];
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `Tensor`, `sycl` showing the main symbols being prepared or consumed here. It corresponds to block 69 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `Tensor`、`sycl` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 69/93 个代码块。

### Lines 377-390
````cpp
      // Stage 1: AB(m, n) = sum_h A(m, h) * B(n, h)  (standard convention)
      // SLM store (row-major) + load (col-major) transposes the intermediate,
      // so stage 2 effectively sees AB^T as the A operand:
      //   D(i, j) = sum_p AB(p, i) * C(j, p)
      AccType d_val = AccType(0);
      for (int p = 0; p < n; p++) {
        AccType ab = AccType(0);
        for (int h = 0; h < k; h++)
          ab += AccType(A(p, h)) * AccType(B(i, h));   // AB(p, i)
        d_val += ab * AccType(C(j, p));
      }
      auto tol = AccType(2e-1f);   // two-stage accumulation has larger error
      if (std::abs(SignedAccType(d_val - AccType(D(i, j)))) > tol) {
        printf("Error at (%d,%d): got %f, expected %f\n", i, j, double(D(i, j)), double(d_val));
````
**EN:** This block iterates over problem instances or data movement steps. The loop around `Stage`, `AB`, `m`, `n` advances the file toward execution, checking, or benchmarking. It corresponds to block 70 of 93 in the file order.
**CN:** 这一段围绕问题规模或数据移动步骤进行迭代。围绕 `Stage`、`AB`、`m`、`n` 的循环把文件推进到执行、校验或基准测试阶段。 它对应本文件顺序中的第 70/93 个代码块。

### Lines 391-393
````cpp
        *ok = false;
      }
    }).wait();
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `ok`, `false`, `wait` showing the main symbols being prepared or consumed here. It corresponds to block 71 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `ok`、`false`、`wait` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 71/93 个代码块。

### Lines 395-398
````cpp
  bool read_ok = *ok;
  sycl::free(ok, Q);
  return read_ok;
}
````
**EN:** This block finalizes a local computation or status path. The use of `sycl` helps conclude the current stage cleanly before the next block. It corresponds to block 72 of 93 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `sycl`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 72/93 个代码块。

### Lines 400-411
````cpp
// ---------------------------------------------------------------------------
// test_case
// ---------------------------------------------------------------------------
template <typename TA, typename TB, typename TC,
          char layoutA = 'R', char layoutB = 'R'>
void
test_case(sycl::queue &Q, int m, int n, int k, int iterations, int verify)
{
  std::cout << "Two-stage GEMM: D = (A*B)*C\n  "
            << type_str<TA>() << " (" << layoutA << ") x "
            << type_str<TB>() << " (" << layoutB << ") -> "
            << type_str<TC>() << ": \t";
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `sycl`, `verify` showing the main symbols being prepared or consumed here. It corresponds to block 73 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `sycl`、`verify` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 73/93 个代码块。

### Lines 413-413
````cpp
  constexpr char tlayoutB = layoutB ^ ('R' ^ 'C');
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `constexpr`, `char`, `tlayoutB`, `layoutB` showing the main symbols being prepared or consumed here. It corresponds to block 74 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `constexpr`、`char`、`tlayoutB`、`layoutB` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 74/93 个代码块。

### Lines 415-419
````cpp
  // A (M, K),  B (N, K),  C (N, N),  D (M, N)
  auto A = make_shared_usm_tensor<TA,   layoutA>(Q, m, k);
  auto B = make_shared_usm_tensor<TB, tlayoutB>(Q, n, k);
  auto C_mat = make_shared_usm_tensor<TB, tlayoutB>(Q, n, n);   // second operand, same type as B
  auto D = make_shared_usm_tensor<TC,      'R'>(Q, m, n);
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `A`, `M`, `K`, `B` showing the main symbols being prepared or consumed here. It corresponds to block 75 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `A`、`M`、`K`、`B` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 75/93 个代码块。

### Lines 421-424
````cpp
  random_fill(A);
  random_fill(B);
  random_fill(C_mat);
  zero_fill(D);
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `random_fill`, `A`, `B`, `C_mat` showing the main symbols being prepared or consumed here. It corresponds to block 76 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `random_fill`、`A`、`B`、`C_mat` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 76/93 个代码块。

### Lines 427-430
````cpp
  auto A_ref = make_shared_usm_tensor<float,  layoutA>(Q, m, k);
  auto B_ref = make_shared_usm_tensor<float, tlayoutB>(Q, n, k);
  auto C_ref = make_shared_usm_tensor<float, tlayoutB>(Q, n, n);
  auto D_ref = make_shared_usm_tensor<float,      'R'>(Q, m, n);
````
**EN:** This block introduces executable logic through a function or method. Here, `A_ref`, `make_shared_usm_tensor<float`, `layoutA`, `Q` drive a concrete step in the file's gemm pipeline flow. It corresponds to block 77 of 93 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `A_ref`、`make_shared_usm_tensor<float`、`layoutA`、`Q` 推动了本文件GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 77/93 个代码块。

### Lines 432-435
````cpp
  copy(A, A_ref);
  copy(B, B_ref);
  copy(C_mat, C_ref);
  copy(D, D_ref);
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `copy`, `A`, `A_ref`, `B` showing the main symbols being prepared or consumed here. It corresponds to block 78 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `copy`、`A`、`A_ref`、`B` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 78/93 个代码块。

### Lines 437-439
````cpp
  subbyte_pack(A);
  subbyte_pack(B);
  subbyte_pack(C_mat);
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `subbyte_pack`, `A`, `B`, `C_mat` showing the main symbols being prepared or consumed here. It corresponds to block 79 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `subbyte_pack`、`A`、`B`、`C_mat` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 79/93 个代码块。

### Lines 441-443
````cpp
  gemm_two_stage<decltype(A), decltype(B), decltype(C_mat), decltype(D),
                 TA, TB, layoutA, layoutB>(Q, A, B, C_mat, D);
  Q.wait_and_throw();
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `gemm_two_stage<decltype`, `A`, `decltype`, `B` showing the main symbols being prepared or consumed here. It corresponds to block 80 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `gemm_two_stage<decltype`、`A`、`decltype`、`B` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 80/93 个代码块。

### Lines 445-452
````cpp
  bool ok = true;
  if (verify) {
    copy(D, D_ref);  // copy kernel output to D_ref for verification
    bool ok = verify_two_stage(Q, A_ref, B_ref, C_ref, D_ref);
    std::cout << (ok ? "passed" : "failed");
  } else {
    std::cout << "skipped verification";
  }
````
**EN:** This block applies conditional control flow. It uses `verify` to select a path, validate assumptions, or handle special cases in the gemm pipeline implementation. It corresponds to block 81 of 93 in the file order.
**CN:** 这一段实现条件控制流。它借助 `verify` 在GEMM 流程实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 81/93 个代码块。

### Lines 454-461
````cpp
  if (ok) {
    const int timing_iterations = iterations;
    GPU_Clock timer;
    timer.start();
    for (int i = 0; i < timing_iterations; ++i)
      gemm_two_stage<decltype(A), decltype(B), decltype(C_mat), decltype(D),
                     TA, TB, layoutA, layoutB>(Q, A, B, C_mat, D);
    Q.wait_and_throw();
````
**EN:** This block iterates over problem instances or data movement steps. The loop around `ok`, `timing_iterations`, `iterations`, `GPU_Clock` advances the file toward execution, checking, or benchmarking. It corresponds to block 82 of 93 in the file order.
**CN:** 这一段围绕问题规模或数据移动步骤进行迭代。围绕 `ok`、`timing_iterations`、`iterations`、`GPU_Clock` 的循环把文件推进到执行、校验或基准测试阶段。 它对应本文件顺序中的第 82/93 个代码块。

### Lines 463-467
````cpp
    double avg  = timer.seconds() / timing_iterations;
    // Two GEMMs: M*N*K + M*N*N
    double tops = (2.0*m*n*k + 2.0*m*n*n) * 1e-12;
    printf(", %4.3f TF/s", tops / avg);
  }
````
**EN:** This block introduces executable logic through a function or method. Here, `double`, `avg`, `timer`, `seconds` drive a concrete step in the file's gemm pipeline flow. It corresponds to block 83 of 93 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `double`、`avg`、`timer`、`seconds` 推动了本文件GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 83/93 个代码块。

### Lines 469-472
````cpp
  free_usm_tensor(A, Q);
  free_usm_tensor(B, Q);
  free_usm_tensor(C_mat, Q);
  free_usm_tensor(D, Q);
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `free_usm_tensor`, `A`, `Q`, `B` showing the main symbols being prepared or consumed here. It corresponds to block 84 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `free_usm_tensor`、`A`、`Q`、`B` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 84/93 个代码块。

### Lines 474-477
````cpp
  free_usm_tensor(A_ref, Q);
  free_usm_tensor(B_ref, Q);
  free_usm_tensor(C_ref, Q);
  free_usm_tensor(D_ref, Q);
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `free_usm_tensor`, `A_ref`, `Q`, `B_ref` showing the main symbols being prepared or consumed here. It corresponds to block 85 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `free_usm_tensor`、`A_ref`、`Q`、`B_ref` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 85/93 个代码块。

### Lines 479-480
````cpp
  std::cout << '\n';
}
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `n` showing the main symbols being prepared or consumed here. It corresponds to block 86 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `n` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 86/93 个代码块。

### Lines 482-487
````cpp
// ---------------------------------------------------------------------------
// main
// ---------------------------------------------------------------------------
int main(int argc, const char** argv)
{
  Options options;
````
**EN:** This block defines the entry path of the example, connecting setup, execution, and reporting for the gemm pipeline run. It corresponds to block 87 of 93 in the file order.
**CN:** 这一段定义了示例的入口路径，把GEMM 流程运行中的初始化、执行与结果报告串联起来。 它对应本文件顺序中的第 87/93 个代码块。

### Lines 489-489
````cpp
  options.parse(argc, argv);
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `options`, `parse`, `argc`, `argv` showing the main symbols being prepared or consumed here. It corresponds to block 88 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `options`、`parse`、`argc`、`argv` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 88/93 个代码块。

### Lines 491-495
````cpp
  auto m = options.m;
  auto n = options.n;
  auto k = options.k;
  auto iterations = options.iterations;
  auto verify = options.verify;
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `verify` showing the main symbols being prepared or consumed here. It corresponds to block 89 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `verify` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 89/93 个代码块。

### Lines 497-500
````cpp
  if (options.help) {
    options.print_usage(std::cout) << std::endl;
    return 0;
  }
````
**EN:** This block applies conditional control flow. It uses `options`, `help`, `print_usage` to select a path, validate assumptions, or handle special cases in the gemm pipeline implementation. It corresponds to block 90 of 93 in the file order.
**CN:** 这一段实现条件控制流。它借助 `options`、`help`、`print_usage` 在GEMM 流程实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 90/93 个代码块。

### Lines 502-505
````cpp
  if (options.error) {
    std::cerr << "Aborting execution." << std::endl;
    return -1;
  }
````
**EN:** This block applies conditional control flow. It uses `options`, `error`, `Aborting`, `execution` to select a path, validate assumptions, or handle special cases in the gemm pipeline implementation. It corresponds to block 91 of 93 in the file order.
**CN:** 这一段实现条件控制流。它借助 `options`、`error`、`Aborting`、`execution` 在GEMM 流程实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 91/93 个代码块。

### Lines 507-507
````cpp
  sycl::queue Q = compat::get_default_queue();
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `sycl` showing the main symbols being prepared or consumed here. It corresponds to block 92 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `sycl` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 92/93 个代码块。

### Lines 509-511
````cpp
  // D = (A * B) * C,   A(M,K) * B(N,K) -> AB(M,N),  AB(M,N) * C(N,N) -> D(M,N)
  test_case<half_t, half_t, float, 'R', 'R'>(Q, m, n, k, iterations, verify);
}
````
**EN:** This block continues the file's gemm pipeline setup or compute path, with `verify` showing the main symbols being prepared or consumed here. It corresponds to block 93 of 93 in the file order.
**CN:** 这一段继续推进本文件的GEMM 流程初始化或计算流程，其中 `verify` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 93/93 个代码块。

## Key Concepts / 关键概念
- **EN:** Template-based GEMM composition with CUTLASS collectives, adapters, or visitors.
  **CN:** 使用 CUTLASS collective、adapter 或 visitor 进行模板化 GEMM 组合。
- **EN:** CUTE layout algebra is used to describe shapes, coordinates, and tensor views.
  **CN:** 使用 CUTE 布局代数来描述形状、坐标和张量视图。
- **EN:** SYCL objects and queues act as the execution interface for heterogeneous kernels.
  **CN:** SYCL 对象与队列构成异构内核的执行接口。
- **EN:** The file mixes compile-time configuration with runtime problem sizes or arguments.
  **CN:** 该文件把编译期配置与运行期问题规模或参数结合起来。
- **EN:** Validation, benchmarking, or reporting code is interleaved with kernel setup.
  **CN:** 校验、基准测试或结果报告代码与内核配置过程交织在一起。

## Dependencies / 依赖关系
- **Direct dependencies / 直接依赖:** `sycl/sycl.hpp`, `cute/util/compat.hpp`, `sycl/ext/intel/experimental/grf_size_properties.hpp`, `cute/tensor.hpp`, `cutlass/kernel_hardware_info.h`, `cutlass/platform/platform.h`, `cutlass/tensor_ref.h`, `cutlass/util/sycl_event_manager.hpp`, `cutlass/util/GPU_Clock.hpp`, `cutlass/util/reference/device/gemm_complex.h`, `cutlass/util/reference/device/tensor_compare.h`, `cutlass/util/reference/host/tensor_fill.h`, ...
- **Runtime expectations / 运行时依赖:** SYCL runtime / SYCL 运行时; CUTLASS/CUTE templates / CUTLASS/CUTE 模板
- **Build/context note / 构建说明:** The file relies on surrounding repository infrastructure, compile flags, and example utilities. / 该文件依赖仓库周边基础设施、编译选项以及示例辅助代码。
