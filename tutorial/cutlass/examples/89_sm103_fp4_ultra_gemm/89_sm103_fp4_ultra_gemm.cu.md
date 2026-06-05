# 89_sm103_fp4_ultra_gemm.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/89_sm103_fp4_ultra_gemm/89_sm103_fp4_ultra_gemm.cu`  
**Purpose / 用途**: Demonstrates a single-problem SM103 Ultra FP4 GEMM with block scaling, compares 1SM and 2SM Blackwell kernel variants, verifies against a host reference, and reports throughput. / 演示单问题 SM103 Ultra FP4 GEMM：使用分块缩放，比较 1SM 与 2SM 两种 Blackwell 内核变体，使用主机参考实现做验证，并报告吞吐。

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
    \brief A GEMM example using CUTLASS for the NVIDIA Blackwell SM103 architecture.

    This example demonstrates a simple way to instantiate and run a blockscaled ultra FP4 GEMM on the NVIDIA Blackwell SM103 architecture.

    Usage:

      $ ./examples/89_sm103_fp4_ultra_gemm/89_sm103_fp4_ultra_gemm --m=2048 --n=2048 --k=2048
*/
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Targets the newer sm103 narrow-precision path. Handles fp4/nvfp4-style narrow precision.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。面向更新的 SM103 窄精度路径。 处理 FP4/NVFP4 一类窄精度数据。

---

```cpp
#include <iostream>

#include "cutlass/cutlass.h"

#include "cute/tensor.hpp"
#include "cutlass/tensor_ref.h"
#include "cutlass/epilogue/thread/linear_combination.h"
#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/gemm/collective/collective_builder.hpp"
#include "cutlass/epilogue/collective/collective_builder.hpp"
#include "cutlass/detail/sm100_blockscaled_layout.hpp"
#include "cutlass/gemm/device/gemm_universal_adapter.h"
#include "cutlass/gemm/kernel/gemm_universal.hpp"
#include "cutlass/gemm/kernel/tile_scheduler_params.h"
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Targets blackwell sm100 execution paths. Targets the newer sm103 narrow-precision path.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。面向 Blackwell SM100 执行路径。 面向更新的 SM103 窄精度路径。

---

```cpp
#include "cutlass/util/command_line.h"
#include "cutlass/util/distribution.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/packed_stride.hpp"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/reference/device/gemm.h"
#include "cutlass/util/reference/device/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/gett.hpp"
#include "cutlass/util/reference/host/tensor_norm.h"
#include "cutlass/util/reference/host/tensor_compare.h"
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Targets the newer sm103 narrow-precision path.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。面向更新的 SM103 窄精度路径。

---

```cpp
#include <iostream>

#include "helper.h"

using namespace cute;

#if defined(CUTLASS_ARCH_MMA_SM103_SUPPORTED)
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Targets the newer sm103 narrow-precision path.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。面向更新的 SM103 窄精度路径。

---

```cpp
/// GEMM kernel configurations
```

**EN**: This configuration block defines compile-time aliases or constants that control data types, layouts, alignments, tile sizes, or enum-style modes. These choices are not cosmetic: in CUTLASS they steer which iterators, MMA atoms, and memory movement strategies become legal and efficient. Targets the newer sm103 narrow-precision path.  
**CN**: 这个配置块定义了编译期别名或常量，用来控制数据类型、布局、对齐、tile 大小或枚举模式。这些选择并非表面参数：在 CUTLASS 中，它们会直接决定哪些迭代器、MMA atom 和数据搬运策略既合法又高效。面向更新的 SM103 窄精度路径。

---

```cpp
// A matrix configuration
using         ElementA    = cutlass::float_e2m1_t;                          // Element type for A matrix operand
using         ElementSFA  = cutlass::float_ue4m3_t;
using         LayoutATag  = cutlass::layout::RowMajor;                      // Layout type for A matrix operand
constexpr int AlignmentA  = 32;                                             // Memory access granularity/alignment of A matrix in units of elements (up to 16 bytes)

