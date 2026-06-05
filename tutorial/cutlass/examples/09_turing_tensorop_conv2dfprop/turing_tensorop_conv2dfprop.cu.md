# turing_tensorop_conv2dfprop.cu — Code Analysis / 代码分析

## Source / 源文件

`examples/09_turing_tensorop_conv2dfprop/turing_tensorop_conv2dfprop.cu`

## Purpose / 用途

**EN:** Demonstrates CUTLASS implicit-GEMM forward 2D convolution on Turing Tensor Cores using int4 tensors, optional verification, workspace dumping, and benchmarking.

**CN:** 演示在 Turing Tensor Core 上使用 int4 张量的 CUTLASS implicit-GEMM 二维前向卷积，并支持可选验证、工作区导出和性能基准。

## Line-by-Line Analysis / 逐行分析

### Lines 1-145 / 第 1-145 行

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

/**


This example shows how to run convolution kernels using functions and data structures
provided by CUTLASS using tensor cores; which we run on a NVIDIA Turing GPU.

Writing a single high performance convolution kernel is hard but do-able. Whereas writing
high performance kernels at scale which works for multiple problem sizes with good abstractions is
really hard. CUTLASS solves this problem by providing simplified abstractions to compose
multiple sections of implicit gemm kernel. When used properly, the kernels can hit peak performance
of GPU easily.

CUTLASS divides a kernel into hierarchical composable sections. Which means, at each thread, warp
and thread-block level, they compute on their own tile-size with higher level of tile sizes being
composed from lower level ones. Multiple thread-tiles (tile size each thread computes) can be used
to form warp-tiles (tile size each warp computes) and multiple warp tiles can be used to compute
threadblock-tile (tile size computed by a threadblock).

In thie example, we split variable initialization into
1. Setting up data properties : describes how tensors are laid out in the memory and how the kernel
can view them (logical to physical mapping)
2. Setting up computation properties : describes how the above set tensors will be used to compute
output of convolution.

First, we setup the data types of the input tensor A, weights' tensor B and output tensor C along
with alpha, beta as the equation for convolution is C = alpha * Conv(A, B) + beta * C. In CUTLASS,
the kernels first compute Conv(A, B) and leave the rest of the computation to end of the kernel as
alpha * X + beta * C is a simple element-wise operation on X (Conv(A, B)) and C. We call this as 
epilogue of kernel. Hence, we setup data types for alpha and beta to be equal to 
ElementComputeEpilogue = float. We want to use MMA instructions on Turing and they support 4-bit
signed integer. But int4b_t is not fully supported by Nvidia software stack, so CUTLASS introduces
cutlass::int4b_t. We use the data type for elements in input tensor A and B as cutlass::int4b_t. We
convey this to CUTLASS kernel by initializing template variables ElementAccumulator (int32_t),
ElementComputeEpilogue (float), ElementInputA (cutlass::int4b_t), ElementInputB (cutlass::int4b_t),
ElementOutput (int32_t). Communicating just the data type is not enough. As the data is laid out 
linearly in memory, we have to convey the layout of tensors. We do that by initializing template
variables LayoutInputA, LayoutInputB and LayoutOutput to TensorNHWC cutlass variable. Next, we setup
rules to compute alpha * X + beta * C which is called epilogue of the kernel. We initialize template
variable EpilogueOp, which takes the data type of output ElementOutput (int32_t), the number of
elements per vector memory access (32), data type of accumulator (int32_t) and data type of
computation of linear combination (alpha * X + beta * C).

Now that we setup the properties of data, we have to setup properties of computation.

Second, we create template variables of tile sizes for thread-block, warp and mma-op to 128x128x128,
64x64x128, 8x8x32 (MxNxK) respectively. When passed to instantiate CUTLASS Implicit GEMM kernel, it
internally deduces the amount of threads needed per thread-block, amount of shared memory, storing
data in bank-conflict free manner, and ton of other variables required to compose, initialize and
launch a high performance Implicit GEMM kernel. This is the beauty of CUTLASS, it relieves developer
from understanding and coding complicated hardware optimizations which can easily go wrong.

CUTLASS also supports multiple MMA pipelines in a threadblock. What are MMA pipelines? MMA pipelines
constitute the whole process of loading input data from global memory to shared memory, loading data
from shared memory to registers, doing matrix multiplication, store to global memory. The below flow
sequence shows a typical mma pipeline.

tensor in global memory -> registers -> tile in shared memory -> registers -> mma -> registers ->
output to global memory

The problem with single pipeline is, each stage is synchronous which means, each stage has to wait
until the previous finished executing. There are stages in the pipeline which do not have fixed
latency, for example, the loads from global memory and shared memory. Therefore, we can add one more
pipeline with a phase shift in mma kernel to hide latency from global and shared memory loads.
Finally, the pipeline in a kernel looks like

(1) tensor in global memory -> (2) registers -> (3) tile in shared memory -> (4) registers -> (5)
mma -> (6) registers -> (7) output to global memory (1) <null> -> (2) <null> -> (3) tensor in global
memory -> (4) registers -> (5) tile in shared memory -> (6) registers -> (7) mma -> (8) registers ->
(9) output to global memory

This way, you can hide the second global memory load latency by doing computation on already loaded
input data.

There are few more template variables initialized such as, which threadblock tile of output matrix
is done which threadblock launched on an SM, CUDA SM architecture of GPU you want to run on.

These are all put together to create a template variable which describes CUTLASS Implicit GEMM
kernel using cutlass::conv::device::ImplicitGemm template.

The next step is to initialize physical data, instantiate and initialize CUTLASS kernel and run it.
We use CUTLASS utilities to initialize, fill, compare tensors as they are simple and doesn't come
in the way of learning CUTLASS.

Once all the tensors are initialized and filled with data, create arguments tuple to launch CUTLASS
kernel which takes problem size (N = 1, H = 64, W = 64, C = 128), filter size (K = 64,
R = 3, S = 3, C = 128 ), padding, strides, dilation, tensors, alpha, beta and the
important one, split k-dimension factor. Along with that, we query CUTLASS if any scratch-space
memory required by the kernel we instantiated. If yes, we create it and pass it along with other
arguments created to initialize CUTLASS kernel then, the kernel is launched.

In this example, we later on launch a reference convolution kernel (from CUTLASS utilities) to
compare if the output from CUTLASS kernel is same as the reference implicit GEMM kernel.
*/

