# gemm_testbed_3x_tensor_broadcast.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/unit/gemm/device/gemm_testbed_3x_tensor_broadcast.hpp`
- **Purpose / 用途:** GEMM testbed variant that validates tensor-broadcast inputs and epilogue behavior.

## Line-by-Line Analysis / 逐行分析

### Lines 1-33

```cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
    \brief Tests for device-wide GEMM interface with elementwise tensor-tensor broadcast epilogue
*/
```
- **EN:** Provides the standard BSD-3-Clause license header and ownership notice for this source file.
- **CN:** 给出该源文件的标准 BSD-3-Clause 许可证头和版权归属说明。

### Lines 35-35

```cpp
#pragma once
```
- **EN:** Uses `#pragma once` to ensure the header is included only once per translation unit.
- **CN:** 使用 `#pragma once`，确保该头文件在一个编译单元中只被包含一次。

### Lines 37-39

```cpp
#include <iostream>
#include <fstream>
#include <sstream>
```
- **EN:** Imports dependencies such as `iostream`, `fstream`, `sstream` so the file can access the required CUTLASS/CUTE APIs and shared test utilities.
- **CN:** 引入依赖头文件，例如 `iostream`, `fstream`, `sstream`，使本文件能够使用所需的 CUTLASS/CUTE API 与共享测试工具。

### Lines 41-41

```cpp
#include "../../common/cutlass_unit_test.h"
```
- **EN:** Imports dependencies such as `cutlass_unit_test.h` so the file can access the required CUTLASS/CUTE APIs and shared test utilities.
- **CN:** 引入依赖头文件，例如 `cutlass_unit_test.h`，使本文件能够使用所需的 CUTLASS/CUTE API 与共享测试工具。

### Lines 43-44

```cpp
#include "testbed_utils.h"
#include "gemm_testbed_3x.hpp"
```
- **EN:** Imports dependencies such as `testbed_utils.h`, `gemm_testbed_3x.hpp` so the file can access the required CUTLASS/CUTE APIs and shared test utilities.
- **CN:** 引入依赖头文件，例如 `testbed_utils.h`, `gemm_testbed_3x.hpp`，使本文件能够使用所需的 CUTLASS/CUTE API 与共享测试工具。

### Lines 46-48

```cpp
namespace test {
namespace gemm {
namespace device {
```
- **EN:** Opens or closes namespaces so the declarations live in the expected CUTLASS or test scope.
- **CN:** 打开或关闭命名空间，使这些声明位于预期的 CUTLASS 或测试作用域中。

### Lines 50-50

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 52-53

```cpp
template <typename Gemm>
struct Testbed3xTensorBroadcast {
```
- **EN:** Defines templated type `Testbed3xTensorBroadcast` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `Testbed3xTensorBroadcast`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 55-57

```cpp
  using TestBedImpl = typename detail::TestbedImpl<Gemm>;
  using Kernel      = typename Gemm::GemmKernel;
  using Epilogue    = typename Gemm::GemmKernel::CollectiveEpilogue;
```
- **EN:** Defines aliases such as `TestBedImpl`, `Kernel`, `Epilogue` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `TestBedImpl`, `Kernel`, `Epilogue`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 59-66

```cpp
  using ElementA = typename Kernel::ElementA;
  using StrideA  = typename Kernel::StrideA;
  using ElementB = typename Kernel::ElementB;
  using StrideB  = typename Kernel::StrideB;
  using ElementC = typename Kernel::ElementC;
  using StrideC  = typename Kernel::StrideC;
  using ElementD = typename Kernel::ElementD;
  using StrideD  = typename Kernel::StrideD;
```
- **EN:** Defines aliases such as `ElementA`, `StrideA`, `ElementB`, `StrideB`, `ElementC`, `StrideC`, ... (+2) to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ElementA`, `StrideA`, `ElementB`, `StrideB`, `ElementC`, `StrideC`, ... (+2)，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 68-73

```cpp
  using ElementAccumulator   = typename Kernel::ElementAccumulator;
  using ElementCompute       = typename Epilogue::ElementCompute;
  using ElementScalar        = typename Epilogue::ElementScalar;
  using ProblemShapeType     = typename Kernel::ProblemShape;
  using ElementBias          = typename Epilogue::ElementBias;
  using ActivationFunctor    = typename Epilogue::ActivationFunctor;
```
- **EN:** Defines aliases such as `ElementAccumulator`, `ElementCompute`, `ElementScalar`, `ProblemShapeType`, `ElementBias`, `ActivationFunctor` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ElementAccumulator`, `ElementCompute`, `ElementScalar`, `ProblemShapeType`, `ElementBias`, `ActivationFunctor`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 75-77

