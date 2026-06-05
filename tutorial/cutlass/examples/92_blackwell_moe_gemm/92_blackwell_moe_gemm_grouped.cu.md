# 92_blackwell_moe_gemm_grouped.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/92_blackwell_moe_gemm/92_blackwell_moe_gemm_grouped.cu`  
**Purpose / 用途**: Shows Blackwell grouped MoE GEMM for FP8 inputs, where each expert contributes its own GEMM shape and the device scheduler consumes a problem list instead of a uniform batch. / 展示用于 FP8 输入的 Blackwell grouped MoE GEMM：每个 expert 都有自己的 GEMM 形状，设备调度器消费的是问题列表，而不是统一 batch。

---

## Line-by-Line Analysis / 逐行分析

```cpp
/***************************************************************************************************
 * Copyright (c) 2025 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
  \brief Example of Blackwell MoE-style grouped GEMM implementation using TMA to load A and CPASYNC to load B.

  This example demonstrates an implementation of GEMM using mixed TMA+CPASYNC to load input matrices.
  In the decoding stage of Mixture of Experts (MoE) models, the number of tokens in different experts 
  can varies a lot, which requires frequently updates of TMA descriptors in TMA-based implementation.
  This examples uses CPASYNC to load activation (B) matrix to avoid the overhead of updating TMA descriptors.
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Relies on tma-style bulk movement or descriptor handling. Uses cp.async-style staged loads where that path is preferable to tma.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。依赖 TMA 风格的批量搬运或描述符处理。 在适合时使用 cp.async 风格分阶段加载，而不是 TMA。

---

```cpp
  Usage:
  $ ./examples/92_blackwell_moe_gemm/92_blackwell_moe_gemm_grouped
  --m=7168 --n=128 --k=512 --groups=8 --benchmark=benchmark.txt

*/

#include <iostream>
#include <fstream>

#include "cute/tensor.hpp"

#include "cutlass/cutlass.h"
#include "cutlass/tensor_ref.h"
#include "cutlass/epilogue/collective/default_epilogue.hpp"
#include "cutlass/epilogue/thread/linear_combination.h"
#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/gemm/collective/collective_builder.hpp"
#include "cutlass/epilogue/collective/collective_builder.hpp"
#include "cutlass/gemm/device/gemm_universal_adapter.h"
#include "cutlass/gemm/kernel/gemm_universal.hpp"
#include "cutlass/gemm/kernel/tile_scheduler.hpp"

#include "cutlass/util/command_line.h"
#include "cutlass/util/distribution.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/packed_stride.hpp"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/reference/device/gemm.h"
#include "cutlass/util/reference/device/tensor_compare.h"
#include "cutlass/util/reference/device/tensor_fill.h"

#include "helper.h"
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
using namespace cute;
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
/// Command line options parsing
struct Options {

  bool help;
  bool error;
  bool verification;

  int m, n, k, groups;

  int iterations;

  std::string benchmark_path;

  Options():
    help(false),
    error(false),
    verification(true),
    m(2048), n(2048), k(2048), groups(1),
    iterations(10)
  { }

  // Parses the command line
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);

    if (cmd.check_cmd_line_flag("help")) {
      help = true;
      return;
    }

    cmd.get_cmd_line_argument("m", m, 2048);
    cmd.get_cmd_line_argument("n", n, 2048);
    cmd.get_cmd_line_argument("k", k, 2048);
    cmd.get_cmd_line_argument("groups", groups, 1);
    cmd.get_cmd_line_argument("iterations", iterations, 10);
    cmd.get_cmd_line_argument("benchmark", benchmark_path);
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Is tuned for moe-style irregular expert workloads.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
    if (cmd.check_cmd_line_flag("no_verif")) {
      verification = false;
    }
  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << "92_blackwell_moe_gemm_grouped\n\n"
      << "  Blackwell MoE-style grouped GEMM implementation using TMA to load A and CPASYNC to load B\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement\n\n"
      << "  --m=<int>                   Sets the M extent of the GEMM\n"
      << "  --n=<int>                   Sets the N extent of the GEMM\n"
      << "  --k=<int>                   Sets the K extent of the GEMM\n"
      << "  --groups=<int>              Sets the groups extent (batch count) of the GEMM\n"
      << "  --iterations=<int>          Set the number of profiling iterations to perform\n"
      << "  --benchmark=<file>          Executes a benchmark problem size\n"
      << "  --no_verif                  Do not run verification kernels\n";

    return out;
  }
};
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Relies on tma-style bulk movement or descriptor handling. Uses cp.async-style staged loads where that path is preferable to tma.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。依赖 TMA 风格的批量搬运或描述符处理。 在适合时使用 cp.async 风格分阶段加载，而不是 TMA。

