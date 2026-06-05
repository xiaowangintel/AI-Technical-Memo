# 92_blackwell_moe_gemm_rcgrouped.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/92_blackwell_moe_gemm/92_blackwell_moe_gemm_rcgrouped.cu`  
**Purpose / 用途**: Demonstrates Blackwell ragged-contiguous grouped MoE GEMM, where one operand is packed contiguously across groups while other operands remain per-group pointer arrays. / 演示 Blackwell ragged-contiguous grouped MoE GEMM：其中一个操作数在各组之间连续打包，而其他操作数仍保持按组指针数组形式。

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
```

**EN**: This comment block frames the file: it usually states the target architecture, problem family, usage pattern, or design motivation before the code starts. That context matters because later template choices are easier to understand once the intended hardware path and workload are known. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一段注释用于给文件定调：通常先说明目标架构、问题类型、使用方式或设计动机，再进入具体代码。先理解这些背景，再看后面的模板选择与硬件路径会更清晰。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
/*! \file
    \brief Ragged Contiguous Grouped GEMM example using CUTLASS 3 APIs for the NVIDIA Blackwell SM100 architecture.

    This example demonstrates an implementation of Ragged Contiguous Grouped GEMM using a TMA+TMA Blackwell 
    SM100 TensorOp-based warp-specialized kernel. In Ragged Contiguous Grouped Gemms, weights are of same size for each group, 
    i.e. matrix A, whereas Activations differ in shape and stride between groups. Therefore, we used a Batched TMA Load 
    to load Weights matrix into the shared memory for MMA. The Activations are loaded using a PtrArray logic with updates to TMA descriptors.

    To run this example:

      $ ./examples/92_blackwell_moe_gemm/92_blackwell_moe_gemm_rcgrouped --m=128 --k=128 --groups=10

      The above example command makes all 10 groups to be sized at the given m, n, k sizes.
      Skipping any of the problem dimensions randomizes it across the different groups.
      Same applies for alpha and beta values that are randomized across the different groups.

    To run this example for a set of problems using the benchmark option:

      $ ./examples/92_blackwell_grouped_gemm/92_blackwell_moe_gemm_rcgrouped --benchmark=./test_benchmark.txt

      Where the test_benchmark.txt may look as such:
        0 256x512x128
        1 256x512x512
        2 512x256x128
        3 256x256x128
        4 256x512x1024
        5 1024x512x128 and so on
*/

#include <iostream>
#include <fstream>
#include <sstream>
#include <vector>
#include <random>
#include <algorithm>
#include <float.h>

#include "cutlass/cutlass.h"
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Targets blackwell sm100 execution paths. Relies on tma-style bulk movement or descriptor handling.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。面向 Blackwell SM100 执行路径。 依赖 TMA 风格的批量搬运或描述符处理。

---

```cpp
#include "cute/tensor.hpp"
#include "cutlass/tensor_ref.h"
#include "cutlass/epilogue/collective/default_epilogue.hpp"
#include "cutlass/epilogue/thread/linear_combination.h"
#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/gemm/group_array_problem_shape.hpp"
#include "cutlass/gemm/collective/collective_builder.hpp"
#include "cutlass/epilogue/collective/collective_builder.hpp"
#include "cutlass/gemm/device/gemm_universal_adapter.h"
#include "cutlass/gemm/kernel/gemm_universal.hpp"

#include "cutlass/util/command_line.h"
#include "cutlass/util/distribution.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/packed_stride.hpp"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/reference/device/gemm.h"
#include "cutlass/util/reference/device/tensor_compare.h"
#include "cutlass/util/reference/device/tensor_fill.h"

#include "helper.h"
using namespace cute;

using ProblemShape = cutlass::gemm::MoEProblemShape<Shape<int,int,int>>; // <M,N,K> per group

