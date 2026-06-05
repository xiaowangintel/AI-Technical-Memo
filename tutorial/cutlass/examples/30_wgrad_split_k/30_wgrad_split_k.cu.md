# examples/30_wgrad_split_k/30_wgrad_split_k.cu — Code Analysis / 代码分析
**Source / 源文件**: `examples/30_wgrad_split_k/30_wgrad_split_k.cu`
**Purpose / 用途**: Demonstrates CUTLASS conv2d weight-gradient on SM80 using parallel split-K implicit GEMM plus a separate reduction stage. / 演示在 SM80 上使用 CUTLASS 通过并行 split-K 隐式 GEMM 加独立归约阶段来计算 conv2d 权重梯度。
---
## Line-by-Line Analysis / 逐行分析
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

/*
This example shows how to compute conv2d gradient with respect to weight (wgrad). In wgrad, the K dimension of
impligit GEMM, corresponding to the sequential reduction loop, is very large (N * P * Q). Split-k with parallel
reduction is highly effective for such cases. Given split_k_slices parameter, it partitions the K loop into
split_k_slices chunks and computes partial reductions in parallel across different blocks. After that,
a parallel reduction kernel is launched to accumulate partial reductions.
In practice, wgrad requires fp32 accumulation to avoid overflow. When the input is fp16, some care is needed
to correctly instantiate the GEMM template.
*/
```
**EN**: The file starts with the standard CUTLASS license and a high-level description of the sample. The important point is the algorithmic motivation: conv2d wgrad turns into an implicit GEMM whose reduction dimension is `N * P * Q`, so the example uses parallel split-K and a follow-up reduction kernel to keep large reductions efficient. The comment also explains the numeric choice of FP16 inputs with FP32 accumulation.
**CN**: 文件先给出标准 CUTLASS 许可证和示例总览。关键背景是算法动机：conv2d 的 wgrad 会映射成一个隐式 GEMM，其归约维度是 `N * P * Q`，因此示例采用并行 split-K，并在之后追加一个归约内核，以高效处理超大的归约维度。注释还说明了数值类型选择：输入用 FP16，但累加使用 FP32。

```cpp

#include <iostream>
#include <fstream>
#include <sstream>

#include "cutlass/cutlass.h"
#include "cutlass/gemm/device/gemm.h"
#include "cutlass/conv/kernel/default_conv2d_wgrad.h"
#include "cutlass/conv/device/implicit_gemm_convolution.h"

#include "cutlass/util/command_line.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/reference/device/gemm.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/device/convolution.h"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/reduction/device/reduce_split_k.h"
#include "cutlass/reduction/thread/reduction_operators.h"

#include "helper.h"
```
**EN**: This include block brings in standard I/O utilities, CUTLASS core headers, the default conv2d wgrad kernel generator, the device wrapper for implicit GEMM convolution, host tensor helpers, reference implementations, and split-K reduction utilities. Together they show that this example is not a hand-written kernel; it is a composition of reusable CUTLASS convolution and reduction building blocks.
**CN**: 这组头文件引入标准输入输出工具、CUTLASS 核心头文件、默认 conv2d wgrad 内核生成器、隐式 GEMM 卷积的设备端封装、HostTensor 辅助工具、参考实现，以及 split-K 归约工具。这说明示例并不是手写单一内核，而是由 CUTLASS 的卷积与归约组件组合而成。

```cpp

// The code section below describes datatype for input, output tensors and computation between
// elements
// In Wgrad, fp32 accumulation is necessary in practice.
using ElementAccumulator = float;                  // Data type of accumulator
using ElementComputeEpilogue = float;              // Data type of epilogue computation (alpha, beta)
using ElementInputA = cutlass::half_t;             // Data type of elements in input tensor
using ElementInputB = cutlass::half_t;             // Data type of elements in input tensor
using ElementOutput = cutlass::half_t;                       // Data type of elements in output tensor
using ElementC = ElementOutput;
using ElementCompute = ElementComputeEpilogue;
using LayoutInputA = cutlass::layout::TensorNHWC;
using LayoutInputB = cutlass::layout::TensorNHWC;
using LayoutOutput = cutlass::layout::TensorNHWC;

// This code section describes whether you want to use tensor cores or regular SIMT cores on GPU SM
using MMAOp = cutlass::arch::OpClassTensorOp;

// This code section describes CUDA SM architecture number
using SmArch = cutlass::arch::Sm80;
```
**EN**: These aliases define the numeric contract of the kernel. Inputs and final outputs are `cutlass::half_t`, but both the accumulator and epilogue compute type are `float`, which is the usual CUTLASS pattern for stable wgrad accumulation. All tensors use NHWC layout, and the computation is targeted at Tensor Core (`OpClassTensorOp`) instructions on Ampere (`Sm80`).
**CN**: 这些类型别名定义了内核的数值约定。输入和最终输出都是 `cutlass::half_t`，但累加器与尾处理计算类型都使用 `float`，这是 CUTLASS 中实现稳定 wgrad 累加的常见模式。所有张量都采用 NHWC 布局，计算目标是 Ampere (`Sm80`) 上的 Tensor Core（`OpClassTensorOp`）指令。

```cpp

// This code section describes the tile size a thread block will compute
using ThreadblockShape = cutlass::gemm::GemmShape<128, 128, 32>; // Threadblock tile shape

// This code section describes tile size a warp will compute
using WarpShape = cutlass::gemm::GemmShape<64, 64, 32>;          // Warp tile shape

// This code section describes the size of MMA op
using InstructionShape = cutlass::gemm::GemmShape<16, 8, 16>;    // TensorCore instruction shape

// This code section describes how threadblocks are scheduled on GPU
using SwizzleThreadBlock = cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<>;

// Number of pipelines you want to use
constexpr int NumStages = 3;