#include <iostream>
#include <fstream>
#include <sstream>

#include "cutlass/cutlass.h"
#include "cutlass/gemm/device/gemm.h"
#include "cutlass/conv/kernel/default_conv2d_fprop.h"
#include "cutlass/conv/device/implicit_gemm_convolution.h"

#include "cutlass/util/command_line.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/reference/device/gemm.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/convolution.h"
#include "cutlass/util/tensor_view_io.h"

#include "helper.h"
```

**EN:** License text, tutorial documentation, and includes introduce implicit-GEMM convolution, Turing int4 Tensor Cores, CUTLASS tiling, pipelining, command-line utilities, reference code, and helper macros.

**CN:** 许可证、教程文档和 include 介绍 implicit-GEMM 卷积、Turing int4 Tensor Core、CUTLASS tile、流水线、命令行工具、参考代码和辅助宏。

### Lines 146-207 / 第 146-207 行

```cpp
// The code section below describes datatype for input, output tensors and computation between
// elements
using ElementAccumulator = int32_t;                 // Data type of accumulator
using ElementComputeEpilogue = float;               // Data type of epilogue computation (alpha, beta)
using ElementInputA = cutlass::int4b_t;             // Data type of elements in input tensor
using ElementInputB = cutlass::int4b_t;             // Data type of elements in input tensor
using ElementOutput = cutlass::int4b_t;             // Data type of elements in output tensor

using LayoutInputA = cutlass::layout::TensorNHWC;
using LayoutInputB = cutlass::layout::TensorNHWC;
using LayoutOutput = cutlass::layout::TensorNHWC;

