# 70_blackwell_fp8_gemm.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/70_blackwell_gemm/70_blackwell_fp8_gemm.cu`  
**Purpose / 用途**: FP8 dense GEMM for Blackwell SM100 demonstrating all important FP8 epilogue fusions: per-tensor scale factors for A/B/C/D/Aux, per-row bias, ReLU activation, auxiliary output tensor, and abs-max tracking for FP8 quantization; migrated from Hopper example 54. / 面向 Blackwell SM100 的 FP8 稠密 GEMM，演示所有重要的 FP8 尾声融合：A/B/C/D/Aux 的逐张量缩放因子、逐行偏置、ReLU 激活、辅助输出张量，以及用于 FP8 量化的绝对值最大值跟踪；从 Hopper 示例 54 迁移而来。

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
    \brief A FP8 dense GEMM example for the NVIDIA Blackwell SM100 architecture using CUTLASS.

    This example demonstrates minimal set of changes needed to transition from a Hopper CUTLASS 3.x 
    FP8 GEMM kernel (see example 54_hopper_fp8_warp_specialized_gemm) to a Blackwell SM100 FP8 GEMM kernel.
    
    This example shows all important fusions used by FP8 gemm kernels, 
    i.e., scale factor for A, B, C, D tensor, the abs_max value of D tensor.
    
    The Blackwell SM100 CUTLASS kernel uses of the following Blackwell SM100 features:
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Targets blackwell sm100 execution paths. Includes fp8/mxfp8-style narrow-precision behavior.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。面向 Blackwell SM100 执行路径。 包含 FP8/MXFP8 一类窄精度行为。

---

```cpp
     1. New series of Tensor Core MMA Instructions (tcgen05) introduced on the Blackwell architecture (sm100a) 
    which have 2x throughput compared to Hopper Tensor Core MMA instructions (WGMMA). 
    
    Note that Hopper WGMMA Tensor Core MMA instructions are not compatible on Blackwell (See https://docs.nvidia.com/cuda/parallel-thread-execution). 

    2. A new per-SM memory called Tensor Memory (TMEM) introduced on the Blackwell architecture (sm100a). 
    Blackwell SM100 Tensor Core MMA instructions store their accumulation results in TMEM instead of the 
    Register File. (Please refer to CUDA 12.8 docs on https://docs.nvidia.com/cuda/).

    3. An extended flavor of the warp-specialized kernel design introduced in Hopper enabled by use of TMEM 
    which allows us to decouple the execution of MMA and epilogue into separate warps. 

    4. A new SW controlled dynamic scheduler based on cluster launch control (See https://docs.nvidia.com/cuda/parallel-thread-execution).
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Targets blackwell sm100 execution paths. Manages shared memory and, on blackwell, often tmem-backed intermediate state.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。面向 Blackwell SM100 执行路径。 管理共享内存，并在 Blackwell 上经常涉及 TMEM 中间状态。

---

```cpp
    Usage:
      $ ./examples/70_blackwell_gemm/70_blackwell_fp8_gemm --m=8192 --n=8192 --k=8192
*/
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Includes fp8/mxfp8-style narrow-precision behavior.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。包含 FP8/MXFP8 一类窄精度行为。

---

```cpp
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
```

**EN**: This include block pulls in the CUTLASS/CuTe and utility headers needed for the example. In practice it establishes the compile-time vocabulary for layouts, collectives, kernel wrappers, and host-side helpers (e.g. iostream, cutlass/cutlass.h, cute/tensor.hpp, cutlass/tensor_ref.h).  
**CN**: 这一组 `#include` 引入了示例所需的 CUTLASS/CuTe 与工具头文件，建立布局、collective、内核封装以及主机侧辅助函数的编译期词汇表（例如 iostream, cutlass/cutlass.h, cute/tensor.hpp, cutlass/tensor_ref.h）。

---