using ElementA = cutlass::float_e4m3_t;                                    // Element type for A matrix operand
using ElementB = cutlass::float_e4m3_t;                                    // Element type for B matrix operand
using ElementC = cutlass::half_t;                                          // Element type for C and D matrix operands
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Includes fp8/mxfp8-style narrow-precision behavior. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。包含 FP8/MXFP8 一类窄精度行为。 针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
#if defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Targets blackwell sm100 execution paths. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。面向 Blackwell SM100 执行路径。 针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
/// GEMM kernel configurations
```

**EN**: This configuration block defines compile-time aliases or constants that control data types, layouts, alignments, tile sizes, or enum-style modes. These choices are not cosmetic: in CUTLASS they steer which iterators, MMA atoms, and memory movement strategies become legal and efficient. Is tuned for moe-style irregular expert workloads.  
**CN**: 这个配置块定义了编译期别名或常量，用来控制数据类型、布局、对齐、tile 大小或枚举模式。这些选择并非表面参数：在 CUTLASS 中，它们会直接决定哪些迭代器、MMA atom 和数据搬运策略既合法又高效。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
// A matrix configuration
using         LayoutA     = cutlass::layout::RowMajor;                      // Layout type for A matrix operand
constexpr int AlignmentA  = 128 / cutlass::sizeof_bits<ElementA>::value;    // Alignment of A matrix in units of elements (up to 16 bytes)

// B matrix configuration
using         LayoutB     = cutlass::layout::ColumnMajor;                   // Layout type for B matrix operand
constexpr int AlignmentB  = 128 / cutlass::sizeof_bits<ElementB>::value;    // Alignment of B matrix in units of elements (up to 16 bytes)

// C/D matrix configuration
using         LayoutC     = cutlass::layout::ColumnMajor;                   // Layout type for C and D matrix operands
constexpr int AlignmentC  = 128 / cutlass::sizeof_bits<ElementC>::value;    // Alignment of C matrix in units of elements (up to 16 bytes)

// Core kernel configurations
using ElementAccumulator  = float;                                          // Element type for internal accumulation
using ArchTag             = cutlass::arch::Sm100;                           // Tag indicating the minimum SM that supports the intended feature
using OperatorClass       = cutlass::arch::OpClassTensorOp;                 // Operator class tag
using StageCountType = cutlass::gemm::collective::StageCountAuto;           // Stage count maximized based on the tile size

// Runtime Cluster Shape
using ClusterShape = Shape<int32_t,int32_t,_1>;

// Different configs for 1SM and 2SM MMA kernel
struct MMA1SMConfig {
  using MmaTileShape     = Shape<_128,_128,_64>;
  using KernelSchedule   = cutlass::gemm::KernelPtrArrayTmaWarpSpecialized1SmSm100;   // Kernel to launch
  using EpilogueSchedule = cutlass::epilogue::PtrArrayTmaWarpSpecialized1Sm;          // Epilogue to launch
};

struct MMA2SMConfig {
  using MmaTileShape     = Shape<_256,_256,_64>;
  using KernelSchedule   = cutlass::gemm::KernelPtrArrayTmaWarpSpecialized2SmSm100;   // Kernel to launch
  using EpilogueSchedule = cutlass::epilogue::PtrArrayTmaWarpSpecialized2Sm;          // Epilogue to launch
};
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Targets blackwell sm100 execution paths. Selects tensor-core mma execution instead of a simt fallback.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。面向 Blackwell SM100 执行路径。 选择 Tensor Core MMA 执行，而不是 SIMT 回退路径。

---

```cpp
template <typename ScheduleConfig>
struct GivenGemmSchedule {
  using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    typename ScheduleConfig::MmaTileShape, ClusterShape,
    cutlass::epilogue::collective::EpilogueTileAuto,
    ElementAccumulator, ElementAccumulator,
    ElementC, LayoutC *, AlignmentC,
    ElementC, LayoutC *, AlignmentC,
    typename ScheduleConfig::EpilogueSchedule,
    cutlass::epilogue::fusion::LinearCombination<ElementC, ElementAccumulator>
  >::CollectiveOp;

  using CollectiveMainloop = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
      ElementA, LayoutA, AlignmentA,
      ElementB, LayoutB *, AlignmentB,
    ElementAccumulator,
    typename ScheduleConfig::MmaTileShape, ClusterShape,
    cutlass::gemm::collective::StageCountAutoCarveout<
      static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))>,
    typename ScheduleConfig::KernelSchedule
  >::CollectiveOp;

  using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
      ProblemShape,
      CollectiveMainloop,
      CollectiveEpilogue
  >;

  using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;
};

using GemmKernel1SM = GivenGemmSchedule<MMA1SMConfig>::GemmKernel;
using Gemm1SM = GivenGemmSchedule<MMA1SMConfig>::Gemm;
using Gemm = Gemm1SM;

using GemmKernel2SM = GivenGemmSchedule<MMA2SMConfig>::GemmKernel;
using Gemm2SM = GivenGemmSchedule<MMA2SMConfig>::Gemm;

// Reference device GEMM implementation type
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Makes cluster geometry an explicit compile-time tuning knob. Carves shared memory between pipeline stages and epilogue/shared storage.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。把 cluster 几何作为显式的编译期调优参数。 在流水线阶段与尾声/共享存储之间切分共享内存。

---

