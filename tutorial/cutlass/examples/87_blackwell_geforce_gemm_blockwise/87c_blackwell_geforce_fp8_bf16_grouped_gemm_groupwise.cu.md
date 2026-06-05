# 87c_blackwell_geforce_fp8_bf16_grouped_gemm_groupwise.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/87_blackwell_geforce_gemm_blockwise/87c_blackwell_geforce_fp8_bf16_grouped_gemm_groupwise.cu`  
**Purpose / 用途**: Brings grouped execution to the GeForce FP8/BF16 groupwise path, so one launch can cover many problems while still honoring per-group scale layouts and optional per-group epilogue scalars. / 把 grouped 执行引入 GeForce 的 FP8/BF16 groupwise 路径，让一次 launch 可以覆盖多个问题，同时仍能保留按组 scale 布局和可选的按组 epilogue 标量。

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
    \brief An FP8 groupwise scaled grouped GEMM example for the NVIDIA Blackwell SM120 architecture using CUTLASS.
    
    This example demonstrates an implementation of Grouped GEMM using a TMA + Blackwell SM120 TensorOp-based warp-specialized kernel
    for FP8 with per-group:1x128x128 FP32 scaling factors.
    In this example, M, N, and K are fixed across groups. 
    As RTX 50 series GPUs do not support runtime scaling block sizes, all groups share the same block scaling size.
    For this example all scheduling work is performed on the device, utilizing the device-side modification of TMA descriptors
    to move between groups/problem_count (represented by groups).
    https://docs.nvidia.com/cuda/cuda-c-programming-guide/#encoding-a-tensor-map-on-device
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Reflects geforce-oriented blackwell tuning. Includes fp8/mxfp8-style narrow-precision behavior.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。体现面向 GeForce 的 Blackwell 调优。 包含 FP8/MXFP8 一类窄精度行为。

---

```cpp
    To run this example:

      $ ./examples/87_blackwell_geforce_gemm_blockwise/87c_blackwell_geforce_fp8_bf16_grouped_gemm_groupwise --m=2048 --n=2048 --k=2048 --groups=10

      The above example command makes all 10 groups to be sized at the given m, n, k sizes.
      Same applies for alpha and beta values that are randomized across the different groups.
*/

#include <iostream>

#include "cutlass/cutlass.h"

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

#include "./utils.h"

using namespace cute;
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Reflects geforce-oriented blackwell tuning. Includes fp8/mxfp8-style narrow-precision behavior.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。体现面向 GeForce 的 Blackwell 调优。 包含 FP8/MXFP8 一类窄精度行为。

---

```cpp
#if defined(CUTLASS_ARCH_MMA_SM120_SUPPORTED) || defined(CUTLASS_ARCH_MMA_SM121_SUPPORTED)

using ProblemShape = cutlass::gemm::GroupProblemShape<Shape<int,int,int>>;
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Reflects geforce-oriented blackwell tuning.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。体现面向 GeForce 的 Blackwell 调优。

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
using ElementC            = cutlass::bfloat16_t;                          // Element type for C and D matrix operands
using LayoutC             = cutlass::layout::RowMajor;                   // Layout type for C and D matrix operands
constexpr int AlignmentC  = 128 / cutlass::sizeof_bits<ElementC>::value;    // Memory access granularity/alignment of A matrix in units of elements (up to 16 bytes)

using ElementD           = ElementC;
using LayoutD            = LayoutC;
constexpr int AlignmentD = AlignmentC;

// MMA type
using ElementAccumulator = float;                                           // Element Accumulator will also be our scale factor type
using ElementCompute = float;

// MMA and Cluster Tile Shapes
// Shape of the tile
using MmaTileShape_MNK = Shape<_128,_128,_128>;                          
// Shape of the threadblocks in a cluster
using ClusterShape_MNK = Shape<_1,_1,_1>;

// Scaling Factors
using ElementSF = ElementAccumulator;

