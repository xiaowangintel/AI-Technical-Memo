# ampere_gemm_universal_streamk_broadcast.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/47_ampere_gemm_universal_streamk/ampere_gemm_universal_streamk_broadcast.cu`  
**Purpose / 用途**: This example contrasts classic data-parallel GEMM with Ampere Stream-K decomposition while also demonstrating a fused residual-style epilogue that adds a row-broadcast vector and two auxiliary matrices. It is a practical comparison of scheduling behavior, not just a kernel-definition sample. / 该示例把经典 data-parallel GEMM 与 Ampere Stream-K 分解方式放在同一个程序中对比，同时展示了一个融合 residual 风格 epilogue：它会叠加按行广播的向量和两个辅助矩阵。这不仅是内核定义样例，也是调度行为对比实验。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-173 — File overview, Ampere kernel configuration, and baseline GEMM types

```cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

/***************************************************************************************************
 Example contrasting the Stream-K parallel decomposition for GEMM threadblocks versus the
 "classic data-parallel" and "Split-K" decompositions + residual add.

 For more details regarding the Stream-K method, see "Stream-K: Work-centric Parallel Decomposition
 for Dense Matrix-Matrix Multiplication on the GPU" (https://arxiv.org/abs/2301.03598)

 Requires NVIDIA Ampere or newer device (SM80+).

 - To lock persistence mode, power (400W), clocks (1005MHz) for evaluation (assumes device 0 and A100)

     cutlass$ sudo nvidia-smi -pm 1 -i 0

     cutlass$ sudo nvidia-smi -i 0 -pl 400

     cutlass$ sudo nvidia-smi -i 0 -lgc 1005

 - Build and run:

     cutlass$ mkdir build

     cutlass$ cd build

     cutlass/build$ cmake .. -DCUTLASS_NVCC_ARCHS=80

     cutlass/build$ make 47_ampere_gemm_universal_streamk_broadcast

     cutlass/build$ ./examples/47_ampere_gemm_universal_streamk/47_ampere_gemm_universal_streamk_broadcast

 - Reset clocks when done:

     cutlass$ sudo nvidia-smi -rgc

 **************************************************************************************************/

#include <iostream>
#include <string>

#include "cutlass/cutlass.h"
#include "cutlass/gemm/device/gemm_universal.h"
#include "cutlass/gemm/device/gemm_universal_with_broadcast.h"
#include "cutlass/gemm/device/gemm_universal_streamk_with_broadcast.h"
#include "cutlass/epilogue/thread/linear_combination_residual_block.h"

#include "cutlass/util/command_line.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/reference/device/gemm.h"
#include "cutlass/util/reference/host/error_metrics.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_foreach.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/tensor_view_io.h"

#include "cutlass/epilogue/threadblock/fusion/visitors.hpp"
#include "cutlass/gemm/kernel/default_gemm_universal_with_visitor.h"
#include "cutlass/gemm/device/gemm_universal_adapter.h"

#include "helper.h"



/////////////////////////////////////////////////////////////////////////////////////////////////
/// GEMM kernel configurations (cutlass_tensorop_h16816gemm_128x128_32x4_nn_align8)
/////////////////////////////////////////////////////////////////////////////////////////////////

// A matrix configuration
using         ElementA         = cutlass::half_t;                                  // Element type for A matrix operand
using         LayoutA          = cutlass::layout::RowMajor;                        // Layout type for A matrix operand
constexpr int AlignmentA       = 128 / cutlass::sizeof_bits<ElementA>::value;      // Memory access granularity/alignment of A matrix in units of elements (up to 16 bytes)

// B matrix configuration
using         ElementB         = cutlass::half_t;                                  // Element type for B matrix operand
using         LayoutB          = cutlass::layout::RowMajor;                        // Layout type for B matrix operand
constexpr int AlignmentB       = 128 / cutlass::sizeof_bits<ElementB>::value;      // Memory access granularity/alignment of B matrix in units of elements (up to 16 bytes)

// C1/C2/D matrix configuration
using         ElementC         = cutlass::half_t;                                  // Element type for C matrix operands
using         LayoutC          = cutlass::layout::RowMajor;                        // Layout type for C matrix operands
constexpr int AlignmentC       = 128 / cutlass::sizeof_bits<ElementC>::value;      // Memory access granularity/alignment of C matrices in units of elements (up to 16 bytes)

// Output matrix configuration
using         ElementOutput    = cutlass::half_t;                                  // Element type for output matrix operands
using         LayoutOutput     = cutlass::layout::RowMajor;                        // Layout type for output matrix operands
// constexpr int AlignmentOutput  = 128 / cutlass::sizeof_bits<ElementOutput>::value; // Memory access granularity/alignment of output matrices in units of elements (up to 16 bytes)

// Multiply-accumulate blocking/pipelining details
using ElementAccumulator  = cutlass::half_t;                          // Element type for internal accumulation
using ElementCompute      = cutlass::half_t;                          // Element type for compute
using ArchTag             = cutlass::arch::Sm80;                      // Tag indicating the minimum SM that supports the intended feature
using OperatorClass       = cutlass::arch::OpClassTensorOp;           // Operator class tag
using ThreadblockShape    = cutlass::gemm::GemmShape<128, 128, 32>;   // Threadblock-level tile size (concept: GemmShape)
using WarpShape           = cutlass::gemm::GemmShape<64, 64, 32>;     // Warp-level tile size (concept: GemmShape)
using InstructionShape    = cutlass::gemm::GemmShape<16, 8, 16>;      // Instruction-level tile size (concept: GemmShape)
constexpr int NumStages   = 4;                                        // Number of global->shared pipeline stages used in the GEMM mainloop
constexpr int EVTEpilogueStages = 1;                                  // Number of epilogue stages in EVT

// Residual block configuration

// Epilogue output operator
/// Using LinearCombinationResidualBlock
/// Models a residual block of the form: UnaryOp(BinaryOp(BinaryOp(ActivationOp(TensorOp(X) + bias), residual1), residual2))
using EpilogueOp = cutlass::epilogue::thread::LinearCombinationResidualBlock<  
    ElementOutput,                        // Element type for output matrix
    ElementAccumulator,                   // Element type from internal accumulation
    ElementCompute,                       // Element type from internal accumulation
    ElementC,                             // Element type for C1/C2/D matrix operands
    AlignmentC,                           // Memory access granularity of C and D matrix in units of elements
    cutlass::epilogue::thread::Identity,  // Activation
    cutlass::plus,                        // Binary operation 1
    cutlass::epilogue::thread::Identity,  // Unary operation
    cutlass::plus                         // Binary operation 2
    >;

// Reference device GEMM implementation type
using DeviceGemmReference = cutlass::reference::device::Gemm<
  ElementA,
  LayoutA,
  ElementB,
  LayoutB,
  ElementC,
  LayoutC,
  ElementAccumulator,
  ElementAccumulator>;

// Classic data-parallel device GEMM implementation type
using DeviceGemmBasic = cutlass::gemm::device::GemmUniversalWithBroadcast<
    ElementA, LayoutA,
    ElementB, LayoutB,
    ElementC, LayoutC,
    ElementAccumulator,
    OperatorClass,
    ArchTag,
    ThreadblockShape,
    WarpShape,
    InstructionShape,
    EpilogueOp,
    cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<>,
    NumStages,
    AlignmentA,
    AlignmentB>;
```