---

```cpp
/// Helper to initialize a block of device data
template <class Element>
bool initialize_block(
  cutlass::DeviceAllocation<Element>& block,
  uint64_t seed=2023) {

  Element scope_max, scope_min;
  int bits_input = cutlass::sizeof_bits<Element>::value;

  if (bits_input == 1) {
    scope_max = static_cast<Element>(2);
    scope_min = static_cast<Element>(0);
  }
  else if (bits_input <= 8) {
    scope_max = static_cast<Element>(2);
    scope_min = static_cast<Element>(-2);
  }
  else {
    scope_max = static_cast<Element>(8);
    scope_min = static_cast<Element>(-8);
  }

  cutlass::reference::device::BlockFillRandomUniform(
    block.get(), block.size(), seed, scope_max, scope_min, 0);

  return true;
}
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
template<
  // Type of kernel schedule to generate
  class MainloopScheduleType = cutlass::gemm::KernelMixedTmaCpAsyncWarpSpecialized1SmSm100,
  // Type of epilogue schedule to generate
  class EpilogueScheduleType = cutlass::epilogue::collective::EpilogueScheduleAuto,
  class ClusterShapeMNK = Shape<_1, _1, _1>
>
struct ExampleRunner {

  using LayoutA = cutlass::layout::RowMajor;
  using LayoutB = cutlass::layout::ColumnMajor;
  using LayoutC = cutlass::layout::ColumnMajor;
  using LayoutD = cutlass::layout::ColumnMajor;

  using ElementA = cutlass::float_e4m3_t;
  using ElementB = cutlass::float_e4m3_t;
  using ElementC = cutlass::half_t;
  using ElementD = cutlass::half_t;
  using ElementAccumulator = float;
  using ElementCompute = float;
  using ElementScalar = float;

  using MmaTileMNK    = Shape<_128,_16,Int<128 / sizeof(ElementA)>>;  // use tile size of N=16 to match real use cases (N is typically very small in decoding stage)

  // 16B alignment lets us use TMA
  static constexpr int AlignmentA = 128 / cutlass::sizeof_bits<ElementA>::value;
  static constexpr int AlignmentB = 128 / cutlass::sizeof_bits<ElementB>::value;
  static constexpr int AlignmentC = 128 / cutlass::sizeof_bits<ElementC>::value;
  static constexpr int AlignmentD = 128 / cutlass::sizeof_bits<ElementD>::value;

  using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
      cutlass::arch::Sm100, cutlass::arch::OpClassTensorOp,
      MmaTileMNK, ClusterShapeMNK,
      cutlass::epilogue::collective::EpilogueTileAuto,
      ElementAccumulator, ElementCompute,
      ElementC, LayoutC, AlignmentC,
      ElementD, LayoutD, AlignmentD,
      EpilogueScheduleType,
      cutlass::epilogue::fusion::LinearCombination<ElementC, ElementAccumulator>
    >::CollectiveOp;
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Targets blackwell sm100 execution paths. Selects tensor-core mma execution instead of a simt fallback.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。面向 Blackwell SM100 执行路径。 选择 Tensor Core MMA 执行，而不是 SIMT 回退路径。

---

```cpp
  using CollectiveMainloop = typename cutlass::gemm::collective::CollectiveBuilder<
      cutlass::arch::Sm100, cutlass::arch::OpClassTensorOp,
      ElementA, LayoutA, AlignmentA,
      ElementB, LayoutB, AlignmentB,
      ElementAccumulator,
      MmaTileMNK, ClusterShapeMNK,
      cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))>,
      MainloopScheduleType
    >::CollectiveOp;

  using ProblemShape = typename cutlass::gemm::MoEProblemShape<Shape<int,int,int>>; // <M,N,K> per group

  using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
      ProblemShape,
      CollectiveMainloop,
      CollectiveEpilogue
      //, cutlass::gemm::MoEScheduler
  >;

  using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;

  using StrideA = typename Gemm::GemmKernel::StrideA;
  using StrideB = typename Gemm::GemmKernel::StrideB;
  using StrideC = typename Gemm::GemmKernel::StrideC;
  using StrideD = typename Gemm::GemmKernel::StrideD;

  using LayoutTagA = cutlass::gemm::detail::StrideToLayoutTagA_t<StrideA>;
  using LayoutTagB = cutlass::gemm::detail::StrideToLayoutTagB_t<StrideB>;
  using LayoutTagC = cutlass::gemm::detail::StrideToLayoutTagC_t<StrideC>;
  using LayoutTagD = cutlass::gemm::detail::StrideToLayoutTagC_t<StrideD>;

  //
  // Data members
  //

  /// Initialization
  StrideC stride_C;
  StrideD stride_D;
  uint64_t seed = 0;
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Targets blackwell sm100 execution paths. Selects tensor-core mma execution instead of a simt fallback.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。面向 Blackwell SM100 执行路径。 选择 Tensor Core MMA 执行，而不是 SIMT 回退路径。

