# gemm_bias_relu.cu — Code Analysis / 代码分析

## Source / 源文件

`examples/12_gemm_bias_relu/gemm_bias_relu.cu`

## Purpose / 用途

**EN:** Demonstrates a CUTLASS device GEMM that fuses bias addition and ReLU activation into the epilogue. The mainloop performs half-precision Tensor Core GEMM with float accumulation/output, and the epilogue stores max(0, alpha*A*B + bias).

**CN:** 演示 CUTLASS device GEMM 在 epilogue 中融合 bias 加法和 ReLU 激活。主循环执行半精度 Tensor Core GEMM，float 累加/输出，epilogue 写回 max(0, alpha*A*B + bias)。

## Line-by-Line Analysis / 逐行分析

### Lines 1-48: license, includes, and helper headers / 许可证、头文件与辅助工具

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
*/

#include <algorithm>
#include <iostream>

#include "cutlass/cutlass.h"
#include "cutlass/gemm/device/gemm.h"
#include "cutlass/epilogue/thread/linear_combination_relu.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/reference/device/gemm.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/tensor_view_io.h"
#include "helper.h"

```

**EN:** The file begins with the standard license. Includes provide standard max/output streams, the CUTLASS device GEMM template, the `LinearCombinationRelu` epilogue, HostTensor utilities, reference GEMM/fill/compare helpers, tensor IO, and example macros from helper.h.

**CN:** 文件以标准许可证开头。头文件提供标准 max/输出流、CUTLASS device GEMM 模板、`LinearCombinationRelu` epilogue、HostTensor 工具、参考 GEMM/填充/比较工具、张量 IO，以及 helper.h 中的示例宏。

### Lines 49-71: element types, layouts, and bias orientation / 元素类型、布局与 bias 方向

```cpp
// The code section below describes datatype for input, output matrices and computation between
// elements in input matrices.
using ElementAccumulator = float;                   // <- data type of accumulator
using ElementComputeEpilogue = ElementAccumulator;  // <- data type of epilogue operations
using ElementInputA = cutlass::half_t;              // <- data type of elements in input matrix A
using ElementInputB = cutlass::half_t;              // <- data type of elements in input matrix B
using ElementOutput = float;                        // <- data type of elements in output matrix D

// Note that if the output is column major, the bias has to be per row. i.e. every row has different bias.
// If the output is row major, the bias has to be per column, i.e. every column has different bias.
// Below list some other notices:
//
// Note this example only works for ColumnMajor output because
//   1) we only have row major epilogue.
//   2) we swap A and B if the output is column major then we can still use the
//      row major epilogue.
//   3) Mx1 bias vector becomes 1xM after the swapping/transposing.
//   4) we can use the existing OutputIterator to load 1xM bias vector.

using LayoutInputA = cutlass::layout::ColumnMajor;
using LayoutInputB = cutlass::layout::ColumnMajor;
using LayoutOutput = cutlass::layout::ColumnMajor;

```

**EN:** A and B are half precision, while accumulation, epilogue compute, and output are float. The comments explain bias orientation: for column-major output the bias is per row (M by 1). The example uses column-major A, B, and D while relying on epilogue iterator behavior to broadcast the bias.

**CN:** A 和 B 使用半精度，而累加、epilogue 计算和输出使用 float。注释说明 bias 方向：列主序输出时 bias 按行给出，即 M×1。示例使用列主序 A、B、D，并依赖 epilogue 迭代器行为广播 bias。

### Lines 72-88: Tensor Core architecture and tile shapes / Tensor Core 架构与 tile 形状

```cpp
// This code section describes whether you want to use tensor cores or regular SIMT cores on GPU SM
using MMAOp = cutlass::arch::OpClassTensorOp;

// This code section describes CUDA SM architecture number
using SmArch = cutlass::arch::Sm75;

// This code section describes the tile size a thread block will compute
using ShapeMMAThreadBlock =
    cutlass::gemm::GemmShape<128, 128, 32>;  // <- threadblock tile M = 128, N = 128, K = 32
// This code section describes tile size a warp will compute
using ShapeMMAWarp = cutlass::gemm::GemmShape<64, 64, 32>;  // <- warp tile M = 64, N = 64, K = 32 
// This code section describes the size of MMA op
using ShapeMMAOp = cutlass::gemm::GemmShape<16, 8, 8>;  // <- MMA Op tile M = 16, N = 8, K = 8

// This code section describes how threadblocks are scheduled on GPU
using SwizzleThreadBlock = cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<>;