**EN**: The large header comment frames the example as a comparison among classic data-parallel GEMM, Stream-K, and Split-K-like behavior. The type aliases configure an SM80 tensor-core kernel with half precision throughout, then define `EpilogueOp` as a residual-block style operation and instantiate both a reference GEMM and a baseline `GemmUniversalWithBroadcast`. In other words, this first section sets up the hardware target, numeric types, and the non-Stream-K baseline against which Stream-K will be measured.

**CN**: 开头的大段注释明确说明：这个示例要比较经典 data-parallel GEMM、Stream-K，以及类似 Split-K 的行为。随后的一组类型别名把内核固定为 SM80 上的 half 精度 Tensor Core 路径，并把 `EpilogueOp` 定义为 residual-block 风格操作，再给出参考 GEMM 与基线版 `GemmUniversalWithBroadcast`。因此这一段主要完成三件事：确定硬件目标、数值类型，以及后续用于对照的非 Stream-K 基线。

### Lines 174-260 — EVT visitor tree for broadcast and residual fusion

```cpp
// StreamK device GEMM implementation type with EVT
using namespace cute;

using OutputTileThreadMap = cutlass::epilogue::threadblock::OutputTileThreadLayout<
  ThreadblockShape, 
  WarpShape, 
  ElementC, 
  AlignmentC, 
  EVTEpilogueStages
>;

using Accum = cutlass::epilogue::threadblock::VisitorAccFetch;

using Bias = cutlass::epilogue::threadblock::VisitorRowBroadcast<
    OutputTileThreadMap, ElementC,
    cute::Stride<_0, _1, int32_t>  // StrideMNL
>;

using C1 = cutlass::epilogue::threadblock::VisitorAuxLoad<
    OutputTileThreadMap, ElementC, 
    cute::Stride<int64_t, _1, int64_t> // StrideMNL
>;

using C2 = cutlass::epilogue::threadblock::VisitorAuxLoad<
    OutputTileThreadMap, ElementC, 
    cute::Stride<int64_t, _1, int64_t> // StrideMNL
>;

using Compute0 = cutlass::epilogue::threadblock::VisitorCompute<
    cutlass::plus, ElementCompute, ElementCompute,
    cutlass::FloatRoundStyle::round_to_nearest
>;

using EVTCompute0 = cutlass::epilogue::threadblock::Sm80EVT<
    Compute0,
    Accum,
    Bias>;
  
using Compute1 = cutlass::epilogue::threadblock::VisitorCompute<
    cutlass::plus, ElementCompute, ElementCompute,
    cutlass::FloatRoundStyle::round_to_nearest
>;

using EVTCompute1 = cutlass::epilogue::threadblock::Sm80EVT<
    Compute1,
    EVTCompute0,
    C1>;

using Compute2 = cutlass::epilogue::threadblock::VisitorCompute<
    cutlass::plus, ElementOutput, ElementCompute,
    cutlass::FloatRoundStyle::round_to_nearest
>;

using EVTCompute2 = cutlass::epilogue::threadblock::Sm80EVT<
    Compute2,
    EVTCompute1,
    C2>;

using D = cutlass::epilogue::threadblock::VisitorAuxStore<
    OutputTileThreadMap, ElementOutput, cutlass::FloatRoundStyle::round_to_nearest,
    cute::Stride<int64_t, _1, int64_t> // StrideMNL
>;

using EVTD = cutlass::epilogue::threadblock::Sm80EVT<
    D,
    EVTCompute2>;

using EVTKernelStreamK =
    typename cutlass::gemm::kernel::DefaultGemmWithVisitor<
    ElementA, LayoutA, cutlass::ComplexTransform::kNone, AlignmentA,
    ElementB, LayoutB, cutlass::ComplexTransform::kNone, AlignmentB,
    ElementC, LayoutC, AlignmentC,
    ElementAccumulator,
    ElementCompute,
    cutlass::arch::OpClassTensorOp,
    cutlass::arch::Sm80,
    ThreadblockShape,
    WarpShape,
    InstructionShape,
    EVTD,
    cutlass::gemm::threadblock::ThreadblockSwizzleStreamK,
    NumStages,
    cutlass::arch::OpMultiplyAdd,
    EVTEpilogueStages
>::GemmKernel;

using DeviceGemmStreamK = cutlass::gemm::device::GemmUniversalAdapter<EVTKernelStreamK>;
```

