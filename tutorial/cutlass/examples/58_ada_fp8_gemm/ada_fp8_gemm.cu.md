# ada_fp8_gemm.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/58_ada_fp8_gemm/ada_fp8_gemm.cu`  
**Purpose / 用途**: This example builds an Ada-targeted FP8 GEMM with a specialized epilogue that applies operand scaling, adds a broadcast bias vector, writes an optional auxiliary tensor, and tracks absolute maxima for quantized outputs. It also compares staged accumulation against Ada fast accumulation. / 该示例构建了一个面向 Ada 的 FP8 GEMM，并配套专用 epilogue：它会应用输入/输出缩放、加入广播 bias 向量、可选写出辅助张量，并统计量化输出的绝对值最大值。同时它还对比了 staged accumulation 与 Ada fast accumulation 两种路径。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-117 — Header, FP8 epilogue contract, and Ada GEMM type aliases

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
    \brief Example of running an Ada FP8 GEMM.

    In addition to using FP8 Tensor Core instructions, the Ada FP8 GEMM uses a distinct epilogue
    that enables additional scaling of operands/outputs, storing a pre-activation-function output
    tensor (called the "auxiliary" output), and computing the absolute maximum value of the
    outputs.

    Pseudocode for this epilogue is as follows:

    Aux = ((alpha * scale_a * scale_b) * accumulator) + ((beta * scale_c) * source) + bias
    D = activation(Aux)

    if Aux is fp8 type:
        abs_max_output = max( abs(aux) | (for every aux in Aux))
        Aux = scale_aux * Aux
    endif

    if D is fp8 type:
        abs_max_output = max( abs(d) | (for every d in D))
        D = scale_d * D
    endif

    Parameter Aux is optionally stored to global memory
*/

#include <iostream>
#include <fstream>
#include <sstream>

#include "cutlass/cutlass.h"
#include "cutlass/numeric_conversion.h"
#include "cutlass/util/command_line.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/reference/host/gemm_complex.h"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/distribution.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_norm.h"
#include "cutlass/util/reference/host/gemm.h"

#include "cutlass/epilogue/thread/activation.h"
#include "cutlass/epilogue/thread/linear_combination_generic_with_scaling.h"
#include "cutlass/gemm/device/gemm_universal_with_absmax.h"

#include "cutlass/layout/matrix.h"
#include "cutlass/matrix_coord.h"
#include "cutlass/gemm/device/gemm_universal_adapter.h"


using ElementA = cutlass::float_e4m3_t;
using ElementB = cutlass::float_e4m3_t;
using ElementOutput = cutlass::float_e4m3_t;
using ElementAuxOutput = ElementOutput;
using ElementAccumulator = float;
using LayoutA = cutlass::layout::RowMajor;
using LayoutB = cutlass::layout::ColumnMajor;
using LayoutC = cutlass::layout::RowMajor;
static int const kStages = 3;
static int const kAlignmentA = 16;
static int const kAlignmentB = 16;

using EpilogueOutputOp = cutlass::epilogue::thread::LinearCombinationGenericWithScalingAndAbsMax<
    cutlass::epilogue::thread::ReLu,
    ElementOutput,
    ElementAuxOutput,
    8,
    ElementAccumulator,
    ElementAccumulator
    >;

template <typename MathOperator>
using Gemm_ = cutlass::gemm::device::GemmUniversalWithAbsMax<
    ElementA, LayoutA, ElementB, LayoutB, ElementOutput, LayoutC,
    ElementAccumulator, cutlass::arch::OpClassTensorOp, cutlass::arch::Sm89,
    cutlass::gemm::GemmShape<128, 64, 128>, cutlass::gemm::GemmShape<64, 32, 128>, cutlass::gemm::GemmShape<16, 8, 32>,
    EpilogueOutputOp, cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<>, kStages,
    kAlignmentA, kAlignmentB, MathOperator
  >;

using ElementAbsmax = typename EpilogueOutputOp::ElementAbsmax;


// Command line options parsing
```

**EN**: The file comment explains the epilogue contract in pseudocode: GEMM output is scaled, bias is added, ReLU is applied, and FP8 outputs may also emit absmax statistics. The includes pull in CUTLASS reference helpers plus `GemmUniversalWithAbsMax`, while the type aliases lock the example to SM89, FP8 E4M3 inputs/outputs, row-major A/C, column-major B, and a fixed tile shape. The `EpilogueOutputOp` alias is the core Ada-specific piece because it combines activation, output scaling, auxiliary output support, and absolute-maximum tracking in one thread-level epilogue operator.

**CN**: 文件开头先用伪代码说明 epilogue 语义：GEMM 累加结果会被缩放、叠加 bias、经过 ReLU，并且在输出为 FP8 时还会统计 absmax。头文件既引入了 CUTLASS 的参考实现与张量工具，也引入了 `GemmUniversalWithAbsMax`。随后的一组类型别名把示例固定在 SM89、FP8 E4M3 输入/输出、A/C 行主序、B 列主序和指定 tile 形状上。这里最关键的 Ada 特性是 `EpilogueOutputOp`，它把激活、输出缩放、辅助输出和绝对值最大值统计整合到了同一个线程级 epilogue 运算符中。

### Lines 118-208 — Command-line options and benchmark metadata

```cpp
struct Options {

