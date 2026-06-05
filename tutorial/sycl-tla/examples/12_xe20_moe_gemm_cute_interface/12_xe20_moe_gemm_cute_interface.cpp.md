# 12_xe20_moe_gemm_cute_interface.cpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `examples/12_xe20_moe_gemm_cute_interface/12_xe20_moe_gemm_cute_interface.cpp`
- **Purpose / 目的:** Demonstrates and validates the repository's mixture-of-experts gemm flow implementation. / 演示并验证仓库中的MoE GEMM 流程实现。

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
**EN:** This opening block carries the license banner and file-level description, framing the mixture-of-experts gemm flow example before the executable code begins. It corresponds to block 1 of 70 in the file order.
**CN:** 这一开头代码块给出许可证声明和文件级说明，在真正代码开始前先交代MoE GEMM 流程示例的背景。 它对应本文件顺序中的第 1/70 个代码块。

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
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `Neither`, `the`, `name`, `of` showing the main symbols being prepared or consumed here. It corresponds to block 2 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `Neither`、`the`、`name`、`of` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 2/70 个代码块。

### Lines 29-33
````cpp
 *POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/
/*! \file
    \brief CUTLASS Intel BMG MoE API example based on sycl-tla Group GEMM
````
**EN:** This opening block carries the license banner and file-level description, framing the mixture-of-experts gemm flow example before the executable code begins. It corresponds to block 3 of 70 in the file order.
**CN:** 这一开头代码块给出许可证声明和文件级说明，在真正代码开始前先交代MoE GEMM 流程示例的背景。 它对应本文件顺序中的第 3/70 个代码块。

### Lines 35-39
````cpp
    Usage:
      To skip verification, modify the verify parameter in the launcher() calls in main():
        launcher(total_rows_for_each_expert[i], 5760, 2880, num_experts, 0);  // 0 = skip verify
        launcher(total_rows_for_each_expert[i], 2880, 2880, num_experts, 1);  // 1 = enable verify (default)
*/
````
**EN:** This block defines the entry path of the example, connecting setup, execution, and reporting for the mixture-of-experts gemm flow run. It corresponds to block 4 of 70 in the file order.
**CN:** 这一段定义了示例的入口路径，把MoE GEMM 流程运行中的初始化、执行与结果报告串联起来。 它对应本文件顺序中的第 4/70 个代码块。

### Lines 41-41
````cpp
#include "cutlass/util/GPU_Clock.hpp"
````
**EN:** This block pulls in dependencies required by the file, especially `include`, `cutlass`, `util`, `GPU_Clock`, so the later mixture-of-experts gemm flow code can use the needed APIs and data structures. It corresponds to block 5 of 70 in the file order.
**CN:** 这一段引入本文件所需的依赖，尤其是 `include`、`cutlass`、`util`、`GPU_Clock`，使后续MoE GEMM 流程代码可以使用相应 API 和数据结构。 它对应本文件顺序中的第 5/70 个代码块。

### Lines 43-44
````cpp
#include <cute/tensor.hpp>
#include <random>
````
**EN:** This block pulls in dependencies required by the file, especially `cute`, so the later mixture-of-experts gemm flow code can use the needed APIs and data structures. It corresponds to block 6 of 70 in the file order.
**CN:** 这一段引入本文件所需的依赖，尤其是 `cute`，使后续MoE GEMM 流程代码可以使用相应 API 和数据结构。 它对应本文件顺序中的第 6/70 个代码块。

### Lines 46-48
````cpp
#include <cute/util/compat.hpp>
#include <sycl/ext/intel/experimental/grf_size_properties.hpp>
#include <sycl/sycl.hpp>
````
**EN:** This block pulls in dependencies required by the file, especially `sycl`, `cute`, so the later mixture-of-experts gemm flow code can use the needed APIs and data structures. It corresponds to block 7 of 70 in the file order.
**CN:** 这一段引入本文件所需的依赖，尤其是 `sycl`、`cute`，使后续MoE GEMM 流程代码可以使用相应 API 和数据结构。 它对应本文件顺序中的第 7/70 个代码块。

### Lines 50-50
````cpp
#include <cute/tensor.hpp>
````
**EN:** This block pulls in dependencies required by the file, especially `cute`, so the later mixture-of-experts gemm flow code can use the needed APIs and data structures. It corresponds to block 8 of 70 in the file order.
**CN:** 这一段引入本文件所需的依赖，尤其是 `cute`，使后续MoE GEMM 流程代码可以使用相应 API 和数据结构。 它对应本文件顺序中的第 8/70 个代码块。

### Lines 52-62
````cpp
#include "cutlass/kernel_hardware_info.h"
#include "cutlass/platform/platform.h"
#include "cutlass/tensor_ref.h"
#include "cutlass/util/command_line.h"
#include "cutlass/util/GPU_Clock.hpp"
#include "cutlass/util/device_memory.h"
#include "cutlass/util/initialize_block.hpp"
#include "cutlass/util/reference/device/gemm_complex.h"
#include "cutlass/util/reference/device/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/sycl_event_manager.hpp"
````
**EN:** This block pulls in dependencies required by the file, especially `sycl`, `reference`, so the later mixture-of-experts gemm flow code can use the needed APIs and data structures. It corresponds to block 9 of 70 in the file order.
**CN:** 这一段引入本文件所需的依赖，尤其是 `sycl`、`reference`，使后续MoE GEMM 流程代码可以使用相应 API 和数据结构。 它对应本文件顺序中的第 9/70 个代码块。

