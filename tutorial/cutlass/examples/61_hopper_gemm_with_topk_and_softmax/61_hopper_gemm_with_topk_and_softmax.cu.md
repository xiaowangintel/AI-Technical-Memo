# 61_hopper_gemm_with_topk_and_softmax.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/61_hopper_gemm_with_topk_and_softmax/61_hopper_gemm_with_topk_and_softmax.cu`  
**Purpose / 用途**: Demonstrates a Hopper SM90 CUTLASS GEMM whose epilogue fuses Top-K filtering and Softmax over N, then validates and profiles it. / 演示一个面向 Hopper SM90 的 CUTLASS GEMM，在 epilogue 中沿 N 维融合 Top-K 过滤与 Softmax，并对结果进行校验与性能分析。

---

## Line-by-Line Analysis / 逐行分析

**Lines 1-30**
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
```
- **EN**: Standard BSD-3-Clause license header. It is legal metadata only and does not affect kernel behavior.
- **CN**: 标准 BSD-3-Clause 许可证头，仅提供法律声明，不影响内核行为。

**Lines 31-55**
```cpp

/*! \file
    \brief  Hopper GEMM + Top-K + Softmax fusion

    This example illustrates how to use the LinCombTopKSoftmaxCol EVT node to fuse
    Top-K and Softmax into the GEMM epilogue, with certain assumptions made.

    Those assumptions are as:
      1. Fusion is over the N dimension.
      2. Top-K value is static (meaning multiple kernels have to be compiled to support 
         different values.)
         * NOTE: Only K=2 and K=4 cases are performance-optimized and enabled by default.
           There is also a generic sort that supports all K values greater than 1, but it can lead to serious performance implications to the underlying kernel.
           If necessary, users can simply remove the K==2 || K ==4 assertion under cutlass/epilogue/fusion/sm90_visitor_topk_softmax.hpp, and the generic sort will automatically be used for all other Ks.
      3. The GEMM tile shape along N is greater than or equal to problem size
         along N.


    The example runs the fused GEMM kernel, along with a standard unfused host reference, and
    manually performs Top-K and softmax, and compares the error between tensors.

    Note that some numerical error (smaller than 1e-5) is to be expected, but this is true
    in most efficient reduction kernels, because floating point addition is not necessarily
    associative.
*/
```
- **EN**: The file-level comment explains the example: a Hopper GEMM whose epilogue fuses Top-K and Softmax along N. It also states key constraints: static K, preferred K=2/K=4, tile-N must cover problem N, and small floating-point error is expected.
- **CN**: 文件级注释说明该示例：一个面向 Hopper 的 GEMM，在 epilogue 中沿 N 维融合 Top-K 与 Softmax。这里还给出关键约束：K 为编译期常量、优先优化 K=2/K=4、tile 的 N 方向需覆盖问题规模 N，并且允许小的浮点误差。

**Lines 56-84**
```cpp

#include <iostream>

#include "cutlass/cutlass.h"
#include "cutlass/numeric_types.h"

#include "cute/tensor.hpp"
#include "cutlass/tensor_ref.h"
#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/gemm/collective/collective_builder.hpp"
#include "cutlass/gemm/device/gemm_universal_adapter.h"
#include "cutlass/gemm/kernel/gemm_universal.hpp"
#include "cutlass/epilogue/dispatch_policy.hpp"
#include "cutlass/epilogue/collective/collective_builder.hpp"

#include "cutlass/util/command_line.h"
#include "cutlass/util/distribution.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/packed_stride.hpp"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/reference/host/error_metrics.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_norm.h"
#include "cutlass/util/reference/host/gett.hpp"


#include "helper.h"
```
- **EN**: These headers pull in CUTLASS/CUTE tensor abstractions, GEMM collective builders, epilogue builders, host-side utilities, and helper timing/runtime support. Together they enable both the fused device kernel and the host reference path.
- **CN**: 这些头文件引入 CUTLASS/CUTE 张量抽象、GEMM collective builder、epilogue builder、主机端工具以及 helper 中的计时/运行时支持，从而同时支撑融合后的设备内核与主机参考实现。

**Lines 85-92**
```cpp