constexpr int ScaleGranularityM = 1;
constexpr int ScaleGranularityN = 128;
constexpr int ScaleGranularityK = 128;
using ScaleConfig = cutlass::detail::Sm120BlockwiseScaleConfig<ScaleGranularityM, ScaleGranularityN, ScaleGranularityK>;

using LayoutSFA             = decltype(ScaleConfig::deduce_layoutSFA());                     // Layout type for SFA matrix operand
using LayoutSFB             = decltype(ScaleConfig::deduce_layoutSFB());                     // Layout type for SFB matrix operand
```

**EN**: This configuration block defines compile-time aliases or constants that control data types, layouts, alignments, tile sizes, or enum-style modes. These choices are not cosmetic: in CUTLASS they steer which iterators, MMA atoms, and memory movement strategies become legal and efficient. Reflects geforce-oriented blackwell tuning. Includes fp8/mxfp8-style narrow-precision behavior.  
**CN**: 这个配置块定义了编译期别名或常量，用来控制数据类型、布局、对齐、tile 大小或枚举模式。这些选择并非表面参数：在 CUTLASS 中，它们会直接决定哪些迭代器、MMA atom 和数据搬运策略既合法又高效。体现面向 GeForce 的 Blackwell 调优。 包含 FP8/MXFP8 一类窄精度行为。

---

```cpp
using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
    cutlass::arch::Sm120, cutlass::arch::OpClassTensorOp,
    MmaTileShape_MNK, ClusterShape_MNK,
    cutlass::epilogue::collective::EpilogueTileAuto,
    ElementAccumulator, ElementCompute,
    ElementC, LayoutC *, AlignmentC,
    ElementD, LayoutD *, AlignmentD,
    cutlass::epilogue::collective::EpilogueScheduleAuto
  >::CollectiveOp;

using CollectiveMainloop = typename cutlass::gemm::collective::CollectiveBuilder<
    cutlass::arch::Sm120, cutlass::arch::OpClassTensorOp,
    ElementA, cute::tuple<LayoutA *, LayoutSFA *>, AlignmentA,
    ElementB, cute::tuple<LayoutB *, LayoutSFB *>, AlignmentB,
    ElementAccumulator,
    MmaTileShape_MNK, ClusterShape_MNK,
    cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))>,
    cutlass::gemm::KernelScheduleSm120Blockwise
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
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Reflects geforce-oriented blackwell tuning. Selects tensor-core mma execution instead of a simt fallback.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。体现面向 GeForce 的 Blackwell 调优。 选择 Tensor Core MMA 执行，而不是 SIMT 回退路径。

---

```cpp
/// Initialization
uint64_t seed;

std::vector<StrideA> stride_A_host;
std::vector<StrideB> stride_B_host;
std::vector<StrideC> stride_C_host;
std::vector<StrideD> stride_D_host;
std::vector<LayoutSFA> layout_SFA_host;
std::vector<LayoutSFB> layout_SFB_host;

std::vector<ElementAccumulator> alpha_host;
std::vector<ElementAccumulator> beta_host;

using HostTensorA = cutlass::HostTensor<ElementA, cutlass::layout::PackedVectorLayout>;
using HostTensorB = cutlass::HostTensor<ElementB, cutlass::layout::PackedVectorLayout>;
using HostTensorC = cutlass::HostTensor<ElementC, cutlass::layout::PackedVectorLayout>;
using HostTensorD = cutlass::HostTensor<Gemm::EpilogueOutputOp::ElementOutput, cutlass::layout::PackedVectorLayout>;
using HostTensorSFA = cutlass::HostTensor<ElementAccumulator, cutlass::layout::PackedVectorLayout>;
using HostTensorSFB = cutlass::HostTensor<ElementAccumulator, cutlass::layout::PackedVectorLayout>;

std::vector<HostTensorA> block_A;
std::vector<HostTensorB> block_B;
std::vector<HostTensorC> block_C;
std::vector<HostTensorD> block_D;
std::vector<HostTensorD> block_ref_D;
std::vector<HostTensorSFA> block_SFA;
std::vector<HostTensorSFB> block_SFB;

cutlass::DeviceAllocation<typename ProblemShape::UnderlyingProblemShape> problem_sizes;
cutlass::DeviceAllocation<ElementA const*> ptr_A;
cutlass::DeviceAllocation<ElementB const*> ptr_B;
cutlass::DeviceAllocation<ElementSF const*> ptr_SFA;
cutlass::DeviceAllocation<ElementSF const*> ptr_SFB;
cutlass::DeviceAllocation<ElementC const*> ptr_C;
cutlass::DeviceAllocation<ElementD *> ptr_D;
cutlass::DeviceAllocation<ElementD *> ptr_ref_D;
```

