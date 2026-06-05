# ampere_3xtf32_tensorop_symm.cu — Code Analysis / 代码分析
**Source / 源文件**: `examples/33_ampere_3xtf32_tensorop_symm/ampere_3xtf32_tensorop_symm.cu`
**Purpose / 用途**: Demonstrates Ampere SM80 symmetric matrix multiply with CUTLASS `device::Symm` in 3xTF32, 1xTF32, and FP64 reference modes, then compares numerical error against FP64 and optionally cuBLAS / 演示在 Ampere SM80 上用 CUTLASS `device::Symm` 运行 3xTF32、1xTF32 与 FP64 参考 SYMM，并与 FP64 及可选的 cuBLAS 做误差比较
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-59: File header and 3xTF32 idea / 文件头与 3xTF32 思路
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
NVIDIA Ampere architecture starts supporting tfloat32 (see include/cutlass/tfloat32.h)
data types in tensor cores.  One big advantage is that we can load in F32 data and convert them
implicitly to tf32 inside the SYMM kernel which means no change is needed to accelerate traditional
F32 data by using NVIDIA Ampere architecture.

We can use the tf32 mode of tensor core to emulate a fast accurate SYMM kernel which is accelerated
using Ampere Tensor Cores (see include/cutlass/gemm/warp/mma_tensor_op_fast_f32.h).

The trick is very simple
  a x b = (a_big + a_small) x (b_big + b_small) = a_big x b_big + a_big x b_small + a_small x b_big
  big = convert_to_tf32(F32)
  small = convert_to_tf32(F32 - big)

a_small x b_small is discarded because they are too small.

This example demonstrates usage of this kernel, along with accuracy measurements w.r.t. actual F32
results (SSYMM from cuBLAS) and against F64 results (DSYMM from CUTLASS)

To enable this feature, the only change needs to make is to change the default OpMultiplyAdd to
OpMultiplyAddFastF32.

Now, we have two different flavors of SSYMM in the profiler for Ampere:

  s1688symm       // Use 3xTF32 to emulate F32.  F32 in, converted in TF32-big and TF32-small internally,
                  // accumulated in F32, F32 out.
  s1688tf32symm   // Use 1xTF32.  F32 in, converted to one TF32 internally, accumulated in F32, F32 out.
*/
```
**EN**: The opening comment is the conceptual core of the example. It explains why `OpMultiplyAddFastF32` exists: CUTLASS decomposes each FP32 operand into a large TF32 part and a residual TF32 part, then performs three TF32 MMA terms to approximate FP32 more accurately than a single TF32 multiply-accumulate.
**CN**: 开头注释给出了示例的核心思想。它解释了 `OpMultiplyAddFastF32` 的意义：CUTLASS 将每个 FP32 操作数拆成较大的 TF32 部分和残差 TF32 部分，再执行三个 TF32 MMA 项，以比单次 TF32 乘加更接近 FP32 的结果。

### Lines 61-84: Includes / 头文件
```cpp
#include <iostream>
#include <vector>
#include <limits>

#include "cutlass/blas3.h"
#include "cutlass/gemm/device/symm.h"

#include "cutlass/util/command_line.h"
#include "cutlass/util/host_tensor.h"

#include "cutlass/util/reference/host/symm.h"
#include "cutlass/util/reference/host/tensor_reduce.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_norm.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/error_metrics.h"
#include "cutlass/util/tensor_view_io.h"

#include "helper.h"

#if CUTLASS_ENABLE_CUBLAS
#include <cublas_v2.h>
#endif
```
**EN**: The file depends on CUTLASS BLAS-3 SYMM, `HostTensor` utilities, host-side fill/copy/error helpers, and optional cuBLAS. This mix shows the example’s structure: allocate tensors, launch device kernels, then validate or compare results on the host.
**CN**: 本文件依赖 CUTLASS 的 BLAS-3 SYMM、`HostTensor` 工具、主机端填充/拷贝/误差计算辅助函数，以及可选的 cuBLAS。这个组合体现了示例结构：分配张量、启动设备 kernel，再在主机侧做校验或对比。

### Lines 88-123: `Options` data members and validation / `Options` 数据成员与校验
```cpp
struct Options {

  bool help;

  cutlass::gemm::GemmCoord problem_size;
  float alpha;
  float beta;
  std::string rand_mode;
  int seed;

  Options():
    help(false),
    problem_size({4096, 4096, 4096}),
    seed(1),
    alpha(1),
    beta(),
    rand_mode("uniform") { }