### Lines 64-65
````cpp
#include "moe_grouped_gemm.hpp"
#include "moe_tile_scheduler.hpp"
````
**EN:** This block pulls in dependencies required by the file, especially `include`, `moe_grouped_gemm`, `hpp`, `moe_tile_scheduler`, so the later mixture-of-experts gemm flow code can use the needed APIs and data structures. It corresponds to block 10 of 70 in the file order.
**CN:** 这一段引入本文件所需的依赖，尤其是 `include`、`moe_grouped_gemm`、`hpp`、`moe_tile_scheduler`，使后续MoE GEMM 流程代码可以使用相应 API 和数据结构。 它对应本文件顺序中的第 10/70 个代码块。

### Lines 67-68
````cpp
#pragma clang diagnostic ignored "-Wpass-failed"
#pragma clang diagnostic ignored "-Wdeprecated-declarations"
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `pragma`, `clang`, `diagnostic`, `ignored` showing the main symbols being prepared or consumed here. It corresponds to block 11 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `pragma`、`clang`、`diagnostic`、`ignored` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 11/70 个代码块。

### Lines 70-71
````cpp
using namespace cute;
using namespace MoE;
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `cute` make the later mixture-of-experts gemm flow code easier to assemble and read. It corresponds to block 12 of 70 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `cute` 这样的符号让后续MoE GEMM 流程代码更容易组装和阅读。 它对应本文件顺序中的第 12/70 个代码块。

### Lines 73-73
````cpp
using ElementAccumulator = float; // <- data type of accumulator
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `ElementAccumulator`, `data`, `type`, `of` make the later mixture-of-experts gemm flow code easier to assemble and read. It corresponds to block 13 of 70 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `ElementAccumulator`、`data`、`type`、`of` 这样的符号让后续MoE GEMM 流程代码更容易组装和阅读。 它对应本文件顺序中的第 13/70 个代码块。

### Lines 75-75
````cpp
///////////////////////////////////////////////////////////////////////////////////////////////////
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with the surrounding symbols showing the main symbols being prepared or consumed here. It corresponds to block 14 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 周围的符号 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 14/70 个代码块。

### Lines 77-78
````cpp
// Command line options parsing
struct Options {
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `Command`, `line`, `options`, `parsing` showing the main symbols being prepared or consumed here. It corresponds to block 15 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `Command`、`line`、`options`、`parsing` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 15/70 个代码块。

### Lines 80-81
````cpp
  bool help;
  bool error;
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `bool`, `help`, `error` showing the main symbols being prepared or consumed here. It corresponds to block 16 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `bool`、`help`、`error` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 16/70 个代码块。

### Lines 83-83
````cpp
  int n, k, num_layers, verify;
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `verify` showing the main symbols being prepared or consumed here. It corresponds to block 17 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `verify` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 17/70 个代码块。

### Lines 85-89
````cpp
  Options():
    help(false),
    error(false),
    n(2880), k(2880), num_layers(24), verify(1)
  { }
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `verify` showing the main symbols being prepared or consumed here. It corresponds to block 18 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `verify` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 18/70 个代码块。

### Lines 91-93
````cpp
  // Parses the command line
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `Parses`, `the`, `command`, `line` showing the main symbols being prepared or consumed here. It corresponds to block 19 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `Parses`、`the`、`command`、`line` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 19/70 个代码块。

### Lines 95-98
````cpp
    if (cmd.check_cmd_line_flag("help")) {
      help = true;
      return;
    }
````
**EN:** This block applies conditional control flow. It uses `cmd`, `check_cmd_line_flag`, `help`, `true` to select a path, validate assumptions, or handle special cases in the mixture-of-experts gemm flow implementation. It corresponds to block 20 of 70 in the file order.
**CN:** 这一段实现条件控制流。它借助 `cmd`、`check_cmd_line_flag`、`help`、`true` 在MoE GEMM 流程实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 20/70 个代码块。

### Lines 100-104
````cpp
    cmd.get_cmd_line_argument("n", n, 2880);
    cmd.get_cmd_line_argument("k", k, 2880);
    cmd.get_cmd_line_argument("num_layers", num_layers, 24);
    cmd.get_cmd_line_argument("verify", verify, 1);
  }
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `verify` showing the main symbols being prepared or consumed here. It corresponds to block 21 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `verify` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 21/70 个代码块。

### Lines 106-107
````cpp
  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `Prints`, `the`, `usage`, `statement` showing the main symbols being prepared or consumed here. It corresponds to block 22 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `Prints`、`the`、`usage`、`statement` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 22/70 个代码块。

### Lines 109-115
````cpp
    out << "MoE GEMM Example\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement\n\n"
      << "  --n=<int>                   Sets the N extent of the MoE GEMM (default: 2880)\n"
      << "  --k=<int>                   Sets the K extent of the MoE GEMM (default: 2880)\n"
      << "  --num_layers=<int>          Number of layers to test (default: 24)\n"
      << "  --verify=<int>              Specify whether to verify (0=no, 1=yes, default: 1)\n\n";
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `verify` showing the main symbols being prepared or consumed here. It corresponds to block 23 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `verify` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 23/70 个代码块。

### Lines 117-119
````cpp
    return out;
  }
};
````
**EN:** This block finalizes a local computation or status path. The use of `out` helps conclude the current stage cleanly before the next block. It corresponds to block 24 of 70 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `out`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 24/70 个代码块。

### Lines 121-121
````cpp
///////////////////////////////////////////////////////////////////////////////////////////////////
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with the surrounding symbols showing the main symbols being prepared or consumed here. It corresponds to block 25 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 周围的符号 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 25/70 个代码块。