**EN**: This section defines the epilogue visitor tree (EVT) used by the Stream-K kernel. `Bias` is a `VisitorRowBroadcast` with stride `<_0, _1, batch>`, which means the M stride is zero and the same row vector is broadcast across every output row. `C1` and `C2` are ordinary auxiliary loads, the `VisitorCompute` nodes connect them with addition, and `D` stores the final fused result. The final `EVTKernelStreamK` swaps the classic threadblock swizzle for `ThreadblockSwizzleStreamK`, which is the scheduling hook that enables work-centric Stream-K behavior.

**CN**: 这一段定义了 Stream-K 内核使用的 epilogue visitor tree（EVT）。`Bias` 是一个 `VisitorRowBroadcast`，其步长写成 `<_0, _1, batch>`，含义是 M 维步长为 0，因此同一行向量会广播到所有输出行。`C1` 与 `C2` 是普通辅助张量加载节点，多个 `VisitorCompute` 节点用加法把它们串起来，`D` 负责写出最终融合结果。最后的 `EVTKernelStreamK` 用 `ThreadblockSwizzleStreamK` 替换经典 swizzle，这正是启用 work-centric Stream-K 调度的关键。

### Lines 261-381 — Result container and command-line driven test configuration

```cpp

/////////////////////////////////////////////////////////////////////////////////////////////////
/// Testbed utility types
/////////////////////////////////////////////////////////////////////////////////////////////////

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
    avg_runtime_ms(avg_runtime_ms), gflops(gflops), status(status), error(error), passed(true)
  {}

};


/// Command line options parsing
struct Options
{
  std::string               command_name;
  bool                      help;
  cutlass::gemm::GemmCoord  problem_size;
  float                     alpha;
  float                     beta;
  int                       split_k_factor;
  int                       avail_sms;
  int                       iterations;
  bool                      real;

  cutlass::HostTensor<ElementA, LayoutA> tensor_a;
  cutlass::HostTensor<ElementB, LayoutB> tensor_b;
  cutlass::HostTensor<ElementC, LayoutC> tensor_c1;
  cutlass::HostTensor<ElementC, LayoutC> tensor_c2;
  cutlass::HostTensor<ElementC, LayoutC> tensor_d;
  cutlass::HostTensor<ElementC, LayoutC> tensor_ref_d;
  cutlass::HostTensor<ElementC, LayoutC> tensor_Vector;
  // cutlass::HostTensor<ElementC, LayoutC> tensor_Tensor;

  Options(std::string command_name) :
    command_name(command_name),
    help(false),
    problem_size({2048, 2048, 2048}),
    alpha(1.0f),
    beta(1.0f),
    split_k_factor(1),
    avail_sms(-1),              // Number of device SMs to use is unlimited
    real(false),
    iterations(10000)
  {}

  bool valid() const
  {
    return true;
  }

  void parse(int argc, char const **args)
  {
    cutlass::CommandLine cmd(argc, args);

    if (cmd.check_cmd_line_flag("help")) {
      help = true;
    }

    cmd.get_cmd_line_argument("m", problem_size.m());
    cmd.get_cmd_line_argument("n", problem_size.n());
    cmd.get_cmd_line_argument("k", problem_size.k());
    cmd.get_cmd_line_argument("alpha", alpha);
    cmd.get_cmd_line_argument("beta", beta);
    cmd.get_cmd_line_argument("split", split_k_factor);
    cmd.get_cmd_line_argument("iterations", iterations);
    real = cmd.check_cmd_line_flag("real");
  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const
  {
    out
      << "Performs a GEMM computation.\n"
      << "\n"
      << "Options:\n"
      << "\n"
      << "  --help                      If specified, displays this usage statement.\n\n"
      << "  --m=<int>                   GEMM M dimension\n"
      << "  --n=<int>                   GEMM N dimension\n"
      << "  --k=<int>                   GEMM K dimension\n"
      << "  --alpha=<f32>               Epilogue scalar alpha\n"
      << "  --beta=<f32>                Epilogue scalar beta\n\n"
      << "  --split=<int>               Split-K factor to emulate\n\n"
      << "  --real                      If specified, initializes with real values instead of whole numbers. Errors are to be expected.\n\n"
      << "  --iterations=<int>          Number of profiling iterations to perform.\n\n";

    out
      << "\n\nExamples:\n\n"
      << "$ " << command_name << " --m=1024 --n=512 --k=1024 --alpha=2 --beta=0.707 \n\n";

    return out;
  }

  /// Compute performance in GFLOP/s
  double gflops(double runtime_s) const
  {
    // Two flops per multiply-add
    return 2.0 * double(problem_size.product()) / double(1.0e9) / runtime_s;
  }
};


/////////////////////////////////////////////////////////////////////////////////////////////////
/// GEMM evaluation
/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: The `Result` struct stores timing, status, and pass/fail state for each experiment. `Options` then owns the GEMM shape, alpha/beta, split factor, optional real-valued initialization flag, and all host tensors used in the experiment. Because Stream-K is evaluated under different load-balancing widths, `avail_sms` is exposed as an option-controlled knob in the runtime state even though it is only consumed by the Stream-K path later.

**CN**: `Result` 结构体用于保存每次实验的时间、状态和通过/失败信息。随后 `Options` 持有 GEMM 规模、alpha/beta、split 因子、是否使用实数初始化的标志，以及实验中全部主机张量。由于 Stream-K 需要在不同“负载均衡宽度”下测试，`avail_sms` 也被保存在运行时状态中，尽管它稍后只会被 Stream-K 路径使用。

### Lines 382-474 — Argument builders for baseline broadcast GEMM and Stream-K EVT GEMM

```cpp
/// Populates a DeviceGemmBasic::Arguments structure from the given commandline options
typename DeviceGemmBasic::Arguments args_from_options(
    const DeviceGemmBasic &device_gemm,
    const Options &options,
    cutlass::HostTensor<ElementA, LayoutA> &tensor_a,
    cutlass::HostTensor<ElementB, LayoutB> &tensor_b,
    cutlass::HostTensor<ElementC, LayoutC> &tensor_c1,
    cutlass::HostTensor<ElementC, LayoutC> &tensor_c2,
    cutlass::HostTensor<ElementC, LayoutC> &tensor_d,
    cutlass::HostTensor<ElementC, LayoutC> &tensor_Vector /*,
    cutlass::HostTensor<ElementC, LayoutC> &tensor_Tensor */
    )
{
  return typename DeviceGemmBasic::Arguments(
    cutlass::gemm::GemmUniversalMode::kGemm,  // universal mode
    options.problem_size,                     // problem_size
    options.split_k_factor,                   // batch count / splitk slices
    {                                         // epilogue parameters
      ElementAccumulator(options.alpha),
      ElementAccumulator(options.beta)
    },
    tensor_a.device_data(),                   // ptr_A
    tensor_b.device_data(),                   // ptr_B
    tensor_c1.device_data(),                  // ptr_C1
    tensor_c2.device_data(),                  // ptr_C2
    tensor_d.device_data(),                   // ptr_D
    tensor_Vector.device_data(),              // ptr_Vector
    /* tensor_Tensor.device_data(), */nullptr,// ptr_Tensor
    options.problem_size.mk().product(),      // batch_stride_A
    options.problem_size.nk().product(),      // batch_stride_B
    options.problem_size.mn().product(),      // batch_stride_C1
    options.problem_size.mn().product(),      // batch_stride_C2
    options.problem_size.mn().product(),      // batch_stride_D
    options.problem_size.mn().product(),      // batch_stride_Vector
    options.problem_size.mn().product(),      // batch_stride_Tensor
    tensor_a.layout().stride(0),              // stride_a
    tensor_b.layout().stride(0),              // stride_b
    tensor_c1.layout().stride(0),             // stride_c1
    tensor_c2.layout().stride(0),             // stride_c2
    tensor_d.layout().stride(0),              // stride_d
    /*tensor_Vector.layout().stride(0)*/0,    // stride_Vector
    /*tensor_Tensor.layout().stride(0)*/0);   // stride_Tensor
}

