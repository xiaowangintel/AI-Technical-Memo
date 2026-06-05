# 81_blackwell_gemm_groupwise.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/81_blackwell_gemm_blockwise/81_blackwell_gemm_groupwise.cu`  
**Purpose / 用途**: Shows how Blackwell SM100 reuses the blockwise GEMM structure while switching to coarse `1/128/128` scale granularity, so one scale value can cover larger M/N/K regions. / 展示 Blackwell SM100 如何在保留 blockwise GEMM 结构的同时，切换到更粗的 `1/128/128` 缩放粒度，让一个 scale 值覆盖更大的 M/N/K 区域。

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
    \brief An FP8 groupwise scaled GEMM example for the NVIDIA Blackwell SM100 architecture using CUTLASS.
*/

#include <iostream>

#include "cutlass/cutlass.h"
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Targets blackwell sm100 execution paths. Includes fp8/mxfp8-style narrow-precision behavior.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。面向 Blackwell SM100 执行路径。 包含 FP8/MXFP8 一类窄精度行为。

---

```cpp
#include "cute/tensor.hpp"
#include "cutlass/tensor_ref.h"
#include "cutlass/epilogue/thread/activation.h"
#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/gemm/collective/collective_builder.hpp"
#include "cutlass/epilogue/dispatch_policy.hpp"
#include "cutlass/epilogue/collective/collective_builder.hpp"
#include "cutlass/gemm/device/gemm_universal_adapter.h"
#include "cutlass/gemm/kernel/gemm_universal.hpp"
#include "cutlass/gemm/kernel/tile_scheduler_params.h"

#include "cutlass/util/command_line.h"
#include "cutlass/util/distribution.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/packed_stride.hpp"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_norm.h"

#include "cutlass/util/reference/host/gett.hpp"

#include "helper.h"

using namespace cute;
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。

---

```cpp
#if defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Targets blackwell sm100 execution paths.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。面向 Blackwell SM100 执行路径。

---

```cpp
/// GEMM kernel configurations

// A matrix configuration
using ElementA            = cutlass::float_e4m3_t;                          // Element type for A matrix operand
using LayoutA             = cutlass::layout::RowMajor;                      // Layout type for A matrix operand
constexpr int AlignmentA  = 128 / cutlass::sizeof_bits<ElementA>::value;    // Memory access granularity/alignment of A matrix in units of elements (up to 16 bytes)

// B matrix configuration
using ElementB            = cutlass::float_e4m3_t;                          // Element type for B matrix operand
using LayoutB             = cutlass::layout::ColumnMajor;                   // Layout type for B matrix operand
constexpr int AlignmentB  = 128 / cutlass::sizeof_bits<ElementB>::value;    // Memory access granularity/alignment of A matrix in units of elements (up to 16 bytes)

// C/D matrix configuration
using ElementC            = cutlass::float_e4m3_t;                          // Element type for C and D matrix operands
using LayoutC             = cutlass::layout::ColumnMajor;                   // Layout type for C and D matrix operands
constexpr int AlignmentC  = 128 / cutlass::sizeof_bits<ElementC>::value;    // Memory access granularity/alignment of A matrix in units of elements (up to 16 bytes)

using ElementD           = ElementC;
using LayoutD            = LayoutC;
constexpr int AlignmentD = AlignmentC;

// MMA type
using ElementAccumulator = float;                                           // Element Accumulator will also be our scale factor type
using ElementCompute = float;

// MMA and Cluster Tile Shapes
// Shape of the tile computed by tcgen05 MMA, could be across 2 SMs if Cluster Shape %2 == 0 
using MmaTileShape_MNK = Shape<_256,_128,_128>;                          
// Shape of the threadblocks in a cluster
using ClusterShape_MNK = Shape<_2,_1,_1>;
 
constexpr int ScaleGranularityM = 1;
constexpr int ScaleGranularityN = 128;
constexpr int ScaleGranularityK = 128;
using ScaleConfig = cutlass::detail::Sm100BlockwiseScaleConfig<ScaleGranularityM, ScaleGranularityN, ScaleGranularityK>;

// Note when we have multiple scale factors per tile (in this case 128 scales in M per tile), we will restrict up to a 
// 16B alignment if possible (i.e., we have at least 16B of scales in M).
// In this case the smallest M that can be executed is 16. To avoid this for smaller M, you can swap A and B
// and transpose A, B, C, and scales since B^T A^T = C^T.
using LayoutSFA             = decltype(ScaleConfig::deduce_layoutSFA());                     // Layout type for SFA matrix operand
using LayoutSFB             = decltype(ScaleConfig::deduce_layoutSFB());                     // Layout type for SFB matrix operand
```