  bool help;
  bool error;
  bool reference_check;
  cutlass::gemm::GemmCoord problem_size;

  int iterations;
  int warmup_iterations;

  bool scale_A;
  bool scale_B;
  bool scale_C;

  float alpha;
  float beta;

  Options():
    help(false),
    error(false),
    reference_check(false),
    iterations(20),
    warmup_iterations(5),
    scale_A(true),
    scale_B(true),
    scale_C(true),
    alpha(1.f),
    beta(0.f)
  { }

  // Parses the command line
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);

    if (cmd.check_cmd_line_flag("help")) {
      help = true;
      return;
    }

    cmd.get_cmd_line_argument("iterations", iterations, 20);
    cmd.get_cmd_line_argument("warmup_iterations", warmup_iterations, 5);
    cmd.get_cmd_line_argument("reference-check", reference_check, false);
    cmd.get_cmd_line_argument("scale-A", scale_A, true);
    cmd.get_cmd_line_argument("scale-B", scale_B, true);
    cmd.get_cmd_line_argument("scale-C", scale_C, true);
    cmd.get_cmd_line_argument("alpha", alpha, 1.f);
    cmd.get_cmd_line_argument("beta", beta, 0.f);

    int m, n, k;
    cmd.get_cmd_line_argument("m", m, 1024);
    cmd.get_cmd_line_argument("n", n, 1024);
    cmd.get_cmd_line_argument("k", k, 1024);

    problem_size = cutlass::gemm::GemmCoord{m, n, k};
  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << "58_ada_fp8_gemm\n\n"
      << "  This example executes a GEMM using Ada FP8 Tensor Core operations. In addition to performing\n"
      << "  a normal GEMM, the kernel performs the following operations:\n"
      << "      Aux = ((alpha * scale_a * scale_b) * accumulator) + ((beta * scale_c) * source) + bias\n"
      << "        D = activation(Aux)\n\n"
      << "      if Aux is fp8:\n"
      << "         abs_max_output = max( abs(aux) | (for every aux in Aux) )\n"
      << "         Aux = scale_aux * Aux\n\n"
      << "      if D is fp8 type:\n"
      << "         abs_max_output = max( abs(d) | (for every d in D) )\n"
      << "         D = scale_d * D\n\n"
      << "Options:\n\n"
      << "  --help                           If specified, displays this usage statement\n\n"
      << "  --m=<int>                        Sets the M dimension of the GEMM\n"
      << "  --n=<int>                        Sets the N dimension of the GEMM\n"
      << "  --k=<int>                        Sets the K dimension of the GEMM\n"
      << "  --scale-A=<bool>                 Whether to apply a scaling factor to operand A (default: true)\n"
      << "  --scale-B=<bool>                 Whether to apply a scaling factor to operand B (default: true)\n"
      << "  --scale-C=<bool>                 Whether to apply a scaling factor to operand C (default: true)\n"
      << "  --iterations=<int>               Number of profiling iterations to perform\n"
      << "  --warmup-iterations=<int>        Number of warmup iterations to perform\n"
      << "  --reference-check=<bool>         If true, performs reference check\n";

    return out;
  }

  /// Compute performance in GFLOP/s
  float gflops(float runtime_s) const {
    // Two flops per multiply-add
    return 2.0f * float(problem_size.product()) / float(1.0e9) / runtime_s;
  }
};
```

**EN**: The `Options` struct holds problem size, scaling toggles, alpha/beta, warmup, iteration count, and a reference-check switch. `parse()` reads the CLI flags directly into CUTLASS-friendly fields, and `print_usage()` documents the exact FP8 epilogue behavior exposed by this sample. The small `gflops()` helper uses the classic 2*M*N*K formula, so later timing code can report throughput without duplicating arithmetic.

**CN**: `Options` 结构体集中保存问题规模、缩放开关、alpha/beta、预热次数、计时轮数以及参考校验开关。`parse()` 直接把命令行参数读入 CUTLASS 友好的字段中，而 `print_usage()` 则把该示例暴露的 FP8 epilogue 语义明确打印出来。最后的 `gflops()` 用标准的 2*M*N*K 公式计算吞吐，便于后续计时代码统一输出性能。

### Lines 209-318 — Testbed state and host-side initialization helpers

```cpp

/// Helper class to run the kernel
template <typename Gemm>
struct TestbedRunner {

  using ElementAccumulator = typename Gemm::ElementAccumulator;
  using ElementCompute = typename Gemm::GemmKernel::Epilogue::OutputOp::ElementCompute;
  using ElementScalingFactor = typename Gemm::EpilogueOutputOp::ElementScalingFactor;

  static bool const kScaleAux = Gemm::EpilogueOutputOp::kIsScalingAndAmaxAuxOutputNeeded;
  static bool const kScaleOutput = Gemm::EpilogueOutputOp::kIsScalingAndAmaxOutputNeeded;

  /// Initialization
  cutlass::Distribution::Kind init_A;
  cutlass::Distribution::Kind init_B;
  cutlass::Distribution::Kind init_C;
  uint64_t seed;

