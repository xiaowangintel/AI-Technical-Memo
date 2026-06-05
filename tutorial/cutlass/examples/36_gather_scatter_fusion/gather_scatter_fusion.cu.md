# gather_scatter_fusion.cu — Code Analysis / 代码分析
**Source / 源文件**: `examples/36_gather_scatter_fusion/gather_scatter_fusion.cu`
**Purpose / 用途**: Demonstrates an SM80 CUTLASS `GemmUniversal` kernel that gathers selected columns of B, performs a reduced-N GEMM, and scatters results back into indexed columns of D with correctness and performance checks. / 展示一个面向 SM80 的 CUTLASS `GemmUniversal` 内核：按索引收集 B 的目标列，执行缩减 N 维的 GEMM，并把结果散射回 D 的指定列，同时进行正确性与性能验证。
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
```
**EN**: Lines 1-30 are the standard BSD-3-Clause license header. They are not executable, but they establish the legal terms for redistributing or modifying this example, which is important for a template-heavy sample that users often copy into their own experiments.
**CN**: 第 1-30 行是标准的 BSD-3-Clause 许可证头。它们不会参与执行，但为重新分发或修改该示例设定了法律条件；这对用户经常复制、改造的模板化示例尤其重要。

```cpp
// This example fuses gather before GEMM and scatter after GEMM into the same
// GEMM kernel.  Gather and scatter operation is controlled by an index vector
// to select rows or columns from A, B, C or D matrices.
//
// Suppose, all matrices are column major.  The pseudo code of the fused kernel
// in this example is essentially
//
//    for (int i = 0; i < problem_size.m(); ++i) {
//      for (int j = 0; j < options.index_size; ++j) {
//        int b_c_d_col = tensor_indices.at({j, 0});
//
//        for (int k = 0; k < options.index_size; ++k) {
//            tensor_d_ref.at({i, b_c_d_col}) +=
//              alpha * tensor_a.at({i, k}) * tensor_b.at({k, b_c_d_col});
//        }
//      }
//
// Note that the index vector contains unique random integers with max to be N - 1
//
// The gather/scatter operation works best when we can still keep the biggest
// alignment. For example, when the matrix is row major, we select rows. When
// the matrix is column major, we select columns.
//
// Not all the combination of gather and scatter are legal. For example, if A is
// row major and C/D is column major, we cannot gather A and scatter C/D at the
// same time.
//
// Also, we don't check the index value is legal and index array point is valid
// for the sake of the performance.
```
**EN**: Lines 32-60 explain the fused algorithm. The example compresses the logical N dimension by using an index vector, gathers the needed columns from B, and scatters the computed results back into selected columns of D (and effectively the output path that also reads C for beta scaling). The comments also state an important CUTLASS design rule: gather/scatter should follow the layout's naturally aligned dimension whenever possible. Because the matrices are column-major here, selecting columns preserves favorable access patterns and vector alignment. The notes about illegal combinations warn that some layout pairings cannot support fused gather and scatter simultaneously without breaking iterator assumptions.
**CN**: 第 32-60 行说明了融合算法。该示例通过索引向量压缩逻辑上的 N 维，对 B 的目标列执行 gather，再把结果 scatter 回 D 的指定列（同时输出路径也会按同一映射读取 C 以完成 beta 缩放）。注释还点出了一个关键的 CUTLASS 设计原则：gather/scatter 最好沿着布局天然更易对齐的维度进行。这里矩阵均为列主序，因此选择“列”能保持更好的访存模式和向量对齐。关于“非法组合”的说明则提醒：某些布局搭配无法在不破坏迭代器假设的前提下同时支持融合 gather 与 scatter。

```cpp
#include <cstdlib>
#include <cstdio>
#include <ctime>
#include <cmath>
#include <cassert>
#include <cuda_runtime.h>

#include <algorithm>
#include <iostream>
#include <fstream>
#include <random>
#include <numeric>

#include "cutlass/cutlass.h"
#include "cutlass/gemm/device/gemm_universal.h"
#include "cutlass/epilogue/thread/linear_combination.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/command_line.h"
#include "cutlass/util/reference/device/gemm.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/tensor_view_io.h"
#include "helper.h"
```
**EN**: Lines 62-85 collect C/C++, CUDA, and CUTLASS headers. Standard library headers support randomness, I/O, and simple utilities; CUDA runtime APIs provide synchronization and timing; CUTLASS headers contribute the GEMM kernel type, epilogue operator, host tensor wrappers, command-line parsing, and reference helpers. The local helper header supplies the example's error-checking macros.
**CN**: 第 62-85 行汇集了 C/C++、CUDA 与 CUTLASS 头文件。标准库头文件用于随机数、I/O 和基础工具；CUDA Runtime API 负责同步与计时；CUTLASS 头文件提供 GEMM 内核类型、epilogue 算子、HostTensor 封装、命令行解析以及参考实现辅助工具。本地 `helper.h` 则提供示例使用的错误检查宏。

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////

/// Result structure
struct Result {

  double runtime_ms;
  double gflops;
  cutlass::Status status;
  cudaError_t error;
  bool passed;

  //
  // Methods
  //

  Result(
    double runtime_ms = 0,
    double gflops = 0,
    cutlass::Status status = cutlass::Status::kSuccess,
    cudaError_t error = cudaSuccess
  ):
    runtime_ms(runtime_ms), gflops(gflops), status(status), error(error), passed(true) { }
};
```
**EN**: Lines 87-109 define a lightweight `Result` struct. It stores runtime, throughput, CUTLASS status, CUDA error state, and a pass/fail bit. The constructor supplies success defaults so the profiling path can fill only the fields that change. This is typical in CUTLASS examples: keep kernel execution state separate from command-line options and tensor data.
**CN**: 第 87-109 行定义了一个轻量级 `Result` 结构体，用于保存运行时间、吞吐率、CUTLASS 状态、CUDA 错误状态以及通过标志。构造函数提供“成功”默认值，因此性能测试路径只需填写发生变化的字段。这也是 CUTLASS 示例的常见写法：把内核执行状态与命令行选项、张量数据分开管理。