**EN**: This configuration block defines compile-time aliases or constants that control data types, layouts, alignments, tile sizes, or enum-style modes. These choices are not cosmetic: in CUTLASS they steer which iterators, MMA atoms, and memory movement strategies become legal and efficient. Targets blackwell sm100 execution paths. Includes fp8/mxfp8-style narrow-precision behavior.  
**CN**: 这个配置块定义了编译期别名或常量，用来控制数据类型、布局、对齐、tile 大小或枚举模式。这些选择并非表面参数：在 CUTLASS 中，它们会直接决定哪些迭代器、MMA atom 和数据搬运策略既合法又高效。面向 Blackwell SM100 执行路径。 包含 FP8/MXFP8 一类窄精度行为。

---

```cpp
using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
    cutlass::arch::Sm100, cutlass::arch::OpClassTensorOp,
    MmaTileShape_MNK, ClusterShape_MNK,
    cutlass::epilogue::collective::EpilogueTileAuto,
    ElementAccumulator, ElementCompute,
    ElementC, LayoutC, AlignmentC,
    ElementD, LayoutC, AlignmentD,
    cutlass::epilogue::collective::EpilogueScheduleAuto
  >::CollectiveOp;

using CollectiveMainloop = typename cutlass::gemm::collective::CollectiveBuilder<
    cutlass::arch::Sm100, cutlass::arch::OpClassTensorOp,
    ElementA, cute::tuple<LayoutA, LayoutSFA>, AlignmentA,
    ElementB, cute::tuple<LayoutB, LayoutSFB>, AlignmentB,
    ElementAccumulator,
    MmaTileShape_MNK, ClusterShape_MNK,
    cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))>,
    cutlass::gemm::KernelScheduleSm100Blockwise
  >::CollectiveOp;

using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
    Shape<int,int,int,int>,
    CollectiveMainloop,
    CollectiveEpilogue,
    void>;                // Default to ClusterLaunchControl (CLC) based tile scheduler

using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;

using StrideA = typename Gemm::GemmKernel::StrideA;
using StrideB = typename Gemm::GemmKernel::StrideB;
using StrideC = typename Gemm::GemmKernel::StrideC;
using StrideD = typename Gemm::GemmKernel::StrideD;

/// Initialization
StrideA stride_A;
StrideB stride_B;
StrideC stride_C;
StrideD stride_D;
// Strides just iterate over scalars and have no zeros
LayoutSFA layout_SFA;
LayoutSFB layout_SFB;
// Layouts are tiled to the problem size and the strides have zeros
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Targets blackwell sm100 execution paths. Selects tensor-core mma execution instead of a simt fallback.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。面向 Blackwell SM100 执行路径。 选择 Tensor Core MMA 执行，而不是 SIMT 回退路径。

---

```cpp
uint64_t seed;

cutlass::HostTensor<ElementA          , LayoutA> tensor_A;
cutlass::HostTensor<ElementAccumulator, cutlass::layout::PackedVectorLayout> tensor_SFA;
cutlass::HostTensor<ElementB          , LayoutB> tensor_B;
cutlass::HostTensor<ElementAccumulator, cutlass::layout::PackedVectorLayout> tensor_SFB;
cutlass::HostTensor<ElementC          , LayoutC> tensor_C;
cutlass::HostTensor<ElementD          , LayoutD> tensor_D;
cutlass::HostTensor<ElementD          , LayoutD> tensor_ref_D;