```

**EN:** The kernel targets Turing SM75 Tensor Core operations. Threadblock, warp, and instruction shapes define the generated GEMM tiling hierarchy, and identity swizzle maps CTAs to tiles in default order.

**CN:** kernel 目标是 Turing SM75 Tensor Core。threadblock、warp 和指令形状定义生成 GEMM 的分块层次，identity swizzle 以默认顺序把 CTA 映射到 tile。

### Lines 89-103: fused `LinearCombinationRelu` epilogue / 融合 `LinearCombinationRelu` epilogue

```cpp
// Define the epilogue operation as LinearCombinationRelu. This is approximately equal to
//
//    d_ij = max(0, alpha * sum_k(a_ik * b_kj) + c_ij )
//
using EpilogueOp = cutlass::epilogue::thread::LinearCombinationRelu<
    ElementOutput,                                        // <- data type of output matrix
    128 / cutlass::sizeof_bits<ElementOutput>::value,     // <- this is the number of elements per
                                                          // vectorized memory access. For half
                                                          // precision, it's 8 elements. This becomes
                                                          // the vector width of math instructions in
                                                          // epilogue too
    ElementAccumulator,                                   // <- data type of accumulator
    ElementComputeEpilogue,                               // <- data type for alpha in linear combination function
    cutlass::epilogue::thread::ScaleType::NoBetaScaling>; // <- alpha x C + bias

```

**EN:** `LinearCombinationRelu` fuses scaling, bias addition, and ReLU during the store path. For float output, the vector length is 128/32 = 4 elements per vectorized access. `NoBetaScaling` means the source C operand is used as an additive bias rather than beta-scaled C.

**CN:** `LinearCombinationRelu` 在写回路径中融合缩放、bias 加法和 ReLU。float 输出时向量长度为 128/32=4 个元素。`NoBetaScaling` 表示源 C 操作数作为加性 bias 使用，而不是乘 beta 的 C。

### Lines 104-122: GEMM template parameters / GEMM 模板参数

```cpp
// Number of pipelines you want to use
constexpr int NumStages = 2;

using Gemm = cutlass::gemm::device::Gemm<ElementInputA,
                                         LayoutInputA,
                                         ElementInputB,
                                         LayoutInputB,
                                         ElementOutput,
                                         LayoutOutput,
                                         ElementAccumulator,
                                         MMAOp,
                                         SmArch,
                                         ShapeMMAThreadBlock,
                                         ShapeMMAWarp,
                                         ShapeMMAOp,
                                         EpilogueOp,
                                         SwizzleThreadBlock,
                                         NumStages>;