### Lines 123-123
````cpp
struct VerificationHelper {
````
**EN:** This block declares a type-level building block for the file, with `VerificationHelper` indicating the configuration, traits, or storage policy used later. It corresponds to block 26 of 70 in the file order.
**CN:** 这一段声明了文件中的类型级构件，其中 `VerificationHelper` 表明后续会使用的配置、traits 或存储策略。 它对应本文件顺序中的第 26/70 个代码块。

### Lines 125-126
````cpp
  bool error = false;
  bool help = false;
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `bool`, `error`, `false`, `help` showing the main symbols being prepared or consumed here. It corresponds to block 27 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `bool`、`error`、`false`、`help` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 27/70 个代码块。

### Lines 128-134
````cpp
  float alpha = 1.f;
  float beta = 0.f;
  int iterations;
  int m = 0, n = 0, k = 0, groups;
  int *num_rows_per_expert = nullptr;
  std::vector<typename MoE::ProblemShape::UnderlyingProblemShape>
      problem_sizes_host;
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `Shape` showing the main symbols being prepared or consumed here. It corresponds to block 28 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `Shape` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 28/70 个代码块。

### Lines 136-137
````cpp
  VerificationHelper()
      : error(false), help(false), alpha(1.f), beta(0.f), iterations(100) {}
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `VerificationHelper`, `error`, `false`, `help` showing the main symbols being prepared or consumed here. It corresponds to block 29 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `VerificationHelper`、`error`、`false`、`help` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 29/70 个代码块。

### Lines 139-152
````cpp
  void parse(const int num_experts, const int *num_tokens_per_expert_host,
             int moe_n, int moe_k,
             const int *num_tokens_per_expert_device = nullptr) {
    n = moe_n;
    k = moe_k;
    groups = num_experts;
    iterations = 2;
    num_rows_per_expert = const_cast<int *>(num_tokens_per_expert_device);
    assert(groups > 0);
    problem_sizes_host.clear();
    problem_sizes_host.reserve(groups);
    for (int i = 0; i < groups; i++) {
      problem_sizes_host.push_back({num_tokens_per_expert_host[i], n, k});
      m += num_tokens_per_expert_host[i];
````
**EN:** This block introduces executable logic through a function or method. Here, `parse`, `num_experts`, `num_tokens_per_expert_host`, `moe_n` drive a concrete step in the file's mixture-of-experts gemm flow flow. It corresponds to block 30 of 70 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `parse`、`num_experts`、`num_tokens_per_expert_host`、`moe_n` 推动了本文件MoE GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 30/70 个代码块。

### Lines 153-154
````cpp
    }
  }
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with the surrounding symbols showing the main symbols being prepared or consumed here. It corresponds to block 31 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 周围的符号 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 31/70 个代码块。

### Lines 156-163
````cpp
  /// Compute performance in GFLOP/s
  std::tuple<double, double, double>
  gflops(double runtime_s,
         std::vector<typename MoE::ProblemShape::UnderlyingProblemShape>
             problem_sizes_host) const {
    // Number of real-valued multiply-adds
    uint64_t fmas = uint64_t();
    uint64_t bytes_loaded = 0;
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `Shape` showing the main symbols being prepared or consumed here. It corresponds to block 32 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `Shape` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 32/70 个代码块。

### Lines 165-178
````cpp
    for (auto const &problem : problem_sizes_host) {
      auto M = static_cast<uint64_t>(get<0>(problem));
      auto N = static_cast<uint64_t>(get<1>(problem));
      auto K = static_cast<uint64_t>(get<2>(problem));
      fmas += M * N * K;
      bytes_loaded +=
          /* sizeof(cutlass::bfloat16_t) */ 2 * (2 * M * N + N * K + M * K);
    }
    // Two flops per multiply-add
    uint64_t flop = uint64_t(2) * uint64_t(fmas);
    double gflop = double(flop) / double(1.0e9);
    double arithmetic_intensity = double(flop) / double(bytes_loaded);
    double peak_mwm_bw = 456.0;
    double gflops_attainable = std::min<double>(
````
**EN:** This block iterates over problem instances or data movement steps. The loop around `problem`, `problem_sizes_host`, `M`, `static_cast<uint64_t` advances the file toward execution, checking, or benchmarking. It corresponds to block 33 of 70 in the file order.
**CN:** 这一段围绕问题规模或数据移动步骤进行迭代。围绕 `problem`、`problem_sizes_host`、`M`、`static_cast<uint64_t` 的循环把文件推进到执行、校验或基准测试阶段。 它对应本文件顺序中的第 33/70 个代码块。

### Lines 179-186
````cpp
        117 * double(1.0e12),
        arithmetic_intensity * (peak_mwm_bw * 1024 * 1024 * 1024));
    double projected_time = flop / gflops_attainable;
    return std::make_tuple(gflop / runtime_s,
                           double(bytes_loaded) / 1024 / 1024 / 1024 /
                               runtime_s,
                           projected_time * 1000);
  }
````
**EN:** This block introduces executable logic through a function or method. Here, `double`, `arithmetic_intensity`, `peak_mwm_bw`, `projected_time` drive a concrete step in the file's mixture-of-experts gemm flow flow. It corresponds to block 34 of 70 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `double`、`arithmetic_intensity`、`peak_mwm_bw`、`projected_time` 推动了本文件MoE GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 34/70 个代码块。

### Lines 188-201
````cpp
  template <class ElementA, class ElementB, class ElementD,
            class = std::enable_if_t<
                is_any_of_v<ElementA, cute::bfloat16_t, cute::half_t> &&
                is_any_of_v<ElementB, cute::bfloat16_t, cute::half_t> &&
                is_any_of_v<ElementD, cute::bfloat16_t, cute::half_t>>>
  bool verify(const ElementA *activations, const ElementB *weights,
              ElementD *outputs) {
    cutlass::DeviceAllocation<ElementD> output_ref;
    cutlass::DeviceAllocation<ElementD> unused_c_matrix;
    output_ref.reset(m * n);
    unused_c_matrix.reset(m * n);
    using LayoutA = cutlass::layout::RowMajor;
    using LayoutB = cutlass::layout::RowMajor;
    using LayoutC = cutlass::layout::RowMajor;
````
**EN:** This block declares a type-level building block for the file, with `Layout`, `cute`, `verify` indicating the configuration, traits, or storage policy used later. It corresponds to block 35 of 70 in the file order.
**CN:** 这一段声明了文件中的类型级构件，其中 `Layout`、`cute`、`verify` 表明后续会使用的配置、traits 或存储策略。 它对应本文件顺序中的第 35/70 个代码块。

### Lines 202-215
````cpp
    using LayoutD = cutlass::layout::RowMajor;
    bool passed = true;
    // Verify against individual reference GEMMs
    int cumulative_sum = 0;
    for (int32_t i = 0; i < groups; ++i) {
      auto problem = problem_sizes_host.at(i);
      auto M = get<0>(problem);
      cutlass::TensorRef ref_A(activations + int64_t(cumulative_sum) * k,
                               LayoutA::packed({M, k}));
      cutlass::TensorRef ref_B(weights + int64_t(i) * n * k, LayoutB::packed({k, n}));
      cutlass::TensorRef ref_C(unused_c_matrix.get() + int64_t(cumulative_sum) * n,
                               LayoutC::packed({M, n}));
      cutlass::TensorRef ref_D(output_ref.get() + int64_t(cumulative_sum) * n,
                               LayoutD::packed({M, n}));
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `Layout`, `Tensor`, `TensorRef`, `reference` make the later mixture-of-experts gemm flow code easier to assemble and read. It corresponds to block 36 of 70 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `Layout`、`Tensor`、`TensorRef`、`reference` 这样的符号让后续MoE GEMM 流程代码更容易组装和阅读。 它对应本文件顺序中的第 36/70 个代码块。

### Lines 217-229
````cpp
      //
      // Compute reference output
      //
      cutlass::reference::device::GemmComplex(
          {M, n, k}, 1.0, ref_A, cutlass::ComplexTransform::kNone, ref_B,
          cutlass::ComplexTransform::kNone, 0.0, ref_C, ref_D,
          ElementAccumulator(0),
          1,     // batch_count
          M * k, // batch_stride_A
          k * n, // batch_stride_B
          M * n, // batch_stride_C
          M * n  // batch_stride_D
      );
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `reference` showing the main symbols being prepared or consumed here. It corresponds to block 37 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `reference` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 37/70 个代码块。

### Lines 231-232
````cpp
      // Wait for kernel to finish
      compat::wait();
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `Wait`, `kernel`, `to`, `finish` showing the main symbols being prepared or consumed here. It corresponds to block 38 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `Wait`、`kernel`、`to`、`finish` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 38/70 个代码块。

### Lines 234-247
````cpp
      // Check if output from CUTLASS kernel and reference kernel are equal or
      // not
      passed &= cutlass::reference::device::BlockCompareEqual(
          output_ref.get() + int64_t(cumulative_sum) * n, outputs + int64_t(cumulative_sum) * n,
          M * n);
      if (!passed) {
        break;
      }
      cumulative_sum += M;
    }
    return passed;
  }
};
///////////////////////////////////////////////////////////////////////////////////////////////////
````
**EN:** This block applies conditional control flow. It uses `reference` to select a path, validate assumptions, or handle special cases in the mixture-of-experts gemm flow implementation. It corresponds to block 39 of 70 in the file order.
**CN:** 这一段实现条件控制流。它借助 `reference` 在MoE GEMM 流程实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 39/70 个代码块。