#endif // defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Targets blackwell sm100 execution paths.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。面向 Blackwell SM100 执行路径。

---

```cpp
/// Testbed utility types
```

**EN**: This configuration block defines compile-time aliases or constants that control data types, layouts, alignments, tile sizes, or enum-style modes. These choices are not cosmetic: in CUTLASS they steer which iterators, MMA atoms, and memory movement strategies become legal and efficient.  
**CN**: 这个配置块定义了编译期别名或常量，用来控制数据类型、布局、对齐、tile 大小或枚举模式。这些选择并非表面参数：在 CUTLASS 中，它们会直接决定哪些迭代器、MMA atom 和数据搬运策略既合法又高效。

---

```cpp
// Command line options parsing
struct Options {

  bool help = false;
  bool skip_verification = false;

  float alpha = 1.f, beta = 0.f;
  int iterations = 1000;
  int m = 1024, n = 512, k = 1024, l = 1;

  // Parses the command line
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);

    if (cmd.check_cmd_line_flag("help")) {
      help = true;
      return;
    }

    if (cmd.check_cmd_line_flag("skip-verification")) {
      skip_verification = true;
    }

    cmd.get_cmd_line_argument("m", m);
    cmd.get_cmd_line_argument("n", n);
    cmd.get_cmd_line_argument("k", k);
    cmd.get_cmd_line_argument("l", l);
    cmd.get_cmd_line_argument("alpha", alpha, 1.f);
    cmd.get_cmd_line_argument("beta", beta, 0.f);
    cmd.get_cmd_line_argument("iterations", iterations);
  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。

---

```cpp
    out << "81_blackwell_gemm_groupwise\n\n"
      << "  Blackwell FP8 GEMM with Groupwise Scaling using a Warp Specialized kernel.\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement\n\n"
      << "  --m=<int>                   Sets the M extent of the GEMM\n"
      << "  --n=<int>                   Sets the N extent of the GEMM\n"
      << "  --k=<int>                   Sets the K extent of the GEMM\n"
      << "  --l=<int>                   Sets the l extent (batch) of the GEMM\n"
      << "  --alpha=<f32>               Epilogue scalar alpha\n"
      << "  --beta=<f32>                Epilogue scalar beta\n"
      << "  --iterations=<int>          Number of profiling iterations to perform.\n\n"
      << "  --skip-verification         Skip verification.\n\n";

    out
      << "\n\nExamples:\n\n"
      << "$ " << "81_blackwell_gemm_groupwise" << " --m=1024 --n=512 --k=1024 --alpha=2 --beta=0.707 \n\n";

    return out;
  }

  /// Compute performance in GFLOP/s
  double gflops(double runtime_s) const {
    // Two flops per multiply-add
    uint64_t flop = uint64_t(2) * m * n * k;
    double gflop = double(flop) / double(1.0e9);
    return gflop / runtime_s;
  }
};