  cutlass::HostTensor<typename Gemm::ElementA, typename Gemm::LayoutA> tensor_A;
  cutlass::HostTensor<typename Gemm::ElementB, typename Gemm::LayoutB> tensor_B;
  cutlass::HostTensor<typename Gemm::ElementC, typename Gemm::LayoutC> tensor_C;
  cutlass::HostTensor<typename Gemm::EpilogueOutputOp::ElementAuxOutput, typename Gemm::LayoutC> tensor_Aux;
  cutlass::HostTensor<typename Gemm::EpilogueOutputOp::ElementOutput, typename Gemm::LayoutC> tensor_D;
  cutlass::HostTensor<typename Gemm::ElementC, typename Gemm::LayoutC> tensor_Vector;
  cutlass::HostTensor<ElementAccumulator, typename Gemm::LayoutC> tmp_D;
  cutlass::HostTensor<typename Gemm::EpilogueOutputOp::ElementOutput, typename Gemm::LayoutC> reference_D;
  cutlass::HostTensor<typename Gemm::EpilogueOutputOp::ElementAuxOutput, typename Gemm::LayoutC> reference_Aux;
  cutlass::HostTensor<ElementScalingFactor, typename Gemm::LayoutC> scale_A;
  cutlass::HostTensor<ElementScalingFactor, typename Gemm::LayoutC> scale_B;
  cutlass::HostTensor<ElementScalingFactor, typename Gemm::LayoutC> scale_C;
  cutlass::HostTensor<ElementScalingFactor, typename Gemm::LayoutC> scale_D;
  cutlass::HostTensor<ElementScalingFactor, typename Gemm::LayoutC> scale_Aux;
  cutlass::HostTensor<ElementAbsmax, typename Gemm::LayoutC> abs_max_Aux;
  cutlass::HostTensor<ElementAbsmax, typename Gemm::LayoutC> abs_max_D;
  cutlass::HostTensor<ElementAbsmax, typename Gemm::LayoutC> reference_abs_max_Aux;
  cutlass::HostTensor<ElementAbsmax, typename Gemm::LayoutC> reference_abs_max_D;

  //
  // Methods
  //

  TestbedRunner(
    bool scaleA = true,
    bool scaleB = true,
    bool scaleC = true,
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_C_ = cutlass::Distribution::Uniform,
    uint64_t seed_ = 2080
  ):
    init_A(init_A_), init_B(init_B_), init_C(init_C_), seed(seed_) { }

  /// Helper to initialize scaling factors
  template <typename Element, typename Layout>
  bool initialize_scale_factor(cutlass::TensorView<Element, Layout> view, uint64_t seed, int bits=0) {
    cutlass::reference::host::TensorFillRandomUniform(view, seed, double(1.), double(0.), bits);
    return true;
  }