### Lines 249-252
````cpp
template <class TA, class TB> auto choose_tiled_mma(TA *A, TB *B) {
  using TA_non_CV = cutlass::platform::remove_cv_t<TA>;
  using TB_non_CV = cutlass::platform::remove_cv_t<TB>;
  auto op = XE_DPAS_TT<8, float, TA_non_CV, TB_non_CV>{};
````
**EN:** This block declares a type-level building block for the file, with `TA`, `TB`, `choose_tiled_mma`, `A` indicating the configuration, traits, or storage policy used later. It corresponds to block 40 of 70 in the file order.
**CN:** 这一段声明了文件中的类型级构件，其中 `TA`、`TB`、`choose_tiled_mma`、`A` 表明后续会使用的配置、traits 或存储策略。 它对应本文件顺序中的第 40/70 个代码块。

### Lines 254-256
````cpp
  using WGTile = Shape<_256, _128, _32>; // 256x128 WG tile size
  using SGLayout =
      Layout<Shape<_8, _2, _1>, Stride<_2, _1, _0>>; // 8x2 SG tiling, n-major
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `Shape`, `Layout` make the later mixture-of-experts gemm flow code easier to assemble and read. It corresponds to block 41 of 70 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `Shape`、`Layout` 这样的符号让后续MoE GEMM 流程代码更容易组装和阅读。 它对应本文件顺序中的第 41/70 个代码块。

### Lines 258-259
````cpp
  using MMA = typename TiledMMAHelper<MMA_Atom<decltype(op)>, Layout<WGTile>,
                                      SGLayout>::TiledMMA;
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `Layout` make the later mixture-of-experts gemm flow code easier to assemble and read. It corresponds to block 42 of 70 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `Layout` 这样的符号让后续MoE GEMM 流程代码更容易组装和阅读。 它对应本文件顺序中的第 42/70 个代码块。

### Lines 261-262
````cpp
  return MMA{};
}
````
**EN:** This block finalizes a local computation or status path. The use of `MMA` helps conclude the current stage cleanly before the next block. It corresponds to block 43 of 70 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `MMA`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 43/70 个代码块。

### Lines 264-265
````cpp
// type tag to define a unique sycl kernel name
template <typename, typename, typename, char, char> class GemmCuteName;
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `sycl` showing the main symbols being prepared or consumed here. It corresponds to block 44 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `sycl` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 44/70 个代码块。