// This code section describes whether you want to use tensor cores or regular SIMT cores on GPU SM
using MMAOp = cutlass::arch::OpClassTensorOp;

// This code section describes CUDA SM architecture number
using SmArch = cutlass::arch::Sm75;

// This code section describes the tile size a thread block will compute
using ThreadblockShape = cutlass::gemm::GemmShape<128, 128, 128>;  // Threadblock tile shape

// This code section describes tile size a warp will compute
using WarpShape = cutlass::gemm::GemmShape<64, 64, 128>;         // Warp tile shape

// This code section describes the size of MMA op
using InstructionShape = cutlass::gemm::GemmShape<8, 8, 32>;    // TensorCore instruction shape

// This code section describes how threadblocks are scheduled on GPU
using SwizzleThreadBlock = cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<>;

// Number of pipelines you want to use
constexpr int NumStages = 2;

// This code section describes the epilogue part of the kernel, we use default value
using EpilogueOp = cutlass::epilogue::thread::LinearCombinationClamp<
    ElementOutput,                                     // Data type of output matrix.
    8,                                                 // The number of elements per vectorized.
                                                       // memory access. This becomes the vector width of
                                                       // math instructions in the epilogue too.
    ElementAccumulator,                                // Data type of accumulator
    ElementComputeEpilogue>;                           // Data type for alpha/beta in linear combination


using Conv2dFpropKernel = typename cutlass::conv::kernel::DefaultConv2dFprop<
  ElementInputA, LayoutInputA,
  ElementInputB, LayoutInputB,
  ElementOutput, LayoutOutput,
  ElementAccumulator,
  MMAOp,
  SmArch,
  ThreadblockShape,
  WarpShape,
  InstructionShape,
  EpilogueOp,
  SwizzleThreadBlock,
  NumStages,
  cutlass::arch::OpMultiplyAddSaturate,
  cutlass::conv::IteratorAlgorithm::kAnalytic
>::Kernel;

using ImplicitGemm = cutlass::conv::device::ImplicitGemmConvolution<Conv2dFpropKernel>;

```

**EN:** Type/layout aliases select int32 accumulation, float epilogue scalars, CUTLASS int4 inputs/output, NHWC tensor layouts, Tensor Core `Sm75`, `128x128x128`/`64x64x128`/`8x8x32` shapes, identity swizzle, two stages, clamping epilogue, saturating multiply-add, analytic iterators, and the device `ImplicitGemmConvolution` wrapper.

**CN:** 类型/布局别名选择 int32 累加、float 尾声标量、CUTLASS int4 输入/输出、NHWC 张量布局、Tensor Core `Sm75`、`128x128x128`/`64x64x128`/`8x8x32` 形状、identity swizzle、两级流水、钳位尾声、饱和乘加、analytic 迭代器，以及 device 级 `ImplicitGemmConvolution` 封装。

### Lines 211-243 / 第 211-243 行

```cpp
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
  bool benchmark;
  std::string tag;

  Options():
    help(false),
    input_size(1, 32, 32, 32),
    filter_size(32, 3, 3, 32),
    padding(1, 1, 1, 1),
    conv_stride(1, 1),
    dilation(1, 1),
    reference_check(false),
    measure_performance(true),
    iterations(20),
    save_workspace(false),
    alpha(1),
    beta(0),
    benchmark(false) { }

