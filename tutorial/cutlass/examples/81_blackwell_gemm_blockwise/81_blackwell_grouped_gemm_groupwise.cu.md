# 81_blackwell_grouped_gemm_groupwise.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/81_blackwell_gemm_blockwise/81_blackwell_grouped_gemm_groupwise.cu`  
**Purpose / 用途**: Combines grouped execution with coarse `1/128/128` groupwise scaling, showing how Blackwell SM100 handles both per-problem pointer indirection and per-problem scale-layout broadcasting. / 把 grouped 执行与粗粒度 `1/128/128` 的 groupwise 缩放结合起来，展示 Blackwell SM100 如何同时处理“按问题指针间接层”和“按问题 scale 布局广播”。

---

## Line-by-Line Analysis / 逐行分析

```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
    \brief An FP8 blockwise-scaled grouped GEMM example for the NVIDIA Blackwell SM100 architecture using CUTLASS.
    In this example M, N, and K are fixed across groups.
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

#if defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Targets blackwell sm100 execution paths.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。面向 Blackwell SM100 执行路径。

---

```cpp
using ProblemShape = cutlass::gemm::GroupProblemShape<Shape<int,int,int>>; // <M,N,K> per group
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。

---

```cpp
/// GEMM kernel configurations
```

**EN**: This configuration block defines compile-time aliases or constants that control data types, layouts, alignments, tile sizes, or enum-style modes. These choices are not cosmetic: in CUTLASS they steer which iterators, MMA atoms, and memory movement strategies become legal and efficient.  
**CN**: 这个配置块定义了编译期别名或常量，用来控制数据类型、布局、对齐、tile 大小或枚举模式。这些选择并非表面参数：在 CUTLASS 中，它们会直接决定哪些迭代器、MMA atom 和数据搬运策略既合法又高效。

---

```cpp
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
using ElementAccumulator = float;
using ElementCompute = float;

// MMA and Cluster Tile Shapes
// Shape of the tile computed by tcgen05 MMA, could be across 2 SMs if Cluster Shape %2 == 0 
using MmaTileShape_MNK = Shape<_256,_128,_128>;                          
// Shape of the threadblocks in a cluster
using ClusterShape_MNK = Shape<_2,_1,_1>;
// Shape of the threadblocks participating in a tcgen05 MMA. <1, 1, 1> for cta_group = 1, <2, 1, 1> for cta_group = 2

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

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Targets blackwell sm100 execution paths. Includes fp8/mxfp8-style narrow-precision behavior.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。面向 Blackwell SM100 执行路径。 包含 FP8/MXFP8 一类窄精度行为。

---

```cpp
using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
    cutlass::arch::Sm100, cutlass::arch::OpClassTensorOp,
    MmaTileShape_MNK, ClusterShape_MNK,
    cutlass::epilogue::collective::EpilogueTileAuto,
    ElementAccumulator, ElementCompute,
    ElementC, LayoutC *, AlignmentC,
    ElementD, LayoutC *, AlignmentD,
    cutlass::epilogue::PtrArrayTmaWarpSpecialized2Sm
  >::CollectiveOp;

using CollectiveMainloop = typename cutlass::gemm::collective::CollectiveBuilder<
    cutlass::arch::Sm100, cutlass::arch::OpClassTensorOp,
    ElementA, cute::tuple<LayoutA *, LayoutSFA *>, AlignmentA,
    ElementB, cute::tuple<LayoutB *, LayoutSFB *>, AlignmentB,
    ElementAccumulator,
    MmaTileShape_MNK, ClusterShape_MNK,
    cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))>,
    cutlass::gemm::KernelPtrArrayTmaWarpSpecializedBlockwise2SmSm100
  >::CollectiveOp;

using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
    ProblemShape,
    CollectiveMainloop,
    CollectiveEpilogue,
    void>;                // Default to ClusterLaunchControl (CLC) based tile scheduler

using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;