```

**EN:** The `Gemm` alias binds input/output element types and layouts, accumulator type, TensorOp instruction class, SM75 target, CTA/warp/MMA tile shapes, fused epilogue, threadblock swizzle, and two pipeline stages. These template parameters fully determine the generated kernel.

**CN:** `Gemm` 别名绑定输入/输出元素类型与布局、累加器类型、TensorOp 指令类别、SM75 目标、CTA/warp/MMA tile 形状、融合 epilogue、threadblock swizzle 和两级流水线。这些模板参数完整决定生成的 kernel。

### Lines 123-147: problem size and tensor allocation / 问题尺寸与张量分配

```cpp
int run() {

  const int length_m = 5120;
  const int length_n = 4096;
  const int length_k = 4096;

  // Create a tuple of problem size for matrix multiplication
  cutlass::gemm::GemmCoord problem_size(length_m, length_n, length_k);

  // Initialize tensors using CUTLASS helper functions
  cutlass::HostTensor<ElementInputA, LayoutInputA> tensor_a(
      problem_size.mk());  // <- Create matrix A with dimensions M x K
  cutlass::HostTensor<ElementInputB, LayoutInputB> tensor_b(
      problem_size.kn());  // <- Create matrix B with dimensions K x N

  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_c_bias(
      {problem_size.m(), 1});  // <- Create matrix C with dimensions M x 1

  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_d(
      problem_size.mn());  // <- Create matrix D with dimensions M x N used to store output from
                           // CUTLASS kernel
  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_ref_d(
      problem_size.mn());  // <- Create matrix D with dimensions M x N used to store output from
                           // reference kernel

```

**EN:** run() defines a 5120x4096x4096 GEMM. HostTensor allocates synchronized host/device storage for A (M by K), B (K by N), row bias (M by 1), D, and reference D.

**CN:** run() 定义 5120×4096×4096 GEMM。HostTensor 为 A (M×K)、B (K×N)、按行 bias (M×1)、D 和参考 D 分配可同步的主机/设备存储。

### Lines 148-178: random fill and host-to-device sync / 随机填充与同步到设备

```cpp
  // Fill input and output matrices on host using CUTLASS helper functions
  cutlass::reference::host::TensorFillRandomUniform(
      tensor_a.host_view(),
      1,
      ElementInputA(4),
      ElementInputA(-4),
      0);  // <- Fill matrix A on host with uniform-distribution random data
  cutlass::reference::host::TensorFillRandomUniform(
      tensor_b.host_view(),
      1,
      ElementInputB(4),
      ElementInputB(-4),
      0);  // <- Fill matrix B on host with uniform-distribution random data
  cutlass::reference::host::TensorFillRandomUniform(
      tensor_c_bias.host_view(),
      1,
      ElementOutput(4),
      ElementOutput(-4),
      0);  // <- Fill matrix C on host with uniform-distribution random data
  cutlass::reference::host::TensorFill(
      tensor_d.host_view());  // <- fill matrix D on host with zeros
  cutlass::reference::host::TensorFill(
      tensor_ref_d.host_view());  // <- fill matrix D for reference on host with zeros

  // Copy data from host to GPU
  tensor_a.sync_device();
  tensor_b.sync_device();
  tensor_c_bias.sync_device();
  tensor_d.sync_device();
  tensor_ref_d.sync_device();

```

**EN:** A, B, and bias are filled with bounded random values; D and reference D are zero-filled. All tensors are synchronized to device memory before launching kernels.

**CN:** A、B 和 bias 用有界随机值填充；D 和参考 D 填零。启动 kernel 前所有张量都同步到设备内存。

### Lines 179-219: arguments, bias broadcasting, and launch / 参数、bias 广播与启动

```cpp
  // Initialize alpha for dot product computation
  ElementComputeEpilogue alpha = ElementComputeEpilogue(1);

  // Split K dimension into 1 partitions
  int split_k_slices = 1;

  // Create a tuple of gemm kernel arguments. This is later passed as arguments to launch
  // instantiated CUTLASS kernel
  typename Gemm::Arguments arguments{
    problem_size,                       // <- problem size of matrix multiplication
    tensor_a.device_ref(),              // <- reference to matrix A on device
    tensor_b.device_ref(),              // <- reference to matrix B on device

    {tensor_c_bias.device_data(), 0},   // <- the C matrix is treated as the bias vector. We can enable the GEMM
                                        //    to project away the N dimension by setting the stride to zero.

    tensor_d.device_ref(),              // <- reference to matrix D on device
    {alpha},                              // <- alpha
    split_k_slices};                    // <- k-dimension split factor

  // Using the arguments, query for extra workspace required for matrix multiplication computation
  size_t workspace_size = Gemm::get_workspace_size(arguments);

  // Allocate workspace memory
  cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);

  // Instantiate CUTLASS kernel depending on templates
  Gemm gemm_op;

  // Check the problem size is supported or not 
  cutlass::Status status = gemm_op.can_implement(arguments);
  CUTLASS_CHECK(status);

  // Initialize CUTLASS kernel with arguments and workspace pointer
  status = gemm_op.initialize(arguments, workspace.get());
  CUTLASS_CHECK(status);

  // Launch initialized CUTLASS kernel
  status = gemm_op();
  CUTLASS_CHECK(status);

```

**EN:** alpha is one and split-K is disabled. The C argument is replaced by `{tensor_c_bias.device_data(), 0}`; the zero stride projects/broadcasts the Mx1 bias across the N dimension. The code then queries workspace, checks `can_implement`, initializes, and launches the GEMM.

**CN:** alpha 为 1，split-K 被禁用。C 参数被 `{tensor_c_bias.device_data(), 0}` 替代；零 stride 将 M×1 bias 沿 N 维投影/广播。随后代码查询 workspace，检查 `can_implement`，初始化并启动 GEMM。

### Lines 220-242: reference product GEMM / 参考乘积 GEMM

```cpp
  //
  // Create instantiation for device reference gemm kernel
  //

  cutlass::reference::device::Gemm<ElementInputA,
                                   LayoutInputA,
                                   ElementInputB,
                                   LayoutInputB,
                                   ElementOutput,
                                   LayoutOutput,
                                   ElementComputeEpilogue,
                                   ElementComputeEpilogue>
      gemm_device_reference;

  // Launch device reference to compute strictly the product A * B
  gemm_device_reference(
    problem_size,
    alpha,
    tensor_a.device_ref(),
    tensor_b.device_ref(),
    0,
    tensor_ref_d.device_ref());