// B matrix configuration
using         ElementB    = cutlass::float_e2m1_t;                          // Element type for A matrix operand
using         ElementSFB  = cutlass::float_ue4m3_t;
using         LayoutBTag  = cutlass::layout::ColumnMajor;                   // Layout type for B matrix operand
constexpr int AlignmentB  = 32;                                             // Memory access granularity/alignment of B matrix in units of elements (up to 16 bytes)

// C/D matrix configuration
using         ElementD    = cutlass::bfloat16_t;                            // Element type for D matrix operand
using         ElementC    = cutlass::bfloat16_t;                            // Element type for C matrix operand
using         LayoutCTag  = cutlass::layout::RowMajor;                      // Layout type for C matrix operand
using         LayoutDTag  = cutlass::layout::RowMajor;                      // Layout type for D matrix operand
constexpr int AlignmentD  = 128 / cutlass::sizeof_bits<ElementD>::value;    // Memory access granularity/alignment of C matrix in units of elements (up to 16 bytes)
constexpr int AlignmentC  = 128 / cutlass::sizeof_bits<ElementC>::value;    // Memory access granularity/alignment of C matrix in units of elements (up to 16 bytes)
// Kernel functional config
using ElementAccumulator  = float;                                          // Element type for internal accumulation
using ArchTag             = cutlass::arch::Sm103;                           // Tag indicating the minimum SM that supports the intended feature
using OperatorClass       = cutlass::arch::OpClassBlockScaledTensorOp;      // Operator class tag

// using ElementD = cutlass::float_e2m1_t; // Enable for SF Output          // Element type for D matrix operands

// Kernel Perf config
using MmaTileShape1Sm        = cute::Shape<cute::_128, cute::_256, Int<768>>;// 1SM MMA's tile size
using MmaTileShape2Sm        = cute::Shape<cute::_256, cute::_256, Int<768>>;// 2SM MMA's tile size
using ClusterShape        = cute::Shape<int, int, cute::_1>;                 // Cluster shape

using CollectiveEpilogue1Sm = typename cutlass::epilogue::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    MmaTileShape1Sm, ClusterShape,
    cutlass::epilogue::collective::EpilogueTileAuto,
    ElementAccumulator, ElementAccumulator,
    ElementC, LayoutCTag, AlignmentC,
    ElementD, LayoutDTag, AlignmentD,
    cutlass::epilogue::NoSmemWarpSpecialized1Sm
  >::CollectiveOp;
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Targets the newer sm103 narrow-precision path. Selects block-scaled tensor-core instructions, so data tensors and scale tensors travel together.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。面向更新的 SM103 窄精度路径。 选择分块缩放 Tensor Core 指令，因此数据张量与缩放张量需要协同传递。

---

```cpp
using CollectiveEpilogue2Sm = typename cutlass::epilogue::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    MmaTileShape2Sm, ClusterShape,
    cutlass::epilogue::collective::EpilogueTileAuto,
    ElementAccumulator, ElementAccumulator,
    ElementC, LayoutCTag, AlignmentC,
    ElementD, LayoutDTag, AlignmentD,
    cutlass::epilogue::NoSmemWarpSpecialized2Sm
  >::CollectiveOp;

using CollectiveMainloop1Sm = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    cute::tuple<ElementA,ElementSFA>, LayoutATag, AlignmentA,
    cute::tuple<ElementB,ElementSFB>, LayoutBTag, AlignmentB,
    ElementAccumulator,
    MmaTileShape1Sm, ClusterShape,
    cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(sizeof(typename CollectiveEpilogue1Sm::SharedStorage))>,
    cutlass::gemm::KernelTmaWarpSpecialized1SmBlockScaledMxNvf4UltraVs16Sm103     // Kernel schedule policy. Auto or using targeted scheduling policy
  >::CollectiveOp;
using CollectiveMainloop2Sm = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    cute::tuple<ElementA,ElementSFA>, LayoutATag, AlignmentA,
    cute::tuple<ElementB,ElementSFB>, LayoutBTag, AlignmentB,
    ElementAccumulator,
    MmaTileShape2Sm, ClusterShape,
    cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(sizeof(typename CollectiveEpilogue2Sm::SharedStorage))>,
    cutlass::gemm::KernelTmaWarpSpecialized2SmBlockScaledMxNvf4UltraVs16Sm103     // Kernel schedule policy. Auto or using targeted scheduling policy
  >::CollectiveOp;

using GemmKernel1Sm = cutlass::gemm::kernel::GemmUniversal<
    Shape<int,int,int,int>,                                                   // Indicates ProblemShape
    CollectiveMainloop1Sm,
    CollectiveEpilogue1Sm>;

using Gemm1Sm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel1Sm>;
using Gemm = Gemm1Sm;
using GemmKernel2Sm = cutlass::gemm::kernel::GemmUniversal<
    Shape<int,int,int,int>,                                                   // Indicates ProblemShape
    CollectiveMainloop2Sm,
    CollectiveEpilogue2Sm>;
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Targets the newer sm103 narrow-precision path. Makes cluster geometry an explicit compile-time tuning knob.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。面向更新的 SM103 窄精度路径。 把 cluster 几何作为显式的编译期调优参数。

---

```cpp
using Gemm2Sm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel2Sm>;