  /// Helper to initialize a tensor view
  template <typename Element, typename Layout>
  bool initialize_tensor(
    cutlass::TensorView<Element, Layout> view,
    cutlass::Distribution::Kind dist_kind,
    uint64_t seed) {

    if (dist_kind == cutlass::Distribution::Uniform) {

      double scope_max, scope_min;
      int bits_input = cutlass::sizeof_bits<Element>::value;
      int bits_output = cutlass::sizeof_bits<typename Gemm::ElementC>::value;

      if (bits_input == 1) {
        scope_max = 2;
        scope_min = 0;
      } else if (bits_input <= 8) {
        scope_max = 2;
        scope_min = -2;
      } else if (bits_output == 16) {
        scope_max = 5;
        scope_min = -5;
      } else {
        scope_max = 8;
        scope_min = -8;
      }

      cutlass::reference::host::TensorFillRandomUniform(
        view, seed, scope_max, scope_min, 0);
    }
    else if (dist_kind == cutlass::Distribution::Identity) {

      cutlass::reference::host::TensorFillIdentity(view);
    }
    else if (dist_kind == cutlass::Distribution::Gaussian) {

      cutlass::reference::host::TensorFillRandomGaussian(view, seed, 0, 0.5);
    }
    else if (dist_kind == cutlass::Distribution::Sequential) {

      cutlass::reference::host::BlockFillSequential(
        view.data(), view.capacity());
    }
    else {
      std::cerr << "Not implemented";
      return false;
    }

    return true;
  }
```

**EN**: `TestbedRunner` owns every tensor used by the run: A/B/C, the output D, the broadcast vector, the optional auxiliary tensor, scaling-factor buffers, and absmax buffers. Its helper methods abstract two common tasks: filling tensors with meaningful distributions and generating scalar scaling factors. The template is parameterized by the GEMM type, so the same host harness can drive both `OpMultiplyAdd` and `OpMultiplyAddFastAccum` without changing the surrounding logic.

**CN**: `TestbedRunner` 持有运行过程中用到的全部张量：A/B/C、输出 D、广播向量、可选辅助输出，以及各种缩放因子和 absmax 缓冲区。它的辅助函数把两类常见工作封装起来：一类是按分布初始化张量，另一类是生成标量缩放因子。由于该测试床按 GEMM 类型模板化，所以同一套主机端流程既能驱动 `OpMultiplyAdd`，也能驱动 `OpMultiplyAddFastAccum`。

### Lines 319-402 — Allocating tensors and preparing scaling / absmax buffers

```cpp
  /// Initializes data structures
  void initialize(const Options& options) {
    //
    // Allocate the GEMM workspace
    //

    tensor_A.resize(options.problem_size.mk());
    tensor_B.resize(options.problem_size.kn());
    tensor_C.resize(options.problem_size.mn());
    tensor_D.resize(options.problem_size.mn());
    tensor_Vector.resize({1, options.problem_size.n()});
    reference_D.resize(options.problem_size.mn(), false);
    tmp_D.resize(options.problem_size.mn(), false);

    initialize_tensor(tensor_A.host_view(), init_A, seed + 2019);
    initialize_tensor(tensor_B.host_view(), init_B, seed + 2018);
    initialize_tensor(tensor_C.host_view(), init_C, seed + 2017);
    initialize_tensor(tensor_Vector.host_view(), init_C, seed + 2020);

    // It is possible to randomly initialize to all zeros, so override this with non-zeros
    // in the upper left corner of each operand.
    cutlass::Coord<2> origin(0);
    tensor_A.host_view().at(origin) = typename Gemm::ElementA(1);
    tensor_B.host_view().at(origin) = typename Gemm::ElementB(1);
    tensor_C.host_view().at(origin) = typename Gemm::ElementC(1);
    tensor_Vector.host_view().at(origin) = typename Gemm::ElementC(1);

    cutlass::reference::host::TensorFill(tensor_D.host_view());
    cutlass::reference::host::TensorCopy(reference_D.host_view(), tensor_C.host_view());

    tensor_A.sync_device();
    tensor_B.sync_device();
    tensor_C.sync_device();
    tensor_D.sync_device();
    tensor_Vector.sync_device();

    int scale_bits = 2;
    if (options.scale_A) {
      scale_A.resize({1, 1});
      initialize_scale_factor(scale_A.host_view(), seed + 2021, scale_bits);
      scale_A.sync_device();
    }

    if (options.scale_B) {
      scale_B.resize({1, 1});
      initialize_scale_factor(scale_B.host_view(), seed + 2022, scale_bits);
      scale_B.sync_device();
    }

    if (options.scale_C) {
      scale_C.resize({1, 1});
      initialize_scale_factor(scale_C.host_view(), seed + 2023, scale_bits);
      scale_C.sync_device();
    }

    if (kScaleOutput) {
      scale_D.resize({1, 1});
      initialize_scale_factor(scale_D.host_view(), seed + 2024, scale_bits);
      scale_D.sync_device();

      abs_max_D.resize({1, 1});
      cutlass::reference::host::TensorFill(abs_max_D.host_view());
      abs_max_D.sync_device();

      reference_abs_max_D.resize({1, 1});
    }

    if (kScaleAux) {
      tensor_Aux.resize(options.problem_size.mn());
      cutlass::reference::host::TensorFill(tensor_Aux.host_view());
      tensor_Aux.sync_device();

      scale_Aux.resize({1, 1});
      initialize_scale_factor(scale_Aux.host_view(), seed + 2025, scale_bits);
      scale_Aux.sync_device();

      abs_max_Aux.resize({1, 1});
      cutlass::reference::host::TensorFill(abs_max_Aux.host_view());
      abs_max_Aux.sync_device();

      reference_Aux.resize(options.problem_size.mn(), false);
      reference_abs_max_Aux.resize({1, 1});
    }
  }
```

**EN**: The `initialize()` method sizes every tensor from the runtime GEMM shape, fills A/B/C and the row-broadcast vector, forces a nonzero element in the upper-left corner to avoid degenerate all-zero inputs, and synchronizes the host tensors to device memory. It then conditionally allocates scale tensors for A, B, C, D, and Aux, as well as the single-element absmax buffers used when the epilogue needs to report FP8 dynamic range. This setup mirrors how FP8 training pipelines usually manage per-tensor scales and amax collection outside the kernel proper.

**CN**: `initialize()` 按运行时问题规模分配全部张量，填充 A/B/C 与按行广播的向量，并强制左上角元素非零，以避免退化成全零输入。随后它把这些张量同步到设备端。接下来，函数按需为 A、B、C、D、Aux 分配缩放因子张量，并在需要时准备单元素 absmax 缓冲区，用于接收 FP8 动态范围统计。这种准备方式与真实 FP8 训练流程中的“每张量 scale + amax 收集”模式是一致的。

### Lines 403-463 — Reference comparison and debug dump path

```cpp

