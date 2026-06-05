# 92_blackwell_moe_gemm_fp4_grouped.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/92_blackwell_moe_gemm/92_blackwell_moe_gemm_fp4_grouped.cu`  
**Purpose / 用途**: Shows grouped Blackwell NVFP4 MoE GEMM, combining per-expert problem lists with packed FP4 payloads, scale-factor metadata, and grouped scheduling. / 展示 grouped Blackwell NVFP4 MoE GEMM：把按 expert 划分的问题列表与打包 FP4 载荷、缩放元数据以及 grouped 调度结合起来。

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
  \brief Example of Blackwell MoE-style grouped NVFP4 GEMM implementation using TMA to load A and CPASYNC to load B.

  This example demonstrates an implementation of GEMM using mixed TMA+CPASYNC to load input matrices.
  In the decoding stage of Mixture of Experts (MoE) models, the number of tokens in different experts 
  can varies a lot, which requires frequently updates of TMA descriptors in TMA-based implementation.
  This examples uses CPASYNC to load activation (B) matrix to avoid the overhead of updating TMA descriptors.
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Handles fp4/nvfp4-style narrow precision. Relies on tma-style bulk movement or descriptor handling.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。处理 FP4/NVFP4 一类窄精度数据。 依赖 TMA 风格的批量搬运或描述符处理。

---

```cpp
  Usage:
  $ ./examples/92_blackwell_moe_gemm/92_blackwell_moe_gemm_fp4_grouped
  --m=7168 --n=128 --k=512 --group=8 --benchmark=benchmark.txt

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

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Targets blackwell sm100 execution paths. Handles fp4/nvfp4-style narrow precision.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。面向 Blackwell SM100 执行路径。 处理 FP4/NVFP4 一类窄精度数据。

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

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
#include "cutlass/util/command_line.h"
#include "cutlass/util/distribution.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/packed_stride.hpp"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/reference/device/gemm_complex.h"
#include "cutlass/util/reference/device/tensor_compare.h"
#include "cutlass/util/reference/device/tensor_fill.h"
#include "cutlass/util/reference/device/gemm.h"
#include "cutlass/util/reference/device/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/gett.hpp"
#include "cutlass/util/reference/host/tensor_norm.h"
#include "cutlass/util/reference/host/tensor_compare.h"
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
#include "helper.h"
```