using StrideA = typename Gemm::GemmKernel::InternalStrideA;
using StrideB = typename Gemm::GemmKernel::InternalStrideB;
using StrideC = typename Gemm::GemmKernel::InternalStrideC;
using StrideD = typename Gemm::GemmKernel::InternalStrideD;

static_assert(cute::is_same_v<typename Gemm::GemmKernel::CollectiveMainloop::InternalLayoutSFA, LayoutSFA>);
static_assert(cute::is_same_v<typename Gemm::GemmKernel::CollectiveMainloop::InternalLayoutSFB, LayoutSFB>);

/// Initialization
uint64_t seed;

// Host-side allocations
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Targets blackwell sm100 execution paths. Selects tensor-core mma execution instead of a simt fallback.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。面向 Blackwell SM100 执行路径。 选择 Tensor Core MMA 执行，而不是 SIMT 回退路径。

---

```cpp
std::vector<int64_t> offset_A;
std::vector<int64_t> offset_B;
std::vector<int64_t> offset_C;
std::vector<int64_t> offset_D;
std::vector<int64_t> offset_SFA;
std::vector<int64_t> offset_SFB;

std::vector<StrideA> stride_A_host;
std::vector<StrideB> stride_B_host;
std::vector<StrideC> stride_C_host;
std::vector<StrideD> stride_D_host;
std::vector<LayoutSFA> layout_SFA_host;
std::vector<LayoutSFB> layout_SFB_host;

std::vector<ElementD *> ptr_ref_D_host;

std::vector<ElementA *> ptr_A_host;
std::vector<ElementB *> ptr_B_host;
std::vector<ElementC *> ptr_C_host;
std::vector<ElementD *> ptr_D_host;
std::vector<ElementAccumulator *> ptr_SFA_host;
std::vector<ElementAccumulator *> ptr_SFB_host;

// Shared Allocations

cutlass::HostTensor<ElementA, cutlass::layout::PackedVectorLayout> block_A;
cutlass::HostTensor<ElementB, cutlass::layout::PackedVectorLayout> block_B;
cutlass::HostTensor<ElementC, cutlass::layout::PackedVectorLayout> block_C;
cutlass::HostTensor<ElementD, cutlass::layout::PackedVectorLayout> block_D;
cutlass::HostTensor<ElementD, cutlass::layout::PackedVectorLayout> block_ref_D;
cutlass::HostTensor<ElementAccumulator, cutlass::layout::PackedVectorLayout> block_SFA;
cutlass::HostTensor<ElementAccumulator, cutlass::layout::PackedVectorLayout> block_SFB;

// Device-side allocations
cutlass::DeviceAllocation<typename ProblemShape::UnderlyingProblemShape> problem_sizes;
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。

---

```cpp
cutlass::DeviceAllocation<const typename Gemm::ElementA *> ptr_A;
cutlass::DeviceAllocation<const typename Gemm::ElementB *> ptr_B;
cutlass::DeviceAllocation<const typename Gemm::ElementC *> ptr_C;
cutlass::DeviceAllocation<typename Gemm::EpilogueOutputOp::ElementOutput *> ptr_D;
cutlass::DeviceAllocation<const ElementAccumulator *> ptr_SFA;
cutlass::DeviceAllocation<const ElementAccumulator *> ptr_SFB;

cutlass::DeviceAllocation<StrideA> stride_A;
cutlass::DeviceAllocation<StrideB> stride_B;
cutlass::DeviceAllocation<StrideC> stride_C;
cutlass::DeviceAllocation<StrideD> stride_D;
cutlass::DeviceAllocation<LayoutSFA> layout_SFA;
cutlass::DeviceAllocation<LayoutSFB> layout_SFB;
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。

---