### Lines 267-280
````cpp
template <char layoutA, char layoutB, typename ElementA, typename ElementB,
          typename ElementS, typename ElementD>
void MoEGEMMLauncher(const ElementA *activations, const ElementB *weights,
                     const ElementS *scales, ElementD *outputs,
                     const int gemm_n, const int gemm_k,
                     const int *num_rows_per_expert_device,
                     const int *num_tokens_per_expert_host,
                     const int num_experts, const bool verify = true) {
  // Change device_id to another value if you are running on a machine with
  // multiple GPUs and wish to use a GPU other than that with device ID 0.
  // For example, in a framework, you could query device ID.
  int sm_count =
      cutlass::KernelHardwareInfo::query_device_multiprocessor_count(0);
  cutlass::KernelHardwareInfo hw_info{0, sm_count};
````
**EN:** This block declares a type-level building block for the file, with `verify` indicating the configuration, traits, or storage policy used later. It corresponds to block 45 of 70 in the file order.
**CN:** 这一段声明了文件中的类型级构件，其中 `verify` 表明后续会使用的配置、traits 或存储策略。 它对应本文件顺序中的第 45/70 个代码块。

### Lines 281-294
````cpp
  auto dummy_problem_shape = cute::Shape<int, int, int>{1, gemm_k, gemm_n};
  // The GroupedGEMM API requires creation of  a vector of ProblemShape objects
  // for each GEMM problem, which is used in the GroupedGEMM tile-scheduler. If
  // there are 32 groups, then a vector of 32 `ProblemShape` objects is created.
  // Since these would not be known at compile time for a framework, they would
  // have to be created at run-time instead. However, for MoEGEMM, I just
  // provide one dummy shape, and then the custom code in tile scheduler can
  // derive the shape of each GEMM problem.
  auto dummy_group_problem_shape =
      cutlass::gemm::GroupProblemShape<Shape<int, int, int>>{
          1, &dummy_problem_shape, nullptr};
  using TileShape = Shape<_256, _128, _32>;
  using ClusterShape = Shape<_1, _1, _1>;
  auto scheduler_params =
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `TileShape`, `ClusterShape`, `Shape`, `cute` showing the main symbols being prepared or consumed here. It corresponds to block 46 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `TileShape`、`ClusterShape`、`Shape`、`cute` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 46/70 个代码块。

### Lines 295-307
````cpp
      PersistentTileSchedulerXeMoE<ProblemShape>::to_underlying_arguments(
          dummy_group_problem_shape, TileShape{}, ClusterShape{}, hw_info,
          PersistentTileSchedulerXeMoE<ProblemShape>::Arguments{
              1, RasterOrderOptions::AlongN});
  auto group_distribution =
      PersistentTileSchedulerXeMoE<ProblemShape>::get_grid_shape(
          scheduler_params, dummy_group_problem_shape, TileShape{},
          ClusterShape{}, hw_info,
          PersistentTileSchedulerXeMoE<ProblemShape>::Arguments{
              1, RasterOrderOptions::AlongN});
  auto mma = choose_tiled_mma(activations, weights);
  auto MaxThreadsPerWorkgroup = size(mma);
  dim3 local_range{MaxThreadsPerWorkgroup, 1, 1};
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `TileShape`, `ClusterShape`, `Shape` showing the main symbols being prepared or consumed here. It corresponds to block 47 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `TileShape`、`ClusterShape`、`Shape` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 47/70 个代码块。

### Lines 309-313
````cpp
  sycl::range<3> local = {local_range.z, local_range.y, local_range.x};
  sycl::range<3> groups = {group_distribution.z, group_distribution.y,
                           group_distribution.x};
  sycl::range<3> global = {local[0] * groups[0], local[1] * groups[1],
                           local[2] * groups[2]};
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `sycl` showing the main symbols being prepared or consumed here. It corresponds to block 48 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `sycl` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 48/70 个代码块。

### Lines 315-316
````cpp
  namespace syclex = sycl::ext::oneapi::experimental;
  namespace intelex = sycl::ext::intel::experimental;
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `sycl` showing the main symbols being prepared or consumed here. It corresponds to block 49 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `sycl` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 49/70 个代码块。

### Lines 318-325
````cpp
  syclex::properties kernel_props{syclex::sub_group_size<16>,
#if (defined(SYCL_INTEL_TARGET) && (SYCL_INTEL_TARGET == 35))
                                  intelex::grf_size<512>
#else
                                  intelex::grf_size<256>
#endif
  };
  sycl::queue Q = compat::get_default_queue();
````
**EN:** This block applies conditional control flow. It uses `SYCL`, `sycl` to select a path, validate assumptions, or handle special cases in the mixture-of-experts gemm flow implementation. It corresponds to block 50 of 70 in the file order.
**CN:** 这一段实现条件控制流。它借助 `SYCL`、`sycl` 在MoE GEMM 流程实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 50/70 个代码块。

### Lines 327-340
````cpp
  GPU_Clock timer;
  timer.start();
  auto event = Q.parallel_for<
      GemmCuteName<ElementA, ElementB, ElementD, layoutA, layoutB>>(
      sycl::nd_range<3>(global, local), kernel_props, [=](auto) {
        // Can also use void for copy atoms.
        // In that case, they will be chosen automatically.
        MoE::MoEGEMM<XE_LOAD_2D<16, 32, 32, 16>,
                     XE_LOAD_2D_VNNI<16, 32, 16, 16>, XE_STORE_2D<16, 8, 32>,
                     'R', 'R', 'R'>(activations, weights, scales, outputs, mma,
                                    num_rows_per_expert_device, num_experts,
                                    gemm_n, gemm_k, scheduler_params);
      });
  EventManager::getInstance().addEvent(event);
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `sycl` showing the main symbols being prepared or consumed here. It corresponds to block 51 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `sycl` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 51/70 个代码块。

### Lines 341-343
````cpp
  Q.wait_and_throw();
  float cute_time = timer.seconds() * 1000;
  double cute_average_time = double(cute_time);
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `cute` showing the main symbols being prepared or consumed here. It corresponds to block 52 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `cute` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 52/70 个代码块。

### Lines 345-353
````cpp
  VerificationHelper helper;
  helper.parse(num_experts, num_tokens_per_expert_host, gemm_n, gemm_k);
  if (verify) {
    if (helper.verify(activations, weights, outputs) == false) {
      std::cout << "\n\nFailed accuracy verification :(\n\n";
    }
  }
  auto [gflops, mem_bw_util, projected_time] =
      helper.gflops(cute_average_time / 1000.0, helper.problem_sizes_host);
````
**EN:** This block applies conditional control flow. It uses `cute`, `verify` to select a path, validate assumptions, or handle special cases in the mixture-of-experts gemm flow implementation. It corresponds to block 53 of 70 in the file order.
**CN:** 这一段实现条件控制流。它借助 `cute`、`verify` 在MoE GEMM 流程实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 53/70 个代码块。

### Lines 355-366
````cpp
  std::cout << "  Problem Sizes" << std::endl;
  for (int32_t i = 0; i < num_experts; ++i) {
    std::cout << "    " << num_tokens_per_expert_host[i] << std::endl;
  }
  std::cout << "  N      : " << gemm_n << std::endl;
  std::cout << "  K      : " << gemm_k << std::endl;
  std::cout << "  Groups      : " << num_experts << std::endl;
  std::cout << "  Avg runtime : " << cute_average_time << " ms" << std::endl;
  std::cout << "  GFLOPS      : " << gflops << std::endl;
  std::cout << "  Memory BW utilization : " << mem_bw_util << "  GBPs"
            << std::endl;
}
````
**EN:** This block iterates over problem instances or data movement steps. The loop around `cute` advances the file toward execution, checking, or benchmarking. It corresponds to block 54 of 70 in the file order.
**CN:** 这一段围绕问题规模或数据移动步骤进行迭代。围绕 `cute` 的循环把文件推进到执行、校验或基准测试阶段。 它对应本文件顺序中的第 54/70 个代码块。