---

```cpp
  cutlass::DeviceAllocation<typename Gemm::ElementA> block_A;
  cutlass::DeviceAllocation<typename Gemm::ElementB> block_B;
  cutlass::DeviceAllocation<typename Gemm::ElementC> block_C;
  cutlass::DeviceAllocation<typename Gemm::ElementD> block_D;
  cutlass::DeviceAllocation<typename Gemm::ElementD> block_ref_D;

  cutlass::DeviceAllocation<int32_t> tokens_per_expert;

  std::vector<ProblemShape::UnderlyingProblemShape> problem_sizes_host;
  std::vector<int32_t> tokens_per_expert_host;
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
  //
  // Methods
  //

  bool verify(ProblemShape const& problem_size, float alpha, float beta) {
    auto [maxM, maxN, maxK] = problem_size.get_host_problem_shape(0); //gets max problem shape
    for (int i = 0; i < problem_size.num_groups; i++) {
      auto problem = problem_sizes_host.at(i);
      auto [M, N, K] = problem;
      printf("group [%d] : M = %d, N = %d, K = %d\n", i, M, N, K);

      cutlass::TensorRef ref_A(block_A.get() + size_t(1) * i * maxM * maxK, typename Gemm::LayoutA(maxK));
      cutlass::TensorRef ref_B(block_B.get() + size_t(1) * i * maxN * maxK, typename Gemm::LayoutB(maxK));
      cutlass::TensorRef ref_C(block_C.get() + size_t(1) * i * maxN * maxM, typename Gemm::LayoutC(maxM));
      cutlass::TensorRef ref_D(block_ref_D.get() + size_t(1) * i * maxN * maxM, typename Gemm::LayoutD(maxM));

      using DeviceGemmReference = cutlass::reference::device::Gemm<
        ElementA,
        LayoutA,
        ElementB,
        LayoutB,
        ElementC,
        LayoutC,
        ElementScalar,
        ElementAccumulator>;

      DeviceGemmReference gemm_reference;

      gemm_reference(
        {M, N, K},
        ElementScalar(alpha),
        ref_A,
        ref_B,
        ElementScalar(beta),
        ref_C,
        ref_D);
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
      cudaError_t result = cudaDeviceSynchronize();
      if (result != cudaSuccess) {
        std::cerr << "Reference kernel failed. Last CUDA error: "
                  << cudaGetErrorString(result) << std::endl;
        return false;
      }

      // Check if output from CUTLASS kernel and reference kernel are equal or not
      // assume all M == maxM
      bool passed = cutlass::reference::device::BlockCompareEqual(block_ref_D.get() + size_t(1) * i * maxN * maxM, block_D.get() + size_t(1) * i * maxN * maxM, M * N);
      if (!passed) {
        return false;
      }
    }

    return true;
  }

  /// Initialize operands to be used in the GEMM and reference GEMM
  void initialize(ProblemShape const& problem_size) {
    auto problem_shape_MNKL = cute::append<4>(problem_size.get_host_problem_shape(0), problem_size.groups());
    auto [M, N, K, L] = problem_shape_MNKL;

    stride_C = cutlass::make_cute_packed_stride(StrideC{}, cute::make_shape(M, N, L));
    stride_D = cutlass::make_cute_packed_stride(StrideD{}, cute::make_shape(M, N, L));

    block_A.reset(size_t(1) * M * K * L);
    block_B.reset(size_t(1) * K * N * L);
    block_C.reset(size_t(1) * M * N * L);
    block_D.reset(size_t(1) * M * N * L);
    block_ref_D.reset(size_t(1) * M * N * L);

    initialize_block(block_A, seed + 2023);
    initialize_block(block_B, seed + 2022);
    initialize_block(block_C, seed + 2021);
  }

  /// Load a benchmark
  void benchmark_problems(std::string const& benchmark_path) {
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
    std::ifstream file(benchmark_path);
    if (!file.good()) {
      std::cout << "Issues with benchmark file." << std::endl;
      return;
    }

    while (file.good()) {

      int idx = -1;
      std::string extent_str;

      file >> idx >> extent_str;

      if (idx < 0 || extent_str.empty()) {
        break;
      }

      cutlass::gemm::GemmCoord extent;
      std::vector<std::string> tokens;

      cutlass::CommandLine::tokenize(tokens, extent_str, 'x');

      for (int i = 0; i < int(tokens.size()); ++i) {
        extent.at(i) = std::atoi(tokens.at(i).c_str());
      }
      problem_sizes_host.push_back({extent.m(), extent.n(), extent.k()});
    }

    return;
  }

  void benchmark_tokens_per_expert(std::string const& benchmark_path) {

    std::ifstream file(benchmark_path);
    if (!file.good()) {
      return;
    }

    while (file.good()) {
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
      int idx = -1;
      std::string extent_str;

      file >> idx >> extent_str;

      if (idx < 0 || extent_str.empty()) {
        break;
      }

      cutlass::gemm::GemmCoord extent;
      std::vector<std::string> tokens;

      cutlass::CommandLine::tokenize(tokens, extent_str, 'x');

      for (int i = 0; i < int(tokens.size()); ++i) {
        extent.at(i) = std::atoi(tokens.at(i).c_str());
      }
      tokens_per_expert_host.push_back(extent.n());
    }

    return;
  }

  bool run(Options const& options, cutlass::KernelHardwareInfo const& hw_info) {

    benchmark_problems(options.benchmark_path);
    if (problem_sizes_host.empty()) {
      return false;
    }

    benchmark_tokens_per_expert(options.benchmark_path);
    if (tokens_per_expert_host.empty()) {
      return false;
    }

    tokens_per_expert.reset(tokens_per_expert_host.size());
    tokens_per_expert.copy_from_host(tokens_per_expert_host.data());

    ProblemShape problem_size {options.m, options.n, options.k, options.groups, tokens_per_expert.get()};

    initialize(problem_size);
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
    typename Gemm::Arguments arguments{
      cutlass::gemm::GemmUniversalMode::kGrouped,
      problem_size,
      {block_A.get(), block_B.get()},
      {{}, // epilogue.thread
       block_C.get(), stride_C, block_D.get(), stride_D},
      hw_info
    };

    // arguments.scheduler.max_swizzle_size = options.swizzle;
    
    arguments.epilogue.thread.alpha = 1.0f;
    arguments.epilogue.thread.beta = 0.0f;

    Gemm gemm_op;

    size_t workspace_size = Gemm::get_workspace_size(arguments);
    cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);

    cutlass::Status status = gemm_op.can_implement(arguments);
    if (status != cutlass::Status::kSuccess) {
      std::cerr << "This kernel is not supported. Last CUDA error is: "
                << cudaGetErrorString(cudaGetLastError()) << std::endl;
      return false;
    }

    status = gemm_op.initialize(arguments, workspace.get());
    if (status != cutlass::Status::kSuccess) {
      std::cerr << "Failed to initialize the CUTLASS kernel. Last CUDA error is: "
                << cudaGetErrorString(cudaGetLastError()) << std::endl;
      return false;
    }

    // Run the GEMM
    status = gemm_op.run();
    if (status != cutlass::Status::kSuccess) {
      std::cerr << "Failed to launch the CUTLASS kernel. Last CUDA error is: "
                << cudaGetErrorString(cudaGetLastError()) << std::endl;
      return false;
    }
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
    cudaError_t result = cudaDeviceSynchronize();
    if (result != cudaSuccess) {
      std::cerr << "Error running the CUTLASS kernel. Last CUDA error is: "
                << cudaGetErrorString(result) << std::endl;
      return false;
    }

    if (options.verification) {
      // Verify that the result is correct
      bool passed = verify(problem_size, 1.0f, 0.0f);

      std::cout << "  Disposition: " << (passed ? "Passed" : "Failed") << std::endl;

      if (!passed) {
        exit(-1);
        return false;
      }
    }

    // Run profiling loop
    if (options.iterations > 0)
    {
      GpuTimer timer;
      timer.start();
      for (int iter = 0; iter < options.iterations; ++iter) {
        CUTLASS_CHECK(gemm_op.initialize(arguments, workspace.get()));
        CUTLASS_CHECK(gemm_op.run());
      }
      timer.stop();

      // Compute average setup and runtime and FLOPs.
      float elapsed_ms       = timer.elapsed_millis();
      double avg_runtime_ms  = double(elapsed_ms) / double(options.iterations);
      double flops           = double(int64_t(2) * options.m * options.n * options.k * options.groups) / (avg_runtime_ms / 1000.0);

      std::cout << "  Avg runtime : " << avg_runtime_ms << " ms" << std::endl;
      std::cout << "  TFLOPS      : " << flops / 1e12 << std::endl;
    }

    return true;
  }
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
};
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
int main(int argc, char const **args) {

  cudaDeviceProp props;

  cudaError_t error = cudaGetDeviceProperties(&props, 0);
  if (error != cudaSuccess) {
    std::cerr << "cudaGetDeviceProperties() returned an error: " << cudaGetErrorString(error) << std::endl;
    return -1;
  }

  if (__CUDACC_VER_MAJOR__ < 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ < 8)) {
    std::cerr << "This example requires CUDA 12.8 or newer." << std::endl;
    // Returning zero so this test passes on older Toolkits. Its actions are no-op.
    return 0;
  }
  
  if (!(props.major == 10 && props.minor == 0)) {
    std::cerr << "This example requires a GPU of NVIDIA's Blackwell architecture (compute capability 100)." << std::endl;
    return 0;
  }

  //
  // Parse options
  //

  Options options;

  options.parse(argc, args);

  if (options.help) {
    options.print_usage(std::cout) << std::endl;
    return 0;
  }

  if (options.error) {
    std::cerr << "Aborting execution." << std::endl;
    return -1;
  }

#if defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)

  //
```