```cpp
  static constexpr bool IsBinaryOp0Enabled = Epilogue::IsBinaryOp0Enabled;
  static constexpr bool IsBinaryOp1Enabled = Epilogue::IsBinaryOp1Enabled;
  static constexpr bool IsUnaryOpEnabled   = Epilogue::IsUnaryOpEnabled;
```
- **EN:** Declares member fields or local variables related to `IsBinaryOp0Enabled`, `Epilogue`, `IsBinaryOp1Enabled`, `IsUnaryOpEnabled` for later setup, execution, or verification.
- **CN:** 声明与 `IsBinaryOp0Enabled`, `Epilogue`, `IsBinaryOp1Enabled`, `IsUnaryOpEnabled` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 79-79

```cpp
  static constexpr bool PerColBias = Epilogue::PerColumnBias;
```
- **EN:** Declares member fields or local variables related to `PerColBias`, `Epilogue`, `PerColumnBias` for later setup, execution, or verification.
- **CN:** 声明与 `PerColBias`, `Epilogue`, `PerColumnBias` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 81-85

```cpp
  using LayoutTagA = typename TestBedImpl::LayoutTagA;
  using LayoutTagB = typename TestBedImpl::LayoutTagB;
  using LayoutTagC = typename TestBedImpl::LayoutTagC;
  using LayoutTagD = typename TestBedImpl::LayoutTagD;
  using LayoutTagVector = cutlass::layout::PackedVectorLayout;
```
- **EN:** Defines aliases such as `LayoutTagA`, `LayoutTagB`, `LayoutTagC`, `LayoutTagD`, `LayoutTagVector` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `LayoutTagA`, `LayoutTagB`, `LayoutTagC`, `LayoutTagD`, `LayoutTagVector`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 87-89

```cpp
  cutlass::HostTensor<ElementBias, LayoutTagVector> bias;
  cutlass::HostTensor<ElementC, LayoutTagC> tensor_C1;
  // tensor_C0 is taken from TestbedImpl's tensor_C
```
- **EN:** Implements or wires together logic around `HostTensor`, `ElementBias`, `LayoutTagVector`, `bias`, `ElementC` for the current test scenario.
- **CN:** 围绕 `HostTensor`, `ElementBias`, `LayoutTagVector`, `bias`, `ElementC` 实现或连接当前测试场景所需的逻辑。

### Lines 92-93

```cpp
  // Detail Implementation
  TestBedImpl impl_;
```
- **EN:** Implements or wires together logic around `Detail`, `Implementation`, `TestBedImpl`, `impl_` for the current test scenario.
- **CN:** 围绕 `Detail`, `Implementation`, `TestBedImpl`, `impl_` 实现或连接当前测试场景所需的逻辑。

### Lines 95-105

```cpp
  //
  // Methods
  //
  Testbed3xTensorBroadcast(
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_C_ = cutlass::Distribution::Uniform,
    uint64_t seed_ = TestBedImpl::kDefaultSeed
  ) :
    impl_(CheckEquality::EXACT, ScalarLoc::ON_DEVICE, VectorScale::ENABLED,
          init_A_, init_B_, init_C_, cutlass::Distribution::Uniform, cutlass::Distribution::Uniform, seed_) { }
```
- **EN:** Implements or wires together logic around `Methods`, `Testbed3xTensorBroadcast`, `Distribution`, `Kind`, `init_A_` for the current test scenario.
- **CN:** 围绕 `Methods`, `Testbed3xTensorBroadcast`, `Distribution`, `Kind`, `init_A_` 实现或连接当前测试场景所需的逻辑。

### Lines 107-124

```cpp
  Testbed3xTensorBroadcast(
    typename LayoutTagA::Stride stride_factor_A_,
    typename LayoutTagB::Stride stride_factor_B_,
    typename LayoutTagC::Stride stride_factor_C_,
    typename LayoutTagD::Stride stride_factor_D_,
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_C_ = cutlass::Distribution::Uniform,
    uint64_t seed_ = TestBedImpl::kDefaultSeed
  ) :
    impl_(stride_factor_A_,
          stride_factor_B_,
          stride_factor_C_,
          stride_factor_D_,
          CheckEquality::EXACT, ScalarLoc::ON_HOST, VectorScale::ENABLED,
          init_A_,
          init_B_,
          init_C_,
```
- **EN:** Implements or wires together logic around `Testbed3xTensorBroadcast`, `LayoutTagA`, `Stride`, `stride_factor_A_`, `LayoutTagB` for the current test scenario.
- **CN:** 围绕 `Testbed3xTensorBroadcast`, `LayoutTagA`, `Stride`, `stride_factor_A_`, `LayoutTagB` 实现或连接当前测试场景所需的逻辑。

### Lines 125-127