/// Populates a DeviceGemmStreamK::Arguments structure from the given commandline options
typename DeviceGemmStreamK::Arguments args_from_options(
    const DeviceGemmStreamK &device_gemm,
    const Options &options,
    cutlass::HostTensor<ElementA, LayoutA> &tensor_a,
    cutlass::HostTensor<ElementB, LayoutB> &tensor_b,
    cutlass::HostTensor<ElementC, LayoutC> &tensor_c1,
    cutlass::HostTensor<ElementC, LayoutC> &tensor_c2,
    cutlass::HostTensor<ElementC, LayoutC> &tensor_d,
    cutlass::HostTensor<ElementC, LayoutC> &tensor_Vector/*,
    cutlass::HostTensor<ElementC, LayoutC> &tensor_Tensor*/
    )
{ 
  typename EVTD::Arguments callback_args{
    {
      {
        {
          {},                                                                                                          // Accum
          {tensor_Vector.device_data(), ElementC(0), {_0{}, _1{}, int32_t(options.problem_size.n())}},                 // Bias
          {}                                                                                                           // Compute0
        },                                                                                                             // EVTCompute0
        {tensor_c1.device_data(), ElementC(0), {options.problem_size.n(), _1{}, options.problem_size.mn().product()}}, // C1
        {}                                                                                                             // Compute1
      },                                                                                                               // EVTCompute1
      {tensor_c2.device_data(), ElementC(0), {options.problem_size.n(), _1{}, options.problem_size.mn().product()}},   // C2
      {}                                                                                                               // Compute2
    },                                                                                                                 // EVTCompute2
    {tensor_d.device_data(), {options.problem_size.n(), _1{}, options.problem_size.mn().product()}},                   // D
  };                                                                                                                   // EVTD

  return typename DeviceGemmStreamK::Arguments(
    cutlass::gemm::GemmUniversalMode::kGemm,  // universal mode
    options.problem_size,                     // problem_size
    options.split_k_factor,                   // batch count / splitk slices
    callback_args,                            // argument of EVT callbacks
    tensor_a.device_data(),                   // ptr_A
    tensor_b.device_data(),                   // ptr_B
    nullptr,                                  // ptr_C (unused)
    nullptr,                                  // ptr_D (unused)
    options.problem_size.mk().product(),      // batch_stride_A
    options.problem_size.nk().product(),      // batch_stride_B
    0,                                        // batch_stride_C (unused)
    0,                                        // batch_stride_D (unused)
    tensor_a.layout().stride(0),              // stride_a
    tensor_b.layout().stride(0),              // stride_b
    0,                                        // stride_c (unused)
    0,                                        // stride_d (unused)
    options.avail_sms);                       // avail_sms
}
```

**EN**: The overloaded `args_from_options()` helpers are the clearest expression of the two execution models. The baseline path passes direct pointers to A, B, C1, C2, D, and the broadcast vector into `GemmUniversalWithBroadcast`, with zero stride for the vector so it behaves as a broadcast. The Stream-K path instead builds a nested `EVTD::Arguments` tree that describes how accumulators, the row-broadcast bias, and the two residual tensors should be loaded and combined. This is also where `avail_sms` enters the picture, allowing Stream-K to load-balance over all SMs or intentionally emulate data parallelism by constraining the balancing width.

**CN**: 两个重载的 `args_from_options()` 是理解这两种执行模型最直接的入口。基线路径把 A、B、C1、C2、D 和广播向量的指针直接传给 `GemmUniversalWithBroadcast`，并通过向量步长为 0 的方式实现广播。Stream-K 路径则显式构造嵌套的 `EVTD::Arguments` 树，描述累加器、按行广播 bias 与两个 residual 张量该如何加载和组合。`avail_sms` 也在这里进入参数体系：它既可以让 Stream-K 跨全部 SM 做负载均衡，也可以故意把均衡宽度限制到 1，从而模拟普通 data-parallel 行为。

### Lines 475-550 — Generic execution helper for correctness and timing

```cpp