**EN**: This block forms the outer control flow of the sample. It performs capability checks, selects the concrete specialization to run, invokes the operation, and reports the final status or throughput back to the user. Targets blackwell sm100 execution paths. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一块构成了示例的最外层控制流：先做能力检查，再选择具体特化版本，调用 operation，最后把状态或吞吐结果回报给用户。面向 Blackwell SM100 执行路径。 针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
  // Run examples
  //

  // The KernelHardwareInfo struct holds the number of SMs on the GPU with a given device ID. This
  // information is used by the underlying kernel.
  cutlass::KernelHardwareInfo hw_info;

  // Change device_id to another value if you are running on a machine with multiple GPUs and wish
  // to use a GPU other than that with device ID 0.
  hw_info.device_id = 0;
  hw_info.sm_count = cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);

  std::cout << "Running kernel with mixed TMA+CPASYNC load, 1SM:" << std::endl;
  ExampleRunner runner_mixed_tma_cpasync;
  runner_mixed_tma_cpasync.run(options, hw_info);

  std::cout << "\n\n\nRunning kernel with mixed TMA+CPASYNC load and 1x1 cluster:" << std::endl;
  ExampleRunner<
    cutlass::gemm::KernelMixedTmaCpAsyncWarpSpecialized1SmSm100,
    cutlass::epilogue::collective::EpilogueScheduleAuto,
    Shape<_1, _1, _1>
  > runner_mixed_tma_cpasync_1sm;
  runner_mixed_tma_cpasync_1sm.run(options, hw_info);

  std::cout << "Running kernel with mixed TMA+CPASYNC load with 1SM instr, 2x2 cluster:" << std::endl;
  ExampleRunner<
    cutlass::gemm::KernelMixedTmaCpAsyncWarpSpecialized1SmSm100,
    cutlass::epilogue::collective::EpilogueScheduleAuto,
    Shape<_2, _2, _1>
  > runner_mixed_tma_cpasync_1sm_2x2;
  runner_mixed_tma_cpasync_1sm_2x2.run(options, hw_info);

  std::cout << "\n\n\nRunning kernel with mixed TMA+CPASYNC load and 4x4 cluster:" << std::endl;
  ExampleRunner<
    cutlass::gemm::KernelMixedTmaCpAsyncWarpSpecialized1SmSm100,
    cutlass::epilogue::collective::EpilogueScheduleAuto,
    Shape<_4, _4, _1>
  > runner_mixed_tma_cpasync_1sm_4x4;
  runner_mixed_tma_cpasync_1sm_4x4.run(options, hw_info);