// This code section describe iterator algorithm selected is Analytic or Optimized
static cutlass::conv::IteratorAlgorithm const IteratorAlgorithm = cutlass::conv::IteratorAlgorithm::kOptimized;
```
**EN**: This section fixes the execution tiling and iterator strategy. `ThreadblockShape<128,128,32>` defines the CTA tile in the implicit GEMM space, `WarpShape<64,64,32>` partitions that tile across warps, and `InstructionShape<16,8,16>` matches the hardware MMA instruction. `GemmIdentityThreadblockSwizzle<>` keeps a straightforward CTA mapping, `NumStages = 3` enables pipelined mainloop staging, and `IteratorAlgorithm::kOptimized` selects the optimized convolution iterators instead of the simpler analytic path.
**CN**: 这一段确定执行分块和迭代器策略。`ThreadblockShape<128,128,32>` 定义隐式 GEMM 空间中的 CTA 分块，`WarpShape<64,64,32>` 表示 warp 级切分，`InstructionShape<16,8,16>` 对应硬件 MMA 指令形状。`GemmIdentityThreadblockSwizzle<>` 使用直接的 CTA 映射，`NumStages = 3` 启用主循环流水线阶段，而 `IteratorAlgorithm::kOptimized` 选择优化版卷积迭代器，而不是更直观但更慢的 analytic 路径。

```cpp

// We need two epilogue functors - one for GEMM and another for the final reduction.
// The epilogue for GEMM is not used, but needed to instantiate the CUTLASS kernel template.
// Note that, when the input is fp16 and accumulation is fp32, the output of GEMM needs to be fp32,
// the final reduction is done in fp32, and the reduction epilogue converts fp32 outputs to fp16.
// Therefore, the output type of the GEMM epilogue is ElementCompute, not ElementOutput.

// This code section describes the epilogue part of the kernel, we use default value
using EpilogueOpGEMM = cutlass::epilogue::thread::LinearCombination<
    ElementCompute,                                     // Data type of output matrix.
    128 / cutlass::sizeof_bits<ElementCompute>::value,  // The number of elements per vectorized.
    // memory access. This becomes the vector width of
    // math instructions in the epilogue too.
    ElementAccumulator,                                // Data type of accumulator
    ElementComputeEpilogue>;                           // Data type for alpha/beta in linear combination

// The epilogue functor for reduction. This is the one that is actually used.
using EpilogueOpReduction = cutlass::epilogue::thread::LinearCombination<
    ElementOutput,                                     // Data type of output matrix.
    128 / cutlass::sizeof_bits<ElementOutput>::value,  // The number of elements per vectorized.
    // memory access. This becomes the vector width of
    // math instructions in the epilogue too.
    ElementAccumulator,                                // Data type of accumulator
    ElementComputeEpilogue>;                           // Data type for alpha/beta in lin
```
**EN**: The sample defines two epilogues because the computation is split into two phases. `EpilogueOpGEMM` writes FP32 partial sums from the implicit GEMM into workspace, so its output type is `ElementCompute` rather than the final FP16 type. `EpilogueOpReduction` is the real finalization step: it reduces FP32 partial tiles across split-K slices, applies `alpha`/`beta`, and converts the result back to `ElementOutput` (`half`). This separation is the key CUTLASS-specific API choice that makes split-K wgrad numerically safe.
**CN**: 由于计算被拆成两个阶段，这里定义了两个 epilogue。`EpilogueOpGEMM` 将隐式 GEMM 的 FP32 部分和写入工作区，因此它的输出类型是 `ElementCompute`，而不是最终的 FP16 类型。`EpilogueOpReduction` 才是真正的结果收尾步骤：它在 split-K 切片之间归约 FP32 部分结果，应用 `alpha`/`beta`，并把结果转换回 `ElementOutput`（`half`）。这种分离正是该示例中最关键的 CUTLASS API 设计点，可在 split-K wgrad 中保持数值稳定。

```cpp
using Conv2dWgradKernel = typename cutlass::conv::kernel::DefaultConv2dWgrad<
    ElementInputA, LayoutInputA,
    ElementInputB, LayoutInputB,
    ElementAccumulator, LayoutOutput,
    ElementAccumulator,
    MMAOp,
    SmArch,
    ThreadblockShape,
    WarpShape,
    InstructionShape,
    EpilogueOpGEMM,
    SwizzleThreadBlock,
    NumStages,
    cutlass::arch::OpMultiplyAdd,
    IteratorAlgorithm
    >::Kernel;

using ImplicitGemm = cutlass::conv::device::ImplicitGemmConvolution<Conv2dWgradKernel>;

using EpilogueOutputOp = EpilogueOpReduction;

/// Reduction kernel
using ReductionOp = cutlass::reduction::thread::ReduceAdd<
    ElementAccumulator,
    typename EpilogueOutputOp::ElementAccumulator,
    EpilogueOutputOp::kCount
   >;

using ReductionKernel = cutlass::reduction::kernel::ReduceSplitK<
    cutlass::MatrixShape<4, 32 * EpilogueOutputOp::kCount>,
    EpilogueOutputOp,
    ReductionOp
   >;