```cpp
#include "helper.h"

using namespace cute;

#if defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Targets blackwell sm100 execution paths.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。面向 Blackwell SM100 执行路径。

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

// Epilogue types
using ElementBias        = cutlass::half_t;
using ElementCompute     = float;
using ElementAux         = ElementC;
using LayoutAux          = LayoutC;
using ElementAmax        = float;

// MMA and Cluster Tile Shapes
// Shape of the tile computed by tcgen05 MMA, could be across 2 SMs if Cluster Shape %2 == 0 
using MmaTileShape_MNK = Shape<_256,_128,_64>;                          
// Shape of the threadblocks in a cluster
using ClusterShape_MNK = Shape<_2,_2,_1>;

using FusionOp = cutlass::epilogue::fusion::ScaledLinCombPerRowBiasEltActAmaxAux<
  LayoutC, cutlass::epilogue::thread::ReLU, ElementD, ElementCompute, ElementAux, ElementAmax, ElementBias>;
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Includes fp8/mxfp8-style narrow-precision behavior. Makes cluster geometry an explicit compile-time tuning knob.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。包含 FP8/MXFP8 一类窄精度行为。 把 cluster 几何作为显式的编译期调优参数。

---

```cpp
using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
    cutlass::arch::Sm100, cutlass::arch::OpClassTensorOp,
    MmaTileShape_MNK, ClusterShape_MNK,
    cutlass::epilogue::collective::EpilogueTileAuto,
    ElementAccumulator, ElementCompute,
    ElementC, LayoutC, AlignmentC,
    ElementD, LayoutC, AlignmentD,
    cutlass::epilogue::collective::EpilogueScheduleAuto,
    FusionOp
  >::CollectiveOp;

using CollectiveMainloop = typename cutlass::gemm::collective::CollectiveBuilder<
    cutlass::arch::Sm100, cutlass::arch::OpClassTensorOp,
    ElementA, LayoutA, AlignmentA,
    ElementB, LayoutB, AlignmentB,
    ElementAccumulator,
    MmaTileShape_MNK, ClusterShape_MNK,
    cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))>,
    cutlass::gemm::collective::KernelScheduleAuto
  >::CollectiveOp;

using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
    Shape<int,int,int,int>,
    CollectiveMainloop,
    CollectiveEpilogue,
    void>;                // Default to ClusterLaunchControl (CLC) based tile scheduler

using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;

// Extract information from Gemm kernel.
using EpilogueOutputOp  = typename Gemm::EpilogueOutputOp;
using ElementScalar     = typename EpilogueOutputOp::ElementScalar;
using ElementAmax       = typename EpilogueOutputOp::ElementAmax;
using ActivationFunctor = typename EpilogueOutputOp::ActivationFn;

using StrideA = typename Gemm::GemmKernel::StrideA;
using StrideB = typename Gemm::GemmKernel::StrideB;
using StrideC = typename Gemm::GemmKernel::StrideC;
using StrideD = typename Gemm::GemmKernel::StrideD;
using StrideAux = StrideC;
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Targets blackwell sm100 execution paths. Selects tensor-core mma execution instead of a simt fallback.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。面向 Blackwell SM100 执行路径。 选择 Tensor Core MMA 执行，而不是 SIMT 回退路径。

---