/// Result structure
struct Result {
  double avg_runtime_ms;
  double gflops;
  cutlass::Status status;
  cudaError_t error;
  bool passed;
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Includes fp8/mxfp8-style narrow-precision behavior.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。包含 FP8/MXFP8 一类窄精度行为。

---

```cpp
  Result(
    double avg_runtime_ms = 0,
    double gflops = 0,
    cutlass::Status status = cutlass::Status::kSuccess,
    cudaError_t error = cudaSuccess)
  :
    avg_runtime_ms(avg_runtime_ms), gflops(gflops), status(status), error(error), passed(false)
  {}

};

#if defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Targets blackwell sm100 execution paths.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。面向 Blackwell SM100 执行路径。

---

```cpp
/// GEMM setup and evaluation
```

**EN**: This configuration block defines compile-time aliases or constants that control data types, layouts, alignments, tile sizes, or enum-style modes. These choices are not cosmetic: in CUTLASS they steer which iterators, MMA atoms, and memory movement strategies become legal and efficient.  
**CN**: 这个配置块定义了编译期别名或常量，用来控制数据类型、布局、对齐、tile 大小或枚举模式。这些选择并非表面参数：在 CUTLASS 中，它们会直接决定哪些迭代器、MMA atom 和数据搬运策略既合法又高效。

---

```cpp
/// Helper to initialize a block of device data
template <typename Element, typename Layout>
bool initialize_tensor(
  cutlass::TensorView<Element, Layout> view,
  cutlass::Distribution::Kind dist_kind,
  uint64_t seed) {

  if (dist_kind == cutlass::Distribution::Uniform) {

    double scope_max, scope_min;
    int bits_input = cutlass::sizeof_bits<Element>::value;
    int bits_output = cutlass::sizeof_bits<Element>::value;

    if (bits_input == 1) {
      scope_max = 2;
      scope_min = 0;
    } 
    else if (bits_input <= 8) {
      scope_max = 2;
      scope_min = -2;
    } 
    else if (bits_output == 16) {
      scope_max = 5;
      scope_min = -5;
    } 
    else {
      scope_max = 8;
      scope_min = -8;
    }

    cutlass::reference::host::TensorFillRandomUniform(
      view, seed, scope_max, scope_min, 0);
  }
  else if (dist_kind == cutlass::Distribution::AllZeros) {
    cutlass::reference::host::TensorFill(view);
  }
  else if (dist_kind == cutlass::Distribution::Identity) {

    cutlass::reference::host::TensorFillIdentity(view);
  }
  else if (dist_kind == cutlass::Distribution::Gaussian) {
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。

---

```cpp
    cutlass::reference::host::TensorFillRandomGaussian(view, seed, 0, 0.5);
  }
  else if (dist_kind == cutlass::Distribution::Sequential) {
    cutlass::reference::host::BlockFillSequential(view.data(), view.capacity());
  }
  else {
    throw std::runtime_error("Not implementated.");
  }

  return true;
}

/// Helper to initialize a block of device data (scale_tensors)
template <typename Element, typename Layout>
bool initialize_scale_tensor(
  cutlass::TensorView<Element, Layout> view,
  cutlass::Distribution::Kind dist_kind,
  uint64_t seed) {

  if (dist_kind == cutlass::Distribution::Uniform) {

    double scope_max, scope_min;

    scope_min = -8;
    scope_max = 8;

    cutlass::reference::host::TensorFillRandomUniform(
      view, seed, scope_max, scope_min, 0);
  }
  else if (dist_kind == cutlass::Distribution::AllZeros) {
    cutlass::reference::host::TensorFill(view);
  }
  else if (dist_kind == cutlass::Distribution::Identity) {

    cutlass::reference::host::TensorFillIdentity(view);
  }
  else if (dist_kind == cutlass::Distribution::Gaussian) {
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。

---

```cpp
    cutlass::reference::host::TensorFillRandomGaussian(view, seed, 0, 0.5);
  }
  else if (dist_kind == cutlass::Distribution::Sequential) {
    cutlass::reference::host::BlockFillSequential(view.data(), view.capacity());
  }
  else {
    throw std::runtime_error("Not implementated.");
  }

  return true;
}

/// Initialize operands to be used in the GEMM and reference GEMM
void initialize(const Options &options) {
  using namespace cute;

  auto gemm_problem_shape = cute::make_shape(options.m, options.n, options.k);

  stride_A = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(options.m, options.k, options.l));
  stride_B = cutlass::make_cute_packed_stride(StrideB{}, cute::make_shape(options.n, options.k, options.l));
  stride_C = cutlass::make_cute_packed_stride(StrideC{}, cute::make_shape(options.m, options.n, options.l));
  stride_D = cutlass::make_cute_packed_stride(StrideD{}, cute::make_shape(options.m, options.n, options.l));