using ReductionDevice = cutlass::reduction::device::ReduceSplitK<ReductionKernel>;
using ReductionStrideIndex = typename ReductionDevice::StrideIndex;
```
**EN**: Here the actual kernel types are composed. `DefaultConv2dWgrad` expands all the preceding type choices into a concrete kernel type, `ImplicitGemmConvolution` wraps it in a device API with `Arguments`, `initialize`, and call operators, and the reduction aliases build a `ReduceSplitK` kernel for summing the workspace. `ReductionOp` uses `ReduceAdd`, so the full algorithm is: run many partial implicit GEMMs, then sum them into the final filter gradient tensor.
**CN**: 这里开始组合真正的内核类型。`DefaultConv2dWgrad` 会把前面所有类型选择展开成一个具体内核类型，`ImplicitGemmConvolution` 再把它包装成带有 `Arguments`、`initialize` 和函数调用操作符的设备 API；后面的 reduction 别名则构建出用于汇总工作区的 `ReduceSplitK` 内核。`ReductionOp` 使用 `ReduceAdd`，所以完整算法就是：先运行多个部分隐式 GEMM，再把它们累加成最终的滤波器梯度张量。

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////

// Command line options parsing
struct Options {

  bool help;
  cutlass::Tensor4DCoord input_size;
  cutlass::Tensor4DCoord filter_size;
  cutlass::Tensor4DCoord padding;
  cutlass::MatrixCoord conv_stride;
  cutlass::MatrixCoord dilation;
  bool reference_check;
  bool measure_performance;
  int iterations;
  bool save_workspace;
  ElementComputeEpilogue alpha;
  ElementComputeEpilogue beta;
  int split_k_slices;
  bool benchmark;
  std::string tag;

  Options():
    help(false),
    input_size(1, 32, 32, 32),
    filter_size(32, 3, 3, 32),
    padding(1, 1, 1, 1),
    conv_stride(1, 1),
    dilation(1, 1),
    reference_check(true),
    measure_performance(false),
    iterations(20),
    save_workspace(false),
    alpha(1),
    beta(0),
    split_k_slices(8),
    benchmark(false) { }
```
**EN**: `Options` collects all runtime parameters: tensor extents, padding/stride/dilation, reference and performance flags, split-K factor, epilogue scalars, workspace dumping, and benchmark mode. The constructor provides a valid default 3x3 same-padding convolution so the sample can run without extra arguments.
**CN**: `Options` 汇总了所有运行时参数：张量尺寸、padding/stride/dilation、参考校验与性能测试开关、split-K 因子、epilogue 标量、工作区导出，以及 benchmark 模式。构造函数给出一个可直接运行的默认 3x3 same-padding 卷积，因此示例无需额外参数也能执行。

```cpp

  // Verify the problem size is compatible with the CUTLASS Convolution implementation.
  bool valid() {

    //
    // CUTLASS attempts to load 128b vectors of cutlass::half_t (F16) elements. Consequently,
    // all pointers, strides, and tensor extents must be divisible by 8 elements.
    //
    int const kAlignment = 8;

    if ((input_size.c() % kAlignment) ||
	(filter_size.n() % kAlignment)) {

      // misaligned tensors
      return false;
    }

    // Invalid padding
    if ((padding.h() != filter_size.h() / 2) ||
	(padding.w() != filter_size.w() / 2)) {

      return false;
    }

    return true;
  }
```
**EN**: `valid()` encodes two important preconditions. First, the NHWC tensors must be aligned to 128-bit vector accesses, so the channel counts must be multiples of 8 FP16 values. Second, the sample only accepts padding equal to half the filter size, which effectively constrains it to the simplified same-padding cases that the rest of the example assumes.
**CN**: `valid()` 编码了两个重要前提。第一，NHWC 张量需要满足 128-bit 向量访存对齐，因此通道数必须是 8 个 FP16 元素的倍数。第二，示例只接受等于滤波器半尺寸的 padding，这等价于把问题限制在后续代码假定的简化 same-padding 场景中。

```cpp

  /// Updates input and filter sizes
  void update(
	      cutlass::Tensor4DCoord input_size,
	      cutlass::Tensor4DCoord filter_size,
	      cutlass::MatrixCoord stride) {

    this->input_size = input_size;
    this->filter_size = filter_size;
    conv_stride = stride;

    padding.n() = filter_size.h() / 2;
    padding.h() = filter_size.h() / 2;
    padding.w() = filter_size.w() / 2;
    padding.c() = filter_size.w() / 2;
  }
```
**EN**: `update()` is used by benchmark mode to rewrite the problem dimensions in one place. It also recomputes the padding fields from the filter shape, so every benchmark layer is forced back into the sample's symmetric-padding convention.
**CN**: `update()` 供 benchmark 模式统一改写问题尺寸。它还会根据滤波器形状重新计算 padding，因此每个基准层都会被强制映射回该示例使用的对称 padding 约定。

```cpp
  // Parses the command line
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);

    if (cmd.check_cmd_line_flag("help")) {
      help = true;
    }

    if (cmd.check_cmd_line_flag("ref-check")) {
      reference_check = true;
    }

    if (cmd.check_cmd_line_flag("perf-check")) {
      measure_performance = true;
    }

    if (cmd.check_cmd_line_flag("save-workspace")) {
      save_workspace = true;
    }

    if (cmd.check_cmd_line_flag("benchmark")) {
      benchmark = true;
    }

    cmd.get_cmd_line_argument("n", input_size.n());
    cmd.get_cmd_line_argument("h", input_size.h());
    cmd.get_cmd_line_argument("w", input_size.w());
    cmd.get_cmd_line_argument("c", input_size.c());

    cmd.get_cmd_line_argument("k", filter_size.n());
    cmd.get_cmd_line_argument("r", filter_size.h());
    cmd.get_cmd_line_argument("s", filter_size.w());
    filter_size.c() = input_size.c();

    cmd.get_cmd_line_argument("alpha", alpha);
    cmd.get_cmd_line_argument("beta", beta);
    cmd.get_cmd_line_argument("split-k-slices", split_k_slices);

    cmd.get_cmd_line_argument("iterations", iterations);
    cmd.get_cmd_line_argument("tag", tag);

    if (filter_size.h() == 3 && filter_size.w() == 3) {
      padding = {1, 1, 1, 1};
    }
    else {
      filter_size.h() = 1;
      filter_size.w() = 1;
      padding = {0, 0, 0, 0};
    }
  }
```
**EN**: `parse()` uses CUTLASS's lightweight `CommandLine` helper to read flags and scalar options. A notable API simplification appears at the end: the sample only keeps true 3x3 filters as 3x3, otherwise it coerces the filter to 1x1 and resets padding to zero. That makes the benchmark driver easy to manage, but it also means the command-line interface intentionally supports only a narrow subset of general convolutions.
**CN**: `parse()` 使用 CUTLASS 轻量级的 `CommandLine` 辅助类解析标志位和标量参数。结尾处有一个明显的 API 简化：只有真正的 3x3 滤波器会被保留为 3x3，其余情况都会被强制改成 1x1，并把 padding 设为 0。这样可以让基准驱动更简单，但也意味着命令行接口是有意只支持一般卷积中的一个较窄子集。