using namespace cute;

#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)

static constexpr int TopK = 2;
static constexpr bool EnableTopKSoftmax = TopK > 1;

```
- **EN**: The example switches to CUTE names, gates the Hopper-specific path behind SM90 support, and fixes TopK=2. `EnableTopKSoftmax` decides whether the epilogue uses the fused visitor or falls back to a plain linear combination.
- **CN**: 代码启用 CUTE 命名空间，并用 SM90 宏保护 Hopper 专用路径，同时固定 `TopK=2`。`EnableTopKSoftmax` 用来决定 epilogue 采用融合 visitor，还是退化为普通线性组合。

**Lines 93-126**
```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
/// GEMM kernel configurations
/////////////////////////////////////////////////////////////////////////////////////////////////

// A matrix configuration
using         ElementA    = cutlass::half_t;                                // Element type for A matrix operand
using         LayoutA     = cutlass::layout::RowMajor;                      // Layout type for A matrix operand
constexpr int AlignmentA  = 128 / cutlass::sizeof_bits<ElementA>::value;    // Memory access granularity/alignment of A matrix in units of elements (up to 16 bytes)

// B matrix configuration
using         ElementB    = cutlass::half_t;                                // Element type for B matrix operand
using         LayoutB     = cutlass::layout::ColumnMajor;                   // Layout type for B matrix operand
constexpr int AlignmentB  = 128 / cutlass::sizeof_bits<ElementB>::value;    // Memory access granularity/alignment of B matrix in units of elements (up to 16 bytes)

// C matrix configuration
using         ElementC    = void;
using         LayoutC     = cutlass::layout::RowMajor;
constexpr int AlignmentC  = 1;

// D matrix configuration
using         ElementD    = cutlass::half_t;                                // Element type for C and D matrix operands
using         LayoutD     = cutlass::layout::RowMajor;                      // Layout type for output
constexpr int AlignmentD  = 128 / cutlass::sizeof_bits<ElementD>::value;    // Memory access granularity/alignment of output in units of elements (up to 16 bytes)

// Core kernel configurations
using ElementAccumulator  = float;                                          // Element type for internal accumulation
using ElementCompute      = float;                                          // Element type for epilogue computation
using ArchTag             = cutlass::arch::Sm90;                            // Tag indicating the minimum SM that supports the intended feature
using OperatorClass       = cutlass::arch::OpClassTensorOp;                 // Operator class tag
using TileShape           = Shape<_64,_64,_128>;                            // Threadblock-level tile size
using ClusterShape        = Shape<_1,_1,_1>;                                // Shape of the threadblocks in a cluster
using KernelSchedule      = cutlass::gemm::KernelTmaWarpSpecialized;
using EpilogueSchedule    = cutlass::epilogue::TmaWarpSpecialized;

```
- **EN**: This block defines the operand/data-layout contract for the kernel: A/B/D use FP16, A and D are row-major, B is column-major, and accumulation/epilogue math use FP32. The Hopper composition is also fixed here: SM90 tensor cores, a 64x64x128 tile, single-CTA cluster, and TMA warp-specialized mainloop/epilogue schedules.
- **CN**: 这一段定义了内核的操作数与数据布局契约：A/B/D 使用 FP16，A 与 D 为行主序，B 为列主序，而累加与 epilogue 计算使用 FP32。Hopper 的内核组合也在此确定：SM90 Tensor Core、64x64x128 tile、单 CTA cluster，以及基于 TMA 的 warp-specialized mainloop/epilogue 调度。

**Lines 127-145**
```cpp
// Top-K + Softmax fusion operation
using FusionOperation     = std::conditional_t<EnableTopKSoftmax,
  typename cutlass::epilogue::fusion::LinCombTopKSoftmaxCol<TopK, ElementD, ElementCompute>,
  typename cutlass::epilogue::fusion::LinearCombination<ElementD, ElementCompute, ElementC, ElementCompute>
