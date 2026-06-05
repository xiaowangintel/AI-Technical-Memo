# testbed_planar_complex.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/unit/gemm/device/testbed_planar_complex.h`
- **Purpose / 用途:** Planar complex GEMM testbed and comparison logic.

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
    \brief Tests for device-wide GEMM interface
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

### Lines 43-49

```cpp
#include "cutlass/util/distribution.h"
#include "cutlass/util/reference/host/gemm_planar_complex.h"
#include "cutlass/util/host_tensor_planar_complex.h"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_fill.h"
```
- **EN:** Imports dependencies such as `distribution.h`, `gemm_planar_complex.h`, `host_tensor_planar_complex.h`, `tensor_view_io.h`, `tensor_compare.h`, `tensor_copy.h`, ... (+1) so the file can access the required CUTLASS/CUTE APIs and shared test utilities.
- **CN:** 引入依赖头文件，例如 `distribution.h`, `gemm_planar_complex.h`, `host_tensor_planar_complex.h`, `tensor_view_io.h`, `tensor_compare.h`, `tensor_copy.h`, ... (+1)，使本文件能够使用所需的 CUTLASS/CUTE API 与共享测试工具。

### Lines 51-51

```cpp
////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 53-55

```cpp
namespace test {
namespace gemm {
namespace device {
```
- **EN:** Opens or closes namespaces so the declarations live in the expected CUTLASS or test scope.
- **CN:** 打开或关闭命名空间，使这些声明位于预期的 CUTLASS 或测试作用域中。

### Lines 57-57

```cpp
////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 59-61

```cpp
template <typename Gemm>
class TestbedPlanarComplex {
public:
```
- **EN:** Defines templated type `TestbedPlanarComplex` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `TestbedPlanarComplex`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 63-70

```cpp
  using ElementA = typename Gemm::ElementA;
  using LayoutA = typename Gemm::LayoutA;
  using ElementB = typename Gemm::ElementB;
  using LayoutB = typename Gemm::LayoutB;
  using ElementC = typename Gemm::ElementC;
  using LayoutC = typename Gemm::LayoutC;
  using ElementCompute = typename Gemm::EpilogueOutputOp::ElementCompute;
  using ElementAccumulator = typename Gemm::ElementAccumulator;
```
- **EN:** Defines aliases such as `ElementA`, `LayoutA`, `ElementB`, `LayoutB`, `ElementC`, `LayoutC`, ... (+2) to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ElementA`, `LayoutA`, `ElementB`, `LayoutB`, `ElementC`, `LayoutC`, ... (+2)，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 72-74

```cpp
  //
  // Data members
  //
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 76-81

```cpp
  cutlass::gemm::GemmCoord problem_size;
  cutlass::HostTensorPlanarComplex<ElementA, LayoutA> tensor_A;
  cutlass::HostTensorPlanarComplex<ElementB, LayoutB> tensor_B;
  cutlass::HostTensorPlanarComplex<ElementC, LayoutC> tensor_C;
  cutlass::HostTensorPlanarComplex<ElementC, LayoutC> tensor_D;
  cutlass::HostTensorPlanarComplex<ElementC, LayoutC> tensor_D_ref;
```
- **EN:** Declares member fields or local variables related to `gemm`, `GemmCoord`, `problem_size`, `HostTensorPlanarComplex`, `ElementA` for later setup, execution, or verification.
- **CN:** 声明与 `gemm`, `GemmCoord`, `problem_size`, `HostTensorPlanarComplex`, `ElementA` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 83-85

```cpp
  //
  // Methods
  //
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 87-87

```cpp
  TestbedPlanarComplex(cutlass::gemm::GemmCoord const & problem_size): problem_size(problem_size) {
```
- **EN:** Begins function or method `TestbedPlanarComplex`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `TestbedPlanarComplex`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 89-94

```cpp
    tensor_A.reset({problem_size.m(), problem_size.k()});
    tensor_B.reset({problem_size.k(), problem_size.n()});
    tensor_C.reset({problem_size.m(), problem_size.n()});
    tensor_D.reset({problem_size.m(), problem_size.n()});
    tensor_D_ref.reset({problem_size.m(), problem_size.n()}, false);
  }
```
- **EN:** Implements or wires together logic around `tensor_A`, `reset`, `problem_size`, `tensor_B`, `tensor_C` for the current test scenario.
- **CN:** 围绕 `tensor_A`, `reset`, `problem_size`, `tensor_B`, `tensor_C` 实现或连接当前测试场景所需的逻辑。

### Lines 96-96

```cpp
  void initialize() {
```
- **EN:** Begins function or method `initialize`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `initialize`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 98-98

```cpp
    uint64_t seed = 1073;
```
- **EN:** Declares member fields or local variables related to `uint64_t`, `seed` for later setup, execution, or verification.
- **CN:** 声明与 `uint64_t`, `seed` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 100-101

```cpp
    int scope_max = 8;
    int scope_min = -8;
```
- **EN:** Declares member fields or local variables related to `scope_max`, `scope_min` for later setup, execution, or verification.
- **CN:** 声明与 `scope_max`, `scope_min` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 103-104

```cpp
    cutlass::reference::host::TensorFillRandomUniform(
        tensor_A.host_view(), seed, scope_max, scope_min, 0);
```
- **EN:** Implements or wires together logic around `reference`, `host`, `TensorFillRandomUniform`, `tensor_A`, `host_view` for the current test scenario.
- **CN:** 围绕 `reference`, `host`, `TensorFillRandomUniform`, `tensor_A`, `host_view` 实现或连接当前测试场景所需的逻辑。

### Lines 106-107

```cpp
    cutlass::reference::host::TensorFillRandomUniform(
        tensor_B.host_view(), seed * 2019, scope_max, scope_min, 0);
```
- **EN:** Implements or wires together logic around `reference`, `host`, `TensorFillRandomUniform`, `tensor_B`, `host_view` for the current test scenario.
- **CN:** 围绕 `reference`, `host`, `TensorFillRandomUniform`, `tensor_B`, `host_view` 实现或连接当前测试场景所需的逻辑。

### Lines 109-110

```cpp
    cutlass::reference::host::TensorFillRandomUniform(
        tensor_C.host_view(), seed * 2020, scope_max, scope_min, 0);
```
- **EN:** Implements or wires together logic around `reference`, `host`, `TensorFillRandomUniform`, `tensor_C`, `host_view` for the current test scenario.
- **CN:** 围绕 `reference`, `host`, `TensorFillRandomUniform`, `tensor_C`, `host_view` 实现或连接当前测试场景所需的逻辑。

### Lines 112-113

```cpp
    cutlass::reference::host::TensorFill(tensor_D.host_view(), cutlass::complex<ElementC>());
    cutlass::reference::host::TensorFill(tensor_D_ref.host_view(), cutlass::complex<ElementC>());
```
- **EN:** Declares member fields or local variables related to `reference`, `host`, `TensorFill`, `tensor_D`, `host_view` for later setup, execution, or verification.
- **CN:** 声明与 `reference`, `host`, `TensorFill`, `tensor_D`, `host_view` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 115-119

```cpp
    tensor_A.sync_device();
    tensor_B.sync_device();
    tensor_C.sync_device();
    tensor_D.sync_device();
  }
```
- **EN:** Implements or wires together logic around `tensor_A`, `sync_device`, `tensor_B`, `tensor_C`, `tensor_D` for the current test scenario.
- **CN:** 围绕 `tensor_A`, `sync_device`, `tensor_B`, `tensor_C`, `tensor_D` 实现或连接当前测试场景所需的逻辑。

### Lines 121-125

```cpp
  /// Returns true if the CUDA device is sufficient to execute the kernel.
  bool sufficient() const {
    //
    // Determine SMEM requirements and waive if not satisfied
    //
```
- **EN:** Implements or wires together logic around `Returns`, `the`, `CUDA`, `device`, `sufficient` for the current test scenario.
- **CN:** 围绕 `Returns`, `the`, `CUDA`, `device`, `sufficient` 实现或连接当前测试场景所需的逻辑。

### Lines 127-127

```cpp
    size_t smem_size = sizeof(typename Gemm::GemmKernel::SharedStorage);
```
- **EN:** Declares member fields or local variables related to `size_t`, `smem_size`, `sizeof`, `Gemm`, `GemmKernel` for later setup, execution, or verification.
- **CN:** 声明与 `size_t`, `smem_size`, `sizeof`, `Gemm`, `GemmKernel` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 129-131

```cpp
    cudaDeviceProp properties;
    int device_idx;
    cudaError_t result = cudaGetDevice(&device_idx);
```
- **EN:** Declares member fields or local variables related to `cudaDeviceProp`, `properties`, `device_idx`, `cudaError_t`, `result` for later setup, execution, or verification.
- **CN:** 声明与 `cudaDeviceProp`, `properties`, `device_idx`, `cudaError_t`, `result` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 133-135

```cpp
    if (result != cudaSuccess) {
      throw std::runtime_error("cudaGetDevice() API call failed.");
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 137-137

```cpp
    result = cudaGetDeviceProperties(&properties, device_idx);
```
- **EN:** Declares member fields or local variables related to `result`, `cudaGetDeviceProperties`, `properties`, `device_idx` for later setup, execution, or verification.
- **CN:** 声明与 `result`, `cudaGetDeviceProperties`, `properties`, `device_idx` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 139-141

```cpp
    if (result != cudaSuccess) {
      throw std::runtime_error("cudaGetDeviceProperties() failed");
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 143-145

```cpp
    if (properties.sharedMemPerBlockOptin < smem_size) {
      return false;
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 147-148

```cpp
    return true;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 150-152

```cpp
  bool run(
      cutlass::complex<ElementCompute> alpha = {1, 0},
      cutlass::complex<ElementCompute> beta = {0, 0}) {
```
- **EN:** Implements or wires together logic around `run`, `complex`, `ElementCompute`, `alpha`, `beta` for the current test scenario.
- **CN:** 围绕 `run`, `complex`, `ElementCompute`, `alpha`, `beta` 实现或连接当前测试场景所需的逻辑。

### Lines 154-160

```cpp
    // Waive test if insufficient CUDA device
    if (!sufficient()) {
      if (CUTLASS_TEST_UNIT_ENABLE_WARNINGS) {
        std::cerr << "Test waived due to insufficient CUDA device." << std::endl;
      }
      return true;
    }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 162-162

```cpp
    initialize();
```
- **EN:** Declares member fields or local variables related to `initialize` for later setup, execution, or verification.
- **CN:** 声明与 `initialize` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 164-164

```cpp
    int batch_count = 1;
```
- **EN:** Declares member fields or local variables related to `batch_count` for later setup, execution, or verification.
- **CN:** 声明与 `batch_count` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 166-169

```cpp
    ElementA *ptr_A = tensor_A.device_data();
    ElementB *ptr_B = tensor_B.device_data();
    ElementC *ptr_C = tensor_C.device_data();
    ElementC *ptr_D = tensor_D.device_data();
```
- **EN:** Declares member fields or local variables related to `ElementA`, `ptr_A`, `tensor_A`, `device_data`, `ElementB` for later setup, execution, or verification.
- **CN:** 声明与 `ElementA`, `ptr_A`, `tensor_A`, `device_data`, `ElementB` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 171-174

```cpp
    typename LayoutA::Stride::Index lda = tensor_A.layout().stride(0);
    typename LayoutB::Stride::Index ldb = tensor_B.layout().stride(0);
    typename LayoutC::Stride::Index ldc = tensor_C.layout().stride(0);
    typename LayoutC::Stride::Index ldd = tensor_D.layout().stride(0);
```
- **EN:** Declares member fields or local variables related to `LayoutA`, `Stride`, `Index`, `lda`, `tensor_A` for later setup, execution, or verification.
- **CN:** 声明与 `LayoutA`, `Stride`, `Index`, `lda`, `tensor_A` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 176-179

```cpp
    int64_t imag_stride_A = tensor_A.imaginary_stride();
    int64_t imag_stride_B = tensor_B.imaginary_stride();
    int64_t imag_stride_C = tensor_C.imaginary_stride();
    int64_t imag_stride_D = tensor_D.imaginary_stride();
```
- **EN:** Declares member fields or local variables related to `int64_t`, `imag_stride_A`, `tensor_A`, `imaginary_stride`, `imag_stride_B` for later setup, execution, or verification.
- **CN:** 声明与 `int64_t`, `imag_stride_A`, `tensor_A`, `imaginary_stride`, `imag_stride_B` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 181-183

```cpp
    //
    // Launch device kernel
    //
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 185-185

```cpp
    Gemm gemm_op;
```
- **EN:** Declares member fields or local variables related to `Gemm`, `gemm_op` for later setup, execution, or verification.
- **CN:** 声明与 `Gemm`, `gemm_op` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 187-204

```cpp
    typename Gemm::Arguments args{
      cutlass::gemm::GemmUniversalMode::kGemm,
      problem_size,
      batch_count,
      {alpha, beta},
      ptr_A,
      ptr_A + imag_stride_A,
      ptr_B,
      ptr_B + imag_stride_B,
      ptr_C,
      ptr_C + imag_stride_C,
      ptr_D,
      ptr_D + imag_stride_D,
      lda,
      lda,
      ldb,
      ldb,
      ldc,
```
- **EN:** Implements or wires together logic around `Gemm`, `Arguments`, `args`, `gemm`, `GemmUniversalMode` for the current test scenario.
- **CN:** 围绕 `Gemm`, `Arguments`, `args`, `gemm`, `GemmUniversalMode` 实现或连接当前测试场景所需的逻辑。

### Lines 205-208

```cpp
      ldc,
      ldd,
      ldd
    };
```
- **EN:** Implements or wires together logic around `ldc`, `ldd` for the current test scenario.
- **CN:** 围绕 `ldc`, `ldd` 实现或连接当前测试场景所需的逻辑。

### Lines 210-210

```cpp
    cutlass::Status status = gemm_op(args);
```
- **EN:** Declares member fields or local variables related to `Status`, `status`, `gemm_op`, `args` for later setup, execution, or verification.
- **CN:** 声明与 `Status`, `status`, `gemm_op`, `args` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 212-212

```cpp
    EXPECT_EQ(status, cutlass::Status::kSuccess);
```
- **EN:** Declares member fields or local variables related to `EXPECT_EQ`, `status`, `Status`, `kSuccess` for later setup, execution, or verification.
- **CN:** 声明与 `EXPECT_EQ`, `status`, `Status`, `kSuccess` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 214-214

```cpp
    cudaError_t error = cudaDeviceSynchronize();
```
- **EN:** Declares member fields or local variables related to `cudaError_t`, `error`, `cudaDeviceSynchronize` for later setup, execution, or verification.
- **CN:** 声明与 `cudaError_t`, `error`, `cudaDeviceSynchronize` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 216-216

```cpp
    tensor_D.sync_host();
```
- **EN:** Declares member fields or local variables related to `tensor_D`, `sync_host` for later setup, execution, or verification.
- **CN:** 声明与 `tensor_D`, `sync_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 218-220

```cpp
    //
    // Compute reference
    //
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 222-237

```cpp
    cutlass::reference::host::GemmPlanarComplex<
      ElementA, LayoutA,
      ElementB, LayoutB,
      ElementC, LayoutC,
      ElementAccumulator
    >(
      problem_size,
      alpha,
      tensor_A.host_ref(),
      Gemm::kTransformA,
      tensor_B.host_ref(),
      Gemm::kTransformB,
      beta,
      tensor_C.host_ref(),
      tensor_D_ref.host_ref()
    );
```
- **EN:** Implements or wires together logic around `reference`, `host`, `GemmPlanarComplex`, `ElementA`, `LayoutA` for the current test scenario.
- **CN:** 围绕 `reference`, `host`, `GemmPlanarComplex`, `ElementA`, `LayoutA` 实现或连接当前测试场景所需的逻辑。

### Lines 239-242

```cpp
    bool passed = cutlass::reference::host::TensorEquals(
      tensor_D.host_view(), 
      tensor_D_ref.host_view()
    );
```
- **EN:** Implements or wires together logic around `passed`, `reference`, `host`, `TensorEquals`, `tensor_D` for the current test scenario.
- **CN:** 围绕 `passed`, `reference`, `host`, `TensorEquals`, `tensor_D` 实现或连接当前测试场景所需的逻辑。

### Lines 244-244

```cpp
    EXPECT_TRUE(passed);
```
- **EN:** Declares member fields or local variables related to `EXPECT_TRUE`, `passed` for later setup, execution, or verification.
- **CN:** 声明与 `EXPECT_TRUE`, `passed` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 246-247

```cpp
    if (!passed) {
      std::ofstream output("gemm_planar_complex.txt");
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 249-257

```cpp
      output
        << "A:\n" << tensor_A.host_view() << "\n"
        << "B:\n" << tensor_B.host_view() << "\n"
        << "C:\n" << tensor_C.host_view() << "\n"
        << "Reference:\n"
        << tensor_D_ref.host_view() << "\n"
        << "Computed:\n"
        << tensor_D.host_view() << "\n";
    }
```
- **EN:** Implements or wires together logic around `output`, `tensor_A`, `host_view`, `tensor_B`, `tensor_C` for the current test scenario.
- **CN:** 围绕 `output`, `tensor_A`, `host_view`, `tensor_B`, `tensor_C` 实现或连接当前测试场景所需的逻辑。

### Lines 259-261

```cpp
    return passed;
  }
};
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 263-264

```cpp
template <typename Gemm>
bool TestOneGemmPlanarComplex(cutlass::gemm::GemmCoord problem_size) {
```
- **EN:** Implements or wires together logic around `Gemm`, `TestOneGemmPlanarComplex`, `gemm`, `GemmCoord`, `problem_size` for the current test scenario.
- **CN:** 围绕 `Gemm`, `TestOneGemmPlanarComplex`, `gemm`, `GemmCoord`, `problem_size` 实现或连接当前测试场景所需的逻辑。

### Lines 266-266

```cpp
  TestbedPlanarComplex<Gemm> testbed(problem_size);
```
- **EN:** Declares member fields or local variables related to `TestbedPlanarComplex`, `Gemm`, `testbed`, `problem_size` for later setup, execution, or verification.
- **CN:** 声明与 `TestbedPlanarComplex`, `Gemm`, `testbed`, `problem_size` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 268-269

```cpp
  return testbed.run();
}
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 271-272

```cpp
template <typename Gemm>
bool TestAllGemmPlanarComplex() {
```
- **EN:** Implements or wires together logic around `Gemm`, `TestAllGemmPlanarComplex` for the current test scenario.
- **CN:** 围绕 `Gemm`, `TestAllGemmPlanarComplex` 实现或连接当前测试场景所需的逻辑。

### Lines 274-276

```cpp
  int M[] = {
    16, 64, 72, 144, 264, 520,
  };
```
- **EN:** Implements or wires together logic around `(none)` for the current test scenario.
- **CN:** 围绕 `(none)` 实现或连接当前测试场景所需的逻辑。

### Lines 278-280

```cpp
  int N[] = {
    16, 64, 72, 144, 248, 264, 520
  };
```
- **EN:** Implements or wires together logic around `(none)` for the current test scenario.
- **CN:** 围绕 `(none)` 实现或连接当前测试场景所需的逻辑。

### Lines 282-284

```cpp
  int K[] = {
    8, 64, 72, 96,  264, 520
  };
```
- **EN:** Implements or wires together logic around `(none)` for the current test scenario.
- **CN:** 围绕 `(none)` 实现或连接当前测试场景所需的逻辑。

### Lines 286-286

```cpp
  using ElementCompute = typename Gemm::EpilogueOutputOp::ElementCompute;
```
- **EN:** Defines aliases such as `ElementCompute` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ElementCompute`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 288-290

```cpp
  cutlass::complex<ElementCompute> alpha_values[] = {
    {ElementCompute(1.25), ElementCompute(-0.5)}
  };
```
- **EN:** Implements or wires together logic around `complex`, `ElementCompute`, `alpha_values` for the current test scenario.
- **CN:** 围绕 `complex`, `ElementCompute`, `alpha_values` 实现或连接当前测试场景所需的逻辑。

### Lines 292-294

```cpp
  cutlass::complex<ElementCompute> beta_values[] = {
    {ElementCompute(-2.25), ElementCompute(1.5)}
  };
```
- **EN:** Implements or wires together logic around `complex`, `ElementCompute`, `beta_values` for the current test scenario.
- **CN:** 围绕 `complex`, `ElementCompute`, `beta_values` 实现或连接当前测试场景所需的逻辑。

### Lines 296-298

```cpp
  for (int m : M) {
    for (int n : N) {
      for (int k : K) {
```
- **EN:** Begins function or method `for`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `for`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 300-300

```cpp
        test::gemm::device::TestbedPlanarComplex<Gemm> testbed({m, n, k});
```
- **EN:** Declares member fields or local variables related to `test`, `gemm`, `device`, `TestbedPlanarComplex`, `Gemm` for later setup, execution, or verification.
- **CN:** 声明与 `test`, `gemm`, `device`, `TestbedPlanarComplex`, `Gemm` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 302-303

```cpp
        for (auto const &alpha : alpha_values) {
          for (auto const &beta : beta_values) {
```
- **EN:** Begins function or method `for`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `for`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 305-313

```cpp
            bool passed = testbed.run(alpha, beta);
            if (!passed) {
              return false;
            }            
          }
        }
      }
    }
  }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 315-316

```cpp
  return true;
}
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 318-318

```cpp
////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 320-322

```cpp
} // namespace device
} // namespace gemm
} // namespace test
```
- **EN:** Opens or closes namespaces so the declarations live in the expected CUTLASS or test scope.
- **CN:** 打开或关闭命名空间，使这些声明位于预期的 CUTLASS 或测试作用域中。

### Lines 324-324

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
- **EN:** Epilogue fusion, callbacks, and post-processing composition.  
  **CN:** Epilogue 融合、回调与后处理组合。
- **EN:** CUTLASS device, collective, epilogue, and reference utilities.  
  **CN:** CUTLASS 的 device、collective、epilogue 与参考实现工具。
- **EN:** GoogleTest-based unit validation.  
  **CN:** 基于 GoogleTest 的单元验证。

## Dependencies / 依赖关系

- `iostream`
- `fstream`
- `sstream`
- `../../common/cutlass_unit_test.h`
- `cutlass/util/distribution.h`
- `cutlass/util/reference/host/gemm_planar_complex.h`
- `cutlass/util/host_tensor_planar_complex.h`
- `cutlass/util/tensor_view_io.h`
- `cutlass/util/reference/host/tensor_compare.h`
- `cutlass/util/reference/host/tensor_copy.h`
- `cutlass/util/reference/host/tensor_fill.h`
- CUTLASS template APIs and namespaces form the main external programming model / CUTLASS 模板 API 与命名空间构成主要外部编程模型