```cpp

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << "30_wgrad_split_k example\n\n"
	<< "  This example shows how to compute conv2d gradient with respect to weight (wgrad).\n"
	<< "  In wgrad, the K dimension of impligit GEMM, corresponding to the sequential reduction loop, is very large (N * P * Q).\n"
	<< "  Split-k with parallel reduction is highly effective for such cases.\n\n"
	<< "Options:\n\n"
	<< "  --help               If specified, displays this usage statement.\n\n"
	<< "  --n=<int>            Input tensor extent N\n"
	<< "  --h=<int>            Input tensor extent H\n"
	<< "  --w=<int>            Input tensor extent W\n"
	<< "  --c=<int>            Input tensor extent C\n"
	<< "  --k=<int>            Filter extent K\n"
	<< "  --r=<int>            Filter extent R\n"
	<< "  --s=<int>            Filter extent S\n\n"
	<< "  --alpha=<float>      Epilogue scalar alpha\n"
	<< "  --beta=<float>       Epilogue scalar beta\n\n"
	<< "  --split-k-slices=<int>   Split-k factor \n\n"
	<< "  --ref-check          If set (true), reference check on the host is computed\n"
	<< "  --perf-check         If set (true), performance is measured.\n"
	<< "  --benchmark          If set (true), performance benchmarking on several layers and batch-size.\n"
	<< "  --iterations=<int>   Number of profiling iterations to perform.\n"
	<< "  --save-workspace     If set, workspace is written to a text file.\n"
	<< "  --tag=<string>       String to replicate across the first column in the results table\n";

    out << "\n\nExamples:\n\n"
	<< "$ ./examples/30_wgrad_split_k/30_wgrad_split_k --n=32 --h=224 --w=224 --c=128 --k=256 --r=3 --s=3 --split-k-slices=8\n\n";

    return out;
  }

  /// Computes the output tensor size (NPQK)
  cutlass::Tensor4DCoord output_size() const {
    return cutlass::Tensor4DCoord(input_size.n(),
				  (input_size.h() + padding.n() + padding.h() - filter_size.h()) / conv_stride.row() + 1,
				  (input_size.w() + padding.w() + padding.c() - filter_size.w()) / conv_stride.column() + 1,
				  filter_size.n());
  }

  /// Compute performance in GFLOP/s
  double gflops(double runtime_s) const {

    // Number of multiply-adds = NPQK * CRS
    int64_t fmas = output_size().product() * int64_t(filter_size.h() * filter_size.w() * filter_size.c());

    // Two flops per multiply-add
    return 2.0 * double(fmas) / double(1.0e9) / runtime_s;
  }
};
```
**EN**: These utility methods handle user-facing reporting. `print_usage()` documents the example and its flags, `output_size()` computes the forward output tensor shape `(N,P,Q,K)` from convolution geometry, and `gflops()` converts runtime to throughput by counting the wgrad multiply-adds implied by the output volume and filter footprint.
**CN**: 这些辅助方法负责面向用户的说明与统计。`print_usage()` 输出示例用途和参数说明，`output_size()` 根据卷积几何关系计算前向输出张量 `(N,P,Q,K)` 的形状，而 `gflops()` 则依据输出体积与滤波器覆盖范围统计 wgrad 所需的乘加次数，并把运行时间换算为吞吐率。

```cpp

/////////////////////////////////////////////////////////////////////////////////////////////////

struct Result {
  double runtime_ms;
  double gflops;
  cutlass::Status status;
  cutlass::Status reference_check;
  cudaError_t error;

  Result():
    runtime_ms(0),
    gflops(0),
    status(cutlass::Status::kSuccess),
    reference_check(cutlass::Status::kInvalid),
    error(cudaSuccess) { }

  static std::ostream & print_header(std::ostream &out, Options const &options) {

    if (!options.tag.empty()) {
      out << "Name,";
    }

    out << "Layer,N,H,W,C,K,R,S,Stride_H,Stride_W,Runtime,GFLOPs";

    return out;
  }

  std::ostream & print(std::ostream &out, int idx, Options const &options) {

    if (!options.tag.empty()) {
      out << options.tag << ",";
    }

    out
      << "conv_" << idx << ","
      << options.input_size.n() << ","
      << options.input_size.h() << ","
      << options.input_size.w() << ","
      << options.input_size.c() << ","
      << options.filter_size.n() << ","
      << options.filter_size.h() << ","
      << options.filter_size.w() << ","
      << options.conv_stride.row() << ","
      << options.conv_stride.column() << ","
      << runtime_ms << ","
      << gflops;

    return out;
  }
};
```
**EN**: `Result` is a small reporting container for runtime, throughput, CUTLASS status, reference-check status, and CUDA error code. The two print helpers emit CSV-friendly headers and rows, which is why benchmark mode can stream many layers directly to stdout without extra formatting logic.
**CN**: `Result` 是一个小型结果容器，用来保存运行时间、吞吐率、CUTLASS 状态、参考校验状态以及 CUDA 错误码。两个打印辅助函数会输出适合 CSV 的表头和数据行，因此 benchmark 模式可以直接把多层结果连续写到标准输出，而不需要额外格式化逻辑。