>;

// The fusion op only allows for epilogue tiles matching the mainloop tile.
using EpilogueTileType    = decltype(cute::take<0,2>(TileShape{}));

using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    TileShape, ClusterShape,
    EpilogueTileType,
    ElementAccumulator, ElementCompute,
    ElementC, LayoutC, AlignmentC,
    ElementD, LayoutD, AlignmentD,
    EpilogueSchedule,
    FusionOperation
  >::CollectiveOp;
```
- **EN**: The fusion operator is selected with a compile-time conditional. When enabled, `LinCombTopKSoftmaxCol<TopK,...>` performs scaling plus column-wise Top-K/Softmax in the epilogue; otherwise a standard linear combination is used. The epilogue tile is constrained to match the mainloop tile, then `CollectiveEpilogue` is built from those types.
- **CN**: 融合算子通过编译期条件选择。启用时，`LinCombTopKSoftmaxCol<TopK,...>` 在 epilogue 中执行缩放以及按列方向（对应 N 维）的 Top-K/Softmax；否则使用标准线性组合。这里还要求 epilogue tile 与 mainloop tile 匹配，随后据此构建 `CollectiveEpilogue`。

**Lines 146-188**
```cpp

using CollectiveMainloop = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    ElementA, LayoutA, AlignmentA,
    ElementB, LayoutB, AlignmentB,
    ElementAccumulator,
    TileShape, ClusterShape,
    cutlass::gemm::collective::StageCountAutoCarveout<
      static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))
    >,
    KernelSchedule
  >::CollectiveOp;

using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
    Shape<int,int,int,int>, // Indicates ProblemShape
    CollectiveMainloop,
    CollectiveEpilogue
>;

using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;

// Extract information from Gemm kernel.
using EpilogueOutputOp  = typename Gemm::EpilogueOutputOp;
using ElementScalar     = typename EpilogueOutputOp::ElementScalar;

using StrideA = typename Gemm::GemmKernel::StrideA;
using StrideB = typename Gemm::GemmKernel::StrideB;
using StrideD = typename Gemm::GemmKernel::StrideD;

/// Initialization
StrideA stride_A;
StrideB stride_B;
StrideD stride_D;
uint64_t seed;

cutlass::HostTensor<ElementA  , LayoutA  > tensor_A;
cutlass::HostTensor<ElementB  , LayoutB  > tensor_B;
cutlass::HostTensor<ElementD  , LayoutD  > tensor_D;
cutlass::HostTensor<ElementD  , LayoutD  > tensor_ref_D;

using LayoutScalar = cutlass::layout::PackedVectorLayout;

#endif // defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
```
- **EN**: This block builds the Hopper mainloop collective, carving stage count around epilogue shared-memory usage. It then forms the universal GEMM kernel/device adapter, extracts epilogue scalar and stride types, and declares global host tensors plus stride state used throughout initialization, execution, and verification. The dataflow is A/B host fill -> device sync -> GEMM writes D -> host sync for checking.
- **CN**: 这一段构建 Hopper mainloop collective，并依据 epilogue 共享内存占用自动调整 stage 数。随后定义 universal GEMM kernel/device adapter，提取 epilogue 标量与 stride 类型，并声明初始化、执行、校验阶段都会使用的全局张量与步长状态。整体数据流为：主机填充 A/B -> 同步到设备 -> GEMM 写入 D -> 再同步回主机校验。

**Lines 189-218**
```cpp

/////////////////////////////////////////////////////////////////////////////////////////////////
/// Testbed utility types
/////////////////////////////////////////////////////////////////////////////////////////////////

// Command line options parsing
struct Options {

  bool help = false;