  layout_SFA = ScaleConfig::tile_atom_to_shape_SFA(make_shape(options.m, options.n, options.k, options.l));
  layout_SFB = ScaleConfig::tile_atom_to_shape_SFB(make_shape(options.m, options.n, options.k, options.l));

  auto a_coord = cutlass::make_Coord(options.m * options.l, options.k);
  auto c_coord = cutlass::make_Coord(options.m * options.l, options.n);
  auto b_coord = cutlass::make_Coord(options.k, options.n * options.l);
  auto blockscale_a_coord = cutlass::make_Coord(size(filter_zeros(layout_SFA)));
  auto blockscale_b_coord = cutlass::make_Coord(size(filter_zeros(layout_SFB)));

  tensor_A.resize(a_coord);
  tensor_B.resize(b_coord);
  tensor_C.resize(c_coord);
  tensor_D.resize(c_coord);
  tensor_ref_D.resize(c_coord);
  tensor_SFA.resize(blockscale_a_coord);
  tensor_SFB.resize(blockscale_b_coord);
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。

---

```cpp
  initialize_tensor(tensor_A.host_view(), cutlass::Distribution::Uniform, seed + 2022);
  initialize_tensor(tensor_B.host_view(), cutlass::Distribution::Uniform, seed + 2023);
  initialize_tensor(tensor_C.host_view(), cutlass::Distribution::Uniform, seed + 2024);

  initialize_scale_tensor(tensor_SFA.host_view(), cutlass::Distribution::Uniform, seed + 2025);
  initialize_scale_tensor(tensor_SFB.host_view(), cutlass::Distribution::Uniform, seed + 2026);

  tensor_A.sync_device();
  tensor_B.sync_device();
  tensor_C.sync_device();
  tensor_D.sync_device();

  tensor_SFA.sync_device();
  tensor_SFB.sync_device();

}

/// Populates a Gemm::Arguments structure from the given commandline options
typename Gemm::Arguments args_from_options(const Options &options) {
  typename Gemm::Arguments arguments {
    cutlass::gemm::GemmUniversalMode::kGemm,
    {options.m, options.n, options.k, options.l},
    {tensor_A.device_data(), stride_A, 
     tensor_B.device_data(), stride_B,
     tensor_SFA.device_data(), layout_SFA,
     tensor_SFB.device_data(), layout_SFB},
    {
      {}, // epilogue.thread
      tensor_C.device_data(), stride_C,
      tensor_D.device_data(), stride_D
    }
  };

  auto &fusion_args = arguments.epilogue.thread;
  fusion_args.alpha = options.alpha;
  fusion_args.beta = options.beta;

  return arguments;
}

bool verify(const Options &options) {
  //
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。

---

```cpp
  // Compute reference output
  //

  // Create instantiation for device reference gemm kernel
  auto A = cute::make_tensor(tensor_A.host_data(),
      cute::make_layout(cute::make_shape(options.m, options.k, options.l), stride_A));
  auto B = cute::make_tensor(tensor_B.host_data(),
      cute::make_layout(cute::make_shape(options.n, options.k, options.l), stride_B));
  auto C = cute::make_tensor(tensor_C.host_data(),
      cute::make_layout(cute::make_shape(options.m, options.n, options.l), stride_C));
  auto D = cute::make_tensor(tensor_ref_D.host_data(),
      cute::make_layout(cute::make_shape(options.m, options.n, options.l), stride_D));
  auto SFA = cute::make_tensor(tensor_SFA.host_data(), layout_SFA);
  auto SFB = cute::make_tensor(tensor_SFB.host_data(), layout_SFB);

  using unused_t = decltype(D);

  cutlass::reference::host::GettBlockScalingMainloopParams<
      ElementAccumulator,
      decltype(A), 
      decltype(SFA), 
      decltype(B),
      decltype(SFB)
    > mainloop_params{A, SFA, B, SFB};

  cutlass::reference::host::GettEpilogueParams<
      ElementAccumulator,
      ElementAccumulator,
      ElementAccumulator,
      ElementCompute,
      decltype(C),
      decltype(D)
  > epilogue_params;