```cpp
// Command line options parsing
struct Options {

  bool help;

  cutlass::gemm::GemmCoord problem_size;
  int index_size;

  bool reference_check;
  int iterations;
  
  Options():
    help(false),
    problem_size({248, 1024, 1024}),
    index_size(240),
    reference_check(true),
    iterations(20) { }

  bool valid() {
    return true;
  }

  // Parses the command line
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);

    if (cmd.check_cmd_line_flag("help")) {
      help = true;
    }

    cmd.get_cmd_line_argument("m", problem_size.m());
    cmd.get_cmd_line_argument("n", problem_size.n());
    cmd.get_cmd_line_argument("k", problem_size.k());

    cmd.get_cmd_line_argument("index_size", index_size);
    
    cmd.get_cmd_line_argument("iterations", iterations);

  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << "36_gather_scatter_fusion example\n\n"
      << "  This example uses the CUTLASS Library to fuse gather/scatter into GEMM\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement.\n\n"
      << "  --m=<int>                   GEMM M dimension\n"
      << "  --n=<int>                   GEMM N dimension\n"
      << "  --k=<int>                   GEMM K dimension\n"
      << "  --index_size=<int>          size of N dimension index\n"
      << "  --iterations=<int>          Number of profiling iterations to perform.\n\n";

    out << "\n\nExamples:\n\n"
      << "$ ./examples/36_gather_scatter_fusion/36_gather_scatter_fusion --m=1024 --n=512 --k=1024 \\\n"
      << "     --index_size=128\n\n";

    return out;
  }

  /// Compute performance in GFLOP/s
  double gflops(double runtime_s) const {

    // Number of real-valued multiply-adds
    int64_t fmas = problem_size.m() * int64_t(index_size) * problem_size.k();
    
    // Two flops per multiply-add
    return 2.0 * double(fmas) / double(1.0e9) / runtime_s;
  }
};
```
**EN**: Lines 113-182 implement `Options`, which owns the GEMM shape (`GemmCoord` for M/N/K), the compressed index length, validation toggle, and profiling iteration count. `parse()` uses `cutlass::CommandLine`, a small utility commonly used across CUTLASS examples, to read named arguments such as `--m`, `--n`, `--k`, and `--index_size`. `print_usage()` documents the sample, and `gflops()` deliberately uses `index_size` instead of the original N because the fused gather makes the effective output width equal to the number of selected columns, not the full dense matrix width. `valid()` currently returns true unconditionally, so this example relies on the kernel capability query later for real feasibility checks.
**CN**: 第 113-182 行实现了 `Options`，负责保存 GEMM 规模（`GemmCoord` 中的 M/N/K）、压缩后的索引长度、参考校验开关以及性能测试迭代次数。`parse()` 使用 CUTLASS 示例中常见的 `cutlass::CommandLine` 来读取 `--m`、`--n`、`--k`、`--index_size` 等命名参数。`print_usage()` 输出帮助信息，而 `gflops()` 之所以使用 `index_size` 而不是原始 N，是因为融合 gather 后，实际输出宽度等于被选中的列数，而不是完整稠密矩阵的宽度。`valid()` 目前始终返回 true，因此真正的可行性检查主要依赖后面的内核能力查询。

```cpp
// The code section below describes datatype for input, output matrices and computation between
// elements in input matrices.
using ElementAccumulator = float;                   // <- data type of accumulator
using ElementComputeEpilogue = ElementAccumulator;  // <- data type of epilogue operations
using ElementInputA = cutlass::half_t;              // <- data type of elements in input matrix A
using ElementInputB = cutlass::half_t;              // <- data type of elements in input matrix B
using ElementOutput = float;                        // <- data type of elements in output matrix D
```
**EN**: Lines 186-193 choose numerical types. A and B use `cutlass::half_t` to target Tensor Core-friendly FP16 inputs, accumulation happens in `float`, the epilogue also computes in `float`, and the final output type is `float`. This mixed-precision setup is a standard CUTLASS pattern for balancing throughput and numerical stability.
**CN**: 第 186-193 行选择数值类型。A 与 B 使用 `cutlass::half_t`，以匹配 Tensor Core 友好的 FP16 输入；累加类型是 `float`；epilogue 计算类型同样是 `float`；最终输出也是 `float`。这种混合精度配置是 CUTLASS 中很常见的折中方案，兼顾吞吐率与数值稳定性。