**EN**: This block prepares tensors, layouts, strides, seeds, or auxiliary buffers before launch. Although it lives on the host side, it mirrors device expectations closely so the runtime data layout matches the compile-time kernel configuration. Reflects geforce-oriented blackwell tuning.  
**CN**: 这一块在启动前准备张量、布局、stride、随机种子或辅助缓冲。虽然它运行在主机侧，但会严格映射设备端预期，从而保证运行期数据布局与编译期内核配置一致。体现面向 GeForce 的 Blackwell 调优。

---

```cpp
cutlass::DeviceAllocation<StrideA> stride_A;
cutlass::DeviceAllocation<StrideB> stride_B;
cutlass::DeviceAllocation<StrideC> stride_C;
cutlass::DeviceAllocation<StrideD> stride_D;
cutlass::DeviceAllocation<LayoutSFA> layout_SFA;
cutlass::DeviceAllocation<LayoutSFB> layout_SFB;

cutlass::DeviceAllocation<ElementAccumulator*> alpha_device;
cutlass::DeviceAllocation<ElementAccumulator*> beta_device;
cutlass::DeviceAllocation<ElementAccumulator> block_alpha;
cutlass::DeviceAllocation<ElementAccumulator> block_beta;
```

**EN**: This block prepares tensors, layouts, strides, seeds, or auxiliary buffers before launch. Although it lives on the host side, it mirrors device expectations closely so the runtime data layout matches the compile-time kernel configuration. Reflects geforce-oriented blackwell tuning.  
**CN**: 这一块在启动前准备张量、布局、stride、随机种子或辅助缓冲。虽然它运行在主机侧，但会严格映射设备端预期，从而保证运行期数据布局与编译期内核配置一致。体现面向 GeForce 的 Blackwell 调优。

---

```cpp
#endif // defined(CUTLASS_ARCH_MMA_SM120_SUPPORTED) || defined(CUTLASS_ARCH_MMA_SM121_SUPPORTED)
```

**EN**: This block prepares tensors, layouts, strides, seeds, or auxiliary buffers before launch. Although it lives on the host side, it mirrors device expectations closely so the runtime data layout matches the compile-time kernel configuration. Reflects geforce-oriented blackwell tuning.  
**CN**: 这一块在启动前准备张量、布局、stride、随机种子或辅助缓冲。虽然它运行在主机侧，但会严格映射设备端预期，从而保证运行期数据布局与编译期内核配置一致。体现面向 GeForce 的 Blackwell 调优。

---

```cpp
/// Testbed utility types
```

**EN**: This configuration block defines compile-time aliases or constants that control data types, layouts, alignments, tile sizes, or enum-style modes. These choices are not cosmetic: in CUTLASS they steer which iterators, MMA atoms, and memory movement strategies become legal and efficient. Reflects geforce-oriented blackwell tuning.  
**CN**: 这个配置块定义了编译期别名或常量，用来控制数据类型、布局、对齐、tile 大小或枚举模式。这些选择并非表面参数：在 CUTLASS 中，它们会直接决定哪些迭代器、MMA atom 和数据搬运策略既合法又高效。体现面向 GeForce 的 Blackwell 调优。

---