```cpp
using DeviceGemmReference = cutlass::reference::device::Gemm<
  ElementA,
  LayoutA,
  ElementB,
  LayoutB,
  ElementC,
  LayoutC,
  ElementAccumulator,
  ElementAccumulator>;

using StrideA = cutlass::detail::TagToStrideA_t<LayoutA>;
using StrideB = typename Gemm::GemmKernel::InternalStrideB;
using StrideC = typename Gemm::GemmKernel::InternalStrideC;
using StrideD = typename Gemm::GemmKernel::InternalStrideD;

// Host-side allocations
std::vector<int64_t> offset_B;
std::vector<int64_t> offset_C;
std::vector<int64_t> offset_D;

std::vector<StrideC> stride_C_host;
std::vector<StrideD> stride_D_host;

std::vector<ElementAccumulator> alpha_host;
std::vector<ElementAccumulator> beta_host;

// Device-side allocations
cutlass::DeviceAllocation<int32_t> tokens_per_expert;

cutlass::DeviceAllocation<typename Gemm::ElementA> block_A;
cutlass::DeviceAllocation<typename Gemm::ElementB> block_B;
cutlass::DeviceAllocation<typename Gemm::ElementC> block_C;
cutlass::DeviceAllocation<typename Gemm::EpilogueOutputOp::ElementOutput> block_D;
cutlass::DeviceAllocation<typename Gemm::EpilogueOutputOp::ElementOutput> block_ref_D;

cutlass::HostTensor<typename Gemm::EpilogueOutputOp::ElementOutput, cutlass::layout::PackedVectorLayout> tensor_D;
cutlass::HostTensor<typename Gemm::EpilogueOutputOp::ElementOutput, cutlass::layout::PackedVectorLayout> tensor_ref_D;
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
cutlass::DeviceAllocation<const typename Gemm::ElementB *> ptr_B;
cutlass::DeviceAllocation<const typename Gemm::ElementC *> ptr_C;
cutlass::DeviceAllocation<typename Gemm::EpilogueOutputOp::ElementOutput *> ptr_D;
cutlass::DeviceAllocation<typename Gemm::EpilogueOutputOp::ElementOutput *> ptr_ref_D;

cutlass::DeviceAllocation<StrideC> stride_C;
cutlass::DeviceAllocation<StrideD> stride_D;

// Note, this is an array of pointers to alpha and beta scaling values per group
cutlass::DeviceAllocation<ElementAccumulator*> alpha_device;
cutlass::DeviceAllocation<ElementAccumulator*> beta_device;
cutlass::DeviceAllocation<ElementAccumulator> block_alpha;
cutlass::DeviceAllocation<ElementAccumulator> block_beta;
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
#endif // defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Targets blackwell sm100 execution paths. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。面向 Blackwell SM100 执行路径。 针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
/// Testbed utility types
```

**EN**: This configuration block defines compile-time aliases or constants that control data types, layouts, alignments, tile sizes, or enum-style modes. These choices are not cosmetic: in CUTLASS they steer which iterators, MMA atoms, and memory movement strategies become legal and efficient. Is tuned for moe-style irregular expert workloads.  
**CN**: 这个配置块定义了编译期别名或常量，用来控制数据类型、布局、对齐、tile 大小或枚举模式。这些选择并非表面参数：在 CUTLASS 中，它们会直接决定哪些迭代器、MMA atom 和数据搬运策略既合法又高效。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
using RasterOrderOptions = cutlass::gemm::kernel::detail::RasterOrderOptions;
// Command line options parsing
struct Options {

  bool help = false;
  bool use_pdl = false;
  bool sparse_test = false;

  float alpha = FLT_MAX;
  float beta  = FLT_MAX;
  int iterations = 1000;
  int warmup = 1000;
  int m = 128, n = 128, k = 128, groups = 10;
  double sparse_prob = 0.1;
  dim3 cluster_shape = dim3(4,2,1);
  dim3 cluster_shape_fallback = dim3(2,1,1);
  RasterOrderOptions raster_order = RasterOrderOptions::AlongM;
  int max_sm_count = INT_MAX;
  std::string benchmark_path;
  std::vector<int32_t> tokens_per_expert_host;
  std::vector<typename ProblemShape::UnderlyingProblemShape> problem_sizes_host;
  int const tma_alignment_bits = 128;
  int const alignment = tma_alignment_bits / cutlass::sizeof_bits<ElementA>::value;