```cpp
// The code section below describes matrix layout of input and output matrices.
// Column Major for Matrix A, B and C.
//
using LayoutInputA = cutlass::layout::ColumnMajor;
using LayoutInputB = cutlass::layout::ColumnMajor;
using LayoutOutput = cutlass::layout::ColumnMajor;

// This code section describes whether you want to use tensor cores or regular SIMT cores on GPU SM
using MMAOp = cutlass::arch::OpClassTensorOp;

// This code section describes CUDA SM architecture number
using SmArch = cutlass::arch::Sm80;
```
**EN**: Lines 194-205 select layouts and target hardware. All tensors are `ColumnMajor`, which directly matches the earlier comment about gathering/scattering columns. `MMAOp = OpClassTensorOp` instructs CUTLASS to build a Tensor Core MMA pipeline instead of a SIMT pipeline, and `SmArch = Sm80` binds the kernel specialization to Ampere-class GPUs where the chosen instruction shape is valid.
**CN**: 第 194-205 行指定布局与目标硬件。所有张量都采用 `ColumnMajor`，与前文“按列 gather/scatter”的设计完全一致。`MMAOp = OpClassTensorOp` 告诉 CUTLASS 使用 Tensor Core MMA 管线而不是普通 SIMT 管线，`SmArch = Sm80` 则把该内核特化到支持所选指令形状的 Ampere 架构 GPU。

```cpp
// This code section describes the tile size a thread block will compute
using ShapeMMAThreadBlock =
    cutlass::gemm::GemmShape<128, 128, 32>;  // <- threadblock tile M = 128, N = 128, K = 32
// This code section describes tile size a warp will compute
using ShapeMMAWarp = cutlass::gemm::GemmShape<64, 64, 32>;  // <- warp tile M = 64, N = 64, K = 32 
// This code section describes the size of MMA op
using ShapeMMAOp = cutlass::gemm::GemmShape<16, 8, 16>;  // <- MMA Op tile M = 8, N = 8, K = 4
// 16, 8, 8 -> Turing
// 16, 8, 16 -> Ampere

// This code section describes how threadblocks are scheduled on GPU
using SwizzleThreadBlock = cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<>;
```
**EN**: Lines 207-218 define the tiling hierarchy and threadblock swizzle. `ShapeMMAThreadBlock<128,128,32>` is the CTA tile, `ShapeMMAWarp<64,64,32>` is the warp tile, and `ShapeMMAOp<16,8,16>` is the Tensor Core instruction tile used on Ampere. CUTLASS composes GEMM from these nested shapes: many instruction tiles form a warp tile, many warp tiles form a threadblock tile. `GemmIdentityThreadblockSwizzle<>` keeps launch ordering simple, which is appropriate for an example focused on fused data movement rather than advanced scheduling.
**CN**: 第 207-218 行定义了分块层次和线程块 swizzle。`ShapeMMAThreadBlock<128,128,32>` 是 CTA 级 tile，`ShapeMMAWarp<64,64,32>` 是 warp 级 tile，`ShapeMMAOp<16,8,16>` 则是 Ampere 上 Tensor Core 指令级 tile。CUTLASS 正是通过这些嵌套形状来组合 GEMM：多个指令 tile 组成一个 warp tile，多个 warp tile 再组成一个线程块 tile。`GemmIdentityThreadblockSwizzle<>` 采用最直接的线程块调度顺序，适合这个以融合数据搬运为重点的示例。

```cpp
// Define the epilogue operation as LinearCombination. This is approximately equal to
//
//    d_ij = alpha * sum_k(a_ik * b_kj) + c_ij
//
using EpilogueOp = cutlass::epilogue::thread::LinearCombination<
    ElementOutput,                                        // <- data type of output matrix
    128 / cutlass::sizeof_bits<ElementOutput>::value,     // <- this is the number of elements per
                                                          // vectorized memory access. For half
                                                          // precision, it's 8 elements. This becomes
                                                          // the vector width of math instructions in
                                                          // epilogue too
    ElementAccumulator,                                   // <- data type of accumulator
    ElementComputeEpilogue>;                              // <- data type for alpha in linear combination function
```
**EN**: Lines 220-232 define the epilogue as `LinearCombination`, i.e. `D = alpha * Accumulator + beta * C`. The first template argument is the output element type (`float`), the second is the vector length per memory access, and because `ElementOutput` is 32 bits this expression becomes `128 / 32 = 4` elements per 128-bit access. The last two template parameters state that accumulator fragments and alpha/beta math are performed in float. This epilogue choice matters for gather/scatter because the output iterator still needs to apply the linear combination after the MMA mainloop finishes.
**CN**: 第 220-232 行把 epilogue 定义为 `LinearCombination`，也就是 `D = alpha * Accumulator + beta * C`。第一个模板参数是输出元素类型（`float`），第二个参数是每次访存的向量长度；由于 `ElementOutput` 是 32 位，因此这里实际得到的是 `128 / 32 = 4` 个元素的 128 位访问。后两个模板参数说明累加片段与 alpha/beta 的计算都在 float 中完成。对于 gather/scatter 来说，这个 epilogue 很重要，因为 MMA 主循环结束后，输出迭代器仍要在重映射后的输出位置上完成线性组合。