  bool valid() {
    //
    // CUTLASS attempts to load 128b vectors of F32 elements. Consequently,
    // all pointers, strides, and tensor extents must be divisible by 4 elements.
    //
    int const kAlignment = 4;

    if ((problem_size.m() % kAlignment) ||
      (problem_size.n() % kAlignment) ||
      (problem_size.k() % kAlignment)) {

      // misaligned tensors
      return false;
    }

    return true;
  }
```
**EN**: `Options` stores the GEMM-like SYMM shape `(M,N,K)`, scaling factors, RNG mode, and seed. The `valid()` rule reflects CUTLASS vectorized memory accesses: FP32 uses 128-bit accesses, so dimensions must be multiples of 4 elements.
**CN**: `Options` 保存类 GEMM 的 SYMM 尺寸 `(M,N,K)`、缩放系数、随机初始化模式和种子。`valid()` 体现了 CUTLASS 的向量化访存约束：FP32 以 128 位访问，因此维度需要是 4 个元素的倍数。

### Lines 125-166: Command-line parsing and usage / 命令行解析与帮助信息
```cpp
  // Parses the command line
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);

    if (cmd.check_cmd_line_flag("help")) {
      help = true;
    }

    cmd.get_cmd_line_argument("m", problem_size.m());
    cmd.get_cmd_line_argument("n", problem_size.n());
    // Since the kernels in this example are in Left Side Mode
    cmd.get_cmd_line_argument("m", problem_size.k());

    cmd.get_cmd_line_argument("alpha", alpha);
    cmd.get_cmd_line_argument("beta", beta);

    cmd.get_cmd_line_argument("seed", seed);
    cmd.get_cmd_line_argument("rand_mode", rand_mode);

  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << "33_ampere_3xtf32_tensorop_symm example\n\n"
      << "  This example uses the CUTLASS Library to execute 3xTF32 tensorop SYMM computations.\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement.\n\n"
      << "  --m=<int>                   SYMM M dimension\n"
      << "  --n=<int>                   SYMM N dimension\n"
      << "  --alpha=<f32>               Epilogue scalar alpha\n"
      << "  --beta=<f32>                Epilogue scalar beta\n\n"
      << "  --rand_mode=<string>        gauss / uniform*\n\n"
      << "  --seed=<int>                Random number seed (1*)\n\n";
```
**EN**: Parsing is intentionally SYMM-specific. Because `SideModeA` is `kLeft`, the symmetric matrix `A` is `M x M`, so the code reuses `m` for `k` instead of exposing an independent `k`. The usage text also makes clear that `alpha` and `beta` are epilogue scalars applied as `D = alpha * AB + beta * C`.
**CN**: 这里的解析逻辑是针对 SYMM 定制的。由于 `SideModeA` 为 `kLeft`，对称矩阵 `A` 的形状是 `M x M`，因此代码直接让 `k` 复用 `m`，而不是单独暴露一个 `k` 参数。帮助信息也说明了 `alpha` 与 `beta` 是 epilogue 标量，对应 `D = alpha * AB + beta * C`。

### Lines 170-211: Layout, architecture, tile, and epilogue configuration / 布局、架构、分块与 epilogue 配置
```cpp
// The code section below describes matrix layout of input and output matrices. Column Major for
// Matrix A, Matrix B and Matrix C (since that's what cuBLAS supports, CUTLASS supports Row Major too)
using LayoutInputA = cutlass::layout::ColumnMajor;
using LayoutInputB = cutlass::layout::ColumnMajor;
using LayoutOutput = cutlass::layout::ColumnMajor;

// Symmetric Matrix A is in Left Side mode
constexpr cutlass::SideMode SideModeA = cutlass::SideMode::kLeft;
// Symmetric Matrix A is in Lower Filled mode
constexpr cutlass::FillMode FillModeA = cutlass::FillMode::kLower;

// This code section describes whether you want to use tensor cores or regular SIMT cores on GPU SM
using MMAOp = cutlass::arch::OpClassTensorOp;

// This code section describes CUDA SM architecture number
using SmArch = cutlass::arch::Sm80;

// This code section describes the tile size a thread block will compute
using ShapeMMAThreadBlock =
    cutlass::gemm::GemmShape<128, 64, 16>;  // <- threadblock tile M = 128, N = 128, K = 16
// This code section describes tile size a warp will compute
using ShapeMMAWarp = cutlass::gemm::GemmShape<64, 32, 16>;  // <- warp tile M = 64, N = 64, K = 16
// This code section describes the size of MMA op
using ShapeMMAOp = cutlass::gemm::GemmShape<16, 8, 8>;  // <- MMA Op tile M = 16, N = 8, K = 8

// This code section describes how threadblocks are scheduled on GPU
using SwizzleThreadBlock = cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<>;

// This code section describes the epilogue part of the kernel
using EpilogueOp = cutlass::epilogue::thread::LinearCombination<
    float,                                     // <- data type of output matrix
    128 / cutlass::sizeof_bits<float>::value,  // <- the number of elements per vectorized
    float,                                     // <- data type of accumulator
    float>;                                    // <- data type for alpha/beta in linear combination function