  int iterations = 1000;
  int m = 16, n = 8, k = 64, l = 1;
  double eps = 1e-5;

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
    cmd.get_cmd_line_argument("iterations", iterations);
    cmd.get_cmd_line_argument("eps", eps);
  }
```
- **EN**: `Options` stores runtime parameters and parses command-line flags. The example exposes matrix extents, batch count `l`, iteration count, and error tolerance, which makes the same compiled kernel easy to benchmark under different problem sizes.
- **CN**: `Options` 保存运行时参数并解析命令行。示例暴露了矩阵尺寸、批次维 `l`、迭代次数和误差阈值，因此同一个已编译内核可以方便地在不同问题规模下做基准测试。

**Lines 219-253**
```cpp

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << "61_hopper_gemm_with_topk_and_softmax\n\n"
      << "  Hopper FP8 GEMM with Top-K and softmax fusion.\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement\n\n"
      << "  --m=<int>                   Sets the M extent of the GEMM\n"
      << "  --n=<int>                   Sets the N extent of the GEMM\n"
      << "  --k=<int>                   Sets the K extent of the GEMM\n"
      << "  --l=<int>                   Sets the l extent (batch) of the GEMM\n"
      << "  --iterations=<int>          Number of profiling iterations to perform.\n\n"
      << "  --eps=<float>               Threshold of numerical verification. Default: 1e-5.\n\n";

    out
      << "\n\nExamples:\n\n"
      << "$ " << "61_hopper_gemm_with_topk_and_softmax" << " --m=16 --n=8 --k=1024 \n\n";

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

  float alpha() const {
    return 1.f / static_cast<float>(k);
  }
};
```
- **EN**: The usage printer documents the executable interface. `gflops()` computes the conventional 2*M*N*K cost, while `alpha()` returns `1/k`; that scale is folded into the epilogue so the GEMM output is normalized before the later Top-K/Softmax comparison.
- **CN**: 这里的帮助输出描述了可执行程序的接口。`gflops()` 按传统公式计算 2*M*N*K 的代价，而 `alpha()` 返回 `1/k`；这个缩放会被折叠进 epilogue，使 GEMM 输出在后续 Top-K/Softmax 对比前先完成归一化。

**Lines 254-272**
```cpp

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
- **EN**: `Result` is a small record for runtime, throughput, CUTLASS/CUDA status, and pass/fail state. The constructor defaults everything to a safe success-like baseline, then later verification updates `passed`.
- **CN**: `Result` 是一个轻量结果结构，记录运行时间、吞吐率、CUTLASS/CUDA 状态以及是否通过校验。构造函数先给出安全的默认“成功”基线，随后由校验逻辑更新 `passed`。

**Lines 273-288**
```cpp

#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)

/////////////////////////////////////////////////////////////////////////////////////////////////
/// GEMM setup and evaluation
/////////////////////////////////////////////////////////////////////////////////////////////////

/// Helper to initialize a block of device data
template <typename Element, typename Layout>
bool initialize_tensor(
    cutlass::TensorView<Element, Layout> view,
    uint64_t seed) {
  cutlass::reference::host::TensorFillRandomUniform(
    view, seed, /* max = */ 1, /* min = */ -1, /* bits = */ 2);
  return true;
}
```
- **EN**: The code re-enters the SM90-only region and defines a helper that fills tensors with small random values in [-1, 1]. Keeping the input range narrow makes correctness checking more stable for fused reductions.
- **CN**: 代码重新进入仅限 SM90 的区域，并定义辅助函数，用 [-1, 1] 范围内的小随机值填充张量。较窄的输入范围有助于让融合归约场景下的正确性对比更稳定。