// Reference device GEMM implementation type
using StrideA   = typename Gemm::GemmKernel::StrideA;
using LayoutA   = decltype(cute::make_layout(make_shape(0,0,0), StrideA{}));
using LayoutSFA = typename Gemm::GemmKernel::CollectiveMainloop::LayoutSFA;      // Scale Factor tensors have an interleaved layout. Bring Layout instead of stride.
using StrideB   = typename Gemm::GemmKernel::StrideB;
using LayoutB   = decltype(cute::make_layout(make_shape(0,0,0), StrideB{}));
using LayoutSFB = typename Gemm::GemmKernel::CollectiveMainloop::LayoutSFB;      // Scale Factor tensors have an interleaved layout. Bring Layout instead of stride.
using StrideC   = typename Gemm::GemmKernel::StrideC;
using LayoutC   = decltype(cute::make_layout(make_shape(0,0,0), StrideC{}));
using StrideD   = typename Gemm::GemmKernel::StrideD;
using LayoutD   = decltype(cute::make_layout(make_shape(0,0,0), StrideD{}));

//
// Data members
//

/// Initialization
StrideA stride_A;
LayoutA layout_A;
LayoutSFA layout_SFA;
StrideB stride_B;
LayoutB layout_B;
LayoutSFB layout_SFB;
StrideC stride_C;
LayoutC layout_C;
StrideD stride_D;
LayoutD layout_D;
uint64_t seed;

// The HostTensors are only used for allocating memory on host and device, and transferring data between host and device
// Use cute::Tensor and cute::Layout for iterating thru the matrix elements
cutlass::HostTensor<ElementA, cutlass::layout::PackedVectorLayout> block_A;
cutlass::HostTensor<ElementSFA, cutlass::layout::PackedVectorLayout> block_SFA;
cutlass::HostTensor<ElementB, cutlass::layout::PackedVectorLayout> block_B;
cutlass::HostTensor<ElementSFB, cutlass::layout::PackedVectorLayout> block_SFB;
cutlass::HostTensor<ElementC, cutlass::layout::PackedVectorLayout> block_C;
// Output Tensor
cutlass::HostTensor<ElementD, cutlass::layout::PackedVectorLayout> block_D;
// Reference Output Tensor
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Targets the newer sm103 narrow-precision path.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。面向更新的 SM103 窄精度路径。

---

```cpp
cutlass::HostTensor<ElementD, cutlass::layout::PackedVectorLayout> block_reference_D;
#endif // defined(CUTLASS_ARCH_MMA_SM103_SUPPORTED)

template <typename T>
auto make_iterator(T* ptr) {
  return cute::recast_ptr<T>(ptr);
}
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Targets the newer sm103 narrow-precision path.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。面向更新的 SM103 窄精度路径。

---

```cpp
/// Testbed utility types
```

**EN**: This configuration block defines compile-time aliases or constants that control data types, layouts, alignments, tile sizes, or enum-style modes. These choices are not cosmetic: in CUTLASS they steer which iterators, MMA atoms, and memory movement strategies become legal and efficient. Targets the newer sm103 narrow-precision path.  
**CN**: 这个配置块定义了编译期别名或常量，用来控制数据类型、布局、对齐、tile 大小或枚举模式。这些选择并非表面参数：在 CUTLASS 中，它们会直接决定哪些迭代器、MMA atom 和数据搬运策略既合法又高效。面向更新的 SM103 窄精度路径。

---

```cpp
// Command line options parsing
struct Options {