**EN**: This include block pulls in the CUTLASS/CuTe and utility headers needed for the example. In practice it establishes the compile-time vocabulary for layouts, collectives, kernel wrappers, and host-side helpers (e.g. helper.h). Is tuned for moe-style irregular expert workloads.  
**CN**: 这一组 `#include` 引入了示例所需的 CUTLASS/CuTe 与工具头文件，建立布局、collective、内核封装以及主机侧辅助函数的编译期词汇表（例如 helper.h）。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
using namespace cute;
```

**EN**: This configuration block defines compile-time aliases or constants that control data types, layouts, alignments, tile sizes, or enum-style modes. These choices are not cosmetic: in CUTLASS they steer which iterators, MMA atoms, and memory movement strategies become legal and efficient. Is tuned for moe-style irregular expert workloads.  
**CN**: 这个配置块定义了编译期别名或常量，用来控制数据类型、布局、对齐、tile 大小或枚举模式。这些选择并非表面参数：在 CUTLASS 中，它们会直接决定哪些迭代器、MMA atom 和数据搬运策略既合法又高效。针对 MoE 风格的不规则 expert 工作负载进行调优。

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

    out << "92_blackwell_moe_gemm_fp4_grouped\n\n"
      << "  Blackwell MoE-style grouped NVFP4 GEMM implementation using TMA to load A and CPASYNC to load B\n\n"
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

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Handles fp4/nvfp4-style narrow precision. Relies on tma-style bulk movement or descriptor handling.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。处理 FP4/NVFP4 一类窄精度数据。 依赖 TMA 风格的批量搬运或描述符处理。

---

```cpp
template <class Element, class Layout>
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
    if constexpr (cute::is_same_v<Element, cutlass::float_ue8m0_t> || cute::is_same_v<Element, cutlass::float_ue4m3_t>) {
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

template <class T>
auto make_iterator(T* ptr) {
  return cute::recast_ptr<T>(ptr);
}
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Includes fp8/mxfp8-style narrow-precision behavior. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。包含 FP8/MXFP8 一类窄精度行为。 针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
template <
  // Type of kernel schedule to generate
  class MainloopScheduleType = cutlass::gemm::KernelMixedTmaCpAsyncWarpSpecialized1SmBlockScaledSm100,
  // Type of epilogue schedule to generate
  class EpilogueScheduleType = cutlass::epilogue::collective::EpilogueScheduleAuto,
  class ClusterShapeMNK = Shape<_1, _1, _1>,
  bool FuseQuantization = false
>
struct ExampleRunner {

  using LayoutATag = cutlass::layout::RowMajor;
  using LayoutBTag = cutlass::layout::ColumnMajor;
  using LayoutCTag = cutlass::layout::ColumnMajor;
  using LayoutDTag = cutlass::layout::ColumnMajor;
  using LayoutSFDTag = LayoutDTag;                                    // Layout type for SFD should be same as D matrix operand

  using ElementInput = cutlass::float_e2m1_t;                                // Element type for Input matrix operands
  using ElementSF    = cutlass::float_ue4m3_t;                               // Element type for SF matrix operands

  using ElementA = cutlass::nv_float4_t<ElementInput>;
  using ElementB = cutlass::nv_float4_t<ElementInput>;
  using ElementC = cutlass::half_t;
  using ElementD = cute::conditional_t<FuseQuantization, ElementInput, ElementC>;
  using ElementSFD = ElementSF;
  using ElementAccumulator = float;
  using ElementCompute = float;
  using ElementScalar = float;

  static constexpr int TileM = cute::is_base_of_v<cutlass::gemm::KernelSchedule2Sm, MainloopScheduleType> ? 256 : 128;
  using MmaTileMNK    = Shape<Int<TileM>,_64,_256>;  // use tile size of N=64 to match real use cases (N is typically very small in decoding stage)

  static constexpr int AlignmentA = 32;
  static constexpr int AlignmentB = 32;
  static constexpr int AlignmentC = 128 / cutlass::sizeof_bits<ElementC>::value;
  static constexpr int AlignmentD = 128 / cutlass::sizeof_bits<ElementD>::value;

  static constexpr int OutputSFVectorSize = 16;

  // D = alpha * acc + beta * C
  //      With BlockScaleFactor generation.
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Targets blackwell sm100 execution paths. Handles fp4/nvfp4-style narrow precision.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。面向 Blackwell SM100 执行路径。 处理 FP4/NVFP4 一类窄精度数据。

---

```cpp
  using FusionOperation = cutlass::epilogue::fusion::LinCombBlockScaleFactor<
      OutputSFVectorSize,
      ElementD,
      ElementCompute,
      ElementSFD, LayoutSFDTag,
      ElementC>;

  using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
      cutlass::arch::Sm100, cutlass::arch::OpClassBlockScaledTensorOp,
      MmaTileMNK, ClusterShapeMNK,
      cutlass::epilogue::collective::EpilogueTileAuto,
      ElementAccumulator, ElementCompute,
      ElementC, LayoutCTag, AlignmentC,
      ElementD, LayoutDTag, AlignmentD,
      EpilogueScheduleType,
      cute::conditional_t<
        FuseQuantization, 
        FusionOperation, 
        cutlass::epilogue::fusion::LinearCombination<ElementC, ElementAccumulator>>
    >::CollectiveOp;

  using CollectiveMainloop = typename cutlass::gemm::collective::CollectiveBuilder<
      cutlass::arch::Sm100, cutlass::arch::OpClassBlockScaledTensorOp,
      ElementA, LayoutATag, AlignmentA,
      ElementB, LayoutBTag, AlignmentB,
      ElementAccumulator,
      MmaTileMNK, ClusterShapeMNK,
      cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))>,
      MainloopScheduleType
    >::CollectiveOp;

  using ProblemShape = cutlass::gemm::MoEProblemShape<Shape<int,int,int>>; // <M,N,K> per group

  using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
      ProblemShape,
      CollectiveMainloop,
      CollectiveEpilogue
  >;

  using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Targets blackwell sm100 execution paths. Selects block-scaled tensor-core instructions, so data tensors and scale tensors travel together.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。面向 Blackwell SM100 执行路径。 选择分块缩放 Tensor Core 指令，因此数据张量与缩放张量需要协同传递。

---

```cpp
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

  using FusionOp = typename Gemm::EpilogueOutputOp;
  static constexpr bool IsBlockScaleSupported = FusionOp::IsBlockScaleSupported;
  using SfdOutputCfg = cutlass::detail::Sm1xxBlockScaledOutputConfig<OutputSFVectorSize>;
  using LayoutSFD = typename SfdOutputCfg::LayoutSF;

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
  LayoutSFD layout_SFD;
  uint64_t seed = 0;
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
  cutlass::HostTensor<ElementA::DataType, cutlass::layout::PackedVectorLayout> block_A;
  cutlass::HostTensor<ElementA::ScaleFactorType, cutlass::layout::PackedVectorLayout> block_SFA;
  cutlass::HostTensor<ElementB::DataType, cutlass::layout::PackedVectorLayout> block_B;
  cutlass::HostTensor<ElementB::ScaleFactorType, cutlass::layout::PackedVectorLayout> block_SFB;
  cutlass::HostTensor<ElementC, cutlass::layout::PackedVectorLayout> block_C;
  cutlass::HostTensor<ElementD, cutlass::layout::PackedVectorLayout> block_D;
  cutlass::HostTensor<ElementSFD, cutlass::layout::PackedVectorLayout> block_SFD;
  cutlass::HostTensor<ElementD, cutlass::layout::PackedVectorLayout> block_reference_D;
  cutlass::HostTensor<ElementSFD, cutlass::layout::PackedVectorLayout> block_reference_SFD;
  cutlass::HostTensor<ElementCompute, cutlass::layout::PackedVectorLayout> block_Normconst;

  cutlass::DeviceAllocation<int32_t> tokens_per_expert;
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
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
    // Create the arguments for host reference implementation
    Tensor tensor_A = make_tensor(make_iterator(block_A.host_data()), layout_A);
    Tensor tensor_SFA = make_tensor(block_SFA.host_data(), layout_SFA);
    Tensor tensor_B = make_tensor(make_iterator(block_B.host_data()), layout_B);
    Tensor tensor_SFB = make_tensor(block_SFB.host_data(), layout_SFB);

    // think about how to simplify the gemm3x interface.
    cutlass::reference::host::GettBlockScalingMainloopParams<
        ElementAccumulator,                   // ElementAccumulator
        decltype(tensor_A),                   // TensorA
        decltype(tensor_SFA),                 // TensorSfA
        decltype(tensor_B),                   // TensorB
        decltype(tensor_SFB)                  // TensorSfB
      > mainloop_params{tensor_A, tensor_SFA, tensor_B, tensor_SFB};

    Tensor tensor_C = cute::make_tensor(make_iterator(block_C.host_data()), layout_C);
    Tensor tensor_D = cute::make_tensor(make_iterator(block_reference_D.host_data()), layout_D);
    Tensor tensor_SFD = make_tensor(block_reference_SFD.host_data(), layout_SFD);

    if constexpr (FuseQuantization) {
      cutlass::reference::host::GettBlockScalingEpilogueParams<
          ElementCompute,                       // ElementScalar
          ElementAccumulator,                   // ElementAccumulator
          ElementCompute,                       // ElementCompute
          decltype(tensor_C),                   // TensorC
          decltype(tensor_D),                   // TensorD
          decltype(tensor_SFD),                 // TensorSfD
          cute::Int<OutputSFVectorSize>,
          cutlass::reference::host::SfStrategy::SfDGen
        > epilogue_params {alpha, beta, tensor_C, tensor_D, tensor_SFD, block_Normconst.at(cutlass::make_Coord(0))};
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
      cutlass::reference::host::Gemm3x(mainloop_params, epilogue_params);
    } 
    else {
      cutlass::reference::host::GettBlockScalingEpilogueParams<
          ElementCompute,                       // ElementScalar
          ElementAccumulator,                   // ElementAccumulator
          ElementCompute,                       // ElementCompute
          decltype(tensor_C),                   // TensorC
          decltype(tensor_D)                   // TensorD
        > epilogue_params {alpha, beta, tensor_C, tensor_D };

      cutlass::reference::host::Gemm3x(mainloop_params, epilogue_params);
    }

    bool passed = true;

    // Comparison
    block_D.sync_host();

    auto [maxM, maxN, maxK] = problem_size.get_host_problem_shape(0); //gets max problem shape
    for (int i = 0; i < problem_size.num_groups; i++) {
      auto problem = problem_sizes_host.at(i);
      auto [M, N, K] = problem;
      printf("group [%d] : M = %d, N = %d, K = %d\n", i, M, N, K);

      // assume all M == maxM
      auto refD_view = block_reference_D.host_view().subview(cutlass::make_Coord(M * N), cutlass::make_Coord(i * maxN * maxM));
      auto D_view = block_D.host_view().subview(cutlass::make_Coord(M * N), cutlass::make_Coord(i * maxN * maxM));

      passed &= cutlass::reference::host::TensorEquals(refD_view, D_view);
    }

    return passed;
  }

  /// Initialize operands to be used in the GEMM and reference GEMM
  void initialize(ProblemShape const& problem_size) {
    auto problem_shape_MNKL = cute::append<4>(problem_size.get_host_problem_shape(0), problem_size.groups());
    auto [M, N, K, L] = problem_shape_MNKL;

    // For SFA and SFB tensors layouts
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
    using Sm1xxBlkScaledConfig = typename Gemm::GemmKernel::CollectiveMainloop::Sm1xxBlkScaledConfig;
    // For SFD tensor layout
    using Sm1xxBlockScaledOutputConfig = typename Gemm::GemmKernel::CollectiveMainloop::Sm1xxBlkScaledConfig;

    stride_A = cutlass::make_cute_packed_stride(StrideA{}, {M, K, L});
    stride_B = cutlass::make_cute_packed_stride(StrideB{}, {N, K, L});
    stride_C = cutlass::make_cute_packed_stride(StrideC{}, {M, N, L});
    stride_D = cutlass::make_cute_packed_stride(StrideD{}, {M, N, L});

    layout_A = make_layout(make_shape(M, K, L), stride_A);
    layout_B = make_layout(make_shape(N, K, L), stride_B);
    layout_C = make_layout(make_shape(M, N, L), stride_C);
    layout_D = make_layout(make_shape(M, N, L), stride_D);
    layout_SFA = Sm1xxBlkScaledConfig::tile_atom_to_shape_SFA(cute::make_shape(M, N, K, L));
    layout_SFB = Sm1xxBlkScaledConfig::tile_atom_to_shape_SFB(cute::make_shape(M, N, K, L));
    layout_SFD = SfdOutputCfg::tile_atom_to_shape_SFD(cute::make_shape(M, N, K, L));

    block_A.reset(cutlass::make_Coord(size(layout_A)));
    block_B.reset(cutlass::make_Coord(size(layout_B)));
    block_C.reset(cutlass::make_Coord(size(layout_C)));
    block_D.reset(cutlass::make_Coord(size(layout_D)));
    block_reference_D.reset(cutlass::make_Coord(size(layout_D)));
    block_reference_SFD.reset(cutlass::make_Coord(size(filter_zeros(layout_SFD))));
    block_Normconst.reset(cutlass::make_Coord(1));

    block_SFA.reset(cutlass::make_Coord(size(filter_zeros(layout_SFA))));
    block_SFB.reset(cutlass::make_Coord(size(filter_zeros(layout_SFB))));
    block_SFD.reset(cutlass::make_Coord(size(filter_zeros(layout_SFD))));

    initialize_block(block_A.host_view(), seed + 2021);
    initialize_block(block_B.host_view(), seed + 2022);
    initialize_block(block_C.host_view(), seed + 2023);
    initialize_block(block_SFA.host_view(), seed + 2024);
    initialize_block(block_SFB.host_view(), seed + 2025);
    block_Normconst.at(cutlass::make_Coord(0)) = 2;
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
    block_A.sync_device();
    block_B.sync_device();
    block_C.sync_device();
    block_D.sync_device();
    block_SFA.sync_device();
    block_SFB.sync_device();
    block_SFD.sync_device();
    block_Normconst.sync_device();
  }

  /// Load a benchmark
  void benchmark_problems(std::string const& benchmark_path) {

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
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
    return;
  }

  void benchmark_tokens_per_expert(std::string const& benchmark_path) {

    std::ifstream file(benchmark_path);
    if (!file.good()) {
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
      tokens_per_expert_host.push_back(extent.n());
    }

    return;
  }

  bool run(Options const& options, cutlass::KernelHardwareInfo const& hw_info) {

    benchmark_problems(options.benchmark_path);
    if (problem_sizes_host.empty()) {
      return false;
    }
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
    benchmark_tokens_per_expert(options.benchmark_path);
    if (tokens_per_expert_host.empty()) {
      return false;
    }

    tokens_per_expert.reset(tokens_per_expert_host.size());
    tokens_per_expert.copy_from_host(tokens_per_expert_host.data());

    ProblemShape problem_size {options.m, options.n, options.k, options.groups, tokens_per_expert.get(), tokens_per_expert_host.data()};

    initialize(problem_size);

    typename Gemm::Arguments arguments{
      cutlass::gemm::GemmUniversalMode::kGrouped,
      problem_size,
      { // Mainloop arguments
        block_A.device_data(),
        block_B.device_data(),
        block_SFA.device_data(),
        block_SFB.device_data()
      },
      { // Epilogue arguments
        {},
        block_C.device_data(), stride_C,
        block_D.device_data(), stride_D
      },
      hw_info
    };

    auto f = [&](auto blockscale) {
      auto impl = [this](auto& arguments) {
        arguments.epilogue.thread.block_scale_factor_ptr = block_SFD.device_data();
        arguments.epilogue.thread.norm_constant_ptr      = block_Normconst.device_data();
      };
      if constexpr (decltype(blockscale)::value) {
        impl(arguments);
      }
    };
    f(std::bool_constant<IsBlockScaleSupported>());

    // arguments.scheduler.max_swizzle_size = options.swizzle;
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
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

    cudaError_t result = cudaDeviceSynchronize();
    if (result != cudaSuccess) {
      std::cerr << "Error running the CUTLASS kernel. Last CUDA error is: "
                << cudaGetErrorString(result) << std::endl;
      return false;
    }

    if (options.verification) {
      // Verify that the result is correct
      bool passed = verify(problem_size, 1.0f, 0.0f);
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
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

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
};
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。针对 MoE 风格的不规则 expert 工作负载进行调优。

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

  std::cout << "\n\n\nRunning kernel with mixed TMA+CPASYNC load, 1SM, 2x2 cluster:" << std::endl;
  ExampleRunner<
    cutlass::gemm::KernelMixedTmaCpAsyncWarpSpecialized1SmBlockScaledSm100,
    cutlass::epilogue::collective::EpilogueScheduleAuto,
    Shape<_2, _2, _1>
  > runner_mixed_tma_cpasync_1sm_2x2;
  runner_mixed_tma_cpasync_1sm_2x2.run(options, hw_info);

  std::cout << "\n\n\nRunning 2SM kernel with mixed TMA+CPASYNC load, 2x1 cluster:" << std::endl;
  ExampleRunner<
    cutlass::gemm::KernelMixedTmaCpAsyncWarpSpecialized2SmBlockScaledSm100,
    cutlass::epilogue::collective::EpilogueScheduleAuto,
    Shape<_2, _1, _1>
  > runner_mixed_tma_cpasync_2sm_2x1;
  runner_mixed_tma_cpasync_2sm_2x1.run(options, hw_info);

  std::cout << "\n\n\nRunning 2SM kernel with mixed TMA+CPASYNC load, 2x4 cluster:" << std::endl;
  ExampleRunner<
    cutlass::gemm::KernelMixedTmaCpAsyncWarpSpecialized2SmBlockScaledSm100,
    cutlass::epilogue::collective::EpilogueScheduleAuto,
    Shape<_2, _4, _1>
  > runner_mixed_tma_cpasync_2sm_2x4;
  runner_mixed_tma_cpasync_2sm_2x4.run(options, hw_info);

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
- Block-scaled or FP4-class narrow precision data paths / 分块缩放或 FP4 类窄精度数据路径
- FP8-family mixed-precision execution / FP8 家族混合精度执行
- TMA-driven data movement / 基于 TMA 的数据搬运

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
- `cutlass/detail/sm100_blockscaled_layout.hpp` — SM100/SM103 block-scaled layout definitions / SM100/SM103 分块缩放布局定义
- `cutlass/gemm/kernel/tile_scheduler_params.h` — tile scheduler runtime parameters / tile 调度器运行参数
- `cutlass/util/command_line.h` — command-line parsing helpers / 命令行解析辅助
- `cutlass/util/distribution.h` — random/distribution-based tensor initialization / 随机与分布初始化辅助
- `cutlass/util/host_tensor.h` — host/device tensor allocation utilities / host/device 张量分配工具
- `cutlass/util/packed_stride.hpp` — packed stride helpers / 紧凑 stride 辅助
- `cutlass/util/tensor_view_io.h` — tensor printing and inspection utilities / 张量打印与查看工具
- `...` — additional direct includes omitted for brevity / 其余直接包含头文件因篇幅省略