**Lines 289-312**
```cpp

/// Initialize operands to be used in the GEMM and reference GEMM
void initialize(const Options &options) {

  stride_A = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(options.m, options.k, options.l));
  stride_B = cutlass::make_cute_packed_stride(StrideB{}, cute::make_shape(options.n, options.k, options.l));
  stride_D = cutlass::make_cute_packed_stride(StrideD{}, cute::make_shape(options.m, options.n, options.l));

  auto a_coord = cutlass::make_Coord(options.m * options.l, options.k);
  auto c_coord = cutlass::make_Coord(options.m * options.l, options.n);
  auto b_coord = cutlass::make_Coord(options.k, options.n * options.l);

  tensor_A.resize(a_coord);
  tensor_B.resize(b_coord);
  tensor_D.resize(c_coord);
  tensor_ref_D.resize(c_coord);

  initialize_tensor(tensor_A.host_view(), seed + 2022);
  initialize_tensor(tensor_B.host_view(), seed + 2023);

  tensor_A.sync_device();
  tensor_B.sync_device();
  tensor_D.sync_device();
}
```
- **EN**: `initialize()` derives packed strides from `(m,k,l)`, `(n,k,l)`, and `(m,n,l)`, allocates host/device-backed tensors, fills A and B, and synchronizes them to the GPU. `tensor_ref_D` stays on the host for the unfused reference path, while `tensor_D` becomes the device output buffer for the fused GEMM.
- **CN**: `initialize()` 根据 `(m,k,l)`、`(n,k,l)` 与 `(m,n,l)` 生成 packed stride，分配主机/设备双端张量，填充 A 与 B，并同步到 GPU。`tensor_ref_D` 保留在主机端用于非融合参考路径，而 `tensor_D` 作为融合 GEMM 的设备输出缓冲区。

**Lines 313-328**
```cpp

/// Populates a Gemm::Arguments structure from the given commandline options
typename Gemm::Arguments args_from_options(const Options &options) {
  typename Gemm::Arguments arguments{
    cutlass::gemm::GemmUniversalMode::kGemm,
    {options.m, options.n, options.k, options.l},
    {tensor_A.device_data(), stride_A, tensor_B.device_data(), stride_B},
    {
      {options.alpha(), 0.f}, // alpha, beta
      nullptr, stride_D,
      tensor_D.device_data(), stride_D
    }
  };

  return arguments;
}
```
- **EN**: `args_from_options()` converts the runtime configuration into `Gemm::Arguments`. The problem shape, tensor pointers, strides, and epilogue scalars are packed here; `C` is `nullptr` because this example does not consume a source C tensor, only writes D.
- **CN**: `args_from_options()` 将运行时配置打包成 `Gemm::Arguments`。这里封装了问题形状、张量指针、步长以及 epilogue 标量；由于该示例不读取源张量 C、只写 D，所以 `C` 指针被设为 `nullptr`。

**Lines 329-365**
```cpp

bool verify(const Options &options) {
  //
  // Compute reference output
  //

  // Create instantiation for device reference gemm kernel
  auto A = cute::make_tensor(tensor_A.host_data(),
      cute::make_layout(cute::make_shape(options.m, options.k, options.l), stride_A));
  auto B = cute::make_tensor(tensor_B.host_data(),
      cute::make_layout(cute::make_shape(options.n, options.k, options.l), stride_B));
  auto D = cute::make_tensor(tensor_ref_D.host_data(),
      cute::make_layout(cute::make_shape(options.m, options.n, options.l), stride_D));
  using unused_t = decltype(D);

  cutlass::reference::host::GettMainloopParams<ElementAccumulator, decltype(A), decltype(B)> mainloop_params{A, B};

  cutlass::reference::host::GettEpilogueParams<
      ElementScalar,
      ElementScalar,
      ElementAccumulator,
      ElementCompute,
      unused_t,
      decltype(D),
      unused_t, // bias
      unused_t, // aux
      unused_t, // valpha
      unused_t  // vbeta
  > epilogue_params;

  epilogue_params.D = D;
  epilogue_params.alpha = options.alpha();
  epilogue_params.beta = 0.f;

  // get reference result
  cutlass::reference::host::Gemm3x(mainloop_params, epilogue_params);

```
- **EN**: The first half of `verify()` builds a host reference GEMM path. It wraps raw host buffers into CUTE tensors with the same strides as the device kernel, prepares `GettMainloopParams` and `GettEpilogueParams`, sets alpha/beta, and calls `Gemm3x()` to produce the unfused reference output.
- **CN**: `verify()` 的前半部分构建主机端参考 GEMM 路径。它把原始主机缓冲区包装成与设备内核一致步长的 CUTE 张量，准备 `GettMainloopParams` 和 `GettEpilogueParams`，设置 alpha/beta，然后调用 `Gemm3x()` 生成未融合的参考输出。