/// Execute a given example GEMM computation
template <typename DeviceGemmT>
Result run(std::string description, Options &options)
{
  // Display test description
  std::cout << std::endl << description << std::endl;

  // Zero-initialize test output matrix D
  cutlass::reference::host::TensorFill(options.tensor_d.host_view());
  options.tensor_d.sync_device();

  // Instantiate CUTLASS kernel depending on templates
  DeviceGemmT device_gemm;

  // Create a structure of gemm kernel arguments suitable for invoking an instance of DeviceGemmT
  auto arguments = args_from_options(device_gemm, options, 
      options.tensor_a, options.tensor_b, options.tensor_c1, options.tensor_c2, options.tensor_d, 
      options.tensor_Vector/*, options.tensor_Tensor*/);

  // Using the arguments, query for extra workspace required for matrix multiplication computation
  size_t workspace_size = DeviceGemmT::get_workspace_size(arguments);

  // Allocate workspace memory
  cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);

  // Check the problem size is supported or not
  CUTLASS_CHECK(device_gemm.can_implement(arguments));

  // Initialize CUTLASS kernel with arguments and workspace pointer
  CUTLASS_CHECK(device_gemm.initialize(arguments, workspace.get()));

  // Correctness / Warmup iteration
  CUTLASS_CHECK(device_gemm());

  // Copy output data from CUTLASS and reference kernel to host for comparison
  options.tensor_d.sync_host();

  // Check if output from CUTLASS kernel and reference kernel are equal or not
  Result result;
  result.passed = cutlass::reference::host::TensorEquals(
    options.tensor_d.host_view(),
    options.tensor_ref_d.host_view());

  double err = cutlass::reference::host::TensorRelativeErrorMetric(
    options.tensor_d.host_view(),
    options.tensor_ref_d.host_view());

  std::cout << "  Disposition: " << (result.passed ? "Passed" : "Failed") << " \t Relative error: " << err << std::endl;

  // Run profiling loop
  if (options.iterations > 0)
  {
    GpuTimer timer;
    timer.start();
    for (int iter = 0; iter < options.iterations; ++iter) {
      CUTLASS_CHECK(device_gemm());
    }
    timer.stop();

    // Compute average runtime and GFLOPs.
    float elapsed_ms = timer.elapsed_millis();
    result.avg_runtime_ms = double(elapsed_ms) / double(options.iterations);
    result.gflops = options.gflops(result.avg_runtime_ms / 1000.0);

    std::cout << "  Avg runtime: " << result.avg_runtime_ms << " ms" << std::endl;
    std::cout << "  GFLOPs: " << result.gflops << std::endl;
  }

  // TODO: uncomment when results match
  //if (!result.passed) {
  //  exit(-1);
  //}

  return result;
}
```

**EN**: The templated `run()` helper works for both device GEMM types. It clears the output tensor, creates the kernel arguments, allocates any workspace, checks `can_implement()`, runs one correctness iteration, synchronizes the result back to host memory, and compares it with the prepared reference tensor. If iterations are enabled, it then benchmarks repeated launches with `GpuTimer`. This makes the experiment logic identical for the baseline and Stream-K kernels, so any performance difference is primarily about the decomposition strategy.

**CN**: 模板化的 `run()` 同时适用于两类 device GEMM。它会先清空输出张量、构造参数、分配 workspace、检查 `can_implement()`，再执行一次正确性运行，把结果同步回主机并与预先准备好的参考张量比较。如果开启计时，它随后还会用 `GpuTimer` 对重复执行进行基准测试。这样一来，基线版与 Stream-K 版在实验流程上完全一致，性能差异就主要来自分解与调度策略本身。

### Lines 551-738 — Program flow: initialization, reference computation, and Stream-K comparisons

```cpp