```cpp
using RasterOrderOptions = cutlass::gemm::kernel::detail::RasterOrderOptions;
// Command line options parsing
struct Options {

  bool help = false;
  bool skip_verification = false;

  float alpha = 1.f, beta = 0.f;
  int iterations = 1000;
  int m = 1024, n = 512, k = 1024, l = 1, groups = 10;
  std::vector<typename ProblemShape::UnderlyingProblemShape> problem_sizes_host;
  RasterOrderOptions raster_order = RasterOrderOptions::AlongN;

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

    char raster_char;
    cmd.get_cmd_line_argument("raster", raster_char, 'N');
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Reflects geforce-oriented blackwell tuning.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。体现面向 GeForce 的 Blackwell 调优。

---

```cpp
    if (raster_char == 'N' || raster_char == 'n') {
      raster_order = RasterOrderOptions::AlongN;
    } else if (raster_char == 'M' || raster_char == 'm') {
      raster_order = RasterOrderOptions::AlongM;
    }

    for (int i = 0; i < groups; ++i) {
      problem_sizes_host.push_back({m, n, k});
    }
  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << "87c_blackwell_geforce_grouped_gemm_groupwise\n\n"
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
      << "$ " << "87c_blackwell_geforce_grouped_gemm_groupwise" << " --m=1024 --n=512 --k=1024 --groups=8 --alpha=2 --beta=0.707 \n\n";

    return out;
  }

  /// Compute performance in GFLOP/s
  double gflops(double runtime_s) const {
    // Two flops per multiply-add
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Reflects geforce-oriented blackwell tuning. Includes fp8/mxfp8-style narrow-precision behavior.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。体现面向 GeForce 的 Blackwell 调优。 包含 FP8/MXFP8 一类窄精度行为。

---

```cpp
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

  Result(
    double avg_runtime_ms = 0,
    double gflops = 0,
    cutlass::Status status = cutlass::Status::kSuccess,
    cudaError_t error = cudaSuccess)
  :
    avg_runtime_ms(avg_runtime_ms), gflops(gflops), status(status), error(error), passed(false)
  {}

};
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Reflects geforce-oriented blackwell tuning.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。体现面向 GeForce 的 Blackwell 调优。

---

```cpp
#if defined(CUTLASS_ARCH_MMA_SM120_SUPPORTED) || defined(CUTLASS_ARCH_MMA_SM121_SUPPORTED)
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Reflects geforce-oriented blackwell tuning.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。体现面向 GeForce 的 Blackwell 调优。

---

```cpp
/// GEMM setup and evaluation
```

**EN**: This configuration block defines compile-time aliases or constants that control data types, layouts, alignments, tile sizes, or enum-style modes. These choices are not cosmetic: in CUTLASS they steer which iterators, MMA atoms, and memory movement strategies become legal and efficient. Reflects geforce-oriented blackwell tuning.  
**CN**: 这个配置块定义了编译期别名或常量，用来控制数据类型、布局、对齐、tile 大小或枚举模式。这些选择并非表面参数：在 CUTLASS 中，它们会直接决定哪些迭代器、MMA atom 和数据搬运策略既合法又高效。体现面向 GeForce 的 Blackwell 调优。

---

```cpp
/// Initialize operands to be used in the GEMM and reference GEMM
void initialize(const Options &options) {
  using namespace cute;

  std::vector<ElementA *> ptr_A_host(options.groups);
  std::vector<ElementB *> ptr_B_host(options.groups);
  std::vector<ElementSF *> ptr_SFA_host(options.groups);
  std::vector<ElementSF *> ptr_SFB_host(options.groups);
  std::vector<ElementC *> ptr_C_host(options.groups);
  std::vector<ElementD *> ptr_D_host(options.groups);
  std::vector<ElementAccumulator *> ptr_alpha_host(options.groups);
  std::vector<ElementAccumulator *> ptr_beta_host(options.groups);

  block_alpha.reset(options.groups);
  block_beta.reset(options.groups);
  for (int i = 0; i < options.groups; ++i) {
    auto problem = options.problem_sizes_host.at(i);
    auto [M, N, K] = problem;
    
    auto stride_A = cutlass::make_cute_packed_stride(StrideA{}, {M, K, 1});
    auto stride_B = cutlass::make_cute_packed_stride(StrideB{}, {N, K, 1});
    auto stride_C = cutlass::make_cute_packed_stride(StrideC{}, {M, N, 1});
    auto stride_D = cutlass::make_cute_packed_stride(StrideC{}, {M, N, 1});

    auto layout_A = make_layout(make_shape(M, K, 1), stride_A);
    auto layout_B = make_layout(make_shape(N, K, 1), stride_B);
    auto layout_C = make_layout(make_shape(M, N, 1), stride_C);
    auto layout_D = make_layout(make_shape(M, N, 1), stride_D);

    auto layout_SFA = ScaleConfig::tile_atom_to_shape_SFA(make_shape(M, N, K, 1));
    auto layout_SFB = ScaleConfig::tile_atom_to_shape_SFB(make_shape(M, N, K, 1));

    stride_A_host.push_back(stride_A);
    stride_B_host.push_back(stride_B);
    layout_SFA_host.push_back(layout_SFA);
    layout_SFB_host.push_back(layout_SFB);
    stride_C_host.push_back(stride_C);
    stride_D_host.push_back(stride_D);
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface. Reflects geforce-oriented blackwell tuning.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。体现面向 GeForce 的 Blackwell 调优。

---

```cpp
    block_A.push_back(HostTensorA(cutlass::make_Coord(size(layout_A))));
    block_B.push_back(HostTensorB(cutlass::make_Coord(size(layout_B))));
    block_C.push_back(HostTensorC(cutlass::make_Coord(size(layout_C))));
    block_D.push_back(HostTensorD(cutlass::make_Coord(size(layout_D))));
    block_SFA.push_back(HostTensorSFA(cutlass::make_Coord(size(filter_zeros(layout_SFA)))));
    block_SFB.push_back(HostTensorSFB(cutlass::make_Coord(size(filter_zeros(layout_SFB)))));
    block_ref_D.push_back(HostTensorD(cutlass::make_Coord(size(layout_D))));
  }

  for (int i = 0; i < options.groups; ++i) {
    initialize_tensor(block_A.at(i).host_view(), cutlass::Distribution::Uniform, seed + 2022);
    initialize_tensor(block_B.at(i).host_view(), cutlass::Distribution::Uniform, seed + 2023);
    initialize_tensor(block_C.at(i).host_view(), cutlass::Distribution::Uniform, seed + 2024);
    initialize_tensor(block_SFA.at(i).host_view(), cutlass::Distribution::Uniform, seed + 2025);
    initialize_tensor(block_SFB.at(i).host_view(), cutlass::Distribution::Uniform, seed + 2026);

    block_A.at(i).sync_device();
    block_B.at(i).sync_device();
    block_C.at(i).sync_device();
    block_SFA.at(i).sync_device();
    block_SFB.at(i).sync_device();

    ptr_A_host.at(i) = block_A.at(i).device_data();
    ptr_B_host.at(i) = block_B.at(i).device_data();
    ptr_C_host.at(i) = block_C.at(i).device_data();
    ptr_D_host.at(i) = block_D.at(i).device_data();
    ptr_SFA_host.at(i) = block_SFA.at(i).device_data();
    ptr_SFB_host.at(i) = block_SFB.at(i).device_data();

    alpha_host.push_back((options.alpha == std::numeric_limits<float>::max()) ? static_cast<ElementAccumulator>((rand() % 5) + 1) : options.alpha);
    beta_host.push_back((options.beta == std::numeric_limits<float>::max()) ? static_cast<ElementAccumulator>(rand() % 5) : options.beta);
    ptr_alpha_host.at(i) = block_alpha.get() + i;
    ptr_beta_host.at(i) = block_beta.get() + i;
  }

  problem_sizes.reset(options.groups);
  problem_sizes.copy_from_host(options.problem_sizes_host.data());

  ptr_A.reset(options.groups);
  ptr_A.copy_from_host(ptr_A_host.data());
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface. Reflects geforce-oriented blackwell tuning.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。体现面向 GeForce 的 Blackwell 调优。