```cpp
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
  int m = 1024, n = 2048, k = 512, groups = 10;
  std::vector<typename ProblemShape::UnderlyingProblemShape> problem_sizes_host;

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
    cmd.get_cmd_line_argument("groups", groups);
    cmd.get_cmd_line_argument("alpha", alpha, 1.f);
    cmd.get_cmd_line_argument("beta", beta, 0.f);
    cmd.get_cmd_line_argument("iterations", iterations);

    for (int i = 0; i < groups; ++i) {
      problem_sizes_host.push_back({m, n, k});
    }

  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。

---

```cpp
    out << "81_blackwell_grouped_gemm_groupwise\n\n"
      << "  Blackwell FP8 GEMM with Groupwise Scaling using a Warp Specialized kernel.\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement\n\n"
      << "  --m=<int>                   Sets the M extent of the GEMM\n"
      << "  --n=<int>                   Sets the N extent of the GEMM\n"
      << "  --k=<int>                   Sets the K extent of the GEMM\n"
      << "  --groups=<int>              Sets the number of individual GEMM problems for Grouped GEMM\n"
      << "  --alpha=<f32>               Epilogue scalar alpha\n"
      << "  --beta=<f32>                Epilogue scalar beta\n"
      << "  --iterations=<int>          Number of profiling iterations to perform.\n\n"
      << "  --skip-verification         Skip verification.\n\n";

    out
      << "\n\nExamples:\n\n"
      << "$ " << "81_blackwell_grouped_gemm_groupwise" << " --m=1024 --n=512 --k=1024 --alpha=2 --beta=0.707 \n\n";

    return out;
  }

  /// Compute performance in GFLOP/s
  double gflops(double runtime_s) const {
    // Two flops per multiply-add
    uint64_t flop = uint64_t(2) * m * n * k * groups;
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
  else if (dist_kind == cutlass::Distribution::AllOnes) {
    cutlass::reference::host::TensorFill(view, Element(1));
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

    scope_min = -1;
    scope_max = 1;

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
  else if (dist_kind == cutlass::Distribution::AllOnes) {
    cutlass::reference::host::TensorFill(view, Element(1));
  }
  else {
    throw std::runtime_error("Not implementated.");
  }

  return true;
}

/// Initialize operands to be used in the GEMM and reference GEMM
void initialize(const Options &options) {
  int32_t total_elements_A = 0;
  int32_t total_elements_B = 0;
  int32_t total_elements_C = 0;
  int32_t total_elements_D = 0;
  int32_t total_elements_SFA = 0;
  int32_t total_elements_SFB = 0;

  for (int32_t i = 0; i < options.groups; ++i) {

    auto problem = options.problem_sizes_host.at(i);
    auto M = get<0>(problem);
    auto N = get<1>(problem);
    auto K = get<2>(problem);

    offset_A.push_back(total_elements_A);
    offset_B.push_back(total_elements_B);
    offset_C.push_back(total_elements_C);
    offset_D.push_back(total_elements_D);
    offset_SFA.push_back(total_elements_SFA);
    offset_SFB.push_back(total_elements_SFB);
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。

---

```cpp
    int32_t elements_A = M * K;
    int32_t elements_B = K * N;
    int32_t elements_C = M * N;
    int32_t elements_D = M * N;

    auto gemm_layout_SFA = ScaleConfig::tile_atom_to_shape_SFA(make_shape(M, N, K, 1));
    auto gemm_layout_SFB = ScaleConfig::tile_atom_to_shape_SFB(make_shape(M, N, K, 1));

    int32_t elements_SFA = cosize(gemm_layout_SFA);
    int32_t elements_SFB = cosize(gemm_layout_SFB);

    total_elements_A += elements_A;
    total_elements_B += elements_B;
    total_elements_C += elements_C;
    total_elements_D += elements_D;
    total_elements_SFA += elements_SFA;
    total_elements_SFB += elements_SFB;

    stride_A_host.push_back(cutlass::make_cute_packed_stride(StrideA{}, {M, K, 1}));
    stride_B_host.push_back(cutlass::make_cute_packed_stride(StrideB{}, {N, K, 1}));
    stride_C_host.push_back(cutlass::make_cute_packed_stride(StrideC{}, {M, N, 1}));
    stride_D_host.push_back(cutlass::make_cute_packed_stride(StrideD{}, {M, N, 1}));
    layout_SFA_host.push_back(gemm_layout_SFA);
    layout_SFB_host.push_back(gemm_layout_SFB);
  }

  block_A.resize(cutlass::make_Coord(total_elements_A));
  block_B.resize(cutlass::make_Coord(total_elements_B));
  block_C.resize(cutlass::make_Coord(total_elements_C));
  block_D.resize(cutlass::make_Coord(total_elements_D));
  block_ref_D.resize(cutlass::make_Coord(total_elements_D));
  block_SFA.resize(cutlass::make_Coord(total_elements_SFA));
  block_SFB.resize(cutlass::make_Coord(total_elements_SFB));

  initialize_tensor(block_A.host_view(), cutlass::Distribution::Uniform, seed + 2022);
  initialize_tensor(block_B.host_view(), cutlass::Distribution::Uniform, seed + 2023);
  initialize_tensor(block_C.host_view(), cutlass::Distribution::Uniform, seed + 2024);
  initialize_scale_tensor(block_SFA.host_view(), cutlass::Distribution::Uniform, seed + 2026);
  initialize_scale_tensor(block_SFB.host_view(), cutlass::Distribution::Uniform, seed + 2027);
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。

---

```cpp
  block_A.sync_device();
  block_B.sync_device();
  block_C.sync_device();
  block_SFA.sync_device();
  block_SFB.sync_device();

  // copy problem sizes
  problem_sizes.reset(options.groups);
  problem_sizes.copy_from_host(options.problem_sizes_host.data());

  std::vector<ElementA *> device_ptr_A_host(options.groups);
  std::vector<ElementB *> device_ptr_B_host(options.groups);
  std::vector<ElementC *> device_ptr_C_host(options.groups);
  std::vector<ElementD *> device_ptr_D_host(options.groups);
  std::vector<ElementAccumulator *> device_ptr_SFA_host(options.groups);
  std::vector<ElementAccumulator *> device_ptr_SFB_host(options.groups);

  ptr_A_host = std::vector<ElementA *>(options.groups);
  ptr_B_host = std::vector<ElementB *>(options.groups);
  ptr_C_host = std::vector<ElementC *>(options.groups);
  ptr_D_host = std::vector<ElementD *>(options.groups);
  ptr_SFA_host = std::vector<ElementAccumulator *>(options.groups);
  ptr_SFB_host = std::vector<ElementAccumulator *>(options.groups);
  ptr_ref_D_host = std::vector<ElementD *>(options.groups);

  for (int32_t i = 0; i < options.groups; ++i) {
    // Ptrs for A
    ptr_A_host.at(i) = block_A.host_data() + offset_A.at(i);
    device_ptr_A_host.at(i) = block_A.device_data() + offset_A.at(i);

    // Ptrs for B
    ptr_B_host.at(i) = block_B.host_data() + offset_B.at(i);
    device_ptr_B_host.at(i) = block_B.device_data() + offset_B.at(i);

    // Ptrs for C
    ptr_C_host.at(i) = block_C.host_data() + offset_C.at(i);
    device_ptr_C_host.at(i) = block_C.device_data() + offset_C.at(i);

    // Ptrs for D
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。

---

```cpp
    ptr_D_host.at(i) = block_D.host_data() + offset_D.at(i);
    device_ptr_D_host.at(i) = block_D.device_data() + offset_D.at(i);
    ptr_ref_D_host.at(i) = block_ref_D.host_data() + offset_D.at(i);

    // Ptrs for SFA
    ptr_SFA_host.at(i) = block_SFA.host_data() + offset_SFA.at(i);
    device_ptr_SFA_host.at(i) = block_SFA.device_data() + offset_SFA.at(i);

    // Ptrs for SFB
    ptr_SFB_host.at(i) = block_SFB.host_data() + offset_SFB.at(i);
    device_ptr_SFB_host.at(i) = block_SFB.device_data() + offset_SFB.at(i);
  }

  ptr_A.reset(options.groups);
  ptr_A.copy_from_host(device_ptr_A_host.data());

  ptr_B.reset(options.groups);
  ptr_B.copy_from_host(device_ptr_B_host.data());

  ptr_C.reset(options.groups);
  ptr_C.copy_from_host(device_ptr_C_host.data());

  ptr_D.reset(options.groups);
  ptr_D.copy_from_host(device_ptr_D_host.data());

  ptr_SFA.reset(options.groups);
  ptr_SFA.copy_from_host(device_ptr_SFA_host.data());

  ptr_SFB.reset(options.groups);
  ptr_SFB.copy_from_host(device_ptr_SFB_host.data());

  stride_A.reset(options.groups);
  stride_A.copy_from_host(stride_A_host.data());

  stride_B.reset(options.groups);
  stride_B.copy_from_host(stride_B_host.data());

  stride_C.reset(options.groups);
  stride_C.copy_from_host(stride_C_host.data());
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。

---

```cpp
  stride_D.reset(options.groups);
  stride_D.copy_from_host(stride_D_host.data());

  layout_SFA.reset(options.groups);
  layout_SFA.copy_from_host(layout_SFA_host.data());

  layout_SFB.reset(options.groups);
  layout_SFB.copy_from_host(layout_SFB_host.data());

}

/// Populates a Gemm::Arguments structure from the given commandline options
typename Gemm::Arguments args_from_options(const Options &options) {
  cutlass::KernelHardwareInfo hw_info;
  hw_info.device_id = 0;
  hw_info.sm_count = cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);

  typename Gemm::Arguments arguments{
    cutlass::gemm::GemmUniversalMode::kGrouped,
    {options.groups, problem_sizes.get(), options.problem_sizes_host.data()},
    {ptr_A.get(), stride_A.get(),
     ptr_B.get(), stride_B.get(),
     ptr_SFA.get(), layout_SFA.get(),
     ptr_SFB.get(), layout_SFB.get()
    },
    {
      {}, // epilogue.thread
      ptr_C.get(), stride_C.get(),
      ptr_D.get(), stride_D.get()
    },
    hw_info
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
  
  block_D.sync_host();

  for (int i = 0; i < options.groups; ++i) {
    auto problem = options.problem_sizes_host.at(i);
    auto M = get<0>(problem);
    auto N = get<1>(problem);
    auto K = get<2>(problem);

    // Create instantiation for device reference gemm kernel
    auto A = cute::make_tensor(ptr_A_host.at(i),
        cute::make_layout(cute::make_shape(M, K, 1), stride_A_host.at(i)));
    auto B = cute::make_tensor(ptr_B_host.at(i),
        cute::make_layout(cute::make_shape(N, K, 1), stride_B_host.at(i)));
    auto C = cute::make_tensor(ptr_C_host.at(i),
        cute::make_layout(cute::make_shape(M, N, 1), stride_C_host.at(i)));
    auto D = cute::make_tensor(ptr_ref_D_host.at(i),
        cute::make_layout(cute::make_shape(M, N, 1), stride_D_host.at(i)));

    auto SFA = cute::make_tensor(ptr_SFA_host.at(i), layout_SFA_host.at(i));
    auto SFB = cute::make_tensor(ptr_SFB_host.at(i), layout_SFB_host.at(i));

    using unused_t = decltype(D);

    cutlass::reference::host::GettBlockScalingMainloopParams<
        ElementAccumulator,
        decltype(A), 
        decltype(SFA), 
        decltype(B),
        decltype(SFB)
      > mainloop_params{A, SFA, B, SFB};
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。

---

```cpp
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

  }

  bool passed = cutlass::reference::host::TensorEquals(block_ref_D.host_view(), block_D.host_view());

  return passed;
}

/// Execute a given example GEMM computation
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。

---

```cpp
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

    std::cout << "  Problem Size: " << options.m << 'x' << options.n << 'x' << options.k << 'x' << options.groups << std::endl;
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
- TMA-driven data movement / 基于 TMA 的数据搬运
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