**Lines 366-385**
```cpp
  if constexpr (EnableTopKSoftmax) {
    // top-K + softmax
    for (int i = 0; i < options.m; ++i) {

      // Find Top-K
      cutlass::Array<ElementAccumulator, TopK> top_k;
      top_k.fill(-cutlass::platform::numeric_limits<ElementCompute>::infinity());
      for (int j = 0; j < options.n; ++j) {
        auto val = static_cast<ElementAccumulator>(tensor_ref_D.host_view().ref().at({i, j}));
        for (int top_k_idx = 0; top_k_idx < TopK; ++top_k_idx) {
          if (val > top_k[top_k_idx]) {
            // Shift down
            for (int l = TopK - 1; l > top_k_idx; --l) {
              top_k[l] = top_k[l - 1];
            }
            top_k[top_k_idx] = val;
            break;
          }
        }
      }
```
- **EN**: When fusion is enabled, verification manually reproduces the Top-K stage row by row over N. A fixed-size array tracks the current best values in descending order by insertion, mirroring the epilogue visitor’s intent without reproducing its exact device implementation details.
- **CN**: 当启用融合时，校验逻辑会沿 N 维逐行手工重现 Top-K 阶段。一个固定大小的数组通过插入式更新保持降序的最优值，这在语义上对应 epilogue visitor 的目标，但不逐字复制设备端实现细节。

**Lines 386-412**
```cpp

      // This formulation of top-K + softmax only works when it is
      // guaranteed that none of the top-K elements are repeated!
      // If this is the case, the device kernel can also make mistakes, because
      //   A. Once the top-K values are reduced, and the operation is being applied,
      //      there is no way to tell repeated elements apart, so none are masked.
      //   B. The softmax sum of exps will be incorrect (because the repeated elements
      //      are not repeated in it.)

      ElementAccumulator max = top_k[0];
      ElementAccumulator sum = ElementAccumulator(0.f);
      for (int top_k_idx = 0; top_k_idx < TopK; ++top_k_idx) {
        sum = sum + cutlass::fast_exp(top_k[top_k_idx] - max);
      }

      for (int j=0; j < options.n; ++j) {
        auto val = tensor_ref_D.host_view().ref().at({i, j});
        if (val < top_k[TopK - 1]) {
          tensor_ref_D.host_view().ref().at({i, j}) = static_cast<ElementD>(0.f);
        } else {
          // Softmax
          auto softmax_val = cutlass::fast_exp(val - max) / sum;
          tensor_ref_D.host_view().ref().at({i, j}) = static_cast<ElementD>(softmax_val);
        }
      }
    }
  }
```
- **EN**: This section explains an important correctness caveat: repeated Top-K values can break the masking semantics. It then performs the fused postprocessing model on the reference tensor: compute the Top-K max, accumulate exp-sum only over retained elements, zero everything below the K-th threshold, and write softmax-normalized values for the survivors. This is the key fused dataflow the example is demonstrating.
- **CN**: 这一段强调了一个重要正确性前提：如果 Top-K 值重复，掩码语义可能失效。随后它在参考张量上执行融合后处理模型：先求 Top-K 最大值，只对保留元素累加指数和，把低于第 K 大阈值的元素置零，并为保留下来的元素写回 softmax 归一化结果。这正是示例要展示的核心融合数据流。