```

**EN**: This block forms the outer control flow of the sample. It performs capability checks, selects the concrete specialization to run, invokes the operation, and reports the final status or throughput back to the user. Targets blackwell sm100 execution paths. Lets cutlass auto-select a schedule from the template metadata.  
**CN**: 这一块构成了示例的最外层控制流：先做能力检查，再选择具体特化版本，调用 operation，最后把状态或吞吐结果回报给用户。面向 Blackwell SM100 执行路径。 让 CUTLASS 根据模板元数据自动选择调度策略。

---

```cpp
  std::cout << "\n\n\nRunning 2SM kernel with mixed TMA+CPASYNC load 2x1:" << std::endl;
  ExampleRunner<cutlass::gemm::KernelMixedTmaCpAsyncWarpSpecialized2SmSm100,
    cutlass::epilogue::collective::EpilogueScheduleAuto,
    Shape<_2, _1, _1>
  > runner_mixed_tma_cpasync_2sm_2x1;
  runner_mixed_tma_cpasync_2sm_2x1.run(options, hw_info);

  std::cout << "\n\n\nRunning 2SM kernel with mixed TMA+CPASYNC load 4x1:" << std::endl;
  ExampleRunner<cutlass::gemm::KernelMixedTmaCpAsyncWarpSpecialized2SmSm100,
    cutlass::epilogue::collective::EpilogueScheduleAuto,
    Shape<_4, _1, _1>
  > runner_mixed_tma_cpasync_2sm_4x1;
  runner_mixed_tma_cpasync_2sm_4x1.run(options, hw_info);

  std::cout << "\n\n\nRunning 2SM kernel with mixed TMA+CPASYNC load 2x4:" << std::endl;
  ExampleRunner<cutlass::gemm::KernelMixedTmaCpAsyncWarpSpecialized2SmSm100,
    cutlass::epilogue::collective::EpilogueScheduleAuto,
    Shape<_2, _4, _1>
  > runner_mixed_tma_cpasync_2sm_2x4;
  runner_mixed_tma_cpasync_2sm_2x4.run(options, hw_info);

  std::cout << "\n\n\nRunning 2SM kernel with mixed TMA+CPASYNC load 4x4:" << std::endl;
  ExampleRunner<cutlass::gemm::KernelMixedTmaCpAsyncWarpSpecialized2SmSm100,
    cutlass::epilogue::collective::EpilogueScheduleAuto,
    Shape<_4, _4, _1>
  > runner_mixed_tma_cpasync_2sm_4x4;
  runner_mixed_tma_cpasync_2sm_4x4.run(options, hw_info);