  /// Compares computed reference with device reference and outputs to a file if incorrect
  bool compare_reference(const Options& options) {

    tensor_D.sync_host();

    bool passed = cutlass::reference::host::TensorEquals(reference_D.host_view(), tensor_D.host_view());

    if (kScaleAux) {
      tensor_Aux.sync_host();
      abs_max_Aux.sync_host();
      passed &= cutlass::reference::host::TensorEquals(reference_Aux.host_view(), tensor_Aux.host_view());
      passed &= cutlass::reference::host::TensorEquals(abs_max_Aux.host_view(), reference_abs_max_Aux.host_view());
    }

    if (kScaleOutput) {
      abs_max_D.sync_host();
      passed &= cutlass::reference::host::TensorEquals(abs_max_D.host_view(), reference_abs_max_D.host_view());
    }

    if (!passed) {
      std::cerr << "Reference check failed" << std::endl;

      std::string output_file = "testbed_with_amax_errors.txt";
      std::ofstream file(output_file);

      file
        << "problem: " << options.problem_size
        << ", alpha: " << options.alpha << ", beta: " << options.beta << "\n\n";

      file
        << "A =\n" << tensor_A.host_view()
        << "\nB =\n" << tensor_B.host_view()
        << "\nC =\n" << tensor_C.host_view()
        << "\nVector =\n" << tensor_Vector.host_view()
        << "\nScaleA = " << scale_A.host_view()
        << "\nScaleB = " << scale_B.host_view()
        << "\nScaleC = " << scale_C.host_view()
        << "\nScaleD = " << scale_D.host_view()
        << "\nScaleAux = " << scale_Aux.host_view()
        << "\n\nReference D =\n" << reference_D.host_view()
        << "\nComputed D =\n" << tensor_D.host_view();
      if (kScaleAux) {
        file
          << "\n\nReference Aux =\n" << reference_Aux.host_view()
          << "\nComputed Aux =\n" << tensor_Aux.host_view()
          << "\n\nReference Absmax Aux = " << reference_abs_max_Aux.host_view()
          << "\nComputed Absmax Aux = " << abs_max_Aux.host_view();
      }
      if (kScaleOutput) {
        file
          << "\n\nReference Absmax D = " << reference_abs_max_D.host_view()
          << "\nComputed Absmax D = " << abs_max_D.host_view();
      }

      std::cerr << "Dumped results to " << output_file << std::endl;

    }

    return passed;
  }
```

**EN**: After kernel execution, `compare_reference()` synchronizes D and optional Aux/amax outputs back to host memory and compares them against the host-generated reference tensors. If any comparison fails, it emits a detailed text dump containing the inputs, scaling factors, computed outputs, and expected outputs. That makes this helper more than a boolean checker: it is also the diagnostics path for subtle FP8 saturation or scaling mistakes.

**CN**: 内核执行后，`compare_reference()` 会把 D 以及可选的 Aux/amax 结果同步回主机，并与主机端生成的参考张量逐项比较。一旦有任何比较失败，它就会把输入、缩放因子、计算结果和参考结果完整写入文本文件。因此这个函数不仅是一个布尔校验器，也是定位 FP8 饱和、缩放错误等细微问题的调试出口。

### Lines 464-543 — Host-side reference math for Ada FP8 semantics

```cpp

  /// Verifies the result is a GEMM
  bool verify(const Options& options) {

    cutlass::Coord<2> origin(0);
    ElementCompute scaled_alpha = options.alpha;
    if (options.scale_A) {
      scaled_alpha *= scale_A.host_view().at(origin);
    }
    if (options.scale_B) {
      scaled_alpha *= scale_B.host_view().at(origin);
    }

    ElementCompute scaled_beta = options.beta;
    if (options.scale_C) {
      scaled_beta *= scale_C.host_view().at(origin);
    }

    //
    // Verify
    //

    cutlass::reference::host::GemmComplex<
        typename Gemm::ElementA, typename Gemm::LayoutA,
        typename Gemm::ElementB, typename Gemm::LayoutB,
        typename Gemm::ElementC, typename Gemm::LayoutC,
        ElementCompute, ElementAccumulator, ElementAccumulator
    >(
      options.problem_size,
      scaled_alpha,
      tensor_A.host_ref(),
      Gemm::kTransformA,
      tensor_B.host_ref(),
      Gemm::kTransformB,
      scaled_beta,
      tensor_C.host_ref(),
      tmp_D.host_ref(),
      ElementAccumulator(0)
    );

    ElementCompute tmp_abs_max_Aux(0.);
    ElementCompute tmp_abs_max_D(0.);

    cutlass::NumericConverter<ElementCompute, typename Gemm::ElementC> cvt_c_to_compute;
    cutlass::NumericConverter<ElementCompute, ElementAccumulator> cvt_accum_to_compute;
    cutlass::NumericConverter<ElementAccumulator, ElementCompute> cvt_compute_to_accum;
    cutlass::NumericConverter<typename Gemm::EpilogueOutputOp::ElementOutput, ElementCompute> cvt_compute_to_d;
    cutlass::NumericConverter<typename Gemm::EpilogueOutputOp::ElementAuxOutput, ElementCompute> cvt_compute_to_aux;

    cutlass::absolute_value_op<ElementCompute> abs;
    cutlass::maximum_with_nan_propogation<ElementCompute> max;
    cutlass::epilogue::thread::ReLu<ElementCompute> act;

    ElementScalingFactor d_scale = kScaleOutput ? scale_D.host_view().at(origin) : ElementScalingFactor(1.);

    for (int m = 0; m < options.problem_size.m(); ++m) {
      for (int n = 0; n < options.problem_size.n(); ++n) {
        ElementCompute intermediate = cvt_accum_to_compute(tmp_D.host_view().at({m, n}));
        ElementCompute bias = cvt_c_to_compute(tensor_Vector.host_view().at({0, n}));
        ElementCompute aux = intermediate + bias;
        ElementCompute d = act(aux);
        tmp_abs_max_Aux = max(abs(aux), tmp_abs_max_Aux);
        tmp_abs_max_D = max(abs(d), tmp_abs_max_D);
        reference_D.host_view().at({m, n}) = cvt_compute_to_d(d * d_scale);

        if (kScaleAux) {
          reference_Aux.host_view().at({m, n}) = cvt_compute_to_aux(aux * scale_Aux.host_view().at(origin));
        }
      }
    }

    if (kScaleAux) {
      reference_abs_max_Aux.host_view().at(origin) = cvt_compute_to_accum(tmp_abs_max_Aux);
    }

    if (kScaleOutput) {
      reference_abs_max_D.host_view().at(origin) = cvt_compute_to_accum(tmp_abs_max_D);
    }

    return compare_reference(options);
```

**EN**: `verify()` reconstructs the exact epilogue semantics on the host. It first folds optional A/B/C scaling into `scaled_alpha` and `scaled_beta`, runs a reference GEMM into a temporary accumulator tensor, then applies the broadcast bias, ReLU activation, optional D/Aux scaling, and absolute-maximum reduction. This block is the clearest place to understand the sample’s intended math: the auxiliary tensor captures the pre-activation value, while D stores the post-ReLU result, and both can participate in FP8 rescaling.

**CN**: `verify()` 在主机端重建了与设备端一致的 epilogue 数学语义。它先把可选的 A/B/C 缩放折叠进 `scaled_alpha` 与 `scaled_beta`，再执行一次参考 GEMM 得到临时累加结果，随后叠加广播 bias、应用 ReLU、执行可选的 D/Aux 缩放，并归约出绝对值最大值。理解该示例数学含义时，这一段最关键：辅助张量保存激活前的值，而 D 保存 ReLU 之后的值，两者都可能再被重新缩放为 FP8。

### Lines 544-755 — Capability checks, kernel argument assembly, launch, and profiling

```cpp
  }