  epilogue_params.C = C;
  epilogue_params.D = D;
  epilogue_params.alpha = options.alpha;
  epilogue_params.beta = options.beta;

  // get reference result
  cutlass::reference::host::Gemm3x(mainloop_params, epilogue_params);
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。

---

```cpp
  // compare_reference
  tensor_D.sync_host();
  bool passed = cutlass::reference::host::TensorEquals(tensor_ref_D.host_view(), tensor_D.host_view());

  return passed;
}

/// Execute a given example GEMM computation
template <typename Gemm>
int run(Options &options) {
  initialize(options);
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。

---

```cpp
  // Instantiate CUTLASS kernel depending on templates
  Gemm gemm;

  // Create a structure of gemm kernel arguments suitable for invoking an instance of Gemm
  auto arguments = args_from_options(options);

  // Using the arguments, query for extra workspace required for matrix multiplication computation
  size_t workspace_size = Gemm::get_workspace_size(arguments);

  // Allocate workspace memory
  cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);

  // Check if the problem size is supported or not
  CUTLASS_CHECK(gemm.can_implement(arguments));
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。

---

```cpp
  // Initialize CUTLASS kernel with arguments and workspace pointer
  CUTLASS_CHECK(gemm.initialize(arguments, workspace.get()));
```

**EN**: This block prepares tensors, layouts, strides, seeds, or auxiliary buffers before launch. Although it lives on the host side, it mirrors device expectations closely so the runtime data layout matches the compile-time kernel configuration.  
**CN**: 这一块在启动前准备张量、布局、stride、随机种子或辅助缓冲。虽然它运行在主机侧，但会严格映射设备端预期，从而保证运行期数据布局与编译期内核配置一致。

---

```cpp
  // Correctness / Warmup iteration
  CUTLASS_CHECK(gemm.run());

  Result result;
  if (!options.skip_verification) {
    // Check if output from CUTLASS kernel and reference kernel are equal or not
    result.passed = verify(options);

    std::cout << "  Disposition: " << (result.passed ? "Passed" : "Failed") << std::endl;

    if (!result.passed) {
      exit(-1);
    }
  }

  // Run profiling loop
  if (options.iterations > 0) {
    GpuTimer timer;
    timer.start();
    for (int iter = 0; iter < options.iterations; ++iter) {
      CUTLASS_CHECK(gemm.run());
    }
    timer.stop();

    // Compute average runtime and GFLOPs.
    float elapsed_ms = timer.elapsed_millis();
    result.avg_runtime_ms = double(elapsed_ms) / double(options.iterations);
    result.gflops = options.gflops(result.avg_runtime_ms / 1000.0);

    std::cout << "  Problem Size: " << options.m << 'x' << options.n << 'x' << options.k << 'x' << options.l << std::endl;
    std::cout << "  Avg runtime: " << result.avg_runtime_ms << " ms" << std::endl;
    std::cout << "  GFLOPS: " << result.gflops << std::endl;
  }

  return 0;
}

#endif // defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Targets blackwell sm100 execution paths.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。面向 Blackwell SM100 执行路径。

---

```cpp
int main(int argc, char const **args) {

  // CUTLASS must be compiled with CUDA 12.0 Toolkit to run this example
  // and must have compute capability at least sm100a.
  
  if (__CUDACC_VER_MAJOR__ < 12) {
    std::cerr << "This example requires CUDA 12 or newer.\n";
    // Returning zero so this test passes on older Toolkits. Its actions are no-op.
    return 0;
  }

  cudaDeviceProp props;
  int current_device_id;
  CUDA_CHECK(cudaGetDevice(&current_device_id));
  CUDA_CHECK(cudaGetDeviceProperties(&props, current_device_id));
  cudaError_t error = cudaGetDeviceProperties(&props, 0);
  if (props.major != 10 || props.minor != 0) {
    std::cerr << "This example requires a GPU with compute capability 100a)." << std::endl;
    return 0;
  }
```

**EN**: This block forms the outer control flow of the sample. It performs capability checks, selects the concrete specialization to run, invokes the operation, and reports the final status or throughput back to the user. Targets blackwell sm100 execution paths.  
**CN**: 这一块构成了示例的最外层控制流：先做能力检查，再选择具体特化版本，调用 operation，最后把状态或吞吐结果回报给用户。面向 Blackwell SM100 执行路径。

---

```cpp
  //
  // Parse options
  //

  Options options;

  options.parse(argc, args);

  if (options.help) {
    options.print_usage(std::cout) << std::endl;
    return 0;
  }

  //
  // Run
  //
#if defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)
  run<Gemm>(options);
#endif // defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)

  return 0;
}
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Targets blackwell sm100 execution paths.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。面向 Blackwell SM100 执行路径。

