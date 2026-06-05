# 84a_blackwell_nvfp4_bf16_sparse_gemm.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/84_blackwell_narrow_precision_sparse_gemm/84a_blackwell_nvfp4_bf16_sparse_gemm.cu`  
**Purpose / 用途**: Shows how Blackwell SM100 combines structured sparsity with NVFP4 narrow inputs and BF16 output, including sparse metadata generation and block-scale tensor handling. / 展示 Blackwell SM100 如何把结构化稀疏与 NVFP4 窄精度输入、BF16 输出结合起来，并同时处理稀疏元数据生成和 block-scale tensor。

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
    \brief A Narrow Precision Sparse GEMM example using CUTLASS for the NVIDIA Blackwell SM100 architecture.

    This example demonstrates a simple way to instantiate and run a blockscaled NVFP4 Sparse GEMM on the NVIDIA Blackwell SM100 architecture.

    The Blackwell SM100 CUTLASS kernel uses the new Block Scaled Tensor Core MMA Instructions (tcgen05.mma.blockscaled) introduced
    on the Blackwell architecture (sm100a) which have 2x throughput compared to fp8 Tensor Core MMA instructions (tcgen05.mma)
    and 4x throughput compared to fp8 Hopper Tensor Core MMA Instructions (WGMMA) (See https://docs.nvidia.com/cuda/parallel-thread-execution).
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Targets blackwell sm100 execution paths. Handles fp4/nvfp4-style narrow precision.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。面向 Blackwell SM100 执行路径。 处理 FP4/NVFP4 一类窄精度数据。

---

```cpp
    Similar to 83_blackwell_sparse_gemm, this kernel leverages:
    1. Per-SM memory called Tensor Memory (TMEM)  (Please refer to CUDA 12.8 docs on https://docs.nvidia.com/cuda/).

    2. The extended warp-specialized kernel design introduced in Hopper enabled by use of TMEM
    which allows us to decouple the execution of MMA and epilogue into separate warps.

    3. A new SW controlled dynamic scheduler based on cluster launch control (See https://docs.nvidia.com/cuda/parallel-thread-execution).

    Usage:
      $ ./examples/84_blackwell_narrow_precision_sparse_gemm/84a_blackwell_nvfp4_bf16_sparse_gemm --m=2048 --n=2048 --k=2048
*/

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
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/transform/kernel/sparse_gemm_compressor.hpp"
#include "cutlass/transform/device/transform_universal_adapter.hpp"
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Targets blackwell sm100 execution paths. Handles fp4/nvfp4-style narrow precision.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。面向 Blackwell SM100 执行路径。 处理 FP4/NVFP4 一类窄精度数据。

---

```cpp
#include "helper.h"

using namespace cute;

#if defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Targets blackwell sm100 execution paths. Carries sparse metadata or structured-sparsity constraints alongside math operands.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。面向 Blackwell SM100 执行路径。 在数学操作数之外还携带稀疏元数据或结构化稀疏约束。

---

```cpp
/// GEMM kernel configurations
```

**EN**: This configuration block defines compile-time aliases or constants that control data types, layouts, alignments, tile sizes, or enum-style modes. These choices are not cosmetic: in CUTLASS they steer which iterators, MMA atoms, and memory movement strategies become legal and efficient. Carries sparse metadata or structured-sparsity constraints alongside math operands.  
**CN**: 这个配置块定义了编译期别名或常量，用来控制数据类型、布局、对齐、tile 大小或枚举模式。这些选择并非表面参数：在 CUTLASS 中，它们会直接决定哪些迭代器、MMA atom 和数据搬运策略既合法又高效。在数学操作数之外还携带稀疏元数据或结构化稀疏约束。

---

```cpp
// A matrix configuration
using         ElementA     = cutlass::float_e2m1_t;
using         ElementAPair = cutlass::nv_float4_t<ElementA>;                 // Element type for A matrix operand
using         LayoutTagA   = cutlass::layout::RowMajor;                      // Layout type for A matrix operand
constexpr int AlignmentA   = 64;                                             // Memory access granularity/alignment of A matrix in units of elements (up to 16 bytes), 2x for compress along k

// E matrix config
using         ElementE    = cute::uint8_t;
using         LayoutTagE  = LayoutTagA;

// B matrix configuration
using         ElementB     = cutlass::float_e2m1_t;
using         ElementBPair = cutlass::nv_float4_t<ElementB>;                 // Element type for B matrix operand
using         LayoutTagB   = cutlass::layout::ColumnMajor;                   // Layout type for B matrix operand
constexpr int AlignmentB   = 32;                                             // Memory access granularity/alignment of B matrix in units of elements (up to 16 bytes)

// SF
using         ElementSF   = typename ElementAPair::ScaleFactorType;

// C/D matrix configuration
using         ElementD    = cutlass::bfloat16_t;                            // Element type for D matrix operand
using         ElementC    = cutlass::bfloat16_t;                            // Element type for C matrix operand
using         LayoutTagC  = cutlass::layout::RowMajor;                      // Layout type for C matrix operand
using         LayoutTagD  = cutlass::layout::RowMajor;                      // Layout type for D matrix operand
constexpr int AlignmentD  = (16 * 8) / cutlass::sizeof_bits<ElementD>::value;    // Memory access granularity/alignment of C matrix in units of elements (up to 16 bytes)
constexpr int AlignmentC  = (16 * 8) / cutlass::sizeof_bits<ElementC>::value;    // Memory access granularity/alignment of C matrix in units of elements (up to 16 bytes)

// Kernel functional config
using ElementAccumulator  = float;                                           // Element type for internal accumulation
using ArchTag             = cutlass::arch::Sm100;                            // Tag indicating the minimum SM that supports the intended feature
using OperatorClass       = cutlass::arch::OpClassBlockScaledSparseTensorOp; // Operator class tag

// MMA and Cluster Tile Shapes
// Shape of the tile computed by tcgen05 MMA, could be across 2 SMs if Cluster Shape %2 == 0
using MmaTileShape = Shape<_256,_128,_256>;
// Shape of the threadblocks in a cluster
using ClusterShape = Shape<_2,_1,_1>;

// Build the epilogue
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Targets blackwell sm100 execution paths. Handles fp4/nvfp4-style narrow precision.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。面向 Blackwell SM100 执行路径。 处理 FP4/NVFP4 一类窄精度数据。

---

```cpp
using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    MmaTileShape, ClusterShape,
    cutlass::epilogue::collective::EpilogueTileAuto,
    ElementAccumulator, ElementAccumulator,
    ElementC, LayoutTagC, AlignmentC,
    ElementD, LayoutTagD, AlignmentD,
    cutlass::epilogue::TmaWarpSpecialized2SmNvf4
  >::CollectiveOp;

// Build the mainloop
using CollectiveMainloop = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    ElementAPair, LayoutTagA, AlignmentA,
    ElementBPair, LayoutTagB, AlignmentB,
    ElementAccumulator,
    MmaTileShape, ClusterShape,
    cutlass::gemm::collective::StageCountAutoCarveoutEpi<CollectiveEpilogue>,
    cutlass::gemm::KernelSparseTmaWarpSpecialized2SmNvf4Sm100
  >::CollectiveOp;

using ProblemShape = Shape<int,int,int,int>;

// Compose into a kernel
using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
    ProblemShape,
    CollectiveMainloop,
    CollectiveEpilogue,
    void>;                   // Default to ClusterLaunchControl (CLC) based tile scheduler

using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;

//
// Blockscale
//
using Sm1xxBlkScaledConfig =  typename Gemm::GemmKernel::CollectiveMainloop::Sm1xxBlkScaledConfig;
using Blk_MN   = typename Sm1xxBlkScaledConfig::Blk_MN;
using Blk_SF   = typename Sm1xxBlkScaledConfig::Blk_SF;
using SfAtom   = typename Sm1xxBlkScaledConfig::SfAtom;
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Targets blackwell sm100 execution paths. Makes cluster geometry an explicit compile-time tuning knob.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。面向 Blackwell SM100 执行路径。 把 cluster 几何作为显式的编译期调优参数。

---

```cpp
using LayoutA = typename Gemm::GemmKernel::CollectiveMainloop::LayoutA;
using LayoutE = typename Gemm::GemmKernel::CollectiveMainloop::LayoutE;
using StrideA = cutlass::gemm::TagToStrideA_t<LayoutTagA>;
using StrideE = StrideA;
using StrideB = typename Gemm::GemmKernel::StrideB;

using LayoutSFA = typename Gemm::GemmKernel::CollectiveMainloop::LayoutSFA;      // Scale Factor tensors have an interleaved layout. Bring Layout instead of stride.
using LayoutSFB = typename Gemm::GemmKernel::CollectiveMainloop::LayoutSFB;      // Scale Factor tensors have an interleaved layout. Bring Layout instead of stride.

using StrideC = typename Gemm::GemmKernel::StrideC;
using StrideD = typename Gemm::GemmKernel::StrideD;

//
// Compressor
//
using SparseConfig = typename Gemm::GemmKernel::CollectiveMainloop::SparseConfig;

using CompressorUtility = cutlass::transform::kernel::StructuredSparseCompressorUtility<
                            ProblemShape,
                            ElementA,
                            LayoutTagA,
                            SparseConfig>;

using CompressorKernel = cutlass::transform::kernel::StructuredSparseCompressor<
                            ProblemShape,
                            ElementA,
                            LayoutTagA,
                            SparseConfig,
                            ArchTag>;

using Compressor = cutlass::transform::device::TransformUniversalAdapter<CompressorKernel>;

//
// Data members
//

/// Initialization
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Carries sparse metadata or structured-sparsity constraints alongside math operands.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。在数学操作数之外还携带稀疏元数据或结构化稀疏约束。

---

```cpp
StrideA stride_A;
StrideA stride_A_compressed;
StrideE stride_E;
StrideB stride_B;
StrideC stride_C;
StrideD stride_D;

LayoutA layout_A;
LayoutE layout_E;
LayoutSFA layout_SFA;
LayoutSFB layout_SFB;

typename LayoutTagA::Stride stride_factor_A;
typename LayoutTagB::Stride stride_factor_B;
typename LayoutTagE::Stride stride_factor_E;
typename LayoutTagC::Stride stride_factor_C;
typename LayoutTagD::Stride stride_factor_D;

uint64_t seed;

ProblemShape problem_shape;

// The HostTensors are only used for allocating memory on host and device, and transferring data between host and device
// Use cute::Tensor and cute::Layout for iterating thru the matrix elements
cutlass::HostTensor<ElementA, LayoutTagA> tensor_A;
cutlass::HostTensor<ElementA, LayoutTagA> tensor_A_compressed;
cutlass::HostTensor<ElementE, LayoutTagE> tensor_E;
cutlass::HostTensor<ElementB, LayoutTagB> tensor_B;
cutlass::HostTensor<ElementC, LayoutTagC> tensor_C;
cutlass::HostTensor<ElementSF, LayoutTagA> tensor_SFA;
cutlass::HostTensor<ElementSF, LayoutTagB> tensor_SFB;
cutlass::HostTensor<ElementD, LayoutTagD> tensor_D;
cutlass::HostTensor<ElementD, LayoutTagD> reference_D;

#endif // defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Targets blackwell sm100 execution paths. Carries sparse metadata or structured-sparsity constraints alongside math operands.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。面向 Blackwell SM100 执行路径。 在数学操作数之外还携带稀疏元数据或结构化稀疏约束。

---

```cpp
template <typename T>
auto make_iterator(T* ptr) {
  return cute::recast_ptr<T>(ptr);
}
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Carries sparse metadata or structured-sparsity constraints alongside math operands.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。在数学操作数之外还携带稀疏元数据或结构化稀疏约束。

---

```cpp
/// Testbed utility types
```

**EN**: This configuration block defines compile-time aliases or constants that control data types, layouts, alignments, tile sizes, or enum-style modes. These choices are not cosmetic: in CUTLASS they steer which iterators, MMA atoms, and memory movement strategies become legal and efficient. Carries sparse metadata or structured-sparsity constraints alongside math operands.  
**CN**: 这个配置块定义了编译期别名或常量，用来控制数据类型、布局、对齐、tile 大小或枚举模式。这些选择并非表面参数：在 CUTLASS 中，它们会直接决定哪些迭代器、MMA atom 和数据搬运策略既合法又高效。在数学操作数之外还携带稀疏元数据或结构化稀疏约束。

---

```cpp
// Command line options parsing
struct Options {

  bool help;

  float alpha, beta;
  int iterations;
  int m, n, k, l;

  Options():
    help(false),
    m(1024), n(1024), k(1024), l(1),
    alpha(1.f), beta(0.f),
    iterations(10)
  { }

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
    cmd.get_cmd_line_argument("iterations", iterations);
  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Carries sparse metadata or structured-sparsity constraints alongside math operands.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。在数学操作数之外还携带稀疏元数据或结构化稀疏约束。

---

```cpp
    out << "84a_blackwell_nvfp4_bf16_sparse_gemm\n\n"
      << "  Blackwell NVFP4 GEMM using a Warp Specialized kernel.\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement\n\n"
      << "  --m=<int>                   Sets the M extent of the GEMM\n"
      << "  --n=<int>                   Sets the N extent of the GEMM\n"
      << "  --k=<int>                   Sets the K extent of the GEMM\n"
      << "  --l=<int>                   Sets the L extent of the GEMM\n"
      << "  --alpha=<f32>               Epilogue scalar alpha\n"
      << "  --beta=<f32>                Epilogue scalar beta\n"
      << "  --iterations=<int>          Number of profiling iterations to perform.\n\n";

    out << "\n\nExamples:\n\n"
      << "$ " << "./examples/84_blackwell_narrow_precision_sparse_gemm/84a_blackwell_nvfp4_bf16_sparse_gemm" << " --m=1024 --n=512 --k=1024 --alpha=2 --beta=0.707 \n\n";

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
struct Result
{
  double avg_runtime_ms;
  double gflops;
  cutlass::Status status;
  cudaError_t error;
  bool passed;
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Handles fp4/nvfp4-style narrow precision. Carries sparse metadata or structured-sparsity constraints alongside math operands.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。处理 FP4/NVFP4 一类窄精度数据。 在数学操作数之外还携带稀疏元数据或结构化稀疏约束。

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

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Targets blackwell sm100 execution paths. Carries sparse metadata or structured-sparsity constraints alongside math operands.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。面向 Blackwell SM100 执行路径。 在数学操作数之外还携带稀疏元数据或结构化稀疏约束。

---

```cpp
/// GEMM setup and evaluation
```

**EN**: This configuration block defines compile-time aliases or constants that control data types, layouts, alignments, tile sizes, or enum-style modes. These choices are not cosmetic: in CUTLASS they steer which iterators, MMA atoms, and memory movement strategies become legal and efficient. Carries sparse metadata or structured-sparsity constraints alongside math operands.  
**CN**: 这个配置块定义了编译期别名或常量，用来控制数据类型、布局、对齐、tile 大小或枚举模式。这些选择并非表面参数：在 CUTLASS 中，它们会直接决定哪些迭代器、MMA atom 和数据搬运策略既合法又高效。在数学操作数之外还携带稀疏元数据或结构化稀疏约束。

---

```cpp
/// Helper to initialize a block of device data
template <typename Element, typename Layout>
void initialize_tensor(
  cutlass::TensorView<Element, Layout> view,
  uint64_t seed) {

  double scope_max, scope_min;
  int bits_input = cutlass::sizeof_bits<Element>::value;

  if (bits_input == 1) {
    scope_max = 2;
    scope_min = 0;
  }
    else if (bits_input <= 6) {
    scope_max = 2;
    scope_min = -2;
  }
  else if (bits_input <= 8) {
    if constexpr (cute::is_same_v<Element, cutlass::float_ue8m0_t>){
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
}

/// Initialize operands to be used in the GEMM and reference GEMM
bool initialize(const Options &options) {

  problem_shape = make_tuple(options.m, options.n, options.k, options.l);

  // * Get A B C D size
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Carries sparse metadata or structured-sparsity constraints alongside math operands.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。在数学操作数之外还携带稀疏元数据或结构化稀疏约束。

---

```cpp
  stride_A = cutlass::make_cute_packed_stride(StrideA{}, {options.m, options.k, 1});
  stride_B = cutlass::make_cute_packed_stride(StrideB{}, {options.n, options.k, 1});
  stride_C = cutlass::make_cute_packed_stride(StrideC{}, {options.m, options.n, 1});
  stride_D = cutlass::make_cute_packed_stride(StrideD{}, {options.m, options.n, 1});
  layout_A = SparseConfig::fill_layoutA(problem_shape);
  layout_E = SparseConfig::fill_layoutE(problem_shape);
  layout_SFA = Sm1xxBlkScaledConfig::tile_atom_to_shape_SFA(problem_shape);
  layout_SFB = Sm1xxBlkScaledConfig::tile_atom_to_shape_SFB(problem_shape);

  // * Get ACompress & E size
  CompressorUtility compressor_utility(problem_shape, stride_A);

  // TensorE
  // In unit of ElementE (uint8_t), after alignment requirement
  // M-dim: TensorEAtom_M alignment
  // K-dim: TensorEAtom_K alignment
  int KAlignedE = compressor_utility.get_metadata_k_physical();
  int MAlignedE = compressor_utility.get_metadata_m_physical();

  // TensorA Compressed
  // In unit of ElementARaw, after alignment requirement
  // M-dim: TMA alignment
  // K-dim: TMA alignment
  int KAlignedAC = compressor_utility.get_tensorA_k_physical();
  int MAlignedAC = compressor_utility.get_tensorA_m_physical();

  stride_A_compressed = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(options.m, KAlignedAC, options.l));
  stride_E            = cutlass::make_cute_packed_stride(StrideE{}, cute::make_shape(MAlignedE, KAlignedE, options.l));

  // * Get SFA & SFB size
  auto k_blks = cutlass::ceil_div(options.k, cute::size<1>(shape(SfAtom{})));
  auto m_blks = cutlass::ceil_div(options.m, Blk_MN{});
  auto n_blks = cutlass::ceil_div(options.n, Blk_MN{});

  // * Allocate Tensor
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Relies on tma-style bulk movement or descriptor handling. Carries sparse metadata or structured-sparsity constraints alongside math operands.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。依赖 TMA 风格的批量搬运或描述符处理。 在数学操作数之外还携带稀疏元数据或结构化稀疏约束。

---

```cpp
  auto a_coord = cutlass::make_Coord(options.m * options.l, options.k);
  auto b_coord = cutlass::make_Coord(options.k, options.n * options.l);
  auto e_coord = cutlass::make_Coord(MAlignedE * options.l, KAlignedE);
  auto a_comp_coord = cutlass::make_Coord(MAlignedAC * options.l, KAlignedAC);
  auto c_coord = cutlass::make_Coord(options.m * options.l, options.n);
  auto d_coord = cutlass::make_Coord(options.m * options.l, options.n);
  auto sfa_coord   = cutlass::make_Coord(m_blks * Blk_MN{} * options.l, k_blks * Blk_SF{});
  auto sfb_coord   = cutlass::make_Coord(n_blks * Blk_MN{} * options.l, k_blks * Blk_SF{});

  tensor_A.resize(a_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagA>::layout_factory(a_coord, stride_factor_A));
  tensor_A_compressed.resize(a_comp_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagA>::layout_factory(a_comp_coord, stride_factor_A));
  tensor_B.resize(b_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagB>::layout_factory(b_coord, stride_factor_B));
  tensor_E.resize(e_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagE>::layout_factory(e_coord, stride_factor_E));
  tensor_C.resize(c_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagC>::layout_factory(c_coord, stride_factor_C));
  tensor_D.resize(c_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagD>::layout_factory(d_coord, stride_factor_D));
  reference_D.resize(c_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagD>::layout_factory(d_coord, stride_factor_D), false);
  tensor_SFA.resize(sfa_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagA>::layout_factory(sfa_coord, stride_factor_A));
  tensor_SFB.resize(sfb_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagB>::layout_factory(sfb_coord, stride_factor_B));

  // * Random init
  initialize_tensor(tensor_A.host_view(), seed + 2021);
  initialize_tensor(tensor_B.host_view(), seed + 2022);
  initialize_tensor(tensor_C.host_view(), seed + 2023);
  initialize_tensor(tensor_SFA.host_view(), seed + 2024);
  initialize_tensor(tensor_SFB.host_view(), seed + 2025);
  cutlass::reference::host::TensorCopy(reference_D.host_view(), tensor_C.host_view());

  // * Random fill 50% A with zero
  compressor_utility.structure_sparse_zero_mask_fill(tensor_A.host_data(), static_cast<int>(seed + 2023));

  tensor_A.sync_device();
  tensor_B.sync_device();
  tensor_C.sync_device();
  tensor_SFA.sync_device();
  tensor_SFB.sync_device();

  // * Compress
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Carries sparse metadata or structured-sparsity constraints alongside math operands.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。在数学操作数之外还携带稀疏元数据或结构化稀疏约束。

---

```cpp
  cutlass::KernelHardwareInfo hw_info;
  hw_info.device_id = 0;
  hw_info.sm_count = cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);
  typename Compressor::Arguments arguments{
    problem_shape,
    {tensor_A.device_data(),
      stride_A,
      tensor_A_compressed.device_data(),
      tensor_E.device_data()},
    {hw_info}
  };

  Compressor compressor_op;
  size_t workspace_size = Compressor::get_workspace_size(arguments);
  cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);

  cutlass::Status status {cutlass::Status::kSuccess };
  status = compressor_op.can_implement(arguments);
  if (status != cutlass::Status::kSuccess) {
    return false;
  }

  status = compressor_op.initialize(arguments, workspace.get());
  if (status != cutlass::Status::kSuccess) {
    return false;
  }

  status = compressor_op.run();
  if (status != cutlass::Status::kSuccess) {
    return false;
  }

  auto result = cudaDeviceSynchronize();
  if (result != cudaSuccess) {
    return false;
  }

  tensor_E.sync_host();
  tensor_A_compressed.sync_host();
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Carries sparse metadata or structured-sparsity constraints alongside math operands.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。在数学操作数之外还携带稀疏元数据或结构化稀疏约束。

---

```cpp
  return true;
}

// Populates a Gemm::Arguments structure from the given commandline options
typename Gemm::Arguments args_from_options(const Options &options)
{
  using ArrayElementA = typename Gemm::GemmKernel::CollectiveMainloop::ArrayElementA;
  using ArrayElementB = typename Gemm::GemmKernel::CollectiveMainloop::ArrayElementB;

  typename Gemm::Arguments arguments {
    cutlass::gemm::GemmUniversalMode::kGemm,
    {options.m, options.n, options.k, 1},
    {
      reinterpret_cast<ArrayElementA *>(tensor_A_compressed.device_data()), layout_A,
      reinterpret_cast<ArrayElementB *>(tensor_B.device_data()), stride_B,
      tensor_E.device_data(), layout_E,
      tensor_SFA.device_data(), layout_SFA,
      tensor_SFB.device_data(), layout_SFB
    },
    {
      {options.alpha, options.beta},
      tensor_C.device_data(), stride_C,
      tensor_D.device_data(), stride_D
    }
  };

  return arguments;
}

bool verify(const Options &options) {
  using namespace cute;

  // Create the arguments for host reference implementation
  auto A = make_tensor(make_iterator(tensor_A.host_data()), layout_A);
  auto SFA = make_tensor(tensor_SFA.host_data(), layout_SFA);
  auto B = make_tensor(make_iterator(tensor_B.host_data()),
    make_layout(make_shape(options.n, options.k, options.l), stride_B));
  auto SFB = make_tensor(tensor_SFB.host_data(), layout_SFB);
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Carries sparse metadata or structured-sparsity constraints alongside math operands.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。在数学操作数之外还携带稀疏元数据或结构化稀疏约束。

---

```cpp
  cutlass::reference::host::GettMainloopParams<
      ElementAccumulator,
      decltype(A),
      decltype(B),
      decltype(SFA),
      decltype(SFB)> mainloop_params{A, SFA, B, SFB};

  auto C = make_tensor(make_iterator(tensor_C.host_data()),
    make_layout(make_shape(options.m, options.n, options.l), stride_C));
  auto D = make_tensor(make_iterator(reference_D.host_data()),
    make_layout(make_shape(options.m, options.n, options.l), stride_D));

  cutlass::reference::host::GettBlockScalingEpilogueParams<
      ElementAccumulator,                   // ElementScalar
      ElementAccumulator,                   // ElementAccumulator
      ElementAccumulator,                   // ElementCompute
      decltype(C),                          // TensorC
      decltype(D)                           // TensorD
    > epilogue_params{
      options.alpha,
      options.beta,
      C,
      D};

  cutlass::reference::host::Gemm3x(mainloop_params, epilogue_params);

  // Comparison
  tensor_D.sync_host();
  bool passed = cutlass::reference::host::TensorEquals(reference_D.host_view(), tensor_D.host_view());
  passed &= (cutlass::reference::host::TensorNorm(reference_D.host_view()) > 0);
  passed &= (cutlass::reference::host::TensorNorm(tensor_D.host_view()) > 0);

  return passed;
}

/// Execute a given example GEMM computation
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Relies on tma-style bulk movement or descriptor handling. Carries sparse metadata or structured-sparsity constraints alongside math operands.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。依赖 TMA 风格的批量搬运或描述符处理。 在数学操作数之外还携带稀疏元数据或结构化稀疏约束。

---

```cpp
template <typename Gemm>
int run(Options &options)
{
  auto init_pass = initialize(options);
  if (not init_pass) {
    std::cout << "Initialization failure" << std::endl;
    exit(EXIT_FAILURE);
  }

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

  // Initialize CUTLASS kernel with arguments and workspace pointer
  CUTLASS_CHECK(gemm.initialize(arguments, workspace.get()));

  // Correctness / Warmup iteration
  CUTLASS_CHECK(gemm.run());

  cudaDeviceSynchronize();

  // Check if output from CUTLASS kernel and reference kernel are equal or not
  Result result;
  result.passed = verify(options);

  std::cout << "  Disposition: " << (result.passed ? "Passed" : "Failed") << std::endl;

  if (not result.passed) {
    exit(-1);
  }
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Carries sparse metadata or structured-sparsity constraints alongside math operands.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。在数学操作数之外还携带稀疏元数据或结构化稀疏约束。

---

```cpp
  // Run profiling loop
  if (options.iterations > 0)
  {
    GpuTimer timer;
    timer.start();
    for (int iter = 0; iter < options.iterations; ++iter) {
      CUTLASS_CHECK(gemm.initialize(arguments, workspace.get()));
      CUTLASS_CHECK(gemm.run());
    }
    timer.stop();

    // Compute average runtime and GFLOPs.
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Carries sparse metadata or structured-sparsity constraints alongside math operands.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。在数学操作数之外还携带稀疏元数据或结构化稀疏约束。

---

```cpp
    float elapsed_ms = timer.elapsed_millis();
    result.avg_runtime_ms = double(elapsed_ms) / double(options.iterations);
    result.gflops = options.gflops(result.avg_runtime_ms / 1000.0);
```

**EN**: This block wires CUTLASS collective types together. It is where architecture tags, operator classes, operand layouts, tile/cluster shapes, and stage policies are assembled so the library can deduce a concrete mainloop or epilogue implementation. Carries sparse metadata or structured-sparsity constraints alongside math operands.  
**CN**: 这一块负责把 CUTLASS 的 collective 类型真正接起来：架构标签、算子类别、操作数布局、tile/cluster 形状以及 stage 策略都会在这里组合，从而让库推导出具体的 mainloop 或 epilogue 实现。在数学操作数之外还携带稀疏元数据或结构化稀疏约束。

---

```cpp
    std::cout << "  Problem Size: " << options.m << 'x' << options.n << 'x' << options.k << std::endl;
    std::cout << "  Avg runtime: " << result.avg_runtime_ms << " ms" << std::endl;
    std::cout << "  GFLOPS: " << result.gflops << std::endl;
  }

  return 0;
}

#endif // defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Targets blackwell sm100 execution paths. Carries sparse metadata or structured-sparsity constraints alongside math operands.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。面向 Blackwell SM100 执行路径。 在数学操作数之外还携带稀疏元数据或结构化稀疏约束。

---

```cpp
int main(int argc, char const **args) {

  // CUTLASS must be compiled with CUDA 12.8 or higher Toolkit to run this example
  // and must have compute capability at least 100.
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
  if (not (props.major == 10 && props.minor == 0)) {
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

  //
  // Evaluate CUTLASS kernels
  //
#if defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)
  run<Gemm>(options);
#endif // defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)

  return 0;
}
```

**EN**: This block forms the outer control flow of the sample. It performs capability checks, selects the concrete specialization to run, invokes the operation, and reports the final status or throughput back to the user. Targets blackwell sm100 execution paths. Carries sparse metadata or structured-sparsity constraints alongside math operands.  
**CN**: 这一块构成了示例的最外层控制流：先做能力检查，再选择具体特化版本，调用 operation，最后把状态或吞吐结果回报给用户。面向 Blackwell SM100 执行路径。 在数学操作数之外还携带稀疏元数据或结构化稀疏约束。

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