---

```cpp
  ptr_B.reset(options.groups);
  ptr_B.copy_from_host(ptr_B_host.data());

  ptr_SFA.reset(options.groups);
  ptr_SFA.copy_from_host(ptr_SFA_host.data());

  ptr_SFB.reset(options.groups);
  ptr_SFB.copy_from_host(ptr_SFB_host.data());

  ptr_C.reset(options.groups);
  ptr_C.copy_from_host(ptr_C_host.data());

  ptr_D.reset(options.groups);
  ptr_D.copy_from_host(ptr_D_host.data());

  stride_A.reset(options.groups);
  stride_A.copy_from_host(stride_A_host.data());

  stride_B.reset(options.groups);
  stride_B.copy_from_host(stride_B_host.data());

  layout_SFA.reset(options.groups);
  layout_SFA.copy_from_host(layout_SFA_host.data());

  layout_SFB.reset(options.groups);
  layout_SFB.copy_from_host(layout_SFB_host.data());

  stride_C.reset(options.groups);
  stride_C.copy_from_host(stride_C_host.data());

  stride_D.reset(options.groups);
  stride_D.copy_from_host(stride_D_host.data());

  alpha_device.reset(options.groups);
  alpha_device.copy_from_host(ptr_alpha_host.data());
  beta_device.reset(options.groups);
  beta_device.copy_from_host(ptr_beta_host.data());

  block_alpha.copy_from_host(alpha_host.data());
  block_beta.copy_from_host(beta_host.data());
}
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface. Reflects geforce-oriented blackwell tuning.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。体现面向 GeForce 的 Blackwell 调优。

---

```cpp
/// Populates a Gemm::Arguments structure from the given commandline options
typename Gemm::Arguments args_from_options(const Options &options) {
  cutlass::KernelHardwareInfo hw_info;
  hw_info.device_id = 0;
  hw_info.sm_count = cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);

  typename Gemm::GemmKernel::TileSchedulerArguments scheduler;
  scheduler.raster_order = options.raster_order;

  typename Gemm::Arguments arguments;
  decltype(arguments.epilogue.thread) fusion_args;
  fusion_args.alpha_ptr = nullptr;
  fusion_args.beta_ptr = nullptr;

  if (options.alpha != std::numeric_limits<float>::max()) {
    fusion_args.alpha = options.alpha;
    fusion_args.alpha_ptr_array = nullptr;
    fusion_args.dAlpha = {_0{}, _0{}, 0};
  } else {
    fusion_args.alpha = 0;
    fusion_args.alpha_ptr_array = alpha_device.get();
    fusion_args.dAlpha = {_0{}, _0{}, 1};
  }

  if (options.beta != std::numeric_limits<float>::max()) {
    fusion_args.beta = options.beta;
    fusion_args.beta_ptr_array = nullptr;
    fusion_args.dBeta = {_0{}, _0{}, 0};
  } else {
    fusion_args.beta = 0;
    fusion_args.beta_ptr_array = beta_device.get();
    fusion_args.dBeta = {_0{}, _0{}, 1};
  }
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface. Reflects geforce-oriented blackwell tuning. Exposes scheduler policy as a first-class performance choice.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。体现面向 GeForce 的 Blackwell 调优。 把调度策略作为一级性能选择暴露出来。