---

## Key Concepts / 关键概念

- Architecture-tagged specialization (`Sm100`/`Sm103`/`Sm120`) / 基于架构标签的特化（`Sm100`/`Sm103`/`Sm120`）
- CUTLASS collective builders that deduce mainloop and epilogue implementations / 推导 mainloop 与 epilogue 实现的 CUTLASS collective 构建器
- Cluster-shape tuning and possible multi-SM cooperation / Cluster 形状调优与可能的多 SM 协作
- FP8-family mixed-precision execution / FP8 家族混合精度执行
- Reference-based numerical validation / 基于参考实现的数值验证

## Dependencies / 依赖项

- `iostream` — included dependency used by this example / 此示例使用的包含依赖
- `cutlass/cutlass.h` — core CUTLASS types, architecture tags, and utilities / CUTLASS 核心类型、架构标签与工具
- `cute/tensor.hpp` — CuTe tensor and layout primitives / CuTe 张量与布局原语
- `cutlass/tensor_ref.h` — tensor reference wrappers / 张量引用封装
- `cutlass/epilogue/thread/activation.h` — CUTLASS core component used by this example / 此示例使用的 CUTLASS 核心组件
- `cutlass/gemm/dispatch_policy.hpp` — kernel scheduling and dispatch policy tags / 内核调度与分发策略标签
- `cutlass/gemm/collective/collective_builder.hpp` — GEMM collective mainloop builder / GEMM collective 主循环构建器
- `cutlass/epilogue/dispatch_policy.hpp` — CUTLASS core component used by this example / 此示例使用的 CUTLASS 核心组件
- `cutlass/epilogue/collective/collective_builder.hpp` — epilogue collective builder / 尾声 collective 构建器
- `cutlass/gemm/device/gemm_universal_adapter.h` — device-side universal GEMM launcher adapter / 设备侧通用 GEMM 启动适配器
- `cutlass/gemm/kernel/gemm_universal.hpp` — universal GEMM kernel composition / 通用 GEMM 内核组合
- `cutlass/gemm/kernel/tile_scheduler_params.h` — tile scheduler runtime parameters / tile 调度器运行参数
- `cutlass/util/command_line.h` — command-line parsing helpers / 命令行解析辅助
- `cutlass/util/distribution.h` — random/distribution-based tensor initialization / 随机与分布初始化辅助
- `cutlass/util/host_tensor.h` — host/device tensor allocation utilities / host/device 张量分配工具
- `cutlass/util/packed_stride.hpp` — packed stride helpers / 紧凑 stride 辅助
- `cutlass/util/tensor_view_io.h` — tensor printing and inspection utilities / 张量打印与查看工具
- `cutlass/util/reference/host/tensor_fill.h` — host tensor fill helpers / 主机张量填充辅助
- `cutlass/util/reference/host/tensor_copy.h` — reference implementation or comparison helper / 参考实现或比较辅助
- `cutlass/util/reference/host/tensor_compare.h` — host tensor comparison helpers / 主机张量比较辅助
- `...` — additional direct includes omitted for brevity / 其余直接包含头文件因篇幅省略