// Number of pipelines you want to use
constexpr int NumStages = 3;
// Alignment
constexpr int Alignment = 4;
```
**EN**: This is the CUTLASS kernel recipe. Column-major layouts are chosen to match cuBLAS. `SideModeA = kLeft` means the symmetric operand multiplies from the left; `FillModeA = kLower` means only the lower triangle is stored and CUTLASS reconstructs the symmetric view logically. The three `GemmShape` aliases define threadblock, warp, and instruction tile sizes, while `LinearCombination` describes the epilogue vector width, accumulator type, and scalar type.
**CN**: 这里给出了 CUTLASS kernel 的“配方”。列主序布局与 cuBLAS 对齐。`SideModeA = kLeft` 表示对称矩阵从左侧参与乘法；`FillModeA = kLower` 表示只存下三角，CUTLASS 会在逻辑上恢复完整对称视图。三个 `GemmShape` 分别定义 threadblock、warp 与单条 MMA 指令的 tile；`LinearCombination` 则规定 epilogue 的向量宽度、累加器类型和标量类型。

### Lines 217-239: `Symm_3xTF32` specialization / `Symm_3xTF32` 特化
```cpp
using Symm_3xTF32 = cutlass::gemm::device::Symm<
                                              float,
                                              LayoutInputA,
                                              SideModeA,
                                              FillModeA,
                                              float,
                                              LayoutInputB,
                                              float,
                                              LayoutOutput,
                                              float,
                                              MMAOp,
                                              SmArch,
                                              ShapeMMAThreadBlock,
                                              ShapeMMAWarp,
                                              ShapeMMAOp,
                                              EpilogueOp,
                                              SwizzleThreadBlock,
                                              NumStages,
                                              1, // Symmetric matrix is always align 1
                                              Alignment,
                                              false,
                                              cutlass::arch::OpMultiplyAddFastF32>;
```
**EN**: This alias binds `cutlass::gemm::device::Symm` to the fast-FP32 Tensor Core path. The template parameters, in order, specify A/B/C/accumulator element types, A/B/C layouts, side/fill mode for the symmetric operand, op class, architecture tag, block/warp/instruction shapes, epilogue functor, block swizzle, pipeline stages, A alignment, B alignment, split-K serial flag, and the inner math operator. The key choice is the last one: `OpMultiplyAddFastF32` activates the 3xTF32 decomposition.
**CN**: 这个别名把 `cutlass::gemm::device::Symm` 绑定到快速 FP32 Tensor Core 路径。模板参数按顺序指定 A/B/C/累加器元素类型、A/B/C 布局、对称矩阵的 side/fill 模式、运算类别、架构标签、threadblock/warp/instruction 形状、epilogue functor、threadblock swizzle、流水级数、A 对齐、B 对齐、是否串行 split-K，以及底层数学运算符。最关键的是最后一个参数：`OpMultiplyAddFastF32` 启用了 3xTF32 分解。

### Lines 241-263: `Symm_1xTF32` specialization / `Symm_1xTF32` 特化
```cpp
using Symm_1xTF32 = cutlass::gemm::device::Symm<
                                              float,
                                              LayoutInputA,
                                              SideModeA,
                                              FillModeA,
                                              float,
                                              LayoutInputB,
                                              float,
                                              LayoutOutput,
                                              float,
                                              MMAOp,
                                              SmArch,
                                              ShapeMMAThreadBlock,
                                              ShapeMMAWarp,
                                              ShapeMMAOp,
                                              EpilogueOp,
                                              SwizzleThreadBlock,
                                              NumStages,
                                              1, // Symmetric matrix is always align 1
                                              Alignment,
                                              false,
                                              cutlass::arch::OpMultiplyAdd>;
```
**EN**: This alias is identical except for the math operator. `OpMultiplyAdd` means the kernel uses the standard TF32 Tensor Core pipeline without the 3-term FP32 emulation. That makes it the direct comparison point for showing why `FastF32` improves accuracy.
**CN**: 这个别名除了数学运算符之外与前者相同。`OpMultiplyAdd` 表示 kernel 使用标准 TF32 Tensor Core 路径，而不做 3 项 FP32 模拟。因此它正好是展示 `FastF32` 精度收益的直接对照组。

### Lines 265-288: `Symm_F64` reference kernel / `Symm_F64` 参考 kernel
```cpp
using Symm_F64 = cutlass::gemm::device::Symm<
                                              double,
                                              LayoutInputA,
                                              SideModeA,
                                              FillModeA,
                                              double,
                                              LayoutInputB,
                                              double,
                                              LayoutOutput,
                                              double,
                                              cutlass::arch::OpClassTensorOp,
                                              cutlass::arch::Sm80,
                                              cutlass::gemm::GemmShape<32, 32, 16>,
                                              cutlass::gemm::GemmShape<16, 16, 16>,
                                              cutlass::gemm::GemmShape<8, 8, 4>,
                                              cutlass::epilogue::thread::LinearCombination<
                                                double,
                                                1,
                                                double,
                                                double
                                              >,
                                              cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<>,
                                              4>;