---

```cpp
  arguments = {
    cutlass::gemm::GemmUniversalMode::kGrouped,
    {options.groups, problem_sizes.get(), options.problem_sizes_host.data()},
    {ptr_A.get(), stride_A.get(), 
     ptr_B.get(), stride_B.get(),
     ptr_SFA.get(), layout_SFA.get(),
     ptr_SFB.get(), layout_SFB.get()},
    {
      fusion_args,
      ptr_C.get(), stride_C.get(),
      ptr_D.get(), stride_D.get()
    },
    hw_info, scheduler
  };

  return arguments;
}

bool verify(const Options &options) {
  //
  // Compute reference output
  //

  bool passed = true;

  for (int i = 0; i < options.groups; ++i) {
    auto problem = options.problem_sizes_host.at(i);
    auto [M, N, K] = problem;

    auto A = cute::make_tensor(block_A.at(i).host_data(),
        cute::make_layout(cute::make_shape(M, K, 1), stride_A_host.at(i)));
    auto B = cute::make_tensor(block_B.at(i).host_data(),
        cute::make_layout(cute::make_shape(N, K, 1), stride_B_host.at(i)));
    auto C = cute::make_tensor(block_C.at(i).host_data(),
        cute::make_layout(cute::make_shape(M, N, 1), stride_C_host.at(i)));
    auto D = cute::make_tensor(block_ref_D.at(i).host_data(),
        cute::make_layout(cute::make_shape(M, N, 1), stride_D_host.at(i)));
    auto SFA = cute::make_tensor(block_SFA.at(i).host_data(), layout_SFA_host.at(i));
    auto SFB = cute::make_tensor(block_SFB.at(i).host_data(), layout_SFB_host.at(i));
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface. Reflects geforce-oriented blackwell tuning.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。体现面向 GeForce 的 Blackwell 调优。

---

```cpp
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
    epilogue_params.alpha = alpha_host.at(i);
    epilogue_params.beta = beta_host.at(i);
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface. Reflects geforce-oriented blackwell tuning.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。体现面向 GeForce 的 Blackwell 调优。