  bool help;

  float alpha, beta;
  int iterations;
  int m, n, k;
  int swizzle = 0;

  dim3 cluster_shape = dim3(2,1,1);
  dim3 cluster_shape_fallback = dim3(2,1,1);

  bool verification = true;
  int batch = 1;

  Options():
    help(false),
    m(1024), n(1024), k(1024),
    alpha(1.f), beta(0.f),
    iterations(10),
    swizzle(0)
  { }

  // Parses the command line
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);

    if (cmd.check_cmd_line_flag("help")) {
      help = true;
      return;
    }
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Targets the newer sm103 narrow-precision path.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。面向更新的 SM103 窄精度路径。

---

```cpp
    cmd.get_cmd_line_argument("m", m);
    cmd.get_cmd_line_argument("n", n);
    cmd.get_cmd_line_argument("k", k);
    cmd.get_cmd_line_argument("alpha", alpha, 1.f);
    cmd.get_cmd_line_argument("beta", beta, 0.f);
    cmd.get_cmd_line_argument("iterations", iterations);
    cmd.get_cmd_line_argument("swizzle", swizzle);
    cmd.get_cmd_line_argument("cluster_m", cluster_shape.x);
    cmd.get_cmd_line_argument("cluster_n", cluster_shape.y);
    cmd.get_cmd_line_argument("cluster_fallback_m", cluster_shape_fallback.x);
    cmd.get_cmd_line_argument("cluster_fallback_n", cluster_shape_fallback.y);
    if (cmd.check_cmd_line_flag("no_verif")) {
      verification = false;
    }
    cmd.get_cmd_line_argument("batch", batch);
  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << "89_sm103_fp4_ultra_gemm\n\n"
      << "  Sm103 ultra FP4 GEMM using a Warp Specialized kernel.\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement\n\n"
      << "  --m=<int>                   Sets the M extent of the GEMM\n"
      << "  --n=<int>                   Sets the N extent of the GEMM\n"
      << "  --k=<int>                   Sets the K extent of the GEMM\n"
      << "  --alpha=<f32>               Epilogue scalar alpha\n"
      << "  --beta=<f32>                Epilogue scalar beta\n"
      << "  --cluster_m=<int>           Preferred cluster X dimension (input only)\n"
      << "  --cluster_n=<int>           Preferred cluster Y dimension (input only)\n"
      << "  --cluster_fallback_m=<int>  Fallback cluster X dimension (input only)\n"
      << "  --cluster_fallback_n=<int>  Fallback cluster Y dimension (input only)\n"
      << "  --swizzle=<int>             Cluster rasterization swizzle\n"
      << "  --batch=<int>               Number of batches (L dimension)\n"
      << "  --no_verif                   Do not run host-side verification\n"
      << "  --iterations=<int>          Number of profiling iterations to perform.\n\n";
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Targets the newer sm103 narrow-precision path. Handles fp4/nvfp4-style narrow precision.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。面向更新的 SM103 窄精度路径。 处理 FP4/NVFP4 一类窄精度数据。

---

```cpp
    out << "\n\nExamples:\n\n"
      << "$ " << "./examples/89_sm103_fp4_ultra_gemm/89_sm103_fp4_ultra_gemm"
      << " --m=1024 --n=512 --k=1024 --alpha=2 --beta=0.707"
      << " --cluster_m=4 --cluster_n=4 --cluster_fallback_m=2 --cluster_fallback_n=1\n\n";

    return out;
  }

  /// Compute performance in GFLOP/s
  double gflops(double runtime_s) const
  {
    // Two flops per multiply-add, times batch (L dimension)
    uint64_t flop = uint64_t(2) * uint64_t(m) * uint64_t(n) * uint64_t(k) * uint64_t(batch);
    double gflop = double(flop) / double(1.0e9);
    return gflop / runtime_s;
  }
};