/// Program entrypoint
int main(int argc, const char **argv)
{
  // CUTLASS must be compiled with CUDA 11.0 Toolkit to run these examples.
  if (!(__CUDACC_VER_MAJOR__ >= 11)) {
    std::cerr << "Ampere Tensor Core operations must be compiled with CUDA 11.0 Toolkit or later." << std::endl;

    // Returning zero so this test passes on older Toolkits. Its actions are no-op.
    return 0;
  }

  // Current device must must have compute capability at least 80
  cudaDeviceProp props;
  int current_device_id;
  CUDA_CHECK(cudaGetDevice(&current_device_id));
  CUDA_CHECK(cudaGetDeviceProperties(&props, current_device_id));
  if (!((props.major * 10 + props.minor) >= 80))
  {
    std::cerr << "Ampere Tensor Core operations must be run on a machine with compute capability at least 80."
              << std::endl;

    // Returning zero so this test passes on older Toolkits. Its actions are no-op.
    return 0;
  }

  // Parse commandline options
  Options options("ampere_streamk_broadcast_gemm");
  options.parse(argc, argv);

  if (options.help) {
    options.print_usage(std::cout) << std::endl;
    return 0;
  }

  std::cout <<
    options.iterations << " timing iterations of " <<
    options.problem_size.m() << " x " <<
    options.problem_size.n() << " x " <<
    options.problem_size.k() << " matrix-matrix multiply" << std::endl;

  if (!options.valid()) {
    std::cerr << "Invalid problem." << std::endl;
    return -1;
  }


  //
  // Initialize GEMM datasets
  //

  // Initialize tensors using CUTLASS helper functions
  options.tensor_a.resize(options.problem_size.mk());           // <- Create matrix A with dimensions M x K
  options.tensor_b.resize(options.problem_size.kn());           // <- Create matrix B with dimensions K x N
  options.tensor_c1.resize(options.problem_size.mn());          // <- Create matrix C1 with dimensions M x N
  options.tensor_c2.resize(options.problem_size.mn());          // <- Create matrix C2 with dimensions M x N
  options.tensor_d.resize(options.problem_size.mn());           // <- Create matrix D with dimensions M x N used to store output from CUTLASS kernel
  options.tensor_ref_d.resize(options.problem_size.mn());       // <- Create matrix D with dimensions M x N used to store output from reference kernel
  options.tensor_Vector.resize({1, options.problem_size.n()});  // <- Create broadcast vector with dimensions N x 1
  // options.tensor_Tensor.resize(options.problem_size.mn());   // <- Create T matrix with dimensions M x N

  int _init_bits = options.real ? -1 : 0;

  // Fill matrix A on host with uniform-random data [-2, 2]
  cutlass::reference::host::TensorFillRandomUniform(
      options.tensor_a.host_view(),
      1,
      ElementA(2),
      ElementA(-2), _init_bits);

  // Fill matrix B on host with uniform-random data [-2, 2]
  cutlass::reference::host::TensorFillRandomUniform(
      options.tensor_b.host_view(),
      1,
      ElementB(2),
      ElementB(-2), _init_bits);

  // Fill matrix C1 on host with uniform-random data [-2, 2]
  cutlass::reference::host::TensorFillRandomUniform(
      options.tensor_c1.host_view(),
      1,
      ElementC(2),
      ElementC(-2), _init_bits);

  // Fill matrix C2 on host with uniform-random data [-2, 2]
  cutlass::reference::host::TensorFillRandomUniform(
      options.tensor_c2.host_view(),
      1,
      ElementC(2),
      ElementC(-2), _init_bits);

  cutlass::reference::host::TensorFillRandomUniform(
      options.tensor_Vector.host_view(),
      1,
      ElementC(2),
      ElementC(-2), _init_bits);

  //
  // Compute reference output
  //

  // Copy data from host to GPU
  options.tensor_a.sync_device();
  options.tensor_b.sync_device();
  options.tensor_c1.sync_device();
  options.tensor_c2.sync_device();
  options.tensor_Vector.sync_device();
  // options.tensor_Tensor.sync_device();

  // Zero-initialize reference output matrix D
  cutlass::reference::host::TensorFill(options.tensor_ref_d.host_view());
  options.tensor_ref_d.sync_device();

  // Create instantiation for device reference gemm kernel
  DeviceGemmReference gemm_reference;

  // Launch device reference gemm kernel
  gemm_reference(
    options.problem_size,
    ElementAccumulator(options.alpha),
    options.tensor_a.device_ref(),
    options.tensor_b.device_ref(),
    ElementAccumulator(options.beta),
    options.tensor_c1.device_ref(),
    options.tensor_ref_d.device_ref());

  // Wait for kernels to finish
  CUDA_CHECK(cudaDeviceSynchronize());

  // Copy output data from reference kernel to host for comparison
  options.tensor_ref_d.sync_host();

  // Add broadcast vector (without multiplier)
  // This is only possible because BinaryOp is addition, and UnaryOps are identity.
  // This makes the addition of broadcast vector commutable.
  /// identity(plus(identity(alpha * (a * b) + v), beta * c)) ==
  /// alpha * a * b + v + beta * c                            ==
  /// (alpha * a * b + beta * c) + v                          ==
  /// GEMM(a, b, c) + v
  // Vector broadcast on host
  for (int i=0; i < options.problem_size.m(); ++i) {
    for (int j=0; j < options.problem_size.n(); ++j) {
      options.tensor_ref_d.host_view().ref().at({i, j}) += options.tensor_Vector.host_view().ref().at({0, j});
      options.tensor_ref_d.host_view().ref().at({i, j}) += options.tensor_c2.host_view().ref().at({i, j});
    }
  }

  // Sync back with device just in case
  options.tensor_ref_d.sync_device();

  //
  // Evaluate CUTLASS kernels
  //

  // Test default operation
  if (options.split_k_factor == 1)
  {
    // Compare basic data-parallel version versus StreamK version using default load-balancing heuristics
    Result basic_dp         = run<DeviceGemmBasic>("Basic data-parallel GEMM", options);
    Result streamk_default  = run<DeviceGemmStreamK>("StreamK GEMM with default load-balancing", options);

    printf("  Speedup vs Basic-DP: %.3f\n", (basic_dp.avg_runtime_ms / streamk_default.avg_runtime_ms));

    // Show that StreamK can emulate basic data-parallel GEMM when we set the number of SMs to load-balance across = 1
    options.avail_sms       = 1;        // Set loadbalancing width to 1 SM (no load balancing)
    Result streamk_dp       = run<DeviceGemmStreamK>("StreamK emulating basic data-parallel GEMM", options);
    options.avail_sms       = -1;       // Reset loadbalancing width to unspecified SMs (i.e., the number of device SMs)

    printf("  Speedup vs Basic-DP: %.3f\n", (basic_dp.avg_runtime_ms / streamk_dp.avg_runtime_ms));

    options.split_k_factor++;     // Increment splitting factor for next evaluation

  }

  // Show that StreamK can emulate "Split-K" with a tile-splitting factor
  Result basic_splitk = run<DeviceGemmBasic>(
    std::string("Basic split-K GEMM with tile-splitting factor ") + std::to_string(options.split_k_factor),
    options);

  Result streamk_splitk = run<DeviceGemmStreamK>(
    std::string("StreamK emulating Split-K GEMM with tile-splitting factor ") + std::to_string(options.split_k_factor),
    options);

  printf("  Speedup vs Basic-SplitK: %.3f\n", (basic_splitk.avg_runtime_ms / streamk_splitk.avg_runtime_ms));

  return 0;
}
```

**EN**: The `main()` routine validates CUDA and SM80 capability, parses options, allocates tensors, fills them with random data, and computes a reference GEMM on device. Because the fused operation is additive and uses identity activations, the program can add the row-broadcast vector and `C2` on the host after the reference GEMM to form the expected final answer. The final experiment sequence is the real teaching point: first compare classic data parallel against Stream-K with default load balancing, then force `avail_sms = 1` to show Stream-K can emulate data parallel scheduling, and finally compare both paths again under a split-K factor greater than one. This clearly demonstrates Stream-K’s broadcast behavior and its work-centric balancing model.

**CN**: `main()` 先检查 CUDA 版本与 SM80 能力，再解析参数、分配张量、填充随机数据，并在设备端执行一次参考 GEMM。由于该融合操作只包含加法和 identity 激活，所以程序可以在主机端把按行广播向量和 `C2` 继续加到参考结果上，得到最终期望输出。真正有教学意义的是最后的实验顺序：先比较经典 data parallel 与默认负载均衡的 Stream-K，再把 `avail_sms = 1` 设成只在单个 SM 宽度上均衡，以说明 Stream-K 可以模拟普通 data-parallel 调度；最后再在 split-K 因子大于 1 的条件下比较两条路径。这样就把 Stream-K 的广播行为和 work-centric 负载均衡模型展示得很清楚。

---

## Key Concepts / 关键概念

- **Stream-K scheduling**
  - **EN**: Stream-K decomposes work by progress in K and tile work units instead of assigning one fixed CTA per output tile. This reduces tail inefficiency and improves load balance on irregular workloads.
  - **CN**: Stream-K 不是简单地给每个输出 tile 固定分配一个 CTA，而是按 K 维进度和工作单元来分解任务，从而减少尾部空转并改善负载均衡。
- **Broadcast vector encoding**
  - **EN**: The row-broadcast visitor uses an M stride of zero, so a single length-N vector is reused for every output row.
  - **CN**: 按行广播 visitor 通过把 M 维步长设为 0，使长度为 N 的同一向量能够复用于所有输出行。
- **Residual block fusion**
  - **EN**: The fused formula is effectively `alpha * A@B + beta * C1 + Vector + C2` because all activation/unary nodes are identity in this sample.
  - **CN**: 由于本示例中的激活与一元节点都是 identity，融合公式本质上就是 `alpha * A@B + beta * C1 + Vector + C2`。
- **Emulation via `avail_sms`**
  - **EN**: Setting `avail_sms` to 1 narrows the Stream-K balancing window so the kernel behaves much more like classic data parallel scheduling.
  - **CN**: 把 `avail_sms` 设为 1 会显著缩小 Stream-K 的负载均衡窗口，因此其行为会更接近经典 data-parallel 调度。

## Dependencies / 依赖项

- `cutlass/gemm/device/gemm_universal_with_broadcast.h`
  - **EN**: Provides the baseline GEMM path with built-in support for auxiliary tensors and broadcast inputs.
  - **CN**: 提供带辅助张量与广播输入支持的基线 GEMM 路径。
- `cutlass/gemm/device/gemm_universal_streamk_with_broadcast.h`
  - **EN**: Supplies the Stream-K variant used for work-centric load balancing on Ampere.
  - **CN**: 提供在 Ampere 上进行 work-centric 负载均衡的 Stream-K 变体。
- EVT visitor headers
  - **EN**: The visitor-tree types from CUTLASS epilogue fusion define how accumulators, broadcasts, residual loads, and stores are composed.
  - **CN**: CUTLASS epilogue fusion 的 visitor-tree 类型定义了累加器、广播、残差加载与写回的组合方式。
- Reference utilities and `helper.h`
  - **EN**: Reference GEMM, tensor comparison helpers, and `GpuTimer` are used to validate and benchmark both execution strategies.
  - **CN**: 参考 GEMM、张量比较工具以及 `helper.h` 中的 `GpuTimer` 用来验证并测量两种执行策略。