---

```cpp
    cutlass::reference::host::Gemm3x(mainloop_params, epilogue_params);
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface. Reflects geforce-oriented blackwell tuning.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。体现面向 GeForce 的 Blackwell 调优。

---

```cpp
    block_D.at(i).sync_host();
    passed &= cutlass::reference::host::TensorEquals(block_ref_D.at(i).host_view(), block_D.at(i).host_view());
  }

  return passed;
}

/// Execute a given example GEMM computation
template <class Gemm>
int run(Options &options) {
  
  // Instantiate CUTLASS kernel depending on templates
  Gemm gemm;

  // Create a structure of gemm kernel arguments suitable for invoking an instance of Gemm
  auto arguments = args_from_options(options);

  // Using the arguments, query for extra workspace required for matrix multiplication computation
  size_t workspace_size = Gemm::get_workspace_size(arguments);

  // Allocate workspace memory
  cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Reflects geforce-oriented blackwell tuning.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。体现面向 GeForce 的 Blackwell 调优。

---

```cpp
  // Check if the problem size is supported or not
  CUTLASS_CHECK(gemm.can_implement(arguments));
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Reflects geforce-oriented blackwell tuning.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。体现面向 GeForce 的 Blackwell 调优。

---

```cpp
  // Initialize CUTLASS kernel with arguments and workspace pointer
  CUTLASS_CHECK(gemm.initialize(arguments, workspace.get()));
```

**EN**: This block prepares tensors, layouts, strides, seeds, or auxiliary buffers before launch. Although it lives on the host side, it mirrors device expectations closely so the runtime data layout matches the compile-time kernel configuration. Reflects geforce-oriented blackwell tuning.  
**CN**: 这一块在启动前准备张量、布局、stride、随机种子或辅助缓冲。虽然它运行在主机侧，但会严格映射设备端预期，从而保证运行期数据布局与编译期内核配置一致。体现面向 GeForce 的 Blackwell 调优。

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

    std::cout << "  Problem Size: " << options.m << 'x' << options.n << 'x' << options.k << " " << options.groups << " Groups" << std::endl;
    std::cout << "  Avg runtime: " << result.avg_runtime_ms << " ms" << std::endl;
    std::cout << "  GFLOPS: " << result.gflops << std::endl;
  }

  return 0;
}

#endif // defined(CUTLASS_ARCH_MMA_SM120_SUPPORTED) || defined(CUTLASS_ARCH_MMA_SM121_SUPPORTED)
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Reflects geforce-oriented blackwell tuning.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。体现面向 GeForce 的 Blackwell 调优。

---

```cpp
int main(int argc, char const **args) {

  // CUTLASS must be compiled with CUDA 12.8 or higher Toolkit for SM120 support,
  // or CUDA 12.9 or higher for SM121 support.
  // Must have compute capability at least 120.
#if defined(CUTLASS_ARCH_MMA_SM120_SUPPORTED)
  if (__CUDACC_VER_MAJOR__ < 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ < 8)) {
    std::cerr << "This example requires CUDA 12.8 or newer for SM120 support." << std::endl;
    // Returning zero so this test passes on older Toolkits. Its actions are no-op.
    return 0;
  }
#elif defined(CUTLASS_ARCH_MMA_SM121_SUPPORTED)
  if (__CUDACC_VER_MAJOR__ < 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ < 9)) {
    std::cerr << "This example requires CUDA 12.9 or newer for SM121 support." << std::endl;
    // Returning zero so this test passes on older Toolkits. Its actions are no-op.
    return 0;
  }