  // Parses the command line
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);

    if (cmd.check_cmd_line_flag("help")) {
      help = true;
      return;
    }
    if (cmd.check_cmd_line_flag("use_pdl")) {
      use_pdl = true;
    }
    if (cmd.check_cmd_line_flag("sparse_test")) {
      sparse_test = true;
      cmd.get_cmd_line_argument("sparse_prob", sparse_prob);
    }
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Relies on tma-style bulk movement or descriptor handling. Is tuned for moe-style irregular expert workloads.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。依赖 TMA 风格的批量搬运或描述符处理。 针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
    cmd.get_cmd_line_argument("m", m);
    cmd.get_cmd_line_argument("n", n);
    cmd.get_cmd_line_argument("k", k);
    cmd.get_cmd_line_argument("groups", groups);
    cmd.get_cmd_line_argument("alpha", alpha, FLT_MAX);
    cmd.get_cmd_line_argument("beta",  beta,  FLT_MAX);
    cmd.get_cmd_line_argument("iterations", iterations);
    cmd.get_cmd_line_argument("warmup", warmup);
    cmd.get_cmd_line_argument("benchmark", benchmark_path);
    cmd.get_cmd_line_argument("cluster_m", cluster_shape.x);
    cmd.get_cmd_line_argument("cluster_n", cluster_shape.y);
    cmd.get_cmd_line_argument("cluster_fallback_m", cluster_shape_fallback.x);
    cmd.get_cmd_line_argument("cluster_fallback_n", cluster_shape_fallback.y);
    cmd.get_cmd_line_argument("max_sm_count", max_sm_count, INT_MAX);

    // Decide how to initialize the problems
    if (!benchmark_path.empty()) {
      if (!benchmark_problems()) {
        problem_sizes_host.clear();
        return;
      }
    }
    else if(sparse_test){
      std::cout << "Running sparse test" << std::endl;
      randomize_sparse_problems(cmd, sparse_prob);
    }
    else {
      randomize_problems(cmd);
    }

    char raster_char;
    cmd.get_cmd_line_argument("raster", raster_char);

    if (raster_char == 'N' || raster_char == 'n') {
      raster_order = RasterOrderOptions::AlongN;
    }
    else if (raster_char == 'M' || raster_char == 'm') {
      raster_order = RasterOrderOptions::AlongM;
    }
  }
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Is tuned for moe-style irregular expert workloads.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
  void randomize_problems(cutlass::CommandLine &cmd) {
    int cmd_line_m = -1, cmd_line_n = -1, cmd_line_k = -1;
    cmd.get_cmd_line_argument("m", cmd_line_m);
    cmd.get_cmd_line_argument("n", cmd_line_n);
    cmd.get_cmd_line_argument("k", cmd_line_k);

    problem_sizes_host.reserve(groups);

    m = cmd_line_m;
    k = cmd_line_k;
    if (m < 1) {
      m = alignment * ((rand() % 64) + 1);
    }
    if (k < 1) {
      k = alignment * ((rand() % 64) + 1);
    }

    for (int i = groups; i > 0; i--) {
      int n = cmd_line_n;
      if (n < 0) {
        n = alignment * ((rand() % 64) + 1);
      }
      problem_sizes_host.push_back({m, n, k});
      tokens_per_expert_host.push_back(n);
    }
  }

  void randomize_sparse_problems(cutlass::CommandLine &cmd, double prob) {
    int cmd_line_m = -1, cmd_line_n = -1, cmd_line_k = -1;
    cmd.get_cmd_line_argument("m", cmd_line_m);
    cmd.get_cmd_line_argument("n", cmd_line_n);
    cmd.get_cmd_line_argument("k", cmd_line_k);

    int num_to_set = int(prob * groups);

    std::vector<bool> set_to_zero(groups, false);
    std::vector<int> indices(groups, false);
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Is tuned for moe-style irregular expert workloads.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
    int index=0;
    for(auto it = indices.begin(); it != indices.end(); ++it){
      *it = index++;
    }

    // Shuffle indices
    std::random_device rd;
    std::mt19937 gen(rd());
    std::shuffle(indices.begin(), indices.end(), gen);
    
    // Set first num_to_set entries to true
    for (int i = 0; i < num_to_set; ++i)
      set_to_zero[indices[i]] = true;

    problem_sizes_host.reserve(groups);

    m = cmd_line_m;
    k = cmd_line_k;
    if (m < 1) {
      m = alignment * ((rand() % 64) + 1);
    }
    if (k < 1) {
      k = alignment * ((rand() % 64) + 1);
    }

    for (int i = groups; i > 0; i--) {
      int n = cmd_line_n;
      if (!set_to_zero[i]){
        if (n < 0) {
          n = alignment * ((rand() % 64) + 1);
        }
      } 
      else{
        n = 0;   
      }
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Is tuned for moe-style irregular expert workloads.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
      problem_sizes_host.push_back({m, n, k});
      tokens_per_expert_host.push_back(n);
    }
  }
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Is tuned for moe-style irregular expert workloads.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
  /// Load a benchmark
  bool benchmark_problems() {
    std::ifstream file(benchmark_path);
    if (!file.good()) {
      return false;
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
      tokens_per_expert_host.push_back(extent.n());
    }
    groups = static_cast<int>(problem_sizes_host.size());
    m = get<0>(problem_sizes_host.at(0));
    k = get<2>(problem_sizes_host.at(0));

    return true;
  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
    out << "92_blackwell_moe_gemm_rcgrouped\n\n"
      << "  Blackwell FP8 Grouped GEMM using a Warp Specialized kernel.\n\n"
      << "Options:\n\n"
      << "  --help                                                       If specified, displays this usage statement\n\n"
      << "  --m=<int>                                                    Sets the M extent of the GEMM for all groups\n"
      << "  --n=<int>                                                    Sets the N extent of the GEMM for all groups\n"
      << "  --k=<int>                                                    Sets the K extent of the GEMM for all groups\n"
      << "  --groups=<int>                                               Sets the number of individual GEMM problems for Grouped GEMM\n"
      << "  --alpha=<f32>                                                Epilogue scalar alpha\n"
      << "  --beta=<f32>                                                 Epilogue scalar beta\n\n"
      << "  --cluster_m=<int>          and --cluster_n=<int>             Sets the X,Y dims of the preferred cluster shape\n"
      << "  --cluster_fallback_m=<int> and --cluster_fallback_n=<int>    Sets the X,Y dims of the fallback cluster shape\n\n"
      << "  --raster=<char>                                              CTA Rasterization direction (N for along N, M for along M)\n\n"
      << "  --iterations=<int>                                           Number of profiling iterations to perform\n\n"
      << "  --benchmark=<str>                                            Executes a benchmark problem size\n"
      << "  --max_sm_count=<int>                                         Run kernels using only these number of SMs\n"
      << "  --use_pdl                                                    Launch kernel with PDL (Programmatic Dependent Launch) enabled\n";
                                                                                             
    out
      << "\n\nExamples:\n\n"
      << "$ " << "92_blackwell_moe_gemm_rcgrouped" << " --m=1024 --n=512 --k=1024 --groups=10 --alpha=2 --beta=0.707 \n\n";

    return out;
  }

  /// Compute performance in GFLOP/s
  double gflops(double runtime_s, std::vector<typename ProblemShape::UnderlyingProblemShape> problem_sizes_host) const
  {
    // Number of real-valued multiply-adds
    uint64_t fmas = uint64_t();

    for (auto const & problem : problem_sizes_host) {
      fmas += static_cast<uint64_t>(get<0>(problem)) *
              static_cast<uint64_t>(get<1>(problem)) *
              static_cast<uint64_t>(get<2>(problem));
    }
    // Two flops per multiply-add
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Includes fp8/mxfp8-style narrow-precision behavior. Makes cluster geometry an explicit compile-time tuning knob.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。包含 FP8/MXFP8 一类窄精度行为。 把 cluster 几何作为显式的编译期调优参数。

---

```cpp
    uint64_t flop = uint64_t(2) * uint64_t(fmas);
    double gflop = double(flop) / double(1.0e9);
    return gflop / runtime_s;
  }
};

/// Result structure
struct Result
{
  double avg_runtime_ms = 0.0;
  double gflops = 0.0;
  cutlass::Status status = cutlass::Status::kSuccess;
  cudaError_t error = cudaSuccess;
  bool passed = false;
};
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
#if defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Targets blackwell sm100 execution paths. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。面向 Blackwell SM100 执行路径。 针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
/// GEMM setup and evaluation
```

**EN**: This configuration block defines compile-time aliases or constants that control data types, layouts, alignments, tile sizes, or enum-style modes. These choices are not cosmetic: in CUTLASS they steer which iterators, MMA atoms, and memory movement strategies become legal and efficient. Is tuned for moe-style irregular expert workloads.  
**CN**: 这个配置块定义了编译期别名或常量，用来控制数据类型、布局、对齐、tile 大小或枚举模式。这些选择并非表面参数：在 CUTLASS 中，它们会直接决定哪些迭代器、MMA atom 和数据搬运策略既合法又高效。针对 MoE 风格的不规则 expert 工作负载进行调优。

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
  } else if (bits_input <= 8) {
    scope_max = static_cast<Element>(2);
    scope_min = static_cast<Element>(-2);
  } else {
    scope_max = static_cast<Element>(8);
    scope_min = static_cast<Element>(-8);
  }

  cutlass::reference::device::BlockFillRandomUniform(
    block.get(), block.size(), seed, scope_max, scope_min, 0);

  return true;
}