/// Result structure
struct Result
{
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

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Targets the newer sm103 narrow-precision path. Handles fp4/nvfp4-style narrow precision.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。面向更新的 SM103 窄精度路径。 处理 FP4/NVFP4 一类窄精度数据。

---

```cpp
#if defined(CUTLASS_ARCH_MMA_SM103_SUPPORTED)
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Targets the newer sm103 narrow-precision path.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。面向更新的 SM103 窄精度路径。

---

```cpp
/// GEMM setup and evaluation
```

**EN**: This configuration block defines compile-time aliases or constants that control data types, layouts, alignments, tile sizes, or enum-style modes. These choices are not cosmetic: in CUTLASS they steer which iterators, MMA atoms, and memory movement strategies become legal and efficient. Targets the newer sm103 narrow-precision path.  
**CN**: 这个配置块定义了编译期别名或常量，用来控制数据类型、布局、对齐、tile 大小或枚举模式。这些选择并非表面参数：在 CUTLASS 中，它们会直接决定哪些迭代器、MMA atom 和数据搬运策略既合法又高效。面向更新的 SM103 窄精度路径。

---

```cpp
/// Helper to initialize a block of device data
template <typename Element, typename Layout>
bool initialize_block(
  cutlass::TensorView<Element, Layout> view,
  uint64_t seed) {

  double scope_max, scope_min;
  constexpr int bits_input = cutlass::sizeof_bits<Element>::value;

  if constexpr (bits_input == 1) {
    scope_max = 2;
    scope_min = 0;
  }
  else if constexpr (bits_input <= 6) {
    scope_max = 2;
    scope_min = -2;
  }
  else if constexpr (bits_input <= 8) {
    if constexpr (cute::is_same_v<Element, cutlass::float_ue8m0_t>) {
      scope_max = 4;
      scope_min = 1;
    }
    else {
      scope_max = 1;
      scope_min = -1;
    }
  }
  else{
    scope_max = 4;
    scope_min = -4;
  }
  cutlass::reference::host::TensorFillRandomUniform(
    view, seed, scope_max, scope_min, 0);

  return true;
}

/// Initialize operands to be used in the GEMM and reference GEMM
void initialize(const Options &options) {
  using namespace cute;
  // For SFA and SFB tensors layouts
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Targets the newer sm103 narrow-precision path.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。面向更新的 SM103 窄精度路径。

---

```cpp
  using Sm1xxBlkScaledConfig =  typename Gemm::GemmKernel::CollectiveMainloop::Sm1xxBlkScaledConfig;

  stride_A = cutlass::make_cute_packed_stride(StrideA{}, {options.m, options.k, options.batch});
  stride_B = cutlass::make_cute_packed_stride(StrideB{}, {options.n, options.k, options.batch});
  stride_C = cutlass::make_cute_packed_stride(StrideC{}, {options.m, options.n, options.batch});
  stride_D = cutlass::make_cute_packed_stride(StrideD{}, {options.m, options.n, options.batch});

  layout_A = make_layout(make_shape(options.m, options.k, options.batch), stride_A);
  layout_B = make_layout(make_shape(options.n, options.k, options.batch), stride_B);
  layout_C = make_layout(make_shape(options.m, options.n, options.batch), stride_C);
  layout_D = make_layout(make_shape(options.m, options.n, options.batch), stride_D);
  layout_SFA = Sm1xxBlkScaledConfig::tile_atom_to_shape_SFA(cute::make_shape(options.m, options.n, options.k, options.batch));
  layout_SFB = Sm1xxBlkScaledConfig::tile_atom_to_shape_SFB(cute::make_shape(options.m, options.n, options.k, options.batch));

  block_A.reset(cutlass::make_Coord(size(layout_A)));
  block_B.reset(cutlass::make_Coord(size(layout_B)));
  block_C.reset(cutlass::make_Coord(size(layout_C)));
  block_D.reset(cutlass::make_Coord(size(layout_D)));
  block_reference_D.reset(cutlass::make_Coord(size(layout_D)));
  block_SFA.reset(cutlass::make_Coord(size(filter_zeros(layout_SFA))));
  block_SFB.reset(cutlass::make_Coord(size(filter_zeros(layout_SFB))));

  initialize_block(block_A.host_view(), seed + 2021);
  initialize_block(block_B.host_view(), seed + 2022);
  initialize_block(block_C.host_view(), seed + 2023);
  initialize_block(block_SFA.host_view(), seed + 2024);
  initialize_block(block_SFB.host_view(), seed + 2025);

  block_A.sync_device();
  block_B.sync_device();
  block_C.sync_device();
  block_SFA.sync_device();
  block_SFB.sync_device();
}

// Populates a Gemm::Arguments structure from the given commandline options
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Targets the newer sm103 narrow-precision path.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。面向更新的 SM103 窄精度路径。

---

```cpp
template <typename Gemm>
typename Gemm::Arguments args_from_options(const Options &options)
{
  typename Gemm::Arguments arguments {
    cutlass::gemm::GemmUniversalMode::kGemm,
    {options.m, options.n, options.k, options.batch},
    { // Mainloop arguments
      block_A.device_data(), stride_A,
      block_B.device_data(), stride_B,
      block_SFA.device_data(), layout_SFA,
      block_SFB.device_data(), layout_SFB
    },
    { // Epilogue arguments
      {options.alpha, options.beta},
      block_C.device_data(), stride_C,
      block_D.device_data(), stride_D
    }
  };

  arguments.scheduler.max_swizzle_size = options.swizzle;
  arguments.hw_info.cluster_shape = options.cluster_shape;
  arguments.hw_info.cluster_shape_fallback = options.cluster_shape_fallback;
  return arguments;
}

bool verify(const Options &options) {
  using namespace cute;
  // Create the arguments for host reference implementation
  Tensor tensor_A = make_tensor(make_iterator(block_A.host_data()), layout_A);
  Tensor tensor_SFA = make_tensor(block_SFA.host_data(), layout_SFA);
  Tensor tensor_B = make_tensor(make_iterator(block_B.host_data()), layout_B);
  Tensor tensor_SFB = make_tensor(block_SFB.host_data(), layout_SFB);

  cutlass::reference::host::GettBlockScalingMainloopParams<
      ElementAccumulator,                 // ElementAccumulator
      decltype(tensor_A),                 // TensorA
      decltype(tensor_SFA),               // TensorSfA
      decltype(tensor_B),                 // TensorB
      decltype(tensor_SFB)                // TensorSfB
    > mainloop_params{tensor_A, tensor_SFA, tensor_B, tensor_SFB};
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Targets the newer sm103 narrow-precision path.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。面向更新的 SM103 窄精度路径。

---

```cpp
  auto tensor_C = cute::make_tensor(make_iterator(block_C.host_data()), layout_C);
  auto tensor_D = cute::make_tensor(make_iterator(block_reference_D.host_data()), layout_D);

  cutlass::reference::host::GettBlockScalingEpilogueParams<
      ElementAccumulator,                   // ElementScalar
      ElementAccumulator,                   // ElementAccumulator
      ElementAccumulator,                   // ElementCompute
      decltype(tensor_C),                   // TensorC
      decltype(tensor_D)                    // TensorD
    > epilogue_params{options.alpha, options.beta, tensor_C, tensor_D};

  cutlass::reference::host::Gemm3x(mainloop_params, epilogue_params);

  // Comparison
  block_D.sync_host();
  bool passed = cutlass::reference::host::TensorEquals(block_reference_D.host_view(), block_D.host_view());
  passed &= (cutlass::reference::host::TensorNorm(block_reference_D.host_view()) > 0);
  passed &= (cutlass::reference::host::TensorNorm(block_D.host_view()) > 0);

  return passed;
}

/// Execute a given example GEMM computation
template <typename Gemm>
int run(Options &options)
{
  initialize(options);

  // Instantiate CUTLASS kernel depending on templates
  Gemm gemm;

  // Create a structure of gemm kernel arguments suitable for invoking an instance of Gemm
  auto arguments = args_from_options<Gemm>(options);

  // Using the arguments, query for extra workspace required for matrix multiplication computation
  size_t workspace_size = Gemm::get_workspace_size(arguments);

  // Allocate workspace memory
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Targets the newer sm103 narrow-precision path.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。面向更新的 SM103 窄精度路径。

---

```cpp
  uint8_t* workspace = nullptr;
  cudaError_t status = cudaMalloc(&workspace, workspace_size);
  if (status != cudaSuccess) {
    std::cerr << "Failed to allocate workspace memory: " << cudaGetErrorString(status) << std::endl;
    return -1;
  }

  // Check if the problem size is supported or not
  CUTLASS_CHECK(gemm.can_implement(arguments));

  // Initialize CUTLASS kernel with arguments and workspace pointer
  CUTLASS_CHECK(gemm.initialize(arguments, workspace));

  // Correctness / Warmup iteration
  CUTLASS_CHECK(gemm.run());

  // Free workspace memory
  cudaFree(workspace);

  cudaDeviceSynchronize();

  // Check if output from CUTLASS kernel and reference kernel are equal or not
  Result result;
  if (options.verification) {
    result.passed = verify(options);
    std::cout << "  Disposition: " << (result.passed ? "Passed" : "Failed") << std::endl;
    if (!result.passed) {
      exit(-1);
    }
  } else {
    std::cout << "  Disposition: Skipped verification" << std::endl;
    result.passed = true;
  }

  // Run profiling loop
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Targets the newer sm103 narrow-precision path.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。面向更新的 SM103 窄精度路径。

---

```cpp
  if (options.iterations > 0)
  {
    GpuTimer timer;
    timer.start();
    for (int iter = 0; iter < options.iterations; ++iter) {
      CUTLASS_CHECK(gemm.initialize(arguments, workspace));
      CUTLASS_CHECK(gemm.run());
    }
    timer.stop();

    // Compute average runtime and GFLOPs.
    float elapsed_ms = timer.elapsed_millis();
    result.avg_runtime_ms = double(elapsed_ms) / double(options.iterations);
    result.gflops = options.gflops(result.avg_runtime_ms / 1000.0);
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Targets the newer sm103 narrow-precision path.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。面向更新的 SM103 窄精度路径。

---

```cpp
    std::cout << "  Problem Size: " << options.m << 'x' << options.n << 'x' << options.k << " (batch: " << options.batch << ")" << std::endl;
    std::cout << "  Avg runtime: " << result.avg_runtime_ms << " ms" << std::endl;
    std::cout << "  GFLOPS: " << result.gflops << std::endl;
  }

  return 0;
}

#endif // defined(CUTLASS_ARCH_MMA_SM103_SUPPORTED)
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Targets the newer sm103 narrow-precision path.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。面向更新的 SM103 窄精度路径。

---

```cpp
int main(int argc, char const **args) {

  // CUTLASS must be compiled with CUDA 12.9 or higher Toolkit to run this example
  // and must have compute capability at least 100.
  if (__CUDACC_VER_MAJOR__ < 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ < 9)) {
    std::cerr << "This example requires CUDA 12.9 or newer." << std::endl;
    // Returning zero so this test passes on older Toolkits. Its actions are no-op.
    return 0;
  }

  cudaDeviceProp props;
  int current_device_id;
  CUDA_CHECK(cudaGetDevice(&current_device_id));

  CUDA_CHECK(cudaGetDeviceProperties(&props, current_device_id));

  if (!(props.major == 10 && props.minor == 3)) {
    std::cerr << "This example requires a GPU of NVIDIA's Blackwell architecture (compute capability 103)." << std::endl;
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

  //
  // Evaluate CUTLASS kernels
  //
```

**EN**: This block forms the outer control flow of the sample. It performs capability checks, selects the concrete specialization to run, invokes the operation, and reports the final status or throughput back to the user. Targets the newer sm103 narrow-precision path.  
**CN**: 这一块构成了示例的最外层控制流：先做能力检查，再选择具体特化版本，调用 operation，最后把状态或吞吐结果回报给用户。面向更新的 SM103 窄精度路径。

---

```cpp
#if defined(CUTLASS_ARCH_MMA_SM103_SUPPORTED)
  std::cout << "Running kernel with 1SM MMA config:" << std::endl;
  run<Gemm1Sm>(options);
  std::cout << "Running kernel with 2SM MMA config:" << std::endl;
  run<Gemm2Sm>(options);
#endif // defined(CUTLASS_ARCH_MMA_SM103_SUPPORTED)

  return 0;
}
```

**EN**: This block forms the outer control flow of the sample. It performs capability checks, selects the concrete specialization to run, invokes the operation, and reports the final status or throughput back to the user. Targets the newer sm103 narrow-precision path.  
**CN**: 这一块构成了示例的最外层控制流：先做能力检查，再选择具体特化版本，调用 operation，最后把状态或吞吐结果回报给用户。面向更新的 SM103 窄精度路径。

---

## Key Concepts / 关键概念

- Architecture-tagged specialization (`Sm100`/`Sm103`/`Sm120`) / 基于架构标签的特化（`Sm100`/`Sm103`/`Sm120`）
- CUTLASS collective builders that deduce mainloop and epilogue implementations / 推导 mainloop 与 epilogue 实现的 CUTLASS collective 构建器
- Cluster-shape tuning and possible multi-SM cooperation / Cluster 形状调优与可能的多 SM 协作
- Block-scaled or FP4-class narrow precision data paths / 分块缩放或 FP4 类窄精度数据路径
- FP8-family mixed-precision execution / FP8 家族混合精度执行
- TMA-driven data movement / 基于 TMA 的数据搬运

## Dependencies / 依赖项

- `iostream` — included dependency used by this example / 此示例使用的包含依赖
- `cutlass/cutlass.h` — core CUTLASS types, architecture tags, and utilities / CUTLASS 核心类型、架构标签与工具
- `cute/tensor.hpp` — CuTe tensor and layout primitives / CuTe 张量与布局原语
- `cutlass/tensor_ref.h` — tensor reference wrappers / 张量引用封装
- `cutlass/epilogue/thread/linear_combination.h` — default linear-combination epilogue operator / 默认线性组合尾声算子
- `cutlass/gemm/dispatch_policy.hpp` — kernel scheduling and dispatch policy tags / 内核调度与分发策略标签
- `cutlass/gemm/collective/collective_builder.hpp` — GEMM collective mainloop builder / GEMM collective 主循环构建器
- `cutlass/epilogue/collective/collective_builder.hpp` — epilogue collective builder / 尾声 collective 构建器
- `cutlass/detail/sm100_blockscaled_layout.hpp` — SM100/SM103 block-scaled layout definitions / SM100/SM103 分块缩放布局定义
- `cutlass/gemm/device/gemm_universal_adapter.h` — device-side universal GEMM launcher adapter / 设备侧通用 GEMM 启动适配器
- `cutlass/gemm/kernel/gemm_universal.hpp` — universal GEMM kernel composition / 通用 GEMM 内核组合
- `cutlass/gemm/kernel/tile_scheduler_params.h` — tile scheduler runtime parameters / tile 调度器运行参数
- `cutlass/util/command_line.h` — command-line parsing helpers / 命令行解析辅助
- `cutlass/util/distribution.h` — random/distribution-based tensor initialization / 随机与分布初始化辅助
- `cutlass/util/host_tensor.h` — host/device tensor allocation utilities / host/device 张量分配工具
- `cutlass/util/packed_stride.hpp` — packed stride helpers / 紧凑 stride 辅助
- `cutlass/util/tensor_view_io.h` — tensor printing and inspection utilities / 张量打印与查看工具
- `cutlass/util/reference/device/gemm.h` — device reference GEMM / 设备参考 GEMM
- `cutlass/util/reference/device/tensor_compare.h` — device-side tensor comparison helpers / 设备侧张量比较辅助
- `cutlass/util/reference/host/tensor_fill.h` — host tensor fill helpers / 主机张量填充辅助
- `...` — additional direct includes omitted for brevity / 其余直接包含头文件因篇幅省略