```
**EN**: The FP64 reference kernel uses a much smaller tile configuration and a scalar-width epilogue (`count = 1`). Its role is not performance parity but numerical reference quality, so the example launches it mainly to compute relative L2 error for the TF32-based outputs.
**CN**: FP64 参考 kernel 采用更小的 tile 配置，并使用标量宽度的 epilogue（`count = 1`）。它的目标不是追求同等性能，而是提供高精度参考结果，因此示例主要用它来计算基于 TF32 输出的相对 L2 误差。

### Lines 290-347: Create and initialize FP32 tensors / 创建并初始化 FP32 张量
```cpp
bool run(Options &options) {

  // Create a tuple of problem size for matrix multiplication
  cutlass::gemm::GemmCoord problem_size = options.problem_size;

  ////////////////////////////////////////////////////////////////////////////////
  /// 1. Initialize F32 Precision input tensors using CUTLASS helper functions
  ////////////////////////////////////////////////////////////////////////////////
  cutlass::HostTensor<float, LayoutInputA> tensor_a_F32(problem_size.mk());
  cutlass::HostTensor<float, LayoutInputB> tensor_b_F32(problem_size.kn());
  cutlass::HostTensor<float, LayoutOutput> tensor_c_F32(problem_size.mn());
  cutlass::HostTensor<float, LayoutOutput> tensor_d_F32(problem_size.mn());

  if (options.rand_mode == "uniform") {
    const float min = -1;
    const float max =  1;
    cutlass::reference::host::TensorFillRandomUniform(
        tensor_a_F32.host_view(),
        options.seed,
        double(max),
        double(min));
    cutlass::reference::host::TensorFillRandomUniform(
        tensor_b_F32.host_view(),
        options.seed,
        double(max),
        double(min));
    cutlass::reference::host::TensorFillRandomUniform(
        tensor_c_F32.host_view(),
        options.seed,
        double(max),
        double(min));
  } else if (options.rand_mode == "gauss") {
    cutlass::reference::host::TensorFillRandomGaussian(
        tensor_a_F32.host_view(),
        options.seed,
        double(0),
        double(5));
    cutlass::reference::host::TensorFillRandomGaussian(
        tensor_b_F32.host_view(),
        options.seed,
        double(0),
        double(5));
    cutlass::reference::host::TensorFillRandomGaussian(
        tensor_c_F32.host_view(),
        options.seed,
        double(0),
        double(5));
  }
  cutlass::reference::host::TensorFill(tensor_d_F32.host_view());

  tensor_a_F32.sync_device();
  tensor_b_F32.sync_device();
  tensor_c_F32.sync_device();
  tensor_d_F32.sync_device();
```
**EN**: `HostTensor` is a CUTLASS convenience wrapper that owns host and device storage plus synchronization helpers. `mk()`, `kn()`, and `mn()` materialize the expected matrix extents from `GemmCoord`. The example fills A/B/C on the host, zeroes D, then copies everything to device memory.
**CN**: `HostTensor` 是 CUTLASS 的便捷封装，统一管理主机/设备存储以及同步辅助函数。`mk()`、`kn()`、`mn()` 会从 `GemmCoord` 生成对应矩阵维度。示例先在主机端填充 A/B/C，将 D 清零，然后整体同步到设备端。

### Lines 349-388: Build FP64 and output comparison tensors / 构造 FP64 与对比输出张量
```cpp
  ////////////////////////////////////////////////////////////////////////////////
  /// 2. Initialize F64 tensors, Output tensors and setup arguments
  ////////////////////////////////////////////////////////////////////////////////
  cutlass::HostTensor<double, LayoutInputA> tensor_a_F64(problem_size.mk());
  cutlass::HostTensor<double, LayoutInputB> tensor_b_F64(problem_size.kn());
  cutlass::HostTensor<double, LayoutOutput> tensor_c_F64(problem_size.mn());

  cutlass::HostTensor<float, LayoutOutput> tensor_d_3xTF32(problem_size.mn());
  cutlass::HostTensor<float, LayoutOutput> tensor_d_1xTF32(problem_size.mn());
  cutlass::HostTensor<double, LayoutOutput> tensor_d_F64(problem_size.mn());
#if CUTLASS_ENABLE_CUBLAS
  cutlass::HostTensor<float, LayoutOutput> tensor_d_cublasF32(problem_size.mn());
#endif

  cutlass::reference::host::TensorCopy(tensor_a_F64.host_view(), tensor_a_F32.host_view());
  cutlass::reference::host::TensorCopy(tensor_b_F64.host_view(), tensor_b_F32.host_view());
  cutlass::reference::host::TensorCopy(tensor_c_F64.host_view(), tensor_c_F32.host_view());
  cutlass::reference::host::TensorCopy(tensor_d_F64.host_view(), tensor_d_F32.host_view());
  cutlass::reference::host::TensorCopy(tensor_d_3xTF32.host_view(), tensor_d_F32.host_view());
  cutlass::reference::host::TensorCopy(tensor_d_1xTF32.host_view(), tensor_d_F32.host_view());
#if CUTLASS_ENABLE_CUBLAS
  cutlass::reference::host::TensorCopy(tensor_d_cublasF32.host_view(), tensor_d_F32.host_view());
#endif

  tensor_a_F64.sync_device();
  tensor_b_F64.sync_device();
  tensor_c_F64.sync_device();
  tensor_d_F64.sync_device();
  tensor_d_3xTF32.sync_device();
  tensor_d_1xTF32.sync_device();
#if CUTLASS_ENABLE_CUBLAS
  tensor_d_cublasF32.sync_device();
#endif
```
**EN**: This block mirrors the FP32 data into FP64 so all kernels evaluate the same values. Separate D tensors are allocated for each backend, which avoids overwriting and makes later pairwise error computations straightforward.
**CN**: 这一段把 FP32 数据镜像成 FP64，保证所有 kernel 使用相同输入。每个后端都分配独立的 D 张量，避免相互覆盖，也让后续两两误差比较更直接。

### Lines 390-399: Scalars and batch-stride setup / 标量与 batch stride 设置
```cpp
  float alpha = float(options.alpha);
  float beta =  float(options.beta);

  // Batch count as 1
  int batch_count = 1;

  // Batch stride for A, when matrix A is in Left Side mode
  int batch_stride_A = problem_size.m()*problem_size.m();
```
**EN**: Although this is a single SYMM, CUTLASS uses a universal launcher interface that still expects batch metadata. `batch_stride_A` is `M*M` because the symmetric left operand is a square matrix, while the later B/C/D strides use `M*N` for dense rectangular outputs.
**CN**: 虽然这里只运行单个 SYMM，但 CUTLASS 采用通用 launcher 接口，因此仍然需要 batch 元数据。由于左侧对称矩阵是方阵，`batch_stride_A` 为 `M*M`；后面 B/C/D 的 stride 则使用矩形输出对应的 `M*N`。

### Lines 401-445: Launch the 3xTF32 SYMM kernel / 启动 3xTF32 SYMM kernel
```cpp
  typename Symm_3xTF32::Arguments arguments_3xtf32{
                                     cutlass::gemm::GemmUniversalMode::kGemm,
                                     problem_size,
                                     batch_count,
                                     {alpha, beta},
                                     tensor_a_F32.device_data(),
                                     tensor_b_F32.device_data(),
                                     tensor_c_F32.device_data(),
                                     tensor_d_3xTF32.device_data(),
                                     batch_stride_A,
                                     problem_size.m() * problem_size.n(),
                                     problem_size.m() * problem_size.n(),
                                     problem_size.m() * problem_size.n(),
                                     tensor_a_F32.layout().stride(0),
                                     tensor_b_F32.layout().stride(0),
                                     tensor_c_F32.layout().stride(0),
                                     tensor_d_3xTF32.layout().stride(0)
                                     };

  size_t workspace_size_3xtf32 = Symm_3xTF32::get_workspace_size(arguments_3xtf32);
  cutlass::device_memory::allocation<uint8_t> workspace_3xtf32(workspace_size_3xtf32);

  Symm_3xTF32 symm_op_3xtf32;

  cutlass::Status status_3xtf32 = symm_op_3xtf32.can_implement(arguments_3xtf32);
  CUTLASS_CHECK(status_3xtf32);

  status_3xtf32 = symm_op_3xtf32.initialize(arguments_3xtf32, workspace_3xtf32.get());
  CUTLASS_CHECK(status_3xtf32);

  status_3xtf32 = symm_op_3xtf32();
  CUTLASS_CHECK(status_3xtf32);

  tensor_d_3xTF32.sync_host();
```
**EN**: This is the standard CUTLASS device API pattern: build `Arguments`, query workspace, allocate workspace, call `can_implement()`, call `initialize()`, then invoke `operator()`. Even though the operation is SYMM, the launch mode is `kGemm` because the universal front end uses GEMM-style runtime descriptors.
**CN**: 这是标准的 CUTLASS 设备端 API 流程：构造 `Arguments`、查询 workspace、大块分配、调用 `can_implement()`、调用 `initialize()`，最后执行 `operator()`. 虽然运算是 SYMM，但启动模式依然写成 `kGemm`，因为通用前端使用 GEMM 风格的运行时描述符。

### Lines 448-492: Launch the 1xTF32 kernel / 启动 1xTF32 kernel
```cpp
  typename Symm_1xTF32::Arguments arguments_1xtf32{
                                     cutlass::gemm::GemmUniversalMode::kGemm,
                                     problem_size,
                                     batch_count,
                                     {alpha, beta},
                                     tensor_a_F32.device_data(),
                                     tensor_b_F32.device_data(),
                                     tensor_c_F32.device_data(),
                                     tensor_d_1xTF32.device_data(),
                                     batch_stride_A,
                                     problem_size.m() * problem_size.n(),
                                     problem_size.m() * problem_size.n(),
                                     problem_size.m() * problem_size.n(),
                                     tensor_a_F32.layout().stride(0),
                                     tensor_b_F32.layout().stride(0),
                                     tensor_c_F32.layout().stride(0),
                                     tensor_d_1xTF32.layout().stride(0)
                                     };

  size_t workspace_size_1xtf32 = Symm_1xTF32::get_workspace_size(arguments_1xtf32);
  cutlass::device_memory::allocation<uint8_t> workspace_1xtf32(workspace_size_1xtf32);

  Symm_1xTF32 symm_op_1xtf32;

  cutlass::Status status_1xtf32 = symm_op_1xtf32.can_implement(arguments_1xtf32);
  CUTLASS_CHECK(status_1xtf32);

  status_1xtf32 = symm_op_1xtf32.initialize(arguments_1xtf32, workspace_1xtf32.get());
  CUTLASS_CHECK(status_1xtf32);

  status_1xtf32 = symm_op_1xtf32();
  CUTLASS_CHECK(status_1xtf32);

  tensor_d_1xTF32.sync_host();
```
**EN**: The only meaningful difference from the previous block is the kernel type. Keeping the runtime path identical isolates the effect of the math operator and makes the final error comparison fair.
**CN**: 与上一段相比，这里真正变化的只有 kernel 类型。保持运行时路径完全一致，可以把最终误差差异主要归因于底层数学运算符本身。

### Lines 494-541: Launch the FP64 reference kernel / 启动 FP64 参考 kernel
```cpp
  typename Symm_F64::Arguments arguments_f64{
                                     cutlass::gemm::GemmUniversalMode::kGemm,
                                     problem_size,
                                     batch_count,
                                     {double(options.alpha), double(options.alpha)},
                                     tensor_a_F64.device_data(),
                                     tensor_b_F64.device_data(),
                                     tensor_c_F64.device_data(),
                                     tensor_d_F64.device_data(),
                                     batch_stride_A,
                                     problem_size.m() * problem_size.n(),
                                     problem_size.m() * problem_size.n(),
                                     problem_size.m() * problem_size.n(),
                                     tensor_a_F64.layout().stride(0),
                                     tensor_b_F64.layout().stride(0),
                                     tensor_c_F64.layout().stride(0),
                                     tensor_d_F64.layout().stride(0)
                                     };

  size_t workspace_size_f64 = Symm_F64::get_workspace_size(arguments_f64);
  cutlass::device_memory::allocation<uint8_t> workspace_f64(workspace_size_f64);

  Symm_F64 symm_op_f64;

  cutlass::Status status_f64 = symm_op_f64.can_implement(arguments_f64);
  CUTLASS_CHECK(status_f64);

  status_f64 = symm_op_f64.initialize(arguments_f64, workspace_f64.get());
  CUTLASS_CHECK(status_f64);

  status_f64 = symm_op_f64();
  CUTLASS_CHECK(status_f64);

  cudaDeviceSynchronize();

  tensor_d_F64.sync_host();
```
**EN**: The FP64 path follows the same composition pattern, which is useful pedagogically: once you understand one CUTLASS device operator, the others look structurally similar. Notice that the scalar tuple here is written literally as `{alpha, alpha}` in double precision; the analysis should track the code as written.
**CN**: FP64 路径沿用了同样的组合模式，这在教学上很有价值：理解了一个 CUTLASS 设备算子后，其它算子的结构通常也类似。需要注意，这里双精度标量元组在源码中被直接写成 `{alpha, alpha}`；分析应以源码实际写法为准。

### Lines 543-576: Optional cuBLAS reference / 可选的 cuBLAS 参考
```cpp
#if CUTLASS_ENABLE_CUBLAS
  cublasStatus_t cublas_status;
  cublasHandle_t handle;

  cublas_status = cublasCreate(&handle);
  if (cublas_status != CUBLAS_STATUS_SUCCESS) {
  std::cerr << "Failed to create cuBLAS handle." << std::endl;
    return false;
  }

  cublas_status = cublasSsymm(
      handle,
      CUBLAS_SIDE_LEFT,
      CUBLAS_FILL_MODE_LOWER,
      problem_size.m(),
      problem_size.n(),
      static_cast<const float*>(&alpha),
      static_cast<const float*>(tensor_a_F32.device_data()),
      int(tensor_a_F32.layout().stride(0)),
      static_cast<const float*>(tensor_b_F32.device_data()),
      int(tensor_b_F32.layout().stride(0)),
      static_cast<const float*>(&beta),
      static_cast<float*>(tensor_d_cublasF32.device_data()),
      int(tensor_d_cublasF32.layout().stride(0))
    );

  cudaDeviceSynchronize();

  tensor_d_cublasF32.sync_host();
#endif
```
**EN**: cuBLAS is used only as an external FP32 baseline. The call matches the CUTLASS configuration exactly—left-side, lower-filled, column-major—so the later relative error metrics compare equivalent operations rather than different conventions.
**CN**: cuBLAS 这里只作为外部 FP32 基线。调用参数与 CUTLASS 配置保持一致：左侧对称、下三角、列主序，因此后面的相对误差是在“同一种运算语义”之间比较，而不是不同约定之间比较。

### Lines 578-611: Relative error metrics / 相对误差度量
```cpp
#if CUTLASS_ENABLE_CUBLAS
  cutlass::HostTensor<double, LayoutOutput> tensor_d_cublasF32_in_F64(problem_size.mn());
  cutlass::reference::host::TensorCopy(tensor_d_cublasF32_in_F64.host_view(), tensor_d_cublasF32.host_view());

  double l2_norm_cublasf32_vs_f64 = cutlass::reference::host::TensorRelativeErrorMetric(
    tensor_d_cublasF32_in_F64.host_view(), tensor_d_F64.host_view());
#endif

  cutlass::HostTensor<double, LayoutOutput> tensor_d_3xTF32_in_F64(problem_size.mn());
  cutlass::reference::host::TensorCopy(tensor_d_3xTF32_in_F64.host_view(), tensor_d_3xTF32.host_view());
  double l2_norm_3xtf32_vs_f64 = cutlass::reference::host::TensorRelativeErrorMetric(
    tensor_d_3xTF32_in_F64.host_view(), tensor_d_F64.host_view());

  cutlass::HostTensor<double, LayoutOutput> tensor_d_1xTF32_in_F64(problem_size.mn());
  cutlass::reference::host::TensorCopy(tensor_d_1xTF32_in_F64.host_view(), tensor_d_1xTF32.host_view());
  double l2_norm_1xtf32_vs_f64 = cutlass::reference::host::TensorRelativeErrorMetric(
    tensor_d_1xTF32_in_F64.host_view(), tensor_d_F64.host_view());

#if CUTLASS_ENABLE_CUBLAS
  double l2_norm_3xtf32_vs_cublasf32 = cutlass::reference::host::TensorRelativeErrorMetric(
    tensor_d_3xTF32.host_view(), tensor_d_cublasF32.host_view());
#endif

  double l2_norm_3xtf32_vs_1xtf32 = cutlass::reference::host::TensorRelativeErrorMetric(
    tensor_d_3xTF32.host_view(), tensor_d_1xTF32.host_view());
```
**EN**: The example normalizes all comparisons as relative L2 error. Converting FP32 outputs into FP64 host tensors before comparing against the FP64 reference avoids accidental loss of precision in the metric itself.
**CN**: 这里统一使用相对 L2 误差。先把 FP32 输出拷贝到 FP64 主机张量，再与 FP64 参考比较，可以避免误差度量过程本身受到额外精度损失影响。

### Lines 615-633: Reporting / 输出结果
```cpp
  std::cout << "Problem Size: (" << problem_size.m() << "," << problem_size.n() << "," << problem_size.k() << ") "
            << "Alpha: "  << alpha << "," << " Beta: "  << beta << std::endl;
  std::cout << std::fixed;
  std::cout << "Normalized L2 norm of" << std::endl;
  std::cout.precision(8);
  std::cout << std::scientific
#if CUTLASS_ENABLE_CUBLAS
            << " - cuBLAS F32 error with F64 reference    : " << l2_norm_cublasf32_vs_f64 << std::endl
#endif
            << " - 3xTF32 error with F64 reference        : " << l2_norm_3xtf32_vs_f64 << std::endl
            << " - 1xTF32 error with F64 reference        : " << l2_norm_1xtf32_vs_f64 << std::endl
#if CUTLASS_ENABLE_CUBLAS
            << " - 3xTF32 error with cuBLAS F32 reference : " << l2_norm_3xtf32_vs_cublasf32 << std::endl
#endif
            << " - 3xTF32 error with 1xTF32 reference     : " << l2_norm_3xtf32_vs_1xtf32 << std::endl;

  return true;
}
```
**EN**: The output focuses on numerical behavior rather than throughput. This matches the purpose of the sample: show that 3xTF32 can recover much of FP32 accuracy while still using Ampere Tensor Cores.
**CN**: 输出重点是数值表现，而不是吞吐率。这与示例目标一致：展示 3xTF32 在使用 Ampere Tensor Core 的同时，如何尽量逼近 FP32 精度。

### Lines 635-665: Toolchain and device guards / 工具链与设备检查
```cpp
int main(int argc, const char **argv) {

  bool notSupported = false;

  // Ampere Tensor Core operations exposed with mma.sync and ldmatrix are first available
  // in CUDA 11.0.
  //
  // CUTLASS must be compiled with CUDA 11.0 Toolkit to run these examples.
  if (!(__CUDACC_VER_MAJOR__ >= 11)) {
    std::cerr << "Ampere Tensor Core operations must be compiled with CUDA 11.0 Toolkit or later." << std::endl;
    notSupported = true;
  }

  cudaDeviceProp props;

  cudaError_t error = cudaGetDeviceProperties(&props, 0);
  if (error != cudaSuccess) {
    std::cerr << "cudaGetDeviceProperties() returned an error: " << cudaGetErrorString(error) << std::endl;
    return -1;
  }

  if (!((props.major * 10 + props.minor) >= 80)) {
    std::cerr << "Ampere Tensor Core operations must be run on a machine with compute capability at least 80."
              << std::endl;
    notSupported = true;
  }

  if (notSupported) {
    return 0;
  }
```
**EN**: The sample is deliberately conservative: it treats unsupported systems as a no-op success rather than a failure. That keeps automated example suites green on older toolkits or GPUs while still documenting the SM80/CUDA 11 requirement.
**CN**: 这个示例在兼容性处理上比较保守：遇到不支持的环境时返回 0，而不是报错失败。这样在旧工具链或旧 GPU 上运行自动化示例集时不会误报失败，同时仍清楚写明需要 SM80/CUDA 11。

### Lines 667-687: Option flow and execution / 选项流程与执行入口
```cpp
  Options options;
  options.parse(argc, argv);

  if (options.help) {
    options.print_usage(std::cout) << std::endl;
    return 0;
  }

  bool result = true;

  if (!options.valid()) {
    std::cerr << "Invalid problem." << std::endl;
    return -1;
  }

  result = run(options);

  if (!result) return -1;

  return 0;
}
```
**EN**: `main()` is intentionally thin: parse, print usage if requested, validate alignment assumptions, then delegate everything to `run()`. This separation keeps the CUTLASS launch logic isolated from environment checks and CLI plumbing.
**CN**: `main()` 被刻意保持得很薄：解析参数、按需打印帮助、验证对齐约束，然后把主体工作交给 `run()`。这样 CUTLASS 的启动逻辑就能与环境检查、命令行处理解耦。

---
## Key Concepts / 关键概念
- `cutlass::gemm::device::Symm` as the high-level symmetric BLAS operator / `cutlass::gemm::device::Symm` 作为高层对称 BLAS 算子
- 3xTF32 via `cutlass::arch::OpMultiplyAddFastF32` / 通过 `cutlass::arch::OpMultiplyAddFastF32` 实现 3xTF32
- 1xTF32 baseline via standard `OpMultiplyAdd` / 使用标准 `OpMultiplyAdd` 的 1xTF32 基线
- Column-major, left-side, lower-filled symmetric storage / 列主序、左侧对称、下三角存储约定
- CUTLASS device API pattern: `Arguments -> workspace -> can_implement -> initialize -> operator()` / CUTLASS 设备 API 模式：`Arguments -> workspace -> can_implement -> initialize -> operator()`
- Epilogue `LinearCombination` for `D = alpha * accum + beta * C` / 用 `LinearCombination` 实现 `D = alpha * accum + beta * C`
- Accuracy evaluation with relative L2 error against FP64 and cuBLAS / 基于 FP64 与 cuBLAS 的相对 L2 误差评估

## Dependencies / 依赖项
- `<iostream>` — console reporting / 控制台输出
- `<vector>` — standard container support, lightly used by the example context / 标准容器支持，作为示例环境的一部分
- `<limits>` — numeric traits support / 数值范围辅助
- `cutlass/blas3.h` — BLAS-3 related CUTLASS declarations / CUTLASS 的 BLAS-3 相关声明
- `cutlass/gemm/device/symm.h` — device-level SYMM operator / 设备级 SYMM 算子
- `cutlass/util/command_line.h` — simple CLI parser / 简单命令行解析器
- `cutlass/util/host_tensor.h` — unified host/device tensor storage wrapper / 主机/设备统一张量封装
- `cutlass/util/reference/host/symm.h` — host SYMM reference helpers / 主机端 SYMM 参考辅助
- `cutlass/util/reference/host/tensor_reduce.h` — host tensor reductions / 主机端张量规约
- `cutlass/util/reference/host/tensor_compare.h` — tensor comparison helpers / 张量比较辅助
- `cutlass/util/reference/host/tensor_norm.h` — norm computation helpers / 范数计算辅助
- `cutlass/util/reference/host/tensor_copy.h` — tensor copy helpers / 张量拷贝辅助
- `cutlass/util/reference/host/tensor_fill.h` — random/constant tensor initialization / 张量随机或常量填充
- `cutlass/util/reference/host/error_metrics.h` — relative error metrics / 相对误差度量
- `cutlass/util/tensor_view_io.h` — tensor view printing utilities / 张量视图打印工具
- `helper.h` — example-local CUDA/CUTLASS helper macros / 示例本地 CUDA/CUTLASS 辅助宏
- `<cublas_v2.h>` — optional cuBLAS SSYMM reference path / 可选的 cuBLAS SSYMM 参考路径