/// Allocates device-side data
void allocate(const Options &options) {
  int64_t total_elements_B = 0;
  int64_t total_elements_C = 0;
  int64_t total_elements_D = 0;

  for (int32_t i = 0; i < options.groups; ++i) {

    auto problem = options.problem_sizes_host.at(i);
    auto M = get<0>(problem);
    auto N = get<1>(problem);
    auto K = get<2>(problem);
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
    offset_B.push_back(total_elements_B);
    offset_C.push_back(total_elements_C);
    offset_D.push_back(total_elements_D);

    int64_t elements_B = K * N;
    int64_t elements_C = M * N;
    int64_t elements_D = M * N;

    total_elements_B += elements_B;
    total_elements_C += elements_C;
    total_elements_D += elements_D;

    stride_C_host.push_back(cutlass::make_cute_packed_stride(StrideC{}, {M, N, 1}));
    stride_D_host.push_back(cutlass::make_cute_packed_stride(StrideD{}, {M, N, 1}));

  }

  block_B.reset(total_elements_B);
  block_C.reset(total_elements_C);
  block_D.reset(total_elements_D);
  block_ref_D.reset(total_elements_D);
  block_alpha.reset(options.groups);
  block_beta.reset(options.groups);

  auto a_coord = cutlass::make_Coord(options.m * options.groups, options.k);
  block_A.reset(a_coord.product());

}