```cpp
          cutlass::Distribution::Uniform,
          cutlass::Distribution::Uniform,
          seed_) { }
```
- **EN:** Implements or wires together logic around `Distribution`, `Uniform`, `seed_` for the current test scenario.
- **CN:** 围绕 `Distribution`, `Uniform`, `seed_` 实现或连接当前测试场景所需的逻辑。

### Lines 129-135

```cpp
  /// Initializes data structures
  void initialize(ProblemShapeType problem_size) {
    //
    // Allocate the GEMM workspace for A/B/C/D tensor
    //
    impl_.initialize(problem_size);
  }
```
- **EN:** Implements or wires together logic around `Initializes`, `data`, `structures`, `initialize`, `ProblemShapeType` for the current test scenario.
- **CN:** 围绕 `Initializes`, `data`, `structures`, `initialize`, `ProblemShapeType` 实现或连接当前测试场景所需的逻辑。

### Lines 137-140

```cpp
  void initialize_bias(ProblemShapeType problem_size) {
    auto problem_shape_MNKL = cute::append<4>(problem_size, 1);
    auto bias_size = PerColBias ? cute::get<1>(problem_shape_MNKL) : cute::get<0>(problem_shape_MNKL);
    bias.resize(cutlass::Coord<1>(bias_size));
```
- **EN:** Begins function or method `initialize_bias`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `initialize_bias`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 142-144

```cpp
    EXPECT_TRUE(detail::initialize_tensor(bias.host_view(), cutlass::Distribution::Uniform, impl_.collective_mma_inputs.seed + 2023));
    bias.sync_device();
  }
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 146-150

```cpp
  void initialize_c1(ProblemShapeType problem_size) {
    auto problem_shape_MNKL = cute::append<4>(problem_size, 1);
    auto M = cute::get<0>(problem_shape_MNKL);
    auto N = cute::get<1>(problem_shape_MNKL);
    auto L = cute::get<3>(problem_shape_MNKL);
```
- **EN:** Begins function or method `initialize_c1`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `initialize_c1`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 152-152

```cpp
    auto c_coord = cutlass::make_Coord(M * L, N);
```
- **EN:** Declares member fields or local variables related to `c_coord`, `make_Coord` for later setup, execution, or verification.
- **CN:** 声明与 `c_coord`, `make_Coord` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 154-157

```cpp
    tensor_C1.resize(c_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagD>::layout_factory(c_coord, impl_.collective_epilogue.stride_factor_C));
    EXPECT_TRUE(detail::initialize_tensor(tensor_C1.host_view(), cutlass::Distribution::Uniform, impl_.collective_mma_inputs.seed + 2024));
    tensor_C1.sync_device();
  }
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 159-166

```cpp
  /// Compares computed reference with device reference and outputs to a file if incorrect
  bool compare_reference(
      cute::Shape<int,int,int,int> problem_shape_MNKL,
      ElementScalar alpha,
      ElementScalar beta,
      bool use_bias)
  {
    auto [M, N, K, L] = problem_shape_MNKL;
```
- **EN:** Implements or wires together logic around `Compares`, `computed`, `reference`, `with`, `device` for the current test scenario.
- **CN:** 围绕 `Compares`, `computed`, `reference`, `with`, `device` 实现或连接当前测试场景所需的逻辑。

### Lines 168-170

```cpp
    impl_.collective_epilogue.tensor_D.sync_host();
    EXPECT_GT(cutlass::reference::host::TensorNorm(impl_.collective_mma_inputs.tensor_A.host_view()), 0);
    EXPECT_GT(cutlass::reference::host::TensorNorm(impl_.collective_mma_inputs.tensor_B.host_view()), 0);
```
- **EN:** Declares member fields or local variables related to `impl_`, `collective_epilogue`, `tensor_D`, `sync_host`, `EXPECT_GT` for later setup, execution, or verification.
- **CN:** 声明与 `impl_`, `collective_epilogue`, `tensor_D`, `sync_host`, `EXPECT_GT` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 172-174