```

**EN:** A device reference GEMM computes only the product term with beta set to zero. Bias and ReLU are intentionally applied later on the host so the final reference matches the fused epilogue.

**CN:** 设备端参考 GEMM 只计算乘积项，并将 beta 设为 0。bias 和 ReLU 有意稍后在主机端应用，使最终参考结果与融合 epilogue 匹配。

### Lines 243-270: host-side bias/ReLU and comparison / 主机端 bias/ReLU 与比较

```cpp
  // Wait for kernels to finish
  cudaDeviceSynchronize();

  // Copy output data from CUTLASS and reference kernel to host for comparison
  tensor_d.sync_host();
  tensor_ref_d.sync_host();

  // Compute bias + relu in host code
  for (int i = 0; i < problem_size.m(); ++i) {
    for (int j = 0; j < problem_size.n(); ++j) {
      tensor_ref_d.at({i, j}) = std::max(
        ElementOutput(0), 
        ElementOutput(tensor_ref_d.at({i, j}) + tensor_c_bias.at({i, 0}))
      );
    }
  }

  // Check if output from CUTLASS kernel and reference kernel are equal or not
  std::cout << (cutlass::reference::host::TensorEquals(tensor_d.host_view(),
                                                       tensor_ref_d.host_view())
                    ? "Passed"
                    : "Failed")
            << std::endl;

  CUTLASS_CHECK(status);
  return 0;
}

```

**EN:** After synchronization and copy-back, nested loops add `tensor_c_bias(i,0)` to every output element in row i and clamp with `std::max(0, value)`. The host comparison prints Passed or Failed.

**CN:** 同步并回拷后，双重循环把 `tensor_c_bias(i,0)` 加到第 i 行的每个输出元素，并用 `std::max(0, value)` 截断。主机端比较打印 Passed 或 Failed。

### Lines 271-303: main guards and execution / main 检查与执行

```cpp
int main() {

  bool notSupported = false;

  // Turing Tensor Core operations exposed with mma.sync are first available in CUDA 10.2.
  //
  // CUTLASS must be compiled with CUDA 10.1 Toolkit to run these examples.
  if (!(__CUDACC_VER_MAJOR__ > 10 || (__CUDACC_VER_MAJOR__ == 10 && __CUDACC_VER_MINOR__ >= 2))) {
    std::cerr << "Turing Tensor Core operations must be compiled with CUDA 10.2 Toolkit or later." << std::endl;
    notSupported = true;
  }

  cudaDeviceProp props;

  cudaError_t error = cudaGetDeviceProperties(&props, 0);
  if (error != cudaSuccess) {
    std::cerr << "cudaGetDeviceProperties() returned an error: " << cudaGetErrorString(error) << std::endl;
    return -1;
  }

  if (!(props.major * 10 + props.minor >= 75)) {
    std::cerr << "Turing Tensor Ops must be run on a machine with compute capability at least 75."
              << std::endl;
    notSupported = true;
  }

  if (notSupported) {
    // Returning zero so this test passes on older Toolkits. Its actions are no-op.
    return 0;
  }

  return run();
}
```

**EN:** main() requires CUDA 10.2+ and SM75+ for the Turing Tensor Core path. Unsupported environments return 0 as no-op; supported environments call run().

**CN:** main() 要求 CUDA 10.2+ 和 SM75+ 以运行 Turing Tensor Core 路径。不支持的环境以 no-op 返回 0；支持时调用 run()。

## Key Concepts / 关键概念

**EN:**
- **CUTLASS epilogue fusion:** epilogues transform accumulators before storing, enabling bias and ReLU without extra kernels.
- **Bias as C operand:** the C tensor reference points at an Mx1 bias vector and uses stride zero for broadcasting.
- **`LinearCombinationRelu`:** computes scaled accumulator plus additive source, then clamps negative values.
- **`NoBetaScaling`:** treats C as bias, not as `beta*C`.
- **Template control:** element types, layouts, architecture, tile shapes, epilogue, swizzle, and stages specialize the generated GEMM.

**CN:**
- **CUTLASS epilogue 融合：** epilogue 在写回前变换累加器，因此可无需额外 kernel 融合 bias 和 ReLU。
- **把 bias 当作 C 操作数：** C 张量引用指向 M×1 bias 向量，并使用零 stride 广播。
- **`LinearCombinationRelu`：** 计算缩放后的累加器加上加性源项，再把负值截断。
- **`NoBetaScaling`：** 将 C 视为 bias，而不是 `beta*C`。
- **模板控制：** 元素类型、布局、架构、tile 形状、epilogue、swizzle 和 stages 对生成的 GEMM 进行特化。

## Dependencies / 依赖项

**EN:** CUDA runtime, CUTLASS device GEMM, `cutlass/epilogue/thread/linear_combination_relu.h`, HostTensor utilities, reference device/host helpers, helper.h, CUDA 10.2+, and SM75+ hardware.

**CN:** 依赖 CUDA runtime、CUTLASS device GEMM、`cutlass/epilogue/thread/linear_combination_relu.h`、HostTensor 工具、设备端/主机端参考工具、helper.h、CUDA 10.2+ 和 SM75+ 硬件。