/// Initialize operands to be used in the GEMM and reference GEMM
void initialize(const Options &options) {

  uint64_t seed = 2020;

  // Setting up tokens_per_expert array
  tokens_per_expert.reset(options.tokens_per_expert_host.size());
  tokens_per_expert.copy_from_host(options.tokens_per_expert_host.data());

  //
  // Assign pointers
  //
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
  std::vector<ElementB *> ptr_B_host(options.groups);
  std::vector<ElementC *> ptr_C_host(options.groups);
  std::vector<ElementC *> ptr_D_host(options.groups);
  std::vector<ElementAccumulator *> ptr_alpha_host(options.groups);
  std::vector<ElementAccumulator *> ptr_beta_host(options.groups);

  for (int32_t i = 0; i < options.groups; ++i) {
    ptr_B_host.at(i) = block_B.get() + offset_B.at(i);
    ptr_C_host.at(i) = block_C.get() + offset_C.at(i);
    ptr_D_host.at(i) = block_D.get() + offset_D.at(i);
    alpha_host.push_back((options.alpha == FLT_MAX) ? static_cast<ElementAccumulator>((rand() % 5) + 1) : options.alpha);
    beta_host.push_back((options.beta == FLT_MAX) ? static_cast<ElementAccumulator>(rand() % 5) : options.beta);
    ptr_alpha_host.at(i) = block_alpha.get() + i;
    ptr_beta_host.at(i) = block_beta.get() + i;
  }

  ptr_B.reset(options.groups);
  ptr_B.copy_from_host(ptr_B_host.data());

  ptr_C.reset(options.groups);
  ptr_C.copy_from_host(ptr_C_host.data());

  ptr_D.reset(options.groups);
  ptr_D.copy_from_host(ptr_D_host.data());

  stride_C.reset(options.groups);
  stride_C.copy_from_host(stride_C_host.data());

  stride_D.reset(options.groups);
  stride_D.copy_from_host(stride_D_host.data());

  alpha_device.reset(options.groups);
  alpha_device.copy_from_host(ptr_alpha_host.data());
  beta_device.reset(options.groups);
  beta_device.copy_from_host(ptr_beta_host.data());
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
  initialize_block(block_A, seed + 2023);
  initialize_block(block_B, seed + 2022);
  initialize_block(block_C, seed + 2021);
  block_alpha.copy_from_host(alpha_host.data());
  block_beta.copy_from_host(beta_host.data());
}

/// Populates a Gemm::Arguments structure from the given commandline options
template <typename Gemm>
typename Gemm::Arguments args_from_options(Options &options)
{
  cutlass::KernelHardwareInfo hw_info;
  // Change device_id to another value if you are running on a machine with multiple GPUs and wish
  // to use a GPU other than that with device ID 0.
  hw_info.device_id = 0;
  hw_info.sm_count = min(cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id), options.max_sm_count);

  if (!is_static_v<ClusterShape>) {
    if (size<0>(typename Gemm::GemmKernel::CollectiveMainloop::AtomThrShapeMNK{}) == 2 &&
        (options.cluster_shape.x < 2 || options.cluster_shape_fallback.x < 2)) {
      std::cout << "Error: MMA2SMConfig kernel config needs cluster_dim.x >= 2" << std::endl;
    }
    hw_info.cluster_shape = options.cluster_shape;
    hw_info.cluster_shape_fallback = options.cluster_shape_fallback;
  }

  typename Gemm::Arguments arguments;
  decltype(arguments.epilogue.thread) fusion_args;
  fusion_args.alpha_ptr = nullptr;
  fusion_args.beta_ptr = nullptr;

  // If alpha/beta are provided (via cmd line args) and are scalar, then same alpha/beta applies to all batches.
  // If pointers to alpha/beta are provided, then alpha/beta can differ between batches/groups.
  if (options.alpha != FLT_MAX){
    // Single alpha for all groups
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Makes cluster geometry an explicit compile-time tuning knob. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。把 cluster 几何作为显式的编译期调优参数。 针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
    fusion_args.alpha = options.alpha;
    fusion_args.alpha_ptr_array = nullptr;
    fusion_args.dAlpha = {_0{}, _0{}, 0};
  }
  else {
    fusion_args.alpha = 0;
    fusion_args.alpha_ptr_array = alpha_device.get();
    // Only one alpha per each group
    fusion_args.dAlpha = {_0{}, _0{}, 1};
  }
  if (options.beta != FLT_MAX) {
    // Single beta for all groups
    fusion_args.beta = options.beta;
    fusion_args.beta_ptr_array = nullptr;
    fusion_args.dBeta = {_0{}, _0{}, 0};
  }
  else {
    fusion_args.beta = 0;
    fusion_args.beta_ptr_array = beta_device.get();
    // Only one beta per each group
    fusion_args.dBeta = {_0{}, _0{}, 1};
  }

  typename Gemm::GemmKernel::TileSchedulerArguments scheduler;
  scheduler.raster_order = options.raster_order;

  arguments = typename Gemm::Arguments {
    cutlass::gemm::GemmUniversalMode::kGrouped,
    {options.m, options.n, options.k, options.groups, tokens_per_expert.get()},
    {block_A.get(), ptr_B.get()},
    {fusion_args, ptr_C.get(), nullptr, ptr_D.get(), nullptr},
    hw_info, scheduler
  };

  return arguments;
}
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Exposes scheduler policy as a first-class performance choice. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。把调度策略作为一级性能选择暴露出来。 针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
bool verify(const Options &options) {
  bool passed = true;
  for (int32_t i = 0; i < options.groups; ++i) {
    auto problem = options.problem_sizes_host.at(i);
    auto M = get<0>(problem);
    auto N = get<1>(problem);
    auto K = get<2>(problem);
    cutlass::TensorRef ref_A(block_A.get() + size_t(1) * i * M * K, Gemm::LayoutA::packed({M, K}));
    cutlass::TensorRef ref_B(block_B.get() + offset_B.at(i), Gemm::LayoutB::packed({K, N}));
    cutlass::TensorRef ref_C(block_C.get() + offset_C.at(i), Gemm::LayoutC::packed({M, N}));
    cutlass::TensorRef ref_D(block_ref_D.get() + offset_D.at(i), Gemm::LayoutD::packed({M, N}));

    //
    // Compute reference output
    //

    // Create instantiation for device reference gemm kernel
    DeviceGemmReference gemm_reference;

    // Launch device reference gemm kernel
    gemm_reference(
      {M, N, K},
      ElementAccumulator(alpha_host.at(i)),
      ref_A,
      ref_B,
      ElementAccumulator(beta_host.at(i)),
      ref_C,
      ref_D);

    // Wait for kernel to finish
    CUDA_CHECK(cudaDeviceSynchronize());

    // Check if output from CUTLASS kernel and reference kernel are equal or not
    passed &= cutlass::reference::device::BlockCompareEqual(block_ref_D.get() + offset_D.at(i), block_D.get() + offset_D.at(i), M * N);

  }
  return passed;
}