```cpp
constexpr bool IsDFp8 = 
    cute::is_same_v<ElementD, cutlass::float_e4m3_t> or
    cute::is_same_v<ElementD, cutlass::float_e5m2_t>;

constexpr bool IsAuxFp8 =
    cute::is_same_v<ElementAux, cutlass::float_e4m3_t> or
    cute::is_same_v<ElementAux, cutlass::float_e5m2_t>;

/// Initialization
StrideA stride_A;
StrideB stride_B;
StrideC stride_C;
StrideD stride_D;
StrideAux stride_aux;
uint64_t seed;

cutlass::HostTensor<ElementA  , LayoutA  > tensor_A;
cutlass::HostTensor<ElementB  , LayoutB  > tensor_B;
cutlass::HostTensor<ElementC  , LayoutC  > tensor_C;
cutlass::HostTensor<ElementD  , LayoutD  > tensor_D;
cutlass::HostTensor<ElementD  , LayoutD  > tensor_ref_D;
cutlass::HostTensor<ElementAux, LayoutAux> tensor_aux;
cutlass::HostTensor<ElementAux, LayoutAux> tensor_ref_aux;

using LayoutScalar = cutlass::layout::PackedVectorLayout;
cutlass::HostTensor<ElementScalar, LayoutScalar> scalar_alpha;
cutlass::HostTensor<ElementScalar, LayoutScalar> scalar_beta;
cutlass::HostTensor<ElementScalar, LayoutScalar> scale_A;
cutlass::HostTensor<ElementScalar, LayoutScalar> scale_B;
cutlass::HostTensor<ElementScalar, LayoutScalar> scale_C;
cutlass::HostTensor<ElementScalar, LayoutScalar> scale_D;
cutlass::HostTensor<ElementScalar, LayoutScalar> scale_aux;
cutlass::HostTensor<ElementAmax  , LayoutScalar> abs_max_D;
cutlass::HostTensor<ElementAmax  , LayoutScalar> reference_abs_max_D;
cutlass::HostTensor<ElementAmax  , LayoutScalar> abs_max_aux;
cutlass::HostTensor<ElementAmax  , LayoutScalar> reference_abs_max_aux;
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Includes fp8/mxfp8-style narrow-precision behavior.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。包含 FP8/MXFP8 一类窄精度行为。

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

  float alpha = 1.f, beta = 0.f;
  float scale_a = 1.f, scale_b = 1.f, scale_c = 1.f, scale_d = 1.f, scale_aux = 1.f;
  bool device_scale = false;
  bool save_aux = true;
  bool save_amax = true;
  int iterations = 1000;
  int m = 1024, n = 512, k = 1024, l = 1;
  int swizzle = 0;

  // Parses the command line
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);

    if (cmd.check_cmd_line_flag("help")) {
      help = true;
      return;
    }

    cmd.get_cmd_line_argument("m", m);
    cmd.get_cmd_line_argument("n", n);
    cmd.get_cmd_line_argument("k", k);
    cmd.get_cmd_line_argument("l", l);
    cmd.get_cmd_line_argument("alpha", alpha, 1.f);
    cmd.get_cmd_line_argument("beta", beta, 0.f);
    cmd.get_cmd_line_argument("scale_a", scale_a, 1.f);
    cmd.get_cmd_line_argument("scale_b", scale_b, 1.f);
    cmd.get_cmd_line_argument("scale_c", scale_c, 1.f);
    cmd.get_cmd_line_argument("scale_d", scale_d, 1.f);
    cmd.get_cmd_line_argument("scale_aux", scale_aux, 1.f);
    cmd.get_cmd_line_argument("device_scale", device_scale, false);
    cmd.get_cmd_line_argument("save_aux", save_aux, true);
    cmd.get_cmd_line_argument("save_amax", save_amax, true);
    cmd.get_cmd_line_argument("iterations", iterations);
    cmd.get_cmd_line_argument("swizzle", swizzle);
  }

  /// Prints the usage statement.
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。

---

```cpp
  std::ostream & print_usage(std::ostream &out) const {

    out << "70_blackwell_fp8_gemm\n\n"
      << "  Blackwell FP8 GEMM using a Warp Specialized kernel.\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement\n\n"
      << "  --m=<int>                   Sets the M extent of the GEMM\n"
      << "  --n=<int>                   Sets the N extent of the GEMM\n"
      << "  --k=<int>                   Sets the K extent of the GEMM\n"
      << "  --l=<int>                   Sets the l extent (batch) of the GEMM\n"
      << "  --alpha=<f32>               Epilogue scalar alpha\n"
      << "  --beta=<f32>                Epilogue scalar beta\n"
      << "  --swizzle=<int>             Cluster rasterization swizzle\n"
      << "  --scale_a=<f32>             Scaling factor for A\n"
      << "  --scale_b=<f32>             Scaling factor for B\n"
      << "  --scale_c=<f32>             Scaling factor for C\n"
      << "  --scale_d=<f32>             Scaling factor for D (ignored for non-fp8 D)\n"
      << "  --scale_aux=<f32>           Scaling factor for the auxiliary tensor (ignored for non-fp8 aux)\n"
      << "  --device_scale=<bool>       Copy scalars to device memory before kernel launch (default: false)\n"
      << "  --save_aux=<bool>           Save the pre-activation as an auxiliary tensor (default: true)\n"
      << "  --save_amax=<bool>          Save the pre-scaled max absolute value of any fp8 outputs (aux and/or D) (default: true)\n"
      << "  --iterations=<int>          Number of profiling iterations to perform.\n\n";

    out
      << "\n\nExamples:\n\n"
      << "$ " << "70_blackwell_fp8_gemm" << " --m=1024 --n=512 --k=1024 --alpha=2 --beta=0.707 \n\n";

    return out;
  }

  /// Compute performance in GFLOP/s
  double gflops(double runtime_s) const
  {
    // Two flops per multiply-add
    uint64_t flop = uint64_t(2) * m * n * k;
    double gflop = double(flop) / double(1.0e9);
    return gflop / runtime_s;
  }
};