```cpp

/////////////////////////////////////////////////////////////////////////////////////////////////

/// Runs one benchmark
Result profile_convolution(Options const &options) {

  Result result;

  //
  // Allocate host-device tensors using the CUTLASS Utilities.
  //

  // Inputs are the output gradient and the original activation.
  cutlass::HostTensor<ElementInputA, LayoutInputA> tensor_a(options.output_size());
  cutlass::HostTensor<ElementInputB, LayoutInputB> tensor_b(options.input_size);
  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_c(options.filter_size);
  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_d(options.filter_size);
  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_ref_d(options.filter_size);

  //
  // Initialize tensors
  //

  // Fill tensor A on host with uniform-distribution random data
  cutlass::reference::host::TensorFillRandomUniform(
      tensor_a.host_view(),
      1,
      ElementInputA(7),
      ElementInputA(-8),
      0);

  // Fill tensor B on host with uniform-distribution random data
  cutlass::reference::host::TensorFillRandomUniform(
      tensor_b.host_view(),
      1,
      ElementInputB(7),
      ElementInputB(-8),
      0);

  // Fill tensor C, D on host with zeros
  cutlass::reference::host::TensorFill(tensor_c.host_view());

  cutlass::reference::host::TensorFill(tensor_d.host_view());

  // Fill tensor D for reference on host with zeros
  cutlass::reference::host::TensorFill(tensor_ref_d.host_view());

  // Copy data from host to GPU
  tensor_a.sync_device();
  tensor_b.sync_device();
  tensor_c.sync_device();
  tensor_d.sync_device();
  tensor_ref_d.sync_device();
```
**EN**: `profile_convolution()` begins by allocating CUTLASS `HostTensor` objects for the output gradient (`tensor_a`), original activations (`tensor_b`), the source/filter tensor used in the epilogue (`tensor_c`), the CUTLASS result (`tensor_d`), and a reference result buffer. Random initializers populate the inputs, the output-related tensors are zeroed, and `sync_device()` copies the initialized host buffers to GPU memory. This is the sample's main host/device data management layer.
**CN**: `profile_convolution()` 首先为输出梯度（`tensor_a`）、原始激活（`tensor_b`）、epilogue 使用的源/滤波器张量（`tensor_c`）、CUTLASS 结果（`tensor_d`）以及参考结果缓冲区分配 `HostTensor`。输入使用随机值初始化，输出相关张量清零，然后通过 `sync_device()` 把主机端初始化数据复制到 GPU。这里就是示例的主要主机/设备数据管理层。

```cpp

  //
  // Define arguments for CUTLASS Convolution
  //

  cutlass::conv::Mode mode = cutlass::conv::Mode::kCrossCorrelation;

  // Partition the GEMM K loop into split_k_slices chunks
  int split_k_slices = options.split_k_slices;

  // Construct Conv2dProblemSize with user defined output size
  // Do not forget to pass the last argument.
  cutlass::conv::Conv2dProblemSize problem_size(
      options.input_size,
      options.filter_size,
      options.padding,
      options.conv_stride,
      options.dilation,
      options.output_size(),
      mode,
      split_k_slices
  );

  using cutlass::layout::TensorNHWC;

  cutlass::conv::SplitKMode const split_k_mode = cutlass::conv::SplitKMode::kParallel;

  // Since the epilogue is not computed after GEMM, there is no need to pass the C tensor and
  // alpha and beta can be set to 1 and 0 respectively.
  // Moreover, since the output will be written to the workspace, there is no need to pass
  // the D tensor as well.
  // Do not forget to pass the last argument.
  typename ImplicitGemm::Arguments arguments{
    problem_size,
    tensor_a.device_ref(),
    tensor_b.device_ref(),
    {nullptr, TensorNHWC()},
    {nullptr, TensorNHWC()},
    {ElementCompute(1), ElementCompute(0)},
    split_k_mode
  };
```
**EN**: This block constructs the convolution problem and the first-stage kernel arguments. `Conv2dProblemSize` captures input/filter/padding/stride/dilation/output geometry and the split-K slice count. The `ImplicitGemm::Arguments` object then passes device tensor references plus `{nullptr, TensorNHWC()}` for C and D, because the first stage is only producing partial sums into workspace; it does not yet need a final destination tensor. The scalars are fixed to `(1,0)` because scaling is deferred to the reduction epilogue.
**CN**: 这一段构造卷积问题描述和第一阶段内核参数。`Conv2dProblemSize` 打包输入/滤波器/padding/stride/dilation/输出几何信息以及 split-K 切片数。随后 `ImplicitGemm::Arguments` 传入设备张量引用，并把 C、D 都设置为 `{nullptr, TensorNHWC()}`，因为第一阶段只需要把部分和写入工作区，还不需要真正的最终输出张量。标量固定为 `(1,0)`，因为缩放工作被推迟到归约 epilogue。

```cpp

  //
  // Initialize CUTLASS Convolution
  //

  ImplicitGemm implicit_gemm;

  size_t workspace_size = implicit_gemm.get_workspace_size(arguments);

  // Split-K requires non-zero workspace size. The workspace size grows linearly with split_k_slices.
  std::cout << "split-k-slices: " << split_k_slices << std::endl;
  std::cout << "workspace size: " << workspace_size << std::endl;

  // Allocate workspace memory
  cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);

  result.status = implicit_gemm.can_implement(arguments);
  CUTLASS_CHECK(result.status);

  // After the workspace is allocated, we point the GEMM destination pointer to the workspace.
  TensorNHWC layout_D{TensorNHWC::packed(options.filter_size)};
  arguments.ref_D.reset(reinterpret_cast<ElementCompute*>(workspace.get()), layout_D);

  result.status = implicit_gemm.initialize(arguments, workspace.get());
  CUTLASS_CHECK(result.status);

  //
  // Launch initialized CUTLASS kernel
  //
  result.status = implicit_gemm();

  CUTLASS_CHECK(result.status);
```
**EN**: The CUTLASS device wrapper is then instantiated and queried for workspace size. This is a central split-K API pattern: `get_workspace_size()` tells the host how much temporary storage the partial accumulations need, the host allocates it, and `arguments.ref_D` is rebound to that workspace as an `ElementCompute` tensor with packed NHWC layout. After `can_implement()` and `initialize()`, calling `implicit_gemm()` launches the first-stage kernel. In other words, the convolution kernel is composed as a workspace-producing producer rather than a final writer.
**CN**: 接着会实例化 CUTLASS 设备封装对象，并查询工作区大小。这是 split-K 的核心 API 模式：`get_workspace_size()` 告诉主机部分累加需要多少临时存储，主机完成分配后，再把 `arguments.ref_D` 重新绑定到该工作区，并把它视为采用紧凑 NHWC 布局的 `ElementCompute` 张量。通过 `can_implement()` 与 `initialize()` 之后，调用 `implicit_gemm()` 就会发射第一阶段内核。换句话说，这里的卷积内核被组合成“写工作区的生产者”，而不是最终结果写回器。