```cpp
    if (impl_.collective_epilogue.tensor_D.size() > 1) {
      EXPECT_GT(cutlass::reference::host::TensorNorm(impl_.collective_epilogue.tensor_D.host_view()), 0);
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 176-178

```cpp
    if (impl_.collective_epilogue.reference_D.size() > 1) {
      EXPECT_GT(cutlass::reference::host::TensorNorm(impl_.collective_epilogue.reference_D.host_view()), 0);
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 180-180

```cpp
    bool passed = cutlass::reference::host::TensorEquals(impl_.collective_epilogue.reference_D.host_view(), impl_.collective_epilogue.tensor_D.host_view());
```
- **EN:** Declares member fields or local variables related to `passed`, `reference`, `host`, `TensorEquals`, `impl_` for later setup, execution, or verification.
- **CN:** 声明与 `passed`, `reference`, `host`, `TensorEquals`, `impl_` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 182-182

```cpp
    EXPECT_TRUE(passed);
```
- **EN:** Declares member fields or local variables related to `EXPECT_TRUE`, `passed` for later setup, execution, or verification.
- **CN:** 声明与 `EXPECT_TRUE`, `passed` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 184-190

```cpp
    if (!passed) {
      std::stringstream fname;
      fname << "error_Gemm_device_broadcast"
        << M << "x" << N << "x" << K << "x" << L << "_"
        << cute::get<0>(typename Gemm::GemmKernel::TileShape{}) << "_"
        << cute::get<1>(typename Gemm::GemmKernel::TileShape{}) << "_"
        << cute::get<2>(typename Gemm::GemmKernel::TileShape{}) << ".txt";
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 192-196

```cpp
      std::ofstream file(fname.str());
      file
        << "problem: " << ' ' << M << "x" << N << "x" << K << ", Batch count = " << L
        << ", alpha: " << float(alpha) << ", beta: " << float(beta) << ", use_bias: " << use_bias 
        << ", per-col bias: " << PerColBias << "\n\n";
```
- **EN:** Implements or wires together logic around `std`, `ofstream`, `file`, `fname`, `str` for the current test scenario.
- **CN:** 围绕 `std`, `ofstream`, `file`, `fname`, `str` 实现或连接当前测试场景所需的逻辑。

### Lines 198-200

```cpp
      if (use_bias){
        file << "Bias = \n" << bias.host_view()<< "\n\n";
      }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 202-209

```cpp
      file
        << "A =\n" << impl_.collective_mma_inputs.tensor_A.host_view()
        << "\nB =\n" << impl_.collective_mma_inputs.tensor_B.host_view()
        << "\nC0 =\n" << impl_.collective_epilogue.tensor_C.host_view()
        << "\nC1 =\n" << tensor_C1.host_view()
        << "\n\nReference =\n" << impl_.collective_epilogue.reference_D.host_view()
        << "\n\nComputed =\n" <<impl_.collective_epilogue.tensor_D.host_view();
    }
```
- **EN:** Implements or wires together logic around `file`, `impl_`, `collective_mma_inputs`, `tensor_A`, `host_view` for the current test scenario.
- **CN:** 围绕 `file`, `impl_`, `collective_mma_inputs`, `tensor_A`, `host_view` 实现或连接当前测试场景所需的逻辑。

### Lines 211-212

```cpp
    return passed;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 214-226

```cpp
  /// Verifies the result matches the GEMM with elementwise tensor-tensor
  /// broadcast operation
  bool verify(
    ProblemShapeType problem_size,
    ElementScalar alpha,
    ElementScalar beta,
    bool use_bias)
  {
    auto problem_shape_MNKL = cute::append<4>(problem_size, 1);
    auto M = cute::get<0>(problem_shape_MNKL);
    auto N = cute::get<1>(problem_shape_MNKL);
    auto K = cute::get<2>(problem_shape_MNKL);
    auto L = cute::get<3>(problem_shape_MNKL);
```
- **EN:** Implements or wires together logic around `Verifies`, `the`, `result`, `matches`, `GEMM` for the current test scenario.
- **CN:** 围绕 `Verifies`, `the`, `result`, `matches`, `GEMM` 实现或连接当前测试场景所需的逻辑。

### Lines 228-239

```cpp
    auto A = cute::make_tensor(impl_.collective_mma_inputs.tensor_A.host_data(),
        cute::make_layout(cute::make_shape(M, K, L), impl_.collective_mma_inputs.stride_a));
    auto B = cute::make_tensor(impl_.collective_mma_inputs.tensor_B.host_data(),
        cute::make_layout(cute::make_shape(N, K, L), impl_.collective_mma_inputs.stride_b));
    auto D = cute::make_tensor(impl_.collective_epilogue.reference_D.host_data(),
        cute::make_layout(cute::make_shape(M, N, L), impl_.collective_epilogue.stride_d));
    auto Bias = cute::make_tensor(static_cast<ElementBias*>(use_bias ? bias.host_data() : nullptr),
        cute::make_layout(PerColBias ? cute::make_shape(1, N) : cute::make_shape(M, 1)));
    auto C0 = cute::make_tensor(impl_.collective_epilogue.tensor_C.host_data(),
        cute::make_layout(cute::make_shape(M, N, L), impl_.collective_epilogue.stride_c));
    auto C1 = cute::make_tensor(tensor_C1.host_data(),
        cute::make_layout(cute::make_shape(M, N, L), impl_.collective_epilogue.stride_c));
```
- **EN:** Implements or wires together logic around `make_tensor`, `impl_`, `collective_mma_inputs`, `tensor_A`, `host_data` for the current test scenario.
- **CN:** 围绕 `make_tensor`, `impl_`, `collective_mma_inputs`, `tensor_A`, `host_data` 实现或连接当前测试场景所需的逻辑。

### Lines 241-247

```cpp
    // Create host workspace for output of testbed. This computes a portion of the epilogue:
    //    ref_compute_out = Activation(alpha * (A @ B) + bias)
    cutlass::HostTensor<ElementCompute, LayoutTagC> ref_compute_out;
    auto c_coord = cutlass::make_Coord(M * L, N);
    ref_compute_out.resize(c_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagD>::layout_factory(c_coord, impl_.collective_epilogue.stride_factor_C), false);
    auto RefComputeOut = cute::make_tensor(ref_compute_out.host_data(),
        cute::make_layout(cute::make_shape(M, N, L), impl_.collective_epilogue.stride_c));
```
- **EN:** Implements or wires together logic around `Create`, `host`, `workspace`, `output`, `testbed` for the current test scenario.
- **CN:** 围绕 `Create`, `host`, `workspace`, `output`, `testbed` 实现或连接当前测试场景所需的逻辑。

### Lines 249-249

```cpp
    cutlass::reference::host::GettMainloopParams<ElementAccumulator, decltype(A), decltype(B)> mainloop_params{A, B};
```
- **EN:** Declares member fields or local variables related to `reference`, `host`, `GettMainloopParams`, `ElementAccumulator`, `decltype` for later setup, execution, or verification.
- **CN:** 声明与 `reference`, `host`, `GettMainloopParams`, `ElementAccumulator`, `decltype` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 251-260

```cpp
    // Use a dummy null tensor for operand C because the epilogue overrides C.
    auto dummy_C = cute::make_tensor(static_cast<ElementC*>(nullptr),
        cute::make_layout(cute::make_shape(M, N, L), impl_.collective_epilogue.stride_c));
    ElementCompute dummy_beta(0);
    auto dummy_Aux = cute::make_tensor(static_cast<ElementD*>(nullptr),
        cute::make_layout(cute::make_shape(M, N, L), impl_.collective_epilogue.stride_d));
    auto dummy_Valpha = cute::make_tensor(static_cast<ElementCompute*>(nullptr),
        cute::make_layout(cute::make_shape(M, N, 1), cute::make_stride(cute::_1{}, cute::_0{}, M)));
    auto dummy_Vbeta = cute::make_tensor(static_cast<ElementCompute*>(nullptr),
        cute::make_layout(cute::make_shape(M, N, 1), cute::make_stride(cute::_1{}, cute::_0{}, M)));
```
- **EN:** Implements or wires together logic around `Use`, `dummy`, `null`, `tensor`, `operand` for the current test scenario.
- **CN:** 围绕 `Use`, `dummy`, `null`, `tensor`, `operand` 实现或连接当前测试场景所需的逻辑。

### Lines 262-264

```cpp
    auto dummy_SFD = cute::make_tensor(static_cast<ElementD*>(nullptr),
        cute::make_layout(cute::make_shape(M, N, L), impl_.collective_epilogue.stride_c));
    using DummySFDVectorSize = cute::Int<0>;
```
- **EN:** Implements or wires together logic around `dummy_SFD`, `make_tensor`, `static_cast`, `ElementD`, `nullptr` for the current test scenario.
- **CN:** 围绕 `dummy_SFD`, `make_tensor`, `static_cast`, `ElementD`, `nullptr` 实现或连接当前测试场景所需的逻辑。

### Lines 267-284

```cpp
    cutlass::reference::host::GettEpilogueParams<
        ElementScalar,
        ElementScalar,
        ElementAccumulator,
        ElementCompute,
        decltype(dummy_C),
        decltype(RefComputeOut),
        decltype(Bias),
        decltype(dummy_Aux),      
        decltype(dummy_Valpha),
        decltype(dummy_Vbeta),
        ActivationFunctor,
        decltype(dummy_SFD),            
        DummySFDVectorSize,             
        cutlass::plus<ElementCompute>,
        PerColBias> epilogue_params{
          alpha,
          dummy_beta,
```
- **EN:** Implements or wires together logic around `reference`, `host`, `GettEpilogueParams`, `ElementScalar`, `ElementAccumulator` for the current test scenario.
- **CN:** 围绕 `reference`, `host`, `GettEpilogueParams`, `ElementScalar`, `ElementAccumulator` 实现或连接当前测试场景所需的逻辑。

### Lines 285-291

```cpp
          dummy_C,
          RefComputeOut,
          Bias,
          dummy_Aux,
          dummy_Valpha,
          dummy_Vbeta
        };
```
- **EN:** Implements or wires together logic around `dummy_C`, `RefComputeOut`, `Bias`, `dummy_Aux`, `dummy_Valpha` for the current test scenario.
- **CN:** 围绕 `dummy_C`, `RefComputeOut`, `Bias`, `dummy_Aux`, `dummy_Valpha` 实现或连接当前测试场景所需的逻辑。

### Lines 293-293

```cpp
    cutlass::reference::host::Gemm3x(mainloop_params, epilogue_params);
```
- **EN:** Declares member fields or local variables related to `reference`, `host`, `Gemm3x`, `mainloop_params`, `epilogue_params` for later setup, execution, or verification.
- **CN:** 声明与 `reference`, `host`, `Gemm3x`, `mainloop_params`, `epilogue_params` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 295-297

```cpp
    cutlass::NumericConverter<ElementCompute, ElementC, Epilogue::ThreadEpilogueOp::kRound> source_converter;
    cutlass::NumericConverter<ElementD, ElementCompute, Epilogue::ThreadEpilogueOp::kRound> destination_converter;
    cutlass::multiplies<ElementCompute> mul;
```
- **EN:** Declares member fields or local variables related to `NumericConverter`, `ElementCompute`, `ElementC`, `Epilogue`, `ThreadEpilogueOp` for later setup, execution, or verification.
- **CN:** 声明与 `NumericConverter`, `ElementCompute`, `ElementC`, `Epilogue`, `ThreadEpilogueOp` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 299-310

```cpp
    // Compute broadcast operations atop the reference
    #pragma omp parallel for collapse(3)
    for (int64_t l = 0; l < cute::size<2>(A.layout()); ++l) {
      for (int64_t m = 0; m < cute::size<0>(A.layout()); ++m) {
        for (int64_t n = 0; n < cute::size<0>(B.layout()); ++n) {
          ElementCompute intermediate = RefComputeOut(m, n, l);
          // Apply BinaryOp0, if needed
          if constexpr (IsBinaryOp0Enabled) {
            typename Epilogue::ThreadEpilogueOp::BinaryOp0 bin0;
            ElementCompute converted_source = source_converter(C0(m, n, l));
            intermediate = bin0(intermediate, mul(beta, converted_source));
          }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 312-317

```cpp
          // Apply BinaryOp1, if needed
          if constexpr (IsBinaryOp1Enabled) {
            typename Epilogue::ThreadEpilogueOp::BinaryOp1 bin1;
            ElementCompute converted_source = source_converter(C1(m, n, l));
            intermediate = bin1(intermediate, mul(beta, converted_source));
          }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 319-323

```cpp
          // Apply UnaryOp, if needed
          if constexpr (IsUnaryOpEnabled) {
            typename Epilogue::ThreadEpilogueOp::UnaryOp unary;
            intermediate = unary(intermediate);
          }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 325-328

```cpp
          D(m, n, l) = destination_converter(intermediate);
        }
      }
    }
```
- **EN:** Implements or wires together logic around `destination_converter`, `intermediate` for the current test scenario.
- **CN:** 围绕 `destination_converter`, `intermediate` 实现或连接当前测试场景所需的逻辑。

### Lines 330-331

```cpp
    return compare_reference(problem_shape_MNKL, alpha, beta, use_bias);
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 333-349

```cpp
  /// Executes one test
  bool run(
      ProblemShapeType problem_size,
      ElementScalar alpha = ElementScalar(1),
      ElementScalar beta = ElementScalar(0),
      bool profiling = false,
      int iterations = 20,
      bool use_bias = true)
  {
    // Fail test if insufficient CUDA device
    if (!impl_.sufficient()) {
      std::cout << "Test failed due to insufficient CUDA device." << std::endl;
      return false;
    }
    //
    // Initialize the GEMM operator
    //
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 351-361

```cpp
    typename Gemm::Arguments arguments;
    cutlass::KernelHardwareInfo hw_info;
    hw_info.device_id = 0;
    if (not profiling) {
      impl_.sm_count = std::min(impl_.MaxSmCount, cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id));
      hw_info.sm_count = impl_.sm_count;
    }
    else {
      impl_.sm_count = cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);
      hw_info.sm_count = impl_.sm_count;
    }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 363-366

```cpp
    /// Initializes data structures
    /// A/B/C0/D Tensor
    initialize(problem_size);
    initialize_bias(problem_size);
```
- **EN:** Implements or wires together logic around `Initializes`, `data`, `structures`, `Tensor`, `initialize` for the current test scenario.
- **CN:** 围绕 `Initializes`, `data`, `structures`, `Tensor`, `initialize` 实现或连接当前测试场景所需的逻辑。

### Lines 368-370

```cpp
    if constexpr (IsBinaryOp1Enabled) {
      initialize_c1(problem_size);
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 372-389

```cpp
    arguments = typename Gemm::Arguments{
      cutlass::gemm::GemmUniversalMode::kGemm,
        problem_size,
        { impl_.collective_mma_inputs.tensor_A.device_data(), impl_.collective_mma_inputs.stride_a,
          impl_.collective_mma_inputs.tensor_B.device_data(), impl_.collective_mma_inputs.stride_b,
          impl_.mma_promotion_interval
        },
        { // Epilogue arguments
          { alpha, beta }, // ThreadOp arguments
          impl_.collective_epilogue.stride_c,
          impl_.collective_epilogue.tensor_D.device_data(),
          impl_.collective_epilogue.stride_d,
          use_bias ? bias.device_data() : nullptr,
          impl_.collective_epilogue.tensor_C.device_data(),
          tensor_C1.device_data()
        }, // Epilogue arguments end
        hw_info
    };
```
- **EN:** Implements or wires together logic around `arguments`, `Gemm`, `Arguments`, `gemm`, `GemmUniversalMode` for the current test scenario.
- **CN:** 围绕 `arguments`, `Gemm`, `Arguments`, `gemm`, `GemmUniversalMode` 实现或连接当前测试场景所需的逻辑。

### Lines 391-391

```cpp
    Gemm gemm_op;
```
- **EN:** Declares member fields or local variables related to `Gemm`, `gemm_op` for later setup, execution, or verification.
- **CN:** 声明与 `Gemm`, `gemm_op` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 393-394

```cpp
    size_t workspace_size = Gemm::get_workspace_size(arguments);
    cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
```
- **EN:** Declares member fields or local variables related to `size_t`, `workspace_size`, `Gemm`, `get_workspace_size`, `arguments` for later setup, execution, or verification.
- **CN:** 声明与 `size_t`, `workspace_size`, `Gemm`, `get_workspace_size`, `arguments` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 396-396

```cpp
    cutlass::Status status = gemm_op.can_implement(arguments);
```
- **EN:** Declares member fields or local variables related to `Status`, `status`, `gemm_op`, `can_implement`, `arguments` for later setup, execution, or verification.
- **CN:** 声明与 `Status`, `status`, `gemm_op`, `can_implement`, `arguments` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 398-402

```cpp
    if (status != cutlass::Status::kSuccess) {
      cudaError_t error = cudaGetLastError();
      std::cerr << "This test is not supported: " << cudaGetErrorString(error) << "\n";
      return true;
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 404-406

```cpp
    //
    // Run the GEMM
    //
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 408-419

```cpp
    if (profiling) {
      return impl_.profile(problem_size, iterations, gemm_op, arguments, workspace);
    }
    else {
      cudaError_t result;
      status = gemm_op.initialize(arguments, workspace.get());
      status = gemm_op.run();
      result = cudaDeviceSynchronize();
      if (result != cudaSuccess) {
        EXPECT_EQ(result, cudaSuccess) << "Error at Kernel Sync.";
        return false;
      }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 421-421

```cpp
      EXPECT_TRUE(status == cutlass::Status::kSuccess) << to_string(status);
```
- **EN:** Declares member fields or local variables related to `EXPECT_TRUE`, `status`, `Status`, `kSuccess`, `to_string` for later setup, execution, or verification.
- **CN:** 声明与 `EXPECT_TRUE`, `status`, `Status`, `kSuccess`, `to_string` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 423-432

```cpp
      //
      // Verify
      //
      bool passed = this->verify(problem_size, alpha, beta, use_bias);
      if (!passed) {
        std::cout << "Error : Failed : with alpha: " << float(alpha)
                  << ", beta: " << float(beta)
                  << ", use_bias: " << use_bias
                  << "\n";
      }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 434-437

```cpp
      return passed;
    }
  }
};
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 439-439

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 441-444

```cpp
template <typename Gemm>
bool TestAllTensorBroadcast(bool use_bias=true) {
  using ElementScalar = typename Gemm::GemmKernel::CollectiveEpilogue::ElementScalar;
  using ProblemShapeType = typename Gemm::GemmKernel::ProblemShape;
```
- **EN:** Introduces type aliases like `ElementScalar`, `ProblemShapeType` so later declarations can refer to complex CUTLASS/CUTE types more clearly.
- **CN:** 引入类型别名，例如 `ElementScalar`, `ProblemShapeType`，让后续声明能够更清晰地引用复杂的 CUTLASS/CUTE 类型。

### Lines 446-448

```cpp
  int max_alignment = std::max(Gemm::kAlignmentA, Gemm::kAlignmentB);
  std::vector<int> problem_size_m = {max_alignment, 512 - 3 * max_alignment};
  std::vector<int> problem_size_n = {max_alignment, 512 - 2 * max_alignment};
```
- **EN:** Declares member fields or local variables related to `max_alignment`, `std`, `max`, `Gemm`, `kAlignmentA` for later setup, execution, or verification.
- **CN:** 声明与 `max_alignment`, `std`, `max`, `Gemm`, `kAlignmentA` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 450-454

```cpp
  if constexpr (cute::is_same_v<typename Gemm::GemmKernel::DispatchPolicy::Schedule,
                cutlass::gemm::KernelTmaWarpSpecializedPingpong>) {
    problem_size_m.push_back(768);
    problem_size_n.push_back(768);
  }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 456-457

```cpp
  constexpr int Stages = Gemm::GemmKernel::DispatchPolicy::Stages;
  constexpr int TileShapeK = cute::size<2>(typename Gemm::GemmKernel::TileShape{});
```
- **EN:** Declares member fields or local variables related to `Stages`, `Gemm`, `GemmKernel`, `DispatchPolicy`, `TileShapeK` for later setup, execution, or verification.
- **CN:** 声明与 `Stages`, `Gemm`, `GemmKernel`, `DispatchPolicy`, `TileShapeK` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 459-459

```cpp
  std::vector<int> problem_size_k = {max_alignment, TileShapeK * (Stages + 1) - max_alignment};
```
- **EN:** Declares member fields or local variables related to `std`, `vector`, `problem_size_k`, `max_alignment`, `TileShapeK` for later setup, execution, or verification.
- **CN:** 声明与 `std`, `vector`, `problem_size_k`, `max_alignment`, `TileShapeK` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 461-462

```cpp
  Testbed3xTensorBroadcast<Gemm> testbed;
  bool passed = true;
```
- **EN:** Declares member fields or local variables related to `Testbed3xTensorBroadcast`, `Gemm`, `testbed`, `passed` for later setup, execution, or verification.
- **CN:** 声明与 `Testbed3xTensorBroadcast`, `Gemm`, `testbed`, `passed` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 464-473

```cpp
  for (int m : problem_size_m) {
    for (int n : problem_size_n) {
      for (int k : problem_size_k) {
        ProblemShapeType problem_size;
        if constexpr (cute::rank(ProblemShapeType{}) == 4) {
          problem_size = ProblemShapeType{m, n, k, /* l */ 1};
        }
        else {
          problem_size = ProblemShapeType{m, n, k};
        }
```
- **EN:** Begins function or method `for`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `for`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 475-483

```cpp
        for (bool use_bias : {true, false}) {
          passed = testbed.run(
            problem_size,
            cutlass::from_real<ElementScalar>(1),
            cutlass::from_real<ElementScalar>(1),
            false,  // profiling
            20,     // iterations
            use_bias
          );
```
- **EN:** Begins function or method `for`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `for`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 485-491

```cpp
          if (!passed) {
            return false;
          }
        }
      }
    }
  }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 493-507

```cpp
  if constexpr (cute::rank(ProblemShapeType{}) == 4) {
    auto problem_size = ProblemShapeType{256 + max_alignment, 256 + max_alignment, 160 + max_alignment, /* l */ 3};
    passed = testbed.run(
      problem_size,
      cutlass::from_real<ElementScalar>(1),
      cutlass::from_real<ElementScalar>(1),
      false,  // profiling
      20      // iterations
    );
    if (!passed) {
      return false;
    }
  }
  return passed;
}
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 509-509

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 511-513

```cpp
} // namespace device
} // namespace gemm
} // namespace test
```
- **EN:** Opens or closes namespaces so the declarations live in the expected CUTLASS or test scope.
- **CN:** 打开或关闭命名空间，使这些声明位于预期的 CUTLASS 或测试作用域中。

### Lines 515-515

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

## Key Concepts / 关键概念

- **EN:** GEMM kernel configuration, launch, and correctness validation.  
  **CN:** GEMM 内核的配置、启动与正确性验证。
- **EN:** Reusable testbed infrastructure for tensor allocation, initialization, kernel launch, and reference comparison.  
  **CN:** 可复用测试平台基础设施，用于张量分配、初始化、内核启动与参考结果比较。
- **EN:** Broadcasted operands or auxiliary tensors.  
  **CN:** 广播操作数或辅助张量。
- **EN:** Epilogue fusion, callbacks, and post-processing composition.  
  **CN:** Epilogue 融合、回调与后处理组合。
- **EN:** GoogleTest-based unit validation.  
  **CN:** 基于 GoogleTest 的单元验证。

## Dependencies / 依赖关系

- `iostream`
- `fstream`
- `sstream`
- `../../common/cutlass_unit_test.h`
- `testbed_utils.h`
- `gemm_testbed_3x.hpp`
- CUTLASS template APIs and namespaces form the main external programming model / CUTLASS 模板 API 与命名空间构成主要外部编程模型
- CUTE supplies shape/layout metaprogramming primitives / CUTE 提供 shape/layout 元编程原语