/// Result structure
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Includes fp8/mxfp8-style narrow-precision behavior.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。包含 FP8/MXFP8 一类窄精度行为。

---

```cpp
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

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。

---

```cpp
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
  uint64_t seed) {

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

  return true;
}

/// Initialize operands to be used in the GEMM and reference GEMM
void initialize(const Options &options) {

  stride_A = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(options.m, options.k, options.l));
  stride_B = cutlass::make_cute_packed_stride(StrideB{}, cute::make_shape(options.n, options.k, options.l));
  stride_C = cutlass::make_cute_packed_stride(StrideC{}, cute::make_shape(options.m, options.n, options.l));
  stride_D = cutlass::make_cute_packed_stride(StrideD{}, cute::make_shape(options.m, options.n, options.l));
  stride_aux = stride_D;
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。

---

```cpp
  auto a_coord = cutlass::make_Coord(options.m * options.l, options.k);
  auto c_coord = cutlass::make_Coord(options.m * options.l, options.n);
  auto b_coord = cutlass::make_Coord(options.k, options.n * options.l);

  tensor_A.resize(a_coord);
  tensor_B.resize(b_coord);
  tensor_C.resize(c_coord);
  tensor_D.resize(c_coord);
  tensor_ref_D.resize(c_coord);

  initialize_tensor(tensor_A.host_view(), seed + 2022);
  initialize_tensor(tensor_B.host_view(), seed + 2023);
  initialize_tensor(tensor_C.host_view(), seed + 2024);

  tensor_A.sync_device();
  tensor_B.sync_device();
  tensor_C.sync_device();
  tensor_D.sync_device();

  if (options.save_aux) {
    tensor_aux.resize(c_coord);
    tensor_aux.sync_device();
    tensor_ref_aux.resize(c_coord);
  }

  if (options.device_scale) {
    scalar_alpha.resize(cutlass::make_Coord(1));
    scalar_beta.resize(cutlass::make_Coord(1));
    scale_A.resize(cutlass::make_Coord(1));
    scale_B.resize(cutlass::make_Coord(1));
    scale_C.resize(cutlass::make_Coord(1));
    scale_D.resize(cutlass::make_Coord(1));
    scale_aux.resize(cutlass::make_Coord(1));

    cutlass::reference::host::TensorFill(scalar_alpha.host_view(), options.alpha);
    cutlass::reference::host::TensorFill(scalar_beta.host_view(), options.beta);
    cutlass::reference::host::TensorFill(scale_A.host_view(), options.scale_a);
    cutlass::reference::host::TensorFill(scale_B.host_view(), options.scale_b);
    cutlass::reference::host::TensorFill(scale_C.host_view(), options.scale_c);
    cutlass::reference::host::TensorFill(scale_D.host_view(), options.scale_d);
    cutlass::reference::host::TensorFill(scale_aux.host_view(), options.scale_aux);
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。

---

```cpp
    scalar_alpha.sync_device();
    scalar_beta.sync_device();
    scale_A.sync_device();
    scale_B.sync_device();
    scale_C.sync_device();
    scale_D.sync_device();
    scale_aux.sync_device();
  }

  if (IsDFp8 && options.save_amax) {
    abs_max_D.resize(cutlass::make_Coord(1));
    abs_max_D.sync_device();
    reference_abs_max_D.resize(cutlass::make_Coord(1));
  }

  if (IsAuxFp8 && options.save_aux && options.save_amax) {
    abs_max_aux.resize(cutlass::make_Coord(1));
    abs_max_aux.sync_device();
    reference_abs_max_aux.resize(cutlass::make_Coord(1));
  }
}