```cpp
// Number of pipelines you want to use
constexpr int NumStages = 5;
// Ampere -> 4/5
// Turing -> 2

using Gemm = cutlass::gemm::device::GemmUniversal<ElementInputA,
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
                                                  NumStages,
                                                  8,     /*alignmentA*/
                                                  8,     /*alignmentB*/
                                                  cutlass::arch::OpMultiplyAdd,
                                                  cutlass::ComplexTransform::kNone,
                                                  cutlass::ComplexTransform::kNone,
                                                  false, /*GatherA*/
                                                  true,  /*GatherB*/
                                                  true   /*ScatterD*/
                                                 >;
```
**EN**: Lines 234-262 are the core CUTLASS kernel declaration. `NumStages = 5` requests a deep cp.async-style pipeline suitable for Ampere. The `GemmUniversal` template then fixes every major kernel policy: operand types/layouts, accumulator type, Tensor Core op class, SM80 target, CTA/warp/instruction shapes, epilogue operator, threadblock swizzle, stage count, and alignments of 8 half elements for A and B (128-bit accesses). `OpMultiplyAdd` selects real-valued multiply-add math, both complex transforms are `kNone`, and the final three boolean template parameters enable or disable fusion features: `GatherA=false`, `GatherB=true`, `ScatterD=true`. That means the mainloop uses a special iterator for B that dereferences an index array instead of walking dense columns, while the output path writes results through a scatter-aware iterator. This is the essence of the example's fused design: avoid launching separate gather and scatter kernels by teaching the GEMM iterators how to remap coordinates.
**CN**: 第 234-262 行是本文件最核心的 CUTLASS 内核声明。`NumStages = 5` 请求较深的流水线阶段数，适合 Ampere 上基于 `cp.async` 的数据搬运。随后 `GemmUniversal` 模板固定了几乎所有关键策略：操作数类型与布局、累加类型、Tensor Core 运算类别、SM80 目标架构、CTA/warp/指令 tile 形状、epilogue 算子、线程块 swizzle、stage 数，以及 A/B 都按 8 个 half 元素对齐（即 128 位访存）。`OpMultiplyAdd` 选择实数乘加，两个复数变换参数都设为 `kNone`。最后三个布尔模板参数直接控制融合特性：`GatherA=false`、`GatherB=true`、`ScatterD=true`。这意味着主循环为 B 使用了带索引解引用能力的特殊迭代器，而输出路径则通过支持 scatter 的迭代器写回结果。也就是说，该示例把坐标重映射逻辑内嵌进 GEMM 迭代器中，从而避免单独发射 gather/scatter 内核，这正是其“融合”设计的核心。

```cpp
int run(Options &options) {

  // ================================================================================
  // Initialization setup

  // Create a tuple of problem size for matrix multiplication
  cutlass::gemm::GemmCoord problem_size = options.problem_size;

  // Create a tuple of problem size for matrix multiplication
  cutlass::gemm::GemmCoord problem_size_real(problem_size.m(),
                                             options.index_size,
                                             problem_size.k());
```
**EN**: Lines 264-276 begin `run()` by materializing the user-selected problem size and then constructing `problem_size_real`. This second `GemmCoord` replaces the original N with `index_size`, which is the logical width seen by the fused GEMM after column selection. Physically, B/C/D still own storage for the full dense N, but the kernel is asked to iterate only over the compacted indexed columns.
**CN**: 第 264-276 行开始实现 `run()`：先取出用户指定的问题规模，再构造 `problem_size_real`。这个新的 `GemmCoord` 用 `index_size` 替代原始 N，表示融合 GEMM 在逻辑上只看到被索引选中的那部分列。物理上 B/C/D 仍然为完整的稠密 N 分配存储，但内核只会遍历压缩后的列集合。

```cpp
  // Initialize tensors using CUTLASS helper functions
  cutlass::HostTensor<ElementInputA, LayoutInputA> tensor_a(
      problem_size.mk());  // <- Create matrix A with dimensions M x K
  cutlass::HostTensor<ElementInputB, LayoutInputB> tensor_b(
      problem_size.kn());  // <- Create matrix B with dimensions K x N
  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_c(
      problem_size.mn());  // <- Create matrix C with dimensions M x N 
  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_d_scattered(
      problem_size.mn());  // <- Create matrix D with dimensions M x N used to store output from
                           // CUTLASS kernel
```
**EN**: Lines 277-287 allocate host/device-backed tensors via `cutlass::HostTensor`. A is `M x K`, B is `K x N`, C is `M x N`, and `tensor_d_scattered` is another `M x N` buffer that will receive sparse-column results in their original dense coordinate system. `HostTensor` is convenient here because it manages both host storage and device allocation with matching layout metadata.
**CN**: 第 277-287 行通过 `cutlass::HostTensor` 分配同时具有主机端和设备端存储的张量。A 为 `M x K`，B 为 `K x N`，C 为 `M x N`，`tensor_d_scattered` 也是 `M x N`，用于在原始稠密坐标系中接收散射后的结果列。这里使用 `HostTensor` 很方便，因为它同时管理主机数据、设备分配以及一致的布局元数据。

```cpp
  // Fill input and output matrices on host using CUTLASS helper functions
  cutlass::reference::host::TensorFillRandomUniform(
      tensor_a.host_view(),
      1,
      ElementInputA(7),
      ElementInputA(-8),
      0);  // <- Fill matrix A on host with uniform-distribution random data

  cutlass::reference::host::TensorFillRandomUniform(
      tensor_b.host_view(),
      1,
      ElementInputA(7),
      ElementInputA(-8),
      0);  // <- Fill matrix B on host with uniform-distribution random data

  cutlass::reference::host::TensorFillRandomUniform(
      tensor_c.host_view(),
      1,
      ElementOutput(7),
      ElementOutput(-8),
      0);  // <- Fill matrix C on host with uniform-distribution random data

  cutlass::reference::host::TensorFill(
    tensor_d_scattered.host_view());  // <- fill matrix D on host with zeros
```
**EN**: Lines 288-311 fill A, B, and C with random values and zero-initialize D. CUTLASS reference helpers keep initialization code compact and layout-correct. Zeroing `tensor_d_scattered` makes the later comparison easier to reason about, because only indexed output columns should change after the fused GEMM.
**CN**: 第 288-311 行为 A、B、C 填充随机值，并把 D 清零。CUTLASS 提供的参考辅助函数使初始化代码更紧凑，也能自动遵守布局约定。将 `tensor_d_scattered` 置零有助于后续比较，因为在融合 GEMM 执行后，理论上只有被索引命中的输出列会发生变化。