  /// Returns true if the CUDA device is sufficient to execute the kernel.
  bool sufficient() const {

    if (__CUDACC_VER_MAJOR__ < 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ < 4)) {
      std::cerr << "This example requires CUDA 12.4 or greater." << std::endl;
      return false;
    }

    size_t smem_size = sizeof(typename Gemm::GemmKernel::SharedStorage);

    cudaDeviceProp properties;
    int device_idx;
    cudaError_t result = cudaGetDevice(&device_idx);

    if (result != cudaSuccess) {
      std::cerr << "cudaGetDevice() failed with error: " << cudaGetErrorString(result) << std::endl;
      return false;
    }

    result = cudaGetDeviceProperties(&properties, device_idx);

    if (result != cudaSuccess) {
      std::cerr << "cudaGetDeviceProperties() failed with error: " << cudaGetErrorString(result) << std::endl;
      return false;
    }

    if (properties.major < 8 || (properties.major == 8 && properties.minor < 9)) {
      std::cerr << "CUTLASS's Ada FP8 GEMM example requires a device of compute capability 89 or higher.\n" << std::endl;
      return false;
    }

    if (properties.sharedMemPerBlockOptin < smem_size) {
      std::cerr << "Insufficient shared memory. Need " << smem_size
                << ", but device only has " << properties.sharedMemPerBlockOptin << std::endl;
      return false;
    }