### Lines 368-374
````cpp
void launcher(int *M_per_expert, int N, int K, const int &num_experts, const bool verify = true) {
  int n_moe = N;
  int k_moe = K;
  int num_tokens_incl_duplicated = 0;
  for (int i = 0; i < num_experts; i++) {
    num_tokens_incl_duplicated += M_per_expert[i];
  }
````
**EN:** This block introduces executable logic through a function or method. Here, `verify` drive a concrete step in the file's mixture-of-experts gemm flow flow. It corresponds to block 55 of 70 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `verify` 推动了本文件MoE GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 55/70 个代码块。

### Lines 376-389
````cpp
  float M_occupancy = 0.f;
  float actual_num_units = 0.f;
  int total_num_M_tiles = 0;
  for (int i = 0; i < num_experts; i++) {
    total_num_M_tiles += (M_per_expert[i] + 255) / 256;
    actual_num_units += M_per_expert[i] / 256.0;
  }
  M_occupancy = actual_num_units / total_num_M_tiles;
  std::cout << "\n\n M-occupancy is " << M_occupancy << std::endl;
  cutlass::DeviceAllocation<int32_t> num_rows_per_expert_device;
  cutlass::DeviceAllocation<bfloat16_t> activations_data;
  cutlass::DeviceAllocation<bfloat16_t> weights_data;
  cutlass::DeviceAllocation<bfloat16_t> output_data;
  int64_t A_size = int64_t(num_tokens_incl_duplicated) * k_moe;
````
**EN:** This block iterates over problem instances or data movement steps. The loop around `M_occupancy`, `f`, `actual_num_units`, `total_num_M_tiles` advances the file toward execution, checking, or benchmarking. It corresponds to block 56 of 70 in the file order.
**CN:** 这一段围绕问题规模或数据移动步骤进行迭代。围绕 `M_occupancy`、`f`、`actual_num_units`、`total_num_M_tiles` 的循环把文件推进到执行、校验或基准测试阶段。 它对应本文件顺序中的第 56/70 个代码块。

### Lines 390-400
````cpp
  int64_t B_size = int64_t(num_experts) * n_moe * k_moe;
  int64_t D_size = int64_t(num_tokens_incl_duplicated) * n_moe;
  num_rows_per_expert_device.reset(num_experts);
  num_rows_per_expert_device.copy_from_host(M_per_expert);
  activations_data.reset(A_size);
  weights_data.reset(B_size);
  output_data.reset(D_size);
  uint64_t seed = 2023;
  initialize_block(activations_data, seed + 2023);
  initialize_block(weights_data, seed + 2022);
  initialize_block(output_data, seed + 2021);
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `int64_t`, `B_size`, `num_experts`, `n_moe` showing the main symbols being prepared or consumed here. It corresponds to block 57 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `int64_t`、`B_size`、`num_experts`、`n_moe` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 57/70 个代码块。

### Lines 402-406
````cpp
  MoEGEMMLauncher<'R', 'R'>(activations_data.get(), weights_data.get(),
                            static_cast<void *>(nullptr), output_data.get(),
                            n_moe, k_moe, num_rows_per_expert_device.get(),
                            M_per_expert, num_experts, verify);
}
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `verify` showing the main symbols being prepared or consumed here. It corresponds to block 58 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `verify` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 58/70 个代码块。