```cpp
  cutlass::HostTensor<int, LayoutOutput> tensor_indices(
      {options.index_size, 1});  // <- Create scatter indices with dimensions val_len x 1

  // <- Fill tensor_b_indices on host with unique random integers
  std::vector<int> to_fill(problem_size.n()) ; // vector with ints.
  std::iota (std::begin(to_fill), std::end(to_fill), 0); // Fill with 0, 1, ...., problem_size.n()
  { // std::random_shuffle was deprecated in C++14 and removed in C++17
    std::random_device make_seed;
    std::mt19937 source_of_randomness(make_seed());
    std::shuffle(to_fill.begin(), to_fill.end(), source_of_randomness);
  }
  memcpy(tensor_indices.host_data(), to_fill.data(), options.index_size * sizeof(int));
```
**EN**: Lines 313-324 build the gather/scatter index tensor. The code creates a length-`N` host vector, fills it with `0..N-1`, shuffles it, and copies only the first `index_size` values into `tensor_indices`. This guarantees uniqueness, which is especially important for `ScatterD=true`: if indices repeated, multiple logical output columns would collide on the same physical D column and the semantics would be harder to interpret. The index tensor itself uses `LayoutOutput`, but since it is effectively a 1-D list, only the underlying contiguous storage matters.
**CN**: 第 313-324 行构造 gather/scatter 索引张量。代码先创建长度为 `N` 的主机向量，用 `0..N-1` 初始化，再随机打乱，并把前 `index_size` 个值复制到 `tensor_indices` 中。这样可以保证索引唯一，这对 `ScatterD=true` 尤其关键：如果索引重复，多个逻辑输出列就会写到同一个物理 D 列，语义和验证都会复杂很多。索引张量虽然声明为 `LayoutOutput`，但它本质上是 1 维列表，因此真正重要的是底层连续存储。

```cpp
  // Copy data from host to GPU
  tensor_a.sync_device();
  tensor_b.sync_device();
  tensor_indices.sync_device();
  tensor_c.sync_device();
  tensor_d_scattered.sync_device();

  // Initialize alpha/beta for dot product computation
  ElementComputeEpilogue alpha = ElementComputeEpilogue(1);
  ElementComputeEpilogue beta = ElementComputeEpilogue(1);

  // Split K dimension into 1 partitions
  int split_k_slices = 1;
```
**EN**: Lines 326-338 transfer the initialized tensors to the GPU, then define scalar coefficients and the split-K configuration. `alpha = 1` and `beta = 1` make the math match the explanatory pseudocode plus the standard GEMM epilogue. `split_k_slices = 1` means there is no parallel reduction across K; the example focuses purely on fused gather/scatter rather than split-K composition.
**CN**: 第 326-338 行把初始化后的张量同步到 GPU，然后设置标量系数与 split-K 配置。`alpha = 1`、`beta = 1` 使数学形式与前面的伪代码以及标准 GEMM epilogue 保持一致。`split_k_slices = 1` 表示不对 K 维做并行归约；该示例关注点完全在融合 gather/scatter，而不是 split-K 组合。

```cpp
  // Create a tuple of gemm kernel arguments. This is later passed as arguments to launch
  // instantiated CUTLASS kernel
  typename Gemm::Arguments arguments{
      cutlass::gemm::GemmUniversalMode::kGemm, 
      problem_size_real,                  // <- problem size of matrix multiplication
      split_k_slices,                     // <- k-dimension split factor
      {alpha, beta},                      // <- alpha, beta
      tensor_a.device_data(),             // <- reference to matrix A on device
      tensor_b.device_data(),             // <- reference to matrix B on device
      tensor_c.device_data(),             // <- reference to matrix C on device
      tensor_d_scattered.device_data(),   // <- reference to matrix D on device
      tensor_a.layout().capacity(problem_size.mk()),
      tensor_b.layout().capacity(cutlass::make_Coord(options.index_size, problem_size.k())),
      tensor_c.layout().capacity(problem_size.mn()),
      tensor_d_scattered.layout().capacity(problem_size.mn()),
      tensor_a.layout().stride(),
      tensor_b.layout().stride(),
      tensor_c.layout().stride(),
      tensor_d_scattered.layout().stride(),
      nullptr,                            // <- pointer to index vector to gather A on device
      tensor_indices.device_data(),       // <- pointer to index vector to gather B on device
      tensor_indices.device_data()};      // <- pointer to index vector to scatter D on device
```
**EN**: Lines 340-361 assemble `Gemm::Arguments`, the runtime object that binds actual pointers and strides to the compile-time kernel policy. Important fields are: mode `kGemm`, the compressed `problem_size_real`, scalar tuple `{alpha, beta}`, device pointers for A/B/C/D, capacities and strides derived from the layouts, and finally three index-pointer slots. The first index pointer is `nullptr` because `GatherA` is disabled. The second points to `tensor_indices` so B's iterator can gather selected columns. The third reuses the same index array for `ScatterD`, making the output iterator store each logical column `j` into physical column `tensor_indices[j]`. This is the runtime counterpart to the compile-time fusion booleans declared in the `Gemm` alias.
**CN**: 第 340-361 行构造 `Gemm::Arguments`，这是把“编译期内核策略”与“运行时真实地址/步幅”绑定起来的关键对象。重要字段包括：`kGemm` 模式、压缩后的 `problem_size_real`、标量 `{alpha, beta}`、A/B/C/D 的设备指针、由布局推导出的 capacity 与 stride，以及最后三个索引指针槽位。第一个索引指针为 `nullptr`，因为没有启用 `GatherA`。第二个指向 `tensor_indices`，供 B 的迭代器按索引收集列。第三个复用同一索引数组给 `ScatterD`，使输出迭代器把逻辑列 `j` 写入物理列 `tensor_indices[j]`。这正是前面 `Gemm` 模板中融合布尔参数在运行时的对应落地。