    return true;
  }

  /// Executes one test
  bool run(Options& options)
  {

    // Waive test if insufficient CUDA device
    if (!sufficient()) {
      std::cerr << "Insufficient resources to run the kernel." << std::endl;
      return false;
    }

    this->initialize(options);

    //
    // Initialize the GEMM operator
    //

    typename Gemm::EpilogueOutputOp::Params::ActivationParams activation_params{
      ElementCompute(options.alpha),
      ElementCompute(options.beta)
    };
    typename Gemm::EpilogueOutputOp::Params epilogue_params{
      activation_params,
      scale_A.device_data(),
      scale_B.device_data(),
      scale_C.device_data(),
      scale_D.device_data(),
      scale_Aux.device_data(),
      abs_max_Aux.device_data(),
      abs_max_D.device_data()
    };

    typename Gemm::Arguments arguments{
      cutlass::gemm::GemmUniversalMode::kGemm,
      options.problem_size,
      /* batch_count = */ 1,
      epilogue_params,
      tensor_A.device_data(),
      tensor_B.device_data(),
      tensor_C.device_data(),
      tensor_D.device_data(),
      tensor_Aux.device_data(),
      tensor_Vector.device_data(),
      options.problem_size.m() * options.problem_size.k(),
      options.problem_size.n() * options.problem_size.k(),
      options.problem_size.m() * options.problem_size.n(),
      options.problem_size.m() * options.problem_size.n(),
      (int)options.problem_size.m(), // Batch stride vector
      tensor_A.layout().stride(0),
      tensor_B.layout().stride(0),
      tensor_C.layout().stride(0),
      tensor_D.layout().stride(0),
      (int64_t)0 // Leading dimension of vector. This must be 0
    };

    Gemm gemm_op;

    cutlass::Status status = gemm_op.can_implement(arguments);
    if (status != cutlass::Status::kSuccess) {
      std::cerr << "Gemm::can_implement() failed" << std::endl;
      return false;
    }

    size_t workspace_size = Gemm::get_workspace_size(arguments);
    cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);

    status = gemm_op.initialize(arguments, workspace.get());
    if (status != cutlass::Status::kSuccess) {
      std::cerr << "Gemm::initialize() failed" << std::endl;
      return false;
    }

    //
    // Run the GEMM
    //

    status = gemm_op();

    if (status != cutlass::Status::kSuccess) {
      std::cerr << "Gemm::run() failed" << std::endl;
      return false;
    }

    cudaError_t cuda_error = cudaDeviceSynchronize();
    if (cuda_error != cudaSuccess) {
      std::cerr << "CUDA error: " << cudaGetErrorString(cuda_error) << std::endl;
      return false;
    }

    //
    // Verify
    //

    bool passed = true;
    if (options.reference_check) {
      passed &= this->verify(options);
    } else {
      std::cout << "Skipped reference check" << std::endl;
    }

    //
    // Warm up
    //

    for (int i = 0; i < options.warmup_iterations; ++i) {
      gemm_op();
    }

    //
    // Profile
    //

    cudaEvent_t events[2];
    cudaError_t error;
    for (auto & event : events) {
      error = cudaEventCreate(&event);
      if (error != cudaSuccess) {
        std::cerr << "cudaEventCreate() failed: " << cudaGetErrorString(error) << std::endl;
        return false;
      }
    }

    // Record an event at the start of a series of GEMM operations
    error = cudaEventRecord(events[0]);
    if (error != cudaSuccess) {
      std::cerr << "cudaEventRecord() failed: " << cudaGetErrorString(error) << std::endl;
      return false;
    }

    // Run profiling loop
    for (int iter = 0; iter < options.iterations; ++iter) {
      gemm_op();
    }

    // Record an event when the GEMM operations have been launched.
    error = cudaEventRecord(events[1]);
    if (error != cudaSuccess) {
      std::cerr << "cudaEventRecord() failed: " << cudaGetErrorString(error) << std::endl;
      return false;
    }

    // Wait for work on the device to complete.
    error = cudaEventSynchronize(events[1]);
    if (error != cudaSuccess) {
      std::cerr << "cudaEventSynchronize() failed: " << cudaGetErrorString(error) << std::endl;
      return false;
    }

    // Measure elapsed runtime
    float runtime_ms = 0;
    error = cudaEventElapsedTime(&runtime_ms, events[0], events[1]);
    if (error != cudaSuccess) {
      std::cerr << "cudaEventElapsed() failed: " << cudaGetErrorString(error) << std::endl;
      return false;
    }

    // Compute average runtime and GFLOPs.
    runtime_ms = runtime_ms / float(options.iterations);
    float gflops = options.gflops(runtime_ms / 1000.0f);

    std::cout << "Problem size: " << options.problem_size.m() << 'x' << options.problem_size.n() << 'x' << options.problem_size.k() << std::endl;
    std::cout << "Runtime (ms): " << runtime_ms << std::endl;
    std::cout << "GFLOPs/sec:   " << gflops << std::endl;

    // Cleanup
    for (auto event : events) {
      (void)cudaEventDestroy(event);
    }

    return passed;
  }
```

**EN**: The `sufficient()` method enforces Ada-era requirements: CUDA 12.4+, compute capability 8.9+, and enough opt-in shared memory for the kernel’s shared-storage object. `run()` then builds the epilogue parameter pack, including pointers to scale and absmax tensors, constructs `Gemm::Arguments`, and intentionally passes a leading dimension of zero for the vector so the epilogue treats it as a broadcast along M. After initialization and one correctness run, it optionally verifies the result, performs warmup launches, times repeated kernel calls with CUDA events, and reports runtime plus GFLOP/s.

**CN**: `sufficient()` 先检查 Ada 路径所需的运行条件：CUDA 12.4 及以上、计算能力 8.9 及以上，以及足够大的 opt-in 共享内存。随后 `run()` 组装 epilogue 参数包，把缩放因子与 absmax 张量指针传给内核，并构造 `Gemm::Arguments`。其中向量的 leading dimension 被故意设为 0，这样 epilogue 就会把它解释为沿 M 维广播。完成初始化和首次执行后，代码会按需做参考校验、进行预热、再用 CUDA event 对多次 kernel 调用计时，并输出运行时间与 GFLOP/s。

### Lines 756-832 — Program entry and staged-vs-fast accumulation comparison

```cpp

};

/////////////////////////////////////////////////////////////////////////////////////////////////