/// Populates a Gemm::Arguments structure from the given commandline options
typename Gemm::Arguments args_from_options(const Options &options)
{
  typename Gemm::Arguments arguments{
    cutlass::gemm::GemmUniversalMode::kGemm,
    {options.m, options.n, options.k, options.l},
    {tensor_A.device_data(), stride_A, tensor_B.device_data(), stride_B},
    {
      {}, // epilogue.thread
      tensor_C.device_data(), stride_C,
      tensor_D.device_data(), stride_D
    }
  };
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface. Includes fp8/mxfp8-style narrow-precision behavior.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。包含 FP8/MXFP8 一类窄精度行为。

---

```cpp
  auto &fusion_args = arguments.epilogue.thread;
  fusion_args.alpha = options.alpha;
  fusion_args.beta = options.beta;
  fusion_args.alpha_ptr = scalar_alpha.device_data();
  fusion_args.beta_ptr = scalar_beta.device_data();
  fusion_args.scale_a = options.scale_a;
  fusion_args.scale_b = options.scale_b;
  fusion_args.scale_c = options.scale_c;
  fusion_args.scale_a_ptr = scale_A.device_data();
  fusion_args.scale_b_ptr = scale_B.device_data();
  fusion_args.scale_c_ptr = scale_C.device_data();

  // ignored if tensor types are not fp8
  fusion_args.scale_d = options.scale_d;
  fusion_args.scale_aux = options.scale_aux;
  fusion_args.scale_d_ptr = scale_D.device_data();
  fusion_args.scale_aux_ptr = scale_aux.device_data();

  // leaving/setting these as nullptr disables the fusion at runtime
  fusion_args.bias_ptr = nullptr;

  if (options.save_aux) {
    fusion_args.aux_ptr = tensor_aux.device_data();
    fusion_args.dAux = stride_aux;
    if (options.save_amax) {
      fusion_args.amax_aux_ptr = abs_max_aux.device_data();
    }
  }

  if (options.save_amax) {
    fusion_args.amax_D_ptr = abs_max_D.device_data();
  }

  arguments.scheduler.max_swizzle_size = options.swizzle;

  return arguments;
}

bool verify(const Options &options) {
  //
  // Compute reference output
  //
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface. Includes fp8/mxfp8-style narrow-precision behavior.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。包含 FP8/MXFP8 一类窄精度行为。

---

```cpp
  // Create instantiation for device reference gemm kernel
  auto A = cute::make_tensor(tensor_A.host_data(),
      cute::make_layout(cute::make_shape(options.m, options.k, options.l), stride_A));
  auto B = cute::make_tensor(tensor_B.host_data(),
      cute::make_layout(cute::make_shape(options.n, options.k, options.l), stride_B));
  auto C = cute::make_tensor(tensor_C.host_data(),
      cute::make_layout(cute::make_shape(options.m, options.n, options.l), stride_C));
  auto D = cute::make_tensor(tensor_ref_D.host_data(),
      cute::make_layout(cute::make_shape(options.m, options.n, options.l), stride_D));
  auto Aux = cute::make_tensor(tensor_ref_aux.host_data(),
      cute::make_layout(cute::make_shape(options.m, options.n, options.l), stride_aux));
  using unused_t = decltype(D);

  cutlass::reference::host::GettMainloopParams<ElementAccumulator, decltype(A), decltype(B)> mainloop_params{A, B};

  cutlass::reference::host::GettEpilogueParams<
      ElementScalar,
      ElementScalar,
      ElementAccumulator,
      ElementCompute,
      decltype(C),
      decltype(D),
      unused_t, // bias
      decltype(Aux),
      unused_t, // valpha
      unused_t, // vbeta
      ActivationFunctor
  > epilogue_params;
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface. Relies on tma-style bulk movement or descriptor handling.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。依赖 TMA 风格的批量搬运或描述符处理。

---

```cpp
  epilogue_params.C = C;
  epilogue_params.D = D;
  epilogue_params.Aux = Aux;
  epilogue_params.alpha = options.alpha;
  epilogue_params.beta = options.beta;
  epilogue_params.scale_a = options.scale_a;
  epilogue_params.scale_b = options.scale_b;
  epilogue_params.scale_c = options.scale_c;
  epilogue_params.scale_d = options.scale_d;
  epilogue_params.scale_aux = options.scale_aux;
  epilogue_params.abs_max_D = reference_abs_max_D.host_data();
  epilogue_params.abs_max_Aux = reference_abs_max_aux.host_data();

  // get reference result
  cutlass::reference::host::Gemm3x(mainloop_params, epilogue_params);

  // compare_reference
  tensor_D.sync_host();
  bool passed = cutlass::reference::host::TensorEquals(tensor_ref_D.host_view(), tensor_D.host_view());

  if (IsDFp8 && options.save_amax) {
    abs_max_D.sync_host();
    passed &= abs_max_D.at(cutlass::make_Coord(0)) == reference_abs_max_D.at(cutlass::make_Coord(0));
  }

  if (options.save_aux) {
    tensor_aux.sync_host();
    passed &= cutlass::reference::host::TensorEquals(tensor_ref_aux.host_view(), tensor_aux.host_view());
    if (IsAuxFp8 && options.save_amax) {
      abs_max_aux.sync_host();
      passed &= abs_max_aux.at(cutlass::make_Coord(0)) == reference_abs_max_aux.at(cutlass::make_Coord(0));
    }
  }

  return passed;
}

/// Execute a given example GEMM computation
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface. Includes fp8/mxfp8-style narrow-precision behavior.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。包含 FP8/MXFP8 一类窄精度行为。

---

```cpp
template <typename Gemm>
int run(Options &options)
{
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
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。

---

```cpp
  // Check if output from CUTLASS kernel and reference kernel are equal or not
  Result result;
  result.passed = verify(options);

  std::cout << "  Disposition: " << (result.passed ? "Passed" : "Failed") << std::endl;

  if (!result.passed) {
    exit(-1);
  }

  // Run profiling loop
  if (options.iterations > 0)
  {
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

  // CUTLASS must be compiled with CUDA 12.8 Toolkit to run this example
  // and must have compute capability at least sm100a.

  if (__CUDACC_VER_MAJOR__ < 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ < 8)) {
    std::cerr << "This example requires CUDA 12.8 or newer." << std::endl;
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
```

**EN**: This block forms the outer control flow of the sample. It performs capability checks, selects the concrete specialization to run, invokes the operation, and reports the final status or throughput back to the user. Targets blackwell sm100 execution paths.  
**CN**: 这一块构成了示例的最外层控制流：先做能力检查，再选择具体特化版本，调用 operation，最后把状态或吞吐结果回报给用户。面向 Blackwell SM100 执行路径。

---

```cpp
  return 0;
}
```

**EN**: This block forms the outer control flow of the sample. It performs capability checks, selects the concrete specialization to run, invokes the operation, and reports the final status or throughput back to the user.  
**CN**: 这一块构成了示例的最外层控制流：先做能力检查，再选择具体特化版本，调用 operation，最后把状态或吞吐结果回报给用户。

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