#endif
```

**EN**: This block forms the outer control flow of the sample. It performs capability checks, selects the concrete specialization to run, invokes the operation, and reports the final status or throughput back to the user. Targets blackwell sm100 execution paths. Lets cutlass auto-select a schedule from the template metadata.  
**CN**: 这一块构成了示例的最外层控制流：先做能力检查，再选择具体特化版本，调用 operation，最后把状态或吞吐结果回报给用户。面向 Blackwell SM100 执行路径。 让 CUTLASS 根据模板元数据自动选择调度策略。

---

```cpp
  return 0;
}
```

**EN**: This block forms the outer control flow of the sample. It performs capability checks, selects the concrete specialization to run, invokes the operation, and reports the final status or throughput back to the user. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一块构成了示例的最外层控制流：先做能力检查，再选择具体特化版本，调用 operation，最后把状态或吞吐结果回报给用户。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

## Key Concepts / 关键概念

- Architecture-tagged specialization (`Sm100`/`Sm103`/`Sm120`) / 基于架构标签的特化（`Sm100`/`Sm103`/`Sm120`）
- CUTLASS collective builders that deduce mainloop and epilogue implementations / 推导 mainloop 与 epilogue 实现的 CUTLASS collective 构建器
- Cluster-shape tuning and possible multi-SM cooperation / Cluster 形状调优与可能的多 SM 协作
- FP8-family mixed-precision execution / FP8 家族混合精度执行
- TMA-driven data movement / 基于 TMA 的数据搬运
- cp.async staging for latency hiding / 用于隐藏延迟的 cp.async 分阶段加载

## Dependencies / 依赖项

- `iostream` — included dependency used by this example / 此示例使用的包含依赖
- `fstream` — included dependency used by this example / 此示例使用的包含依赖
- `cute/tensor.hpp` — CuTe tensor and layout primitives / CuTe 张量与布局原语
- `cutlass/cutlass.h` — core CUTLASS types, architecture tags, and utilities / CUTLASS 核心类型、架构标签与工具
- `cutlass/tensor_ref.h` — tensor reference wrappers / 张量引用封装
- `cutlass/epilogue/collective/default_epilogue.hpp` — epilogue collective implementation detail / 尾声 collective 实现细节
- `cutlass/epilogue/thread/linear_combination.h` — default linear-combination epilogue operator / 默认线性组合尾声算子
- `cutlass/gemm/dispatch_policy.hpp` — kernel scheduling and dispatch policy tags / 内核调度与分发策略标签
- `cutlass/gemm/collective/collective_builder.hpp` — GEMM collective mainloop builder / GEMM collective 主循环构建器
- `cutlass/epilogue/collective/collective_builder.hpp` — epilogue collective builder / 尾声 collective 构建器
- `cutlass/gemm/device/gemm_universal_adapter.h` — device-side universal GEMM launcher adapter / 设备侧通用 GEMM 启动适配器
- `cutlass/gemm/kernel/gemm_universal.hpp` — universal GEMM kernel composition / 通用 GEMM 内核组合
- `cutlass/gemm/kernel/tile_scheduler.hpp` — tile scheduling policies / tile 调度策略
- `cutlass/util/command_line.h` — command-line parsing helpers / 命令行解析辅助
- `cutlass/util/distribution.h` — random/distribution-based tensor initialization / 随机与分布初始化辅助
- `cutlass/util/host_tensor.h` — host/device tensor allocation utilities / host/device 张量分配工具
- `cutlass/util/packed_stride.hpp` — packed stride helpers / 紧凑 stride 辅助
- `cutlass/util/tensor_view_io.h` — tensor printing and inspection utilities / 张量打印与查看工具
- `cutlass/util/reference/device/gemm.h` — device reference GEMM / 设备参考 GEMM
- `cutlass/util/reference/device/tensor_compare.h` — device-side tensor comparison helpers / 设备侧张量比较辅助
- `...` — additional direct includes omitted for brevity / 其余直接包含头文件因篇幅省略