/// Execute a given example GEMM computation
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
template <typename Gemm>
int run(Options &options, bool host_problem_shapes_available = true)
{
  std::cout << "  Problem Sizes, Alpha, Beta " << std::endl;
  for (int32_t i = 0; i < options.groups; ++i) {
    std::cout << "    " << options.problem_sizes_host.at(i);
    std::cout << ", " << alpha_host.at(i) << ", " << beta_host.at(i) << std::endl;
  }
  std::cout << "  Groups      : " << options.groups  << std::endl;

  // Instantiate CUTLASS kernel depending on templates
  Gemm gemm;

  // Create a structure of gemm kernel arguments suitable for invoking an instance of Gemm
  auto arguments = args_from_options<Gemm>(options);

  // Using the arguments, query for extra workspace required for matrix multiplication computation
  size_t workspace_size = Gemm::get_workspace_size(arguments);

  // Allocate workspace memory
  cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);

  // Check if the problem size is supported or not
  CUTLASS_CHECK(gemm.can_implement(arguments));

  // Initialize CUTLASS kernel with arguments and workspace pointer
  CUTLASS_CHECK(gemm.initialize(arguments, workspace.get()));

  // Correctness / Warmup iteration
  CUTLASS_CHECK(gemm.run(/* stream = */ nullptr, /* cuda_adapter = */ nullptr, /* launch_with_pdl = */ options.use_pdl));

  // Check if output from CUTLASS kernel and reference kernel are equal or not
  Result result;
  result.passed = verify(options);

  std::cout << "  Disposition: " << (result.passed ? "Passed" : "Failed") << std::endl;

  if (!result.passed) {
    exit(-1);
  }

  // Run profiling loop
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
  if (options.iterations > 0)
  {
    for (int iter = 0; iter < options.warmup; ++iter) {
      CUTLASS_CHECK(gemm.initialize(arguments, workspace.get()));
      CUTLASS_CHECK(gemm.run(/* stream = */ nullptr, /* cuda_adapter = */ nullptr, /* launch_with_pdl = */ options.use_pdl));
    }
    GpuTimer timer;
    timer.start();
    for (int iter = 0; iter < options.iterations; ++iter) {
      CUTLASS_CHECK(gemm.initialize(arguments, workspace.get()));
      CUTLASS_CHECK(gemm.run(/* stream = */ nullptr, /* cuda_adapter = */ nullptr, /* launch_with_pdl = */ options.use_pdl));
    }
    timer.stop();

    // Compute average setup and runtime and GFLOPs.
    float elapsed_ms       = timer.elapsed_millis();
    result.avg_runtime_ms  = double(elapsed_ms) / double(options.iterations);
    result.gflops          = options.gflops(result.avg_runtime_ms / 1000.0, options.problem_sizes_host);

    std::cout << "  Avg runtime : " << result.avg_runtime_ms << " ms" << std::endl;
    std::cout << "  TFLOPS      : " << result.gflops / 1000.0 << std::endl;
  }

  return 0;
}
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
#endif // defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)
```

**EN**: This block defines the runtime argument/parameter packing used to cross the host/device boundary. CUTLASS separates high-level `Arguments` from lower-level `Params` so pointers, strides, schedules, and hardware metadata can be normalized before kernel launch. Targets blackwell sm100 execution paths. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一块定义了跨越 host/device 边界时使用的运行期参数封装。CUTLASS 通常把高层 `Arguments` 与更底层的 `Params` 分开，这样指针、stride、调度信息和硬件元数据就能在 kernel 启动前统一整理。面向 Blackwell SM100 执行路径。 针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
int main(int argc, char const **args) {

  // CUTLASS must be compiled with CUDA 12.8 Toolkit to run this example
  if (__CUDACC_VER_MAJOR__ < 12 ||
       ((__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ < 8)
       )
     ) {
    std::cerr << "This example requires CUDA 12.8 or newer.\n";
    // Returning zero so this test passes on older Toolkits. Its actions are no-op.
    return 0;
  }

  cudaDeviceProp props;
  int current_device_id;
  CUDA_CHECK(cudaGetDevice(&current_device_id));
  CUDA_CHECK(cudaGetDeviceProperties(&props, current_device_id));
  cudaError_t error = cudaGetDeviceProperties(&props, 0);
  if (props.major != 10 || (props.minor != 0 && props.minor != 1 && props.minor != 3)) {
    std::cerr << "This example requires a GPU with compute capability 100a|f, 101a|f, or 103a|f)." << std::endl;
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

#if defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)
  allocate(options);
  initialize(options);

  //
  // Evaluate CUTLASS kernels
  //
```