```cpp

  if (split_k_mode == cutlass::conv::SplitKMode::kParallel) {
    // Do reduction
    ReductionDevice reduction_op;
    auto& status = result.status;
    static cutlass::conv::Operator const kConvolutionalOperator = ImplicitGemm::kConvolutionalOperator;
    typename ReductionDevice::Arguments reduction_args(
        cutlass::conv::implicit_gemm_problem_size(kConvolutionalOperator, problem_size).mn(),
        problem_size.split_k_slices,
        cutlass::conv::implicit_gemm_tensor_c_size(kConvolutionalOperator, problem_size),
        // Reduction input
        {
            reinterpret_cast<ElementAccumulator*> (workspace.get()),
            ReductionStrideIndex(tensor_c.stride()[ImplicitGemm::UnderlyingKernel::kTensorCStrideIdx])
        },
        // Destination
        {
            tensor_d.device_data(),
            ReductionStrideIndex(tensor_d.stride()[ImplicitGemm::UnderlyingKernel::kTensorCStrideIdx])
        },
        // Source
        {
            tensor_c.device_data(),
            ReductionStrideIndex(tensor_c.stride()[ImplicitGemm::UnderlyingKernel::kTensorCStrideIdx])
        },
        {options.alpha, options.beta}
    );

    status = reduction_op.initialize(reduction_args, nullptr);
    status = reduction_op();
  }
```
**EN**: If split-K is parallel, the second stage runs explicitly. `ReduceSplitK` is configured with the implicit GEMM output `mn()` extent, the number of slices, and the tensor-C size metadata that CUTLASS uses to stride the reduction correctly. The workspace becomes the reduction input, `tensor_d` is the final destination, `tensor_c` supplies the source tensor used with `beta`, and `{options.alpha, options.beta}` applies the real epilogue scaling. This is the actual completion of the wgrad computation.
**CN**: 如果 split-K 采用并行模式，就会显式执行第二阶段。`ReduceSplitK` 会使用隐式 GEMM 输出的 `mn()` 维度、切片数量以及 CUTLASS 用于正确计算步幅的 tensor-C 大小元数据来完成配置。工作区作为归约输入，`tensor_d` 作为最终输出，`tensor_c` 作为与 `beta` 搭配使用的源张量，而 `{options.alpha, options.beta}` 则应用真正的 epilogue 缩放。这一步才是 wgrad 计算的真正收尾。

```cpp

  //
  // Optional reference check
  //

  if (options.reference_check) {
    std::cout << "Verification on device...\n";

    // Compute with reference implementation
    cutlass::reference::device::Conv2dWgrad<
      ElementInputA,
	LayoutInputA,
	ElementInputB,
	LayoutInputB,
	ElementOutput,
	LayoutOutput,
	ElementComputeEpilogue,
	ElementAccumulator,
	cutlass::NumericConverter<ElementOutput, ElementComputeEpilogue>
	>(
	  problem_size,
	  tensor_a.device_ref(),
	  tensor_b.device_ref(),
	  tensor_c.device_ref(),
	  tensor_ref_d.device_ref(),
	  options.alpha,
	  options.beta
	  );

    // Check if output from CUTLASS kernel and reference kernel are equal or not
    tensor_c.sync_host();
    tensor_d.sync_host();
    tensor_ref_d.sync_host();

    bool passed = cutlass::reference::host::TensorEquals(tensor_d.host_view(), tensor_ref_d.host_view());

    if (!passed) {
      result.reference_check = cutlass::Status::kErrorInternal;
      std::cout << "ERROR - results miscompared.\n";
    }
    else {
      result.reference_check = cutlass::Status::kSuccess;
      std::cout << "Passed.\n";
    }
  }
  else {
    result.reference_check = cutlass::Status::kInvalid;
  }
```
**EN**: The optional verification path runs CUTLASS's device reference `Conv2dWgrad`, synchronizes both outputs back to the host, and compares them with `TensorEquals`. This mirrors the same problem description and makes the sample self-checking without requiring an external framework. If verification is disabled, the status is left as `kInvalid` to make that explicit in any later reporting.
**CN**: 可选校验路径会调用 CUTLASS 自带的设备端参考实现 `Conv2dWgrad`，把两份结果同步回主机，再通过 `TensorEquals` 做比较。这样示例无需依赖外部框架就能自校验，而且问题描述与主内核保持一致。如果关闭校验，代码会把状态保留为 `kInvalid`，以便后续报告明确表示“未检查”。