```

**EN:** `Options` stores command-line state: tensor/filter sizes, padding, stride, dilation, verification/performance flags, iteration count, workspace dumping, epilogue scalars, benchmark mode, and a CSV tag. The constructor sets defaults.

**CN:** `Options` 保存命令行状态：张量/滤波器大小、padding、stride、dilation、验证/性能开关、迭代次数、工作区导出、尾声标量、benchmark 模式和 CSV 标签。构造函数设置默认值。

### Lines 244-282 / 第 244-282 行

```cpp
  // Verify the problem size is compatible with the CUTLASS Convolution implementation.
  bool valid() {

    //
    // CUTLASS attempts to load 128b vectors of int4b_t elements. Consequently,
    // all pointers, strides, and tensor extents must be divisible by 32 elements.
    //
    int const kAlignment = 32;

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

  /// Updates input and filter sizes
  void update(
    cutlass::Tensor4DCoord input_size,
    cutlass::Tensor4DCoord filter_size) {

    this->input_size = input_size;
    this->filter_size = filter_size;

    padding.n() = filter_size.h() / 2;
    padding.h() = filter_size.h() / 2;
    padding.w() = filter_size.w() / 2;
    padding.c() = filter_size.w() / 2;
  }
```

**EN:** `valid` enforces int4 vectorization constraints and symmetric padding matching the filter radius. `update` replaces problem sizes and recomputes padding for benchmark layers.

**CN:** `valid` 强制 int4 向量化约束，并要求 padding 与滤波器半径匹配。`update` 替换问题大小，并为基准层重新计算 padding。

### Lines 284-363 / 第 284-363 行

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

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << "09_turing_tensorop_conv2dfprop example\n\n"
      << "  This example uses Turing's Tensor Core operators on int4 data types to compute\n"
      << "  forward convolution on tensors of layout NHWC.\n\n"
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
      << "  --ref-check          If set (true), reference check on the host is computed\n"
      << "  --perf-check         If set (true), performance is measured.\n"
      << "  --benchmark          If set (true), performance benchmarking on several layers and batch-size.\n"
      << "  --iterations=<int>   Number of profiling iterations to perform.\n"
      << "  --save-workspace     If set, workspace is written to a text file.\n"
      << "  --tag=<string>       String to replicate across the first column in the results table\n";

    out << "\n\nExamples:\n\n"
      << "$ ./examples/09_turing_tensorop_conv2dfprop/09_turing_tensorop_conv2dfprop  --n=32 --h=224 --w=224 --c=128 --k=256 --r=1 --s=1\n\n"
      << "$ ./examples/09_turing_tensorop_conv2dfprop/09_turing_tensorop_conv2dfprop  --n=1 --h=224 --w=224 --c=32 --k=32 --r=3 --s=3 --ref-check\n\n";

    return out;
  }
```

**EN:** `parse` reads flags and dimensions, links filter C to input C, reads alpha/beta/iterations/tag, and normalizes unsupported filters to 1x1. `print_usage` emits help text and example invocations.

**CN:** `parse` 读取标志和维度，把滤波器 C 绑定到输入 C，读取 alpha/beta/iterations/tag，并把不支持的滤波器规范化为 1x1。`print_usage` 输出帮助文本和示例命令。

### Lines 365-383 / 第 365-383 行

```cpp
  /// Computes the output tensor size (NPQK)
  cutlass::Tensor4DCoord output_size() const {
    return cutlass::Tensor4DCoord(
      input_size.n(),
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

**EN:** `output_size` computes NPQK from input, padding, filter, stride, and K. `gflops` counts `NPQK * CRS` FMAs and converts average runtime to GFLOP/s.

**CN:** `output_size` 根据输入、padding、滤波器、stride 和 K 计算 NPQK。`gflops` 统计 `NPQK * CRS` 个 FMA，并把平均运行时间换算为 GFLOP/s。

### Lines 387-432 / 第 387-432 行

```cpp
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

    out << "Layer,N,H,W,C,K,R,S,Runtime,GFLOPs";

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
      << runtime_ms << ","
      << gflops;

    return out;
  }
};
```

**EN:** `Result` stores runtime, throughput, CUTLASS status, reference status, and CUDA error. Its print helpers emit CSV headers and one row per profiled layer.

**CN:** `Result` 保存运行时间、吞吐、CUTLASS 状态、参考状态和 CUDA 错误。打印辅助函数输出 CSV 表头和每个层的一行结果。

### Lines 436-482 / 第 436-482 行

```cpp
/// Runs one benchmark
Result profile_convolution(Options const &options) {

  Result result;

  //
  // Allocate host-device tensors using the CUTLASS Utilities.
  //

  cutlass::HostTensor<ElementInputA, LayoutInputA> tensor_a(options.input_size);
  cutlass::HostTensor<ElementInputB, LayoutInputB> tensor_b(options.filter_size);
  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_c(options.output_size());
  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_ref_c(options.output_size());

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

  // Fill tensor C on host with zeros
  cutlass::reference::host::TensorFill(
      tensor_c.host_view());

  // Fill tensor C for reference on host with zeros
  cutlass::reference::host::TensorFill(
      tensor_ref_c.host_view());

  // Copy data from host to GPU
  tensor_a.sync_device();
  tensor_b.sync_device();
  tensor_c.sync_device();
  tensor_ref_c.sync_device();
```

**EN:** `profile_convolution` allocates host/device tensors for input, filter, output, and reference output. It initializes int4 inputs with random values, zeros outputs, and synchronizes tensors to the device.

**CN:** `profile_convolution` 为输入、滤波器、输出和参考输出分配主机/设备张量。它用随机值初始化 int4 输入，清零输出，并把张量同步到设备。

### Lines 484-538 / 第 484-538 行

```cpp
  //
  // Define arguments for CUTLASS Convolution
  //

  // mode (kCrossCorrelation or kConvolution)
  cutlass::conv::Mode mode = cutlass::conv::Mode::kCrossCorrelation;

  // Split K dimension into 1 partitions
  int split_k_slices = 1;

  // Construct Conv2dProblemSize with user defined output size
  cutlass::conv::Conv2dProblemSize problem_size(      
      options.input_size,
      options.filter_size,
      options.padding,
      options.conv_stride,
      options.dilation,
      options.output_size(),
      mode,
      split_k_slices);

  // Construct ImplicitGemm::Argument structure with conv2d 
  // problem size, data pointers, and epilogue values
  typename ImplicitGemm::Arguments arguments{
    problem_size,
    tensor_a.device_ref(),
    tensor_b.device_ref(),
    tensor_c.device_ref(),
    tensor_c.device_ref(),
    {options.alpha, options.beta},
  };

  //
  // Initialize CUTLASS Convolution
  //

  ImplicitGemm implicit_gemm_op;

  size_t workspace_size = implicit_gemm_op.get_workspace_size(arguments);

  // Allocate workspace memory
  cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);

  result.status = implicit_gemm_op.can_implement(arguments);
  CUTLASS_CHECK(result.status);

  result.status = implicit_gemm_op.initialize(arguments, workspace.get());
  CUTLASS_CHECK(result.status);

  //
  // Launch initialized CUTLASS kernel
  //
  result.status = implicit_gemm_op();

  CUTLASS_CHECK(result.status);