**EN**: This block forms the outer control flow of the sample. It performs capability checks, selects the concrete specialization to run, invokes the operation, and reports the final status or throughput back to the user. Targets blackwell sm100 execution paths. Is tuned for moe-style irregular expert workloads.  
**CN**: 这一块构成了示例的最外层控制流：先做能力检查，再选择具体特化版本，调用 operation，最后把状态或吞吐结果回报给用户。面向 Blackwell SM100 执行路径。 针对 MoE 风格的不规则 expert 工作负载进行调优。

---

```cpp
  std::cout << "Running kernel with 1SM MMA config:" << std::endl;
  run<Gemm1SM>(options);
  std::cout << "Running kernel with 2SM MMA config:" << std::endl;
  run<Gemm2SM>(options);     
#endif

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
- Scheduler policy as a first-class optimization knob / 作为一级优化旋钮的调度策略

## Dependencies / 依赖项

- `iostream` — included dependency used by this example / 此示例使用的包含依赖
- `fstream` — included dependency used by this example / 此示例使用的包含依赖
- `sstream` — included dependency used by this example / 此示例使用的包含依赖
- `vector` — included dependency used by this example / 此示例使用的包含依赖
- `random` — included dependency used by this example / 此示例使用的包含依赖
- `algorithm` — included dependency used by this example / 此示例使用的包含依赖
- `float.h` — included dependency used by this example / 此示例使用的包含依赖
- `cutlass/cutlass.h` — core CUTLASS types, architecture tags, and utilities / CUTLASS 核心类型、架构标签与工具
- `cute/tensor.hpp` — CuTe tensor and layout primitives / CuTe 张量与布局原语
- `cutlass/tensor_ref.h` — tensor reference wrappers / 张量引用封装
- `cutlass/epilogue/collective/default_epilogue.hpp` — epilogue collective implementation detail / 尾声 collective 实现细节
- `cutlass/epilogue/thread/linear_combination.h` — default linear-combination epilogue operator / 默认线性组合尾声算子
- `cutlass/gemm/dispatch_policy.hpp` — kernel scheduling and dispatch policy tags / 内核调度与分发策略标签
- `cutlass/gemm/group_array_problem_shape.hpp` — CUTLASS core component used by this example / 此示例使用的 CUTLASS 核心组件
- `cutlass/gemm/collective/collective_builder.hpp` — GEMM collective mainloop builder / GEMM collective 主循环构建器
- `cutlass/epilogue/collective/collective_builder.hpp` — epilogue collective builder / 尾声 collective 构建器
- `cutlass/gemm/device/gemm_universal_adapter.h` — device-side universal GEMM launcher adapter / 设备侧通用 GEMM 启动适配器
- `cutlass/gemm/kernel/gemm_universal.hpp` — universal GEMM kernel composition / 通用 GEMM 内核组合
- `cutlass/util/command_line.h` — command-line parsing helpers / 命令行解析辅助
- `cutlass/util/distribution.h` — random/distribution-based tensor initialization / 随机与分布初始化辅助
- `...` — additional direct includes omitted for brevity / 其余直接包含头文件因篇幅省略