### Lines 408-410
````cpp
int main(int argc, const char **argv) {
  Options options;
  options.parse(argc, argv);
````
**EN:** This block introduces executable logic through a function or method. Here, `main`, `argc`, `char`, `argv` drive a concrete step in the file's mixture-of-experts gemm flow flow. It corresponds to block 59 of 70 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `main`、`argc`、`char`、`argv` 推动了本文件MoE GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 59/70 个代码块。

### Lines 412-415
````cpp
  if (options.help) {
    options.print_usage(std::cout) << std::endl;
    return 0;
  }
````
**EN:** This block applies conditional control flow. It uses `options`, `help`, `print_usage` to select a path, validate assumptions, or handle special cases in the mixture-of-experts gemm flow implementation. It corresponds to block 60 of 70 in the file order.
**CN:** 这一段实现条件控制流。它借助 `options`、`help`、`print_usage` 在MoE GEMM 流程实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 60/70 个代码块。

### Lines 417-418
````cpp
  constexpr int num_experts = 32;
  constexpr int max_layers = 24;
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `constexpr`, `num_experts`, `max_layers` showing the main symbols being prepared or consumed here. It corresponds to block 61 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `constexpr`、`num_experts`、`max_layers` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 61/70 个代码块。

### Lines 420-426
````cpp
  if (options.num_layers > max_layers) {
    std::cerr << "Error: num_layers (" << options.num_layers 
              << ") exceeds maximum supported layers (" << max_layers 
              << ")." << std::endl;
    std::cerr << "Aborting execution." << std::endl;
    return -1;
  }
````
**EN:** This block applies conditional control flow. It uses `options`, `num_layers`, `max_layers`, `Error` to select a path, validate assumptions, or handle special cases in the mixture-of-experts gemm flow implementation. It corresponds to block 62 of 70 in the file order.
**CN:** 这一段实现条件控制流。它借助 `options`、`num_layers`、`max_layers`、`Error` 在MoE GEMM 流程实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 62/70 个代码块。

### Lines 428-441
````cpp
  int total_rows_for_each_expert[max_layers][num_experts] = {
      {148, 231, 404, 180, 127, 244, 224, 244, 110, 617, 289,
       845, 191, 424, 30,  97,  57,  324, 62,  77,  75,  144,
       250, 287, 629, 370, 161, 101, 215, 113, 224, 35},
      {666, 214, 448, 87, 4,   28,  48,  13,  74,  40,  546,
       397, 487, 350, 26, 95,  517, 487, 295, 58,  637, 97,
       139, 33,  126, 15, 352, 311, 995, 193, 135, 135},
      {1016, 30,  36, 452, 469, 473, 232, 0,   493, 14,  954,
       6,    4,   6,  279, 3,   94,  106, 96,  48,  49,  113,
       142,  169, 75, 99,  25,  220, 249, 289, 4,   1803},
      {350, 229,  703, 154,  8,  64,  80, 339, 2,   56,  5,
       312, 1005, 29,  9,    11, 23,  0,  23,  431, 48,  129,
       496, 476,  8,   1234, 7,  130, 34, 58,  41,  1554},
      {39, 10,  6,    2,   110, 1,  894, 8,    53,  0,   275,
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with `total_rows_for_each_expert`, `max_layers`, `num_experts` showing the main symbols being prepared or consumed here. It corresponds to block 63 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 `total_rows_for_each_expert`、`max_layers`、`num_experts` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 63/70 个代码块。

### Lines 442-455
````cpp
       6,  506, 421,  700, 178, 0,  530, 1623, 15,  231, 74,
       6,  222, 1246, 116, 35,  20, 0,   6,    381, 334},
      {399, 5,  201, 6,   134, 93,  1748, 1,   51,  4,   38,
       336, 53, 88,  328, 724, 15,  388,  706, 52,  19,  55,
       52,  33, 623, 1,   222, 215, 69,   45,  308, 1036},
      {11,  8,   407, 571, 458, 275,  197, 211, 13,  564, 462,
       114, 15,  13,  132, 24,  514,  2,   71,  13,  694, 47,
       16,  203, 610, 40,  0,   1587, 66,  23,  196, 491},
      {0,  230, 116, 136, 315,  643, 6,   183, 37,  26,  960,
       1,  8,   258, 21,  1602, 213, 198, 6,   196, 455, 557,
       47, 282, 493, 18,  101,  11,  616, 45,  268, 0},
      {392, 305,  179, 14, 227, 98,  114, 39, 64,  1456, 465,
       0,   18,   372, 0,  0,   189, 257, 25, 290, 486,  0,
       12,  1534, 468, 4,  555, 35,  146, 0,  161, 143},
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with the surrounding symbols showing the main symbols being prepared or consumed here. It corresponds to block 64 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 周围的符号 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 64/70 个代码块。

### Lines 456-469
````cpp
      {4,  107, 20,   125, 236,  898,  0, 0,   375, 2,  125,
       0,  0,   1429, 36,  195,  1660, 0, 127, 454, 73, 358,
       47, 79,  32,   20,  1465, 0,    0, 6,   109, 66},
      {19, 0, 0,  0,    2, 1638, 75, 135, 392, 2,   1494, 3, 23,  5,  4,   58,
       0,  0, 71, 1285, 8, 441,  0,  145, 209, 408, 450,  2, 824, 13, 326, 16},
      {4,  2, 14,  0, 30,  206, 41, 131, 0,   429,  16, 895, 35, 21, 44,   128,
       12, 0, 417, 0, 838, 917, 42, 115, 109, 1759, 0,  36,  17, 0,  1790, 0},
      {6, 483, 241, 1327, 17,  11, 480, 9,  880, 58, 4,
       0, 61,  30,  16,   176, 9,  309, 26, 0,   0,  1882,
       4, 281, 475, 783,  197, 0,  19,  15, 6,   243},
      {370,  1222, 0, 6,   108, 929,  2, 7, 157, 348, 149, 106, 2, 5,  25, 33,
       1569, 8,    6, 106, 69,  1298, 0, 2, 529, 520, 0,   421, 0, 25, 26, 0},
      {59,  89,   0,  26,  25,  40,   1873, 141, 527, 371, 262,
       62,  16,   0,  127, 234, 1637, 64,   132, 8,   0,   7,
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with the surrounding symbols showing the main symbols being prepared or consumed here. It corresponds to block 65 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 周围的符号 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 65/70 个代码块。

### Lines 470-483
````cpp
       161, 1005, 22, 1,   49,  6,    83,   925, 80,  16},
      {269,  617, 30,  4,   90,  26,  0,   16,  154, 212, 21,
       269,  379, 174, 129, 32,  8,   121, 344, 15,  0,   591,
       1494, 6,   737, 50,  112, 856, 483, 25,  454, 330},
      {0,   98,  1488, 22,  73, 0,    0, 343, 77,  4,   0,
       612, 165, 268,  4,   10, 43,   0, 598, 271, 2,   73,
       185, 0,   112,  779, 24, 1626, 0, 0,   0,   1171},
      {0,   0,  0, 189, 266, 1743, 0, 462, 20,   7, 668, 310, 40, 0, 10,  236,
       423, 18, 0, 0,   0,   999,  0, 139, 1754, 8, 619, 3,   23, 0, 102, 9},
      {131, 1753, 0,   113,  24,  94, 2,    12,  108, 0,  0,
       252, 97,   0,   1319, 233, 93, 1254, 195, 152, 14, 413,
       4,   2,    220, 67,   20,  4,  34,   559, 837, 42},
      {55, 76, 0,  8,   0, 3,  1557, 975, 135, 271, 4,   0, 0,   666, 207, 152,
       5,  2,  97, 364, 0, 13, 1423, 771, 159, 31,  223, 0, 431, 7,   409, 4},
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with the surrounding symbols showing the main symbols being prepared or consumed here. It corresponds to block 66 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 周围的符号 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 66/70 个代码块。

### Lines 484-491
````cpp
      {4,  1026, 1799, 166, 694, 753, 0,  16, 0,  240, 1119, 19, 6,  0, 46, 659,
       10, 0,    112,  808, 181, 0,   28, 22, 90, 0,   176,  0,  37, 5, 10, 22},
      {44, 0, 4, 153, 299, 1357, 6, 23,   0,    12, 4,   419, 73, 24, 16,  24,
       1,  4, 4, 102, 16,  4,    0, 1953, 1850, 0,  908, 4,   0,  13, 708, 23},
      {6, 13, 123, 28, 197,  0, 202, 69,   0, 6,  0,  21, 1434, 1582, 11, 0, 6,
       0, 7,  190, 4,  1700, 6, 434, 1886, 0, 14, 28, 8,  30,   25,   18},
      {5,  27, 1442, 18, 0,  6, 0, 73,  6,    781, 0,  1915, 291, 649, 98,  4,
       33, 77, 6,    22, 73, 9, 8, 587, 1486, 32,  10, 244,  37,  0,   100, 9}};
````
**EN:** This block continues the file's mixture-of-experts gemm flow setup or compute path, with the surrounding symbols showing the main symbols being prepared or consumed here. It corresponds to block 67 of 70 in the file order.
**CN:** 这一段继续推进本文件的MoE GEMM 流程初始化或计算流程，其中 周围的符号 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 67/70 个代码块。

### Lines 493-493
````cpp
  int num_layers = std::min(options.num_layers, max_layers);
````
**EN:** This block introduces executable logic through a function or method. Here, `num_layers`, `options`, `max_layers` drive a concrete step in the file's mixture-of-experts gemm flow flow. It corresponds to block 68 of 70 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `num_layers`、`options`、`max_layers` 推动了本文件MoE GEMM 流程流程中的一个具体步骤。 它对应本文件顺序中的第 68/70 个代码块。

### Lines 495-498
````cpp
  for (int i = 0; i < num_layers; i++) {
    launcher(total_rows_for_each_expert[i], 2 * options.n, options.k, num_experts, options.verify);
    launcher(total_rows_for_each_expert[i], options.n, options.k, num_experts, options.verify);
  }
````
**EN:** This block iterates over problem instances or data movement steps. The loop around `verify` advances the file toward execution, checking, or benchmarking. It corresponds to block 69 of 70 in the file order.
**CN:** 这一段围绕问题规模或数据移动步骤进行迭代。围绕 `verify` 的循环把文件推进到执行、校验或基准测试阶段。 它对应本文件顺序中的第 69/70 个代码块。

### Lines 500-501
````cpp
  return 0;
}
````
**EN:** This block finalizes a local computation or status path. The use of the surrounding symbols helps conclude the current stage cleanly before the next block. It corresponds to block 70 of 70 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 周围的符号，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 70/70 个代码块。

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
- **Direct dependencies / 直接依赖:** `cutlass/util/GPU_Clock.hpp`, `cute/tensor.hpp`, `random`, `cute/util/compat.hpp`, `sycl/ext/intel/experimental/grf_size_properties.hpp`, `sycl/sycl.hpp`, `cute/tensor.hpp`, `cutlass/kernel_hardware_info.h`, `cutlass/platform/platform.h`, `cutlass/tensor_ref.h`, `cutlass/util/command_line.h`, `cutlass/util/GPU_Clock.hpp`, ...
- **Runtime expectations / 运行时依赖:** SYCL runtime / SYCL 运行时; CUTLASS/CUTE templates / CUTLASS/CUTE 模板
- **Build/context note / 构建说明:** The file relies on surrounding repository infrastructure, compile flags, and example utilities. / 该文件依赖仓库周边基础设施、编译选项以及示例辅助代码。