```cpp

  if (options.save_workspace) {

    std::stringstream ss;

    ss << "30_wgrad_split_k_"
       << options.input_size.n() << "x" << options.input_size.h() << "x" << options.input_size.w() << "x" << options.input_size.c()
       << "_"
       << options.filter_size.n() << "x" << options.filter_size.h() << "x" << options.filter_size.w() << "x" << options.filter_size.c()
       << ".dat";

    std::ofstream output_workspace(ss.str());

    output_workspace
      << "Input = \n" << tensor_a.host_view() << "\n\n"
      << "Filters = \n" << tensor_b.host_view() << "\n\n";

    if (options.reference_check) {
      output_workspace << "Reference = \n" << tensor_ref_d.host_view() << "\n\n";
    }

    output_workspace << "Computed = \n" << tensor_c.host_view() << std::endl;

    std::cout << "Results written to '" << ss.str() << "'." << std::endl;
  }
```
**EN**: When `save_workspace` is enabled, the sample writes a human-readable dump file containing the host views of the main tensors. The filename encodes input and filter sizes, which is convenient for offline debugging. Note that the value labeled `Computed` is emitted from `tensor_c.host_view()` exactly as the code is written, so the dump reflects the program's current behavior rather than an additional post-processing step.
**CN**: 启用 `save_workspace` 后，示例会把主要张量的主机端视图写成一个可读的文本文件。文件名会编码输入和滤波器尺寸，便于离线调试。需要注意，标签为 `Computed` 的内容是按当前代码直接从 `tensor_c.host_view()` 输出的，因此这个导出反映的是程序现有行为，而不是额外的后处理结果。

```cpp

  //
  // Performance measurement
  //

  if (options.measure_performance) {

    cudaEvent_t events[2];

    for (auto & event : events) {
      result.error = cudaEventCreate(&event);
      if (result.error != cudaSuccess) {
        std::cerr << "cudaEventCreate() failed: " << cudaGetErrorString(result.error) << std::endl;
        return result;
      }
    }

    // Record an event at the start of a series of convolution operations.
    result.error = cudaEventRecord(events[0]);
    if (result.error != cudaSuccess) {
      std::cerr << "cudaEventRecord() failed: " << cudaGetErrorString(result.error) << std::endl;
      return result;
    }

    // Launch a sequence of implicit GEMM operations on the device
    for (int iteration = 0; iteration < options.iterations; ++iteration) {
      result.status = implicit_gemm();
      CUTLASS_CHECK(result.status);
    }

    // Record an event when the convolutions have been launched.
    result.error = cudaEventRecord(events[1]);
    if (result.error != cudaSuccess) {
      std::cerr << "cudaEventRecord() failed: " << cudaGetErrorString(result.error) << std::endl;
      return result;
    }

    // Wait for work on the device to complete.
    result.error = cudaEventSynchronize(events[1]);
    if (result.error != cudaSuccess) {
      std::cerr << "cudaEventSynchronize() failed: " << cudaGetErrorString(result.error) << std::endl;
      return result;
    }

    // Measure elapsed runtime
    float runtime_ms = 0;
    result.error = cudaEventElapsedTime(&runtime_ms, events[0], events[1]);
    if (result.error != cudaSuccess) {
      std::cerr << "cudaEventElapsed() failed: " << cudaGetErrorString(result.error) << std::endl;
      return result;
    }

    // Print average runtime and GFLOPs.
    result.runtime_ms = double(runtime_ms) / double(options.iterations);
    result.gflops = options.gflops(result.runtime_ms / 1000.0);

    // Cleanup
    for (auto event : events) {
      (void)cudaEventDestroy(event);
    }
  }

  return result;
```
**EN**: The performance path measures runtime with CUDA events and reports average milliseconds plus derived GFLOP/s. The loop repeatedly invokes `implicit_gemm()` after the kernel has already been initialized, which isolates steady-state launch performance and avoids setup overhead. Because the timed loop only relaunches the implicit GEMM stage, the reported runtime emphasizes the producer kernel more than the separate reduction step.
**CN**: 性能路径使用 CUDA event 计时，并给出平均毫秒数与推导出的 GFLOP/s。计时循环在内核完成初始化之后反复调用 `implicit_gemm()`，这样可以突出稳定态执行时间，并避开初始化开销。由于被计时的循环只重复发射隐式 GEMM 阶段，所以报告的运行时间更偏向第一阶段生产者内核，而不是单独的归约步骤。

```cpp
}

/////////////////////////////////////////////////////////////////////////////////////////////////

int main(int argc, char const **args) {
  bool notSupported = false;

  // Ampere Tensor Core operations exposed with mma.sync are first available in CUDA 11.0.
  //
  // CUTLASS must be compiled with CUDA 11 Toolkit to run Conv2dFprop examples.
  if (!(__CUDACC_VER_MAJOR__ > 11 || (__CUDACC_VER_MAJOR__ == 11 && __CUDACC_VER_MINOR__ >= 0))) {
    std::cerr << "Ampere Tensor Core operations must be compiled with CUDA 11.0 Toolkit or later." << std::endl;
    notSupported = true;
  }

  cudaDeviceProp props;
  CUDA_CHECK(cudaGetDeviceProperties(&props, 0));

  if (!(props.major >= 8)) {
    std::cerr << "Ampere Tensor Ops must be run on a machine with compute capability at least 80."
              << std::endl;
    notSupported = true;
  }

  if (notSupported) {
    return 0;
  }

  Options options;

  options.parse(argc, args);

  if (options.help) {
    options.print_usage(std::cout) << std::endl;
    return 0;
  }
```
**EN**: `main()` first enforces the architectural requirements of this instantiation: CUDA 11+ and SM80+ for Ampere Tensor Core instructions. It then parses command-line options and prints usage text immediately if `--help` was requested. This keeps unsupported environments from falling through into kernel setup.
**CN**: `main()` 首先检查该实例化所需的架构前提：需要 CUDA 11 及以上，以及支持 Ampere Tensor Core 指令的 SM80 及以上设备。随后它解析命令行参数，并在请求 `--help` 时立即打印用法说明。这样可以避免不支持的环境继续进入内核配置流程。