```cpp
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
```
**EN**: Lines 363-378 perform the standard CUTLASS device-kernel lifecycle: ask for extra workspace, allocate it, instantiate the kernel object, verify that the kernel can implement the requested problem, and initialize internal state. The explicit `can_implement()` call is valuable because gather/scatter, alignment, architecture, and tile-size constraints can invalidate combinations that still look type-correct at compile time.
**CN**: 第 363-378 行展示了标准的 CUTLASS 设备内核生命周期：查询额外工作空间、分配它、实例化内核对象、检查该问题是否可实现，然后完成初始化。显式调用 `can_implement()` 很有价值，因为 gather/scatter、对齐要求、目标架构以及 tile 大小等限制，可能让某些“编译能过”的组合在运行时仍然不被支持。

```cpp
  // CPU reference calculation
  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_d_ref(problem_size.mn());
  cutlass::reference::host::TensorFill(
    tensor_d_ref.host_view());  // <- Fill matrix D on host with zeros

  status = gemm_op();
  cudaDeviceSynchronize();
  CUTLASS_CHECK(status);
```
**EN**: Lines 380-388 create the host reference output buffer, zero it, then execute the CUTLASS kernel once before validation and benchmarking. This first launch acts as both a correctness-producing run and a warm-up that ensures initialization overhead is not included in the later timing loop. `cudaDeviceSynchronize()` makes the result ready for host-side comparison.
**CN**: 第 380-388 行创建主机端参考输出缓冲区并清零，然后在验证和计时之前先执行一次 CUTLASS 内核。这个首次发射既产生待比较的结果，也起到 warm-up 作用，避免把初始化开销混入后续计时。`cudaDeviceSynchronize()` 则确保结果已经完成，能够安全地进行主机侧比较。

```cpp
  if (options.reference_check) {
    for (int i = 0; i < problem_size.m(); ++i) {
      for (int j = 0; j < options.index_size; ++j) {
        int b_c_d_col = tensor_indices.at({j, 0});

        for (int k = 0; k < problem_size.k(); ++k) {
            tensor_d_ref.at({i, b_c_d_col}) +=
              alpha * tensor_a.at({i, k}) * tensor_b.at({k, b_c_d_col});
        }

        tensor_d_ref.at({i, b_c_d_col}) += (beta * tensor_c.at({i, b_c_d_col}));
      }
    }

    // Copy output data from CUTLASS and reference kernel to host for comparison
    tensor_d_scattered.sync_host();
  
    bool passed = cutlass::reference::host::TensorEquals(
                    tensor_d_scattered.host_view(),
                    tensor_d_ref.host_view());

    if (!passed) {
      std::cout << "Failed!\n";

      std::stringstream fname;
      fname << "error_gather_GEMM_scatter_fusion.txt";
      std::cerr << "Dumping results in " << fname.str() << "\n";

      std::ofstream file(fname.str());

      file 
        << "A =\n" << tensor_a.host_view()
        << "\nB =\n" << tensor_b.host_view()
        << "\nindices =\n" << tensor_indices.host_view()
        << "\nC =\n" << tensor_c.host_view()
        << "\n\nReference =\n" << tensor_d_ref.host_view()
        << "\nComputed =\n" << tensor_d_scattered.host_view();
      return -1;
    } else {
      std::cout << "Passed!\n";
    }
  }
```
**EN**: Lines 389-430 implement the CPU reference for correctness checking. The nested loops mirror the compressed-N formulation from the file header: iterate over every row `i`, every logical indexed column `j`, map it to a physical column with `tensor_indices`, accumulate across `k`, then add `beta * C`. This makes the verification logic explicitly reflect the fused design instead of comparing against a dense GEMM on a pre-gathered temporary. After syncing D back to host, `TensorEquals` compares the entire dense output buffer. On failure, the code dumps A, B, indices, C, reference D, and computed D to a text file for debugging.
**CN**: 第 389-430 行实现了用于正确性校验的 CPU 参考计算。三层循环严格对应文件开头给出的压缩 N 维伪代码：先遍历每个行索引 `i`，再遍历每个逻辑列 `j`，通过 `tensor_indices` 映射到真实物理列，然后沿 `k` 维累加，最后再加上 `beta * C`。这种写法直接反映了融合设计本身，而不是先构造一个预 gather 的临时矩阵再做普通 GEMM。随后把 D 从设备同步回主机，并用 `TensorEquals` 比较整个稠密输出缓冲区。若失败，代码会把 A、B、索引、C、参考 D 和计算得到的 D 一并写入文本文件，便于调试。