#endif

  cudaDeviceProp props;
  int current_device_id;
  CUDA_CHECK(cudaGetDevice(&current_device_id));
  CUDA_CHECK(cudaGetDeviceProperties(&props, current_device_id));
  cudaError_t error = cudaGetDeviceProperties(&props, 0);
  if (!(props.major == 12 && (props.minor == 0 || props.minor == 1))) {
    std::cerr << "This example requires a GPU with compute capability 120a or 121a)." << std::endl;
    return 0;
  }
```

**EN**: This block forms the outer control flow of the sample. It performs capability checks, selects the concrete specialization to run, invokes the operation, and reports the final status or throughput back to the user. Reflects geforce-oriented blackwell tuning.  
**CN**: 这一块构成了示例的最外层控制流：先做能力检查，再选择具体特化版本，调用 operation，最后把状态或吞吐结果回报给用户。体现面向 GeForce 的 Blackwell 调优。

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
#if defined(CUTLASS_ARCH_MMA_SM120_SUPPORTED) || defined(CUTLASS_ARCH_MMA_SM121_SUPPORTED)
  initialize(options);
  run<Gemm>(options);
#endif // defined(CUTLASS_ARCH_MMA_SM120_SUPPORTED) || defined(CUTLASS_ARCH_MMA_SM121_SUPPORTED)

  return 0;
}
```

**EN**: This block prepares tensors, layouts, strides, seeds, or auxiliary buffers before launch. Although it lives on the host side, it mirrors device expectations closely so the runtime data layout matches the compile-time kernel configuration. Reflects geforce-oriented blackwell tuning.  
**CN**: 这一块在启动前准备张量、布局、stride、随机种子或辅助缓冲。虽然它运行在主机侧，但会严格映射设备端预期，从而保证运行期数据布局与编译期内核配置一致。体现面向 GeForce 的 Blackwell 调优。

---

## Key Concepts / 关键概念

- Architecture-tagged specialization (`Sm100`/`Sm103`/`Sm120`) / 基于架构标签的特化（`Sm100`/`Sm103`/`Sm120`）
- CUTLASS collective builders that deduce mainloop and epilogue implementations / 推导 mainloop 与 epilogue 实现的 CUTLASS collective 构建器
- Cluster-shape tuning and possible multi-SM cooperation / Cluster 形状调优与可能的多 SM 协作
- FP8-family mixed-precision execution / FP8 家族混合精度执行
- TMA-driven data movement / 基于 TMA 的数据搬运
- Scheduler policy as a first-class optimization knob / 作为一级优化旋钮的调度策略

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