**Lines 413-430**
```cpp

  // compare_reference
  tensor_D.sync_host();

  double err = cutlass::reference::host::TensorRelativeErrorMetric(
    tensor_D.host_view(),
    tensor_ref_D.host_view());
  bool passed = err < options.eps;

  if (options.m <= 32 && options.n <= 32) {
    std::cout << "GEMM output:\n" << tensor_D.host_view() << "\n\n";
    std::cout << "Reference output:\n" << tensor_ref_D.host_view() << "\n\n";
  }

  std::cout << "  Disposition: " << (passed ? "Passed" : "Failed") << " \t Relative error: " << err << std::endl;

  return passed;
}
```
- **EN**: After postprocessing, the fused device output is copied back, compared with a relative-error metric, and optionally printed for tiny problems. The pass/fail decision is based on `eps`, which acknowledges small numerical differences from reduction ordering.
- **CN**: 完成后处理后，程序把融合设备输出拷回主机，使用相对误差指标与参考结果比较，并在小问题规模下打印张量。是否通过由 `eps` 决定，用来容忍归约次序差异带来的微小数值偏差。

**Lines 431-488**
```cpp

/// Execute a given example GEMM computation
template <typename Gemm>
int run(Options &options) {
  initialize(options);

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

  // Check if output from CUTLASS kernel and reference kernel are equal or not
  Result result;
  result.passed = verify(options);

  if (!result.passed) {
    exit(-1);
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

#endif // defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
```
- **EN**: `run()` drives the full experiment: initialize data, instantiate the Hopper GEMM, build arguments, allocate workspace, check support, initialize the kernel, run once for correctness/warmup, verify against the host model, then profile repeated launches. This is where the collective mainloop and fused epilogue composition become an executable kernel pipeline.
- **CN**: `run()` 驱动整个实验流程：初始化数据、实例化 Hopper GEMM、构造参数、分配 workspace、检查可实现性、初始化内核、先运行一次做正确性/预热，再与主机模型校验，最后进入重复发射的性能测试。也正是在这里，collective mainloop 与融合 epilogue 的组合真正变成可执行的内核流水。

**Lines 489-500**
```cpp

///////////////////////////////////////////////////////////////////////////////////////////////////

int main(int argc, char const **args) {

  // CUTLASS must be compiled with CUDA 12.0 Toolkit to run this example
  // and must have compute capability at least 90.
  if (__CUDACC_VER_MAJOR__ < 12) {
    std::cerr << "This example requires CUDA 12 or newer.\n";
    // Returning zero so this test passes on older Toolkits. Its actions are no-op.
    return 0;
  }
```
- **EN**: `main()` begins with environment guards. The example requires CUDA 12+, but returns success on older toolkits so build/test environments without the right runtime do not fail spuriously.
- **CN**: `main()` 先进行环境保护检查。该示例要求 CUDA 12+，但在旧工具链上直接返回成功，从而避免不满足运行条件的构建/测试环境被误判为失败。

**Lines 501-511**
```cpp

  cudaDeviceProp props;
  int current_device_id;
  CUDA_CHECK(cudaGetDevice(&current_device_id));
  CUDA_CHECK(cudaGetDeviceProperties(&props, current_device_id));
  cudaError_t error = cudaGetDeviceProperties(&props, 0);
  if (props.major != 9 || props.minor != 0) {
    std::cerr
      << "This example requires a GPU of NVIDIA's Hopper Architecture (compute capability 90).\n";
    return 0;
  }
```
- **EN**: The program queries the active GPU and enforces Hopper (compute capability 9.0). There is a redundant second `cudaGetDeviceProperties()` call whose result is stored in `error` and not used further; it is harmless, but the real gate is the `props.major/minor` check.
- **CN**: 程序查询当前 GPU，并强制要求 Hopper（计算能力 9.0）。这里有一次冗余的第二次 `cudaGetDeviceProperties()` 调用，其返回值保存在 `error` 中但后续未使用；不过这并不影响功能，真正的门控条件是 `props.major/minor` 检查。