```cpp
  // Result structure
  Result result;

  //
  // Construct events
  //

  cudaEvent_t events[2];

  for (auto & event : events) {
    result.error = cudaEventCreate(&event);
    if (result.error != cudaSuccess) {
      std::cerr << "cudaEventCreate() failed: " << cudaGetErrorString(result.error) << std::endl;
      return -1;
    }
  }

  // Record an event at the start of a series of GEMMs
  result.error = cudaEventRecord(events[0]);
  if (result.error != cudaSuccess) {
    std::cerr << "cudaEventRecord() failed: " << cudaGetErrorString(result.error) << std::endl;
    return -1;
  }
```
**EN**: Lines 432-454 switch from correctness to performance measurement. A `Result` object is created, two CUDA events are allocated, and the start event is recorded. Using events is the standard CUDA API choice for kernel timing because it measures device-side elapsed time without host scheduling noise.
**CN**: 第 432-454 行从正确性检查切换到性能测量。代码先创建 `Result` 对象，再分配两个 CUDA event，并记录起始事件。使用 event 进行计时是标准的 CUDA API 选择，因为它测量的是设备侧实际耗时，能避开主机调度噪声。

```cpp
  //
  // Run profiling loop
  //

  for (int iter = 0; iter < options.iterations; ++iter) {
    // Launch initialized CUTLASS kernel
    status = gemm_op();
    CUTLASS_CHECK(status);
  }
```
**EN**: Lines 456-464 are the profiling loop. Each iteration simply calls `gemm_op()` again using the already-initialized arguments and workspace. This is another common CUTLASS API pattern: expensive policy selection happens at compile time, one-time state setup happens in `initialize()`, and repeated launches become very lightweight.
**CN**: 第 456-464 行是性能测试循环。每次迭代都直接调用已经完成初始化、参数固定的 `gemm_op()`。这也是 CUTLASS API 的典型使用模式：高成本的策略选择发生在编译期，一次性的状态准备发生在 `initialize()` 中，而重复发射则尽量轻量。

```cpp
  //
  // Stop profiling loop
  //

  // Record an event when the GEMMs are complete
  result.error = cudaEventRecord(events[1]);
  if (result.error != cudaSuccess) {
    std::cerr << "cudaEventRecord() failed: " << cudaGetErrorString(result.error) << std::endl;
    return -1;
  }

  // Wait for work on the device to complete.
  result.error = cudaEventSynchronize(events[1]);
  if (result.error != cudaSuccess) {
    std::cerr << "cudaEventSynchronize() failed: " << cudaGetErrorString(result.error) << std::endl;
    return -1;
  }

  // Measure elapsed runtime
  float runtime_ms = 0;
  result.error = cudaEventElapsedTime(&runtime_ms, events[0], events[1]);
  if (result.error != cudaSuccess) {
    std::cerr << "cudaEventElapsed() failed: " << cudaGetErrorString(result.error) << std::endl;
    return -1;
  }

  // Compute average runtime and GFLOPs.
  result.runtime_ms = double(runtime_ms) / double(options.iterations);
  result.gflops = options.gflops(result.runtime_ms / 1000.0);

  // Cleanup
  for (auto event : events) {
    (void)cudaEventDestroy(event);
  }

  std::cout << "Runtime: " << result.runtime_ms << " ms\n";
  std::cout << " GFLOPs: " << result.gflops << "\n";

  return 0;
}
```
**EN**: Lines 466-505 stop timing, synchronize, compute the average runtime, convert it to GFLOP/s using the compressed problem definition, destroy the events, and print the results. The averaging step divides total elapsed milliseconds by `options.iterations`, so the reported throughput reflects steady-state kernel execution rather than one launch. Returning `0` signals success after both validation and profiling paths finish.
**CN**: 第 466-505 行负责结束计时、同步设备、计算平均运行时间、按压缩后的问题规模换算 GFLOP/s、销毁 event，并输出结果。由于这里把总耗时除以 `options.iterations`，因此最终报告的是稳态内核执行性能，而不是单次发射的偶然值。最后返回 `0`，表示验证与性能测试流程都已成功完成。

```cpp
int main(int argc, const char ** argv) {
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
```
**EN**: Lines 507-529 implement environment gating in `main()`. The sample refuses to run unless it was compiled with CUDA 11+ and executed on a GPU with compute capability 8.0 or newer. Those checks are directly tied to earlier template choices: the selected Tensor Core instruction shape and SM80 specialization depend on Ampere support.
**CN**: 第 507-529 行在 `main()` 中实现运行环境检查。该示例要求使用 CUDA 11 及以上版本编译，并运行在计算能力至少为 8.0 的 GPU 上。这个检查与前面的模板选择是直接关联的：所选 Tensor Core 指令形状和 `Sm80` 特化都依赖 Ampere 支持。