```cpp

  if (options.benchmark) {
    // Benchmark several layers

    int batch_sizes[] = {34, 408};

    struct Benchmark {
      int h, w, c, k, r, s, stride_h, stride_w;
    } layers[] = {
	 {56, 56,   64,  256, 1, 1, 1, 1},
	 {56, 56,   64,   64, 1, 1, 1, 1},
	 {56, 56,   64,   64, 3, 3, 1, 1},
	 {56, 56,  256,   64, 1, 1, 1, 1},
	 {56, 56,  256,  512, 1, 1, 2, 2},
	 {56, 56,  256,  128, 1, 1, 1, 1},
	 {56, 56,  128,  128, 3, 3, 2, 2},
	 {28, 28,  128,  512, 1, 1, 1, 1},
	 {28, 28,  512,  128, 1, 1, 1, 1},
	 {28, 28,  128,  128, 3, 3, 1, 1},
	 {28, 28,  512, 1024, 1, 1, 2, 2},
	 {28, 28,  512,  256, 1, 1, 1, 1},
	 {28, 28,  256,  256, 3, 3, 2, 2},
	 {14, 14,  256, 1024, 1, 1, 1, 1},
	 {14, 14, 1024,  256, 1, 1, 1, 1},
	 {14, 14,  256,  256, 3, 3, 1, 1},
	 {14, 14, 1024, 2048, 1, 1, 2, 2},
	 {14, 14, 1024,  512, 1, 1, 1, 1},
	 {14, 14,  512,  512, 3, 3, 2, 2},
	 { 7,  7,  512, 2048, 1, 1, 1, 1},
	 { 7,  7, 2048,  512, 1, 1, 1, 1},
	 { 7,  7,  512,  512, 3, 3, 1, 1},
    };

    Result::print_header(std::cout, options) << std::endl;

    int idx = 1;

    for (auto const &layer : layers) {
      for (auto N : batch_sizes) {
        options.update({N, layer.h, layer.w, layer.c},
                       {layer.k, layer.r, layer.s, layer.c},
                       {layer.stride_h, layer.stride_w});

        Result result = profile_convolution(options);
        result.print(std::cout, idx, options) << std::endl;
      }

      ++idx;
    }
  }
```
**EN**: Benchmark mode hardcodes a set of ResNet-like layers and two batch sizes, updates the option object for each case, profiles the convolution, and prints one CSV row per run. This turns the example into a lightweight microbenchmark harness built on the same `profile_convolution()` function used for single runs.
**CN**: benchmark 模式内置了一组类似 ResNet 的层配置和两个 batch size，并为每个组合更新 `Options`、执行 profile、输出一行 CSV。这样一来，该示例就不仅是功能样例，也变成了一个基于同一 `profile_convolution()` 函数的轻量级微基准工具。

```cpp
  else {

    // Execute one problem size
    if (!options.valid()) {
      std::cerr << "Invalid problem." << std::endl;
      return -1;
    }

    Result result = profile_convolution(options);

    Result::print_header(std::cout, options) << std::endl;
    result.print(std::cout, 1, options) << std::endl;
  }

  return 0;
}

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: The non-benchmark path validates one problem, runs `profile_convolution()`, prints a header and a single result row, and exits. This is the simplest end-to-end flow: validate geometry, allocate tensors, launch split-K implicit GEMM, reduce, optionally verify, optionally time, then summarize.
**CN**: 非 benchmark 路径只验证一个问题，调用 `profile_convolution()`，打印表头和单行结果，然后退出。这也是最直接的端到端流程：先验证几何配置，再分配张量、发射 split-K 隐式 GEMM、执行归约、可选校验、可选计时，最后输出摘要。

---
## Key Concepts / 关键概念
- Implicit GEMM mapping for conv2d wgrad / 将 conv2d wgrad 映射为隐式 GEMM
- Parallel split-K workspace plus `ReduceSplitK` finalization / 并行 split-K 工作区与 `ReduceSplitK` 收尾
- FP16 inputs with FP32 accumulation and FP16 final output / FP16 输入、FP32 累加、FP16 最终输出
- NHWC tensor layout and optimized convolution iterators / NHWC 张量布局与优化卷积迭代器
- Two-stage kernel composition through CUTLASS device APIs / 通过 CUTLASS 设备 API 组合两阶段内核
## Dependencies / 依赖项
- `iostream` — console output / 控制台输出
- `fstream` — workspace dump file output / 工作区转储文件输出
- `sstream` — string formatting for filenames and messages / 用于文件名和消息的字符串格式化
- `cutlass/cutlass.h` — CUTLASS core types and utilities / CUTLASS 核心类型与工具
- `cutlass/gemm/device/gemm.h` — GEMM-related device abstractions used by convolution plumbing / 卷积封装依赖的 GEMM 设备抽象
- `cutlass/conv/kernel/default_conv2d_wgrad.h` — builds the concrete conv2d wgrad kernel type / 生成具体的 conv2d wgrad 内核类型
- `cutlass/conv/device/implicit_gemm_convolution.h` — device wrapper for launching implicit GEMM convolution / 发射隐式 GEMM 卷积的设备封装
- `cutlass/util/command_line.h` — parses command-line options / 解析命令行参数
- `cutlass/util/host_tensor.h` — host/device tensor container for sample data / 示例数据的主机/设备张量容器
- `cutlass/util/tensor_view_io.h` — formatted tensor printing / 格式化张量输出
- `cutlass/util/reference/device/gemm.h` — reference GEMM utilities included by the sample / 示例包含的参考 GEMM 工具
- `cutlass/util/reference/host/tensor_compare.h` — host-side tensor comparison / 主机端张量比较
- `cutlass/util/reference/host/tensor_copy.h` — host tensor copy helpers / 主机端张量复制辅助
- `cutlass/util/reference/host/tensor_fill.h` — host tensor initialization helpers / 主机端张量初始化辅助
- `cutlass/util/reference/device/convolution.h` — device-side reference conv2d implementations / 设备端参考卷积实现
- `cutlass/reduction/device/reduce_split_k.h` — device API for split-K reduction / split-K 归约的设备 API
- `cutlass/reduction/thread/reduction_operators.h` — thread-level reduction operators such as add / 线程级归约操作，如加法
- `helper.h` — local CUDA/CUTLASS checking macros / 本地 CUDA/CUTLASS 检查宏