int main(int argc, char const** argv) {

  cudaDeviceProp props;

  cudaError_t error = cudaGetDeviceProperties(&props, 0);
  if (error != cudaSuccess) {
    std::cerr << "cudaGetDeviceProperties() returned an error: " << cudaGetErrorString(error) << std::endl;
    return -1;
  }

  bool satisfied;
  if (props.major < 10) {
  }
  else {
    satisfied = (__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 8);
  }

  if (!satisfied) {
    //
    // This example requires an NVIDIA GPU with compute capability 8.9 or greater.
    //

    std::cout
      << "CUTLASS's FP8 SM89 example requires an NVIDIA GPU with compute capability 8.9 or greater "
      << "and CUDA toolkit version 12.4 or later"
      << std::endl;

    return 0;
  }

  //
  // Parse options
  //

  Options options;

  options.parse(argc, argv);

  if (options.help) {
    options.print_usage(std::cout) << std::endl;
    return 0;
  }

  if (options.error) {
    std::cerr << "Aborting execution." << std::endl;
    return -1;
  }

  std::cout << "Running GEMM with staged accumulation (OpMultiplyAdd)" << std::endl;
  std::cout << "=====================================================" << std::endl;
  TestbedRunner<Gemm_<cutlass::arch::OpMultiplyAdd>> testbed_staged_accum;
  bool passed = testbed_staged_accum.run(options);

  if (passed) {
    std::cout << "Passed" << std::endl;
  } else {
    std::cout << "Failed" << std::endl;
  }

  std::cout << "\nRunning GEMM with fast accumulation (OpMultiplyAddFastAccum)" << std::endl;
  std::cout << "============================================================" << std::endl;
  TestbedRunner<Gemm_<cutlass::arch::OpMultiplyAddFastAccum>> testbed_fast_accum;
  passed = testbed_fast_accum.run(options);

  if (passed) {
    std::cout << "Passed" << std::endl;
  } else {
    std::cout << "Failed" << std::endl;
  }

  return 0;
}
```

**EN**: The `main()` function checks device properties, parses user options, and runs the same host harness twice with two math-operator tags. The first instantiation uses `OpMultiplyAdd`, which is the more conservative staged-accumulation path, and the second uses `OpMultiplyAddFastAccum`, which is tuned for Ada FP8 throughput. This makes the example useful not only as a correctness sample, but also as a small A/B experiment for the accumulation strategy CUTLASS exposes on Ada.

**CN**: `main()` 负责检查设备属性、解析用户参数，并用两种数学运算符标签分别运行同一套测试流程。第一次实例化使用 `OpMultiplyAdd`，对应更保守的 staged accumulation；第二次使用 `OpMultiplyAddFastAccum`，对应更强调 Ada FP8 吞吐的快速累加路径。因此，这个示例不仅是正确性样例，也是一个很直观的 CUTLASS Ada 累加策略对比实验。

---

## Key Concepts / 关键概念

- **FP8 scaling pipeline**
  - **EN**: Ada FP8 kernels often store data in E4M3 but compute in higher precision. This sample explicitly models input scaling, output scaling, and absmax collection so quantization can be managed outside the GEMM core.
  - **CN**: Ada FP8 内核通常以 E4M3 存储数据，但在更高精度下累加。本示例显式展示了输入缩放、输出缩放和 absmax 收集，便于在 GEMM 主体之外管理量化。
- **Auxiliary output**
  - **EN**: The auxiliary tensor stores the pre-activation value, which is useful for fused-training workflows that need both the activated output and the raw intermediate.
  - **CN**: 辅助输出张量保存激活前的值，这对既需要激活后输出、又需要原始中间结果的训练融合流程很重要。
- **Broadcast bias handling**
  - **EN**: The vector argument is encoded with zero leading dimension so one row is broadcast across all M rows in the epilogue.
  - **CN**: 向量参数通过 leading dimension 为 0 的方式编码，从而在 epilogue 中把一行数据广播到所有 M 行。
- **Staged vs fast accumulation**
  - **EN**: The two math operators expose the tradeoff between more conservative accumulation and Ada-optimized fast accumulation.
  - **CN**: 两种数学运算符分别体现了“更保守的累加方式”和“Ada 优化的快速累加方式”之间的权衡。

## Dependencies / 依赖项

- `cutlass/gemm/device/gemm_universal_with_absmax.h`
  - **EN**: Provides the Ada-oriented GEMM wrapper that can emit absmax values together with the main output.
  - **CN**: 提供支持输出 absmax 的 Ada 定向 GEMM 设备封装。
- `cutlass/epilogue/thread/linear_combination_generic_with_scaling.h`
  - **EN**: Defines the fused epilogue operator used to apply scaling, activation, auxiliary writes, and amax tracking.
  - **CN**: 定义融合 epilogue 运算符，用于执行缩放、激活、辅助写回和 amax 统计。
- CUTLASS host/reference utilities
  - **EN**: HostTensor, tensor fill/copy helpers, and reference GEMM utilities are used to build a deterministic correctness harness.
  - **CN**: HostTensor、张量填充/拷贝辅助函数和参考 GEMM 工具共同构成可复现的正确性测试床。
- CUDA runtime events
  - **EN**: CUDA events provide the timing path used for benchmarking the staged and fast accumulation kernels.
  - **CN**: CUDA event 为 staged 与 fast accumulation 两个 kernel 提供计时基线。