```

**EN:** The convolution is set to cross-correlation with one split-K slice. `Conv2dProblemSize` and `ImplicitGemm::Arguments` package geometry, tensor refs, and epilogue scalars. Workspace is allocated, support is checked, the operator is initialized, and the kernel is launched.

**CN:** 卷积模式设为 cross-correlation，split-K 为一个切片。`Conv2dProblemSize` 和 `ImplicitGemm::Arguments` 打包几何、张量引用和尾声标量。随后分配工作空间、检查支持性、初始化算子并启动内核。

### Lines 540-612 / 第 540-612 行

```cpp
  //
  // Optional reference check
  //
  
  if (options.reference_check) {
    std::cout << "Verification on host...\n";

    // Compute with reference implementation
    cutlass::reference::host::Conv2dFprop<
      ElementInputA,
      LayoutInputA,
      ElementInputB,
      LayoutInputB,
      ElementOutput,
      LayoutOutput,
      ElementComputeEpilogue,
      ElementAccumulator,
      ElementOutput,
      cutlass::NumericConverterClamp<ElementOutput, ElementComputeEpilogue>
    >(
      problem_size,
      tensor_a.host_ref(),
      tensor_b.host_ref(),
      tensor_c.host_ref(),
      tensor_ref_c.host_ref(),
      options.alpha,
      options.beta
    );

    // Check if output from CUTLASS kernel and reference kernel are equal or not
    tensor_c.sync_host();

    bool passed = cutlass::reference::host::TensorEquals(
      tensor_c.host_view(),
      tensor_ref_c.host_view());

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

  if (options.save_workspace) {

    std::stringstream ss;

    ss << "09_tensor_conv_workspace_conv2dfprop_"
      << options.input_size.n() << "x" << options.input_size.h() << "x" << options.input_size.w() << "x" << options.input_size.c() 
      << "_"
      << options.filter_size.n() << "x" << options.filter_size.h() << "x" << options.filter_size.w() << "x" << options.filter_size.c() 
      << ".dat";

    std::ofstream output_workspace(ss.str());

    output_workspace 
      << "Input = \n" << tensor_a.host_view() << "\n\n"
      << "Filters = \n" << tensor_b.host_view() << "\n\n";

    if (options.reference_check) {
      output_workspace << "Reference = \n" << tensor_ref_c.host_view() << "\n\n";
    }

    output_workspace << "Computed = \n" << tensor_c.host_view() << std::endl;

    std::cout << "Results written to '" << ss.str() << "'." << std::endl;
  }
```

**EN:** When requested, a host reference convolution computes expected output using the same problem and clamped conversion. The CUTLASS result is copied back and compared. Optional workspace saving writes input, filters, reference, and computed tensors to a named `.dat` file.

**CN:** 如请求验证，主机参考卷积用同一问题和钳位转换计算期望输出。CUTLASS 结果拷回并比较。可选工作区保存会把输入、滤波器、参考和计算张量写入命名的 `.dat` 文件。

### Lines 614-676 / 第 614-676 行

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
      result.status = implicit_gemm_op();
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
}
```

**EN:** Performance measurement creates CUDA events, times repeated launches of the initialized implicit GEMM operator, synchronizes, computes average milliseconds and GFLOP/s, destroys events, and returns the result.

**CN:** 性能测量创建 CUDA event，对已初始化的 implicit GEMM 算子进行多次启动计时，同步后计算平均毫秒和 GFLOP/s，销毁 event 并返回结果。

### Lines 679-706 / 第 679-706 行

```cpp
int main(int argc, char const **args) {

  // Turing Tensor Core operations exposed with mma.sync are first available in CUDA 10.2.
  //
  // CUTLASS must be compiled with CUDA 10.2 Toolkit to run these examples.
  if (!(__CUDACC_VER_MAJOR__ > 10 || (__CUDACC_VER_MAJOR__ == 10 && __CUDACC_VER_MINOR__ >= 2))) {
    std::cerr << "Turing Tensor Core operations must be compiled with CUDA 10.2 Toolkit or later." << std::endl;
    return 0;
  }

  cudaDeviceProp props;
  CUDA_CHECK(cudaGetDeviceProperties(&props, 0));

  if (!(props.major > 7 || (props.major == 7 && props.minor >= 5))) {
    std::cerr << "Turing Tensor Ops must be run on a machine with compute capability at least 75."
              << std::endl;
    return 0;
  }

  Options options;
  
  options.parse(argc, args);

  if (options.help) {
    options.print_usage(std::cout) << std::endl;
    return 0;
  }

```

**EN:** `main` checks CUDA 10.2+ and compute capability 7.5+, parses options, prints usage when requested, and exits early for unsupported environments.

**CN:** `main` 检查 CUDA 10.2+ 和计算能力 7.5+，解析选项，在请求帮助时打印用法，并在不支持环境中提前退出。

### Lines 707-749 / 第 707-749 行

```cpp
  if (options.benchmark) {
    // Benchmark several layers

    int batch_sizes[] = {1, 32, 64, 128, 256, 512};

    struct Benchmark {
      int h, w, c, k, r, s;
    } layers[] = {
      {56,  56,   64,   256, 1, 1},
      {56,  56,   64,    64, 1, 1},
      {56,  56,   64,    64, 3, 3},
      {56,  56,  256,    64, 1, 1},
      {56,  56,  256,   512, 1, 1},
      {56,  56,  256,   128, 1, 1},
      {28,  28,  128,   128, 3, 3},
      {28,  28,  128,   512, 1, 1},
      {28,  28,  512,   128, 1, 1},
      {28,  28,  512,  1024, 1, 1},
      {28,  28,  512,   256, 1, 1},
      {14,  14,  256,   256, 3, 3},
      {14,  14,  256,  1024, 1, 1},
      {14,  14,  1024,  256, 1, 1},
      {14,  14,  1024, 2048, 1, 1},
      {14,  14,  1024,  512, 1, 1},
      {7,    7,   512,  512, 3, 3},
    };

    Result::print_header(std::cout, options) << std::endl;

    int idx = 1;

    for (auto const &layer : layers) {
      for (auto N : batch_sizes) {

        options.update({N, layer.h, layer.w, layer.c}, {layer.k, layer.r, layer.s, layer.c});

        Result result = profile_convolution(options);
        result.print(std::cout, idx, options) << std::endl;
      }

      ++idx;
    }
  }
```

**EN:** Benchmark mode defines batch sizes and representative convolution layers, prints a CSV header, updates options for every layer/batch combination, profiles each case, and prints one row per result.

**CN:** Benchmark 模式定义批大小和代表性卷积层，打印 CSV 表头，为每个层/批组合更新选项、进行 profiling，并为每个结果打印一行。

### Lines 750-766 / 第 750-766 行

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

**EN:** Single-problem mode validates the requested geometry, profiles one convolution, prints the CSV header and result row, and returns success.

**CN:** 单问题模式验证请求的几何形状，profile 一次卷积，打印 CSV 表头和结果行，并返回成功。

## Key Concepts / 关键概念

- **EN:** Implicit GEMM maps convolution to a GEMM-like problem; output NPQK acts like M/N dimensions while CRS contributes to the reduction dimension.
  **CN:** Implicit GEMM 把卷积映射为类似 GEMM 的问题；输出 NPQK 对应 M/N 维，CRS 贡献归约维。

- **EN:** `DefaultConv2dFprop` builds a convolution kernel from element types, NHWC layouts, accumulator type, Tensor Core operator class, `Sm75`, tile shapes, epilogue, swizzle, pipeline stages, multiply-add operation, and iterator algorithm.
  **CN:** `DefaultConv2dFprop` 根据元素类型、NHWC 布局、累加器类型、Tensor Core 算子类别、`Sm75`、tile 形状、尾声、swizzle、流水级数、乘加操作和迭代器算法构建卷积内核。

- **EN:** The shape hierarchy is `128x128x128` threadblock, `64x64x128` warp, and `8x8x32` instruction, matching Turing int4 Tensor Core operations.
  **CN:** 形状层级为 `128x128x128` threadblock、`64x64x128` warp 和 `8x8x32` 指令，匹配 Turing int4 Tensor Core 操作。

- **EN:** `LinearCombinationClamp` forms the epilogue and clamps converted results back to the narrow int4 output type.
  **CN:** `LinearCombinationClamp` 构成尾声，并把转换后的结果钳位回较窄的 int4 输出类型。

- **EN:** All tensors use `TensorNHWC`; the sample requires channel/filter alignment to 32 int4 elements for 128-bit vectorized accesses.
  **CN:** 所有张量使用 `TensorNHWC`；示例要求通道/滤波器维按 32 个 int4 元素对齐，以便 128-bit 向量化访问。

## Dependencies / 依赖项

- **EN:** CUTLASS convolution kernel/device headers, GEMM shape definitions, command line parser, host tensors, reference convolution, tensor I/O, and helper macros.
  **CN:** 依赖 CUTLASS 卷积 kernel/device 头、GEMM 形状定义、命令行解析器、主机张量、参考卷积、张量 I/O 和辅助宏。

- **EN:** CUDA runtime events are used for timing; CUDA 10.2+ and compute capability 7.5+ are required.
  **CN:** 使用 CUDA 运行时 event 计时；需要 CUDA 10.2+ 和计算能力 7.5+。

- **EN:** Optional workspace saving writes a `.dat` file in the current working directory when requested.
  **CN:** 请求保存工作区时，会在当前工作目录写出 `.dat` 文件。