**Lines 512-527**
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
```
- **EN**: This block parses CLI options and handles `--help`. It cleanly separates setup from execution, making the example usable both as documentation and as a small benchmark binary.
- **CN**: 这一段负责解析命令行并处理 `--help`。它把配置准备与实际执行清晰分开，使该示例既能作为文档演示，也能作为一个小型基准程序使用。

**Lines 528-540**
```cpp

  //
  // Evaluate CUTLASS kernels
  //

#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
  run<Gemm>(options);
#endif

  return 0;
}

/////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN**: Finally, the code conditionally launches `run<Gemm>(options)` only when SM90 support was compiled in, then returns from `main()`. The trailing separator comment has no runtime effect and simply mirrors CUTLASS example style.
- **CN**: 最后，代码仅在编译时启用了 SM90 支持的情况下才调用 `run<Gemm>(options)`，随后从 `main()` 返回。末尾的分隔注释不影响运行，只是延续 CUTLASS 示例的一贯风格。

---

## Key Concepts / 关键概念

- **Hopper collective composition**: The kernel is assembled from CUTLASS mainloop and epilogue collectives specialized for SM90 TMA warp-specialized execution. / **Hopper 组合式内核**：该内核由 CUTLASS 的 mainloop 与 epilogue collective 组装而成，并针对 SM90 的 TMA warp-specialized 执行方式做了特化。
- **Fused epilogue behavior**: `LinCombTopKSoftmaxCol` pushes scaling, Top-K masking, and Softmax normalization into the GEMM epilogue so postprocessing can happen before writing final D values. / **融合式 epilogue 行为**：`LinCombTopKSoftmaxCol` 将缩放、Top-K 掩码和 Softmax 归一化推入 GEMM epilogue，使后处理可在最终写回 D 之前完成。
- **Dataflow**: A and B are initialized on the host, copied to device memory, consumed by GEMM, and the resulting D is copied back for host-side reference comparison. / **数据流**：A 和 B 在主机端初始化后拷入设备内存，被 GEMM 消费，生成的 D 再拷回主机，与主机侧参考结果比较。
- **Verification model**: The host path computes plain GEMM first, then manually applies Top-K and Softmax to approximate the fused device behavior. / **校验模型**：主机路径先计算普通 GEMM，再手工施加 Top-K 与 Softmax，以近似设备端的融合行为。
- **Practical constraints**: The example assumes fusion over N, compile-time TopK, and problem sizes compatible with the selected tile shape. / **实际约束**：该示例假设融合发生在 N 维、TopK 为编译期常量，且问题规模需与所选 tile 形状兼容。

## Dependencies / 依赖项

- **CUDA / GPU**: CUDA 12+ and an NVIDIA Hopper GPU with compute capability 9.0. / **CUDA / GPU**：需要 CUDA 12+ 和计算能力 9.0 的 NVIDIA Hopper GPU。
- **CUTLASS core**: `cutlass/cutlass.h`, GEMM collectives, epilogue collectives, universal adapter, tensor/layout helpers. / **CUTLASS 核心组件**：`cutlass/cutlass.h`、GEMM collective、epilogue collective、universal adapter 以及张量/布局辅助工具。
- **CUTE**: `cute/tensor.hpp` and shape/layout utilities for stride-aware tensor construction on the host. / **CUTE**：`cute/tensor.hpp` 及其 shape/layout 工具，用于在主机端构造带步长信息的张量。
- **Reference utilities**: host tensor fill/copy/compare/error routines and `Gemm3x` for the unfused reference path. / **参考实现工具**：主机端张量填充/拷贝/比较/误差例程，以及用于非融合参考路径的 `Gemm3x`。
- **Example helper**: `helper.h` supplies utility code such as CUDA checks and GPU timing used by the benchmark loop. / **示例辅助头**：`helper.h` 提供 CUDA 检查、GPU 计时等基准循环所需的辅助代码。