```cpp
  Options options;
  options.parse(argc, argv);

  if (options.help) {
    options.print_usage(std::cout) << "\n";
    return 0;
  }

  if (!options.valid()) {
    std::cerr << "Invalid problem." << "\n";
    return -1;
  }

  return run(options);
}
```
**EN**: Lines 531-545 finish the program flow: parse command-line arguments, print help if requested, reject invalid options, and finally call `run(options)`. The structure is intentionally simple because the real teaching goal is the kernel composition and fused iterator configuration, not elaborate application scaffolding.
**CN**: 第 531-545 行完成程序主流程：解析命令行参数、按需输出帮助、拒绝非法配置，然后调用 `run(options)`。整体控制流刻意保持简洁，因为这个示例真正要讲解的是内核组合方式与融合迭代器配置，而不是复杂的应用框架。

---
## Key Concepts / 关键概念
- `GemmUniversal` specialization / `GemmUniversal` 特化：The kernel is fully defined by compile-time template policy plus runtime `Arguments`; this example uses that flexibility to inject gather/scatter iterators. / 该内核由编译期模板策略和运行时 `Arguments` 共同确定；本示例利用这种灵活性注入 gather/scatter 迭代器。
- Alignment-friendly fused indexing / 面向对齐的融合索引：Because the tensors are column-major, selecting columns preserves more natural vectorized access than selecting rows. / 由于张量是列主序，按列选择比按行选择更容易保持自然的向量化访存。
- Gather on B only / 仅对 B 做 gather：`GatherB=true` compresses the logical N dimension without materializing a temporary gathered matrix. / `GatherB=true` 在不显式构造临时 gathered 矩阵的情况下压缩了逻辑 N 维。
- Scatter on output path / 输出路径 scatter：`ScatterD=true` remaps logical output columns back to their original dense positions, allowing the kernel to write sparse-column results directly. / `ScatterD=true` 把逻辑输出列映射回原始稠密位置，使内核可以直接写出稀疏列结果。
- Mixed precision Tensor Core GEMM / 混合精度 Tensor Core GEMM：FP16 inputs with FP32 accumulation/output match a common CUTLASS high-throughput configuration. / FP16 输入配合 FP32 累加与输出，是 CUTLASS 中常见的高吞吐配置。
- Hierarchical tiling / 分层分块：Instruction, warp, and threadblock shapes compose the full MMA pipeline and determine occupancy, reuse, and architecture compatibility. / 指令级、warp 级和线程块级 tile 共同组成完整 MMA 管线，并决定占用率、数据复用与架构兼容性。
- Epilogue linear combination / Epilogue 线性组合：The example keeps the standard `alpha * AB + beta * C` API even while changing how columns are addressed. / 该示例在改变列寻址方式的同时，仍保留标准的 `alpha * AB + beta * C` API。
- Reference-first workflow / 先校验后计时：The code validates against a host implementation before profiling, which is especially important when iterator remapping changes memory semantics. / 代码先与主机参考实现比对，再进行性能测试；当迭代器重映射改变内存语义时，这一点尤为重要。
## Dependencies / 依赖项
- `cstdlib` — basic C runtime utilities / 基础 C 运行库工具
- `cstdio` — formatted I/O helpers / 格式化输入输出
- `ctime` — time-related utilities; included with the sample boilerplate / 时间相关工具；示例模板的一部分
- `cmath` — math helpers and numeric utilities / 数学辅助函数与数值工具
- `cassert` — assertion support for debugging / 调试断言支持
- `cuda_runtime.h` — CUDA runtime APIs for device properties, synchronization, and events / 用于设备属性、同步与事件计时的 CUDA Runtime API
- `algorithm` — algorithms such as `std::shuffle` / 提供 `std::shuffle` 等算法
- `iostream` — console output for status and usage text / 用于状态信息与帮助文本输出
- `fstream` — writes debug dumps on mismatch / 在结果不匹配时输出调试文件
- `random` — random-device and PRNG support for index shuffling / 为索引打乱提供随机数设备与伪随机引擎
- `numeric` — provides `std::iota` for building the index pool / 提供 `std::iota` 以生成索引池
- `cutlass/cutlass.h` — core CUTLASS definitions and base types / CUTLASS 核心定义与基础类型
- `cutlass/gemm/device/gemm_universal.h` — device GEMM interface with configurable gather/scatter fusion / 提供可配置 gather/scatter 融合的设备 GEMM 接口
- `cutlass/epilogue/thread/linear_combination.h` — epilogue operator implementing `alpha * accum + beta * C` / 实现 `alpha * accum + beta * C` 的 epilogue 算子
- `cutlass/util/host_tensor.h` — host/device tensor wrapper with layout metadata / 带布局元数据的主机/设备张量封装
- `cutlass/util/command_line.h` — small command-line parser used by CUTLASS examples / CUTLASS 示例使用的轻量命令行解析器
- `cutlass/util/reference/device/gemm.h` — reference GEMM utilities; included by many examples / 参考 GEMM 工具；许多示例会一并包含
- `cutlass/util/reference/host/tensor_compare.h` — host-side tensor equality checks / 主机侧张量比较工具
- `cutlass/util/reference/host/tensor_copy.h` — host-side tensor copy helpers / 主机侧张量拷贝辅助函数
- `cutlass/util/reference/host/tensor_fill.h` — host-side tensor fill and random initialization helpers / 主机侧张量填充与随机初始化工具
- `cutlass/util/tensor_view_io.h` — stream output for tensors used in debug dumps / 为调试输出提供张量流式打印
- `helper.h` — example-local error-checking macros such as `CUDA_CHECK` and `CUTLASS_CHECK` / 提供示例本地的错误检查宏，例如 `CUDA_CHECK` 与 `CUTLASS_CHECK`
